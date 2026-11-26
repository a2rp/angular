# Q&A: Submission flows, pending state, and reset behavior

These questions review a complete Reactive Forms submission flow: validate, wait for async checks, send once, report the result, and reset at the right time. Read the [lesson](../lessons/068-submission-flows-pending-state-and-reset-behavior.md) for the full explanation and build the [example](../examples/068-submission-flows-pending-state-and-reset-behavior.md) to see a delayed availability check and simulated save request.

## Submission events and guards

### 1. What should handle a form submission in Angular?

Use the form's **(ngSubmit)** event. It represents submitting the form and works with submit-button activation and keyboard submission.

### 2. Why not put the whole save operation on a button's click handler?

A click-only handler does not represent the form submission itself and can miss keyboard-driven submission. Keep the action on **ngSubmit**.

### 3. Which states should a submit handler guard against?

At minimum, stop when the form is **invalid**, **pending**, or already **saving**. These represent a rejected form, unresolved validation, and an in-flight request.

### 4. Why check both **invalid** and **pending**?

A pending form has not yet received the result of its asynchronous validation. It should not be submitted as though it were valid simply because **invalid** is currently false.

### 5. What does a form's **PENDING** status mean?

At least one asynchronous validation result is unresolved, so Angular has not yet determined the final validity. The status changes when that validation completes.

### 6. Is **PENDING** the same as saving to the server?

No. **PENDING** is Angular validation work. A component flag such as **saving** describes the application's network request. They should have distinct feedback.

### 7. What does **markAllAsTouched()** do in a submit handler?

It marks the controls as touched so validation messages can become visible after a submit attempt. It does not make an invalid form valid or finish an async validator.

### 8. What should happen if a user submits while validation is pending?

A simple flow marks controls as touched, returns without sending, and lets the user submit again after the check finishes. If the UI should automatically continue, implement a deliberate status-observation flow that can only submit once.

### 9. Should an async validator send the final save request?

No. A validator answers a validation question and returns validation errors or **null**. Keep persistence in the submit flow.

### 10. Why check **saving** in the handler if the button is disabled during a save?

The handler is the operation's guard, while the disabled button is user feedback. A second event or another call path should not be able to start a duplicate request.

## Saving, errors, and server responses

### 11. When should the saving flag be set?

Set it before subscribing to or starting the request. That closes the interval in which a second submission could begin another request.

### 12. How can the saving flag be cleared reliably for an Observable request?

Use RxJS **finalize()** when the observable completes, errors, or is unsubscribed. This keeps cleanup out of separate success and error branches.

### 13. Should the request payload be captured before starting the request?

Yes. Capture a deliberate snapshot at submit time. This makes clear which values the request represents.

### 14. What is the difference between a group's **value** and **getRawValue()**?

A group's **value** excludes disabled descendants. **getRawValue()** includes them. Choose based on the API contract and the meaning of those disabled fields.

### 15. Should the UI clear the fields before sending the request?

Usually no. If the request fails, the user would lose the values they need to correct or retry. Wait for confirmed success before resetting.

### 16. What should happen to field values after a request error?

Preserve them, clear the saving state, and show an actionable message. The user should be able to retry or correct the form without re-entering everything.

### 17. Is client-side validation sufficient to enforce business rules?

No. The browser can be modified, stale, or disconnected. The server must validate the submitted payload and permissions.

### 18. Should every server error be stored with **setErrors()** on a control?

Not automatically. A later validation run can replace manually assigned errors. A separate server-error display is often simpler; if server errors are mapped into controls, design how they are cleared and recalculated.

### 19. When should a success message appear?

After the server confirms success. Do not announce completion merely because the client sent a request.

### 20. Does a client-side saving flag prevent duplicate operations completely?

No. It prevents repeat interaction in that component instance. For high-impact operations, the server may also need idempotency or another duplicate-request strategy.

## Reset behavior

### 21. What does **FormGroup.reset()** do?

It resets descendant values and interaction state. Controls become pristine and untouched, and the group's validity is recalculated for the reset values.

### 22. What values do controls get after **reset()** with no argument?

It depends on how they were created. Nullable controls commonly reset to **null**; controls from **FormBuilder.nonNullable** reset to their initial values.

### 23. How can a form reset to explicit empty strings?

Pass a matching value object to **reset()**, such as **{ name: '', email: '' }**, or pass it to the directive's **resetForm()** method.

### 24. What is **FormGroupDirective.resetForm()** for?

It resets the form model and the directive's submitted state. Use it when both the control state and the form directive's submission state should be cleared.

### 25. Does calling **form.reset()** clear all component UI state?

No. It resets the form model and interaction state, not arbitrary component properties. Clear flags such as **saving**, request errors, and success messages according to the desired flow.

### 26. Should reset happen before or after the save request?

Usually after confirmed success for a completed action. Resetting first risks discarding the user's input if the request fails.

### 27. Does resetting a **FormArray** remove its rows?

No. Resetting changes control values and state; it does not remove the array's controls. Use structural methods such as **clear()** or **removeAt()** when the row structure should change.

### 28. Can a reset set a control's disabled state?

Yes. A **FormControlState** object supplies both **value** and **disabled**, for example **{ value: 'Guest', disabled: true }**.

### 29. Does a disabled field automatically return to its old state after reset?

Do not rely on that assumption for an intended reset state. Specify the desired disabled state in the reset value when it matters.

### 30. Is a disabled field safe to trust on the server?

No. Disabled is a client-side UI and form-model state, not an authorization rule. The server must validate values and permissions independently.

## Buttons, retries, and accessible feedback

### 31. Should the submit button be disabled whenever the form is invalid?

Not necessarily. Keeping it enabled lets a submit attempt reveal validation messages. The handler must still guard against invalid and pending forms.

### 32. When should the submit button be disabled?

Disabling it while **saving** prevents repeat clicks and communicates that the request is in progress. The handler must also check the saving flag.

### 33. What type should a custom clear button use?

Use **type="button"**. A button inside a form otherwise defaults to submit behavior in HTML and may trigger an unintended save.

### 34. Does canceling or unsubscribing from a request prove the server did nothing?

No. It can stop client-side observation or abort an in-flight transport, but the server may already have received and processed the request.

### 35. What does a good pending indicator communicate?

It says an async validation check is still running and the user should wait. It should not claim that the save request is underway.

### 36. How should an in-flight save be announced?

Use visible text such as “Saving…” and, where useful, a polite live region. Do not rely only on color or disabling a control.

### 37. What feedback belongs after a failed save?

A clear error that says the save failed and what the user can do next. Keep the form values available and avoid exposing raw internal exception details.

### 38. What should happen to stale messages when the user edits or retries?

Clear or update them. A previous success or server error should not appear to describe newly edited values.

### 39. Should every form value change be announced through a live region?

No. Announce meaningful state changes such as a pending check, saving, success, or failure. Announcing every keystroke can overwhelm assistive-technology users.

### 40. What is the essential submission checklist?

Use **ngSubmit**; guard invalid, pending, and saving states; report progress; clear saving state on all request outcomes; preserve values on failure; and reset form, directive, and application state intentionally after success.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="067-cross-field-and-asynchronous-validation.md">067 - Cross-field and asynchronous validation</a></td><td><a href="../lessons/068-submission-flows-pending-state-and-reset-behavior.md">Lesson</a></td><td><a href="../examples/068-submission-flows-pending-state-and-reset-behavior.md">Example</a></td><td><a href="069-accessible-labels-instructions-and-validation-messages.md">069 - Accessible labels, instructions, and validation messages</a></td>
</tr>
  </tbody>
</table>
