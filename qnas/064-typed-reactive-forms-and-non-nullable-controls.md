# Q&A: Typed reactive forms and non-nullable controls

These questions review how TypeScript models Angular reactive forms. See the [lesson](../lessons/064-typed-reactive-forms-and-non-nullable-controls.md) for the full explanation and the [example](../examples/064-typed-reactive-forms-and-non-nullable-controls.md) for a profile form with both nullable and non-nullable controls.

## Typed forms basics

### 1. What are typed reactive forms?

Typed reactive forms use TypeScript types to describe control values and form structure. Angular can check control names, value types, and group shapes at compile time.

### 2. Are reactive forms typed by default?

Yes. Angular's reactive forms have been strictly typed by default since Angular 14. Older code can still use the explicit **Untyped** APIs.

### 3. Do template-driven forms have the same typed control tree?

No. The strict control-tree typing discussed here applies to reactive forms. Template-driven forms are primarily declared through template directives.

### 4. Does TypeScript type a FormGroup by its value object?

A **FormGroup** is typed by its child controls. The control keys and the value types of those controls determine the group's typed APIs.

### 5. What happens if I reference a misspelled control name?

A statically known typo such as **form.controls.emali** is a TypeScript error. This is one benefit of using a typed group instead of an untyped one.

## Nullability and reset

### 6. Why does new FormControl('') infer a nullable value?

A regular **FormControl** can become **null** when **reset()** is called without a value. Angular includes that runtime possibility in the type, so an initial string control is inferred as **FormControl<string | null>**.

### 7. What does resetting a default nullable control do?

Calling **reset()** without an explicit value sets the control value to **null** and resets its interaction state.

### 8. How do I make a control non-nullable?

Pass **{ nonNullable: true }** in its options, or create it through a **NonNullableFormBuilder**. The control's type excludes **null**, and a no-argument reset restores its initial default value.

### 9. Does nonNullable only change the TypeScript type?

No. It changes both typing and runtime behavior. A non-nullable control resets to its initial value instead of **null**.

### 10. Should I use nonNullable to silence a nullability error?

Only when **null** is not a valid value in the application's model and restoring the initial value on reset is desired. Do not hide a meaningful nullable state just to remove a compiler warning.

### 11. What happens with new FormControl(null)?

TypeScript infers the narrow type **FormControl<null>**, which will not accept a string later. Specify the intended type, for example **new FormControl<string | null>(null)**.

### 12. Can a non-nullable control still be reset to an explicit value?

Yes. **control.reset('Guest')** sets that value and resets the control state. A later no-argument reset returns to the control's default value.

### 13. Do validators make a nullable control non-nullable?

No. Validators affect validation status, not the control's TypeScript type. A required validator does not change **string | null** into **string**.

### 14. Does a non-nullable string control guarantee a non-empty value?

No. Its value is a string, but it can still be an empty string or otherwise fail a validator. Check the control or form validity before processing it.

## FormBuilder and groups

### 15. What is NonNullableFormBuilder?

It is a builder whose automatically constructed controls use non-nullable behavior. It can reduce repeated **{ nonNullable: true }** options in a form with many non-nullable controls.

### 16. How can I use the non-nullable builder from FormBuilder?

Use the **nonNullable** property, such as **inject(FormBuilder).nonNullable**, and call its **group** method. Controls it creates from simple values are non-nullable.

### 17. Does NonNullableFormBuilder change a FormControl I construct myself?

No. If a preconstructed **FormControl** is passed to the builder, its existing type and options are preserved. A manually created nullable control stays nullable.

### 18. Can a group have both nullable and non-nullable controls?

Yes. Each child keeps its own type. For example, a required display name may be **string**, while an optional nickname may be **string | null**.

### 19. Why is FormGroup.value partial?

Disabled controls are excluded from an enabled group's aggregate value. The type therefore allows a declared key to be **undefined** in **group.value**.

### 20. How is getRawValue different from value?

**getRawValue()** includes values for disabled controls as well as enabled ones. The control types remain intact, so a nullable control may still have a **null** value.

### 21. Is undefined in FormGroup.value the same as null in a FormControl?

No. **null** is a possible value of a nullable control. **undefined** in the group's partial value means the control may be excluded from the aggregate value, commonly because it is disabled.

### 22. When should I use value and when should I use getRawValue()?

Use **value** when disabled controls should be omitted. Use **getRawValue()** when the operation intentionally requires every control's current value, including disabled controls.

### 23. Does using getRawValue make disabled fields valid?

No. Disabled controls remain exempt from validation. **getRawValue()** only includes their values; it does not change their status or validate them.

## Optional and dynamic controls

### 24. How do I type a control that may be removed from a group?

Declare that control as an optional property in the group's control interface. Then TypeScript can allow it to be added or removed while keeping required controls present.

### 25. What is FormArray for?

**FormArray** represents an ordered list of controls or groups whose count can change at runtime. It is appropriate for repeated rows such as phone numbers or addresses.

### 26. What is FormRecord for?

**FormRecord** represents a group with open-ended string keys and a consistent control type. Use it when the keys are not known ahead of time but the value/control shape is consistent.

### 27. Can FormGroup have arbitrary runtime keys?

A typed **FormGroup** is intended for known keys, with optional controls for known keys that may be added or removed. For arbitrary open-ended keys, use **FormRecord**.

## Compatibility and migration

### 28. What do the Untyped form classes do?

**UntypedFormControl**, **UntypedFormGroup**, and related classes preserve the older behavior where values and structure use **any**. They are useful for gradual migration, but they remove compile-time checks.

### 29. Should new reactive forms use Untyped APIs?

Usually no. Prefer typed APIs for new code. Use untyped forms only when the data is truly unstructured or as a deliberate step in a migration.

### 30. What should I do when migrating a nullable form?

First identify what values each field can actually hold and what **reset()** should do. Then replace untyped controls with typed ones, choose nullable or non-nullable behavior per field, and fix errors without using broad casts to suppress them.

### 31. Do these types validate server data at runtime?

No. TypeScript types are compile-time checks and do not validate JSON or other runtime data. Validate untrusted server or user data at runtime before relying on it.

### 32. Do typed forms replace checking form validity?

No. Types describe possible value shapes; validators decide whether current values satisfy application rules. Check **form.valid** or **form.invalid** before processing submitted values.

## Practice

1. Create a non-nullable **FormControl<string>** initialized to an empty string.
2. Create a nullable **FormControl<string | null>** initialized to **null**.
3. Call **reset()** on both and compare their values.
4. Add them to one typed **FormGroup** and inspect the inferred types.
5. Disable a control and compare **form.value** with **form.getRawValue()**.
6. Build a form with **NonNullableFormBuilder**, then add one explicitly constructed nullable control.

For a complete profile form, continue to the [example](../examples/064-typed-reactive-forms-and-non-nullable-controls.md).

## Official references

- [Strictly typed reactive forms](https://angular.dev/guide/forms/typed-forms)
- [FormControl API](https://angular.dev/api/forms/FormControl)
- [FormControlOptions API](https://angular.dev/api/forms/FormControlOptions)
- [FormGroup API](https://angular.dev/api/forms/FormGroup)
- [FormBuilder API](https://angular.dev/api/forms/FormBuilder)
- [NonNullableFormBuilder API](https://angular.dev/api/forms/NonNullableFormBuilder)
- [FormArray API](https://angular.dev/api/forms/FormArray)
- [FormRecord API](https://angular.dev/api/forms/FormRecord)
