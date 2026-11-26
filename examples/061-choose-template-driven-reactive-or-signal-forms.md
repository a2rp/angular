# Example: Choose template-driven, reactive, or signal forms

This example implements the same contact form in three ways:

1. Template-driven forms with **FormsModule**, **ngModel**, and **NgForm**.
2. Reactive forms with a typed **FormGroup** and validators.
3. Signal Forms with a signal model, schema validators, and **FormRoot** submission.

Use one implementation at a time. Each version replaces **src/app/app.component.ts** and **src/app/app.component.html** in the same standalone Angular app. Keep the shared stylesheet. Signal Forms are stable starting with Angular 22; check the compatibility of the project version before using that version.

Create a practice project:

~~~sh
ng new angular-form-comparison --style=css
cd angular-form-comparison
ng serve --open
~~~

For each approach below, copy its TypeScript component and matching template into the generated files. Then add the shared stylesheet at the end of the example.

## Version 1: Template-driven form

This version keeps a plain model in the component and declares controls and validators in the template. A form reference is passed to the submit handler so invalid submission can mark fields touched and stop safely.

Replace **src/app/app.component.ts**:

~~~ts
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';
import type { NgForm } from '@angular/forms';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [FormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  model = {
    name: '',
    email: '',
    message: '',
  };

  submitted = false;
  saved = false;

  save(form: NgForm): void {
    this.submitted = true;

    if (form.invalid) {
      form.control.markAllAsTouched();
      return;
    }

    this.saved = true;
    console.log({ ...this.model });
  }
}
~~~

Replace **src/app/app.component.html**:

~~~html
<main class="page">
  <header>
    <p class="eyebrow">Template-driven forms</p>
    <h1>Contact us</h1>
    <p>Complete the fields and submit the form.</p>
  </header>

  <form #contactForm="ngForm" (ngSubmit)="save(contactForm)" novalidate>
    <div class="field">
      <label for="name">Name</label>
      <input
        id="name"
        name="name"
        [(ngModel)]="model.name"
        required
        #name="ngModel"
        [attr.aria-invalid]="name.invalid && (name.touched || submitted) ? 'true' : null"
        [attr.aria-describedby]="name.invalid && (name.touched || submitted) ? 'name-error' : null"
      />
      @if (name.invalid && (name.touched || submitted)) {
        <p class="error" id="name-error">Enter your name.</p>
      }
    </div>

    <div class="field">
      <label for="email">Email</label>
      <input
        id="email"
        name="email"
        type="email"
        [(ngModel)]="model.email"
        required
        email
        #email="ngModel"
        [attr.aria-invalid]="email.invalid && (email.touched || submitted) ? 'true' : null"
        [attr.aria-describedby]="email.invalid && (email.touched || submitted) ? 'email-error' : null"
      />
      @if (email.invalid && (email.touched || submitted)) {
        <p class="error" id="email-error">
          Enter a valid email address.
        </p>
      }
    </div>

    <div class="field">
      <label for="message">Message</label>
      <textarea
        id="message"
        name="message"
        rows="5"
        [(ngModel)]="model.message"
        required
        minlength="10"
        #message="ngModel"
        [attr.aria-invalid]="message.invalid && (message.touched || submitted) ? 'true' : null"
        [attr.aria-describedby]="message.invalid && (message.touched || submitted) ? 'message-error' : null"
      ></textarea>
      @if (message.invalid && (message.touched || submitted)) {
        <p class="error" id="message-error">
          Enter a message with at least 10 characters.
        </p>
      }
    </div>

    <button type="submit">Send message</button>

    @if (saved) {
      <p class="success" role="status">Your message is ready to send.</p>
    }
  </form>
</main>
~~~

The template declares the control names, bindings, and rules. **NgForm** collects the controls. The component keeps the data in a plain object.

## Version 2: Reactive form

This version creates an explicitly typed form model in TypeScript. The template binds to it with **formGroup** and **formControlName**.

Replace **src/app/app.component.ts**:

~~~ts
import { Component } from '@angular/core';
import {
  FormControl,
  FormGroup,
  ReactiveFormsModule,
  Validators,
} from '@angular/forms';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  submitted = false;
  saved = false;

  readonly form = new FormGroup({
    name: new FormControl('', {
      nonNullable: true,
      validators: [Validators.required],
    }),
    email: new FormControl('', {
      nonNullable: true,
      validators: [Validators.required, Validators.email],
    }),
    message: new FormControl('', {
      nonNullable: true,
      validators: [
        Validators.required,
        Validators.minLength(10),
      ],
    }),
  });

  save(): void {
    this.submitted = true;

    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    this.saved = true;
    console.log(this.form.getRawValue());
  }
}
~~~

Replace **src/app/app.component.html**:

