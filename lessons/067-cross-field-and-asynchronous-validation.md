# Cross-field and asynchronous validation

Some rules need more than one field, and some checks need information that is not available in the browser. A **cross-field validator** compares controls in their common parent, usually a **FormGroup**. An **async validator** returns a **Promise** or **Observable** that eventually reports **null** or a validation error.

Use cross-field validation for rules such as matching password fields or start/end dates. Use async validation for external checks such as username availability. Keep server-side validation authoritative, because client state can change and requests can race.

## Learning outcomes

After this lesson, you should be able to:

- Attach a cross-field validator to the common parent of related controls.
- Display an error on the correct parent without manually changing child errors.
- Implement an **AsyncValidatorFn** that returns a finite result.
- Understand when async validators run and how the **PENDING** status appears.
- Choose **updateOn** timing for checks that may call a service.
- Handle service errors deliberately and avoid treating an unavailable check as success by accident.
- Keep the server responsible for enforcing uniqueness and business rules.

## Put cross-field validation on the common parent

A validator that needs two sibling values belongs on the nearest control that contains both, usually their **FormGroup**. The validator reads the values and returns a group-level error:

~~~ts
import {
  AbstractControl,
  ValidationErrors,
  ValidatorFn,
} from '@angular/forms';

export const passwordsMatchValidator: ValidatorFn = (
  control: AbstractControl,
): ValidationErrors | null => {
  const password = control.get('password');
  const confirmation = control.get('confirmPassword');

  if (!password || !confirmation) {
    return null;
  }

  // The child required validators handle blank values.
  if (!password.value || !confirmation.value) {
    return null;
  }

  return password.value === confirmation.value
    ? null
    : { passwordMismatch: true };
};
~~~

Attach it when building the group:

~~~ts
readonly credentialsForm = this.fb.group(
  {
    password: this.fb.control('', [
      Validators.required,
      Validators.minLength(12),
    ]),
    confirmPassword: this.fb.control('', Validators.required),
  },
  {
    validators: passwordsMatchValidator,
  },
);
~~~

When the values differ, **credentialsForm.hasError('passwordMismatch')** is true. The error belongs to the group because the rule evaluates a relationship between two fields. The individual controls keep their own errors, such as **required** or **minlength**.

Do not call **setErrors()** on one child from inside the group validator. Angular recalculates the parent's returned errors as values change; manually mutating a child can overwrite its other validation results or leave confusing state.

### Show the group-level message

Display a group error near the related fields, after the user has interacted with the form:

~~~html
@if (
  credentialsForm.hasError('passwordMismatch') &&
  (credentialsForm.dirty || credentialsForm.touched || attemptedSubmit)
) {
  <p class="error" role="alert">
    The passwords do not match.
  </p>
}
~~~

This keeps the message attached to the rule's owner. A mismatch is not an error that belongs only to the password or only to the confirmation; either value could be the one that needs correction.

## Understand asynchronous validators

An **AsyncValidatorFn** receives a control and returns either:

- a **Promise<ValidationErrors | null>**, or
- an **Observable<ValidationErrors | null>**.

Angular runs async validators only after synchronous validation succeeds. While an async check is unresolved, the control's status becomes **PENDING**. When the result arrives, Angular updates the error map and the status.

For an observable, return a stream that emits a result and completes. A one-response HTTP request normally completes on its own. Use operators such as **take(1)** when the source could otherwise emit more than one result.

## Write an async validator around a service

A validator can adapt a service's result into Angular's error-map contract. This example assumes the service returns an observable boolean:

~~~ts
import {
  AbstractControl,
  AsyncValidatorFn,
  ValidationErrors,
} from '@angular/forms';
import { Observable, catchError, map, of, take } from 'rxjs';

export interface UsernameAvailabilityService {
  isAvailable(username: string): Observable<boolean>;
}

export function usernameAvailabilityValidator(
  service: UsernameAvailabilityService,
): AsyncValidatorFn {
  return (control: AbstractControl) => {
    const username = String(control.value ?? '').trim();

    // Validators.required handles an empty value before async validation runs.
    if (!username) {
      return of(null);
    }

    return service.isAvailable(username).pipe(
      take(1),
      map((available): ValidationErrors | null =>
        available ? null : { usernameTaken: true },
      ),
      catchError(() => of({ usernameCheckFailed: true })),
    );
  };
}
~~~


The validator translates **true** into **null** and **false** into a named error. Its error handler here reports that availability could not be verified instead of silently allowing the username. A product may choose a different recovery flow, but the choice should be explicit and the user should be able to retry.

This validator does not reserve the name. Two users can check the same name before either one submits. The server must enforce uniqueness when the registration is created and return a useful conflict response if the name has since been taken.

### Attach the async validator

When creating a control with options, place synchronous checks in **validators** and async checks in **asyncValidators**:

