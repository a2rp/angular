# Q&A: Cross-field and asynchronous validation

These questions review validators that compare sibling controls and checks that need an asynchronous result. Read the [lesson](../lessons/067-cross-field-and-asynchronous-validation.md) for the complete explanation and build the [example](../examples/067-cross-field-and-asynchronous-validation.md) to see a password match rule and a delayed username availability check working together.

## Cross-field validation

### 1. What is cross-field validation?

It checks a rule that depends on the values of two or more controls, such as whether a password and its confirmation match.

### 2. Where should a cross-field validator be attached?

Attach it to the nearest common parent that contains the controls, usually their **FormGroup**. The parent can inspect its child controls and report one result for their relationship.

### 3. Why not attach a password-match rule to only the confirmation control?

The rule compares two sibling values. Attaching it to the group makes its ownership clear and keeps the validation error with the control that evaluates both fields.

### 4. Where does a group validator's error appear?

On the **FormGroup** that owns the validator. Check it with **group.hasError('passwordMismatch')** or inspect the group's **errors**.

### 5. Does a group-level error automatically become a child error?

No. The error remains on the group. The parent form becomes invalid, but Angular does not copy the error onto the password or confirmation control.

### 6. Should a cross-field validator call setErrors() on a child?

No. Return an error map from the group validator and show the related message near the fields. Manually mutating child errors can overwrite their own validation results.

### 7. What should a password-match validator do when one or both fields are empty?

Let each field's **required** validator report its own empty value. The group validator can skip comparison until both values are present, then return the mismatch error if they differ.

### 8. Does the group validator run when a child value changes?

Angular recalculates parent validity as child values and statuses update, so the group-level rule is evaluated against the current child values.

### 9. How should a group error message be shown?

Check the group's error key and an interaction condition such as **dirty**, **touched**, or a submit-attempt flag. Put the message close to the related fields and make it readable.

### 10. Can a group have both child errors and a cross-field error?

Yes. Child controls keep their own errors, and the group can have its own error map. The root form combines their validity.

### 11. Should a password comparison trim whitespace?

Only if the password policy explicitly says spaces are insignificant. Passwords commonly treat spaces as part of the secret, so do not normalize them without a clear requirement.

### 12. Can cross-field validation be attached to a FormArray?

Yes. A validator can be attached to a common **FormGroup** or **FormArray** when the rule depends on that container's children, such as a duplicate-value rule across rows.

## Asynchronous validator basics

### 13. What is an AsyncValidatorFn?

It is a function that receives an **AbstractControl** and returns a **Promise** or **Observable** that eventually resolves to **ValidationErrors | null**.

### 14. What does an async validator return when the value passes?

Return or emit **null**. Return or emit a validation error object when the value fails the check.

### 15. Where do async validators go on a FormControl?

Use the **asyncValidators** option in a **FormControl** options object. With positional constructor arguments, async validators are the third argument after the synchronous validators.

### 16. When does Angular run an async validator?

Angular first runs synchronous validators. It starts async validation only after the synchronous checks pass, which avoids requests for values that are already locally invalid.

### 17. What does PENDING mean?

The control's asynchronous validation has started but has not yet produced its result. The control can be **PENDING** until the Promise resolves or the Observable emits and completes.

### 18. Is a pending form valid?

No. A pending control's status is neither **VALID** nor **INVALID**. In a submission handler, check **form.pending** as well as **form.invalid** before saving.

### 19. Does an Observable returned by an async validator need to complete?

Yes. Return a finite Observable that emits a result and completes. For a source that can emit repeatedly, use an operator such as **take(1)** when one result is intended.

### 20. Which result does Angular use from an Observable validator?

Angular waits for the asynchronous validator result; the Observable should emit the validation result and complete. Design the stream to produce one clear final result rather than a long-lived stream.

### 21. Is an HTTP request a suitable async validator source?

It can be. A normal Angular **HttpClient** request Observable emits its response and completes. Map the response to **null** or an error object, and handle request failures deliberately.

### 22. Can an async validator use a Promise instead?

Yes. **AsyncValidatorFn** accepts a Promise or an Observable that resolves to **ValidationErrors | null**. Observables are convenient when using Angular **HttpClient** and RxJS operators.

