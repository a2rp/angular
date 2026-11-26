# 28. Q&A: Parent-child communication and one-way data flow

## 1. What does one-way data flow mean in a component tree?

A parent passes current values down to children through inputs. Children report actions or events upward through outputs. The component that owns the shared state handles the event, updates that state, and passes the new value down again.

## 2. Does “one-way” mean a child can never communicate with its parent?

No. It describes how values and events are managed: values flow down, while events flow up. A child communicates with an output, and the parent decides what the event means for shared state.

## 3. What is the source of truth?

It is the component or service that owns the authoritative value. Other components render values derived from or supplied by that owner rather than keeping competing copies of the same shared state.

## 4. Which component should own a value?

Place state in the lowest common ancestor that needs to coordinate it. If one child alone uses a temporary value, keep it in that child. If a parent needs to share the value among children, summarize it, or persist it, the parent is often a suitable owner.

## 5. How does a parent pass data to a child?

Declare an input in the child and bind a value on the child element:

~~~ts
readonly title = input.required<string>();
~~~

~~~html
<app-heading [title]="pageTitle" />
~~~

The child reads a signal input by calling it, such as `title()`.

## 6. Why should a child treat an input as read-only?

The parent owns and supplies the value. A child that changes it behind the parent's back makes ownership unclear and can leave sibling views or parent summaries out of sync. The child should report the requested change and let the owner perform it.

## 7. Are input objects deeply immutable?

No. A read-only input signal prevents the child from setting the signal itself, but it does not deep-freeze an object passed through the input. Avoid mutating nested input data; report an event and let the owner create and publish the intended update.

## 8. How does a child report an action to the parent?

Declare a typed output and emit the information the parent needs:

~~~ts
readonly deleteRequested = output<number>();

requestDelete(): void {
  this.deleteRequested.emit(this.item().id);
}
~~~

The parent listens with `(deleteRequested)="deleteItem($event)"`.

## 9. What should a child output communicate?

It should describe a meaningful event or intent, such as `deleteRequested`, `saved`, or `selectionChanged`. It should not expose private implementation details or require the parent to inspect the child's internal state.

## 10. Does an output automatically update an input or parent state?

No. The parent must listen and decide how to respond. An output reports an event; it does not automatically mutate the parent's state or update another child's input.

## 11. Do Angular outputs bubble to ancestors?

No. Angular custom outputs do not bubble through the DOM. A listener must be attached to the component or directive that declares the output.

## 12. How can a grandparent receive an event from a nested child?

An intermediate component can listen to the child's output and emit its own corresponding output. This explicit forwarding makes the path visible. If many distant components need the same state, a shared state service may fit better.

## 13. How should sibling components communicate?

Use their common parent as the coordinator. The parent supplies current state to each sibling through inputs. One sibling emits an event, the parent updates the shared state, and Angular passes the new values to the other siblings.

## 14. Should sibling components call each other directly?

Usually no. Direct references couple the siblings and make reuse harder. A common parent or an appropriate shared service gives the relationship a clearer owner.

## 15. How can a parent update one item in an array?

Handle the child's event in the parent and create an updated array. For example, use `map()` to replace the changed item while keeping the others:

~~~ts
this.items.update((items) =>
  items.map((item) =>
    item.id === id ? { ...item, active: !item.active } : item,
  ),
);
~~~

This keeps the update near the state owner and makes the state transition explicit.

## 16. Why pass an identifier in an output instead of the entire child component?

An identifier tells the parent which domain item the event concerns without exposing the child instance or its internal state. For richer changes, emit a small typed event object containing the values the parent needs.

## 17. Can a child keep local state?

Yes. Values that only affect the child's own temporary interface, such as whether a tooltip is open, can remain local. Shared, persistent, or cross-component state should have a clear owner.

## 18. What is the risk of copying an input into a local field?

The local copy can become stale when the parent changes the input. If a copy is not intentional, derive display values from the input or use a computed signal. A deliberate local draft is a valid pattern when the user should be able to edit before applying changes.

