# Synchronous validators and validation state

A validator checks a control's current value and immediately returns either **null** when it passes or a **ValidationErrors** object when it fails. Angular runs synchronous validators as the form value changes, then updates the control's validity and error state.

This lesson focuses on validators that finish immediately. Cross-field rules and asynchronous checks are covered in the next lesson.

## Learning outcomes

After this lesson, you should be able to:

- Explain the **ValidatorFn** contract and the meaning of **null** and an error map.
- Apply Angular's built-in validators to typed reactive form controls.
- Write a reusable synchronous validator without mutating form state.
- Read a control's **errors**, **hasError()**, **valid**, and **invalid** state.
- Distinguish validation status from interaction state such as **dirty** and **touched**.
- Show useful, accessible validation feedback at the right time.
- Choose when validation runs with **updateOn**.
- Recognize why optional empty values and **minLength** need deliberate handling.

## The ValidatorFn contract

A synchronous validator receives an **AbstractControl** and immediately returns **ValidationErrors | null**:

~~~ts
import {
  AbstractControl,
  ValidationErrors,
  ValidatorFn,
} from '@angular/forms';

const exampleValidator: ValidatorFn = (
  control: AbstractControl,
): ValidationErrors | null => {
  const value = String(control.value ?? '');

  return value.length <= 20
    ? null
    : { maxTextLength: { allowed: 20, actual: value.length } };
};
~~~

The validator should describe a rule and report its result:

- Return **null** when the value passes.
- Return an object whose keys identify failed rules when it does not.
- Keep the validator synchronous and predictable: do not make HTTP requests or modify the control inside the validator.
- Put useful details in the error value when the interface or application needs them, such as the required and actual lengths.

Angular runs validators and combines their error maps. A control with one or more errors has **INVALID** status. When no validator reports an error, the control can be **VALID**, unless it is pending or disabled.

## Use built-in validators

Angular provides common synchronous validators through **Validators**:

| Validator | What it checks | Common error key |
| --- | --- | --- |
| **required** | A non-empty value | **required** |
| **requiredTrue** | The value is **true**, often for consent | **required** |
| **email** | A common email format | **email** |
| **min(value)** | A numeric value at least the given minimum | **min** |
| **max(value)** | A numeric value no greater than the given maximum | **max** |
| **minLength(length)** | A string or collection with enough items | **minlength** |
| **maxLength(length)** | A string or collection no longer than the limit | **maxlength** |
| **pattern(pattern)** | A value matching a regular expression | **pattern** |

Add a single validator as the second argument to a **FormControl**. Pass an array when a control has multiple synchronous validators:

~~~ts
import { FormControl, Validators } from '@angular/forms';

const displayName = new FormControl('', {
  nonNullable: true,
  validators: [
    Validators.required,
    Validators.minLength(2),
    Validators.maxLength(40),
  ],
});
~~~

The control can have more than one error at once. For example, a short non-empty value can fail **minLength** while another custom rule also fails. Render the messages your product needs instead of assuming there is only one error.

### Required values and optional values

**Validators.required** checks that a value is present. Use it when the field must not be empty.

**Validators.requiredTrue** is useful when the value must be exactly **true**, such as a consent checkbox:

~~~ts
const acceptedTerms = new FormControl(false, {
  nonNullable: true,
  validators: Validators.requiredTrue,
});
~~~

**Validators.minLength()** intentionally does not report an error for an empty string or empty array. That lets optional fields remain empty. If the field is mandatory, combine **required** with **minLength**:

~~~ts
const username = new FormControl('', {
  nonNullable: true,
  validators: [Validators.required, Validators.minLength(3)],
});
~~~

For an optional field, use **minLength** alone so a blank value can remain valid while a non-empty value must meet the minimum length. If a collection must contain at least one item, use a collection-level rule designed for that requirement; **minLength(1)** alone permits an empty array.

### Numeric and pattern validation

Use **min()** and **max()** on a numeric control. A number input can be cleared, so a nullable type is appropriate if the form should represent the empty value:

~~~ts
const age = new FormControl<number | null>(null, [
  Validators.required,
  Validators.min(18),
  Validators.max(120),
]);
~~~

