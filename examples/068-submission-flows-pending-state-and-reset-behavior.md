# Example: Submission flows, pending state, and reset behavior

Build a contact form that demonstrates two independent states:

- An asynchronous email-availability check makes the form **PENDING**.
- A simulated save request sets the component's **saving** flag.

A successful save resets the form and the form directive. A failed save preserves the values. This example uses an in-memory mock service and makes no network request.

Use an Angular CLI application with standalone components and reactive forms. Replace the root component file or adapt the example to your application.

## 1. Add the component and mock API

Replace the root component TypeScript file:

~~~ts
import { Component, Injectable, inject } from '@angular/core';
import {
  AbstractControl,
  AsyncValidatorFn,
  FormBuilder,
  FormGroupDirective,
  ReactiveFormsModule,
  ValidationErrors,
  Validators,
} from '@angular/forms';
import {
  Observable,
  catchError,
  finalize,
  map,
  of,
  switchMap,
  take,
  throwError,
  timer,
} from 'rxjs';

interface ContactPayload {
  name: string;
  email: string;
  message: string;
}

interface SaveResult {
  reference: string;
}

@Injectable({ providedIn: 'root' })
class ContactApi {
  checkEmail(email: string): Observable<boolean> {
    const normalizedEmail = email.trim().toLowerCase();

    // The delay keeps the async PENDING state visible.
    return timer(700).pipe(
      map(() => normalizedEmail !== 'used@example.com'),
    );
  }

  saveContact(payload: ContactPayload): Observable<SaveResult> {
    return timer(900).pipe(
      switchMap(() => {
        // Simulate a server rejection so the failure path can be tried.
        if (payload.email.toLowerCase().endsWith('@fail.example')) {
          return throwError(() => new Error('Simulated save failure'));
        }

        return of({ reference: 'CONTACT-DEMO-068' });
      }),
    );
  }
}

