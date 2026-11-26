# Choose template-driven, reactive, or signal forms

Angular supports three form approaches. Each can collect input, validate it, track interaction, and submit data. They differ in where the form model lives and how much control the component has over it.

- **Template-driven forms** create most form state through directives in the template.
- **Reactive forms** define an explicit control tree in TypeScript.
- **Signal Forms** connect a writable signal model to a field tree and schema.

There is no single best approach for every form. Choose one that fits the form's complexity, the application's existing patterns, and the Angular version you support.

## Learning outcomes

After this lesson, you should be able to:

- Explain the source of truth and data flow in each Angular form approach.
- Choose an approach for a small form, a complex form, or a new signal-based application.
- Recognize the modules and directives each approach uses.
- Compare validation, typing, dynamic fields, testing, and submission behavior.
- Avoid giving one form multiple competing state models.
- Check Signal Forms version support before using it.

## Start with the form's needs

Before choosing an API, list what the form must do:

- How many fields and sections does it have?
- Are fields added, removed, or changed based on user input or server data?
- Are there cross-field rules, asynchronous checks, or server-returned field errors?
- Does the application already use reactive forms, template-driven forms, or signals?
- Which Angular versions must the application support?
- How will the form be tested and maintained by the team?

The number of inputs alone is not decisive. A short form can have complex rules, and a long data-entry form can still be straightforward. Think about the amount of logic, reuse, dynamic behavior, and the need for explicit control.

## Compare the three approaches

| Approach | Main source of truth | Where the form model is defined | A good fit |
| --- | --- | --- | --- |
| Template-driven | Component properties connected by template directives | Mostly in the template through **ngModel** and **NgForm** | Small, straightforward forms and existing template-driven code |
| Reactive | **FormControl**, **FormGroup**, and **FormArray** instances | Explicitly in the component or a form service | Complex, dynamic, reusable, or heavily tested forms |
| Signal Forms | A writable signal model and its generated field tree | In the component, with a schema and field bindings | New signal-based apps on a version where Signal Forms is supported and stable |

Angular's current documentation marks Signal Forms stable starting with Angular 22. If the project supports an earlier Angular version, use the approach available to that version and verify the current compatibility documentation before planning an adoption.

## Template-driven forms

Template-driven forms use **FormsModule**, **[(ngModel)]**, and directives such as **NgForm**. Angular creates and updates the control model from the template. The component usually holds a plain data object.

~~~ts
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';
import type { NgForm } from '@angular/forms';

@Component({
  selector: 'app-contact-form',
  templateUrl: './contact-form.component.html',
  imports: [FormsModule],
})
export class ContactFormComponent {
  model = { name: '' };

  save(form: NgForm): void {
    if (form.invalid) {
      form.control.markAllAsTouched();
      return;
    }

    console.log(this.model);
  }
}
~~~

~~~html
<form #contactForm="ngForm" (ngSubmit)="save(contactForm)">
  <label for="contact-name">Name</label>
  <input
    id="contact-name"
    name="name"
    [(ngModel)]="model.name"
    required
  />

  <button type="submit">
    Send
  </button>
</form>
~~~

A control registered with a parent **NgForm** needs a unique **name**. The template declares much of the form's shape and validation. This keeps a simple form compact, but a large template can become the place where control structure, validation, conditional fields, and submission behavior are all intertwined.

Choose template-driven forms when the form is small and its rules are easy to understand in the template. They are also a reasonable choice when you are extending an existing template-driven form. Do not choose them merely to avoid writing a component model if the form's logic is already complicated.

## Reactive forms

Reactive forms use an explicit model made from **FormControl**, **FormGroup**, and, for dynamic collections, **FormArray**. The template binds to that model with **formGroup**, **formControlName**, and related directives.

~~~ts
import { Component } from '@angular/core';
import {
  FormControl,
  FormGroup,
  ReactiveFormsModule,
  Validators,
} from '@angular/forms';

