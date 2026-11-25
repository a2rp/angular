# Q&A: Choose template-driven, reactive, or signal forms

These answers help choose among Angular's template-driven forms, reactive forms, and Signal Forms. See the [lesson](../lessons/061-choose-template-driven-reactive-or-signal-forms.md) for the full comparison and the [example](../examples/061-choose-template-driven-reactive-or-signal-forms.md) for three versions of the same contact form.

## Core concepts

### 1. What form approaches does Angular provide?

Angular provides template-driven forms, reactive forms, and Signal Forms. All can collect and validate input, track form state, and submit values. Their models and APIs differ.

### 2. What is the main difference between the approaches?

It is where the form model and rules live. Template-driven forms rely mainly on template directives, reactive forms define a control tree in TypeScript, and Signal Forms connect a signal model to a schema-derived field tree.

### 3. Is one approach always the best choice?

No. Choose based on form complexity, the project's existing patterns, version support, and what the team can maintain. A newer API is not automatically a better fit for every form.

### 4. Can all three approaches validate user input?

Yes. They expose validation through different mechanisms: directives and control state, validator functions on reactive controls, or schema rules and field-state signals. Client-side validation is feedback, not a replacement for server validation.

### 5. Can one Angular application use different form approaches?

Yes. Different forms in one application can use different approaches. Keep each form's model ownership clear and avoid binding the same controls to competing form models.

## Template-driven forms

### 6. What does a template-driven form use?

It commonly uses **FormsModule**, **[(ngModel)]**, **NgForm**, and **ngSubmit**. The template directives create and manage much of the form control structure.

### 7. Where is the data model for a template-driven form?

The component typically owns a plain object or properties. **[(ngModel)]** synchronizes a control with those properties.

### 8. Why does an ngModel control need a name inside a form?

Angular uses the control's **name** to register it with its parent **NgForm**. Each registered control should have a unique name.

### 9. When is template-driven a good choice?

It fits a small, mostly static form with straightforward rules, such as a basic contact form or signup form. It is also suitable when extending an existing template-driven feature.

### 10. Can a template-driven form handle validation?

Yes. HTML validation attributes and Angular directives can provide common rules, and custom validators are possible. The tradeoff is that much of the form configuration is expressed in the template.

### 11. What is a common template-driven maintenance problem?

A large template can accumulate control registration, validation, conditional fields, error display, and submission behavior. When it becomes hard to follow, an explicit reactive model may be easier to maintain.

## Reactive forms

### 12. What does a reactive form use?

It uses **FormControl**, **FormGroup**, and **FormArray** from **@angular/forms**, with **ReactiveFormsModule** and directives such as **formGroup** and **formControlName**.

### 13. Where is the reactive form model created?

Usually in the component class or a form service. The template binds to that explicitly created control tree.

### 14. What makes reactive forms useful for complex forms?

The component can inspect and update controls directly, compose validators, group related fields, observe value and status streams, and add or remove controls at runtime.

### 15. What is a FormArray useful for?

**FormArray** represents a collection of controls whose number can change, such as repeated phone numbers, addresses, or line items.

### 16. What is the benefit of typed reactive forms?

TypeScript can check control names and form value shapes during development. This catches many structural mistakes, but runtime input still needs validation.

### 17. When are reactive forms a good choice?

Choose them for dynamic forms, cross-field rules, reusable form logic, workflows with detailed state handling, or projects that already use reactive forms consistently.

### 18. Are reactive forms only for large forms?

No. A small form may use reactive forms when explicit control, testability, or consistency with the rest of the application matters.

### 19. What is the difference between setValue() and patchValue()?

**setValue()** expects a value matching the entire form structure. **patchValue()** updates only supplied parts of the structure. Use **setValue()** when a strict shape check is useful and **patchValue()** for intentional partial updates.

### 20. What is the difference between value and getRawValue()?

A form group's **value** omits disabled controls in its aggregate value. **getRawValue()** includes disabled controls. Decide deliberately which fields should be sent to an API.

## Signal Forms

### 21. What is the source of truth in Signal Forms?

A writable signal model is the source of truth. The **form()** function creates a field tree linked to that model.

### 22. How are fields connected to the template?

Use **FormField** and bind a control with **[formField]** to the corresponding field in the form tree.

### 23. How are Signal Forms validators configured?

Pass a schema function to **form()** and apply rules such as **required()**, **email()**, **minLength()**, or **max()** to the relevant schema paths.

### 24. How do you read Signal Forms field state?

Call the field to get its state, then read signals such as **invalid()**, **valid()**, **touched()**, **pending()**, or **errors()**.

### 25. How does Signal Forms handle submission?

Use the **submit()** API directly or configure a submission action and bind **FormRoot**. Its submission flow marks interactive fields touched, checks validation, and runs the action when the form is valid.

