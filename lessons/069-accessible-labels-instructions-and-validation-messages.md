# Accessible labels, instructions, and validation messages

A form is easier to complete when each control has a clear name, its instructions are available when needed, and errors explain how to recover. These relationships should work visually, by keyboard, and with assistive technology.

Angular tracks form values and validation state. It does not automatically make a custom form accessible: the template must provide labels, descriptions, and error relationships.

## Learning outcomes

After this lesson, you should be able to:

- Associate a visible **label** with each input, select, textarea, and other form control.
- Give related controls a group name with **fieldset** and **legend**.
- Provide instructions for formats, required fields, and other constraints.
- Connect instructions and errors with **aria-describedby**.
- Use Angular validation state to set **aria-invalid** and reveal useful messages.
- Choose when errors appear and how a form-level error summary behaves.
- Avoid relying on placeholders, color, or an asterisk as the only explanation.

## Give every control a clear label

A label tells people what information a control expects. Use a visible HTML **label** and associate it with the control. Explicit association uses a matching **for** and **id**:

~~~html
<label for="email">Email address</label>
<input id="email" name="email" type="email" />
~~~

When the user activates the label, focus moves to the associated input. Assistive technology can also announce the label as the input's name. The **id** must be unique in the document, and the label's **for** value must match it exactly.

Wrapping a control in a label is another valid association:

~~~html
<label>
  Email address
  <input name="email" type="email" />
</label>
~~~

Use one clear label per control. A placeholder is not a label: it can disappear as someone types and is not consistently announced as the control's name. A **title** attribute is not a reliable replacement either.

A label should identify the field's purpose, not just its current value or appearance. For example, **“Email address”** is clearer than **“Your details”**.

## Name related controls with fieldset and legend

When several controls form one logical question, group them with **fieldset** and give the group a **legend**:

~~~html
<fieldset>
  <legend>Preferred contact method</legend>

  <label>
    <input type="radio" name="contactMethod" value="email" />
    Email
  </label>

  <label>
    <input type="radio" name="contactMethod" value="phone" />
    Phone
  </label>
</fieldset>
~~~

The legend gives the radio options a shared context. Each individual option still needs its own label. The same pattern is useful for related checkboxes, address sections, and other groups of controls.

In Angular reactive forms, **formControlName** connects each input to the model; it does not create the label or group semantics. Keep both responsibilities in the template.

## Provide instructions before they are needed

Tell users about relevant constraints before they enter a value. Examples include:

- Which fields are required and which are optional.
- The accepted date or identifier format.
- A password's length or character requirements.
- Whether a value is limited to a particular range.

Put whole-form guidance before the form or near its heading. Put field-specific guidance beside the field and associate it with **aria-describedby**:

~~~html
<label for="start-date">Start date (required)</label>
<p id="start-date-help">Use the format YYYY-MM-DD.</p>

<input
  id="start-date"
  type="text"
  required
  aria-describedby="start-date-help"
/>
~~~

**aria-describedby** accepts one or more element IDs separated by spaces. The referenced descriptions are announced in addition to the control's accessible name. Keep the text concise and relevant.

Do not use a placeholder as the only source of a format example or instruction. If the placeholder is useful as an example, repeat essential guidance in persistent text associated with the field.

## Represent required fields clearly

Make required status clear in both the visible text and the form semantics:

~~~html
<label for="email">Email address (required)</label>
<input id="email" type="email" required />
~~~

For a Reactive Form, also put the corresponding validator in the model:

~~~ts
readonly form = this.fb.nonNullable.group({
  email: ['', [Validators.required, Validators.email]],
});
~~~

The HTML **required** attribute communicates required status to browsers and assistive technology. Angular's **Validators.required** calculates Angular form validity. They serve related but distinct jobs, so keep them consistent.

If most fields are required, a short instruction such as **“All fields are required unless marked optional”** can reduce repeated wording, but make each field's status understandable when encountered. Do not indicate required status only with color or an unexplained symbol.

## Connect Angular errors to their controls

An error should identify the field and say what needs to change. Connect the error's element to the field using **aria-describedby**. Set **aria-invalid** when you are showing that field as invalid:

~~~html
<label for="email">Email address (required)</label>
<p id="email-help">Use an address such as name@example.com.</p>

<input
  id="email"
  type="email"
  formControlName="email"
  required
  aria-describedby="email-help email-error"
  [attr.aria-invalid]="showEmailError ? 'true' : null"
/>

@if (showEmailError) {
  <p id="email-error">Enter a valid email address.</p>
}
~~~

The validator belongs in the form model:

~~~ts
readonly email = this.fb.nonNullable.control('', [
  Validators.required,
  Validators.email,
]);
~~~

