# 44. OnPush change detection and immutable updates

Angular uses change detection to keep a component's rendered view in sync with application state. The OnPush strategy lets Angular skip a component subtree when Angular has no notification that the subtree needs an update. Knowing what counts as a notification—and how object identity affects it—helps prevent stale UI and avoid unnecessary checks.

OnPush has been the default change-detection strategy since Angular v22. Making it explicit in a component is still useful when reading code, documenting intent, and supporting earlier Angular versions.

## Learning goals

After this lesson, you should be able to:

- Explain what an OnPush component subtree is and when Angular checks it.
- Identify the main notifications that make an OnPush view eligible for refresh.
- Explain why mutating an object while keeping the same reference can leave a child view stale.
- Update object and array state immutably, including nested state.
- Use signals, AsyncPipe, and markForCheck() as Angular-aware update paths.
- Recognize when direct TypeScript changes to a child input need an explicit notification.
- Apply OnPush without assuming that it makes every checked template inexpensive.

## What OnPush means

A component's change-detection strategy affects when Angular checks its view and descendants. With OnPush, Angular can skip the component's subtree during a synchronization when the subtree has no new input or other relevant notification.

Skipping is temporary. The view is checked again when Angular receives a reason to refresh it. OnPush does not disable change detection, freeze a component, or require manual refresh after every normal event.

For an explicitly configured component, import ChangeDetectionStrategy and set the strategy in the component metadata:

~~~ts
import { ChangeDetectionStrategy, Component } from '@angular/core';

@Component({
  selector: 'app-summary',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: '<p>Summary content</p>',
})
export class SummaryComponent {}
~~~

Angular v22 uses OnPush by default. In older versions, the default was Eager, previously also called Default. Existing applications can also set a strategy explicitly, so check the project version and component metadata before assuming which behavior applies.

## What makes Angular check an OnPush subtree?

Angular's current guidance identifies these main notifications:

| Notification | Example |
| --- | --- |
| A template binding supplies a changed input to the subtree's root component | A parent passes a new task object to a child |
| Angular handles an event in the subtree | A click handler runs in the component or one of its descendants |
| The component is marked for checking | markForCheck() or an API that marks the view, such as AsyncPipe |
| A signal read by the component's template changes | The template reads a signal and that signal receives a new value |

For template-bound inputs, Angular compares the previous and current values using Object.is. A changed primitive value counts as a change. For an object or array, the reference must change for the input binding to count as changed.

When an event runs inside an OnPush subtree, Angular checks that subtree and its ancestors. Other OnPush subtrees outside the event path can be skipped if they did not receive changed inputs. The event does not mean every component in the application must always be refreshed.

A template-read signal is also a notification. Angular tracks which template reads which signal, so a changed signal refreshes its template consumer. A signal read only in ordinary TypeScript code does not by itself tell a component template to refresh.

## Why object mutation can leave the child stale

Consider a parent that passes a task object to an OnPush child:

~~~html
<app-task-row [task]="task" />
~~~

If the parent mutates the task but keeps the same object reference, the child input binding still points to the same object:

~~~ts
task.done = true;
~~~

The object has changed internally, but the input value's identity has not. Angular's documented input comparison sees the same reference, so it does not treat that binding as a changed input. If the child subtree has no other notification, Angular may skip it and its DOM can continue showing the old value.

This is especially easy to miss with nested objects and arrays because mutation changes their contents without changing their identity.

## Immutable updates create visible state changes

An immutable update creates a replacement for each changed object or array on the path from the state root to the changed value. Unchanged items can keep their existing identities.

### Replace one object in an array

~~~ts
function markTaskDone(tasks: Task[], taskId: number): Task[] {
  return tasks.map((task) =>
    task.id === taskId
      ? { ...task, done: true }
      : task,
  );
}
~~~

The returned array has a new reference, and the updated task has a new reference. A parent can pass the replacement object to an OnPush child:

~~~ts
this.tasks = markTaskDone(this.tasks, taskId);
~~~

With a signal, use update() to produce a replacement value:

~~~ts
readonly tasks = signal<Task[]>([]);

