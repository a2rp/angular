# 44. OnPush change detection and immutable updates - Q&A

## Core model

### 1. What does OnPush change detection mean?

It lets Angular skip a component subtree during a synchronization when there is no relevant notification that the subtree needs a refresh. The subtree is checked again when an input or another Angular notification makes it eligible.

### 2. Does OnPush turn change detection off?

No. It changes when Angular checks a view. Events, changed template-bound inputs, signals read by the template, and explicit marks can all make an OnPush view eligible for checking.

### 3. Since which Angular version is OnPush the default?

Angular v22 uses OnPush by default. Earlier versions defaulted to Eager, historically also called Default. Explicit component configuration can override the default, so check both the framework version and the component metadata.

### 4. Should I still write changeDetection: ChangeDetectionStrategy.OnPush?

It is optional for the default behavior in Angular v22+, but explicit configuration can document intent and preserve behavior for projects that use earlier Angular versions. Follow the conventions and supported versions of the project.

## Inputs and reference identity

### 5. When does a changed input make Angular check an OnPush child?

When Angular supplies the input through a template binding and the new value differs from the previous value. Angular compares the values with Object.is.

### 6. Why does changing a property on an input object not always update the child?

Changing a property mutates the existing object. The reference remains the same, so Angular does not see a changed input value by identity. If the child has no other notification, its OnPush subtree can be skipped.

### 7. Does Object.is compare object contents?

No. For objects and arrays, Object.is checks whether the two values are the same reference. It does not recursively compare their properties.

### 8. What if the input is a number or string?

If the new primitive value differs from the previous value under Object.is, Angular sees a changed input and checks the child. No object-copy pattern is needed for primitive inputs.

### 9. Does changing an array item in place count as a new input?

No. If the array reference and the item reference remain the same, an input binding that passes either one still has the same identity. Create replacement references for the changed collection and item.

### 10. If the parent gets checked, will Angular always check all its OnPush children?

No. A parent view being checked does not force every OnPush child to be checked. An unchanged child input can leave that child subtree skipped unless it has another relevant notification.

### 11. How can I update one child input immutably?

Create a new object for the changed item and pass it through the template binding. For example, map an array and replace only the item whose ID matches. Unchanged items can keep their references.

### 12. Should I clone every object in the application on every update?

No. Replace the collection and the objects along the changed path. Keep unrelated objects stable. This makes changes observable while avoiding needless recreation of every item.

## Events and subtree behavior

### 13. What happens when an event runs inside an OnPush component?

Angular checks the event's component subtree and its ancestors. It can skip unrelated OnPush subtrees outside that event path if they received no changed inputs or other notifications.

### 14. Does an event anywhere in the application always refresh every OnPush component?

No. The event's location matters. Angular checks the relevant path and skips unaffected OnPush subtrees when possible.

### 15. If a child handles a click, can its OnPush parent be checked?

Yes. Angular checks the component handling the event and its ancestors. This allows state owned by an ancestor to participate in the resulting update.

### 16. Can an event make an in-place mutation appear to work?

Yes. If the event occurs in the component that renders the mutated state, the event itself can make that view eligible for checking. But an OnPush child that receives the same object reference can still be skipped. With signals, setting the same reference also may not notify signal consumers. Prefer immutable updates.

### 17. Should I call detectChanges() in every click handler?

No. Angular template events are already notifications. Use normal event and state flow; reserve detectChanges() for a deliberate local-rendering design.

## Signals and asynchronous values

### 18. How do signals interact with OnPush?

If a component template reads a signal and that signal changes, Angular tracks the dependency and refreshes the component that reads it.

### 19. Does reading a signal only in TypeScript mark the template consumer?

Not by itself. The template should read the signal if its rendered output depends on it, or code should use another appropriate Angular notification.

### 20. Why does signal.set() with the same mutated object often fail to notify?

Writable signals use equality to decide whether a value changed. With default equality, setting the same object or array reference does not count as a new value. Use update() to return a replacement object or array.

### 21. What is a safe way to update a signal holding an array of objects?

Use update() and return a new array. In the map callback, create a new object for the changed item and return existing object references for unchanged items.

### 22. Does a readonly signal make its object value immutable?

