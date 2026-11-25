# Example: Cross-field and asynchronous validation

Build a registration form with a password confirmation rule and a username availability check. The password rule belongs to the parent **FormGroup**. The username check uses a finite async validator and a local mock service that delays its answer so the **PENDING** state is visible.

The mock service recognizes **taken** as unavailable and **offline** as a simulated service failure. It makes no network request. Replace it with an API call in a real application, and enforce uniqueness on the server when the account is created.

Use an Angular CLI application with standalone components and reactive forms. Replace the root component files or adapt them to your application.

## 1. Create the form, validators, and mock service

Replace the root component TypeScript file:

~~~ts
import { JsonPipe } from '@angular/common';
import { Component, Injectable, inject } from '@angular/core';
import {
  AbstractControl,
  AsyncValidatorFn,
  FormBuilder,
  FormControl,
  ReactiveFormsModule,
  ValidationErrors,
  ValidatorFn,
  Validators,
} from '@angular/forms';
import {
  Observable,
  catchError,
  map,
  of,
  switchMap,
  take,
  throwError,
  timer,
} from 'rxjs';

export const passwordsMatchValidator: ValidatorFn = (
  control: AbstractControl,
): ValidationErrors | null => {
  const password = control.get('password');
  const confirmation = control.get('confirmPassword');

  if (!password || !confirmation || !password.value || !confirmation.value) {
    // Required validators on the child controls explain blank values.
    return null;
  }

  return password.value === confirmation.value
    ? null
    : { passwordMismatch: true };
};

@Injectable({ providedIn: 'root' })
class UsernameAvailabilityService {
  private readonly takenUsernames = new Set(['taken', 'already-used']);

  check(username: string): Observable<boolean> {
    const normalized = username.trim().toLocaleLowerCase();

    // Keep a delay so the pending indicator is easy to observe.
    return timer(700).pipe(
      switchMap(() => {
        if (normalized === 'offline') {
          return throwError(() => new Error('Simulated service failure'));
        }

        return of(!this.takenUsernames.has(normalized));
      }),
    );
  }
}

function usernameAvailabilityValidator(
  service: UsernameAvailabilityService,
): AsyncValidatorFn {
  return (control: AbstractControl) => {
    const username = String(control.value ?? '').trim();

    if (!username) {
      return of(null);
    }

    return service.check(username).pipe(
      take(1),
      map((available): ValidationErrors | null =>
        available ? null : { usernameTaken: true },
      ),
      catchError(() => of({ usernameCheckFailed: true })),
    );
  };
}

