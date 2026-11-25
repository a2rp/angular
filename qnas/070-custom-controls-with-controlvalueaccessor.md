# Q&A: Custom controls with ControlValueAccessor

These questions review how a custom Angular component connects its UI with the Angular Forms model. Read the [lesson](../lessons/070-custom-controls-with-controlvalueaccessor.md) for the full explanation and build the [example](../examples/070-custom-controls-with-controlvalueaccessor.md) to see a rating widget used with **formControlName**.

## CVA purpose and methods

### 1. What problem does **ControlValueAccessor** solve?

It is the bridge between Angular Forms and a control's UI. It lets a custom component participate in model-to-view updates, view-to-model changes, touched state, and disabled state.

### 2. Which four methods make up the interface?

**writeValue()**, **registerOnChange()**, **registerOnTouched()**, and the optional **setDisabledState()**.

### 3. Which direction does **writeValue()** handle?

Model to view. Angular calls it to make the custom UI display a value supplied by the form model.

### 4. When is **writeValue()** called?

For example, when the form initializes the accessor, code calls **setValue()** or **patchValue()**, or a control is reset. Treat it as an instruction to render the model value.

### 5. Should **writeValue()** call the registered change callback?

No. It is a programmatic model-to-view update. Calling **onChange** from it can report a model update as user input and produce confusing events or feedback loops.

### 6. Which direction does **registerOnChange()** support?

Angular gives the component a callback that the component saves and calls when a user action changes the control value. The callback informs the form model about the view change.

### 7. Should the component call **onChange** during initialization?

No. Initialization or rendering is not a user edit. Call it after a real interaction changes the value.

### 8. Why initialize the callbacks to no-op functions?

The component should be safe before Angular registers its callbacks. No-op defaults avoid undefined-function errors during early lifecycle work.

### 9. What does **registerOnTouched()** provide?

Angular gives the component a callback to invoke when the control has been touched, commonly when focus leaves the control. The form can then update its touched state and display validation messages.

### 10. How should a composite control decide it was blurred?

Treat the component as one form control. If focus moves among its internal buttons or inputs, it is still inside the widget; report touched when focus leaves the entire widget.

### 11. Why does blur handling matter with **updateOn: 'blur'**?

Angular can defer committing a view change until the accessor reports touched. Calling the touched callback whenever focus moves between internal elements can commit too early.

### 12. What does **setDisabledState()** do?

It receives the form control's disabled status. The component should synchronize that state with every interactive part of its UI.

### 13. Is **setDisabledState()** required by the interface?

It is optional in the interface, but a robust custom control should implement it so parent-form disable and enable operations work correctly.

### 14. Is styling a widget gray enough to disable it?

No. Its interactive child elements must actually be disabled or otherwise prevented from accepting input, and the disabled state should be represented to users.

## Provider registration and parent usage

### 15. What is **NG_VALUE_ACCESSOR**?

It is the Angular injection token used to register value accessors for form controls. A custom component provides itself with this token.

### 16. Why use **multi: true** with **NG_VALUE_ACCESSOR**?

The token is a collection of accessors. **multi: true** adds this accessor to the collection instead of replacing the provider.

### 17. Why use **useExisting**?

It tells Angular to use the custom component instance as its accessor. Using **useClass** could create another instance that does not match the component Angular rendered.

### 18. Why is **forwardRef()** often used in the provider?

The component class is referenced from its own decorator before the class declaration is complete. **forwardRef()** defers resolving that reference.

### 19. What causes a “No value accessor” error?

Angular Forms found a control binding such as **formControlName** but could not find a compatible value accessor for the bound element or component.

### 20. How does a parent form bind a custom accessor?

Use it as the host of **formControlName** or bind a **FormControl** with **[formControl]**, just as you would bind a supported native control.

### 21. Does CVA only work with Reactive Forms?

No. A correctly registered accessor can be used with Angular Forms directives such as **formControl**, **formControlName**, and **ngModel**, with the corresponding forms setup.

### 22. Which part owns the form value and validators?

The parent **FormControl** remains the model's source of truth. The component renders the value and reports UI changes through its callbacks.

## Change detection, validation, and interaction

### 23. What should an OnPush accessor do after **writeValue()**?

If it changes a component property from a forms callback, call **ChangeDetectorRef.markForCheck()** so the OnPush view is checked.

### 24. What should happen when the user selects a new rating?

Update the component's displayed selection and call the saved **onChange** callback with the selected value. Do not emit a different value when the widget is disabled.

### 25. Should selecting the already selected value emit another change?

Usually no. If the model value has not changed, there is no new value to report.

### 26. How is the component's value cleared by a parent reset?

Angular calls **writeValue()** with the reset value. The component must update its UI in response without calling **onChange**.

### 27. Does CVA provide validation automatically?

No. CVA is the value/touched/disabled bridge. Use validators on the parent control, or separately implement Angular's **Validator** interface and register **NG_VALIDATORS** for rules owned by the custom control.

### 28. Does a CVA automatically expose the parent control's errors inside its template?

No. If the component needs to render a parent validation message, design an explicit input or another deliberate integration. The CVA callbacks alone do not provide the parent's error map.

### 29. Does **formControlName** automatically label the custom component's inner controls?

No. The component still needs accessible internal labels, group names, descriptions, and keyboard behavior.

### 30. Should a CVA define a separate disabled input?

Usually not for the same state. It can drift from the parent control. Prefer the form model's **disable()** and **enable()** methods and implement **setDisabledState()**.

## Composite widget behavior

### 31. Why might a star picker use native buttons instead of custom radio roles?

Native buttons already support keyboard activation with Enter and Space. Custom radio roles require the component to implement radio-group keyboard behavior, including arrow-key navigation, so buttons can be simpler when they fit the interaction.

### 32. How can the selected rating be communicated accessibly?

Give each button an accessible name such as “3 stars” and expose the current choice with **aria-pressed**. Group the choices under a meaningful **fieldset** and **legend**.

### 33. Should a decorative star glyph be the only accessible name?

No. Provide a meaningful accessible name for the button. The decorative glyph can be hidden from assistive technology if it would otherwise be announced confusingly.

### 34. How should a custom control report required or invalid state?

The parent form owns the validators. The widget should expose a visible required label and, when necessary, display or associate the relevant error with its internal control group through an explicit component API.

### 35. What should keyboard testing check for a composite CVA?

Check that all interactive elements can be reached and operated, focus remains inside the widget while moving among its elements, and leaving the widget marks the parent control touched.

### 36. What should be tested when a parent disables the custom control?

Confirm that every interactive child becomes disabled, cannot change the value, and returns to an operable state after the parent enables the control.

### 37. What should be tested for programmatic updates?

Call **setValue()**, **patchValue()**, and **reset()** on the parent control and confirm that the view updates each time without treating those updates as user changes.

### 38. What is the essential CVA checklist?

Render **writeValue()**, call **onChange** only for actual user changes, call **onTouched** at the right focus boundary, propagate disabled state to the UI, register with **NG_VALUE_ACCESSOR**, and provide custom validation and accessibility separately where needed.
