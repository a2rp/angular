# 20. Two-way binding and ngModel

Two-way binding keeps a value synchronized in both directions: the component supplies the current value to the view, and user changes are sent back to the component.

Angular writes this syntax with square brackets inside parentheses:

~~~html
<input [(ngModel)]="displayName">
~~~

The Angular community often calls this "banana in a box." For native form controls, ngModel from FormsModule connects the control to component data and tracks form-control state.

## Learning goals

After this lesson, you should be able to:

- Explain the two directions of a two-way binding.
- Read and write the [()] syntax.
- Use FormsModule and [(ngModel)] in a standalone component.
- Register controls with an Angular template-driven form using name.
- Use standalone ngModel controls and updateOn options appropriately.
- Read NgModel and NgForm values and interaction states.
- Choose between template-driven and reactive forms without mixing their APIs.

## 1. How two-way binding works

Suppose a component has a mutable displayName property:

~~~ts
displayName = 'Ada';
~~~

A two-way binding connects it to an input:

~~~html
<input [(ngModel)]="displayName">
<p>Hello, {{ displayName }}!</p>
~~~

The value flows in both directions:

1. Angular writes displayName into the control.
2. The user edits the control.
3. ngModel reports the new value.
4. Angular assigns the new value to displayName.
5. Other template bindings that read displayName show the updated value.

If application code changes displayName, Angular updates the control too. The component property and the control's view value stay synchronized.

## 2. The syntax is shorthand

Two-way binding combines property binding and event binding. This:

~~~html
<input [(ngModel)]="displayName">
~~~

is conceptually equivalent to:

~~~html
<input
  [ngModel]="displayName"
  (ngModelChange)="displayName = $event"
>
~~~

The property side supplies the current value. The ngModelChange event sends an updated value back. The target expression must be writable, such as a component field or a property on a component object.

Use [()] when the normal synchronization is what you want. Use separate property and event bindings when you need custom handling between the value arriving and the model being updated.

## 3. Import FormsModule

A standalone component must import FormsModule for ngModel:

~~~ts
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-profile-editor',
  standalone: true,
  imports: [FormsModule],
  templateUrl: './profile-editor.html',
})
export class ProfileEditor {
  displayName = 'Ada';
}
~~~

An NgModule-based application imports FormsModule in the NgModule that declares the component. Angular standalone components and NgModules have different import locations; use the one that matches your application.

Importing FormsModule makes template-driven form directives such as ngModel and ngForm available in that template scope.

## 4. Bind controls with ngModel

Text input example:

~~~html
<label for="display-name">Display name</label>
<input
  id="display-name"
  name="displayName"
  [(ngModel)]="displayName"
>
<p>Preview: {{ displayName }}</p>
~~~

ngModel works with standard form controls that have an Angular value accessor, including text inputs, checkboxes, radio controls, selects, and textareas.

For a checkbox, the model value is boolean:

~~~html
<label>
  <input type="checkbox" name="subscribed" [(ngModel)]="subscribed">
  Receive updates
</label>
~~~

Use a value type that matches the control. A checkbox naturally represents checked or unchecked state; a text input naturally supplies text.

## 5. Register controls in a template-driven form

When ngModel is inside a form, give the control a name:

~~~html
<form #profileForm="ngForm" (ngSubmit)="saveProfile(profileForm)">
  <label for="display-name">Display name</label>
  <input
    id="display-name"
    name="displayName"
    [(ngModel)]="displayName"
    required
  >

  <button type="submit">Save</button>
</form>
~~~

With FormsModule in scope, Angular attaches an NgForm to the form element. Each non-standalone ngModel control registers with that form under its name. The form tracks its aggregate value, validity, and interaction state.

The name is the control's key in the form value. Use meaningful, unique names within a form:

~~~ts
{
  displayName: 'Ada'
}
~~~

Without a name, an ngModel control inside a form cannot register normally. Give it a name or explicitly make it standalone when it should not be part of the parent form.

## 6. Standalone controls inside a form

A control can use ngModel without participating in its parent form:

~~~html
<input
  type="checkbox"
  [(ngModel)]="showAdvanced"
  [ngModelOptions]="{ standalone: true }"
>
~~~

This control still synchronizes with showAdvanced, but it is excluded from the parent NgForm's value and validation state. Use this for presentation settings or other values that should not be submitted as part of the form.

