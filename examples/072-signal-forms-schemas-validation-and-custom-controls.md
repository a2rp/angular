# Example: Signal Forms: schemas, validation, and custom controls

Build a small registration form that demonstrates:

- Built-in rules in a Signal Forms schema.
- A custom cross-field password confirmation rule.
- Error messages tied to their controls.
- Submission through the Signal Forms `submit()` helper.
- A reusable custom rating select that implements `FormValueControl<number>`.

This is a local learning example. Its submit action stores a short confirmation message in component state; it does not send credentials or personal data to a server.

## 1. Create a custom rating control

Create `rating-select.component.ts`:

~~~ts
import {Component, input, model, output} from '@angular/core';
import {
  FormValueControl,
  ValidationError,
  WithOptionalFieldTree,
} from '@angular/forms/signals';

@Component({
  selector: 'app-rating-select',
  template: `
    <div class="field">
      <label [for]="id()">{{ label() }}</label>

      <select
        [id]="id()"
        [value]="value()"
        [disabled]="disabled()"
        [required]="required()"
        [attr.aria-invalid]="
          touched() && invalid() ? 'true' : null
        "
        [attr.aria-describedby]="
          touched() && invalid() ? id() + '-errors' : null
        "
        (change)="updateValue($event)"
        (blur)="touch.emit()"
      >
        <option [value]="0">Choose a rating</option>
        @for (rating of ratings; track rating) {
          <option [value]="rating">
            {{ rating }} {{ rating === 1 ? 'star' : 'stars' }}
          </option>
        }
      </select>

      @if (touched() && invalid()) {
        <ul class="errors" [id]="id() + '-errors'">
          @for (error of errors(); track $index) {
            <li>{{ error.message ?? 'Choose a valid rating.' }}</li>
          }
        </ul>
      }
    </div>
  `,
  styles: [`
    :host {
      display: block;
    }

    .field {
      display: grid;
      gap: 0.35rem;
    }

    select {
      box-sizing: border-box;
      width: 100%;
      min-height: 2.75rem;
      padding: 0.55rem 0.7rem;
      border: 1px solid #687078;
      border-radius: 0.3rem;
      font: inherit;
    }

    select[aria-invalid='true'] {
      border-color: #a40000;
    }

    .errors {
      margin: 0;
      padding-inline-start: 1.25rem;
      color: #a40000;
    }

    select:focus-visible {
      outline: 3px solid #075db8;
      outline-offset: 2px;
    }
  `],
})
export class RatingSelectComponent implements FormValueControl<number> {
  readonly id = input.required<string>();
  readonly label = input.required<string>();

  // Required value model for a FormValueControl.
  readonly value = model<number>(0);

  // Optional state properties are synchronized by FormField.
  readonly disabled = input(false);
  readonly required = input(false);
  readonly touched = input(false);
  readonly invalid = input(false);
  readonly errors =
    input<readonly WithOptionalFieldTree<ValidationError>[]>([]);

  // Tell Signal Forms when the internal native select blurs.
  readonly touch = output<void>();

  readonly ratings = [1, 2, 3, 4, 5];

  updateValue(event: Event): void {
    const target = event.target;

    if (target instanceof HTMLSelectElement) {
      this.value.set(Number(target.value));
    }
  }
}
~~~

The select is still a native, labeled control; the component simply packages it as a reusable form control. Its `value` model signal is the contract required by `FormValueControl`. The optional `disabled`, `required`, `touched`, `invalid`, and `errors` inputs let `FormField` pass field state into the component.

The `touch` output reports blur from the internal select. If this component later contains multiple focusable elements, treat it as one widget and report touch only after focus leaves the whole widget.

## 2. Create the registration form

Create `registration.component.ts` and adjust the relative import path if your files are in a different folder:

~~~ts
import {Component, signal} from '@angular/core';
import {
  FormField,
  email,
  form,
  min,
  minLength,
  required,
  submit as submitForm,
  validate,
} from '@angular/forms/signals';
import {RatingSelectComponent} from './rating-select.component';

interface RegistrationModel {
  name: string;
  email: string;
  password: string;
  confirmPassword: string;
  rating: number;
  acceptsTerms: boolean;
}

