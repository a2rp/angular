# Signal Forms: schemas, validation, and custom controls

Signal Forms keep validation rules in a schema associated with a form. The schema identifies which rules apply to which fields; Angular evaluates those rules reactively and exposes their results as field state. A custom control can use the same field binding as a native input when it implements the appropriate Signal Forms control interface.

This lesson introduces built-in and custom validators, accessible error feedback, and the contracts for reusable value and checkbox controls.

> **Version note:** Signal Forms are available in Angular v21 and later. The current Angular API reference marks the schema, validation, and custom-control APIs in this lesson stable since Angular v22. Check your project's Angular version before adopting them.

## Learning outcomes

After this lesson, you should be able to:

- Attach built-in validation rules through the schema callback passed to `form()`.
- Choose validators such as `required()`, `email()`, `min()`, `minLength()`, and `pattern()`.
- Return a typed error from a custom `validate()` rule.
- Display errors from a field's reactive state without showing them before the user interacts.
- Build a single-value custom control with `FormValueControl<T>`.
- Build a boolean custom control with `FormCheckboxControl`.
- Keep value synchronization in the control and validation rules in the form schema.

## Schemas connect rules to fields

A schema callback is the second argument to `form()`. Angular calls the callback when constructing the form's logic tree. The callback receives paths to model fields; pass those paths to rules such as `required()` and `email()`:

~~~ts
import {form, required, email, minLength} from '@angular/forms/signals';

registrationForm = form(this.registrationModel, (path) => {
  required(path.email, {message: 'Enter your email address.'});
  email(path.email, {message: 'Enter an email in the expected format.'});
  minLength(path.displayName, 2, {
    message: 'Display name must contain at least 2 characters.',
  });
});
~~~

The schema callback sets up the rules. The rule functions evaluate reactively as relevant values change. You do not call the schema again after every keystroke.

A schema is a good place to describe form behavior, including validation and conditional rules. Keep it separate from presentation: templates decide how and when to show the error messages.

## Useful built-in validators

Signal Forms provides validators for common constraints:

| Validator | Typical purpose |
| --- | --- |
| `required(path)` | Require a non-empty value |
| `email(path)` | Check an email-shaped string |
| `min(path, number)` / `max(path, number)` | Restrict a numeric value |
| `minLength(path, count)` / `maxLength(path, count)` | Restrict a string or collection length |
| `pattern(path, regex)` | Require a value to match a pattern |

Built-in rules accept a configuration object with a user-facing `message`. Prefer a message that states what is wrong and how to fix it.

~~~ts
required(path.password, {message: 'Password is required.'});
minLength(path.password, 12, {
  message: 'Use at least 12 characters.',
});
~~~

### Understand `required()` semantics

`required()` checks whether a value is empty according to Signal Forms. Empty values include `null`, `undefined`, the empty string, `false`, and `NaN`. The number `0` and an empty array are considered non-empty. To require at least one item in an array, use `minLength(path.items, 1)`.

This matters for checkboxes: an unchecked checkbox is `false`, so `required()` can express a required consent checkbox. For a numeric field, use `min()` or a custom rule if zero is not an acceptable value.

### Validation is not browser constraint validation

Signal Forms evaluates its validators in Angular and exposes the result through field state. It does not rely on the browser to run the rules. When using `FormRoot`, Angular disables native constraint validation for that form. Some rules mirror their constraints to HTML attributes where supported, but the schema rule remains the validation source.

Use Signal Forms rules even if you add HTML attributes for semantics or hints. An HTML `required` attribute by itself is not a substitute for `required(path)`.

## Write a custom validator

Use `validate()` when the built-in rules do not express a domain rule. The validator receives a context with the current `value` and helpers such as `valueOf()` for related paths. Return an object with a `kind` and optional user-facing `message` when invalid. Return `null` or `undefined` when valid.

~~~ts
import {validate} from '@angular/forms/signals';

registrationForm = form(this.registrationModel, (path) => {
  validate(path.confirmPassword, ({value, valueOf}) => {
    if (value() !== valueOf(path.password)) {
      return {
        kind: 'passwordMismatch',
        message: 'The passwords do not match.',
      };
    }

    return null;
  });
});
~~~

Keep validators pure: derive their result from the values and context they receive. Do not send requests, modify the model, or update component state from inside a validator. For asynchronous checks, use Signal Forms' async validation APIs, covered separately.

## Read validation state and show feedback

Calling a field tree node returns its state. Common signals include `valid()`, `invalid()`, `touched()`, `dirty()`, and `errors()`:

~~~ts
const emailState = this.registrationForm.email();

emailState.invalid();
emailState.touched();
emailState.errors();
~~~

A typical template shows feedback after the user has interacted with an invalid field:

~~~html
<label for="email">Email</label>
<input
  id="email"
  type="email"
  [formField]="registrationForm.email"
  [attr.aria-invalid]="
    registrationForm.email().touched() && registrationForm.email().invalid()
  "
  [attr.aria-describedby]="
    registrationForm.email().touched() && registrationForm.email().invalid()
      ? 'email-errors'
      : null
  "
