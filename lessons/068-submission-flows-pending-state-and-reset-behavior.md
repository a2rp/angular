# Submission flows, pending state, and reset behavior

A valid form is only one part of a reliable submission flow. A user can submit while an asynchronous validator is still checking, click twice before the first request finishes, or need to correct a value after the server rejects it. A clear flow checks the form state, reports work in progress, handles success and failure, and resets only when appropriate.

This lesson focuses on **Reactive Forms**. Angular validation and the network request have separate states:

- **PENDING** means an Angular async validator has not finished.
- **saving** (or **submitting**) is application state meaning the save request is in progress.

Keep those states distinct so the interface can explain what is happening.

## Learning outcomes

After this lesson, you should be able to:

- Handle submissions with **(ngSubmit)**.
- Guard against invalid, pending, or already-saving forms.
- Distinguish **PENDING** validation from a request in progress.
- Prevent duplicate requests and present success and server-error feedback.
- Reset values and interaction state at the right point in the flow.
- Choose between **FormGroup.reset()** and **FormGroupDirective.resetForm()**.
- Set reset defaults and disabled state intentionally.

## Model the submission lifecycle

A reliable submission usually follows these steps:

1. The user submits the form.
2. The handler checks the form status and app-level **saving** flag.
3. If the form is invalid or pending, show validation feedback and stop.
4. Otherwise, capture the values and start the request.
5. Show progress and prevent another request.
6. On success, confirm the result and reset only if the action is complete.
7. On failure, preserve the user's input and explain what they can do next.
8. Clear the saving flag when the request completes or errors.

Use the form's status rather than checking only individual fields. A group-level validator can make the whole form invalid, while an async validator can make it pending.

## Handle the submit event

Bind **ngSubmit** on the form. It covers submit-button activation and keyboard submission:

~~~html
<form [formGroup]="contactForm" (ngSubmit)="submit()" novalidate>
  <!-- fields -->
  <button type="submit">Send</button>
</form>
~~~

Do not attach the main action only to a button's **click** event. The form's submit event is the semantic action and supports keyboard use.

A handler should guard all three relevant conditions:

~~~ts
submit(): void {
  if (this.contactForm.invalid || this.contactForm.pending || this.saving) {
    this.contactForm.markAllAsTouched();
    return;
  }

  // Snapshot the exact values being submitted.
  const payload = this.contactForm.getRawValue();

  this.saving = true;
  this.submitError = null;
  this.successMessage = null;

  this.contactApi.saveContact(payload)
    .pipe(
      take(1),
      finalize(() => {
        this.saving = false;
      }),
    )
    .subscribe({
      next: (result) => {
        this.successMessage = `Message saved. Reference: ${result.reference}`;
      },
      error: () => {
        this.submitError = 'The message could not be saved. Your entries are still here; please try again.';
      },
    });
}
~~~

This uses **take(1)** and **finalize()** from RxJS. A normal HTTP request emits once and completes; **take(1)** makes that one-result expectation explicit. **finalize()** runs on completion, error, or unsubscription, so the saving state is not left stuck after a normal request failure.

The payload is captured before the request starts. Choose **value** or **getRawValue()** intentionally: a group's **value** omits disabled descendants, while **getRawValue()** includes them.

## Treat PENDING as validation work

An async validator can put a control, and its parent form, into **PENDING**. While pending, Angular has not yet determined whether the value is valid. Do not treat **!form.invalid** as proof that the form is ready: a pending form is not necessarily invalid.

Use a guard such as:

~~~ts
if (this.contactForm.invalid || this.contactForm.pending || this.saving) {
  this.contactForm.markAllAsTouched();
  return;
}
~~~

The user can wait for the async check and submit again. If the product should automatically continue after validation finishes, implement that deliberately by observing form status and ensuring submission runs once. Do not launch a request from a validator or assume the result has arrived.

Show a status message while a check is pending. Validation work is distinct from saving the form to the server.

## Prevent duplicate requests and show progress

Set **saving** before subscribing to the request. The handler guard closes the gap where a second click could start another request. Disabling the button during a save gives the user matching feedback:

~~~html
<button type="submit" [disabled]="saving">
  {{ saving ? 'Saving…' : 'Send' }}
</button>

<p role="status" aria-live="polite">
  @if (saving) {
    Saving your message…
  }
</p>
~~~

It is often useful to leave the submit button enabled while the form is invalid, so the user's submit attempt can reveal validation messages. The handler remains the authoritative guard. Disable it while **saving** to prevent repeated requests.

For high-impact operations, a client-side flag is not enough. The server should also make the operation safe against retries, for example with an idempotency key for a payment.

## Keep server errors separate from client validation

