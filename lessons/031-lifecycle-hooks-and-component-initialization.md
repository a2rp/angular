# 31. Lifecycle hooks and component initialization

An Angular component moves through creation, input updates, content and view checks, rendering, and destruction. Lifecycle hooks let a component run code at specific points in that process.

The most useful initialization sequence is:

~~~text
constructor
  → ngOnChanges (first input change, if any)
  → ngOnInit
  → ngDoCheck
  → ngAfterContentInit
  → ngAfterContentChecked
  → ngAfterViewInit
  → ngAfterViewChecked
  → afterNextRender / afterEveryRender callbacks
~~~

On later checks, Angular calls `ngOnChanges` when inputs change, then check hooks and the relevant content/view checked hooks. Render callbacks run after Angular has rendered the application. Before destruction, Angular calls `ngOnDestroy` and registered `DestroyRef` callbacks.

## Learning goals

After this lesson, you should be able to:

- Distinguish the JavaScript constructor from Angular lifecycle hooks.
- Choose between `ngOnChanges` and `ngOnInit` for input-dependent initialization.
- Know when content and view queries are ready.
- Avoid expensive work in hooks that run on every check.
- Use `afterNextRender` or `afterEveryRender` for appropriate post-render DOM work.
- Clean up resources with `ngOnDestroy`, `DestroyRef`, or `takeUntilDestroyed`.

## Constructor and input initialization

The constructor runs when Angular creates the component instance. It is ordinary TypeScript class construction, not a lifecycle hook. Use it for dependency injection and simple instance setup.

Angular sets bound inputs as part of initialization after the instance is constructed. Do not assume a parent-provided input is ready in the constructor. Use `ngOnInit` for one-time setup that needs the initial inputs, or `ngOnChanges` when work must respond to each input change.

## `ngOnChanges`

Angular calls `ngOnChanges` after one or more inputs change and before it checks the component's own template. During initialization, the first `ngOnChanges` runs before `ngOnInit`.

The hook receives `SimpleChanges`, a record of `SimpleChange` objects containing previous value, current value, and `firstChange`:

~~~ts
import { Component, input, OnChanges, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-profile',
  standalone: true,
  template: '<p>{{ name() }}</p>',
})
export class Profile implements OnChanges {
  readonly name = input.required<string>();

  ngOnChanges(changes: SimpleChanges<Profile>): void {
    const nameChange = changes.name;
    if (nameChange) {
      console.log(nameChange.previousValue, nameChange.currentValue);
      console.log('First change:', nameChange.firstChange);
    }
  }
}
~~~

`ngOnChanges` works with signal-based and decorator-based inputs. If an input uses a template alias, the `SimpleChanges` key is still the TypeScript property name.

Use this hook for an imperative response to a changed input, such as resetting an internal non-derived workflow. If a value is simply derived from inputs, prefer a `computed()` signal instead of copying it in `ngOnChanges`.

## `ngOnInit`

Angular calls `ngOnInit` exactly once after it has initialized the component's inputs with their initial values and before it initializes the component's own template. It is useful for one-time setup that depends on those initial values.

~~~ts
import { Component, OnInit, input } from '@angular/core';

@Component({
  selector: 'app-greeting',
  standalone: true,
  template: '<p>Hello, {{ name() }}.</p>',
})
export class Greeting implements OnInit {
  readonly name = input.required<string>();

  ngOnInit(): void {
    console.log('Initial name:', this.name());
  }
}
~~~

The hook runs once. It does not rerun when the parent changes an input later; use `ngOnChanges` or reactive signal derivation for subsequent updates.

## `ngDoCheck`

Angular calls `ngDoCheck` whenever it checks a component for changes. It can support specialized manual change detection, but it runs frequently and may hurt performance. Avoid it unless the normal signal and input mechanisms cannot express the requirement.

## Content lifecycle hooks

`ngAfterContentInit` runs once after the component's projected content has been initialized. It is a lifecycle point for reading content query results in decorator-based query code.

`ngAfterContentChecked` runs whenever the projected content has been checked. It runs frequently; avoid expensive work there.

Do not use these hooks to change state that an ancestor's template has already rendered. Such changes can cause `ExpressionChangedAfterItHasBeenCheckedError`.

## View lifecycle hooks

`ngAfterViewInit` runs once after the component's own view and its child components have been initialized. It is a lifecycle point for reading view query results in decorator-based query code.

`ngAfterViewChecked` runs whenever the component's view has been checked. It also runs frequently, so keep it out of routine rendering and computation.

The signal query APIs from the previous chapter provide reactive query results directly; in new code, a lifecycle hook is usually not needed just to read a signal query.

## Render callbacks

`afterNextRender()` registers a callback for the next time Angular finishes rendering all components to the DOM. `afterEveryRender()` registers a callback for every completed application render. These are functions, not class methods, and they must be called in an injection context, typically a constructor.

