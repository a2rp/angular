# 26. Q&A: Component outputs and custom events

## 1. What is a component output?

An output is a public event a component or directive can emit so a consumer can react. It usually communicates from a child component to its parent. For example, a save button can emit `saved` after a save action.

## 2. How are outputs different from inputs?

An input carries data from parent to child. An output carries an event from child to consumer. A child can use an input to receive a selected record and an output to report that the user selected a different record.

## 3. How do I declare an output in modern Angular?

Import `output` from `@angular/core` and call it in a class property initializer:

~~~ts
readonly saved = output<SaveResult>();
~~~

The generic parameter describes the value the output emits.

## 4. What does `output()` return?

It returns an `OutputEmitterRef<T>`. Call `.emit(value)` to send an event. The output reference is not a signal and is not read by calling it.

## 5. Why should I declare the property as `readonly`?

The output reference should stay the same for the lifetime of the component. `readonly` prevents accidental reassignment while still allowing calls such as `this.saved.emit(result)`.

## 6. Where can I call `output()`?

Declare it directly as a property initializer in a component or directive class. Angular recognizes `output()` as a compiler API in that location. Do not call it from a method, constructor, or conditional branch.

## 7. How does a child emit an output?

Call `.emit()` at the point the event occurs:

~~~ts
this.saved.emit({ id: 'a1', title: 'Draft' });
~~~

The payload must match the output's generic type.

## 8. How do I declare an output with no useful payload?

Use `output<void>()` and call `.emit()` without an argument:

~~~ts
readonly dismissed = output<void>();

close(): void {
  this.dismissed.emit();
}
~~~

## 9. What is the purpose of the generic type?

The generic type documents the event data and lets TypeScript check emitted values and consumers. Use `output<number>()` for a number, a named interface for a structured event, or `output<void>()` when there is no payload.

## 10. How does the parent listen to an output?

Use parentheses around its public output name on the child element:

~~~html
<app-editor (saved)="handleSave($event)" />
~~~

Angular invokes the listener when the child emits.

## 11. What is `$event` in an output listener?

It is the value passed to `.emit()`. If the output is declared as `output<SaveResult>()`, the listener receives a `SaveResult` as `$event`.

## 12. Does declaring an output automatically update parent state?

No. An output only notifies listeners. The parent must bind a handler and decide whether to update state, call a service, navigate, or take another action.

## 13. Can I use a method instead of writing a long template expression?

Yes. For meaningful behavior, bind to a parent method such as `(saved)="handleSave($event)"`. A method keeps validation and state changes out of a complicated template expression.

## 14. Do component outputs bubble through the DOM?

No. Angular outputs do not bubble. A parent can listen on the child component instance that declares the output. To notify a higher ancestor, an intermediate component must listen and emit its own output, or the application can use an appropriate shared state mechanism.

## 15. How can a grandparent respond to a nested child's output?

The intermediate component explicitly forwards the event: bind to the nested child's output and emit a corresponding output from its own component. This makes the component boundary and event contract visible at every level.

## 16. Are Angular outputs the same as native DOM events?

No. They share template binding syntax, but outputs are Angular component/directive events. Native DOM events come from browser elements. Outputs do not bubble like many DOM events do.

## 17. Can I name a component output `click` or `change`?

Avoid it. These names overlap with familiar native DOM events and can make a template binding ambiguous to readers. Prefer a semantic component name such as `actionRequested`, `valueChanged`, or `selectionChanged`.

## 18. What naming style should I use?

Use camelCase, describe the event that happened, and do not prefix the name with `on`. Examples include `saved`, `closed`, and `selectionChanged`. The name is case-sensitive in a template.

## 19. Should the output name describe an event or a handler?

Describe the event, not the consumer's implementation. `saved` communicates the component contract; `handleSave` sounds like a particular handler method.

## 20. What should an output payload contain?

Send the smallest stable set of data the consumer needs to respond. A typed object is useful when multiple related values belong together. Avoid exposing private component state or sending the entire component instance.

## 21. How do I give an output a template alias?

Pass an `alias` option:

~~~ts
readonly saved = output<SaveResult>({ alias: 'recordSaved' });
~~~

The TypeScript property is still `saved`; consumers bind to `(recordSaved)`.

## 22. When is an alias useful?

An alias can preserve an existing public template name while renaming the TypeScript property, or avoid a collision with a native event name. Avoid aliases without a concrete reason because the code and template then use different names.

## 23. Can a class inherit outputs from a base class?

Yes. A component inherits outputs declared by its base class. Use this intentionally and document shared behavior so consumers can discover the events exposed by the concrete component.

## 24. Can I subscribe to an output without a template?

Yes. For a dynamically created component, subscribe through its instance:

~~~ts
const subscription = componentRef.instance.saved.subscribe((value) => {
  this.handleSave(value);
});
~~~

This is useful when the component instance is created programmatically.

## 25. Do I need to unsubscribe from every output subscription?

Angular automatically cleans up a subscription when the component that owns the output is destroyed. Call `unsubscribe()` if the listener should stop before that component is destroyed.

## 26. Is `OutputEmitterRef` an RxJS `Observable` or `Subject`?

No. It is the Angular output reference API. It provides `.emit()` and `.subscribe()` for output events, but it does not expose the RxJS `Subject` API or error/completion channels. Do not call `.next()` or `.complete()` on it.

## 27. Is `@Output() saved = new EventEmitter<T>()` still valid?

Yes. The decorator-based API remains supported in existing applications. For new code, Angular recommends the `output()` function. Avoid mixing styles in one component without a reason.

## 28. Can an output be declared with `output()` in a directive?

Yes. Components and directives can declare outputs. Consumers bind to a directive's output on an element where that directive is applied.

## 29. Can a component have multiple outputs?

Yes. Give each event its own typed output property. Keep the API focused; expose events consumers need rather than forwarding every internal detail.

## 30. Should I emit every time an internal field changes?

Only if that is the documented event contract. Some components report every value change; others report a completed action such as `submitted` or `saved`. Name and emit the output to match the intended timing.

## 31. Should I use an output to send a new value to a child?

No. Use an input for values provided by the parent. If a child reports a user change, an output can communicate that event, and the parent can pass the updated value back through an input if needed.

## 32. Does an output have to be triggered by a native event?

No. A component can emit an output from any method or application event where its documented condition occurs. A button click is common, but outputs can also report completion, selection, validation, or other component-level events.

## 33. What migration is available for decorator-based outputs?

Angular provides a schematic:

~~~sh
ng generate @angular/core:output-migration
~~~

Review the migration changes, especially when outputs are part of a library's public API.

## 34. What is a common reason a parent listener does not run?

Check that the binding name exactly matches the output's public name, including letter case and any alias. Also check that the child actually calls `.emit()` and that the listener is attached to the component/directive that declares the output.

## 35. What is the simplest way to decide whether a value belongs in an input or output?

Ask who owns the value and who initiates the communication. If the parent supplies data the child reads, use an input. If the child reports that something happened and the parent should decide what to do, use an output.

## 36. Where can I read the official output API details?

See Angular's component outputs guide, [`output()` API reference](`https://angular.dev/api/core/output`), and output migration guide.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="025-signal-inputs-required-inputs-and-input-transforms.md">025 - 25. Signal inputs, required inputs, and input transforms</a></td><td><a href="../lessons/026-component-outputs-and-custom-events.md">Lesson</a></td><td><a href="../examples/026-component-outputs-and-custom-events.md">Example</a></td><td><a href="027-model-inputs-and-component-level-two-way-binding.md">027 - 27. Model inputs and component-level two-way binding</a></td>
</tr>
  </tbody>
</table>
