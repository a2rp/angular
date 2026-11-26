# Example: Signal Forms: models, fields, and form setup

Build a small contact form with a typed model signal, a field tree, and native controls bound through `[formField]`. The page also shows the current model so you can see the two-way synchronization while trying the form.

This example focuses on setup. It intentionally does not add validation rules or send data to a server; those belong to later lessons.

## 1. Create the form component

Create a standalone component, for example `contact-form.component.ts`:

~~~ts
import {Component, signal} from '@angular/core';
import {FormField, form} from '@angular/forms/signals';

interface ContactFormModel {
  name: string;
  email: string;
  message: string;
  preferredContact: 'email' | 'phone';
  updatesAllowed: boolean;
}

function createInitialContact(): ContactFormModel {
  return {
    name: '',
    email: '',
    message: '',
    preferredContact: 'email',
    updatesAllowed: false,
  };
}

@Component({
  selector: 'app-contact-form',
  imports: [FormField],
  template: `
    <main>
      <h1>Contact us</h1>
      <p>
        Enter sample details. The preview below updates from the form model.
        This demo stores data in memory only.
      </p>

      <form novalidate (submit)="save($event)">
        <div class="field">
          <label for="contact-name">Name</label>
          <input
            id="contact-name"
            type="text"
            autocomplete="name"
            [formField]="contactForm.name"
          />
        </div>

        <div class="field">
          <label for="contact-email">Email</label>
          <input
            id="contact-email"
            type="email"
            autocomplete="email"
            [formField]="contactForm.email"
          />
        </div>

        <div class="field">
          <label for="contact-message">Message</label>
          <textarea
            id="contact-message"
            rows="5"
            [formField]="contactForm.message"
          ></textarea>
        </div>

        <div class="field">
          <label for="preferred-contact">Preferred reply method</label>
          <select
            id="preferred-contact"
            [formField]="contactForm.preferredContact"
          >
            <option value="email">Email</option>
            <option value="phone">Phone</option>
          </select>
        </div>

        <div class="checkbox-field">
          <input
            id="updates-allowed"
            type="checkbox"
            [formField]="contactForm.updatesAllowed"
          />
          <label for="updates-allowed">Send me occasional updates</label>
        </div>

        <div class="actions">
          <button type="submit">Save locally</button>
          <button type="button" (click)="resetForm()">Reset</button>
        </div>
      </form>

      <p class="status" role="status" aria-live="polite">
        {{ statusMessage }}
      </p>

      <section aria-labelledby="preview-heading">
        <h2 id="preview-heading">Current form model</h2>
        <dl>
          <dt>Name</dt>
          <dd>{{ contactModel().name || '—' }}</dd>

          <dt>Email</dt>
          <dd>{{ contactModel().email || '—' }}</dd>

          <dt>Message</dt>
          <dd>{{ contactModel().message || '—' }}</dd>

          <dt>Preferred reply method</dt>
          <dd>{{ contactModel().preferredContact }}</dd>

          <dt>Occasional updates</dt>
          <dd>{{ contactModel().updatesAllowed ? 'Yes' : 'No' }}</dd>
        </dl>
      </section>

      @if (savedContact; as saved) {
        <section aria-labelledby="saved-heading">
          <h2 id="saved-heading">Last locally saved values</h2>
          <p>{{ saved.name || 'No name entered' }} — {{ saved.email || 'No email entered' }}</p>
        </section>
      }
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
      width: min(100% - 2rem, 42rem);
      margin: 2rem auto;
    }

    .field {
      display: grid;
      gap: 0.35rem;
      margin-block: 1rem;
    }

    input,
    select,
    textarea {
      box-sizing: border-box;
      width: 100%;
      min-height: 2.75rem;
      padding: 0.6rem 0.7rem;
      border: 1px solid #687078;
      border-radius: 0.3rem;
      color: inherit;
      font: inherit;
    }

    textarea {
      min-height: 8rem;
      resize: vertical;
    }

    .checkbox-field {
      display: flex;
      align-items: center;
      gap: 0.6rem;
      margin-block: 1rem;
    }

    .checkbox-field input {
      width: 1.2rem;
      min-height: 1.2rem;
    }

    .actions {
      display: flex;
      flex-wrap: wrap;
      gap: 0.75rem;
      margin-block: 1.25rem;
    }

    button {
      min-height: 2.75rem;
      padding: 0.55rem 0.9rem;
      border: 1px solid #39434d;
      border-radius: 0.3rem;
      background: #f5f7f8;
      color: inherit;
      font: inherit;
      cursor: pointer;
    }

    button:focus-visible,
    input:focus-visible,
    select:focus-visible,
    textarea:focus-visible {
      outline: 3px solid #075db8;
      outline-offset: 2px;
    }

    .status {
      min-height: 1.5em;
    }

    dt {
      margin-top: 0.6rem;
      font-weight: 700;
    }

    dd {
      margin-left: 0;
      overflow-wrap: anywhere;
    }
  `],
})
export class ContactFormComponent {
  readonly contactModel = signal<ContactFormModel>(createInitialContact());
  readonly contactForm = form(this.contactModel);

