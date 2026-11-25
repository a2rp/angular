# Reactive forms with FormControl and FormGroup

Reactive forms define the form model explicitly in TypeScript. The component creates each control and groups related controls before the template binds to them. This gives the application a direct, synchronous model for reading values, checking validation state, and updating fields.

This lesson builds a profile form with two controls. It introduces **FormControl**, **FormGroup**, **ReactiveFormsModule**, built-in validators, submission, and programmatic updates.

## Learning outcomes

After this lesson, you should be able to:

- Explain the model-first structure of a reactive form.
- Create individual **FormControl** instances and collect them in a **FormGroup**.
- Import **ReactiveFormsModule** into a standalone component.
- Bind a form group and its controls to the template.
- Read values and validation state from the form model.
- Submit, update, and reset the form programmatically.
- Distinguish **setValue**, **patchValue**, and **reset**.
- Recognize when a fixed **FormGroup** is not suitable for dynamic collections.

## The reactive forms model

A reactive form starts with a control tree in TypeScript:

~~~text
FormGroup
├── firstName: FormControl
└── email: FormControl
~~~

A **FormControl** tracks one value and its validation and interaction state. A **FormGroup** tracks a fixed set of named child controls and aggregates their values and state. Form controls also track whether they are valid, dirty, touched, pending, or disabled.

The form model is explicit: the component creates the controls, and the template binds to those existing controls. User input updates the model, and programmatic changes to the model update the view.

Use a **FormGroup** when the set of controls is known. A collection whose number of controls changes at runtime is represented with **FormArray**, covered later in this course.

## Import ReactiveFormsModule

A standalone component imports the reactive form directives from **@angular/forms**:

~~~ts
import { Component } from '@angular/core';
import {
  FormControl,
  FormGroup,
  ReactiveFormsModule,
  Validators,
} from '@angular/forms';

@Component({
  selector: 'app-profile-editor',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: './profile-editor.component.html',
  styleUrl: './profile-editor.component.css',
})
export class ProfileEditorComponent {
  profileForm = new FormGroup({
    firstName: new FormControl('', [
      Validators.required,
      Validators.minLength(2),
      Validators.maxLength(60),
    ]),
    email: new FormControl('', [
      Validators.required,
      Validators.email,
    ]),
  });

  savedProfile: { firstName: string | null; email: string | null } | null = null;

  save(): void {
    if (this.profileForm.invalid || this.profileForm.pending) {
      this.profileForm.markAllAsTouched();
      return;
    }

    this.savedProfile = this.profileForm.getRawValue();
  }

  reset(): void {
    this.profileForm.reset({
      firstName: '',
      email: '',
    });
    this.savedProfile = null;
  }
}
~~~

In an NgModule-based application, import **ReactiveFormsModule** into the NgModule that declares the component instead.

This introductory example uses regular **FormControl** instances. By default, a control can reset to **null**; the next lesson covers typed forms and non-nullable controls in more depth.

## Bind the model to the template

The template connects the group to a **form** element with **[formGroup]**. Each input uses **formControlName** with the matching key from the group:

~~~html
<form [formGroup]="profileForm" (ngSubmit)="save()" novalidate>
  <div class="field">
    <label for="first-name">First name</label>
    <input
      id="first-name"
      type="text"
      formControlName="firstName"
      autocomplete="given-name"
      aria-describedby="first-name-help"
      [attr.aria-invalid]="
        profileForm.controls.firstName.invalid &&
        profileForm.controls.firstName.touched
          ? 'true'
          : null
      "
    />
    <p id="first-name-help">Use between 2 and 60 characters.</p>

    @if (
      profileForm.controls.firstName.invalid &&
      profileForm.controls.firstName.touched
    ) {
      <p class="error">
        @if (profileForm.controls.firstName.hasError('required')) {
          Enter your first name.
        } @else if (profileForm.controls.firstName.hasError('minlength')) {
          Use at least two characters.
        } @else if (profileForm.controls.firstName.hasError('maxlength')) {
          Use no more than 60 characters.
        }
      </p>
    }
  </div>

  <div class="field">
    <label for="email">Email</label>
    <input
      id="email"
      type="email"
      formControlName="email"
      autocomplete="email"
      aria-describedby="email-help"
      [attr.aria-invalid]="
        profileForm.controls.email.invalid &&
        profileForm.controls.email.touched
          ? 'true'
          : null
      "
    />
    <p id="email-help">Enter an address where you can receive a reply.</p>

    @if (
      profileForm.controls.email.invalid &&
      profileForm.controls.email.touched
    ) {
      <p class="error">
        @if (profileForm.controls.email.hasError('required')) {
          Enter your email address.
        } @else if (profileForm.controls.email.hasError('email')) {
          Enter an email address in a valid format.
        }
      </p>
    }
  </div>

  <button type="submit">Save profile</button>
  <button type="button" (click)="reset()">Reset</button>
