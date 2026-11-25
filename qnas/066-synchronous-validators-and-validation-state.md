# Q&A: Synchronous validators and validation state

Use these questions to review Angular's built-in and custom synchronous validators, the error state they produce, and how a form should communicate that state. See the [lesson](../lessons/066-synchronous-validators-and-validation-state.md) for examples and the [complete example](../examples/066-synchronous-validators-and-validation-state.md) for a working reactive form.

## Validator basics

### 1. What is a synchronous validator?

A synchronous validator checks a control immediately and returns either **null** or a **ValidationErrors** object. It does not wait for a network request or another asynchronous operation.

### 2. What is the ValidatorFn type?

**ValidatorFn** describes a function that receives an **AbstractControl** and returns **ValidationErrors | null**. Angular calls it when the control's value or validation configuration is recalculated.

### 3. What does null mean when returned by a validator?

It means this validator found no error for the current value. If every active validator returns **null**, the enabled control can be valid.

### 4. What does an error object mean?

It reports a validation failure. Its property names are error keys, and its values can contain details that the interface needs, such as the minimum length and the actual length.

### 5. Should a validator return false when a value is invalid?

No. Return an error object, for example **{ forbiddenTerm: true }**. Angular uses the presence of an error object to update the control's errors and invalid status.

### 6. Should a validator return an empty object for success?

No. An empty object is still an error object. Return **null** to report success.

### 7. Where do synchronous validators go when creating a FormControl?

Pass them as the second argument. This can be one validator function, an array of functions, or an options object with a **validators** property.

### 8. How do I apply multiple validators?

Pass an array, such as **[Validators.required, Validators.minLength(3)]**. Angular runs the active synchronous validators and merges their errors.

### 9. Can a control have more than one validation error at once?

Yes. Every applicable synchronous validator runs, and multiple error keys can be present together. Show the messages that apply or intentionally prioritize one message in the template.

### 10. Are synchronous validators the same as native HTML validation?

No. Reactive forms keep their validation functions in the component's form model. HTML attributes such as **required** can still communicate semantics and constraints to users and assistive technology.

## Built-in validators

### 11. What does Validators.required check?

It checks that the control has a non-empty value. Add it when the field is mandatory.

### 12. When should I use Validators.requiredTrue?

Use it when the value must be **true**, commonly for a checkbox that represents consent or agreement. **required** and **requiredTrue** express different rules.

### 13. What do Validators.min() and Validators.max() check?

They check whether a numeric value is above a minimum or below a maximum. For example, **Validators.min(18)** rejects a numeric value below 18.

### 14. What do Validators.minLength() and Validators.maxLength() check?

They check the length of a value that has a numeric **length** or **size**, such as a string or array. Their error keys are **minlength** and **maxlength**.

### 15. Does minLength make a field required?

No. Angular intentionally skips the minimum-length check when the value has length zero, which allows optional controls to remain empty. For a required string with a minimum length, combine **required** and **minLength**.

### 16. How do I validate an optional field with a minimum length?

Apply **minLength** without **required**. An empty value can then pass, while a non-empty value shorter than the minimum fails.

### 17. Does Validators.minLength(1) require a non-empty array?

No. The validator skips empty values. If a collection must contain an entry, add a collection-level rule that explicitly requires at least one item.

### 18. What does Validators.email check?

It checks whether a value matches a format suitable for a common email address. It does not prove that the address exists, belongs to the user, or can receive messages.

### 19. What does Validators.pattern() do?

It checks whether a value matches a supplied string pattern or regular expression. Use it for a well-defined client-side format rule and explain the expected format to the user.

### 20. Why should I avoid a global or sticky RegExp with Validators.pattern()?

A **RegExp** with **g** or **y** retains a match index between calls to **test()**. Consecutive validation runs can therefore produce inconsistent results. Use a regular expression without those flags.

### 21. Do validators make a value safe for a server or database?

No. Client-side validators improve feedback and catch common mistakes early. The server must validate and authorize submitted data independently.

## Custom synchronous validators

### 22. When should I write a custom validator?

Write one when a rule is specific to the application and cannot be expressed clearly with Angular's built-in validators, such as rejecting a reserved account name.

### 23. How can one custom validator be reused with different values?

Return a **ValidatorFn** from a factory that accepts configuration. For example, **forbiddenTermValidator('admin')** can create a validator configured with one term.

### 24. How should a custom validator handle an optional empty value?

Return **null** for the empty value if the validator only checks content and the field is optional. Put the required rule in **Validators.required** so each validator has one clear job.

