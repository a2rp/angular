# Example: Build a reactive profile form

This example builds a standalone Angular profile form using **FormControl**, **FormGroup**, **Validators**, and **ReactiveFormsModule**. It demonstrates the explicit form model, accessible validation feedback, submission, full and partial programmatic updates, and reset behavior.

## 1. Create a practice application

~~~sh
ng new reactive-profile-form --style=css
cd reactive-profile-form
ng serve --open
~~~

Keep the generated project structure. Replace the root component's TypeScript, template, and stylesheet with the corresponding code below. Recent CLI versions may use names such as **app.ts**, **app.html**, and **app.css** instead of **app.component.ts**, **app.component.html**, and **app.component.css**. Use the files that your project generated and keep their existing **templateUrl** and **styleUrl** paths consistent with those filenames.

## 2. Create the control tree

Replace the root component TypeScript file:

~~~ts
import { Component } from '@angular/core';
import { JsonPipe } from '@angular/common';
import {
  FormControl,
  FormGroup,
  ReactiveFormsModule,
  Validators,
} from '@angular/forms';

interface ProfileSnapshot {
  firstName: string | null;
  email: string | null;
}

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ReactiveFormsModule, JsonPipe],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
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

  attemptedSubmit = false;
  submittedProfile: ProfileSnapshot | null = null;

  save(): void {
    this.attemptedSubmit = true;
    this.submittedProfile = null;

    if (this.profileForm.invalid || this.profileForm.pending) {
      this.profileForm.markAllAsTouched();
      return;
    }

    this.submittedProfile = this.profileForm.getRawValue();
  }

  loadSample(): void {
    this.profileForm.setValue({
      firstName: 'Asha Rao',
      email: 'asha@example.com',
    });
    this.attemptedSubmit = false;
    this.submittedProfile = null;
  }

  updateEmailExample(): void {
    this.profileForm.patchValue({
      email: 'asha.rao@example.com',
    });
    this.submittedProfile = null;
  }

  reset(): void {
    this.profileForm.reset({
      firstName: '',
      email: '',
    });
    this.attemptedSubmit = false;
    this.submittedProfile = null;
  }
}
~~~

The form has a known set of two controls, so **FormGroup** is a suitable container. Each control starts with an empty string and has validators. The default control behavior is nullable on reset, so the reset method supplies empty strings explicitly. Typed non-nullable controls are covered in the next lesson.

## 3. Bind the form and show validation

Replace the root component HTML template:

