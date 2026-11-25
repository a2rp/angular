# Example: Synchronous validators and validation state

Build a registration form that uses Angular's built-in synchronous validators and one custom validator. The form demonstrates field-level errors, valid/invalid status, touched and dirty state, submit-time feedback, a checkbox rule, a nullable numeric control, and **updateOn: 'blur'**.

Use an Angular CLI application with standalone components and reactive forms. Replace the root component files or adapt the component to your application.

## 1. Create the form and validator

Replace the root component TypeScript file:

~~~ts
import { JsonPipe } from '@angular/common';
import { Component, inject } from '@angular/core';
import {
  AbstractControl,
  FormBuilder,
  FormControl,
  ReactiveFormsModule,
  ValidationErrors,
  ValidatorFn,
  Validators,
} from '@angular/forms';

interface RegistrationSnapshot {
  displayName: string;
  email: string;
  username: string;
  age: number | null;
  acceptedTerms: boolean;
}

const reservedUsernames = new Set(['admin', 'root', 'support']);

const reservedUsernameValidator: ValidatorFn = (
  control: AbstractControl,
): ValidationErrors | null => {
  const value = String(control.value ?? '').trim().toLocaleLowerCase();

  return reservedUsernames.has(value)
    ? { reservedUsername: { actualValue: control.value } }
    : null;
};

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [JsonPipe, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  private readonly fb = inject(FormBuilder).nonNullable;

  readonly registrationForm = this.fb.group({
    displayName: this.fb.control('', [
      Validators.required,
      Validators.minLength(2),
      Validators.maxLength(50),
    ]),
    email: this.fb.control('', {
      validators: [Validators.required, Validators.email],
      updateOn: 'blur',
    }),
    username: this.fb.control('', [
      Validators.required,
      Validators.minLength(3),
      Validators.maxLength(24),
      reservedUsernameValidator,
    ]),
    age: new FormControl<number | null>(null, [
      Validators.required,
      Validators.min(18),
      Validators.max(120),
    ]),
    acceptedTerms: this.fb.control(false, Validators.requiredTrue),
  });

  attemptedSubmit = false;
  savedRegistration: RegistrationSnapshot | null = null;

  get displayName(): FormControl<string> {
    return this.registrationForm.controls.displayName;
  }

  get email(): FormControl<string> {
    return this.registrationForm.controls.email;
  }

  get username(): FormControl<string> {
    return this.registrationForm.controls.username;
  }

  get age(): FormControl<number | null> {
    return this.registrationForm.controls.age;
  }

  get acceptedTerms(): FormControl<boolean> {
    return this.registrationForm.controls.acceptedTerms;
  }

  showErrors(control: AbstractControl): boolean {
    return control.invalid && (control.dirty || control.touched || this.attemptedSubmit);
  }

  save(): void {
    this.attemptedSubmit = true;
    this.savedRegistration = null;

    if (this.registrationForm.invalid || this.registrationForm.pending) {
      this.registrationForm.markAllAsTouched();
      return;
    }

    this.savedRegistration = this.registrationForm.getRawValue();
    this.registrationForm.markAsPristine();
  }

  loadSample(): void {
    this.registrationForm.setValue({
      displayName: 'Asha Rao',
      email: 'asha@example.com',
      username: 'asha-r',
      age: 28,
      acceptedTerms: true,
    });

    this.registrationForm.markAsPristine();
    this.registrationForm.markAsUntouched();
    this.attemptedSubmit = false;
    this.savedRegistration = null;
  }

  reset(): void {
    this.registrationForm.reset();
    this.attemptedSubmit = false;
    this.savedRegistration = null;
  }
}
~~~

The form builder creates the string and boolean controls as non-nullable. The explicitly constructed **age** control remains **number | null**, because an empty number input can be represented by **null**.

The custom validator checks whether the normalized username exactly matches one of the reserved names. It returns an error map under the **reservedUsername** key or **null**. The built-in **required** validator separately handles an empty username.

## 2. Bind controls and display errors

Replace the component template:

