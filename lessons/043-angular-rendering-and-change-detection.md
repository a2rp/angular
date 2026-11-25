# 43. Angular rendering and change detection

Angular components describe a view: a template, its bindings, and child views. Change detection is Angular's process for checking the current values of those bindings and refreshing the DOM when a rendered value has changed. Rendering is the result: the browser DOM reflects the latest application state.

A state update does not mean that Angular blindly recreates the page. Angular schedules work, checks the relevant view tree, compares binding values, and writes the necessary DOM updates. Understanding this cycle helps explain signals, lifecycle hooks, zoneless applications, and the change-detection strategies covered in the next chapter.

## Learning goals

After this lesson, you should be able to:

- Distinguish application state, change detection, and DOM rendering.
- Describe the high-level steps in an Angular update cycle.
- Identify common notifications that schedule Angular synchronization.
- Explain the role of ZoneJS and how zoneless Angular schedules updates.
- Understand how lifecycle hooks fit into a view traversal.
- Use `markForCheck()` and `detectChanges()` for their distinct purposes.
- Recognize unstable template bindings and the development-mode `NG0100` error.
- Avoid frequent hooks and side-effecting template expressions.

## State, change detection, and rendering

These ideas are related but are not the same:

- **State** is the data held by signals, component fields, services, forms, and other sources.
- **Change detection** is Angular checking template expressions and deciding whether rendered bindings need updates.
- **Rendering** is Angular applying changes to the DOM so the user sees the current state.

For example, calling `.set()` changes a signal's state. If a template reads that signal, Angular is notified that a view depending on it needs to be refreshed. During the scheduled synchronization, Angular evaluates that binding and updates the DOM if its displayed value changed.