## 19. How should a child implement an edit draft?

Keep the draft local while the user edits. When the user saves, emit a semantic event such as `saveRequested` with the draft data. The parent validates, persists, or applies it and then sends the updated value back through an input.

## 20. When should I use a model input instead of separate input and output?

Use a model when the component's contract is specifically “parent provides this value; the component can update the same value,” as with a slider or custom checkbox. Use a regular input and semantic output when the parent should handle a distinct action or event.

## 21. Does a model input remove the need to think about state ownership?

No. A model supports two-way binding, but the application still needs to decide where state is owned and how it is shared. See the model inputs chapter for that API.

## 22. When is a shared service useful?

A service can help when state or communication spans distant parts of the component tree, multiple routes, or several features. It gives those parts a common coordination point without forwarding each event through unrelated intermediate components.

## 23. Should I use a service for every parent-child interaction?

No. For a small, clear component tree, parent state with child inputs and outputs is usually simpler. Add shared state only when the component hierarchy no longer represents the relationship well.

## 24. Where can Angular provide a shared service?

A service can be provided at application, route, or component scope. Application scope is appropriate for broadly shared state; route or component scope can isolate state to a feature or component subtree. Choose a scope that matches the intended lifetime and sharing.

## 25. Can a parent pass a computed value to a child?

Yes. A parent can bind any appropriate expression or computed value to an input. Keep the authoritative state in its owner and derive display data from it rather than duplicating the same state in multiple components.

## 26. How does Angular update siblings after a child event?

The child emits to its parent. The parent updates its state. Angular then reevaluates the bindings and supplies current values to the children that depend on that state.

## 27. What if the child emits but the UI does not change?

Check that the listener is attached to the correct component instance, the output name matches exactly, and the parent handler actually updates the state that is bound back to the child or displayed by the parent. An emitted event alone does not change state.

## 28. What does one-way flow make easier to debug?

It gives each value a clear owner and each update an identifiable event handler. When a value is wrong, you can inspect the owning state and the input bindings that distribute it instead of searching for hidden mutations across many components.

## 29. Should I pass an entire large object or several individual values?

Pass the data the child genuinely needs. A stable domain object is reasonable when the child displays or acts on that entity. Separate smaller inputs may be clearer when they represent unrelated configuration or when the component should depend on a narrower interface.

## 30. How do I keep a parent from becoming overloaded with child details?

Keep each child responsible for its own rendering and local interaction. Let it emit a meaningful event with the needed payload. The parent should own the shared state transition, not every low-level detail of the child's user interface.

## 31. Are custom outputs the same as browser DOM events?

No. They use similar template syntax but belong to Angular components or directives. Angular outputs do not bubble, and output names should avoid collisions with native DOM event names.

## 32. Is a parent-child output the same as a service event bus?

No. An output is a direct contract between a component and its consumer. A service is a separate shared coordination mechanism that can serve components that do not have a direct parent-child relationship.

## 33. What is a useful rule for deciding where state belongs?

Keep state as close as possible to where it is used, but high enough in the tree to coordinate every component that needs it. Promote it to a service when the natural component owner cannot serve all consumers cleanly.

## 34. What should I check before using two-way binding?

Check whether the child is genuinely editing one shared value. If so, a model input may be a concise API. If the child is reporting a separate action such as delete or save, a named output and a parent handler usually express the intent better.

## 35. Where can I read Angular's official guidance?

See Angular's input properties guide, custom events with outputs, and dependency provider guide.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="027-model-inputs-and-component-level-two-way-binding.md">027 - 27. Model inputs and component-level two-way binding</a></td><td><a href="../lessons/028-parent-child-communication-and-one-way-data-flow.md">Lesson</a></td><td><a href="../examples/028-parent-child-communication-and-one-way-data-flow.md">Example</a></td><td><a href="029-content-projection-and-multiple-slots.md">029 - 29. Content projection and multiple slots</a></td>
</tr>
  </tbody>
</table>