interface RegistrationValue {
  email: string;
  password: string;
  confirmPassword: string;
  username: string;
}

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [JsonPipe, ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  private readonly fb = inject(FormBuilder).nonNullable;
  private readonly usernameService = inject(UsernameAvailabilityService);

  readonly registrationForm = this.fb.group(
    {
      email: this.fb.control('', [
        Validators.required,
        Validators.email,
      ]),
      password: this.fb.control('', [
        Validators.required,
        Validators.minLength(12),
      ]),
      confirmPassword: this.fb.control('', Validators.required),
      username: this.fb.control('', {
        validators: [
          Validators.required,
          Validators.minLength(3),
          Validators.maxLength(24),
          Validators.pattern(/^[a-z0-9-]+$/),
        ],
        asyncValidators: [
          usernameAvailabilityValidator(this.usernameService),
        ],
        updateOn: 'blur',
      }),
    },
    {
      validators: passwordsMatchValidator,
    },
  );

  attemptedSubmit = false;
  savedRegistration: RegistrationValue | null = null;

  get email(): FormControl<string> {
    return this.registrationForm.controls.email;
  }

  get password(): FormControl<string> {
    return this.registrationForm.controls.password;
  }

  get confirmPassword(): FormControl<string> {
    return this.registrationForm.controls.confirmPassword;
  }

  get username(): FormControl<string> {
    return this.registrationForm.controls.username;
  }

  showErrors(control: AbstractControl): boolean {
    return control.invalid && (control.dirty || control.touched || this.attemptedSubmit);
  }

  showPasswordMismatch(): boolean {
    return (
      this.registrationForm.hasError('passwordMismatch') &&
      (
        this.registrationForm.dirty ||
        this.registrationForm.touched ||
        this.attemptedSubmit
      )
    );
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
      email: 'asha@example.com',
      password: 'correct-horse-42',
      confirmPassword: 'correct-horse-42',
      username: 'asha-r',
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

The password validator compares values but skips the comparison while either field is empty. Each child control's **required** validator handles that case. A mismatch error is attached to the form group.

The async validator maps the mock service's result to **null** or an error key. It catches the simulated service failure and returns a distinct **usernameCheckFailed** error. It does not silently mark an unchecked name available.

## 2. Bind the form and its validation states

Replace the component template:

~~~html
<main class="page">
  <header class="page-header">
    <p class="eyebrow">Reactive form validation</p>
    <h1>Create an account</h1>
    <p>
      Passwords must match. Usernames are checked after you leave the field.
    </p>
  </header>

  <form
    [formGroup]="registrationForm"
    (ngSubmit)="save()"
    novalidate
  >
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
        aria-describedby="email-help"
        [attr.aria-invalid]="showErrors(email) ? 'true' : null"
      />
      <p class="hint" id="email-help">Enter a valid email address.</p>

      @if (showErrors(email)) {
        <div class="field-errors" role="alert">
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
      <label for="password">
        Password <span class="required">(required)</span>
      </label>
      <input
        id="password"
        type="password"
        formControlName="password"
        autocomplete="new-password"
        required
        aria-describedby="password-help"
        [attr.aria-invalid]="showErrors(password) ? 'true' : null"
      />
      <p class="hint" id="password-help">
        Use at least 12 characters.
      </p>

      @if (showErrors(password)) {
        <div class="field-errors" role="alert">
          @if (password.hasError('required')) {
            <p>Enter a password.</p>
          }
          @if (password.hasError('minlength')) {
            <p>Use at least 12 characters.</p>
          }
        </div>
      }
    </div>

    <div class="field">
      <label for="confirm-password">
        Confirm password <span class="required">(required)</span>
      </label>
      <input
        id="confirm-password"
        type="password"
        formControlName="confirmPassword"
        autocomplete="new-password"
        required
        aria-describedby="confirm-password-help"
        [attr.aria-invalid]="
          showErrors(confirmPassword) || showPasswordMismatch()
            ? 'true'
            : null
        "
      />
      <p class="hint" id="confirm-password-help">
        Enter the same password again.
      </p>

      @if (showErrors(confirmPassword)) {
        <div class="field-errors" role="alert">
          @if (confirmPassword.hasError('required')) {
            <p>Confirm your password.</p>
          }
        </div>
      }

      @if (showPasswordMismatch()) {
        <p class="error" role="alert">
          The passwords do not match.
        </p>
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
        aria-describedby="username-help"
        [attr.aria-invalid]="showErrors(username) ? 'true' : null"
      />
      <p class="hint" id="username-help">
        Use 3 to 24 lowercase letters, numbers, or hyphens. Try “taken” or
        “offline” to explore the async states.
      </p>

      @if (username.pending) {
        <p class="status" role="status">
          Checking whether this username is available…
        </p>
      }

      @if (showErrors(username)) {
        <div class="field-errors" role="alert">
          @if (username.hasError('required')) {
            <p>Enter a username.</p>
          }
          @if (username.hasError('minlength')) {
            <p>Use at least 3 characters.</p>
          }
          @if (username.hasError('maxlength')) {
            <p>Use no more than 24 characters.</p>
          }
          @if (username.hasError('pattern')) {
            <p>Use lowercase letters, numbers, and hyphens only.</p>
          }
          @if (username.hasError('usernameTaken')) {
            <p>That username is already in use. Choose another one.</p>
          }
          @if (username.hasError('usernameCheckFailed')) {
            <p>The availability check failed. Edit the value and try again.</p>
          }
        </div>
      }
    </div>

    @if (registrationForm.pending) {
      <p class="status" role="status">
        A validation check is still running. Wait for it to finish, then submit.
      </p>
    }

    @if (attemptedSubmit && registrationForm.invalid) {
      <p class="error-summary" role="alert">
        The account was not created. Correct the fields marked above.
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
    <h2 id="status-title">Form state</h2>
    <dl>
      <dt>Status</dt>
      <dd>{{ registrationForm.status }}</dd>

      <dt>Valid</dt>
      <dd>{{ registrationForm.valid }}</dd>

      <dt>Invalid</dt>
      <dd>{{ registrationForm.invalid }}</dd>

      <dt>Pending</dt>
      <dd>{{ registrationForm.pending }}</dd>

      <dt>Dirty</dt>
      <dd>{{ registrationForm.dirty }}</dd>

      <dt>Touched</dt>
      <dd>{{ registrationForm.touched }}</dd>
    </dl>
  </section>

  @if (savedRegistration; as registration) {
    <section class="success" aria-labelledby="saved-title" role="status">
      <h2 id="saved-title">Form is valid</h2>
      <p>Email: {{ registration.email }}</p>
      <p>Username: {{ registration.username }}</p>
      <p>The password is intentionally not displayed.</p>
    </section>
  }
</main>
~~~

The mismatch message checks the **FormGroup**. The username messages check the child control, including its **PENDING** status while the service response is unresolved.

## 3. Add responsive styling

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

h2 {
  margin-top: 0;
  font-size: 1.25rem;
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
.status,
.error,
.error-summary {
  margin: 0;
  font-size: 0.9rem;
  line-height: 1.5;
}

.hint {
  color: #475467;
}

.status {
  color: #253b80;
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

1. Submit the blank form. Required errors appear; the form does not save.
2. Enter a valid email and a password of at least 12 characters. Enter a different confirmation value. The mismatch error appears on the group after interaction.
3. Make the two password values match. The group-level error clears.
4. Enter a username shorter than three characters or with uppercase letters. Synchronous validation blocks the async check.
5. Enter **taken** and leave the username field. The status shows **PENDING** during the mock delay, then the taken-name error appears.
6. Change it to **asha-r** and leave the field. After the pending state settles, the username passes.
7. Enter **offline** and leave the field. The mock service emits an error; the validator reports **usernameCheckFailed** rather than treating the username as available.
8. Choose **Load valid example**. The form is populated programmatically and marked pristine and untouched. The availability check still runs for the supplied username.
9. Wait for pending validation to finish, then submit. The valid account summary appears. The password is never rendered in the confirmation panel.
10. Choose **Reset** to restore the initial form values and interaction state.

The mock uses an in-memory set and a timer to make async states visible. It does not prove uniqueness and does not replace an API or a server constraint. A real registration endpoint must handle a name that becomes unavailable after the check.

## Further practice

- Replace the mock service with an **HttpClient** request that returns whether a username is available.
- Add a retry action for a failed availability check.
- Add a cross-field rule that requires a confirmation field only when a checkbox is selected.
- Change **updateOn** from **'blur'** to **'change'** and observe how often the mock runs.
- Add a service-side delay or rate limit appropriate for a real endpoint.
- Make the server reject duplicate usernames during account creation and show its conflict response.
- Add a new password rule and show its child-level error separately from the group mismatch error.
- Add a visible submit progress state for the actual account-creation request.

## Related material

- Read the [lesson](../lessons/067-cross-field-and-asynchronous-validation.md).
- Review the [Q&A](../qnas/067-cross-field-and-asynchronous-validation.md).
- Continue with [Submission flows, pending state, and reset behavior](../lessons/068-submission-flows-pending-state-and-reset-behavior.md).
- Revisit [Synchronous validators and validation state](../lessons/066-synchronous-validators-and-validation-state.md).

## Official references

- [Validate form input](https://angular.dev/guide/forms/form-validation)
- [AsyncValidatorFn API](https://angular.dev/api/forms/AsyncValidatorFn)
- [AsyncValidator API](https://angular.dev/api/forms/AsyncValidator)
- [AbstractControlOptions API](https://angular.dev/api/forms/AbstractControlOptions)
- [HttpClient](https://angular.dev/guide/http)
