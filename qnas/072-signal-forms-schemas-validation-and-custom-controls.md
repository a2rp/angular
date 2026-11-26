# Q&A: Signal Forms: schemas, validation, and custom controls

Review how Signal Forms attach validation rules to fields, expose errors, and connect reusable custom controls. See the [lesson](../lessons/072-signal-forms-schemas-validation-and-custom-controls.md) and the [example](../examples/072-signal-forms-schemas-validation-and-custom-controls.md).

## Schemas and built-in rules

### 1. What does a Signal Forms schema describe?

It describes form behavior associated with model paths, including which validation and other rules apply to fields.

### 2. Where is an inline schema declared?

As the second argument to `form(model, schemaFn)`. The callback receives a schema path tree that mirrors the model.

### 3. Does Angular call the schema callback on every keystroke?

No. The callback sets up the form's logic tree when the form is created. The rules registered in it react to the changing values they depend on.

### 4. Name some built-in Signal Forms validators.

Common rules include `required()`, `email()`, `min()`, `max()`, `minLength()`, `maxLength()`, and `pattern()`.

### 5. How do you provide a custom message for a built-in rule?

Pass an options object with a `message` property:

~~~ts
required(path.email, {message: 'Email is required.'});
~~~

### 6. Which values does `required()` consider empty?

Signal Forms treats `null`, `undefined`, `''`, `false`, and `NaN` as empty. A numeric `0` and an empty array are non-empty.

### 7. How do you require at least one item in an array?

Use `minLength(path.items, 1)`. `required()` alone considers an empty array non-empty.

### 8. Why can `required()` be useful for a consent checkbox?

An unchecked checkbox has the value `false`, which `required()` treats as empty. It can therefore require the user to check a consent box.

### 9. Is `required()` enough to reject a numeric zero?

No. Zero is non-empty under this rule. Use a suitable `min()` rule or a custom validator if zero is outside the allowed range.

### 10. Do validators stop after the first failed rule?

No. Applicable rules run independently, so one field can expose multiple errors. Display messages in an order and amount that help the user correct the input.

## Custom validation and field errors

### 11. When should you write a custom validator?

When the built-in rules do not capture the business rule, such as requiring a secure URL prefix or matching a confirmation field.

### 12. Which function creates a custom validator?

`validate(path, validatorFn)` from `@angular/forms/signals`.

### 13. What does a custom validator return when the value is invalid?

An error object with a stable `kind` and, preferably, a clear `message`:

~~~ts
return {
  kind: 'passwordMismatch',
  message: 'The passwords do not match.',
};
~~~

### 14. What should a custom validator return when the value is valid?

`null` or `undefined`.

### 15. What does the validator context provide?

It includes the current field's value signal and helpers such as `valueOf(path)` to read related field values. It also exposes field and state references.

### 16. How can a confirmation field compare itself with a password field?

Use `value()` for the current confirmation and `valueOf(path.password)` for the password, then return an error if they differ.

### 17. Should a validator update the model or send an HTTP request?

No. Keep validators pure: they should calculate a result from the supplied context. Use separate form or async-validation mechanisms for effects and remote checks.

### 18. How do you read whether a field is invalid?

Call the field tree node and then its state signal, for example `registrationForm.email().invalid()`.

### 19. What information is in `errors()`?

It is a reactive list of the field's current validation errors. Error objects include a `kind` and may have a user-facing `message`.

### 20. Why gate visible errors with `touched()`?

It avoids showing an error before the user has interacted with the field, while still showing feedback after they leave or edit it.

### 21. What does `dirty()` mean?

It indicates that the value has been modified, even if the user later changes it back. `touched()` represents interaction state and has a different meaning.

### 22. Which accessibility attributes can connect an invalid control and its message?

Set `aria-invalid` when the field is currently invalid and `aria-describedby` to reference the visible error text. Keep a real label associated with the control.

### 23. Does Signal Forms rely on the browser's native constraint validation to run its rules?

No. Signal Forms evaluates its own rules and exposes state through the field tree. Some rules mirror constraints to HTML attributes where supported, but the Angular schema remains the source of validation results.

### 24. Is an HTML `required` attribute a replacement for the schema's `required()` rule?

No. Use the Signal Forms validator so the form's reactive validation state knows about the rule. HTML attributes can still provide semantics or hints.

## Custom control contracts

### 25. When should you create a custom form control?

When native controls cannot provide the interaction or presentation the UI needs. Prefer a native input for standard cases.

### 26. Which interface is for a control that edits one value?

`FormValueControl<T>`. A control implementing it provides a writable `value` model signal.

### 27. Which interface is for a boolean switch or checkbox-like control?

`FormCheckboxControl`. It provides a writable `checked` model signal.

### 28. Should a `FormValueControl` also expose `checked`?

No. It represents a single value through `value`. Use the boolean checkbox interface for a control whose meaning is checked/unchecked.

### 29. Should a `FormCheckboxControl` also expose `value`?

No. It represents its boolean state through `checked`.

### 30. How does `[formField]` connect a custom control?

The directive recognizes the implemented interface and synchronizes the form field with the control's `value` or `checked` model signal. It can also pass supported optional state signals.

### 31. Which optional state can a custom control receive?

Depending on what it implements, it can receive states such as `disabled`, `readonly`, `invalid`, `errors`, `touched`, and `dirty`, along with validation constraints and field metadata.

### 32. Is validation logic supposed to live inside the custom control?

No. Keep validation rules in the form schema. The control should render its value and any validation state passed to it.

### 33. What must a custom control do with disabled state?

Bind it to the actual interactive element. A visual disabled style alone does not prevent user interaction.

### 34. What does a custom control's `touch` output communicate?

It reports that interaction with the custom control has ended, commonly when its internal input blurs. This lets the form update touched state and supports blur-based update behavior.

### 35. How should a composite control handle focus moving among its children?

Treat the composite widget as one control. Report touch when focus leaves the whole widget, not on every move between its internal buttons or inputs.

### 36. Can a custom Signal Form control display its validation errors?

Yes. Implement optional state inputs such as `invalid` and `errors`, and render the supplied messages accessibly. The control displays the schema's results; it does not create a second validation system.

## Review checklist

Before moving on, make sure you can:

- Add built-in rules to the schema paths for the intended fields.
- Explain the less obvious `required()` behavior for `false`, `0`, and empty arrays.
- Create a pure `validate()` rule that returns a structured error.
- Render errors using field state and accessible relationships.
- Choose between `FormValueControl` and `FormCheckboxControl`.
- Explain how a custom control receives value and state through `FormField`.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="071-signal-forms-models-fields-and-form-setup.md">071 - Signal Forms: models, fields, and form setup</a></td><td><a href="../lessons/072-signal-forms-schemas-validation-and-custom-controls.md">Lesson</a></td><td><a href="../examples/072-signal-forms-schemas-validation-and-custom-controls.md">Example</a></td><td><a href="073-configure-the-angular-router-in-a-standalone-app.md">073 - Configure the Angular Router in a standalone app</a></td>
</tr>
  </tbody>
</table>