Use render callbacks for work that truly needs the rendered DOM, such as initializing a non-Angular charting library. They run in browser environments only; they do not run during server-side rendering or build-time prerendering. Components are not guaranteed to be hydrated before a callback runs, so use care with direct DOM access.

When reading or writing layout, specify phases where possible to reduce layout thrashing:

1. `earlyRead` for necessary layout reads before writes.
2. `write` for DOM writes.
3. `mixedReadWrite` for operations that cannot be separated; avoid it when possible.
4. `read` for DOM reads after writes.

Prefer Angular templates and bindings for normal UI updates. Render callbacks are for DOM work that cannot be expressed through Angular's regular rendering model.

## Destruction and cleanup

`ngOnDestroy` runs once just before Angular destroys a component. This happens when an `@if` removes it, a route changes, or its owning view is destroyed. Clean up manually managed resources such as timers, event listeners, or subscriptions here.

`DestroyRef` lets you register cleanup next to the code that creates a resource:

~~~ts
import { Component, DestroyRef, inject } from '@angular/core';

@Component({
  selector: 'app-resource-owner',
  standalone: true,
  template: '',
})
export class ResourceOwner {
  private readonly destroyRef = inject(DestroyRef);

  constructor() {
    this.destroyRef.onDestroy(() => {
      console.log('Release component-owned resources here.');
    });
  }
}
~~~

`DestroyRef.destroyed` can help delayed or asynchronous code avoid acting on a component after it has been destroyed. For RxJS subscriptions, `takeUntilDestroyed()` completes the observable subscription with the current destruction scope.

## Lifecycle interfaces

Angular provides TypeScript interfaces such as `OnInit`, `OnChanges`, `AfterViewInit`, and `OnDestroy`. Implementing an interface is optional at runtime, but it lets TypeScript catch misspelled hook names and signatures:

~~~ts
export class Example implements OnInit {
  ngOnInit(): void {
    // One-time initialization.
  }
}
~~~

The method name is what Angular calls; the interface helps type-check your implementation.

## Keep hooks small and avoid changing state mid-check

Angular traverses the component tree from top to bottom while checking bindings. It visits each component once during a traversal. Avoid changing already-checked state in the middle of this process, particularly from `ngAfterContentInit`, `ngAfterContentChecked`, `ngAfterViewInit`, or `ngAfterViewChecked`.

Prefer this order of choices:

- Use an input and a `computed()` signal for values derived from inputs.
- Use `ngOnChanges` for an imperative response to input changes.
- Use `ngOnInit` for one-time setup after initial inputs are ready.
- Use content/view hooks only for work that depends on the corresponding initialized children or query results.
- Use render callbacks for necessary post-render DOM integration.
- Use destruction hooks or `DestroyRef` to release owned resources.

## Common mistakes

- **Reading parent inputs in the constructor:** inputs are not initialized there; use `ngOnInit` or `ngOnChanges`.
- **Expecting `ngOnInit` to run for every input update:** it runs once. Use `ngOnChanges` or signal derivation for later changes.
- **Doing derived-state work in `ngOnChanges`:** a `computed()` signal is usually clearer for values derived from inputs.
- **Putting expensive work in `ngDoCheck` or checked hooks:** these run frequently and can slow rendering.
- **Updating already-checked state in an after-content or after-view hook:** this can trigger an expression-changed error.
- **Using `ngAfterViewInit` for arbitrary DOM manipulation:** prefer bindings; use render callbacks for required post-render DOM integrations.
- **Assuming render callbacks run on the server:** they run only in browser environments and are not a server-rendering hook.
- **Forgetting cleanup:** release resources with `ngOnDestroy`, `DestroyRef`, or `takeUntilDestroyed()`.
- **Assuming a component instance hook order relative to directives on the same element:** Angular does not guarantee that ordering.

## Practical checklist

Before adding a lifecycle hook, ask:

- Do I need initial input values (`ngOnInit`) or must I respond to every input change (`ngOnChanges`)?
- Is this value derived and better represented by `computed()`?
- Do I need initialized projected content or my own view (`ngAfterContentInit` / `ngAfterViewInit`)?
- Will this hook run on every check, and can its work be moved elsewhere?
- Does this operation truly require the rendered DOM (`afterNextRender` / `afterEveryRender`)?
- Does the resource need cleanup, and can `DestroyRef` keep setup and cleanup together?

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="030-view-queries-and-content-queries.md">030 - 30. View queries and content queries</a></td><td><a href="../qnas/031-lifecycle-hooks-and-component-initialization.md">Q&amp;A</a></td><td><a href="../examples/031-lifecycle-hooks-and-component-initialization.md">Example</a></td><td><a href="032-host-bindings-host-listeners-and-host-directives.md">032 - 32. Host bindings, host listeners, and host directives</a></td>
</tr>
  </tbody>
</table>
