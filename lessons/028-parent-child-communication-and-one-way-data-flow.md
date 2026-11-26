# 28. Parent-child communication and one-way data flow

A component tree is easier to reason about when shared state has a clear owner. In a common Angular pattern, a parent passes values down to children through inputs. A child reports user actions upward through outputs. The parent handles those events, changes its state, and passes the updated values down again.

~~~text
Parent owns state
    │
    ├── values flow down through inputs
    ▼
Child renders values and reports intent
    │
    └── events flow up through outputs
            │
            ▼
Parent updates state and renders the new values
~~~

This is often described as one-way data flow. Values travel down the component tree; events travel up to the component that owns the relevant state. It is not a restriction against child interaction: children can respond to clicks and keyboard input, but the parent remains responsible for shared application data.

## Learning goals

After this lesson, you should be able to:

- Choose a component to own a piece of state.
- Pass data to a child through an input and report intent through an output.
- Update parent-owned collections without allowing a child to mutate them directly.
- Coordinate sibling components through their common parent.
- Forward events through nested component boundaries when needed.
- Decide when local child state or a shared service is a better fit.

## The parent owns shared state

A parent is often the right owner when it needs to coordinate multiple children, display a summary, or decide how an event affects the rest of a screen. The parent stores the value and passes each child the data it needs.

A child input should be treated as data to render or use, not as a child-owned writable field. Signal inputs declared with `input()` are read-only in the child:

~~~ts
import { Component, input } from '@angular/core';

interface Task {
  id: number;
  title: string;
  completed: boolean;
}

@Component({
  selector: 'app-task-row',
  standalone: true,
  template: '<span>{{ task().title }}</span>',
})
export class TaskRow {
  readonly task = input.required<Task>();
}
~~~

The parent binds its value into the child:

~~~html
<app-task-row [task]="task" />
~~~

The input signal is read by calling it, such as `task()`. The child can use it to render the task or derive display information, but it should not attempt to call `.set()` on an input signal.

## Children report intent with outputs

When the user asks to complete a task, the row can report which task was requested. The parent handles the event and updates the collection it owns:

~~~ts
readonly toggleRequested = output<number>();

requestToggle(): void {
  this.toggleRequested.emit(this.task().id);
}
~~~

~~~html
<app-task-row
  [task]="task"
  (toggleRequested)="toggleTask($event)"
/>
~~~

The child's output communicates intent: “toggle this task.” It does not reach into the parent's task array. The parent decides how to perform that action.

Angular outputs do not bubble through the DOM. The listener is attached to the component that declares the output. To carry an event farther up a nested tree, an intermediate component must listen to the child and expose its own output, or the application can use a shared state mechanism where that is a better fit.

## Update collections in the state owner

A parent can update a collection by producing a new array. With a signal, use `.update()` and return the changed collection:

~~~ts
readonly tasks = signal<Task[]>(initialTasks);

 toggleTask(id: number): void {
  this.tasks.update((tasks) =>
    tasks.map((task) =>
      task.id === id ? { ...task, completed: !task.completed } : task,
    ),
  );
}
~~~

Here, the parent owns the task list. The child only identifies which task the user acted on. The parent can also remove an item with `filter()`, add an item with a new array, or call a service to persist a change.

Returning new objects and arrays makes the state transition explicit and avoids sharing accidental mutations between components. Keep updates near the state owner so that other views, summaries, and sibling components all receive the same current state.

## Coordinate siblings through their parent

Sibling components do not need to call each other directly. A common parent can pass state to both and route events from either one:

~~~html
<app-task-toolbar
  [total]="tasks().length"
  [completed]="completedCount()"
  (addRequested)="addTask()"
/>

@for (task of tasks(); track task.id) {
  <app-task-row
    [task]="task"
    (toggleRequested)="toggleTask($event)"
  />
}
~~~

When the row emits `toggleRequested`, the parent updates `tasks`. Angular then supplies the current data to the toolbar and each row through their inputs. This avoids a separate direct link between the toolbar and row components.

## Keep local state local

Not every value belongs in a parent. A child can own short-lived interface state that only affects itself, such as whether a tooltip is open, which tab is selected inside a private panel, or whether a local animation is active. Parent ownership is most useful when a value must be shared, persisted, summarized, or coordinated across component boundaries.

Avoid copying an input into a second mutable field without a deliberate reason. The copy can become stale when the parent changes the input. If the child needs a derived display value, use a computed signal or derive it in the template. If the child is editing a draft that should not affect the parent until Save, model that draft explicitly and send a save event when it is ready.

## When a service is a better communication path

For a small tree, route a value through the nearest common parent. If many distant components need the same state or event stream, repeatedly forwarding events through unrelated layers can become awkward. A service or dedicated state store can provide a shared point of coordination.

Choose the simplest owner that matches the state scope:

| State scope | Common approach |
| --- | --- |
| Used by one component only | Keep it in that component |
| Shared by a parent and a few children | Parent state, child inputs, and outputs |
| Shared across distant features or routes | Service or application-level state design |

A service is not automatically better. Start with the component tree when it expresses the relationship clearly; use a shared service when the state or communication genuinely crosses that tree's natural boundaries.

## One-way flow and model inputs

A model input supports a focused two-way binding contract for a value that a component is designed to edit. It does not remove the need to choose a state owner. The parent can still own the bound value while the child edits it through the model API.

Prefer regular inputs and outputs when a child is reporting distinct actions or events and the parent should decide how to handle them. Prefer a model input when the reusable component's API is specifically “provide this value, and let the control update it.” See [model inputs and component-level two-way binding](027-model-inputs-and-component-level-two-way-binding.md) for the model API.

## Common mistakes

- **Mutating an input inside a child:** treat parent-provided input values as read-only. Emit an event describing the requested change.
- **Letting multiple components own competing copies of shared state:** choose one owner and derive other views from it.
- **Making siblings communicate directly:** route their shared data and events through their common parent, unless a shared service is a better fit.
- **Expecting outputs to bubble:** explicitly forward an event through each component boundary or use shared state.
- **Copying an input into local mutable state accidentally:** the local copy can stop reflecting parent updates. Keep a draft only when that behavior is intentional.
- **Putting every interface detail in the parent:** local, private UI state can remain in its child.
- **Using an output as a hidden command:** name it as a meaningful event and pass enough data for the owner to decide what to do.
- **Using two-way binding by default:** use it when editing one shared value is the component's contract; otherwise, an input and semantic output often make ownership clearer.

## Practical checklist

When designing a parent-child interaction, ask:

- Which component owns the source of truth?
- Which values should flow to children as inputs?
- What event should a child emit when the user acts?
- Does the parent update its state and then render the updated value back down?
- Can siblings coordinate through their common parent?
- Is local child state truly private, or does another part of the app need it?
- Would forwarding events through the tree remain clear, or is shared state more appropriate?
- Is a model input more accurate than a separate event for this interaction?

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="027-model-inputs-and-component-level-two-way-binding.md">027 - 27. Model inputs and component-level two-way binding</a></td><td><a href="../qnas/028-parent-child-communication-and-one-way-data-flow.md">Q&amp;A</a></td><td><a href="../examples/028-parent-child-communication-and-one-way-data-flow.md">Example</a></td><td><a href="029-content-projection-and-multiple-slots.md">029 - 29. Content projection and multiple slots</a></td>
</tr>
  </tbody>
</table>