~~~html
<main class="page">
  <header class="page-header">
    <p class="eyebrow">Reactive forms</p>
    <h1>Profile editor</h1>
    <p>
      Enter a name and a valid email address. The TypeScript form model owns
      the controls and validation rules.
    </p>
  </header>

  <form [formGroup]="profileForm" (ngSubmit)="save()" novalidate>
    <div class="field">
      <label for="first-name">
        First name <span class="required">(required)</span>
      </label>
      <input
        id="first-name"
        type="text"
        formControlName="firstName"
        autocomplete="given-name"
        [attr.aria-invalid]="
          profileForm.controls.firstName.invalid &&
          (profileForm.controls.firstName.touched || attemptedSubmit)
            ? 'true'
            : null
        "
        [attr.aria-describedby]="
          profileForm.controls.firstName.invalid &&
          (profileForm.controls.firstName.touched || attemptedSubmit)
            ? 'first-name-help first-name-error'
            : 'first-name-help'
        "
      />
      <p class="hint" id="first-name-help">
        Use between 2 and 60 characters.
      </p>

      @if (
        profileForm.controls.firstName.invalid &&
        (profileForm.controls.firstName.touched || attemptedSubmit)
      ) {
        <p class="error" id="first-name-error">
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
      <label for="email">
        Email <span class="required">(required)</span>
      </label>
      <input
        id="email"
        type="email"
        formControlName="email"
        autocomplete="email"
        [attr.aria-invalid]="
          profileForm.controls.email.invalid &&
          (profileForm.controls.email.touched || attemptedSubmit)
            ? 'true'
            : null
        "
        [attr.aria-describedby]="
          profileForm.controls.email.invalid &&
          (profileForm.controls.email.touched || attemptedSubmit)
            ? 'email-help email-error'
            : 'email-help'
        "
      />
      <p class="hint" id="email-help">
        Enter an address where you can receive a reply.
      </p>

      @if (
        profileForm.controls.email.invalid &&
        (profileForm.controls.email.touched || attemptedSubmit)
      ) {
        <p class="error" id="email-error">
          @if (profileForm.controls.email.hasError('required')) {
            Enter your email address.
          } @else if (profileForm.controls.email.hasError('email')) {
            Enter an email address in a valid format.
          }
        </p>
      }
    </div>

    @if (attemptedSubmit && profileForm.invalid) {
      <p class="error-summary" role="alert">
        The profile was not saved. Correct the fields marked above.
      </p>
    }

    @if (profileForm.pending) {
      <p role="status">Waiting for form validation to finish…</p>
    }

    <div class="actions">
      <button type="submit">Save profile</button>
      <button type="button" class="secondary" (click)="loadSample()">
        Load sample values
      </button>
      <button type="button" class="secondary" (click)="updateEmailExample()">
        Patch email
      </button>
      <button type="button" class="secondary" (click)="reset()">
        Reset
      </button>
    </div>
  </form>

  @if (submittedProfile; as profile) {
    <section class="success" aria-labelledby="success-title" role="status">
      <h2 id="success-title">Profile saved locally</h2>
      <p>This preview shows the value accepted by the component.</p>

      <dl>
        <dt>First name</dt>
        <dd>{{ profile.firstName }}</dd>

        <dt>Email</dt>
        <dd>{{ profile.email }}</dd>
      </dl>
    </section>
  }

  <details class="diagnostics">
    <summary>Inspect reactive form state</summary>
    <p>
      Group status: <strong>{{ profileForm.status }}</strong>
    </p>
    <p>
      Group valid: <strong>{{ profileForm.valid }}</strong>
    </p>
    <p>
      Group dirty: <strong>{{ profileForm.dirty }}</strong>
    </p>
    <p>
      Group touched: <strong>{{ profileForm.touched }}</strong>
    </p>
    <p>
      First-name errors:
      <strong>{{ profileForm.controls.firstName.errors | json }}</strong>
    </p>
  </details>
</main>
~~~

The component imports **JsonPipe** only for the diagnostic display. The **formGroup** directive binds the form element to the control tree. Each **formControlName** must match a group key exactly. The form has a single submission handler, while the sample and patch buttons call explicit component methods.

## 4. Add responsive styles

Replace the component stylesheet:

~~~css
:host {
  display: block;
  color: #172033;
  font-family: system-ui, sans-serif;
}

.page {
  box-sizing: border-box;
  width: min(100% - 2rem, 44rem);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  margin-bottom: 2rem;
}

.eyebrow {
  margin: 0 0 0.5rem;
  color: #4256a6;
  font-size: 0.875rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

h1,
h2 {
  line-height: 1.2;
}

h1 {
  margin: 0 0 0.75rem;
  font-size: clamp(2rem, 6vw, 3rem);
}

.page-header > p:last-child {
  max-width: 62ch;
  color: #475467;
  line-height: 1.6;
}

form {
  display: grid;
  gap: 1.25rem;
}

.field {
  display: grid;
  gap: 0.5rem;
}

label {
  font-weight: 650;
}

.required {
  color: #475467;
  font-size: 0.875rem;
  font-weight: 400;
}

input {
  box-sizing: border-box;
  width: 100%;
  min-height: 2.75rem;
  padding: 0.65rem 0.75rem;
  border: 1px solid #98a2b3;
  border-radius: 0.4rem;
  background: #fff;
  color: inherit;
  font: inherit;
}

input:focus-visible,
button:focus-visible,
summary:focus-visible {
  outline: 3px solid #84adff;
  outline-offset: 2px;
}

[aria-invalid='true'] {
  border-color: #b42318;
}

.hint,
.error,
.error-summary {
  margin: 0;
  font-size: 0.9rem;
  line-height: 1.5;
}

.hint {
  color: #475467;
}

.error,
.error-summary {
  color: #b42318;
}

.error-summary {
  font-weight: 650;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
}

button {
  min-height: 2.75rem;
  padding: 0.65rem 1rem;
  border: 1px solid #253b80;
  border-radius: 0.4rem;
  background: #253b80;
  color: #fff;
  cursor: pointer;
  font: inherit;
  font-weight: 650;
}

button.secondary {
  border-color: #98a2b3;
  background: #fff;
  color: #172033;
}

.success,
.diagnostics {
  margin-top: 1.5rem;
  padding: 1.25rem;
  border: 1px solid #98a2b3;
  border-radius: 0.75rem;
}

.success {
  border-color: #75c79a;
  background: #f0fdf4;
}

.success h2 {
  margin-top: 0;
}

dl {
  display: grid;
  grid-template-columns: minmax(7rem, 9rem) 1fr;
  gap: 0.5rem 1rem;
}

dt {
  font-weight: 700;
}

dd {
  min-width: 0;
  margin: 0;
  overflow-wrap: anywhere;
}

.diagnostics summary {
  cursor: pointer;
  font-weight: 700;
}

.diagnostics p {
  margin-block: 0.75rem 0;
}

@media (max-width: 32rem) {
  .page {
    width: min(100% - 1rem, 44rem);
    padding-block: 1.5rem;
  }

  dl {
    grid-template-columns: 1fr;
    gap: 0.2rem;
  }

  dd {
    margin-bottom: 0.6rem;
  }
}
~~~

The diagnostic panel uses **JsonPipe** only to make the current error object easy to inspect. The form model itself does not depend on the pipe.

## 5. Try the form

1. Submit the empty form. The component marks every control touched and shows the field errors.
2. Enter a one-character first name, then move focus away. The minimum-length message should appear.
3. Enter a malformed email address. The email validator should mark that control invalid.
4. Complete both fields with valid values and submit. The local profile preview appears.
5. Choose **Load sample values**. The complete group value is set programmatically.
6. Choose **Patch email**. Only the email control changes; the first name stays the same.
7. Choose **Reset**. The values return to empty strings and interaction state returns to pristine and untouched.

## Understand the control tree and aggregate value

The component creates this model:

~~~text
FormGroup
├── firstName: FormControl
└── email: FormControl
~~~

The **FormGroup** value is an object with keys matching the control names:

~~~json
{
  "firstName": "Asha Rao",
  "email": "asha@example.com"
}
~~~

The group also tracks aggregate status. If either control is invalid, the group is invalid. Changes to a child control update the parent group's value and status.

## setValue and patchValue in the example

The **Load sample values** action uses **setValue** with the complete group shape:

~~~ts
this.profileForm.setValue({
  firstName: 'Asha Rao',
  email: 'asha@example.com',
});
~~~

The **Patch email** action uses **patchValue** with only one field:

~~~ts
this.profileForm.patchValue({
  email: 'asha.rao@example.com',
});
~~~

Use **setValue** when you have all fields and want the supplied structure to match the group. Use **patchValue** when a partial update is intended.

## Understand reset behavior

The reset method supplies explicit empty-string values:

~~~ts
this.profileForm.reset({
  firstName: '',
  email: '',
});
~~~

That resets the controls' values and their interaction state. Without explicit values, a default nullable **FormControl** may reset to **null**. The following lesson shows how non-nullable controls provide a different default reset behavior.

## Accessibility and validation notes

- Each input has a visible label connected through **for** and **id**.
- Help and validation text are connected with **aria-describedby**.
- **aria-invalid** appears after a field error becomes relevant.
- The form uses text as well as color to communicate errors.
- The invalid-submit guard is in the component method, not only on the button.
- The profile preview is a local demonstration, not a real API request.
- Validate the data again on the server before saving it in a real application.

## Further practice

- Add a required last-name control and update the group, template, and reset values.
- Add a phone control with a custom validator.
- Disable the email control temporarily, then compare **profileForm.value** with **profileForm.getRawValue()**.
- Subscribe to **profileForm.valueChanges** and display a derived character count.
- Move the controls into a nested group and update the template bindings.
- Compare this form with the template-driven form in [Chapter 062](../lessons/062-template-driven-forms-and-ngform.md).

## Related material

- Read the [lesson](../lessons/063-reactive-forms-with-formcontrol-and-formgroup.md).
- Review the [Q&A](../qnas/063-reactive-forms-with-formcontrol-and-formgroup.md).
- Continue with [Typed reactive forms and non-nullable controls](../lessons/064-typed-reactive-forms-and-non-nullable-controls.md).

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="062-template-driven-forms-and-ngform.md">062 - Template-driven forms and NgForm</a></td><td><a href="../lessons/063-reactive-forms-with-formcontrol-and-formgroup.md">Lesson</a></td><td><a href="../qnas/063-reactive-forms-with-formcontrol-and-formgroup.md">Q&amp;A</a></td><td><a href="064-typed-reactive-forms-and-non-nullable-controls.md">064 - Typed reactive forms and non-nullable controls</a></td>
</tr>
  </tbody>
</table>