markTaskDone(taskId: number): void {
  this.tasks.update((tasks) =>
    tasks.map((task) =>
      task.id === taskId
        ? { ...task, done: true }
        : task,
    ),
  );
}
~~~

The new array notifies template consumers of the tasks signal. The child receiving the updated task also gets a changed input reference. Rows for untouched tasks can keep their object references and be skipped when appropriate.

### Update nested state at every changed level

Object spread is shallow. Replacing only an outer object while mutating a nested object still changes shared state in place:

~~~ts
// Still mutates the existing profile object.
const nextUser = { ...user };
nextUser.profile.name = 'Mina';
~~~

Replace every object along the changed path:

~~~ts
const nextUser = {
  ...user,
  profile: {
    ...user.profile,
    name: 'Mina',
  },
};
~~~

For an array of users, also replace the array and the target user:

~~~ts
this.users.update((users) =>
  users.map((user) =>
    user.id === userId
      ? {
          ...user,
          profile: {
            ...user.profile,
            name: 'Mina',
          },
        }
      : user,
  ),
);
~~~

The rule is not “copy every value every time.” Create new references for the changed collection and the objects along the changed path. Keep unaffected objects stable so Angular can reuse them.

### Add and remove items

Use a new array for additions and removals:

~~~ts
this.tasks.update((tasks) => [...tasks, newTask]);
this.tasks.update((tasks) => tasks.filter((task) => task.id !== taskId));
~~~

If a template renders the items with @for, track each item by a stable key such as task.id. Stable tracking keys preserve DOM identity when the collection changes; immutable object references let input bindings identify which child data changed. These solve related but different problems.

## Signals and object identity

A signal is a notification mechanism around a value. Writable signals use equality to decide whether a new value should notify consumers. The default equality is based on Object.is, so setting the same array or object reference again does not count as a changed value:

~~~ts
const current = this.tasks();
current[0].done = true;
this.tasks.set(current); // Same reference: no signal notification.
~~~

Instead, create a replacement:

~~~ts
this.tasks.update((tasks) =>
  tasks.map((task, index) =>
    index === 0 ? { ...task, done: true } : task,
  ),
);
~~~

A readonly signal prevents callers from using set() or update(), but it does not make an object stored inside the signal deeply immutable. Treat signal values as immutable by convention, or use types and state boundaries that discourage mutation.

## Events, observables, and explicit marking

### Events handled by Angular

A template event such as a button click is an Angular notification. Angular checks the event's component subtree and ancestors. Updating state from that handler is normally enough to make the relevant view eligible for checking.

This can make an in-place mutation appear to work when the event occurs inside the very component that displays the mutated value. It does not make mutation a sound pattern: another OnPush child can still receive the same input reference and be skipped, and a signal set to the same reference will not notify its consumers.

### AsyncPipe

AsyncPipe subscribes to an Observable or Promise, exposes its latest value, and marks the component for checking when a new value arrives. It also unsubscribes when the component is destroyed and switches subscriptions when the expression reference changes.

~~~html
<p>{{ status$ | async }}</p>
~~~

This is a useful way to connect asynchronous values to a template, including in an OnPush component. If the Observable emits a mutable object and consumers rely on changed input references, emit replacement values rather than repeatedly mutating and re-emitting the same object.

### markForCheck()

Use ChangeDetectorRef.markForCheck() when an update happens through an imperative path that Angular does not otherwise know about—for example, a legacy integration that writes to a plain field:

~~~ts
import {
  ChangeDetectionStrategy,
  ChangeDetectorRef,
  Component,
  inject,
} from '@angular/core';

@Component({
  selector: 'app-legacy-status',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: '<p>{{ status }}</p>',
})
export class LegacyStatusComponent {
  private readonly changeDetector = inject(ChangeDetectorRef);
  status = 'Waiting';

  receiveExternalStatus(status: string): void {
    this.status = status;
    this.changeDetector.markForCheck();
  }
}
~~~

markForCheck() marks the view so Angular can include it in a future synchronization. It is not a synchronous DOM write. For new template state, a template-read signal or an Angular event is often a clearer update path.

### Directly assigning an input through a component reference

