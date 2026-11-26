# Example: Build a typed profile form

This example builds a standalone Angular profile form with a non-nullable **FormBuilder** and one explicitly nullable control. It shows how control types affect TypeScript, reset behavior, and the values returned by a **FormGroup**.

## 1. Create a practice application

~~~sh
ng new typed-profile-form --style=css
cd typed-profile-form
ng serve --open
~~~

Keep the generated project structure. Replace the root component's TypeScript, template, and stylesheet with the code below. Recent CLI versions may use files such as **app.ts**, **app.html**, and **app.css** instead of **app.component.ts**, **app.component.html**, and **app.component.css**. Use the generated filenames and keep the component's **templateUrl** and **styleUrl** paths consistent with them.

The examples assume strict TypeScript checking. Angular CLI's strict application setup enables strict null checks; if adapting an older project, confirm **strictNullChecks** is enabled so the compiler can check nullable values.

## 2. Create a typed form

Replace the root component TypeScript file:

~~~ts
import { Component, inject } from '@angular/core';
import {
  FormBuilder,
  FormControl,
  ReactiveFormsModule,
  Validators,
} from '@angular/forms';

interface ProfileSnapshot {
  displayName: string;
  email: string;
  nickname: string | null;
}

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  private readonly fb = inject(FormBuilder).nonNullable;

  readonly profileForm = this.fb.group({
    displayName: [
      '',
      [
        Validators.required,
        Validators.minLength(2),
        Validators.maxLength(60),
      ],
    ],
    email: ['', [Validators.required, Validators.email]],
    nickname: new FormControl<string | null>(null, [
      Validators.maxLength(24),
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

  loadExample(): void {
    this.profileForm.setValue({
      displayName: 'Asha Rao',
      email: 'asha@example.com',
      nickname: 'Ash',
    });
    this.attemptedSubmit = false;
    this.submittedProfile = null;
  }

  patchEmail(): void {
    this.profileForm.patchValue({
      email: 'asha.rao@example.com',
    });
    this.submittedProfile = null;
  }

  clearNickname(): void {
    this.profileForm.controls.nickname.reset(null);
    this.submittedProfile = null;
  }

  reset(): void {
    this.profileForm.reset();
    this.attemptedSubmit = false;
    this.submittedProfile = null;
  }
}
~~~

The **FormBuilder** creates **displayName** and **email** as non-nullable string controls. The explicitly constructed **nickname** control stays nullable because its type is **string | null**. The non-nullable builder does not change a control that was already constructed.

## 3. Bind controls and display errors

Replace the root component HTML template:

~~~html
<main class="page">
  <header class="page-header">
    <p class="eyebrow">Typed reactive forms</p>
    <h1>Profile settings</h1>
    <p>
      This form has two non-nullable strings and one nullable nickname.
      Submit it to inspect the values accepted by the typed form model.
    </p>
  </header>

  <form [formGroup]="profileForm" (ngSubmit)="save()" novalidate>
    <div class="field">
      <label for="display-name">
        Display name <span class="required">(required)</span>
      </label>
      <input
        id="display-name"
        type="text"
        formControlName="displayName"
        autocomplete="name"
        [attr.aria-invalid]="
          profileForm.controls.displayName.invalid &&
          (profileForm.controls.displayName.touched || attemptedSubmit)
            ? 'true'
            : null
        "
        [attr.aria-describedby]="
          profileForm.controls.displayName.invalid &&
          (profileForm.controls.displayName.touched || attemptedSubmit)
            ? 'display-name-help display-name-error'
            : 'display-name-help'
        "
      />
      <p class="hint" id="display-name-help">
        Use between 2 and 60 characters.
      </p>

      @if (
        profileForm.controls.displayName.invalid &&
        (profileForm.controls.displayName.touched || attemptedSubmit)
      ) {
        <p class="error" id="display-name-error">
          @if (profileForm.controls.displayName.hasError('required')) {
            Enter a display name.
          } @else if (profileForm.controls.displayName.hasError('minlength')) {
            Use at least two characters.
          } @else if (profileForm.controls.displayName.hasError('maxlength')) {
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
        Enter a valid address where you can receive a reply.
      </p>

      @if (
        profileForm.controls.email.invalid &&
        (profileForm.controls.email.touched || attemptedSubmit)
      ) {
        <p class="error" id="email-error">
          @if (profileForm.controls.email.hasError('required')) {
            Enter an email address.
          } @else if (profileForm.controls.email.hasError('email')) {
            Enter an email address in a valid format.
          }
        </p>
      }
    </div>

    <div class="field">
      <label for="nickname">Nickname <span class="optional">(optional)</span></label>
      <input
        id="nickname"
        type="text"
        formControlName="nickname"
        autocomplete="nickname"
        aria-describedby="nickname-help"
      />
      <p class="hint" id="nickname-help">
        This control allows null and has a 24-character limit.
      </p>

      @if (
        profileForm.controls.nickname.invalid &&
        profileForm.controls.nickname.touched
      ) {
        <p class="error">
          Keep the nickname to 24 characters or fewer.
        </p>
      }
    </div>

    @if (attemptedSubmit && profileForm.invalid) {
      <p class="error-summary" role="alert">
        The profile was not saved. Correct the fields marked above.
      </p>
    }

    <div class="actions">
      <button type="submit">Save profile</button>
      <button type="button" class="secondary" (click)="loadExample()">
        Load complete example
      </button>
      <button type="button" class="secondary" (click)="patchEmail()">
        Patch email only
      </button>
      <button type="button" class="secondary" (click)="clearNickname()">
        Set nickname to null
      </button>
      <button type="button" class="secondary" (click)="reset()">
        Reset all
      </button>
    </div>
  </form>

  <section class="diagnostics" aria-labelledby="diagnostics-title">
    <h2 id="diagnostics-title">Control types and current values</h2>
    <dl>
      <dt>Display name</dt>
      <dd>{{ profileForm.controls.displayName.value }}</dd>

      <dt>Email</dt>
      <dd>{{ profileForm.controls.email.value }}</dd>

      <dt>Nickname</dt>
      <dd>{{ profileForm.controls.nickname.value ?? 'null' }}</dd>

      <dt>Form status</dt>
      <dd>{{ profileForm.status }}</dd>

      <dt>Form value nickname</dt>
      <dd>{{ profileForm.value.nickname ?? 'not present or null' }}</dd>
    </dl>
    <p class="hint">
      The nickname input looks blank for both an empty string and null.
      This panel makes the control's actual value easier to inspect.
    </p>
  </section>

  @if (submittedProfile; as profile) {
    <section class="success" aria-labelledby="success-title" role="status">
      <h2 id="success-title">Valid profile snapshot</h2>
      <p>The accepted values are:</p>
      <dl>
        <dt>Display name</dt>
        <dd>{{ profile.displayName }}</dd>

        <dt>Email</dt>
        <dd>{{ profile.email }}</dd>

        <dt>Nickname</dt>
        <dd>{{ profile.nickname ?? 'null' }}</dd>
      </dl>
    </section>
  }
</main>
~~~

The controls are accessed through the typed **profileForm.controls** object. A typo such as **profileForm.controls.emali** is rejected by TypeScript. The nickname remains optional in meaning, but the form always has a nickname control; its value can be a string or **null**.

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
h2 {
  line-height: 1.2;
}

h1 {
  margin: 0 0 0.75rem;
  font-size: clamp(2rem, 6vw, 3rem);
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

.optional {
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

.diagnostics,
.success {
  margin-top: 1.5rem;
  padding: 1.25rem;
  border: 1px solid #98a2b3;
  border-radius: 0.75rem;
}

.diagnostics h2,
.success h2 {
  margin-top: 0;
  font-size: 1.2rem;
}

.success {
  border-color: #75c79a;
  background: #f0fdf4;
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

## 5. Explore nullability and reset behavior

1. Choose **Load complete example**. The full form shape is supplied and the nickname is a string.
2. Choose **Set nickname to null**. The nickname control now holds **null**.
3. Compare the display name, email, and nickname in the diagnostics panel.
4. Submit the valid form. The snapshot shows the typed values, including a nullable nickname.
5. Choose **Reset all**. The non-nullable display name and email return to their initial empty strings. The explicitly nullable nickname resets to **null**.
6. Type a value into the nickname field and then clear the field manually. The input value becomes an empty string; that is different from **null** even though both look blank.
7. Choose **Patch email only**. Only the email value changes.
8. Inspect the component code and compare **setValue** with **patchValue**.

A nullable control is not the same thing as an optional control. The nickname control always exists in the form tree, but its value can be **null**. An optional group control may not exist in the tree at all.

## Compare FormGroup.value with getRawValue()

When all controls are enabled, **profileForm.value** contains the current values. If a control is disabled, Angular excludes it from the enabled group's **value**, so TypeScript treats that key as possibly **undefined**.

~~~ts
this.profileForm.controls.email.disable();

const enabledValues = this.profileForm.value;
// enabledValues.email can be undefined.

const allValues = this.profileForm.getRawValue();
// allValues.email is still a string.
~~~

The example does not disable the email field during normal submission. The snippet demonstrates the type difference. A disabled control is also excluded from validation; re-enable it when it should be required again.

## See the inferred value types

The non-nullable builder creates these value types:

~~~ts
profileForm.controls.displayName.value; // string
profileForm.controls.email.value;       // string
~~~

The nickname was constructed explicitly as **FormControl<string | null>**, so:

~~~ts
profileForm.controls.nickname.value; // string | null
~~~

The group value has a separate optional-key concern because controls can be disabled:

~~~ts
profileForm.value.email;          // string | undefined
profileForm.getRawValue().email;  // string
~~~

These are different kinds of absence:

- **null** is an allowed value inside the nickname control.
- **undefined** can mean a disabled control is omitted from the group's aggregate **value**.
- **getRawValue()** includes disabled controls but preserves each control's own nullable type.

## Further practice

- Add a non-nullable boolean control for email preferences and reset it to **false**.
- Add a nullable field whose initial value is **null**, then assign it a string.
- Disable and re-enable a field and compare the group's **value** and **getRawValue()**.
- Add an optional control to a typed group and remove it through the group API.
- Replace the injected **FormBuilder** with direct **FormControl** construction using **{ nonNullable: true }**.
- Intentionally misspell a control name and observe the TypeScript error.
- Add a required validator and verify that the type stays **string** while validity changes independently.

## Related material

- Read the [lesson](../lessons/064-typed-reactive-forms-and-non-nullable-controls.md).
- Review the [Q&A](../qnas/064-typed-reactive-forms-and-non-nullable-controls.md).
- Continue with [FormArray and dynamic form structures](../lessons/065-formarray-and-dynamic-form-structures.md).

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="063-reactive-forms-with-formcontrol-and-formgroup.md">063 - Reactive forms with FormControl and FormGroup</a></td><td><a href="../lessons/064-typed-reactive-forms-and-non-nullable-controls.md">Lesson</a></td><td><a href="../qnas/064-typed-reactive-forms-and-non-nullable-controls.md">Q&amp;A</a></td><td><a href="065-formarray-and-dynamic-form-structures.md">065 - FormArray and dynamic form structures</a></td>
</tr>
  </tbody>
</table>
