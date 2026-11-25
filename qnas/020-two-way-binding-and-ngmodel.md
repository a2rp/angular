# Q&A: Two-Way Binding and ngModel

## Two-way binding basics

### 1. What is two-way binding?

It synchronizes a value in both directions: the component supplies a value to the view, and changes made in the view are sent back to the component.

### 2. What does Angular's [()] syntax mean?

It combines property binding with event binding. Angular developers often call it "banana in a box" because the parentheses surround the square brackets.

### 3. What does [(ngModel)] do?

It connects a form control's value to a component property and keeps them synchronized. User edits update the property, and programmatic property changes update the control.

### 4. What is the long form of [(ngModel)]="name"?

Conceptually, it is:

~~~html
<input [ngModel]="name" (ngModelChange)="name = $event">
~~~

The property binding sends the value to the control; ngModelChange sends changes back.

### 5. Does two-way binding update just the component or just the control?

It synchronizes both. It is not a one-time initialization.

### 6. Does the expression on the right side of [(ngModel)] need to be writable?

Yes. Use an assignable property or field, such as name or user.email. A literal or a calculation such as name.toUpperCase() cannot receive the new value.

### 7. Is ngModel the only two-way binding Angular supports?

No. Angular also supports component-level two-way binding with model inputs. That is a different API from FormsModule and ngModel.

## Setup and control types

### 8. Which package exports FormsModule?

FormsModule is exported by @angular/forms. Import it into the template scope that uses ngModel.

### 9. Where does a standalone component import FormsModule?

In that component's imports array:

~~~ts
@Component({
  imports: [FormsModule],
})
export class ProfileEditor {}
~~~

### 10. Where does an NgModule-based app import FormsModule?

In the imports array of the NgModule whose template scope needs the form directives.

### 11. What happens if I forget FormsModule?

Angular cannot resolve template-driven form directives such as ngModel in that template and reports a template compilation error.

### 12. Which controls work with ngModel?

Common native controls include text inputs, checkboxes, radio controls, selects, and textareas. Angular connects them through value accessors.

### 13. What kind of value does a text input normally bind?

A string. The initial component value and values read from the control should match the control's intended type.

### 14. Is a checkbox value a string or a boolean?

For checked state, a checkbox's ngModel value is boolean. Use a boolean property for [(ngModel)].

### 15. How do I bind an object as a select option's value?

Angular provides ngValue for option values that are not ordinary strings, such as object values. Use the select and option binding supported by Angular forms rather than relying on HTML attribute string conversion.

## Forms and registration

### 16. What is NgForm?

When FormsModule is in scope, Angular attaches NgForm to a form element. It tracks registered controls and aggregate form value, validity, and interaction state.

### 17. Does a control inside a form need a name?

A registered ngModel control inside a parent form needs a name so Angular can register it under a key in the form value. Alternatively, mark it standalone if it should not register.

### 18. What does the name attribute do?

It identifies the registered control within the parent form and becomes its key in the form's value object. Use a meaningful, unique name.

### 19. Does ngModel outside a form need a name?

It does not need a form-registration name when no parent form is present. A label still needs a proper association with its control.

### 20. What does [ngModelOptions]="{ standalone: true }" do?

It keeps that ngModel control out of its parent NgForm's registered controls and form value. The control can still synchronize with its component property.

### 21. Does ngModelOptions standalone mean the Angular component is standalone?

No. It means only that this control does not register with the parent form. Component standalone configuration is a separate concept.

### 22. Can I set the NgModel name through ngModelOptions?

Yes. NgModel options support a name option. For a native form control, the ordinary name attribute is usually clearer.

### 23. How do I handle a template-driven form submission?

Use Angular's ngSubmit event on the form, such as (ngSubmit)="save(profileForm)". A submit button triggers it, and native form submission behavior is handled by Angular's form directive.

## Control state and validation

### 24. How do I expose a control's NgModel state?

Export the directive into a template variable:

~~~html
<input name="email" [(ngModel)]="email" #emailControl="ngModel">
~~~

Then read properties such as emailControl.valid or emailControl.touched.

### 25. What is the difference between dirty and touched?

Dirty generally means the user changed the value. Touched means the user focused the control and then left it. They describe different kinds of interaction.

### 26. What do valid and invalid mean?

They report whether the control passes or fails its currently configured validators. Without validators, a control can be valid even when an application might consider its value incomplete.

### 27. Can I inspect the whole form's value and validity?

Yes. Export NgForm from the form with a template variable, then read properties such as form.value, form.valid, form.dirty, and form.submitted.

### 28. Does [(ngModel)] automatically validate my business rules?

No. Two-way binding synchronizes values. Add appropriate validators and application-level validation for the requirements of the form.

### 29. When should I show a validation message?

Usually after the user has interacted with the control, for example when it is touched or dirty and invalid. This avoids presenting errors before the user can reasonably act.

## Update behavior and custom handling

### 30. When does ngModel update by default?

The default updateOn value is change. Angular's NgModel options also support blur and submit.

### 31. What does updateOn: 'blur' change?

The control's value and validity update when it loses focus instead of on each change. This can delay previews and validation feedback.

### 32. What does updateOn: 'submit' change?

It delays updates until the parent form is submitted. Consider whether the component also needs to react to edits before submission.

### 33. How can I run custom code when a value changes?

Use [ngModel] and (ngModelChange) separately, then handle the emitted value in a method. The handler should assign the component property if it owns synchronization.

### 34. Should I use [(ngModel)] for every input?

No. Use it when straightforward two-way synchronization is useful. One-way property binding, event binding, or a form API may better express the intended data flow in other cases.

### 35. Can a custom Angular component use ngModel automatically?

No. A custom form control needs a ControlValueAccessor integration so Angular Forms knows how to read and write its value and disabled state.

## Choosing form APIs

### 36. Is ngModel part of reactive forms?

No. ngModel belongs to template-driven forms. Reactive forms define controls explicitly and use APIs such as FormControl and formControlName.

### 37. Can I combine ngModel with formControlName on one element?

Do not do so. Combining ngModel with reactive form directives on the same control is deprecated. Choose one form approach for that control.

### 38. Is FormsModule the same as ReactiveFormsModule?

No. They expose different form APIs. FormsModule is for template-driven forms; ReactiveFormsModule is for reactive forms.

### 39. What should I debug if the view and component value disagree?

Check that FormsModule is in the correct template scope, the expression is writable, the control has the right name inside a form, and updateOn is not delaying synchronization.

### 40. What is the key idea to remember?

[(ngModel)] is convenient two-way synchronization for template-driven form controls. Import FormsModule, register form controls intentionally, and keep it separate from reactive form directives.

## Further reading

- [Angular two-way binding](https://angular.dev/guide/templates/two-way-binding)
- [Angular template-driven forms](https://angular.dev/guide/forms/template-driven-forms)
- [NgModel API](https://angular.dev/api/forms/NgModel)
- [NgForm API](https://angular.dev/api/forms/NgForm)
- [Angular forms overview](https://angular.dev/guide/forms)
