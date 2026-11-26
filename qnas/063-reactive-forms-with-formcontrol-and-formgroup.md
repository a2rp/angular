# Q&A: Reactive forms with FormControl and FormGroup

These answers review the model-first approach to Angular reactive forms. See the [lesson](../lessons/063-reactive-forms-with-formcontrol-and-formgroup.md) for the walkthrough and the [example](../examples/063-reactive-forms-with-formcontrol-and-formgroup.md) for a complete profile form.

## Core concepts

### 1. What is a reactive form?

A reactive form has an explicit form model created in TypeScript. The component creates its controls and groups, and the template binds inputs to those existing controls.

### 2. What is a FormControl?

A **FormControl** tracks the value, validation status, and interaction state of one field. It can also be used directly with an input through **[formControl]**.

### 3. What is a FormGroup?

A **FormGroup** tracks a fixed set of named child controls as one group. It aggregates their value and validation status and lets the component update or inspect them together.

### 4. Which module enables reactive form directives?

Import **ReactiveFormsModule** from **@angular/forms** into the standalone component that uses the directives, or into the NgModule that declares the component.

### 5. What is the source of truth in a reactive form?

The control tree in TypeScript is the form model. The template displays and edits those controls, while Angular keeps the model and view synchronized.

### 6. How is a FormGroup connected to a form element?

Bind the group with **[formGroup]="profileForm"** on the **form** element. This connects the DOM form to the group instance.

### 7. How is a child control connected to an input?

Use **formControlName** with a key that exists in the parent group. For example, a group key named **email** is connected with **formControlName="email"**.

### 8. What is the difference between formControlName and [formControl]?

**formControlName** looks up a control by name in a parent **FormGroup**. **[formControl]** binds an input directly to a **FormControl** instance. Both are reactive bindings.

### 9. When should I use FormGroup?

Use **FormGroup** when the form has a fixed set of related controls that should share value and validation state. It is useful when code needs to inspect or update controls explicitly.

### 10. What should I use when the number of controls changes at runtime?

Use **FormArray** for a dynamic ordered collection of controls or groups. **FormGroup** is intended for a known set of named members.

## Validators and form state

### 11. How do I add a validator to a control?

Pass a validator or array of validators when constructing the control. For example, use **Validators.required** and **Validators.email** from **@angular/forms**.

### 12. Can one control have multiple validators?

Yes. Pass an array such as **[Validators.required, Validators.minLength(2)]**. Angular evaluates the configured validators and exposes failures through the control's errors and status.

### 13. Where can I read a control's validation errors?

Read **control.errors**, which is an object of validation errors or **null** when there are none. Use **control.hasError('required')** when checking for a specific error.

### 14. What does valid or invalid mean?

A control or group is **valid** when its validators pass. It is **invalid** when at least one applicable validator fails. A group aggregates the status of its child controls.

### 15. What do dirty and touched mean?

**dirty** means the value has changed from its initial or reset state. **touched** means the user has blurred the control. Their opposite states are **pristine** and **untouched**.

### 16. What does pending mean?

**pending** means asynchronous validation has not finished. A submission handler can avoid accepting the form until validation finishes.

### 17. What does markAllAsTouched do?

**markAllAsTouched()** marks the control and its descendants touched. It is commonly called after an invalid submit attempt so validation messages can be displayed for fields the user skipped.

### 18. What is valueChanges?

**valueChanges** is an observable on controls and groups that emits when values change. It can be used to react to edits, derive other UI state, or connect to debounced work. Unsubscribe or use an appropriate lifecycle pattern when subscribing manually.

## Values, updates, and reset

### 19. What does FormGroup.value contain?

It contains the values of the group's enabled controls. Disabled child controls are excluded from the aggregate value.

### 20. How is getRawValue different from value?

**getRawValue()** returns values for all controls, including disabled controls. Use it only when the application intentionally needs disabled-field values.