~~~ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <p>Count: {{ count() }}</p>
    <button type="button" (click)="increment()">Add one</button>
  `,
})
export class CounterComponent {
  readonly count = signal(0);

  increment(): void {
    this.count.update((value) => value + 1);
  }
}
~~~

The template read makes the component a consumer of `count`. Updating `count` notifies Angular and schedules the view to be refreshed. The DOM text changes when Angular renders the new value.

## The high-level update cycle

A useful conceptual model is:

1. **Something changes or Angular is notified.** A user event updates state, an input changes, a template-read signal changes, or an Angular API marks a view for checking.
2. **Angular schedules synchronization.** The scheduler decides when to process pending work. ZoneJS may provide scheduling in older/Zone-based applications; modern zoneless applications use Angular notifications.
3. **Angular traverses views.** It walks the component/view tree and checks eligible views and their template bindings.
4. **Lifecycle work runs in its documented phase.** Hooks such as `ngOnChanges` and `ngOnInit` have defined positions relative to the component's template checks.
5. **Angular updates changed bindings in the DOM.** Values that have not changed do not need a DOM write.
6. **Render callbacks run after rendering.** `afterNextRender()` and `afterEveryRender()` run after Angular has rendered the page in the browser.

This is a mental model, not a promise about every internal scheduler detail. Do not make application correctness depend on an assumed exact number of checks or on the browser painting immediately after a callback.

## View traversal and lifecycle hooks

Angular checks the application tree from top to bottom. During a traversal, each component is visited once. Lifecycle hooks run during specific parts of the component lifecycle and can run while Angular is traversing the tree.

For example, the first `ngOnChanges` runs before `ngOnInit`, and both run before the component's own template is initialized. `ngAfterViewInit` runs after the component's view has initialized. `afterNextRender()` is different: it is an application-wide render callback invoked after Angular has rendered all components to the DOM.

Avoid changing already-checked state in the middle of the traversal. A child changing a parent's binding from a late hook can leave the view unstable and lead to an `ExpressionChangedAfterItHasBeenCheckedError` in development mode.

### Hooks that run frequently

`ngDoCheck`, `ngAfterContentChecked`, and `ngAfterViewChecked` run frequently when their associated views are checked. They are useful for narrow advanced integrations, but most code should not use them for routine state derivation, subscriptions, or DOM updates. Prefer inputs, signals, computed values, event handlers, and focused render callbacks.

## What schedules an update?

The mechanism depends on the application configuration and Angular version.

### Angular notifications

Angular can schedule synchronization in response to framework-aware notifications, including:

- A bound template or host event listener runs.
- A component input is set through Angular, such as a template binding or `ComponentRef.setInput()`.
- A signal read by a template changes.
- A view is marked for checking with `ChangeDetectorRef.markForCheck()` or an API such as `AsyncPipe` that marks it.
- A marked view is attached or removed.

A signal used only in TypeScript does not automatically mark a component whose template never reads that signal. Make state dependencies visible through template reads, or use the appropriate Angular notification API.

### ZoneJS and zoneless scheduling

Historically, ZoneJS patched asynchronous browser APIs and helped Angular notice that application work had occurred. A Zone-based application could schedule change detection after events, timers, and other asynchronous work even if application code did not explicitly mark a view.

Zoneless Angular does not rely on ZoneJS async-task tracking. It schedules work from Angular notifications such as changed template-read signals, bound listeners, input updates, and `markForCheck()`. Angular has been zoneless by default since v21. Applications on Angular 20 can opt in using `provideZonelessChangeDetection()`; older versions and existing applications may have different configuration.

A raw field changed from an unrelated callback is not a reliable zoneless notification:

~~~ts
// In a zoneless application, this assignment alone does not notify Angular.
this.status = 'Connected';
~~~

For template state, a signal is often the clearest choice:

~~~ts
readonly status = signal('Disconnected');

handleConnection(): void {
  this.status.set('Connected');
}
~~~

If an external API requires a plain field, call `markForCheck()` when the view should be checked, or bridge the external source through an Angular-aware API.

## `ChangeDetectorRef`: mark versus detect

`ChangeDetectorRef` exposes APIs for explicitly controlling view checks. Most applications should first use normal framework notifications such as signals and template events.

### `markForCheck()` schedules the view for a future check

It marks a view as needing refresh. It does not synchronously render the DOM at that exact line:

~~~ts
import { ChangeDetectorRef, Component, inject } from '@angular/core';

@Component({
  selector: 'app-legacy-status',
  template: `<p>{{ status }}</p>`,
})
export class LegacyStatusComponent {
  private readonly changeDetector = inject(ChangeDetectorRef);
  status = 'Waiting';

  handleExternalStatus(value: string): void {
    this.status = value;
    this.changeDetector.markForCheck();
  }
}
~~~

This is useful when integrating a non-Angular callback that updates plain component state. For new state, a signal read by the template is often more direct.

### `detectChanges()` checks immediately

`detectChanges()` checks the current view and its children immediately. It is mainly useful with deliberately detached views or carefully controlled local rendering. It is not a general-purpose way to make every update work; frequent manual calls can hide missing notifications and make rendering order harder to reason about.

`detach()` removes a view from the normal change-detection tree. A detached view is not checked during normal runs even if it is marked dirty. `reattach()` restores it. These APIs are specialized and should be used only when a local rendering strategy is intentional.

## Binding evaluation and stable templates

Angular evaluates template bindings when their view is checked. A method or getter used in a template may therefore run more often than expected. Keep template expressions fast, deterministic, and free of side effects.

~~~html
<!-- Keep this method cheap and side-effect free if used in a binding. -->
<p>{{ displayName() }}</p>
~~~

For an expensive derivation over reactive state, a `computed()` signal can cache its value until a dependency changes. Avoid starting requests, mutating state, or registering listeners from template expressions or frequently invoked lifecycle hooks.

## Development-mode stability checks and `NG0100`

In development mode, Angular performs an additional check after a change-detection run to detect bindings that changed after they were already checked. If a binding changes during the same overall cycle, Angular can report `ExpressionChangedAfterItHasBeenCheckedError` (`NG0100`). The error helps find unstable views that could otherwise show inconsistent values or cause repeated work.

Common causes include:

- A child updates a parent value after the parent has already been checked.
- A late lifecycle hook changes a binding that was already rendered.
- A template method returns a different value each time it is called.
- Code performs a state update as a side effect of reading a template binding.

Fix the state flow or choose the lifecycle phase that matches the work. Do not suppress the error by adding arbitrary timers or forcing repeated detection; that can hide the underlying ordering problem.

## DOM work after Angular renders

Use template bindings, queries, and normal component APIs for most UI work. When imperative DOM work genuinely must happen after Angular commits a render, `afterNextRender()` and `afterEveryRender()` are available. They must be registered in an injection context and do not run during server-side rendering or build-time prerendering.

Render callbacks support phases such as `write` and `read`. Separating DOM writes from layout reads helps avoid unnecessary layout recalculation. Prefer `afterNextRender()` for one-time post-render work and `afterEveryRender()` only when work is needed after every render. If the task is to observe element size or intersection, browser observer APIs may be more appropriate.

## Version note: default change-detection strategy

Defaults have changed across Angular versions:

- Angular v22 uses `ChangeDetectionStrategy.OnPush` by default.
- In earlier versions, the default strategy was `Eager` (formerly also named `Default`).
- `OnPush` changes which subtrees Angular needs to refresh; the next chapter covers its behavior and immutable update patterns in detail.

When maintaining an existing project, check its Angular version and any explicit `changeDetection` setting instead of assuming the latest default.

## Common mistakes

- **Assuming a state mutation always updates the DOM immediately:** Angular needs a notification and a scheduled render.
- **Treating change detection as recreating the whole page:** Angular checks bindings and applies required DOM updates.
- **Changing state during a view traversal:** update state in the correct event or lifecycle phase so views remain stable.
- **Using `ngDoCheck` or checked hooks for routine work:** these run frequently and can add cost.
- **Calling `detectChanges()` after every update:** prefer an Angular notification; reserve manual detection for a deliberate local strategy.
- **Reading a signal only in TypeScript and expecting its template consumer to be notified:** the component's template must read it, or another notification mechanism must mark the view.
- **Relying on ZoneJS in a zoneless app:** update a template-read signal or call an Angular notification API.
- **Adding arbitrary `setTimeout()` calls to fix `NG0100`:** correct the unstable state flow or lifecycle timing instead.

## Key takeaways

- Change detection checks view bindings; rendering applies changed values to the DOM.
- Angular traverses the component/view tree, and lifecycle hooks occur at defined points during that work.
- Angular-aware notifications tell the scheduler which views need attention.
- ZoneJS used to infer work from asynchronous browser activity; modern Angular is zoneless by default from v21.
- `markForCheck()` marks a view for a future check; `detectChanges()` checks a view and children immediately.
- Stable, side-effect-free templates and lifecycle code prevent unnecessary work and `NG0100` errors.
- Angular v22 defaults to `OnPush`; check the next chapter for subtree behavior and immutable updates.

## Official references

- [Angular component lifecycle](https://angular.dev/guide/components/lifecycle)
- [Angular advanced component configuration](https://angular.dev/guide/components/advanced-configuration)
- [Angular zoneless guide](https://angular.dev/guide/experimental/zoneless)
- [Angular `ChangeDetectorRef` API](https://angular.dev/api/core/ChangeDetectorRef)
- [Angular `NG0100` error guide](https://angular.dev/errors/NG0100)