The example uses **aria-describedby** so that focusing the field exposes the help text and, when present, its error. The error ID should be unique. If the template references an error ID only while the error is visible, ensure the association is present whenever that message is rendered.

Use text to identify the problem. A red border or icon can reinforce an error, but color or an icon alone does not explain it. Prefer actionable messages such as **“Enter a valid email address, for example name@example.com.”** over a generic **“Invalid input.”**

## Choose when to show errors

Showing every required-field error on initial page load can make a form noisy. A common pattern is:

- Before interaction, show instructions but not an error for an untouched field.
- After blur, show the field's error if it is invalid.
- On a submit attempt, mark all controls touched so errors are discoverable.
- While the user corrects a field, update its message and **aria-invalid** state.

Reactive Forms exposes state such as **touched**, **dirty**, **invalid**, and **pending**. A helper can centralize the rule:

~~~ts
submitted = false;

showError(control: AbstractControl): boolean {
  return control.invalid && (control.touched || this.submitted);
}

submit(): void {
  this.submitted = true;
  this.form.markAllAsTouched();

  if (this.form.invalid || this.form.pending) {
    return;
  }

  // Continue with the valid form.
}
~~~

Only mark a field **aria-invalid="true"** when your UI has identified an error for the user. An untouched required field may be invalid in the model before the user has had a chance to respond; that does not require announcing an error immediately.

If a field has an asynchronous validator, account for **pending** separately from invalidity. Explain that the check is in progress, and do not show a final “available” or “unavailable” message until the check completes.

## Add a form-level error summary when it helps

Long forms may benefit from a summary after a submit attempt. A useful summary tells people that there are errors and provides links to the affected fields:

~~~html
@if (submitted && form.invalid) {
  <section
    id="error-summary"
    tabindex="-1"
    aria-labelledby="error-summary-title"
  >
    <h2 id="error-summary-title">Review these fields</h2>
    <ul>
      @if (form.controls.email.invalid) {
        <li><a href="#email">Enter a valid email address.</a></li>
      }
      @if (form.controls.startDate.invalid) {
        <li><a href="#start-date">Enter a start date in YYYY-MM-DD format.</a></li>
      }
    </ul>
  </section>
}
~~~

For a form that needs explicit navigation after failed submission, move focus to the summary after it appears or focus the first invalid field. Then users can find and correct the problem without searching the whole page. Keep the summary and inline messages consistent; do not create two competing descriptions with different advice.

Short forms may need only clear inline errors. Choose the pattern that makes errors easy to find in the context of the page.

## Use live announcements with care

A message associated through **aria-describedby** is available when the user focuses the field. A live region is useful when important feedback changes without a focus move, such as a save result or an async status update:

~~~html
<p role="status" aria-live="polite">{{ saveMessage }}</p>
~~~

For an urgent failure that needs immediate attention, an alert may be appropriate. Avoid making every keystroke or every validation update an assertive announcement; repeated interruptions make forms harder to use. Choose one primary announcement path for each state change.

## Keep labels and IDs unique in repeated controls

When rendering repeated fields with **@for**, make each control ID unique and derive its label, help, and error IDs from the same stable item identifier:

~~~html
@for (phone of phones.controls; track phone; let i = $index) {
  <label [for]="'phone-' + i">Phone number {{ i + 1 }}</label>
  <input [id]="'phone-' + i" [formControlName]="i" />
}
~~~

For lists where rows can be reordered or removed, prefer a stable row key over the current array index if the UI identity needs to remain consistent. Duplicate IDs can cause a label or description to point to the wrong field.

## Accessibility checklist

Review each form and confirm that:

- Every control has a meaningful accessible name, normally through a visible **label**.
- Related radio buttons or checkboxes have a **fieldset** and **legend** when they form one question.
- Instructions and required status are available before the user needs them.
- A placeholder is not the only label or instruction.
- Field help and errors are programmatically associated with the correct control.
- Errors are text, specific, and explain how to recover.
- Angular validators and HTML required semantics agree.
- Error timing does not expose every untouched field as an error on initial load.
- **aria-invalid** reflects the error currently presented to the user.
- Submit errors can be found and navigated to, with focus handled where appropriate.
- Live announcements are helpful and not repetitive.
- Repeated fields have unique IDs and matching label relationships.

## Further reading

- [Angular reactive forms](https://angular.dev/guide/forms/reactive-forms)
- [Angular form validation](https://angular.dev/guide/forms/form-validation)
- [W3C WAI: Labeling controls](https://www.w3.org/WAI/tutorials/forms/labels/)
- [W3C WAI: Form instructions](https://www.w3.org/WAI/tutorials/forms/instructions/)
- [W3C WAI: Validating input](https://www.w3.org/WAI/tutorials/forms/validation/)
- [W3C WAI: User notification](https://www.w3.org/WAI/tutorials/forms/notifications/)