~~~html
<main class="page">
  <header class="page-header">
    <p class="eyebrow">Reactive form validation</p>
    <h1>Create an account</h1>
    <p>
      Enter valid account details. Errors appear after you edit or leave a
      field, or after you try to submit the form.
    </p>
  </header>

  <form
    [formGroup]="registrationForm"
    (ngSubmit)="save()"
    novalidate
  >
    <div class="field">
      <label for="display-name">
        Display name <span class="required">(required)</span>
      </label>
      <input
        id="display-name"
        type="text"
        formControlName="displayName"
        autocomplete="name"
        required
        aria-describedby="display-name-help display-name-error"
        [attr.aria-invalid]="showErrors(displayName) ? 'true' : null"
      />
      <p class="hint" id="display-name-help">
        Use 2 to 50 characters.
      </p>

      @if (showErrors(displayName)) {
        <div class="field-errors" id="display-name-error">
          @if (displayName.hasError('required')) {
            <p>Enter your display name.</p>
          }
          @if (displayName.hasError('minlength')) {
            <p>Use at least 2 characters.</p>
          }
          @if (displayName.hasError('maxlength')) {
            <p>Use no more than 50 characters.</p>
          }
        </div>
      }
    </div>

    <div class="field">
      <label for="email">
        Email address <span class="required">(required)</span>
      </label>
      <input
        id="email"
        type="email"
        formControlName="email"
        autocomplete="email"
        required
        aria-describedby="email-help email-error"
        [attr.aria-invalid]="showErrors(email) ? 'true' : null"
      />
      <p class="hint" id="email-help">
        Validation updates after you leave this field.
      </p>

      @if (showErrors(email)) {
        <div class="field-errors" id="email-error">
          @if (email.hasError('required')) {
            <p>Enter an email address.</p>
          }
          @if (email.hasError('email')) {
            <p>Enter an address in a valid email format.</p>
          }
        </div>
      }
    </div>

    <div class="field">
      <label for="username">
        Username <span class="required">(required)</span>
      </label>
      <input
        id="username"
        type="text"
        formControlName="username"
        autocomplete="username"
        required
        aria-describedby="username-help username-error"
        [attr.aria-invalid]="showErrors(username) ? 'true' : null"
      />
      <p class="hint" id="username-help">
        Use 3 to 24 characters. The names admin, root, and support are reserved.
      </p>

      @if (showErrors(username)) {
        <div class="field-errors" id="username-error">
          @if (username.hasError('required')) {
            <p>Enter a username.</p>
          }
          @if (username.hasError('minlength')) {
            <p>Use at least 3 characters.</p>
          }
          @if (username.hasError('maxlength')) {
            <p>Use no more than 24 characters.</p>
          }
          @if (username.hasError('reservedUsername')) {
            <p>Choose a different username; this one is reserved.</p>
          }
        </div>
      }
    </div>

    <div class="field">
      <label for="age">
        Age <span class="required">(required)</span>
      </label>
      <input
        id="age"
        type="number"
        formControlName="age"
        min="18"
        max="120"
        required
        aria-describedby="age-help age-error"
        [attr.aria-invalid]="showErrors(age) ? 'true' : null"
      />
      <p class="hint" id="age-help">
        Enter a number from 18 through 120.
      </p>

      @if (showErrors(age)) {
        <div class="field-errors" id="age-error">
          @if (age.hasError('required')) {
            <p>Enter your age.</p>
          }
          @if (age.hasError('min')) {
            <p>You must be at least 18 years old.</p>
          }
          @if (age.hasError('max')) {
            <p>Enter an age of 120 or less.</p>
          }
        </div>
      }
    </div>

    <div class="checkbox-field">
      <input
        id="accepted-terms"
        type="checkbox"
        formControlName="acceptedTerms"
        required
        aria-describedby="terms-error"
        [attr.aria-invalid]="showErrors(acceptedTerms) ? 'true' : null"
      />
      <label for="accepted-terms">
        I agree to the terms <span class="required">(required)</span>
      </label>
    </div>

    @if (showErrors(acceptedTerms)) {
      <p class="error" id="terms-error">
        Confirm that you agree to the terms.
      </p>
    }

    @if (attemptedSubmit && registrationForm.invalid) {
      <p class="error-summary" role="alert">
        The account was not created. Review the fields marked above.
      </p>
    }

    <div class="actions">
      <button class="button button-primary" type="submit">
        Create account
      </button>
      <button class="button button-secondary" type="button" (click)="loadSample()">
        Load valid example
      </button>
      <button class="button button-secondary" type="button" (click)="reset()">
        Reset
      </button>
    </div>
  </form>

  <section class="status-panel" aria-labelledby="status-title">
    <h2 id="status-title">Validation state</h2>
    <dl>
      <dt>Form status</dt>
      <dd>{{ registrationForm.status }}</dd>

      <dt>Form valid</dt>
      <dd>{{ registrationForm.valid }}</dd>

      <dt>Form invalid</dt>
      <dd>{{ registrationForm.invalid }}</dd>

      <dt>Form pending</dt>
      <dd>{{ registrationForm.pending }}</dd>

      <dt>Form dirty</dt>
      <dd>{{ registrationForm.dirty }}</dd>

      <dt>Form touched</dt>
      <dd>{{ registrationForm.touched }}</dd>
    </dl>

    <h3>Current form value</h3>
    <pre>{{ registrationForm.value | json }}</pre>
  </section>

  @if (savedRegistration; as registration) {
    <section class="success" aria-labelledby="saved-title" role="status">
      <h2 id="saved-title">Account details accepted</h2>
      <p>{{ registration.displayName }} ({{ registration.username }})</p>
      <p>{{ registration.email }}</p>
      <p>Age: {{ registration.age }}</p>
      <p>Terms accepted: {{ registration.acceptedTerms }}</p>
    </section>
  }