### 23. What is the difference between AsyncValidatorFn and AsyncValidator?

**AsyncValidatorFn** is a function type. **AsyncValidator** is an interface commonly implemented by a class or directive that exposes a **validate()** method.

### 24. Can async validation run if required validation fails?

No. Angular skips async validation until all synchronous validators pass. An empty required field should therefore fail locally without an availability request.

## Error and service behavior

### 25. How should a username availability response map to validation?

Map an available result to **null** and an unavailable result to a named error such as **{ usernameTaken: true }**. Keep the error key distinct from transport or service failures.

### 26. What should happen if the availability request fails?

Choose an explicit product policy. The form can report a separate check-failed error and allow retry, or use a defined fallback flow. Do not silently report success unless that is an intentional decision.

### 27. Why distinguish usernameTaken from usernameCheckFailed?

The user needs different instructions. A taken name requires choosing another value; a failed check requires retrying or waiting for the service.

### 28. Should every service error become null?

No. Returning **null** marks the value valid even though the check may not have completed. Use that behavior only when the application deliberately accepts the risk and the server remains authoritative.

### 29. Does an availability check reserve a username?

No. It is only a point-in-time check. Another user can claim the same value before submission, so the server must enforce uniqueness when creating the account.

### 30. Should a validator save or reserve data?

No. Validation should report whether the current value meets a rule. Persistence and reservation belong in an explicit application operation with appropriate server-side controls.

### 31. Should validators show notifications or log user-facing events?

Generally no. Angular may rerun validators as values and configuration change. Return the validation result; put notifications and submission side effects in the component or application service flow.

### 32. How can I avoid a request on every keystroke?

Use an appropriate **updateOn** strategy such as **'blur'** or **'submit'**. A backend may also apply rate limits or other safeguards. Choose based on the desired feedback timing.

### 33. What does updateOn: 'blur' do?

It waits until the user leaves the control before committing view changes and recalculating validation for that control. It can make a remote check less chatty than the default **'change'** strategy.

### 34. What does updateOn: 'submit' do?

It delays updates from the view until form submission. Use it only when delayed feedback fits the interaction; it may not be suitable when users need immediate guidance.

### 35. What is the default updateOn value?

The default is **'change'**. A control can also inherit an update strategy from a parent control unless it specifies its own.

## Submission and user feedback

### 36. How should a submit handler respond to PENDING?

Do not save yet. Show a pending message or keep the user informed, and continue the save flow only after validation has settled and the form is valid.

### 37. Should the submit button be disabled while validation is pending?

That is an interface choice. If disabled, provide a visible explanation or progress state. If enabled, the submit handler must still check **pending** and return safely.

### 38. Why mark all controls touched on a rejected submit?

It reveals validation messages for fields the user may not have visited. Call **markAllAsTouched()** before returning when the form is invalid or still pending and the interface should show the relevant messages.

### 39. How should an async pending state be communicated?

Use readable status text near the field, such as “Checking username availability,” and expose it with an appropriate status role. Do not communicate the wait only by disabling the submit button.

### 40. Is client-side cross-field validation a security rule?

No. It improves the form experience but can be bypassed. The server must validate any rule that protects stored data, access, or business integrity.

### 41. Can I use a cross-field validator together with async field validation?

Yes. A form can have a group-level cross-field validator and a child-level async validator. The root form stays invalid or pending until all relevant validations allow it to become valid.

### 42. What should I do if a group error appears while its inputs are blank?

Keep required-field errors on the child controls and have the group validator skip comparison until both values are present. This avoids showing a misleading mismatch alongside empty-field messages.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="066-synchronous-validators-and-validation-state.md">066 - Synchronous validators and validation state</a></td><td><a href="../lessons/067-cross-field-and-asynchronous-validation.md">Lesson</a></td><td><a href="../examples/067-cross-field-and-asynchronous-validation.md">Example</a></td><td><a href="068-submission-flows-pending-state-and-reset-behavior.md">068 - Submission flows, pending state, and reset behavior</a></td>
</tr>
  </tbody>
</table>