~~~html
<main class="page">
  <header>
    <p class="eyebrow">Reactive forms</p>
    <h1>Contact us</h1>
    <p>Complete the fields and submit the form.</p>
  </header>

  <form [formGroup]="form" (ngSubmit)="save()" novalidate>
    <div class="field">
      <label for="name">Name</label>
      <input
        id="name"
        formControlName="name"
        [attr.aria-invalid]="form.controls.name.invalid && (form.controls.name.touched || submitted) ? 'true' : null"
        [attr.aria-describedby]="form.controls.name.invalid && (form.controls.name.touched || submitted) ? 'name-error' : null"
      />
      @if (form.controls.name.invalid && (form.controls.name.touched || submitted)) {
        <p class="error" id="name-error">Enter your name.</p>
      }
    </div>

    <div class="field">
      <label for="email">Email</label>
      <input
        id="email"
        type="email"
        formControlName="email"
        [attr.aria-invalid]="form.controls.email.invalid && (form.controls.email.touched || submitted) ? 'true' : null"
        [attr.aria-describedby]="form.controls.email.invalid && (form.controls.email.touched || submitted) ? 'email-error' : null"
      />
      @if (form.controls.email.invalid && (form.controls.email.touched || submitted)) {
        <p class="error" id="email-error">Enter a valid email address.</p>
      }
    </div>

    <div class="field">
      <label for="message">Message</label>
      <textarea
        id="message"
        rows="5"
        formControlName="message"
        [attr.aria-invalid]="form.controls.message.invalid && (form.controls.message.touched || submitted) ? 'true' : null"
        [attr.aria-describedby]="form.controls.message.invalid && (form.controls.message.touched || submitted) ? 'message-error' : null"
      ></textarea>
      @if (form.controls.message.invalid && (form.controls.message.touched || submitted)) {
        <p class="error" id="message-error">
          Enter a message with at least 10 characters.
        </p>
      }
    </div>

    <button type="submit">Send message</button>

    @if (saved) {
      <p class="success" role="status">Your message is ready to send.</p>
    }
  </form>
</main>
~~~

The component owns the control tree and validation functions. The submit handler marks controls touched before returning from an invalid submission, so the template can show the relevant messages.

## Version 3: Signal Form

Use this variant with Angular 22 or later, where current Angular documentation marks Signal Forms stable. The signal is the model, the schema declares validation, and **FormRoot** connects submission to its configured action.

Replace **src/app/app.component.ts**:

~~~ts
import { Component, signal } from '@angular/core';
import {
  email,
  form,
  FormField,
  FormRoot,
  minLength,
  required,
} from '@angular/forms/signals';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [FormField, FormRoot],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  readonly model = signal({
    name: '',
    email: '',
    message: '',
  });

  readonly saved = signal(false);

  readonly contactForm = form(
    this.model,
    (path) => {
      required(path.name, { message: 'Enter your name.' });
      required(path.email, { message: 'Enter your email address.' });
      email(path.email, { message: 'Enter a valid email address.' });
      required(path.message, { message: 'Enter a message.' });
      minLength(path.message, 10, {
        message: 'Use at least 10 characters.',
      });
    },
    {
      submission: {
        action: async () => {
          this.saved.set(true);
          console.log(this.model());
        },
      },
    },
  );
}
~~~

Replace **src/app/app.component.html**:

~~~html
<main class="page">
  <header>
    <p class="eyebrow">Signal Forms</p>
    <h1>Contact us</h1>
    <p>Complete the fields and submit the form.</p>
  </header>

  <form [formRoot]="contactForm">
    <div class="field">
      <label for="name">Name</label>
      <input
        id="name"
        [formField]="contactForm.name"
        [attr.aria-invalid]="contactForm.name().touched() && contactForm.name().invalid() ? 'true' : null"
        [attr.aria-describedby]="contactForm.name().touched() && contactForm.name().invalid() ? 'name-error' : null"
      />
      @if (contactForm.name().touched() && contactForm.name().invalid()) {
        <p class="error" id="name-error">
          {{ contactForm.name().errors()[0]?.message }}
        </p>
      }
    </div>

    <div class="field">
      <label for="email">Email</label>
      <input
        id="email"
        type="email"
        [formField]="contactForm.email"
        [attr.aria-invalid]="contactForm.email().touched() && contactForm.email().invalid() ? 'true' : null"
        [attr.aria-describedby]="contactForm.email().touched() && contactForm.email().invalid() ? 'email-error' : null"
      />
      @if (contactForm.email().touched() && contactForm.email().invalid()) {
        <p class="error" id="email-error">
          {{ contactForm.email().errors()[0]?.message }}
        </p>
      }
    </div>

    <div class="field">
      <label for="message">Message</label>
      <textarea
        id="message"
        rows="5"
        [formField]="contactForm.message"
        [attr.aria-invalid]="contactForm.message().touched() && contactForm.message().invalid() ? 'true' : null"
        [attr.aria-describedby]="contactForm.message().touched() && contactForm.message().invalid() ? 'message-error' : null"
      ></textarea>
      @if (contactForm.message().touched() && contactForm.message().invalid()) {
        <p class="error" id="message-error">
          {{ contactForm.message().errors()[0]?.message }}
        </p>
      }
    </div>

    <button type="submit" [disabled]="contactForm().submitting()">
      @if (contactForm().submitting()) {
        Sending…
      } @else {
        Send message
      }
    </button>

    @if (saved()) {
      <p class="success" role="status">Your message is ready to send.</p>
    }
  </form>