@Component({
  selector: 'app-contact-form',
  templateUrl: './contact-form.component.html',
  imports: [ReactiveFormsModule],
})
export class ContactFormComponent {
  form = new FormGroup({
    name: new FormControl('', {
      nonNullable: true,
      validators: [Validators.required],
    }),
  });

  save(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    console.log(this.form.getRawValue());
  }
}
~~~

~~~html
<form [formGroup]="form" (ngSubmit)="save()">
  <label for="contact-name">Name</label>
  <input id="contact-name" formControlName="name" />

  <button type="submit">
    Send
  </button>
</form>
~~~

The control tree is directly available in TypeScript. You can set or patch values, subscribe to **valueChanges** and **statusChanges**, compose validators, build nested groups, and add controls to a **FormArray**. Typed reactive forms let TypeScript check the form's value and control names.

Choose reactive forms when the form is central to the feature, has dynamic fields or cross-field rules, needs reusable form logic, or benefits from direct control over validation and submission. They are also a good default for an established reactive-forms codebase.

## Signal Forms

Signal Forms use a writable signal as the data model. The **form()** function creates a field tree, a schema configures validation, and **FormField** binds controls in the template. **FormRoot** can connect submission to a form submission action.

~~~ts
import { Component, signal } from '@angular/core';
import {
  email,
  form,
  FormField,
  FormRoot,
  required,
} from '@angular/forms/signals';

@Component({
  selector: 'app-contact-form',
  templateUrl: './contact-form.component.html',
  imports: [FormField, FormRoot],
})
export class ContactFormComponent {
  model = signal({ name: '', email: '' });

  contactForm = form(this.model, (path) => {
    required(path.name);
    required(path.email);
    email(path.email);
  }, {
    submission: {
      action: async () => {
        console.log(this.model());
      },
    },
  });
}
~~~

~~~html
<form [formRoot]="contactForm">
  <label>
    Name
    <input [formField]="contactForm.name" />
  </label>

  <label>
    Email
    <input type="email" [formField]="contactForm.email" />
  </label>

  <button type="submit" [disabled]="contactForm().submitting()">
    Send
  </button>
</form>
~~~

Field state is read through signals such as **invalid()**, **touched()**, **pending()**, and **errors()**. A schema keeps validation rules in TypeScript and derives field types from the model. For submission behavior, configure the form's **submission.action** or call **submit()** directly; validation, touched state, and duplicate-submission handling are part of that API.

Choose Signal Forms when the application is signal-oriented, the project version supports the API, and the team wants the model-plus-schema approach. Check its status and the migration guidance for your Angular version before introducing it into a production application. A new API being stable in a current release does not mean every application can use it.

## How the choice affects common work

### Validation

- **Template-driven:** add validation attributes and directives in the template, then read control validity from **NgForm** or a control reference.
- **Reactive:** attach validator functions to controls or groups in TypeScript. Use group-level validation for relationships such as matching passwords.
- **Signal Forms:** configure schema validators for field paths and read validation state from field-state signals.

In all three approaches, client-side validation improves feedback but does not replace server-side validation. Treat submitted values as untrusted input.

### Type safety

- **Template-driven:** TypeScript checks the component's data properties, but the template-created form structure has less explicit type information.
- **Reactive:** typed controls and groups make the form structure available to the compiler.
- **Signal Forms:** field types are inferred from the signal model.

Do not confuse a typed form with validated data. Types help during development; users and remote systems can still provide invalid values at runtime.

### Dynamic forms

Reactive forms provide **FormArray** and explicit APIs for adding and removing controls. Signal Forms also support dynamic field trees and schema logic. Template-driven forms can render repeated controls, but complex dynamic structure is usually easier to reason about when the model is explicit.

### Submission

All approaches can handle the browser's submit event. Use a native **<form>** and a **type="submit"** button so keyboard users can submit with Enter. In Angular, bind the form's submit flow rather than using a click handler as the only path.