  statusMessage = '';
  savedContact: ContactFormModel | null = null;

  save(event: SubmitEvent): void {
    event.preventDefault();

    // Copy the current values to demonstrate a submit-time snapshot.
    // No network request is made in this example.
    this.savedContact = {...this.contactModel()};
    this.statusMessage = 'Values saved in this component only.';
  }

  resetForm(): void {
    this.contactModel.set(createInitialContact());
    this.savedContact = null;
    this.statusMessage = 'The form was reset.';
  }
}
`~~~

The model uses plain data: strings, a string union, and a boolean. Every field has an initial value, so `form()` can create the complete field tree.

The component imports `FormField` because its template uses `[formField]`. Each binding points to the matching field-tree property. The labels use matching `for` and `id` values so each control has an accessible name.

## 2. Render the component

Import `ContactFormComponent` into the standalone root component and render its selector. For example:

~~~ts
import {Component} from '@angular/core';
import {ContactFormComponent} from './contact-form.component';

@Component({
  selector: 'app-root',
  imports: [ContactFormComponent],
  template: '<app-contact-form />',
})
export class App {}
~~~

Use the root component filename and class name that already exist in your Angular CLI project. Keep the import path aligned with where you saved the contact form component.

## 3. Try the data flow

1. Start the Angular app with `ng serve`.
2. Type in the name, email, and message fields.
3. Change the select and toggle the checkbox.
4. Watch the **Current form model** section update as you interact.
5. Click **Save locally**. The component takes a snapshot of the current model and announces that it stayed in memory.
6. Click **Reset**. The model signal returns to its initial object, so the bound controls and preview return to their initial values.

The template reads the model with calls such as `contactModel().name`. The values change because `[formField]` keeps each bound control synchronized with its model field.

## 4. Inspect field state

The field tree also exposes reactive state for each field. For example, you can inspect the email field's current value or interaction state:

~~~ts
const emailValue = this.contactForm.email().value();
const emailHasBeenEdited = this.contactForm.email().dirty();
~~~

These are reads, not separate values to keep synchronized. The state APIs can be used by templates or component logic when the UI needs to react to a field. Validation and error presentation should be added with the schema and validation APIs covered in their own lesson.

## 5. Keep the model boundary intentional

This form's model is shaped for the controls. If an application API expects a different payload, create that payload when submitting:

~~~ts
const model = this.contactModel();

const requestBody = {
  displayName: model.name.trim(),
  replyTo: model.email.trim(),
  body: model.message.trim(),
  replyByPhone: model.preferredContact === 'phone',
  subscribed: model.updatesAllowed,
};
~~~

This explicit conversion makes it clear which values are collected by the UI and which values are sent to the application layer. Add real validation and an actual request before using a form to collect production data.

## What to notice

- The writable signal is the form's data source.
- The field tree mirrors the model's properties.
- The template binds controls to the field tree rather than manually copying each input event.
- Programmatic model updates, including reset, flow back to the controls.
- Labels, instructions, validation, and network submission are separate responsibilities.
- This setup uses Signal Forms bindings, not Reactive Forms' `formControlName`.

## References

- [Signal Forms overview](https://angular.dev/guide/forms/signals/overview)
- [Signal Forms essentials](https://angular.dev/essentials/signal-forms)
- [Form models](https://angular.dev/guide/forms/signals/models)
- [form() API](https://angular.dev/api/forms/signals/form)
- [FormField API](https://angular.dev/api/forms/signals/FormField)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="070-custom-controls-with-controlvalueaccessor.md">070 - Custom controls with ControlValueAccessor</a></td><td><a href="../lessons/071-signal-forms-models-fields-and-form-setup.md">Lesson</a></td><td><a href="../qnas/071-signal-forms-models-fields-and-form-setup.md">Q&amp;A</a></td><td><a href="072-signal-forms-schemas-validation-and-custom-controls.md">072 - Signal Forms: schemas, validation, and custom controls</a></td>
</tr>
  </tbody>
</table>