Client validators provide immediate feedback, but the server remains authoritative. A request can fail because the network is unavailable, account state changed, or a server-side rule rejected the values.

Keep request errors distinct from a validator's **errors** map unless you have a deliberate design for mapping server field errors into control state. Calling **setErrors()** directly can be overwritten by a later validation run. Preserve entered values on request failure so the user can correct or retry.

Clear stale success and request-error messages when the user edits or retries. Confirm success only after the server confirms it.

## Reset after success, not before

A reset clears values and interaction state. If the request fails, resetting before its result discards the user's work. Usually preserve values while saving and reset only after confirmed success when the form represents a completed action, such as a sent message.

**FormGroup.reset()** resets values and interaction state. Descendants become pristine and untouched. Reset values depend on the control defaults:

- A nullable control commonly resets to **null**.
- A control created by **FormBuilder.nonNullable** resets to its initial value.
- You can pass values to **reset()** to choose the next values explicitly.

Resetting a form group does not remove controls or rows from a **FormArray**. Reset values separately if needed; use the array's structural APIs when rows should be removed.

## Reset the directive's submitted state too

A **FormGroupDirective** tracks whether the form has been submitted. When that state also needs resetting, call **FormGroupDirective.resetForm()**:

~~~html
<form
  [formGroup]="contactForm"
  #formDirective="ngForm"
  (ngSubmit)="submit(formDirective)"
>
  <!-- fields -->
</form>
~~~

~~~ts
import { FormGroupDirective } from '@angular/forms';

submit(formDirective: FormGroupDirective): void {
  // Validate and send the request first.

  // After confirmed success:
  formDirective.resetForm({
    name: '',
    email: '',
  });
}
~~~

**resetForm()** resets both the form model and the directive's submitted state. Component flags such as **saving**, **submitError**, and **successMessage** are application state; clear or update them separately.

## Choose reset values and disabled state

You can pass values when resetting:

~~~ts
this.contactForm.reset({
  name: '',
  email: '',
});
~~~

A **FormControlState** object sets both a control's value and disabled state:

~~~ts
this.settingsForm.reset({
  displayName: { value: 'Guest', disabled: true },
  updatesEnabled: { value: false, disabled: false },
});
~~~

Use this when the intended reset state includes disabled controls. When that state matters, specify it explicitly rather than assuming reset will restore the previous enabled state.

Disabled controls are excluded from normal aggregate validation and from a group's **value**. Use **getRawValue()** when the request intentionally needs disabled values. A disabled UI control is never an authorization mechanism; the server must validate data and permissions.

## Handle cancel and clear actions

A custom clear action should be a **type="button"** so it does not submit the form:

~~~html
<button type="button" (click)="resetForm(formDirective)">
  Clear form
</button>
~~~

Consider whether clearing should confirm before discarding substantial work. During a save, either prevent clearing or define cancellation behavior. Unsubscribing from an HTTP observable may abort a client request, but does not prove the server has not processed it.

## Accessibility and feedback

Communicate each state without relying only on color or a disabled button:

- Associate field errors with their controls and expose them to assistive technology.
- Use a live status message for async checking and saving.
- Provide clear success or failure feedback.
- Preserve values and keep focus sensible after a failed request.
- After success, consider moving focus to a confirmation message when the context calls for it.
- Do not announce every changing form value as a live update.

## Common mistakes

- Checking **invalid** but forgetting **pending**.
- Treating an unresolved check as success.
- Setting **saving** after starting the request, leaving a duplicate-submit gap.
- Disabling submit for invalid forms before users can reveal errors.
- Resetting before the server confirms success.
- Clearing values on a server failure.
- Assuming **form.reset()** also clears custom status messages.
- Using **form.value** when disabled fields must be included.
- Assuming a client-side guard prevents duplicate server operations.
- Using a native reset action without accounting for Angular and application state.

## Practical checklist

Before shipping, confirm that:

- Submission is handled by **ngSubmit**.
- Invalid, pending, and already-saving states stop a new request.
- Validation feedback appears after interaction or a submit attempt.
- Pending checks and network saves have distinct indicators.
- The saving flag clears on success and failure.
- The request payload is captured intentionally.
- Failure preserves values and offers useful feedback.
- Reset occurs at the right point and uses intended defaults.
- Directive and application-level state are reset where needed.
- The server validates the final payload and handles retries safely.

## Further reading

- [Reactive forms](https://angular.dev/guide/forms/reactive-forms)
- [Form validation](https://angular.dev/guide/forms/form-validation)
- [FormGroup API](https://angular.dev/api/forms/FormGroup)
- [AbstractFormDirective API](https://angular.dev/api/forms/AbstractFormDirective)
- [FormControlStatus API](https://angular.dev/api/forms/FormControlStatus)