### 26. How can Signal Forms prevent duplicate submission?

The form exposes a **submitting()** state while its action runs. Bind that state to the submit button's **disabled** property or use the API's concurrent-submission behavior.

### 27. Which Angular versions support stable Signal Forms?

Current Angular documentation marks Signal Forms stable starting with Angular 22. Verify support for the application's exact Angular version before adopting them.

### 28. Should an existing Angular 20 or 21 application use Signal Forms?

Do not assume the stable v22 API is available there. Check the documentation and compatibility for the exact version. Template-driven or reactive forms are established choices for versions before Signal Forms became stable.

### 29. Are Signal Forms automatically the best choice for a signal-based app?

Not automatically. They may fit a new signal-oriented form, but consider version support, team experience, required integrations, and the maturity expected by the project.

## Choosing and migrating

### 30. Which approach should I use for a small login form?

Any of the three can work. Template-driven forms are often the least setup for a simple form. Reactive forms may fit better if the rest of the app uses them or if explicit state control is needed. Signal Forms may fit a supported signal-based project.

### 31. Which approach is best for a multi-step wizard?

Reactive forms are a strong established choice when steps have dynamic controls, shared validation, or complex state. Signal Forms may also fit on supported versions. Choose based on the workflow and project patterns rather than a rule that every wizard must use one API.

### 32. Which approach is best for server-defined fields?

Reactive forms and Signal Forms make explicit or dynamic model construction easier to manage. The exact choice depends on the API schema, validation requirements, and supported Angular version.

### 33. Can a typed form guarantee that submitted data is valid?

No. Types help the application code during compilation. Users, browser extensions, stale clients, or external callers can still send invalid data, so validate on the server.

### 34. Should I mix template-driven and reactive APIs on the same form?

Usually no. Multiple form models for the same controls create unclear ownership and synchronization. Choose one approach for each form.

### 35. Can I migrate one form at a time?

Yes. Forms are often independent features, so migrating incrementally lets the team evaluate the new approach without rewriting the entire application.

### 36. Should I rewrite a working form just to use a newer API?

Only when there is a concrete benefit, such as improved maintainability, required capabilities, or consistency with a new feature. A rewrite adds risk and should solve a real problem.

## Validation, accessibility, and tests

### 37. Does client-side validation make an API secure?

No. Client-side checks can be bypassed. The server must validate data and enforce authorization and business rules.

### 38. What makes form errors accessible?

Associate each control with a visible label, describe errors in text, connect relevant descriptions to the input, and make errors discoverable after submit. Do not communicate validity through color alone.

### 39. Should an invalid submit button always be disabled?

Not necessarily. Disabling it can make it harder for users to discover why a form cannot submit. Another pattern is to allow submit, mark fields touched, and move focus to or announce the first error. Follow the product's interaction and accessibility requirements.

### 40. Which approach is easiest to test?

Reactive form models can be tested directly without rendering the template for every control and validator check. Template-driven forms depend more on directive and view behavior. Signal Forms expose model and field state through signals. In every case, also verify the rendered user interaction.

### 41. Does reactive forms mean the whole application must use RxJS?

No. Reactive forms expose Observable streams such as **valueChanges**, but the rest of the application can use signals or other patterns. The word “reactive” here describes the form model and its state updates.

### 42. Are HTML required and type=email enough validation?

They provide useful browser and Angular feedback, but they may not cover the product's full rules. Add appropriate Angular validators and server-side checks. A browser's email-format check also does not prove that an address exists or belongs to the user.

### 43. What should I compare when prototyping two approaches?

Implement one representative slice with realistic validation, error display, submission, and a test. Compare the clarity of the model, template, state transitions, and team workflow rather than counting lines in a tiny example.

## References

- [Angular forms overview](https://angular.dev/guide/forms)
- [Angular template-driven forms](https://angular.dev/guide/forms/template-driven-forms)
- [Angular reactive forms](https://angular.dev/guide/forms/reactive-forms)
- [Strictly typed reactive forms](https://angular.dev/guide/forms/typed-forms)
- [Signal Forms overview](https://angular.dev/guide/forms/signals/overview)
- [Signal Forms comparison](https://angular.dev/guide/forms/signals/comparison)
- [Signal Forms submission](https://angular.dev/guide/forms/signals/form-submission)

## Continue learning

- Previous: [Error handling, cleanup, and takeUntilDestroyed](../lessons/060-error-handling-cleanup-and-takeuntildestroyed.md)
- Lesson: [Choose template-driven, reactive, or signal forms](../lessons/061-choose-template-driven-reactive-or-signal-forms.md)
- Example: [Choose template-driven, reactive, or signal forms](../examples/061-choose-template-driven-reactive-or-signal-forms.md)
- Next: [Template-driven forms and ngForm](../lessons/062-template-driven-forms-and-ngform.md)