function emailAvailabilityValidator(api: ContactApi): AsyncValidatorFn {
  return (control: AbstractControl) => {
    const email = String(control.value ?? '').trim().toLowerCase();

    // Required and format validators explain empty or malformed values.
    if (!email) {
      return of(null);
    }

    return api.checkEmail(email).pipe(
      map((available): ValidationErrors | null =>
        available ? null : { emailTaken: true },
      ),
      catchError(() => of({ emailCheckUnavailable: true })),
    );
  };
}

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: \`
    <main>
      <h1>Contact form</h1>
      <p>
        Try <code>used@example.com</code> to see an availability error.
        Use any address ending in <code>@fail.example</code> to simulate a
        save failure.
      </p>

      <form
        [formGroup]="contactForm"
        #formDirective="ngForm"
        (ngSubmit)="submit(formDirective)"
        (input)="clearMessages()"
        novalidate
      >
        <div>
          <label for="name">Name</label>
          <input
            id="name"
            type="text"
            formControlName="name"
            autocomplete="name"
          />
          @if (contactForm.controls.name.invalid &&
               contactForm.controls.name.touched) {
            <p role="alert">Enter your name.</p>
          }
        </div>

        <div>
          <label for="email">Email</label>
          <input
            id="email"
            type="email"
            formControlName="email"
            autocomplete="email"
            aria-describedby="email-status"
          />

          @if (contactForm.controls.email.pending) {
            <p id="email-status" role="status" aria-live="polite">
              Checking whether this email is available…
            </p>
          }

          @if (contactForm.controls.email.invalid &&
               contactForm.controls.email.touched) {
            @if (contactForm.controls.email.hasError('required')) {
              <p role="alert">Enter an email address.</p>
            }
            @if (contactForm.controls.email.hasError('email')) {
              <p role="alert">Enter a valid email address.</p>
            }
            @if (contactForm.controls.email.hasError('emailTaken')) {
              <p role="alert">That email address is already in use.</p>
            }
            @if (contactForm.controls.email.hasError('emailCheckUnavailable')) {
              <p role="alert">The availability check failed. Edit the email and try again.</p>
            }
          }
        </div>

        <div>
          <label for="message">Message</label>
          <textarea id="message" formControlName="message" rows="5"></textarea>
          @if (contactForm.controls.message.invalid &&
               contactForm.controls.message.touched) {
            <p role="alert">
              Enter a message with at least 10 characters.
            </p>
          }
        </div>

        <button type="submit" [disabled]="saving">
          {{ saving ? 'Saving…' : 'Send message' }}
        </button>
        <button
          type="button"
          (click)="resetForm(formDirective)"
          [disabled]="saving"
        >
          Clear form
        </button>
      </form>

      @if (contactForm.pending) {
        <p role="status" aria-live="polite">
          Validation is still in progress. Submit again after it finishes.
        </p>
      }

      @if (saving) {
        <p role="status" aria-live="polite">Saving your message…</p>
      }

      @if (successMessage) {
        <p role="status" aria-live="polite">{{ successMessage }}</p>
      }

      @if (submitError) {
        <p role="alert">{{ submitError }}</p>
      }

      <p>Form status: {{ contactForm.status }}</p>
    </main>
  \`,
})
export class AppComponent {
  private readonly api = inject(ContactApi);
  private readonly fb = inject(FormBuilder);

  readonly contactForm = this.fb.nonNullable.group({
    name: ['', Validators.required],
    email: this.fb.nonNullable.control('', {
      validators: [Validators.required, Validators.email],
      asyncValidators: [emailAvailabilityValidator(this.api)],
    }),
    message: ['', [Validators.required, Validators.minLength(10)]],
  });

  saving = false;
  successMessage = '';
  submitError = '';

  submit(formDirective: FormGroupDirective): void {
    if (this.contactForm.invalid || this.contactForm.pending || this.saving) {
      this.contactForm.markAllAsTouched();
      return;
    }

    // Snapshot the values the user submitted. This includes disabled fields.
    const payload = this.contactForm.getRawValue();

    this.saving = true;
    this.successMessage = '';
    this.submitError = '';

    this.api.saveContact(payload)
      .pipe(
        take(1),
        finalize(() => {
          this.saving = false;
        }),
      )
      .subscribe({
        next: (result) => {
          formDirective.resetForm({
            name: '',
            email: '',
            message: '',
          });
          this.successMessage = \`Message saved. Reference: ${result.reference}\`;
        },
        error: () => {
          // Leave the entered values in place so the user can retry.
          this.submitError =
            'The message could not be saved. Your entries are still here; please try again.';
        },
      });
  }

  resetForm(formDirective: FormGroupDirective): void {
    formDirective.resetForm({
      name: '',
      email: '',
      message: '',
    });
    this.successMessage = '';
    this.submitError = '';
  }

  clearMessages(): void {
    this.successMessage = '';
    this.submitError = '';
  }
}
~~~

## 2. Try each state

1. Submit the empty form. The handler marks every control touched, and the required messages appear.
2. Enter a valid name, message, and a new email address. While the email availability check runs, **contactForm.status** becomes **PENDING** and the status message appears.
3. Submit during that check. The handler returns without saving. After the check completes, submit again.
4. Enter **used@example.com**. After the delayed check, the email control becomes invalid and displays the availability error.
5. Use an address ending in **@fail.example** with otherwise valid values. The save request runs, then fails. The component clears **saving**, shows a request error, and preserves the entered values.
6. Use a different email. After the async check and successful save, **resetForm()** clears the values, marks controls pristine and untouched, and resets the form directive's submitted state. A success message remains visible.
7. Use the **Clear form** button to reset without submitting. Its **type="button"** prevents it from triggering the form submission.

## 3. Understand the state transitions

| Situation | Form status | Component state | Result |
| --- | --- | --- | --- |
| Required fields are empty | **INVALID** | **saving = false** | Submit marks fields touched and returns |
| Email check is unresolved | **PENDING** | **saving = false** | No save request starts |
| Email is already in use | **INVALID** | **saving = false** | Availability error appears |
| Save request is in flight | **VALID** | **saving = true** | Progress is announced and repeat submit is disabled |
| Save request succeeds | Reset form returns to its new status | **saving = false** | Directive state resets and confirmation appears |
| Save request fails | Usually **VALID** | **saving = false** | Input values remain and request error appears |

The mock email check is intentionally simple. In a real application, use an API for the availability check, handle transport failures according to product requirements, and enforce uniqueness again on the server. The save endpoint must validate its payload independently of browser-side state.

## Further reading

- [Submission flows, pending state, and reset behavior](../lessons/068-submission-flows-pending-state-and-reset-behavior.md)
- [Form validation](https://angular.dev/guide/forms/form-validation)
- [FormGroup API](https://angular.dev/api/forms/FormGroup)
- [AbstractFormDirective API](https://angular.dev/api/forms/AbstractFormDirective)