</main>
~~~

With **FormRoot** and a submission action configured, submit marks interactive fields touched, checks validation, and runs the action only when the form is valid. The submit button remains available for an invalid form so the user can trigger the form's error presentation; it is disabled while a submission is in progress.

## Shared stylesheet

Use this as **src/app/app.component.css** for any of the three versions:

~~~css
:host {
  display: block;
  min-height: 100vh;
  color: #192336;
  background: #f3f6fb;
  font-family: system-ui, sans-serif;
}

.page {
  box-sizing: border-box;
  width: min(100% - 2rem, 42rem);
  margin: 0 auto;
  padding: 3rem 0;
}

header {
  margin-bottom: 1.5rem;
}

.eyebrow {
  margin-bottom: 0.4rem;
  color: #3655a4;
  font-size: 0.85rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1 {
  margin: 0 0 0.6rem;
  font-size: clamp(2rem, 5vw, 3rem);
}

header p:last-child {
  color: #536176;
  line-height: 1.6;
}

form {
  display: grid;
  gap: 1.1rem;
  padding: 1.4rem;
  border: 1px solid #d7dfec;
  border-radius: 0.8rem;
  background: #fff;
  box-shadow: 0 0.4rem 1.4rem rgb(27 45 79 / 6%);
}

.field {
  display: grid;
  gap: 0.4rem;
}

label {
  font-weight: 650;
}

input,
textarea {
  box-sizing: border-box;
  width: 100%;
  padding: 0.7rem 0.75rem;
  border: 1px solid #9ba9bd;
  border-radius: 0.4rem;
  color: inherit;
  background: #fff;
  font: inherit;
}

textarea {
  resize: vertical;
}

input:focus-visible,
textarea:focus-visible,
button:focus-visible {
  outline: 3px solid #e6ad36;
  outline-offset: 2px;
}

[aria-invalid='true'] {
  border-color: #a82e37;
}

.error {
  margin: 0;
  color: #a32631;
  font-size: 0.92rem;
}

button {
  justify-self: start;
  min-height: 2.75rem;
  padding: 0.65rem 1rem;
  border: 1px solid #2f4e9a;
  border-radius: 0.45rem;
  color: #fff;
  background: #2f4e9a;
  font: inherit;
  font-weight: 650;
  cursor: pointer;
}

button:hover:not(:disabled) {
  background: #243d7d;
}

button:disabled {
  cursor: wait;
  opacity: 0.7;
}

.success {
  margin: 0;
  color: #17633c;
  font-weight: 650;
}

@media (max-width: 34rem) {
  .page {
    padding: 1.5rem 0;
  }

  form {
    padding: 1rem;
  }
}
~~~

## Compare the experience

Try submitting each version while every field is empty, then correct the fields:

- **Template-driven:** the template declares rules and control names; the component reads the form through **NgForm**.
- **Reactive:** the component declares the form shape and validator functions; the template binds to named controls.
- **Signal Forms:** the signal supplies the model; the schema defines rules and each field exposes signal-based state.

The same visible form can be implemented with each API. The meaningful difference is how the model, rules, state, and submission flow are organized.

## Next experiments

1. Add a phone field with a pattern validator to each version.
2. Add a rule that requires a message when a checkbox is selected.
3. Add an asynchronous email availability check and decide how pending state should appear.
4. Return a server-side field error after submission.
5. Add a repeated phone-number field and compare how each form represents a dynamic list.
6. Write a test for invalid submission, valid submission, and the accessible error text.

## Important boundaries

- These examples log the accepted form model; they do not send data to a server.
- Client-side validators improve feedback but are not a security boundary. Validate input on the server.
- A successful local submission state does not prove a remote request succeeded.
- Signal Forms require an Angular version that supports the stable API; use the compatibility guidance for the project.

## Continue learning

- Previous: [Error handling, cleanup, and takeUntilDestroyed](../lessons/060-error-handling-cleanup-and-takeuntildestroyed.md)
- Lesson: [Choose template-driven, reactive, or signal forms](../lessons/061-choose-template-driven-reactive-or-signal-forms.md)
- Q&A: [Choose template-driven, reactive, or signal forms](../qnas/061-choose-template-driven-reactive-or-signal-forms.md)
- Next: [Template-driven forms and ngForm](../lessons/062-template-driven-forms-and-ngform.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="060-error-handling-cleanup-and-takeuntildestroyed.md">060 - Error handling, cleanup, and takeUntilDestroyed</a></td><td><a href="../lessons/061-choose-template-driven-reactive-or-signal-forms.md">Lesson</a></td><td><a href="../qnas/061-choose-template-driven-reactive-or-signal-forms.md">Q&amp;A</a></td><td><a href="062-template-driven-forms-and-ngform.md">062 - Template-driven forms and NgForm</a></td>
</tr>
  </tbody>
</table>