### 25. What should a custom validator return for an error?

Return a map with a descriptive, stable key and any useful details, for example **{ forbiddenTerm: { term: 'admin' } }**. The template can check the key with **hasError()** and display the appropriate message.

### 26. Should a custom validator call setErrors()?

No. Return the error result. Angular uses that return value to calculate errors; directly mutating the control during validation can overwrite other results or cause confusing recalculation.

### 27. Should validators make HTTP requests?

No. An HTTP request is asynchronous. Keep synchronous validators immediate and use an asynchronous validator for a server-backed check; that topic is introduced in the next lesson.

### 28. Should a validator modify the value it receives?

No. A validator should check the value and report an error. Normalize or transform input in a separate, explicit form or application operation.

## Validation state and error messages

### 29. What is the difference between valid and invalid?

**valid** is true when the control's status is **VALID**. **invalid** is true when the status is **INVALID**. These booleans do not describe whether the user has interacted with the control.

### 30. What values can a control's status have?

A control's status is **VALID**, **INVALID**, **PENDING**, or **DISABLED**. **PENDING** is useful when asynchronous validation is running; **DISABLED** means the control is excluded from validation and enabled-parent aggregate values.

### 31. What is the difference between dirty and touched?

**dirty** indicates that the user changed a value through the UI. **touched** indicates the user focused and then left the control. A user can touch a field without changing it, or change a field and leave it untouched programmatically.

### 32. Why show errors only after dirty or touched?

It avoids showing errors before the user has had a chance to interact. A common condition is **invalid && (dirty || touched)**.

### 33. How do I mark every field touched after an invalid submission?

Call **form.markAllAsTouched()** before returning from the submit handler. This reveals field messages even when the user submits without leaving each input.

### 34. How do I check for a specific error?

Use **control.hasError('required')** to check whether an error key exists. Use **control.getError('minlength')** or **control.errors** when the template or component needs the error's details.

### 35. Should the template use else-if for every error?

Only if the interface should display one prioritized error. Separate **@if** blocks allow multiple applicable messages to be shown at once.

### 36. What are Angular's ng-valid and ng-invalid classes?

Angular adds CSS classes that mirror form state, including **ng-valid**, **ng-invalid**, **ng-pending**, **ng-pristine**, **ng-dirty**, **ng-untouched**, and **ng-touched**. They can support visual styling but do not replace readable error messages.

### 37. What should I include for accessible validation feedback?

Use a real label, connect hints and errors with **aria-describedby**, and expose the relevant invalid state with **aria-invalid**. Include readable text rather than relying on color alone. Keep **required** semantics for mandatory fields.

### 38. Should I disable the submit button whenever the form is invalid?

That depends on the interaction design. If the button is disabled before submission, users may not trigger a submit attempt that reveals errors. A common approach is to allow submission, mark invalid controls touched, and block the actual save in the handler.

## Validation timing and submission

### 39. When do reactive forms run validators by default?

They normally recalculate validation as the control value changes. A control's **updateOn** option can defer view updates and validation to **'blur'** or **'submit'**.

### 40. When is updateOn: 'blur' useful?

It can avoid showing a format error on every keystroke. The control updates and validates after the user leaves the field. Choose it when delayed feedback is clearer for that field.

### 41. Does updateOn delay programmatic setValue() calls?

No. Programmatic updates still change the model and recalculate validity. **updateOn** controls when values from the view are committed.

### 42. What should a submit handler check?

Check **invalid** and, if asynchronous validators can exist in the form, **pending** before saving. Mark the controls touched when the form cannot be submitted so users can locate the errors.

### 43. Do asynchronous validators run if synchronous validation fails?

Angular runs asynchronous validators only after synchronous validation passes. While asynchronous validation is running, the control can have **PENDING** status. Detailed async behavior is covered next.

### 44. Can an invalid form still have useful values?

Yes. Validation status describes whether the values satisfy the rules; it does not erase the values. Read them for feedback or correction, but do not treat an invalid form as ready to submit.

### 45. Does a disabled control fail a required validator?

Disabled controls are exempt from validation. If a value remains required, do not disable that control to bypass its error.

## Official references

- [Validate form input](https://angular.dev/guide/forms/form-validation)
- [Validators API](https://angular.dev/api/forms/Validators)
- [Reactive forms](https://angular.dev/guide/forms/reactive-forms)
- [AbstractControl API](https://angular.dev/api/forms/AbstractControl)
