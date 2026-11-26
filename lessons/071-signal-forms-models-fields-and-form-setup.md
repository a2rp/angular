# Signal Forms: models, fields, and form setup

Signal Forms connect a TypeScript data model to HTML controls with Angular signals. You describe the values your form collects, pass that writable signal to `form()`, and bind its fields in the template with `[formField]`.

This lesson covers the foundation: the model, the generated field tree, standalone component imports, and native control bindings. Validation rules and schemas are covered separately.

> **Version note:** Signal Forms are available in Angular v21 and later. The current Angular API reference marks `form()` and `FormField` stable since Angular v22. Check the version used by your project before adopting them.

## Learning outcomes

After this lesson, you should be able to:

- Define a typed form model with all of its initial values.
- Create a writable model signal with `signal()`.
- Create a field tree with `form()` and explain how it mirrors the model.
- Import `FormField` and bind native controls with `[formField]`.
- Read current form data from the model and field state.
- Organize nested fields and choose useful empty values.
- Recognize common setup errors and keep form input separate from domain data when needed.

## The three pieces

A basic Signal Form has three related pieces:

1. **Model signal:** the writable signal stores the form's data.
2. **Field tree:** `form(model)` creates typed fields matching the model's object and array structure.
3. **Template binding:** `FormField` connects each native control to a field and synchronizes values in both directions.

For example, a model with `name` and `email` properties produces fields that can be accessed as `contactForm.name` and `contactForm.email`.

The model is the source of truth. Signal Forms does not create a separate data copy that must later be synchronized.

## Define a clear model

Create a TypeScript type that represents the information collected by the UI:

~~~ts
interface ContactFormModel {
  name: string;
  email: string;
  message: string;
  preferredContact: 'email' | 'phone';
  updatesAllowed: boolean;
}
~~~

Then initialize every field in a writable signal:

~~~ts
contactModel = signal<ContactFormModel>({
  name: '',
  email: '',
  message: '',
  preferredContact: 'email',
  updatesAllowed: false,
});
~~~

Initial values should match the type and the control that will edit them:

- Text inputs and textareas normally start with an empty string.
- Checkboxes use booleans.
- A select should start with a value represented by one of its options.
- Optional data should have a deliberate empty representation, such as `null`, when the control and model support it.

Fields that are missing from the model do not appear in the field tree. A property initialized to `undefined` is also excluded. Do not expect `form()` to invent a field later; include the property in the initial model.

## Create the field tree

Import `signal` from Angular and `form` from Signal Forms:

~~~ts
import {Component, signal} from '@angular/core';
import {form} from '@angular/forms/signals';
~~~

Create the form from the model signal:

~~~ts
contactForm = form(this.contactModel);
~~~

The field tree follows the model shape and is type checked. If the model has `contactForm.email`, TypeScript understands that field. Trying to access a property that is not in the model should produce a type error.

A field tree node can be used in two ways:

- Access a child through its property, such as `contactForm.email`.
- Call a node to read its current state, such as `contactForm.email()`.

The state exposes reactive values such as `value()`, `valid()`, `invalid()`, `touched()`, and `dirty()`. The model signal is often the simplest place to read the complete data object.

## Bind native controls

Import `FormField` in the component and add it to the standalone component's `imports`:

~~~ts
@Component({
  selector: 'app-contact-form',
  imports: [FormField],
  template: `...`,
})
export class ContactFormComponent {
  // model and field tree
}
~~~

Bind controls to the corresponding field tree nodes:

~~~html
<input type="text" [formField]="contactForm.name" />
<input type="email" [formField]="contactForm.email" />
<textarea [formField]="contactForm.message"></textarea>
~~~

The `[formField]` directive synchronizes user input with the model and model changes with the control. You do not need to write a separate `(input)` handler or subscribe just to copy the value into a component property.

For standard controls, use a native control whose semantics match the data:

~~~html
<select [formField]="contactForm.preferredContact">
  <option value="email">Email</option>
  <option value="phone">Phone</option>
</select>

<input type="checkbox" [formField]="contactForm.updatesAllowed" />
~~~

Keep labels associated with controls. Binding a field does not replace semantic HTML, a visible label, helpful instructions, or accessible error messaging.

## Read and update form data

Read the whole current object by calling the model signal:

~~~ts
const submittedContact = this.contactModel();
~~~

Programmatic updates can use the writable signal API:

~~~ts
this.contactModel.update((current) => ({
  ...current,
  name: 'Asha Rao',
}));
~~~

You can also read a field's current value from its state:

~~~ts
const currentEmail = this.contactForm.email().value();
~~~

Both reads reflect the synchronized form state. Use the model signal when you need a submission-shaped object; use a field state when the UI needs information about that specific field.

## Nested models

Nested plain objects create nested fields:

~~~ts
interface ProfileFormModel {
  name: string;
  address: {
    city: string;
    postalCode: string;
  };
}

profileForm.address.city;
~~~

A template can bind the nested field directly:

~~~html
<input type="text" [formField]="profileForm.address.city" />
~~~

Use nesting when the fields are a meaningful group or the collected data naturally has that shape. The structural objects and arrays in a Signal Forms model should be plain JavaScript objects and arrays. Avoid using class instances, `Map`, or `Set` as the structural layer of the form model. Convert to or from a domain object at the boundary when necessary.

## Signal Forms and other form APIs

Signal Forms use `signal()` to hold form data and `form()` to create the field tree. This `form()` function is different from the HTML `<form>` element and from Reactive Forms APIs such as `FormGroup`, `FormControl`, and `formControlName`.

Keep one form approach clear in a given form:

- **Signal Forms:** bind a field tree using `[formField]`.
- **Reactive Forms:** bind controls using `[formControl]` or `formControlName`.
- **Template-driven forms:** commonly bind using `[(ngModel)]`.

Do not add `formControlName` to a Signal Form field as if the APIs were interchangeable. When a legacy form system needs integration, follow Angular's current custom-control guidance.

## Common setup mistakes

| Mistake | Why it fails | Better approach |
| --- | --- | --- |
| Omitting a property from the initial model | No corresponding field exists in the generated tree | Initialize every field you need |
| Using `undefined` for an intended field | Undefined properties are excluded from the tree | Choose a concrete empty value, often `''`, `false`, or `null` |
| Forgetting `FormField` in component imports | The template cannot resolve the directive | Import it from `@angular/forms/signals` |
| Binding a misspelled or nonexistent field | The model's type does not include that field | Keep the interface and template field names aligned |
| Using a class instance, `Map`, or `Set` as model structure | These are not supported as the tree's structural layer | Use plain objects and arrays |
| Copying values manually in every input event | Duplicates the synchronization the directive already performs | Bind fields with `[formField]` |
| Mixing Signal Forms and Reactive Forms directives | The two APIs have different form models and bindings | Choose the form API intentionally for each form |

## Setup checklist

Before adding validation or submission behavior, confirm that:

- The form model describes the data the user edits.
- Every intended field has an initial value.
- The model signal is writable and passed to `form()`.
- The component imports `FormField`.
- Every control binds to the correct field tree node.
- Input types, labels, and empty values match the data.
- The project uses a compatible Angular version.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="070-custom-controls-with-controlvalueaccessor.md">070 - Custom controls with ControlValueAccessor</a></td><td><a href="../qnas/071-signal-forms-models-fields-and-form-setup.md">Q&amp;A</a></td><td><a href="../examples/071-signal-forms-models-fields-and-form-setup.md">Example</a></td><td><a href="072-signal-forms-schemas-validation-and-custom-controls.md">072 - Signal Forms: schemas, validation, and custom controls</a></td>
</tr>
  </tbody>
</table>
