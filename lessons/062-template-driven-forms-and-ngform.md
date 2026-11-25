# Template-driven forms and NgForm

Template-driven forms let Angular build a form model from directives in the HTML template. They are useful for small, mostly static forms with straightforward validation and submission behavior. The component owns the data, while the template declares the controls, bindings, and validation rules.

This lesson builds a standalone contact form with Angular's `FormsModule`, `[(ngModel)]`, and `NgForm`.

## Learning outcomes

After this lesson, you should be able to:

- Import `FormsModule` into a standalone component and use its template-driven form directives.
- Bind controls to a component model with `[(ngModel)]`.
- Register controls with a parent form using unique `name` attributes.
- Read a control's state and the aggregate form state through template references.
- Add native validation attributes and show useful validation feedback.
- Handle valid submissions with `ngSubmit`, and reset the form and its interaction state.
- Recognize when a form has become complex enough to use reactive forms instead.

## How the pieces fit together

A template-driven form usually has three parts:

1. **The component model** stores the values that the application needs.
2. **The template** declares form controls and binds them to the model.
3. **Angular directives** create and maintain the form controls and aggregate state.

Importing `FormsModule` makes directives such as `NgForm` and `NgModel` available. `NgForm` attaches to a regular `<form>` element and tracks the form's registered controls, aggregate value, and validation status. `NgModel` connects an individual input to the component model and creates a control for it.

The model and the view stay synchronized with two-way binding:

~~~text
component model  <->  [(ngModel)]  <->  input element
~~~

Use one clear source of truth for each value. In this approach, that source is the component property bound by `ngModel`.

## Import FormsModule

A standalone component imports the directives it uses:

~~~ts
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';
import type { NgForm } from '@angular/forms';

@Component({
  selector: 'app-contact-form',
  standalone: true,
  imports: [FormsModule],
  templateUrl: './contact-form.component.html',
  styleUrl: './contact-form.component.css',
})
export class ContactFormComponent {
  model = {
    name: '',
    email: '',
    topic: '',
    message: '',
  };

  submitted = false;
  savedMessage = '';

  save(form: NgForm): void {
    this.submitted = true;

    if (form.invalid) {
      form.control.markAllAsTouched();
      return;
    }

    // Send a copy of the valid data to the application's data layer.
    const submission = { ...this.model };
    console.log(submission);

    this.savedMessage = 'Your message is ready to send.';
  }

  resetForm(form: NgForm): void {
    this.submitted = false;
    this.savedMessage = '';
    this.model = {
      name: '',
      email: '',
      topic: '',
      message: '',
    };
    form.resetForm(this.model);
  }
}
~~~

For a component declared in an NgModule-based application, import `FormsModule` in the NgModule that owns the component instead.

## Bind controls and register them

A form control bound with `[(ngModel)]` inside a parent form needs a `name` attribute:

~~~html
<form #contactForm="ngForm" (ngSubmit)="save(contactForm)">
  <label for="contact-name">Name</label>
  <input
    id="contact-name"
    name="name"
    [(ngModel)]="model.name"
    required
    minlength="2"
    #nameControl="ngModel"
  />

  <button type="submit">Send</button>
</form>
~~~

The `name` is the key Angular uses to register the control with `NgForm`. Use a descriptive, unique name for every registered control in that form. If the name is missing, Angular cannot register the control as expected. If two controls use the same name, they compete for the same entry in the form model.

`#contactForm="ngForm"` creates a template reference to the `NgForm` directive, not to the raw DOM element. Likewise, `#nameControl="ngModel"` exposes the `NgModel` directive for that input. These references let the template inspect values such as `contactForm.valid` and `nameControl.touched`.

If an input is intentionally outside the form's registered control tree, use the standalone option:

~~~html
<input
  name="localSearch"
  [(ngModel)]="searchText"
  [ngModelOptions]="{ standalone: true }"
/>
~~~

A standalone control can still bind to a component property, but it does not contribute to the parent `NgForm` value or validity. Prefer keeping fields that belong to the form inside its registered control tree.

## Track form and control state

Angular updates control and form state as the user interacts with the page.

| State | Meaning |
| --- | --- |
| `valid` / `invalid` | Whether all applicable validation rules pass. |
| `pristine` / `dirty` | Whether the value has changed since it was created or reset. |
| `untouched` / `touched` | Whether the user has blurred the control. |
| `pending` | Whether asynchronous validation is still running. |
| `submitted` | Whether the form has received a submit event. |

The same kinds of state are available on an individual `NgModel` control and on the aggregate `NgForm`. For example, `nameControl.invalid` checks one field, while `contactForm.invalid` checks the form as a whole.

Validation feedback should be helpful without appearing before the user has had a chance to enter a value. A common condition is to show a field error when it is invalid and either touched or when the user has attempted submission:

~~~html
@if (nameControl.invalid && (nameControl.touched || submitted)) {
  <p class="error" id="contact-name-error">
    @if (nameControl.hasError('required')) {
      Enter your name.
    } @else if (nameControl.hasError('minlength')) {
      Use at least two characters.
    }
  </p>
}
~~~