@Component({
  selector: 'app-registration',
  imports: [FormField, RatingSelectComponent],
  template: `
    <main>
      <h1>Create an account</h1>
      <p>Complete the fields to see schema validation and a custom control.</p>

      <form novalidate (submit)="onSubmit($event)">
        <div class="field">
          <label for="display-name">Name</label>
          <input
            id="display-name"
            type="text"
            autocomplete="name"
            [formField]="registrationForm.name"
            [attr.aria-invalid]="
              registrationForm.name().touched() &&
              registrationForm.name().invalid()
                ? 'true'
                : null
            "
            [attr.aria-describedby]="
              registrationForm.name().touched() &&
              registrationForm.name().invalid()
                ? 'name-errors'
                : null
            "
          />
          @if (
            registrationForm.name().touched() &&
            registrationForm.name().invalid()
          ) {
            <ul class="errors" id="name-errors">
              @for (error of registrationForm.name().errors(); track $index) {
                <li>{{ error.message ?? 'Check your name.' }}</li>
              }
            </ul>
          }
        </div>

        <div class="field">
          <label for="email-address">Email</label>
          <input
            id="email-address"
            type="email"
            autocomplete="email"
            [formField]="registrationForm.email"
            [attr.aria-invalid]="
              registrationForm.email().touched() &&
              registrationForm.email().invalid()
                ? 'true'
                : null
            "
            [attr.aria-describedby]="
              registrationForm.email().touched() &&
              registrationForm.email().invalid()
                ? 'email-errors'
                : null
            "
          />
          @if (
            registrationForm.email().touched() &&
            registrationForm.email().invalid()
          ) {
            <ul class="errors" id="email-errors">
              @for (error of registrationForm.email().errors(); track $index) {
                <li>{{ error.message ?? 'Check your email address.' }}</li>
              }
            </ul>
          }
        </div>

        <div class="field">
          <label for="password">Password</label>
          <input
            id="password"
            type="password"
            autocomplete="new-password"
            [formField]="registrationForm.password"
            [attr.aria-invalid]="
              registrationForm.password().touched() &&
              registrationForm.password().invalid()
                ? 'true'
                : null
            "
            [attr.aria-describedby]="
              registrationForm.password().touched() &&
              registrationForm.password().invalid()
                ? 'password-errors'
                : null
            "
          />
          @if (
            registrationForm.password().touched() &&
            registrationForm.password().invalid()
          ) {
            <ul class="errors" id="password-errors">
              @for (
                error of registrationForm.password().errors();
                track $index
              ) {
                <li>{{ error.message ?? 'Check your password.' }}</li>
              }
            </ul>
          }
        </div>

        <div class="field">
          <label for="confirm-password">Confirm password</label>
          <input
            id="confirm-password"
            type="password"
            autocomplete="new-password"
            [formField]="registrationForm.confirmPassword"
            [attr.aria-invalid]="
              registrationForm.confirmPassword().touched() &&
              registrationForm.confirmPassword().invalid()
                ? 'true'
                : null
            "
            [attr.aria-describedby]="
              registrationForm.confirmPassword().touched() &&
              registrationForm.confirmPassword().invalid()
                ? 'confirm-password-errors'
                : null
            "
          />
          @if (
            registrationForm.confirmPassword().touched() &&
            registrationForm.confirmPassword().invalid()
          ) {
            <ul class="errors" id="confirm-password-errors">
              @for (
                error of registrationForm.confirmPassword().errors();
                track $index
              ) {
                <li>{{ error.message ?? 'Check the confirmation.' }}</li>
              }
            </ul>
          }
        </div>

        <app-rating-select
          id="experience-rating"
          label="Rate this learning experience"
          [formField]="registrationForm.rating"
        />

        <div class="checkbox-field">
          <input
            id="accept-terms"
            type="checkbox"
            [formField]="registrationForm.acceptsTerms"
            [attr.aria-invalid]="
              registrationForm.acceptsTerms().touched() &&
              registrationForm.acceptsTerms().invalid()
                ? 'true'
                : null
            "
            [attr.aria-describedby]="
              registrationForm.acceptsTerms().touched() &&
              registrationForm.acceptsTerms().invalid()
                ? 'terms-errors'
                : null
            "
          />
          <label for="accept-terms">I accept the terms</label>
        </div>
        @if (
          registrationForm.acceptsTerms().touched() &&
          registrationForm.acceptsTerms().invalid()
        ) {
          <ul class="errors" id="terms-errors">
            @for (
              error of registrationForm.acceptsTerms().errors();
              track $index
            ) {
              <li>{{ error.message ?? 'You must accept the terms.' }}</li>
            }
          </ul>
        }

        <button
          type="submit"
          [disabled]="registrationForm().submitting()"
        >
          @if (registrationForm().submitting()) {
            Saving...
          } @else {
            Create account
          }
        </button>
      </form>

      <p role="status" aria-live="polite">{{ statusMessage }}</p>
    </main>
  `,
  styles: [`
    :host {
      display: block;
      color: #202124;
      font: 1rem/1.5 system-ui, sans-serif;
    }

    main {
      box-sizing: border-box;
      width: min(100% - 2rem, 38rem);
      margin: 2rem auto;
    }

    form {
      display: grid;
      gap: 1rem;
    }

    .field {
      display: grid;
      gap: 0.35rem;
    }

    input:not([type='checkbox']) {
      box-sizing: border-box;
      width: 100%;
      min-height: 2.75rem;
      padding: 0.55rem 0.7rem;
      border: 1px solid #687078;
      border-radius: 0.3rem;
      font: inherit;
    }

    .checkbox-field {
      display: flex;
      align-items: center;
      gap: 0.6rem;
    }

    .checkbox-field input {
      width: 1.2rem;
      height: 1.2rem;
    }

    .errors {
      margin: 0;
      padding-inline-start: 1.25rem;
      color: #a40000;
    }

    button {
      min-height: 2.75rem;
      width: fit-content;
      padding: 0.55rem 0.9rem;
      border: 1px solid #39434d;
      border-radius: 0.3rem;
      background: #f5f7f8;
      color: inherit;
      font: inherit;
      cursor: pointer;
    }

    button:focus-visible,
    input:focus-visible {
      outline: 3px solid #075db8;
      outline-offset: 2px;
    }
  `],
})
export class RegistrationComponent {
  readonly registrationModel = signal<RegistrationModel>({
    name: '',
    email: '',
    password: '',
    confirmPassword: '',
    rating: 0,
    acceptsTerms: false,
  });

