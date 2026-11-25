# Q&A: Template-driven forms and NgForm

Use these questions to review Angular template-driven forms. See the [lesson](../lessons/062-template-driven-forms-and-ngform.md) for the concepts and the [example](../examples/062-template-driven-forms-and-ngform.md) for a complete working form.

## Core concepts

### 1. What is a template-driven form?

It is an Angular form whose controls and much of its structure are declared with directives in the template. Angular creates and maintains the control model from those directives, while the component usually stores submitted data in a plain object.

### 2. Which Angular package provides template-driven form directives?

The directives are exported by **@angular/forms**. Import **FormsModule** into the standalone component that uses them, or into the NgModule that declares the component.

### 3. What does FormsModule provide?

It makes directives such as **NgForm**, **NgModel**, and **NgModelGroup** available. When active on a regular form, **NgForm** tracks registered controls and aggregate form state.

### 4. What does NgForm do?

**NgForm** creates and manages the top-level form group for a template-driven **form** element. It exposes the aggregate value, validity, interaction state, and the registered controls.

### 5. Do I need to write an ngForm attribute on the form element?

No. With **FormsModule** imported, **NgForm** attaches to a regular **form** element automatically. A template reference such as **#contactForm="ngForm"** is optional, but useful when the template or submit handler needs the directive instance.

### 6. What does NgModel do?

**NgModel** connects a form element to a component property and creates control state for that element. With **[(ngModel)]**, changes flow between the input and the bound property.

### 7. What is the source of truth in a template-driven form?

The component property is the application data model, and the template declares how controls bind to it. Angular's form directives keep the input, component value, and control state synchronized. Avoid maintaining a second competing value for the same field.

## Control registration and binding

### 8. Why does an ngModel control inside a form need a name?

The **name** is the registration key Angular uses to add that control to its parent **NgForm** or **NgModelGroup**. Without it, the control cannot register as a named member of the form in the usual way.

### 9. Do control names need to be unique?

Yes, among controls registered in the same group. Duplicate names collide in the aggregate form value and make it unclear which control owns that key. Use descriptive names such as **email** or **postalCode**.

### 10. What does #contactForm="ngForm" mean?

It declares a template reference variable and exports the **NgForm** directive into that variable. The reference is to Angular's form directive, not to the raw DOM element.

### 11. What does #emailControl="ngModel" mean?

It exposes the **NgModel** directive for that input. The template can then inspect properties such as **emailControl.invalid**, **emailControl.errors**, and **emailControl.touched**.

### 12. What is the difference between [(ngModel)] and [ngModel]?

**[(ngModel)]** is two-way binding: it writes the component value to the input and updates the component property when the input changes. **[ngModel]** passes a value into the control without automatically assigning later user changes back to the component property.

### 13. When should a control be standalone?

Use **[ngModelOptions]="{ standalone: true }"** when a control is intentionally outside the parent form's registered value and validity, such as a local filter next to a form. It can still bind to a component property, but it will not appear in the **NgForm** aggregate value.

### 14. What does ngModelGroup do?

**NgModelGroup** groups related registered controls under a nested key in the parent form's aggregate value. It is useful for related fields such as an address. It does not create a separate HTML form.

## Validation and interaction state

### 15. Which built-in validation rules can be declared in a template?

Common rules include **required**, **minlength**, **maxlength**, **min**, **max**, and **pattern**. Angular also provides validators such as **email**. The relevant directive must be active on the control, typically through **ngModel** and **FormsModule**.

### 16. What does valid or invalid tell me?

**valid** is true when the control or form passes its current validators; **invalid** is true when one or more validators fail. The form's aggregate state accounts for the registered child controls.

### 17. What is the difference between pristine/dirty and untouched/touched?

**pristine** means the value has not changed from its initial or reset state; **dirty** means it has changed. **untouched** means the user has not blurred the control; **touched** means the control has been blurred.

### 18. What does pending mean?

**pending** means an asynchronous validator is still checking the control or form. A pending control is not yet known to be valid, so submission flows may need to wait until validation finishes.

### 19. When should validation messages appear?

A common pattern is to show a message when a control is invalid and either touched or when a submit attempt has occurred. This avoids displaying errors before the user has had a reasonable chance to interact, while still showing errors after an invalid submission.

### 20. How can a template show a specific validation error?