The Angular word standalone here means "not registered with the parent form." It is separate from whether the component itself is standalone.

## 7. Inspect control and form state

Export the NgModel directive into a template variable:

~~~html
<input
  name="email"
  type="email"
  [(ngModel)]="email"
  required
  email
  #emailControl="ngModel"
>

<p>Valid: {{ emailControl.valid }}</p>
<p>Touched: {{ emailControl.touched }}</p>
~~~

The exported directive exposes common control state, including valid, invalid, pending, pristine, dirty, touched, and untouched. A named local variable is associated with the NgModel directive, not the raw input element.

You can also export the form directive:

~~~html
<form #profileForm="ngForm">
  <!-- registered controls -->
</form>

<p>Form valid: {{ profileForm.valid }}</p>
<p>Form submitted: {{ profileForm.submitted }}</p>
~~~

Use these states to show useful feedback. Avoid showing every validation error before the user has had a chance to interact with the field.

## 8. Change update timing when appropriate

By default, ngModel updates on the control's change interaction. The updateOn option can use blur or submit instead:

~~~html
<input
  name="search"
  [(ngModel)]="searchText"
  [ngModelOptions]="{ updateOn: 'blur' }"
>
~~~

Supported values are change, blur, and submit. Choose deliberately: delaying updates can affect previews, validation messages, and code that reads the component property while the user is editing.

For an individual control outside the parent's form, use ngModelOptions standalone as described above. The options object can also set the ngModel name when needed, though the ordinary name attribute is clearer for native controls.

## 9. Respond to changes explicitly

Sometimes a value change should trigger additional behavior. Use the ngModel input and ngModelChange output separately:

~~~html
<input
  [ngModel]="displayName"
  (ngModelChange)="updateDisplayName($event)"
>
~~~

~~~ts
updateDisplayName(value: string): void {
  this.displayName = value.trimStart();
}
~~~

The handler owns the update, so assign the component property there. This is useful for normalization or other deliberate logic. Keep validation and domain rules clear; do not use two-way binding as a substitute for validation.

## 10. Two-way component bindings are a separate feature

Angular also supports two-way bindings between parent and child components using model inputs. That API uses model() in the child component and a matching input/output name. It is distinct from ngModel and FormsModule, and is covered in the component API section.

Do not assume that [(ngModel)] is required for every component binding, or that a custom component automatically works with ngModel. A custom form control needs an Angular ControlValueAccessor integration.

## 11. Template-driven and reactive forms

ngModel belongs to Angular's template-driven forms. In a template-driven form, the template declares controls and Angular creates and manages their control instances.

Reactive forms define controls explicitly in TypeScript and bind them with APIs such as formControlName. Use the approach that fits the form and keep the APIs consistent. Do not combine ngModel with formControl or formControlName on the same control; that integration is deprecated.

This chapter introduces synchronization and ngModel. Validation rules, custom value accessors, and reactive form design are handled in their own lessons.

## Common mistakes

- Forgetting to import FormsModule in the component or NgModule scope.
- Using [(ngModel)] with a component expression that cannot be assigned to.
- Omitting name from a registered ngModel control inside a form.
- Assuming ngModelOptions standalone means the component is not standalone.
- Expecting a standalone ngModel control to appear in the parent form's value.
- Using the string value from a checkbox instead of its boolean model.
- Mixing ngModel with formControl or formControlName on one control.
- Using two-way binding where a one-way binding is sufficient.
- Expecting the view to update after mutating unrelated state in a way Angular cannot observe.

## Practical checklist

1. Is this a template-driven form using FormsModule?
2. Does a standalone component import FormsModule?
3. Does each registered control inside a form have a meaningful name?
4. Is a non-form presentation control intentionally marked standalone?
5. Does the model type match the native control's value?
6. Do I need straightforward two-way synchronization, or custom change logic?
7. Am I reading the right NgModel or NgForm state?
8. Am I keeping template-driven and reactive form APIs separate?

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="019-event-bindings-event-objects-and-keyboard-input.md">019 - 19. Event bindings, event objects, and keyboard input</a></td><td><a href="../qnas/020-two-way-binding-and-ngmodel.md">Q&amp;A</a></td><td><a href="../examples/020-two-way-binding-and-ngmodel.md">Example</a></td><td><a href="021-built-in-control-flow-with-if-and-switch.md">021 - 21. Built-in control flow with if and switch</a></td>
</tr>
  </tbody>
</table>