Changing an @Input property by directly assigning it through a ViewChild or ContentChild reference does not automatically notify Angular to check an OnPush component. Prefer updating data through the template binding. When creating or controlling a component programmatically, use an Angular API that sets the input, such as ComponentRef.setInput(). If an imperative integration must directly change component state, mark the view appropriately with ChangeDetectorRef.

Do not use detectChanges() as the routine answer to every stale view. It immediately checks a local view and its children; reserve it for intentionally controlled local rendering. markForCheck() and detectChanges() have different timing and purposes.

## A practical way to reason about a stale view

When an OnPush view does not show a value you expect, check these in order:

1. **Did the state actually change?** Inspect the value before and after the update.
2. **Did Angular receive a notification?** Check for a template event, changed template input, changed signal read in the template, AsyncPipe emission, or markForCheck().
3. **Did an object input get a new reference?** Compare the previous and next object or array identities.
4. **Is the signal itself notified?** Calling set() with the same reference is not a changed value under default equality.
5. **Is a child subtree being skipped?** A parent being checked does not guarantee that every OnPush child is checked; unchanged child inputs can leave that child skipped.
6. **Is the component deliberately detached?** A detached view is not refreshed by normal change detection until it is reattached or explicitly checked.

Use Angular DevTools or focused logging to inspect the component tree and state transitions rather than adding repeated manual detection calls.

## Performance: use OnPush deliberately

OnPush gives Angular more opportunities to skip unaffected component subtrees. It does not make a checked component free: Angular still evaluates its bindings, and expensive methods or computations can still cost time when that view is refreshed.

Keep template expressions predictable and inexpensive. For derived state, consider computed signals or values calculated when the underlying state changes. Use stable @for tracking keys for collections. Profile before introducing custom equality functions, detached views, or manual detection loops.

OnPush also does not enforce immutable state. JavaScript objects remain mutable, and a readonly signal does not deeply freeze its contents. Immutability is a state-management discipline that makes changes visible to Angular's identity-based checks and easier for people to reason about.

## Common mistakes

- **Mutating an object passed to an OnPush child:** the child's input reference remains unchanged.
- **Calling signal.set() with the same mutated object:** the signal's default equality can treat it as unchanged and send no notification.
- **Copying only the outer object:** nested objects along the modified path still share the old reference.
- **Expecting a parent check to force every child check:** OnPush descendants can still be skipped if their inputs are unchanged and they have no other notification.
- **Assuming every click checks the whole application:** the event's subtree and ancestors are checked; unrelated OnPush subtrees can be skipped.
- **Treating OnPush as a deep immutability feature:** it is a rendering strategy, not a runtime freeze.
- **Using detectChanges() after every update:** prefer the normal notification path and reserve manual checks for a deliberate local-rendering design.
- **Forgetting that AsyncPipe marks a view:** avoid adding redundant manual marking for the same Observable emission.
- **Changing an input by direct TypeScript assignment and expecting Angular to notice:** use template bindings or an Angular-aware programmatic input API.

## Key takeaways

- OnPush lets Angular skip component subtrees until a relevant notification makes them eligible for checking.
- Since Angular v22, OnPush is the default strategy; older versions used Eager/Default by default.
- Template-bound inputs are compared with Object.is. New object and array references make changes visible; in-place mutation preserves identity.
- A template-read signal, an event in the subtree, AsyncPipe, or markForCheck() can notify Angular that a view needs work.
- Immutable updates replace the changed collection and each changed object along the path, while leaving unaffected values stable.
- Parent checks do not automatically force unchanged OnPush children to refresh.
- OnPush helps reduce unnecessary subtree checks, but checked templates still need to be efficient.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="043-angular-rendering-and-change-detection.md">043 - 43. Angular rendering and change detection</a></td><td><a href="../qnas/044-onpush-change-detection-and-immutable-updates.md">Q&amp;A</a></td><td><a href="../examples/044-onpush-change-detection-and-immutable-updates.md">Example</a></td><td><a href="045-zoneless-angular-applications.md">045 - 45. Zoneless Angular applications</a></td>
</tr>
  </tbody>
</table>