Angular also reflects state in CSS classes on form controls, including `ng-valid`, `ng-invalid`, `ng-pristine`, `ng-dirty`, `ng-untouched`, and `ng-touched`. These can help style controls, but visible text should still explain what the user needs to fix.

## Add validation and accessible feedback

Native validation attributes such as `required`, `minlength`, `maxlength`, `min`, `max`, and `pattern` work with Angular's template-driven validators when used on a control with `ngModel`.

~~~html
<label for="contact-email">Email</label>
<input
  id="contact-email"
  type="email"
  name="email"
  [(ngModel)]="model.email"
  required
  email
  #emailControl="ngModel"
  [attr.aria-invalid]="emailControl.invalid && (emailControl.touched || submitted)"
  [attr.aria-describedby]="emailControl.invalid && (emailControl.touched || submitted)
    ? 'contact-email-help contact-email-error'
    : 'contact-email-help'"
/>
<p id="contact-email-help">We will use this address to reply.</p>

@if (emailControl.invalid && (emailControl.touched || submitted)) {
  <p class="error" id="contact-email-error">
    Enter a valid email address.
  </p>
}
~~~

Associate every input with a visible `<label>` using `for` and `id`. Connect help and error text with `aria-describedby` when it is useful to the user. Do not rely on color alone to communicate an error.

Client-side validation improves the interaction, but it is not a security boundary. Validate submitted data on the server too; users can bypass or modify browser-side checks.

## Handle submission with ngSubmit

Bind the form's `ngSubmit` event to a component method:

~~~html
<form #contactForm="ngForm" (ngSubmit)="save(contactForm)" novalidate>
  <!-- registered controls -->

  <button type="submit" [disabled]="contactForm.invalid">
    Send message
  </button>
</form>
~~~

`ngSubmit` handles submission through the submit button and the form's normal keyboard submission behavior. Use a submit button with `type="submit"`. A separate clear button should use `type="button"` so it does not submit accidentally.

Disabling submission while the form is invalid gives immediate feedback, but keep the validity check in the component method too. It protects the submit path if the handler is called from somewhere else and makes the rule explicit. When submission is invalid, mark controls touched so their messages become visible.

The `novalidate` attribute disables the browser's built-in popup validation UI so the application can present its own messages. Angular still tracks validity through its form directives. Choose a consistent validation experience and make sure the custom feedback is accessible.

## Reset values and state

Changing the model property alone does not always express that the user interaction state should also reset. `NgForm.resetForm(value)` resets the form state and can set a new form value:

~~~ts
resetForm(form: NgForm): void {
  this.model = { name: '', email: '', topic: '', message: '' };
  form.resetForm(this.model);
}
~~~

Use the same initial data shape that the form expects. `resetForm` clears flags such as dirty, touched, and submitted along with the validation state for the reset values. If a reset button is present, use `type="button"` and call this method deliberately.

## Group related controls with NgModelGroup

For related fields, `ngModelGroup` groups controls under one key in the aggregate value:

~~~html
<div ngModelGroup="address">
  <label for="city">City</label>
  <input id="city" name="city" [(ngModel)]="model.address.city" />

  <label for="postal-code">Postal code</label>
  <input id="postal-code" name="postalCode" [(ngModel)]="model.address.postalCode" />
</div>
~~~

The form's value then contains an `address` object with `city` and `postalCode` properties. Keep the names of registered controls unique within their group and keep the component model shape aligned with the values the application expects.

## Common mistakes

- **Importing `CommonModule` but not `FormsModule`:** `ngModel` and `ngForm` come from `FormsModule`.
- **Leaving out `name`:** controls inside a form need a name to register with `NgForm`.
- **Using duplicate control names:** registration keys must identify controls distinctly.
- **Using `(click)` instead of `(ngSubmit)` for the form's main action:** this can skip normal form submission behavior, including keyboard submission.
- **Checking only the button state:** the component handler should also reject invalid form data.
- **Showing errors before interaction:** use control state or a submit-attempt flag to avoid overwhelming users.
- **Assuming client-side validation is sufficient:** repeat validation at the trusted server boundary.
- **Building a very large, dynamic form in the template:** when explicit control creation, dynamic fields, cross-field rules, or extensive testing become central, evaluate reactive forms.

## When to choose another form approach

Template-driven forms are a good fit when the form is small, mostly static, and easy to understand directly in the template. Reactive forms make the control tree and validators explicit in TypeScript and usually scale better for dynamic fields, reusable validation, and complex flows. Signal Forms are another option in Angular versions where they are supported; check the version requirements before using them. See [Choose template-driven, reactive, or signal forms](061-choose-template-driven-reactive-or-signal-forms.md).

## Next steps

- Continue with [Reactive forms with FormControl and FormGroup](063-reactive-forms-with-formcontrol-and-formgroup.md).
- Review the [Q&A](../qnas/062-template-driven-forms-and-ngform.md).
- Work through the [complete example](../examples/062-template-driven-forms-and-ngform.md).

## Official references

- [Template-driven forms](https://angular.dev/guide/forms/template-driven-forms)
- [Forms overview](https://angular.dev/guide/forms)
- [Validate form input](https://angular.dev/guide/forms/form-validation)
- [NgForm API](https://angular.dev/api/forms/NgForm)
- [NgModel API](https://angular.dev/api/forms/NgModel)
