# Custom controls with ControlValueAccessor

A component can contain an interactive widget that should behave like one Angular form control: a rating picker, date-range selector, phone-number editor, or token input. **ControlValueAccessor** (CVA) is the bridge between that component's UI and Angular Forms.

A component that only implements CVA participates in value, touched, and disabled-state synchronization. It does not automatically provide custom validation, accessible labels, or error messages; those need their own design.

## Learning outcomes

After this lesson, you should be able to:

- Explain the four **ControlValueAccessor** methods and their direction of data flow.
- Register a component through **NG_VALUE_ACCESSOR**.
- Keep programmatic model updates separate from user-originated changes.
- Report touched state at the right boundary for a composite widget.
- Synchronize the form control's disabled state with every interactive child.
- Use the custom component with **formControlName** or **formControl**.
- Identify when a separate **Validator** implementation or accessibility inputs are needed.

## The CVA bridge

Angular Forms talks to a control through four methods:

| Method | Direction | Responsibility |
| --- | --- | --- |
| **writeValue(value)** | Model to view | Render a value supplied by Angular |
| **registerOnChange(callback)** | Angular gives callback to view | Save the callback and call it after a user changes the value |
| **registerOnTouched(callback)** | Angular gives callback to view | Save the callback and call it when the control is touched or blurred |
| **setDisabledState(disabled)** | Model state to view | Enable or disable the widget's interactive UI |

The interface is intentionally small. Angular provides the callbacks; the component stores and calls them at the appropriate time.

## Implement model-to-view updates

**writeValue()** is called when Angular needs the custom view to reflect a model value, including values set programmatically and form resets:

~~~ts
value: number | null = null;

writeValue(value: number | null): void {
  this.value = value;
}
~~~

For an **OnPush** component, call **markForCheck()** after changing component state from **writeValue()** so the new value is rendered:

~~~ts
writeValue(value: number | null): void {
  this.value = value;
  this.changeDetector.markForCheck();
}
~~~

Do not call the registered **onChange** callback from **writeValue()**. This method is model-to-view synchronization. Calling **onChange** here would report a programmatic update as if the user had changed the control and can create unnecessary or confusing value events.

## Report view-to-model changes

Angular calls **registerOnChange()** during setup. Store the callback and call it when a genuine user action changes the widget's value:

~~~ts
private onChange: (value: number | null) => void = () => {};

registerOnChange(fn: (value: number | null) => void): void {
  this.onChange = fn;
}

selectRating(rating: number): void {
  if (this.disabled || this.value === rating) {
    return;
  }

  this.value = rating;
  this.onChange(rating);
}
~~~

Initialize callbacks to no-op functions. That makes the component safe before Angular has registered the callbacks and avoids non-null assertions.

Only call **onChange** for a user-originated value change. Do not call it for **writeValue()**, rendering, initialization, or a disabled interaction.

## Report touched state at the right boundary

Angular calls **registerOnTouched()** with a callback. Call that callback when the user has finished interacting with the control, commonly when focus leaves it:

~~~ts
private onTouched: () => void = () => {};

registerOnTouched(fn: () => void): void {
  this.onTouched = fn;
}

handleFocusOut(event: FocusEvent): void {
  const nextTarget = event.relatedTarget as Node | null;

  // A rating widget has several buttons. Moving between them is still
  // interaction inside one form control.
  if (nextTarget && this.host.nativeElement.contains(nextTarget)) {
    return;
  }

  this.onTouched();
}
~~~

For a composite control with multiple inner buttons or inputs, do not mark it touched every time focus moves from one inner element to another. Treat the widget as one control and report touched when focus leaves the whole widget. This matters when the parent form uses **updateOn: 'blur'**.

A simple control with one native input can usually report touched from that input's **blur** event.

## Synchronize disabled state

Angular calls **setDisabledState()** when the form control is disabled or enabled:

~~~ts
disabled = false;

setDisabledState(isDisabled: boolean): void {
  this.disabled = isDisabled;
  this.changeDetector.markForCheck();
}
~~~

Bind the state to the actual interactive elements. Disabling only a wrapper's appearance is not enough:

~~~html
<button type="button" [disabled]="disabled">Choose rating</button>
~~~

For a composite widget, disable every interactive child or a native container such as a **fieldset** whose disabled state applies to its form controls. Do not let the UI accept values while its Angular control is disabled.