The **pattern()** validator can receive a string pattern or a regular expression. Avoid a **RegExp** with global (**g**) or sticky (**y**) flags: repeated calls to **test()** can depend on the expression's retained match index and produce inconsistent results.

Pattern validation is not a substitute for server-side validation. A client-side pattern should communicate the input format, while the server remains responsible for checking submitted data.

## Write a reusable synchronous validator

A custom validator is useful for a rule that Angular's built-in set does not express. The following factory rejects a prohibited term while allowing an empty value; a separate **required** validator can make the field mandatory:

~~~ts
import {
  AbstractControl,
  ValidationErrors,
  ValidatorFn,
} from '@angular/forms';

export function forbiddenTermValidator(term: string): ValidatorFn {
  const normalizedTerm = term.trim().toLocaleLowerCase();

  return (control: AbstractControl): ValidationErrors | null => {
    const value = String(control.value ?? '').trim();

    if (!value || !normalizedTerm) {
      return null;
    }

    const containsForbiddenTerm = value
      .toLocaleLowerCase()
      .includes(normalizedTerm);

    return containsForbiddenTerm
      ? { forbiddenTerm: { term } }
      : null;
  };
}
~~~

Use the factory when constructing a control:

~~~ts
const accountName = new FormControl('', {
  nonNullable: true,
  validators: [
    Validators.required,
    Validators.minLength(3),
    forbiddenTermValidator('admin'),
  ],
});
~~~

The custom validator returns an error under the **forbiddenTerm** key. The form model owns the validator; the template reads its state and chooses the message.

A validator should not call **setErrors()** on its own control. Angular uses the validator's returned value to calculate the errors. Directly mutating form state during validation can overwrite other errors or cause confusing recalculation behavior.

## Read errors and validation status

An **AbstractControl** exposes several related properties:

- **errors** contains the current error map, or **null** when there are no errors.
- **hasError(errorCode)** checks for a particular error key.
- **getError(errorCode)** returns the value stored for that error key.
- **valid** is true when the status is **VALID**.
- **invalid** is true when the status is **INVALID**.
- **pending** is true while asynchronous validation is in progress.
- **disabled** is true when validation is disabled for that control.
- **status** is one of **VALID**, **INVALID**, **PENDING**, or **DISABLED**.

A form's status aggregates its enabled descendants. Do not treat **status** as the same thing as interaction state. **dirty**, **pristine**, **touched**, and **untouched** describe user interaction, not whether a value satisfies its rules.

A control is normally marked **dirty** when its value changes through the UI. A control is marked **touched** when the user leaves it after focus. Programmatic updates do not necessarily represent user interaction, so explicitly mark a form dirty or touched when the application needs that behavior.

## Show errors after interaction

Displaying every error as soon as an empty form loads can overwhelm users. A common pattern is to show a field's messages after it becomes dirty or touched:

~~~html
@if (
  accountName.invalid &&
  (accountName.dirty || accountName.touched)
) {
  <div class="field-errors">
    @if (accountName.hasError('required')) {
      <p>Enter an account name.</p>
    }

    @if (accountName.hasError('minlength')) {
      <p>Use at least three characters.</p>
    }

    @if (accountName.hasError('forbiddenTerm')) {
      <p>Choose a name that does not contain a reserved term.</p>
    }
  </div>
}
~~~

The separate **@if** blocks allow more than one message to appear when more than one validator fails. Use **@else if** only when the interface intentionally shows a single prioritized message.

For submission, mark invalid fields touched before returning. This reveals errors even if the user focused and submitted without leaving a field:

~~~ts
save(): void {
  if (this.form.invalid || this.form.pending) {
    this.form.markAllAsTouched();
    return;
  }

  // Send the valid value to the application service.
}
~~~

The server should validate submitted data as well. A user can bypass or alter client-side code, so Angular validation is for a clear interaction and early feedback, not a security boundary.

## Keep validation feedback accessible

Associate hints and errors with the input using **aria-describedby**, and expose the current invalid state with **aria-invalid** after the error is relevant:

