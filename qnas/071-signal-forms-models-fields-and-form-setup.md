# Q&A: Signal Forms: models, fields, and form setup

Use these questions to check your understanding of Signal Forms' model and field setup. See the [lesson](../lessons/071-signal-forms-models-fields-and-form-setup.md) and the [working example](../examples/071-signal-forms-models-fields-and-form-setup.md).

## Core concepts

### 1. What is a Signal Form?

A Signal Form is an Angular form API built on signals. A writable signal stores the form data, `form()` creates a field tree from that model, and `[formField]` binds controls to fields.

### 2. What is the source of truth for form data?

The writable model signal passed to `form()`. The field tree works with that model; it is not a second data object that needs manual synchronization.

### 3. What does `form()` return?

A typed field tree whose structure follows the model's properties and arrays. Each node represents a field or group of fields and exposes reactive field state when called.

### 4. Does `form()` mean the HTML `<form>` element?

No. `form()` is an API imported from `@angular/forms/signals`. The HTML `<form>` element provides the browser's form semantics and submission boundary.

### 5. What is `FormField`?

It is the directive that binds a Signal Forms field tree node to a supported UI control. It is imported from `@angular/forms/signals`.

### 6. What is `[formField]`?

It is the directive input binding syntax used in a template, for example `[formField]="contactForm.email"`. It connects the control to a field tree node.

## Model design

### 7. Why define a TypeScript interface for the model?

It documents the form's data shape and gives the model and field access useful type checking and editor support.

### 8. Should every intended field have an initial value?

Yes. The field tree is derived from the model shape. If a property is missing or `undefined`, it will not create the field you expect.

### 9. What empty value should a text input use?

Usually `''`. A normal text control does not use `null` as its empty string value.

### 10. What value should a checkbox field use?

A boolean, usually initialized to `false` or `true` based on the initial form state.

### 11. How should an optional value be represented?

Choose an explicit type and empty representation, such as `string | null` when that makes sense for the control and domain. Do not omit the property or leave it `undefined` if you need a field in the tree.

### 12. Can a Signal Forms model contain nested objects?

Yes. A nested plain object creates nested field paths, such as `profileForm.address.city`.

### 13. Can a model contain arrays?

Yes. Arrays are supported and their structure is reflected in the field tree. More advanced dynamic-array behavior can be studied in the dedicated array and collection material.

### 14. Can I use a class instance as the model's structural object?

Do not use class instances as the structural layer. Signal Forms expects plain JavaScript objects and arrays there. Translate domain classes to plain form data when needed.

### 15. Are `Map` and `Set` supported as the structural layer?

No. Use plain objects and arrays for the shape that Signal Forms walks to create fields.

### 16. Should the form model always be identical to the API or domain model?

No. It should represent the data the UI edits. If the domain model has a different shape, convert at the form boundary rather than forcing the form structure to match a class or persistence object.

## Creating and reading the form

### 17. Which imports are used for the basic setup?

Import `signal` from `@angular/core`, and `form` and `FormField` from `@angular/forms/signals`.

### 18. Where should a standalone component import `FormField`?

In the component decorator's `imports` array. Standalone components need template directives and components listed there.

### 19. How do I create the field tree from a component property?

Call `form(this.contactModel)` where `contactModel` is the writable signal holding the form model.

### 20. How do I access a top-level field?

Use property access, such as `contactForm.email`. That expression refers to the field tree node, ready to bind or inspect.

### 21. How do I access a nested field?

Follow the model's object path, such as `profileForm.address.city`.

### 22. How do I read a field's current value?

Call the field node to get state, then call its value signal: `contactForm.email().value()`.

### 23. How do I read the entire current model?

Call the writable signal, for example `contactModel()`. This returns the current form data object.

### 24. How do I make a programmatic model update?

Use the writable signal's `set()` or `update()` method. When changing one property in an object, return a new object that preserves the other properties.

~~~ts
this.contactModel.update((current) => ({
  ...current,
  name: 'Asha Rao',
}));
~~~

### 25. Does updating the model signal update bound controls?

Yes. The field tree and `FormField` bindings react to changes in the model, so the UI stays synchronized.

## Template binding

### 26. How does user input reach the model?

The `FormField` directive observes the bound control, updates its field state, and synchronizes the change to the model signal.

### 27. How does a model update reach the control?

The model signal updates reactively, and the directive synchronizes the corresponding field value to the bound control.

### 28. Do I need an `(input)` handler just to copy text into the model?

No. Bind the field with `[formField]`. Add event handling only for separate behavior that the binding does not provide.

### 29. Does field binding make a label accessible automatically?

No. Use semantic HTML and provide an associated label, instructions, and accessible feedback as needed. Data binding does not replace accessible form design.

### 30. Can a select or checkbox use `[formField]`?

Yes, when the model type and control match. A checkbox normally represents a boolean; a select's option values should correspond to the model's possible values.

## API boundaries and troubleshooting

### 31. Is Signal Forms the same API as Reactive Forms?

No. Signal Forms use model signals, `form()`, and `[formField]`. Reactive Forms use objects such as `FormControl` and `FormGroup` with bindings such as `formControlName`.

### 32. Can I put `formControlName` on a Signal Forms field?

Do not treat the APIs as interchangeable. Use the binding that matches the form model. Follow official interoperability guidance if a legacy integration is required.

### 33. What happens if `FormField` is not imported?

Angular cannot resolve the directive in the component template. Import `FormField` from `@angular/forms/signals` and include it in the component's `imports`.

### 34. Why does TypeScript report that a field does not exist?

The requested field name may not be part of the model's declared type, may be misspelled, or may be absent from the initialized model. Align the type, initial object, and template.

### 35. Why might a model property not appear in the field tree?

Check whether it was initialized. Missing or `undefined` properties are not represented as the intended field.

### 36. Which Angular versions can use Signal Forms?

The current overview lists Angular v21 or higher as a prerequisite. The current API pages mark `form()` and `FormField` stable since v22. Check the project's Angular version and the current docs because API status may change.

### 37. Where do validation rules belong in this sequence?

This chapter establishes the model and fields. Signal Forms schemas and validation rules are a separate topic; add them there rather than mixing setup basics with advanced validation.

## Review checklist

Before moving on, make sure you can:

- Describe the relationship between the model signal, field tree, and template binding.
- Initialize all fields with meaningful values.
- Bind top-level and nested native controls.
- Read the whole model and individual field state.
- Explain why Signal Forms and Reactive Forms directives are different.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="070-custom-controls-with-controlvalueaccessor.md">070 - Custom controls with ControlValueAccessor</a></td><td><a href="../lessons/071-signal-forms-models-fields-and-form-setup.md">Lesson</a></td><td><a href="../examples/071-signal-forms-models-fields-and-form-setup.md">Example</a></td><td><a href="072-signal-forms-schemas-validation-and-custom-controls.md">072 - Signal Forms: schemas, validation, and custom controls</a></td>
</tr>
  </tbody>
</table>