</form>

@if (savedProfile) {
  <p role="status">
    Saved profile for {{ savedProfile.firstName }} ({{ savedProfile.email }}).
  </p>
}
~~~

The keys must match exactly: **profileForm** contains **firstName** and **email**, so the template uses **formControlName="firstName"** and **formControlName="email"**. A misspelled or missing key means Angular cannot find the control to bind.

Labels are associated with their inputs by **for** and **id**. The message is shown after the user has touched an invalid control. A real application should use clear, field-specific feedback and validate submitted data again on the server.

## Understand FormControl and FormGroup state

A **FormControl** exposes its current value and status:

- **value** is the current value.
- **valid** and **invalid** report validation results.
- **errors** contains validation errors or **null**.
- **pristine** and **dirty** report whether the value has changed.
- **untouched** and **touched** report whether the user has blurred the control.
- **pending** reports that asynchronous validation is running.
- **disabled** and **enabled** report whether the control participates in normal validation and aggregate values.

A **FormGroup** exposes corresponding aggregate state. If a child changes, the group recalculates its value and status. For example:

~~~ts
const firstNameControl = this.profileForm.controls.firstName;

console.log(firstNameControl.value);
console.log(firstNameControl.valid);
console.log(this.profileForm.value);
console.log(this.profileForm.status);
~~~

The group value contains its enabled children, for example:

~~~ts
{
  firstName: 'Asha',
  email: 'asha@example.com'
}
~~~

A disabled control is excluded from its enabled parent's **value** and validation. Use **getRawValue()** when code intentionally needs the values of disabled controls too. Do not disable controls solely to bypass validation; treat disabled state as a deliberate part of the form behavior.

## Add and compose validators

Reactive forms attach validator functions when controls are created. Angular provides built-in validators through **Validators**:

~~~ts
new FormControl('', [
  Validators.required,
  Validators.minLength(2),
  Validators.maxLength(60),
]);
~~~

The array lets a control use more than one validator. Angular runs them as the value changes and records any failures in **control.errors**. The template can inspect a specific error using **hasError**:

~~~ts
if (this.profileForm.controls.firstName.hasError('required')) {
  // The required validator failed.
}
~~~

This lesson uses common synchronous validators. Custom, cross-field, and asynchronous validation are covered in later lessons. Client-side validators provide useful feedback but do not replace server-side validation.

## Submit the form

Bind the form's **ngSubmit** event to a component method. A submit button with **type="submit"** triggers the form, including the normal Enter-key submission behavior.

The component checks the model before accepting it:

~~~ts
save(): void {
  if (this.profileForm.invalid || this.profileForm.pending) {
    this.profileForm.markAllAsTouched();
    return;
  }

  const profile = this.profileForm.getRawValue();
  // Pass the valid value to a service or API.
}
~~~

**markAllAsTouched()** makes validation feedback visible for controls the user skipped. Keep the guard in the component even if the template disables a button; the component method owns the operation and should not rely on visual state.

The example uses a local preview to make the successful value visible. In an application, pass the value to a service, show request progress, and handle server responses.

## Update controls programmatically