/>

@if (registrationForm.email().touched() && registrationForm.email().invalid()) {
  <ul id="email-errors">
    @for (error of registrationForm.email().errors(); track $index) {
      <li>{{ error.message ?? 'Check this value.' }}</li>
    }
  </ul>
}
~~~

Associate the message with its control using `aria-describedby`, and indicate the current invalid state with `aria-invalid`. Keep labels visible and specific. Avoid relying on color alone to communicate an error.

A field can have more than one failed rule at once. Signal Forms runs applicable rules independently, so `errors()` can contain multiple errors. Decide whether to display all messages or a concise subset, and make sure each message tells the user what to change.

## Custom controls integrate through a contract

Native inputs are usually the simplest choice. Build a custom control when the UI needs behavior that a native control cannot provide, such as a specialized picker or a reusable design-system input.

The `FormField` directive recognizes control interfaces and wires the form field state to the component's signals.

### Single-value controls

A component that edits one value implements `FormValueControl<T>` and provides a `value` model signal:

~~~ts
import {Component, model} from '@angular/core';
import {FormValueControl} from '@angular/forms/signals';

@Component({
  selector: 'app-basic-input',
  template: `
    <input
      [value]="value()"
      (input)="onInput($event)"
    />
  `,
})
export class BasicInput implements FormValueControl<string> {
  value = model('');

  onInput(event: Event): void {
    const target = event.target;
    if (target instanceof HTMLInputElement) {
      this.value.set(target.value);
    }
  }
}
~~~

When the custom control is bound with `[formField]`, the form field's value is synchronized with this `value` model. A `FormValueControl` must not also expose a `checked` property.

### Boolean checkbox controls

A toggle, switch, or other boolean control implements `FormCheckboxControl` and provides a `checked` model signal:

~~~ts
import {Component, model} from '@angular/core';
import {FormCheckboxControl} from '@angular/forms/signals';

@Component({
  selector: 'app-basic-toggle',
  template: `
    <button
      type="button"
      role="switch"
      [attr.aria-checked]="checked()"
      (click)="checked.update((value) => !value)"
    >
      {{ checked() ? 'On' : 'Off' }}
    </button>
  `,
})
export class BasicToggle implements FormCheckboxControl {
  checked = model(false);
}
~~~

A `FormCheckboxControl` must not also expose a `value` property. Use the interface that matches the control's meaning rather than trying to make one component represent both a scalar input and a checkbox.

### Pass field state into a custom control

Both interfaces support optional state properties such as `disabled`, `readonly`, `invalid`, and `errors`. Implement the input signals your control needs to render that state. To report blur/touched behavior, expose a `touch` output and emit it when focus leaves the control.

~~~ts
import {input, model, output} from '@angular/core';
import {
  FormValueControl,
  ValidationError,
  WithOptionalFieldTree,
} from '@angular/forms/signals';

export class StatefulTextInput implements FormValueControl<string> {
  value = model('');
  disabled = input(false);
  invalid = input(false);
  errors = input<readonly WithOptionalFieldTree<ValidationError>[]>([]);
  touch = output<void>();
}
~~~

The template should use those signals to disable the actual interactive element, expose its invalid state accessibly, and display the errors. A composite control with several internal focusable elements should report touch when focus leaves the whole widget, rather than whenever focus moves between its children.

Validation belongs in the form schema. A custom control displays the state it receives; it should not maintain a second, conflicting validation implementation.

## Common mistakes

| Mistake | Better approach |
| --- | --- |
| Adding HTML `required` but no schema rule | Add `required(path)` so Angular field state reflects the rule |
| Assuming `required()` rejects `0` or an empty array | Use `min()` or `minLength()` for those constraints |
| Returning a string from `validate()` | Return an error object with a stable `kind` and a clear `message` |
| Showing errors before the user reaches a field | Gate feedback with `touched()` and `invalid()` |
| Styling an error only by color | Add visible text and programmatic associations |
| Validating separately inside a custom control | Define the rule in the schema and render the provided `invalid`/`errors` state |
| Using `FormValueControl` for a boolean toggle | Use `FormCheckboxControl` with a `checked` model |
| Implementing a custom control without disabled behavior | Pass `disabled` state to the actual interactive element |
| Omitting the custom control's touch output | Emit `touch` when the control is blurred so interaction state can update |

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="071-signal-forms-models-fields-and-form-setup.md">071 - Signal Forms: models, fields, and form setup</a></td><td><a href="../qnas/072-signal-forms-schemas-validation-and-custom-controls.md">Q&amp;A</a></td><td><a href="../examples/072-signal-forms-schemas-validation-and-custom-controls.md">Example</a></td><td><a href="073-configure-the-angular-router-in-a-standalone-app.md">073 - Configure the Angular Router in a standalone app</a></td>
</tr>
  </tbody>
</table>