In many forms, leave the submit button available while the form is invalid so users can trigger error feedback. Disable it for a clear reason such as an in-progress submission, and also guard the submit handler.

Before saving:

1. Make validation errors visible after an invalid attempt.
2. Prevent duplicate submissions while a save is pending.
3. Send only the values required by the API.
4. Handle server validation errors and network failures.
5. Show a clear success or failure state.

Signal Forms' **submit()** flow marks interactive fields touched, checks validation, and runs the configured action when valid. Reactive and template-driven forms require you to implement the corresponding submission state and guard according to your needs.

### Testing

Reactive form models can be tested directly without rendering the template for every value and validator check. Template-driven forms rely more on Angular's template directives and view updates, so tests often exercise the rendered form. Signal Forms expose their model and field state through signals; test the schema rules and rendered interaction using the test patterns for the Angular version in use.

Choose tests that verify user-visible behavior as well as the form model. A form can be internally valid but still have unclear labels, inaccessible error messages, or a broken submit path.

## Can one application use more than one approach?

Yes. A codebase can contain different forms using different approaches. For example, a legacy settings screen may stay template-driven while a new wizard uses reactive forms. Keep each individual form's state ownership clear; avoid binding the same controls to multiple form models.

Migrate incrementally when there is a concrete benefit. A rewrite only to adopt a newer API adds risk and work without necessarily improving the form.

## A quick decision path

- **Simple, mostly static, template-local behavior?** Consider template-driven forms.
- **Dynamic controls, complex validation, reusable logic, or explicit testable state?** Consider reactive forms.
- **Signal-based design, supported Angular version, and a team ready to use its schema model?** Consider Signal Forms.
- **Existing project already uses one approach consistently?** Continue with it unless a real requirement justifies changing.
- **Still unsure?** Build a small representative slice, compare how validation, testing, and submission feel, and check version support before committing to the approach.

## Common mistakes

- **Picking by trend instead of requirements:** choose based on the form's logic, project patterns, and supported Angular versions.
- **Treating template-driven and reactive forms as different validation capabilities:** both can handle substantial validation; they differ in where the control model and rules are managed.
- **Using template-driven forms for a highly dynamic workflow by default:** the template can become difficult to maintain when it owns too much structure and behavior.
- **Assuming reactive forms are only for large forms:** a small form can use them when explicit control or consistency with the application matters.
- **Assuming Signal Forms are available in every Angular release:** verify the project version; current Angular documentation marks them stable starting with v22.
- **Mixing two form APIs on the same controls:** keep one source of truth for a given form.
- **Assuming client validators secure an API:** validate submitted values on the server too.
- **Using a button click as the only submit path:** keep a semantic form and submit button so keyboard submission works.
- **Rewriting a working form without a requirement:** migrate when there is a maintenance, capability, or consistency benefit.

## Continue learning

- Previous: [Error handling, cleanup, and takeUntilDestroyed](060-error-handling-cleanup-and-takeuntildestroyed.md)
- Q&A: [Choose template-driven, reactive, or signal forms](../qnas/061-choose-template-driven-reactive-or-signal-forms.md)
- Example: [Choose template-driven, reactive, or signal forms](../examples/061-choose-template-driven-reactive-or-signal-forms.md)
- Next: [Template-driven forms and ngForm](062-template-driven-forms-and-ngform.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="060-error-handling-cleanup-and-takeuntildestroyed.md">060 - Error handling, cleanup, and takeUntilDestroyed</a></td><td><a href="../qnas/061-choose-template-driven-reactive-or-signal-forms.md">Q&amp;A</a></td><td><a href="../examples/061-choose-template-driven-reactive-or-signal-forms.md">Example</a></td><td><a href="062-template-driven-forms-and-ngform.md">062 - Template-driven forms and NgForm</a></td>
</tr>
  </tbody>
</table>