### 21. What does setValue do on a FormGroup?

**setValue(object)** updates the complete group value and requires a value for every control with the matching structure. A mismatched or missing key is an error, which helps catch shape mistakes.

### 22. What does patchValue do?

**patchValue(object)** updates only the controls included in a partial object. Use it when a partial update is intended; use **setValue** when supplying the complete form shape.

### 23. Can I update one field directly?

Yes. Access the control from the group, then call methods such as **setValue** or **patchValue** on that control. The parent group recalculates its aggregate value and validity.

### 24. What does reset do?

**reset** updates the control values and resets interaction state such as dirty and touched. Supply explicit values when the form should reset to a particular default.

### 25. Why can a FormControl reset to null even if it started with an empty string?

By default, **FormControl** is nullable because a reset may set its value to **null**. Non-nullable controls preserve a non-null default on reset. Typed forms and non-nullability are the focus of the next lesson.

### 26. Do disabled controls still participate in validation?

A disabled control is exempt from validation and excluded from an enabled parent's aggregate **value**. Re-enable it when it should participate again. Do not use disabled state merely to hide invalid data from submission logic.

## Submission and template behavior

### 27. How should a reactive form be submitted?

Bind **(ngSubmit)** on the form and use a button with **type="submit"**. This preserves standard form submission behavior, including Enter-key submission.

### 28. Where should the invalid-form check happen?

Check **form.invalid** and, when relevant, **form.pending** inside the component handler. If invalid, mark controls touched and return before processing the value.

### 29. Why should I still guard the handler if the submit button is disabled?

A disabled button is only a presentation state. The component method owns the operation and must protect it if called through another code path.

### 30. Does the formControlName value have to match exactly?

Yes. Its value must match a key in the parent group. A mismatch or a missing parent group means Angular cannot connect the control and can raise a runtime error.

### 31. Can I put formControlName outside the element with formGroup?

It must be inside the template subtree governed by the corresponding **FormGroup** directive, or inside an appropriate nested group directive. Otherwise Angular has no parent group from which to resolve the control.

### 32. Can I use ngModel and formControlName on the same input?

Do not combine them on one control. They represent different form models and can produce warnings or conflicting state. Use one approach per control.

### 33. Can one application use reactive and template-driven forms?

Yes. Different forms in an application can use different approaches. Avoid mixing both approaches on the same field.

### 34. Does a reactive form replace server validation?

No. Reactive validators provide client-side feedback. Validate submitted data again on the server before trusting or storing it.

### 35. How do I show an error only after the user interacts?

Check both **control.invalid** and an interaction state such as **control.touched** or **control.dirty**. After an invalid submit attempt, call **markAllAsTouched()** so errors are visible.

### 36. When are reactive forms a better fit than template-driven forms?

They are often a better fit when explicit programmatic control, complex validation, dynamic structures, reusable form logic, or focused form testing are important. For a small static form, template-driven forms can be simpler.

## Practice

1. Create a **FormGroup** with required name and email controls.
2. Bind each input using **formControlName** and display the control's state.
3. Add a submit handler that guards invalid and pending forms.
4. Call **setValue** with the full form shape and **patchValue** with one field.
5. Reset the form to explicit empty-string values and observe the state.
6. Disable one control, then compare **value** with **getRawValue()**.

For a complete example, continue to the [example](../examples/063-reactive-forms-with-formcontrol-and-formgroup.md).

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="062-template-driven-forms-and-ngform.md">062 - Template-driven forms and NgForm</a></td><td><a href="../lessons/063-reactive-forms-with-formcontrol-and-formgroup.md">Lesson</a></td><td><a href="../examples/063-reactive-forms-with-formcontrol-and-formgroup.md">Example</a></td><td><a href="064-typed-reactive-forms-and-non-nullable-controls.md">064 - Typed reactive forms and non-nullable controls</a></td>
</tr>
  </tbody>
</table>