No. A readonly signal prevents consumers from calling set() or update() on that signal, but it does not deeply freeze the object stored inside it. Avoid mutating shared signal values.

### 23. Can an Observable update an OnPush component?

Yes, when the template uses AsyncPipe, a new emission causes AsyncPipe to mark the component for checking. AsyncPipe also unsubscribes when the view is destroyed and handles changes to the Observable reference.

### 24. Should an Observable emit the same mutated object repeatedly?

Prefer emitting new objects when consumers depend on changed input references. AsyncPipe marks its own view on an emission, but a child OnPush component can still receive the same object reference and be skipped.

### 25. Does AsyncPipe require a manual markForCheck() after each emission?

No. AsyncPipe marks the component when a new value arrives. Adding a redundant mark for that same emission is unnecessary.

## Immutable update patterns

### 26. How do I immutably add an item to a signal array?

Return a new array, for example with a spread: this.items.update(items => [...items, item]).

### 27. How do I immutably remove an item?

Return a filtered array, for example with this.items.update(items => items.filter(item => item.id !== id)).

### 28. Why must nested objects also be copied?

Spread syntax is shallow. If a nested object is mutated after only copying its parent, shared state still changes in place. Copy each object from the state root down to the nested value being changed.

### 29. What is the difference between immutable updates and @for track?

Immutable updates create references that reveal which state changed. A track key helps Angular preserve DOM identity for rendered collection items. Use stable keys and immutable state for their separate purposes.

### 30. Do I need a library to make updates immutable?

No. For many updates, array methods and object spread are sufficient. Libraries may help with deeply nested or complex state, but they do not replace understanding which values are shared.

## Explicit marking and imperative APIs

### 31. When should I call ChangeDetectorRef.markForCheck()?

Use it when a plain-field update comes from an imperative path Angular does not otherwise know about, such as a legacy callback. It marks the view for a future check; it does not synchronously write to the DOM.

### 32. What is the difference between markForCheck() and detectChanges()?

markForCheck() tells Angular that a view should be included in a future synchronization. detectChanges() immediately checks that view and its children. They have different timing and should not be used interchangeably.

### 33. What happens if I assign an @Input property through a ViewChild reference?

A direct TypeScript assignment does not automatically notify Angular to check an OnPush component. Prefer updating the source bound in the template. For programmatically created components, use ComponentRef.setInput(); if an imperative update is unavoidable, mark the view appropriately.

### 34. Does markForCheck() immediately refresh the browser DOM?

No. It marks the view for a future Angular synchronization. Angular updates bindings during that check.

### 35. Does a detached view respond to markForCheck() during normal change detection?

No. A detached view is excluded from normal change-detection runs. Reattach it or perform the deliberate local check required by the rendering design.

## Debugging and performance

### 36. What should I inspect first when an OnPush view looks stale?

Check whether the state changed, whether Angular received a notification, whether object inputs have new references, whether the template reads the relevant signal, and whether an OnPush child subtree is being skipped.

### 37. Can a computed signal become stale after deep mutation?

If its source signal was not notified-for example, because the same reference was set again under default equality-the computed signal is not invalidated by that update. Replace the signal value immutably so dependencies are notified.

### 38. Does OnPush make expensive template methods fast?

No. If Angular checks the component, it still evaluates its bindings. Keep template expressions inexpensive and profile actual bottlenecks.

### 39. Does OnPush enforce immutability at runtime?

No. JavaScript objects are still mutable. OnPush is a checking strategy; immutable updates are a programming discipline that works well with identity-based input checks and signal equality.

### 40. Is OnPush always a guaranteed performance improvement?

No strategy removes the need to profile. OnPush creates opportunities to skip unaffected subtrees, but performance depends on component structure, update frequency, template work, and state flow.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="043-angular-rendering-and-change-detection.md">043 - 43. Angular rendering and change detection</a></td><td><a href="../lessons/044-onpush-change-detection-and-immutable-updates.md">Lesson</a></td><td><a href="../examples/044-onpush-change-detection-and-immutable-updates.md">Example</a></td><td><a href="045-zoneless-angular-applications.md">045 - 45. Zoneless Angular applications</a></td>
</tr>
  </tbody>
</table>