</main>
~~~

The submit handler allows an invalid submission attempt to reach Angular, then marks all controls touched and returns without saving. The form uses **novalidate** so browser-native constraint popups do not prevent the Angular handler from showing its own messages. The native **required**, **min**, and **max** attributes remain on the corresponding controls for semantics and browser affordances.

## 3. Add responsive styles

Replace the component stylesheet:

~~~css
:host {
  display: block;
  color: #172033;
  font-family: system-ui, sans-serif;
}

.page {
  box-sizing: border-box;
  width: min(100% - 2rem, 46rem);
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
h2,
h3 {
  line-height: 1.2;
}

h1 {
  margin: 0 0 0.75rem;
  font-size: clamp(2rem, 6vw, 3rem);
}

h2 {
  margin-top: 0;
  font-size: 1.3rem;
}

h3 {
  font-size: 1rem;
}

.page-header > p:last-child {
  max-width: 65ch;
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

input[type='text'],
input[type='email'],
input[type='number'] {
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
button:focus-visible {
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

.field-errors,
.error,
.error-summary {
  color: #b42318;
}

.field-errors {
  display: grid;
  gap: 0.25rem;
}

.field-errors p {
  margin: 0;
  font-size: 0.9rem;
  line-height: 1.5;
}

.error-summary {
  font-weight: 650;
}

.checkbox-field {
  display: flex;
  align-items: flex-start;
  gap: 0.65rem;
}

.checkbox-field input {
  width: 1.15rem;
  height: 1.15rem;
  margin: 0.15rem 0 0;
  accent-color: #253b80;
}

.checkbox-field label {
  line-height: 1.5;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
}

.button {
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

.button-primary {
  border-color: #176b3a;
  background: #176b3a;
}

.button-secondary {
  border-color: #98a2b3;
  background: #fff;
  color: #172033;
}

.status-panel,
.success {
  margin-top: 1.5rem;
  padding: 1.25rem;
  border: 1px solid #98a2b3;
  border-radius: 0.75rem;
}

dl {
  display: grid;
  grid-template-columns: minmax(9rem, 12rem) 1fr;
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

pre {
  max-width: 100%;
  padding: 1rem;
  overflow: auto;
  border-radius: 0.5rem;
  background: #172033;
  color: #f8fafc;
  line-height: 1.5;
}

.success {
  border-color: #75c79a;
  background: #f0fdf4;
}

.success h2 {
  margin-top: 0;
}

@media (max-width: 32rem) {
  .page {
    width: min(100% - 1rem, 46rem);
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

## 4. Explore the behavior

1. Submit the blank form. Angular marks the controls touched and shows the required-field messages.
2. Enter a one-character display name. The **minlength** message appears after the control is dirty or touched.
3. Enter an invalid email, then leave the field. Because its **updateOn** option is **'blur'**, Angular recalculates the email validation on blur.
4. Enter **admin**, **root**, or **support** as the username. The custom validator returns the **reservedUsername** error. Try another value such as **asha-r** to clear it.
5. Enter an age below 18 or above 120. The numeric **min** or **max** validator reports the corresponding error.
6. Leave the terms checkbox unchecked. **requiredTrue** reports an error because the value is not **true**.
7. Choose **Load valid example**. The form receives values programmatically and is marked pristine and untouched.
8. Submit the valid example. The accepted account details appear.
9. Choose **Reset**. Non-nullable string and checkbox controls return to their initial values; the explicitly nullable age control returns to **null**.
10. Compare **status**, **valid**, **invalid**, **dirty**, and **touched** in the state panel. They describe different aspects of the form.

## Further practice

- Add a required **minLength** validator to a mandatory field and an optional **minLength** validator to another field. Compare their empty-value behavior.
- Create a reusable validator factory that accepts a maximum word count.
- Add a **Validators.pattern()** rule for a carefully defined identifier format, and provide a readable explanation of that format.
- Change the email control's **updateOn** option to **'change'** and compare when the message appears.
- Replace the full-name messages with a priority display using **@else if**.
- Add a custom regular expression validator without **g** or **y** flags.
- Add an asynchronous check in the next chapter and display the **PENDING** state while it runs.
- Validate the same data on a server endpoint before persisting it.

## Related material

- Read the [lesson](../lessons/066-synchronous-validators-and-validation-state.md).
- Review the [Q&A](../qnas/066-synchronous-validators-and-validation-state.md).
- Continue with [Cross-field and asynchronous validation](../lessons/067-cross-field-and-asynchronous-validation.md).
- Revisit [FormArray and dynamic form structures](../lessons/065-formarray-and-dynamic-form-structures.md).

## Official references

- [Validate form input](https://angular.dev/guide/forms/form-validation)
- [Validators API](https://angular.dev/api/forms/Validators)
- [Reactive forms](https://angular.dev/guide/forms/reactive-forms)
- [AbstractControl API](https://angular.dev/api/forms/AbstractControl)