Prefer **formControl.disable()** and **formControl.enable()** to keep the form model authoritative. A separate component input that independently changes disabled state can drift from the parent control.

## Register the accessor provider

Angular needs to discover the component as a value accessor. Provide it through **NG_VALUE_ACCESSOR** with **multi: true** and **useExisting**:

~~~ts
import { forwardRef } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

@Component({
  selector: 'app-rating-control',
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => RatingControlComponent),
      multi: true,
    },
  ],
})
export class RatingControlComponent implements ControlValueAccessor {
  // Implement the CVA methods.
}
~~~

**useExisting** points to this component instance. **forwardRef()** lets the provider refer to the class declared below the decorator. **multi: true** adds the accessor to Angular's collection rather than replacing other accessors.

If a custom component has no registered accessor, binding it with **formControlName** commonly produces a “No value accessor” error.

## Use the component from a parent form

After the component provides a CVA, a parent can bind it like a native control:

~~~ts
readonly feedbackForm = this.fb.group({
  rating: this.fb.control<number | null>(null, Validators.required),
});
~~~

~~~html
<form [formGroup]="feedbackForm">
  <app-rating-control formControlName="rating"></app-rating-control>
</form>
~~~

The parent **FormControl** owns the value, validation status, and interaction state. The custom component renders the value and reports user input and touched events through the callbacks.

The same accessor supports **[formControl]** and **ngModel** too, as long as the component is used with the matching Angular forms directives and imports.

## Understand validation and accessibility boundaries

CVA is not a validator interface. If the widget has its own validation rule, implement **Validator** and register **NG_VALIDATORS**, or validate the parent **FormControl** in the parent model. Keep the rule in one clear place.

CVA also does not automatically create an accessible name or expose the parent's validation errors. A custom widget still needs appropriate internal semantics, such as:

- A **fieldset** and **legend** for a group of related choices.
- Native buttons or inputs with meaningful names and keyboard behavior.
- A clear disabled state on its interactive elements.
- A label, required state, and help/error text when the widget needs them.
- An input API for a parent-provided error when that message is rendered inside the custom widget.

A **formControlName** attribute on a custom component connects Angular's model; it does not label the internal buttons. For a rating picker, for example, each choice might be a native button with an accessible name and **aria-pressed**, grouped under a **fieldset** and **legend**.

Do not make a generic custom widget imitate a complex ARIA role unless you also implement that role's expected keyboard behavior. Native buttons already support keyboard activation. A row of buttons with pressed states can be simpler than a custom radio-group implementation with arrow-key navigation.

## Common mistakes

- Calling **onChange** from **writeValue()**.
- Updating the component value on a user action but forgetting to call **onChange**.
- Calling **onTouched** on every internal blur in a composite control.
- Ignoring **setDisabledState()** or only styling the component as disabled.
- Replacing the **NG_VALUE_ACCESSOR** multi-provider instead of adding to it.
- Using **useClass** and accidentally creating a different component instance.
- Assuming that CVA automatically adds custom validators to the parent.
- Assuming that **formControlName** creates labels or error messages for the component's inner UI.
- Updating a value in a disabled widget.
- Declaring a separate disabled input that can disagree with the parent **FormControl**.

## Implementation checklist

Before using a custom control, confirm that:

- **writeValue()** renders programmatic values without calling **onChange**.
- A genuine user choice updates local view state and calls **onChange** once.
- Touch is reported on blur or when focus leaves the composite widget.
- Disabled state reaches every interactive child.
- **NG_VALUE_ACCESSOR** uses **useExisting** and **multi: true**.
- Parent validators and any custom validation responsibility are explicit.
- The widget has an accessible name, instructions, keyboard behavior, and visible error feedback.
- Reset, disable, enable, and programmatic value updates all display correctly.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="069-accessible-labels-instructions-and-validation-messages.md">069 - Accessible labels, instructions, and validation messages</a></td><td><a href="../qnas/070-custom-controls-with-controlvalueaccessor.md">Q&amp;A</a></td><td><a href="../examples/070-custom-controls-with-controlvalueaccessor.md">Example</a></td><td><a href="071-signal-forms-models-fields-and-form-setup.md">071 - Signal Forms: models, fields, and form setup</a></td>
</tr>
  </tbody>
</table>