Because the control tree is available in TypeScript, an application can update it directly:

~~~ts
this.profileForm.controls.firstName.setValue('Asha');
this.profileForm.controls.email.setValue('asha@example.com');
~~~

For the whole group, **setValue** requires a complete object matching all controls:

~~~ts
this.profileForm.setValue({
  firstName: 'Asha',
  email: 'asha@example.com',
});
~~~

**patchValue** accepts a partial object and updates only the supplied controls:

~~~ts
this.profileForm.patchValue({
  firstName: 'Asha',
});
~~~

Use **setValue** when the complete shape is available and you want Angular to check that all group keys are provided. Use **patchValue** when intentionally updating only part of the group. Neither method is a substitute for validation; validators still determine whether the resulting values are acceptable.

## Reset values and interaction state

**reset** updates the controls and returns the form's interaction state to pristine and untouched. Supply explicit defaults when the desired reset value is an empty string rather than **null**:

~~~ts
this.profileForm.reset({
  firstName: '',
  email: '',
});
~~~

If a **FormControl** was created with an empty string and default nullability, its reset behavior can still set the value to **null**. Non-nullable controls change that behavior; see [Typed reactive forms and non-nullable controls](064-typed-reactive-forms-and-non-nullable-controls.md).

## FormControl versus formControlName

Use **formControlName** to connect an input to a control that is already a member of a parent **FormGroup**:

~~~html
<form [formGroup]="profileForm">
  <input formControlName="firstName" />
</form>
~~~

Use **[formControl]** when binding an input directly to a **FormControl** instance, often when that control is not part of a group:

~~~ts
favoriteColor = new FormControl('');
~~~

~~~html
<label for="favorite-color">Favorite color</label>
<input id="favorite-color" [formControl]="favoriteColor" />
~~~

Both are reactive form bindings. Do not add **ngModel** to the same control as a reactive directive; choose one form model for each control.

## Common mistakes

- **Forgetting ReactiveFormsModule:** reactive form directives are not available until the owning standalone component or NgModule imports it.
- **Using the wrong directive for the model shape:** use **[formGroup]** on the group and **formControlName** for its named children.
- **Misspelling a control key:** each **formControlName** must match a key in the group.
- **Using a child outside the group:** a **formControlName** needs the matching parent **FormGroup** directive in the template tree.
- **Mixing ngModel and reactive directives on one input:** this creates competing control models and may produce warnings or unexpected state.
- **Assuming value includes disabled controls:** a group's **value** excludes disabled descendants; use **getRawValue()** only when disabled values are intentionally needed.
- **Calling setValue with a partial object:** use **patchValue** for an intentional partial update.
- **Assuming reset means empty string:** by default a control can reset to **null** unless you provide a reset value or use a non-nullable control.
- **Trusting browser validation as the data boundary:** validate values again on the server.

## When to use FormGroup

Use a **FormGroup** when the form has a known set of related controls that should share a value and status. Its explicit control tree is useful when code needs to set or inspect values, compose validators, and test form behavior directly.

For repeated controls whose count changes at runtime, use **FormArray**. For a simpler mostly static form whose structure is easiest to express in markup, template-driven forms may be a better fit. Compare them in [Choose template-driven, reactive, or signal forms](061-choose-template-driven-reactive-or-signal-forms.md).

## Next steps

- Continue with [Typed reactive forms and non-nullable controls](064-typed-reactive-forms-and-non-nullable-controls.md).
- Review the [Q&A](../qnas/063-reactive-forms-with-formcontrol-and-formgroup.md).
- Work through the [complete example](../examples/063-reactive-forms-with-formcontrol-and-formgroup.md).

## Official references

- [Reactive forms](https://angular.dev/guide/forms/reactive-forms)
- [FormControl API](https://angular.dev/api/forms/FormControl)
- [FormGroup API](https://angular.dev/api/forms/FormGroup)
- [ReactiveFormsModule API](https://angular.dev/api/forms/ReactiveFormsModule)
- [Validators API](https://angular.dev/api/forms/Validators)