~~~html
<label for="account-name">Account name</label>
<input
  id="account-name"
  formControlName="accountName"
  required
  aria-describedby="account-name-hint account-name-error"
  [attr.aria-invalid]="
    accountName.invalid &&
    (accountName.dirty || accountName.touched)
      ? 'true'
      : null
  "
/>
<p id="account-name-hint">
  Use at least three characters.
</p>

@if (
  accountName.invalid &&
  (accountName.dirty || accountName.touched)
) {
  <p id="account-name-error">
    @if (accountName.hasError('required')) {
      Enter an account name.
    } @else if (accountName.hasError('minlength')) {
      Use at least three characters.
    }
  </p>
}
~~~

Keep the **required** attribute on a required input for clear native semantics and accessibility even when the reactive form model is the source of Angular validation. Ensure each id is unique, especially inside repeated form rows.

Do not rely on color alone to communicate invalid state. Include readable text and a clear association with the field.

## Choose when validation runs

Reactive controls validate on value changes by default. Set **updateOn** to **'blur'** or **'submit'** when validation should wait for a later interaction:

~~~ts
const email = new FormControl('', {
  nonNullable: true,
  validators: [Validators.required, Validators.email],
  updateOn: 'blur',
});
~~~

Use **'change'** when immediate feedback is appropriate. **'blur'** can avoid showing a format error after every keystroke. **'submit'** delays updates until form submission. Choose a timing that helps users correct errors without hiding important feedback.

**updateOn** controls when the control updates its value and validation state from the view. Programmatic updates such as **setValue()** still update the model and recalculate validity.

## Understand Angular's validation classes

Angular adds state classes to reactive form elements, including:

- **ng-valid** and **ng-invalid**
- **ng-pending**
- **ng-pristine** and **ng-dirty**
- **ng-untouched** and **ng-touched**

These classes can help style controls, but tie error styling to a useful interaction rule. For example, style an invalid field after it has been touched:

~~~css
input.ng-invalid.ng-touched,
select.ng-invalid.ng-touched,
textarea.ng-invalid.ng-touched {
  border-color: #b42318;
}
~~~

Use the form model for business logic and error messages. CSS classes are a presentation aid, not a replacement for checking **hasError()** or the control's status in the component and template.

## Common mistakes

- **Returning false from a validator:** return **null** for success or an error object for failure.
- **Returning an empty object for success:** an error object means the control is invalid; return **null** when valid.
- **Mutating the control inside the validator:** return an error result instead of calling **setErrors()**.
- **Assuming one error at a time:** several synchronous validators can fail together.
- **Using minLength as required:** empty strings and arrays are intentionally skipped by **minLength**. Add **required** for a required string, or a collection-specific rule for a required list.
- **Using required for a checkbox that must be checked:** use **requiredTrue** when the value must be exactly true.
- **Putting a validator in the wrong argument:** synchronous validators are the second control argument; asynchronous validators use the third argument or **asyncValidators** option.
- **Showing all errors before the user interacts:** use **dirty**, **touched**, or a submission-attempt flag to choose when messages appear.
- **Checking only invalid before submit:** a form can be **PENDING** when async validation is part of it. Handle pending state in the submission flow.
- **Treating client validation as security:** validate data again on the server.
- **Using stateful global or sticky regexes with pattern():** repeated checks may produce inconsistent results.
- **Forgetting accessible error text:** associate readable messages with the input and expose its invalid state.

## Next steps

- Continue with [Cross-field and asynchronous validation](067-cross-field-and-asynchronous-validation.md).
- Review the [Q&A](../qnas/066-synchronous-validators-and-validation-state.md).
- Work through the [complete example](../examples/066-synchronous-validators-and-validation-state.md).
- Revisit [FormArray and dynamic form structures](065-formarray-and-dynamic-form-structures.md).

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="065-formarray-and-dynamic-form-structures.md">065 - FormArray and dynamic form structures</a></td><td><a href="../qnas/066-synchronous-validators-and-validation-state.md">Q&amp;A</a></td><td><a href="../examples/066-synchronous-validators-and-validation-state.md">Example</a></td><td><a href="067-cross-field-and-asynchronous-validation.md">067 - Cross-field and asynchronous validation</a></td>
</tr>
  </tbody>
</table>