~~~ts
readonly registrationForm = this.fb.group({
  username: this.fb.control('', {
    validators: [
      Validators.required,
      Validators.minLength(3),
      Validators.maxLength(24),
    ],
    asyncValidators: [
      usernameAvailabilityValidator(this.usernameService),
    ],
    updateOn: 'blur',
  }),
});
~~~

The form builder's non-nullable control remains a **FormControl<string>**. Angular checks the required and length rules first. Only a value that passes those synchronous checks triggers the availability request.

Choose **updateOn: 'blur'** when a check should run after the user leaves the field rather than after each keystroke. For a submission-oriented flow, **'submit'** may be appropriate. The default is **'change'**; use it when immediate validation is helpful and the async work is inexpensive or appropriately rate-limited.

## Show pending and async errors

A pending indicator tells the user that the check is still in progress:

~~~html
@if (username.pending) {
  <p class="status" role="status">Checking username availability…</p>
}

@if (
  username.hasError('usernameTaken') &&
  (username.dirty || username.touched || attemptedSubmit)
) {
  <p class="error" role="alert">
    That username is already in use. Choose another one.
  </p>
}

@if (
  username.hasError('usernameCheckFailed') &&
  (username.dirty || username.touched || attemptedSubmit)
) {
  <p class="error" role="alert">
    Username availability could not be checked. Try again.
  </p>
}
~~~

A form with a pending child can also be **PENDING**. Do not submit while the form is invalid or pending:

~~~ts
save(): void {
  this.attemptedSubmit = true;

  if (this.form.invalid || this.form.pending) {
    this.form.markAllAsTouched();
    return;
  }

  // Continue only when validation has settled and the form is valid.
}
~~~

If the submit button is disabled while pending, keep a visible status message so the user understands why the action is unavailable. If the button remains enabled, the submit handler must still guard against **PENDING**.

## Use group and async validators together

A form can have validators at several levels:

- A **FormControl** can validate its own format or request availability.
- A **FormGroup** can compare related child values.
- A **FormArray** can validate a collection-wide rule.

Each control owns the errors for the rule it evaluates. The parent form combines descendant validity, so a group-level mismatch or a pending async child prevents the root form from being valid.

For larger forms, keep validator functions small and extract service-backed async logic from the component. This makes the form model easier to read and allows the same rules to be reused.

## Error handling and operational choices

An async check can fail for reasons other than the value being invalid: the network may be offline, the service may be unavailable, or the request may time out. Decide how the form should behave in that case.

Common policies include:

- Return a distinct error such as **usernameCheckFailed** and block submission until the user retries.
- Show that the check could not complete and let the server make the authoritative decision on submit.
- Retry under a deliberate policy when retrying is safe and useful.

Do not convert every service error to **null** without considering the consequence. That marks the value as valid even though the check did not complete. Conversely, report operational failure distinctly from a genuine **usernameTaken** result so the user gets the right instruction.

Also avoid putting unrelated side effects in validators. Angular can call them again when values or validator configuration changes. The validator should return a validation result; logging, notifications, and submission belong in the form or service flow.

## Common mistakes

- **Putting a sibling comparison on one child:** attach a cross-field validator to the common parent that owns both values.
- **Manually setting a child's errors from the group validator:** return a group error and display it near the related controls.
- **Running an availability request before format checks:** use synchronous validators too; Angular runs async validation only after they pass.
- **Returning an infinite observable:** async validation needs a result that completes. Limit multi-emission sources where appropriate.
- **Ignoring PENDING:** a pending form is not ready to submit.
- **Checking only a control's invalid flag:** pending is a distinct state; account for it in the submission flow.
- **Calling the async check on every keystroke:** choose an appropriate **updateOn** policy and use service-side safeguards.
- **Treating a failed request as available:** report the failure or follow a clear fallback policy.
- **Trusting an availability check as a reservation:** enforce uniqueness on the server during the actual create operation.
- **Treating a client validator as a security rule:** the server must validate submitted data independently.
- **Showing a mismatch error before either field has content:** let each required validator explain empty fields, then compare completed values.
- **Assuming the error belongs to a child:** a cross-field error is stored on the group that returned it.

## Next steps

- Continue with [Submission flows, pending state, and reset behavior](068-submission-flows-pending-state-and-reset-behavior.md).
- Review the [Q&A](../qnas/067-cross-field-and-asynchronous-validation.md).
- Work through the [complete example](../examples/067-cross-field-and-asynchronous-validation.md).
- Revisit [Synchronous validators and validation state](066-synchronous-validators-and-validation-state.md).

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="066-synchronous-validators-and-validation-state.md">066 - Synchronous validators and validation state</a></td><td><a href="../qnas/067-cross-field-and-asynchronous-validation.md">Q&amp;A</a></td><td><a href="../examples/067-cross-field-and-asynchronous-validation.md">Example</a></td><td><a href="068-submission-flows-pending-state-and-reset-behavior.md">068 - Submission flows, pending state, and reset behavior</a></td>
</tr>
  </tbody>
</table>