  readonly registrationForm = form(
    this.registrationModel,
    (path) => {
      required(path.name, {message: 'Enter your name.'});
      minLength(path.name, 2, {
        message: 'Name must contain at least 2 characters.',
      });

      required(path.email, {message: 'Enter your email address.'});
      email(path.email, {message: 'Enter a valid email address.'});

      required(path.password, {message: 'Enter a password.'});
      minLength(path.password, 12, {
        message: 'Use at least 12 characters.',
      });

      validate(path.confirmPassword, ({value, valueOf}) => {
        if (value() !== valueOf(path.password)) {
          return {
            kind: 'passwordMismatch',
            message: 'The passwords do not match.',
          };
        }

        return null;
      });

      // required(false) catches an unchecked consent box.
      required(path.acceptsTerms, {
        message: 'Accept the terms to continue.',
      });

      // The model starts at 0 as the custom select's "choose" option.
      // required() considers 0 non-empty, so min() enforces a real rating.
      required(path.rating, {message: 'Choose a rating.'});
      min(path.rating, 1, {message: 'Choose a rating from 1 to 5.'});
    },
  );

  statusMessage = '';

  async onSubmit(event: SubmitEvent): Promise<void> {
    event.preventDefault();

    const succeeded = await submitForm(
      this.registrationForm,
      async (field) => {
        // Keep this example local. Do not log or transmit the password.
        const values = field().value();
        this.statusMessage =
          'Form is valid. Registration data was not sent to a server.';
        void values;
      },
    );

    if (!succeeded) {
      this.statusMessage = 'Review the highlighted fields and try again.';
    }
  }
}
~~~

The model contains an initial value for every field. The schema adds built-in rules and one cross-field rule. The rating's initial value is `0` to represent the placeholder option; because `required()` treats zero as non-empty, the `min()` rule is what rejects that placeholder.

The custom rating selector exposes a `value` model and implements `FormValueControl<number>`. When the parent binds `[formField]`, Signal Forms synchronizes the rating value and passes the supported state inputs. The component displays its own error messages using those inputs.

## 3. Render the registration component

Import `RegistrationComponent` into your standalone root component and render `<app-registration />`. For example:

~~~ts
import {Component} from '@angular/core';
import {RegistrationComponent} from './registration.component';

@Component({
  selector: 'app-root',
  imports: [RegistrationComponent],
  template: '<app-registration />',
})
export class App {}
~~~

Adjust the import path and root component class to match the Angular CLI files in your app.

## 4. Try the form

1. Run the app with `ng serve`.
2. Submit the empty form. The submit helper marks interactive fields as touched, validation blocks the action, and field errors appear.
3. Enter a short name, malformed email, or password shorter than 12 characters to see the built-in validators.
4. Enter different password and confirmation values to see the custom cross-field error.
5. Leave the rating on **Choose a rating** to see why `min(path.rating, 1)` is needed.
6. Toggle the terms checkbox and select a rating.
7. Submit valid values. The local status message confirms validation; no request is sent.

The example uses a small custom control but leaves validation in the parent schema. The same `FormValueControl` pattern can wrap a date picker, token selector, or other single-value widget. For a boolean switch, implement `FormCheckboxControl` and provide a `checked` model instead.

## What to notice

- The schema callback maps validators to typed paths.
- Validation recalculates as model values change.
- The custom rule reads another field through `valueOf()`.
- Error messages come from field `errors()` and appear after interaction.
- The custom control implements the value-control contract; it does not validate itself.
- `submit()` marks fields touched and runs its action only when validation allows submission.
- Client-side validation improves interaction but does not replace server-side validation.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="071-signal-forms-models-fields-and-form-setup.md">071 - Signal Forms: models, fields, and form setup</a></td><td><a href="../lessons/072-signal-forms-schemas-validation-and-custom-controls.md">Lesson</a></td><td><a href="../qnas/072-signal-forms-schemas-validation-and-custom-controls.md">Q&amp;A</a></td><td><a href="073-configure-the-angular-router-in-a-standalone-app.md">073 - Configure the Angular Router in a standalone app</a></td>
</tr>
  </tbody>
</table>
