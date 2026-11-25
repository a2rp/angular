# Example: Accessible labels, instructions, and validation messages

Build a contact form that demonstrates visible labels, persistent instructions, a related radio group, inline errors, and a submit-time error summary. Reactive Forms supplies validation state; the template connects that state to accessible HTML and ARIA attributes.

This is a client-side demonstration only. It does not send or store data.

Use an Angular CLI application with standalone components and reactive forms. Replace the root component file or adapt the example to your application.

## 1. Add the standalone component

Replace the root component TypeScript file:

~~~ts
import { Component, inject } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';

type FieldName = 'fullName' | 'email' | 'contactMethod' | 'message';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <main>
      <h1>Contact us</h1>
      <p id="form-help">
        Fields marked “required” must be completed. Your email is used only
        to reply to your message.
      </p>

      <form
        [formGroup]="contactForm"
        (ngSubmit)="submit()"
        (input)="clearStatus()"
        (change)="clearStatus()"
        novalidate
      >
        @if (submitted && contactForm.invalid) {
          <section aria-labelledby="error-summary-title" role="alert">
            <h2 id="error-summary-title">Please correct these fields</h2>
            <ul>
              @if (contactForm.controls.fullName.invalid) {
                <li><a href="#full-name">Enter your full name.</a></li>
              }
              @if (contactForm.controls.email.invalid) {
                <li><a href="#email">Enter a valid email address.</a></li>
              }
              @if (contactForm.controls.contactMethod.invalid) {
                <li><a href="#contact-email">Choose a contact method.</a></li>
              }
              @if (contactForm.controls.message.invalid) {
                <li>
                  <a href="#message">Enter a message with at least 20 characters.</a>
                </li>
              }
            </ul>
          </section>
        }

        <div>
          <label for="full-name">Full name (required)</label>
          <p id="full-name-help">Enter your first and last name.</p>
          <input
            id="full-name"
            name="fullName"
            type="text"
            formControlName="fullName"
            autocomplete="name"
            required
            [attr.aria-describedby]="
              showError('fullName')
                ? 'full-name-help full-name-error'
                : 'full-name-help'
            "
            [attr.aria-invalid]="showError('fullName') ? 'true' : null"
          />
          @if (showError('fullName')) {
            <p id="full-name-error">Enter your full name.</p>
          }
        </div>

        <div>
          <label for="email">Email address (required)</label>
          <p id="email-help">Use an address such as name@example.com.</p>
          <input
            id="email"
            name="email"
            type="email"
            formControlName="email"
            autocomplete="email"
            required
            [attr.aria-describedby]="
              showError('email') ? 'email-help email-error' : 'email-help'
            "
            [attr.aria-invalid]="showError('email') ? 'true' : null"
          />
          @if (showError('email')) {
            <p id="email-error">
              @if (contactForm.controls.email.hasError('required')) {
                Enter your email address.
              } @else if (contactForm.controls.email.hasError('email')) {
                Enter a valid email address, such as name@example.com.
              }
            </p>
          }
        </div>

        <fieldset
          [attr.aria-describedby]="
            showError('contactMethod')
              ? 'contact-method-help contact-method-error'
              : 'contact-method-help'
          "
        >
          <legend>Preferred contact method (required)</legend>
          <p id="contact-method-help">Choose one option.</p>

          <label for="contact-email">
            <input
              id="contact-email"
              type="radio"
              name="contactMethod"
              formControlName="contactMethod"
              value="email"
              required
              [attr.aria-invalid]="
                showError('contactMethod') ? 'true' : null
              "
            />
            Email
          </label>

          <label for="contact-phone">
            <input
              id="contact-phone"
              type="radio"
              name="contactMethod"
              formControlName="contactMethod"
              value="phone"
              [attr.aria-invalid]="
                showError('contactMethod') ? 'true' : null
              "
            />
            Phone
          </label>

          @if (showError('contactMethod')) {
            <p id="contact-method-error">Choose a contact method.</p>
          }
        </fieldset>

        <div>
          <label for="message">Message (required)</label>
          <p id="message-help">
            Include at least 20 characters so we have enough detail to reply.
          </p>
          <textarea
            id="message"
            name="message"
            rows="5"
            formControlName="message"
            required
            [attr.aria-describedby]="
              showError('message') ? 'message-help message-error' : 'message-help'
            "
            [attr.aria-invalid]="showError('message') ? 'true' : null"
          ></textarea>
          @if (showError('message')) {
            <p id="message-error">
              @if (contactForm.controls.message.hasError('required')) {
                Enter a message.
              } @else if (contactForm.controls.message.hasError('minlength')) {
                Add more detail. The message must contain at least 20 characters.
              }
            </p>
          }
        </div>

        <button type="submit">Review message</button>
        <button type="button" (click)="resetForm()">Clear form</button>
      </form>

      <p role="status" aria-live="polite">{{ successMessage }}</p>
    </main>
  `,
})
export class AppComponent {
  private readonly fb = inject(FormBuilder);

  readonly contactForm = this.fb.nonNullable.group({
    fullName: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]],
    contactMethod: ['', Validators.required],
    message: ['', [Validators.required, Validators.minLength(20)]],
  });

  submitted = false;
  successMessage = '';

  showError(field: FieldName): boolean {
    const control = this.contactForm.controls[field];
    return control.invalid && (control.touched || this.submitted);
  }

  submit(): void {
    this.submitted = true;
    this.contactForm.markAllAsTouched();

    if (this.contactForm.invalid || this.contactForm.pending) {
      return;
    }

    // This demo validates the form but does not send its contents anywhere.
    this.successMessage = 'The form is valid. No data was sent.';
    this.contactForm.reset();
    this.submitted = false;
  }

  resetForm(): void {
    this.contactForm.reset();
    this.submitted = false;
    this.successMessage = '';
  }

  clearStatus(): void {
    this.successMessage = '';
  }
}
~~~

## 2. Try the form

1. Submit with every field empty. The form marks its controls touched, inline errors appear, and the summary links to each invalid field.
2. Activate a summary link. The browser navigates to the corresponding input or radio option.
3. Enter a malformed email address. Its message explains the expected format.
4. Leave the contact choice empty or type fewer than 20 message characters. Submit and confirm the matching field message appears.
5. Correct a field. Its error and **aria-invalid** state clear when the control becomes valid.
6. Submit valid values. A polite status message confirms the demo accepted them; no data leaves the browser.
7. Use **Clear form**. The values and interaction state reset. The button's **type="button"** means it does not submit the form.

## 3. Understand the relationships

- Each **label** is explicitly associated with one unique control ID.
- The radio choices share a **fieldset** and **legend**; each option also has its own label.
- Help text is persistent, and each field conditionally includes its current error ID in **aria-describedby**.
- **aria-invalid** is set only when an error is being shown, rather than because a required control starts empty.
- Angular validators determine Reactive Forms validity. Matching HTML **required** attributes communicate required status in the markup.
- The error summary appears after an invalid submit attempt and links to the corresponding fields.
- Success feedback uses a polite status region. Individual keystrokes are not live-announced.

In a production application, connect the submit handler to a server, preserve values on request failure, and present server validation errors with the same field associations. The server must validate submitted data independently.

## Further reading

- [Accessible labels, instructions, and validation messages](../lessons/069-accessible-labels-instructions-and-validation-messages.md)
- [Angular reactive forms](https://angular.dev/guide/forms/reactive-forms)
- [Angular form validation](https://angular.dev/guide/forms/form-validation)
- [W3C WAI: Labeling controls](https://www.w3.org/WAI/tutorials/forms/labels/)
- [W3C WAI: Form instructions](https://www.w3.org/WAI/tutorials/forms/instructions/)
- [W3C WAI: User notification](https://www.w3.org/WAI/tutorials/forms/notifications/)