Use the control reference's **hasError** method or inspect its errors. For example, **emailControl.hasError('required')** checks the required rule. Keep the message clear and connect it to the field with **aria-describedby** when appropriate.

### 21. What are Angular's ng-valid and ng-touched CSS classes?

Angular adds state classes such as **ng-valid**, **ng-invalid**, **ng-pristine**, **ng-dirty**, **ng-untouched**, and **ng-touched** to controls. They can support visual styling. Do not use color alone; provide readable text for errors.

### 22. Does novalidate turn off Angular validation?

No. **novalidate** disables the browser's built-in interactive validation popups. Angular's validators and form state still operate. It is commonly used when the application displays its own validation messages.

### 23. Is client-side validation enough to protect submitted data?

No. Client-side checks help users correct mistakes, but browser code can be changed or bypassed. Validate input again on the server before trusting or storing it.

## Submission and reset

### 24. What is ngSubmit?

**ngSubmit** is the Angular form submission event. Bind it on the **form** element so the same handler supports the submit button and normal keyboard-based form submission.

### 25. Why is ngSubmit usually better than a button click handler?

A form has built-in submission behavior, including submitting from a field with Enter in common cases. Binding the main action to **(ngSubmit)** preserves that form behavior and keeps the submit path on the form itself.

### 26. Should the submit button be disabled while the form is invalid?

It can be disabled to communicate that the form is not ready, but the component method should still check **form.invalid** before saving. A disabled button is a user interface state, not a substitute for guarding the operation.

### 27. Why mark controls touched after an invalid submission?

It makes the relevant validation messages visible, including for fields the user skipped. This gives the user a clear explanation of what must be corrected.

### 28. What is the difference between changing the model and calling resetForm?

Changing the component object changes the bound values. **NgForm.resetForm(value)** also resets the form's interaction and submission state, such as dirty, touched, and submitted flags. Use it when the form should return to a clean initial state.

### 29. Should a clear button use type submit?

No. Use **type="button"** for a button that clears or resets data, then call the reset handler explicitly. A button inside a form that submits should use **type="submit"**.

## Accessibility and design choices

### 30. How should form labels be associated with inputs?

Give each input a stable **id** and point the visible label's **for** attribute to that ID. This provides a programmatic label and a larger clickable target.

### 31. How should help text and error text be connected?

Use **aria-describedby** to reference relevant help and error elements. Set **aria-invalid** when the field is in an invalid state that is being communicated. Make sure the referenced IDs exist when they are included.

### 32. Can two forms on the same page use template-driven forms?

Yes. Each regular form can have its own **NgForm** instance and separately registered controls. Give template references, input IDs, and labels distinct names where needed.

### 33. Can I nest one form inside another?

No. Nested HTML **form** elements are invalid HTML and lead to confusing submit behavior. Use **ngModelGroup** or another container element to group related fields.

### 34. Can one Angular application use template-driven and reactive forms?

Yes. Different forms in the same application can use different approaches. Avoid attaching template-driven and reactive form directives as competing models to the same control.

### 35. When should I choose reactive forms instead?

Consider reactive forms when the form has many dynamic controls, complex cross-field validation, reusable validator logic, explicit programmatic updates, or a large amount of form-specific testing. The form's actual complexity matters more than its raw field count.

### 36. How can I import FormsModule but opt a form out of NgForm?

Angular provides the **ngNoForm** attribute for a form that should not receive the template-driven **NgForm** directive. Use this only when the form intentionally should not participate in Angular's template-driven form model.

## Practice

1. Build a three-field form with a required name, a valid email, and a message length limit.
2. Add a form reference and a reference for each control.
3. Show errors only after the field is touched or a submit attempt occurs.
4. Submit through **ngSubmit** and keep the invalid-data guard in the component.
5. Add a reset button that clears values and returns the form to its pristine state.
6. Inspect the form's aggregate value and validity in the browser's developer tools.

For a complete implementation, continue to the [example](../examples/062-template-driven-forms-and-ngform.md).

## Official references

- [Template-driven forms](https://angular.dev/guide/forms/template-driven-forms)
- [Forms overview](https://angular.dev/guide/forms)
- [Validate form input](https://angular.dev/guide/forms/form-validation)
- [NgForm API](https://angular.dev/api/forms/NgForm)
- [NgModel API](https://angular.dev/api/forms/NgModel)
- [NgModelGroup API](https://angular.dev/api/forms/NgModelGroup)
