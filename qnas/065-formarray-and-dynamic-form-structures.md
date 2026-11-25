# Q&A: FormArray and dynamic form structures

These questions review Angular's typed **FormArray** API and common dynamic-form decisions. Read the [lesson](../lessons/065-formarray-and-dynamic-form-structures.md) for the complete explanation and build the [example](../examples/065-formarray-and-dynamic-form-structures.md) to practice with a repeatable list of grouped controls.

## Choosing a form container

### 1. What does FormArray represent?

A **FormArray** represents an ordered collection of form controls. Its length can change at runtime, and each child is registered at a numeric index.

### 2. When should I use FormArray?

Use it when a form contains a repeatable ordered list, such as skills, phone numbers, addresses, or invoice lines, and you do not know the number of entries in advance.

### 3. How is FormArray different from FormGroup?

A **FormGroup** organizes controls under known string keys such as **email** and **address**. A **FormArray** organizes children by numeric index. The group is suited to a fixed named structure; the array is suited to a changing ordered structure.

### 4. How is FormArray different from FormRecord?

A **FormRecord** is a collection with dynamic string keys and a consistent control type. Use **FormArray** when order and numeric positions matter; use **FormRecord** when keys are names that are discovered at runtime.

### 5. Can a FormArray contain FormGroups?

Yes. This is a common pattern when each repeated row has multiple named fields. Create an array of groups with the same control shape, for example an array of address groups with **street**, **city**, and **postalCode** controls.

### 6. Can different entries have different shapes in a typed FormArray?

A typed array is designed around one consistent child-control type. Prefer a common **FormGroup** shape for repeated rows. If the data is truly heterogeneous, model that explicitly at the application boundary rather than hiding incompatible controls behind **any**.

### 7. Can FormArrays be nested?

Yes. A child can itself be a **FormArray**, which is useful for nested repeatable structures. Each nested level needs a matching nested form directive in the template.

## Typing and construction

### 8. How do I create a FormArray of strings?

Create an array whose element type is **FormControl<string>**, for example **fb.array<FormControl<string>>([...])** with a non-nullable builder.

### 9. Why should I use a factory method for repeated FormGroup rows?

A row factory creates every group with the same controls, types, defaults, and validators. It avoids duplicating setup code and keeps future row changes consistent.

### 10. What does the FormArray generic describe?

The generic describes the child control type, not just the array of values. For example, **FormArray<FormControl<string>>** is an array of string controls, while **FormArray<SkillGroup>** is an array of groups with the same skill controls.

### 11. Are reactive forms typed by default?

Yes. Angular's reactive forms have been strictly typed by default since Angular 14. Explicit **UntypedFormArray** APIs remain available for legacy code, but new forms should normally keep their useful types.

### 12. Does a validator change the value type?

No. A validator affects the control's validity state; it does not transform or narrow the TypeScript value type. Check validity and handle the value according to its declared type.

### 13. Does a non-nullable FormBuilder make every child control non-nullable?

It makes controls that the builder constructs from simple configurations non-nullable. A control constructed explicitly with a nullable type remains nullable.

### 14. How can I access an array nested in a group without casting?

Use the group's typed **controls** property, for example **profileForm.controls.skills**, or expose it through a typed getter. This avoids the **AbstractControl** return type from a string-based lookup.

## Adding, removing, and replacing controls

### 15. How do I append a control?

Call **formArray.push(control)**. Current Angular APIs also accept an array of controls in one **push()** call.

### 16. How do I add a control at a particular position?

Call **formArray.insert(index, control)**. The index refers to the child's numeric position.

### 17. How do I replace an existing child?

Call **formArray.setControl(index, replacementControl)**. Use this when the control at a specific position should be replaced rather than edited.

### 18. How do I remove one child?

Call **formArray.removeAt(index)** after checking that the index is within the array's current range.

### 19. How do I remove all children?

Call **formArray.clear()**. It removes every child control; it is simpler and more efficient than repeatedly removing the first control.

### 20. Should I call native array methods on FormArray.controls?

No. Do not use native **push()**, **splice()**, or direct index assignment on the controls array. Use the **FormArray** API so Angular can register the child and update value and validity state.

### 21. Do push() and removeAt() automatically mark the array dirty?

No. Structural methods update the control tree but do not automatically mark the array dirty. Call **markAsDirty()** when the add or remove action represents a user edit that should count as a change.

### 22. Should loading saved rows mark a form dirty?

Usually not if loading is initialization or restoring server data. Dirty state should follow the application's meaning of a user edit. Mark the array or parent form dirty when the user changes its structure.

### 23. How do I prevent removal of a nonexistent row?

Check the index before calling **removeAt()**:

~~~ts
if (index >= 0 && index < this.skills.length) {
  this.skills.removeAt(index);
  this.skills.markAsDirty();
}
~~~

This makes the component's behavior explicit rather than relying on special negative-index behavior.

### 24. Does changing the array length change its value and validity?

Yes. Angular recalculates the array's aggregate value and validation status when controls are added, inserted, replaced, removed, or cleared. By default, these operations emit the corresponding value and status events.

### 25. Can I suppress events during a batch update?

Some structural methods accept an **emitEvent** option. Setting it to **false** suppresses the array's **valueChanges** and **statusChanges** emissions for that operation. Use it when the form flow intentionally should not react to an intermediate update.

## Template binding and accessibility

### 26. What does formArrayName do?

**formArrayName** connects a nested **FormArray** to its parent reactive form directive using the array's key, such as **skills**.

### 27. How do I bind a control by array index?

Inside the **formArrayName** container, bind a child with **[formControlName]="i"**, where **i** is the current numeric index.

### 28. How do I bind a FormGroup row inside the array?

Use **[formGroupName]="i"** on the repeated row. Inside that group, bind fields by their named **formControlName**, such as **name** and **level**.

### 29. Why track each repeated control in an @for block?

A stable tracking expression lets Angular associate an existing rendered row with the same control object when the list changes. This helps preserve row identity during insertions and removals.

### 30. Why should add and remove buttons have type="button"?

A button inside a form defaults to submit behavior when no type is specified. Use **type="button"** for list-management actions so they do not accidentally submit the form.

### 31. How should repeated inputs be labeled?

Give each repeated input a unique **id** and connect a **label** to that id with **for**. Give each remove button a useful accessible name, such as **Remove skill 2**, so its purpose is clear when read out of context.

### 32. Can FormArray be bound directly to a form element?

Yes. Angular provides **FormArrayDirective** and the **[formArray]** binding for a top-level array. Use **formArrayName** when the array is nested inside a parent **FormGroup**.

## Values, validation, and reset

### 33. What is the difference between controls and value?

**controls** contains the child control instances and exposes methods and state. **value** contains the current aggregate values of enabled descendants. Use controls to render or modify the form structure; use values to read the data.

### 34. What happens to disabled children in FormArray.value?

Disabled controls are omitted from the aggregate value of an enabled parent. This means the values array may not map one-to-one by index with **controls** if some children are disabled.

### 35. When should I use getRawValue()?

Use **getRawValue()** when you intentionally need values from disabled controls as well as enabled controls. It returns values for all descendants without enabling those controls.

### 36. Does a disabled control still participate in validation?

Disabled controls are exempt from validation and excluded from the aggregate value of enabled ancestors. Do not disable a field solely to hide a validation error if the value should still be required.

### 37. Does a FormArray validate each child?

Each child validates itself, and the array's validity reflects its enabled descendants and any validators attached to the array. A row **FormGroup** can also validate rules involving several fields in that row.

### 38. How do I require at least one row?

Add a collection-level validator that expresses the business rule and verify its behavior for both an empty and a non-empty array. A required validator on each text input only validates an input that exists; it does not create or require a row.

### 39. What is the difference between setValue() and patchValue()?

**setValue()** requires a value for every existing control in the expected structure and throws when the shape does not match. **patchValue()** accepts partial values and applies them to controls it can match, which is more permissive.

### 40. Do setValue() and patchValue() create missing controls?

No. They update values of controls that already exist. Construct or reconcile the controls first when incoming data determines how many rows the array should contain.

### 41. Does reset() remove FormArray children?

No. **reset()** resets the existing descendants and their interaction state. Use **clear()** to remove every child control.

### 42. Can I call reset() with values?

Yes. Pass an array of values or form states that corresponds to the existing structure. A non-nullable child resets to its configured default when reset without an explicit value.

### 43. Is an empty FormArray automatically invalid?

Not by itself. Whether zero entries are valid depends on the business rule and validators you attach. Add a collection-level validator if at least one entry is required.

## Practical design and troubleshooting

### 44. Why is a new row missing from the template?

Check that the component uses **push()** or **insert()** on the registered **FormArray**, that the template uses the correct **formArrayName**, and that each child has a matching index binding. Avoid directly changing **controls**.

### 45. Why does a row show the wrong value after removing an earlier row?

Check that the repeated template uses the current array index for **formControlName** or **formGroupName**, and that the **@for** block tracks the control object rather than an unrelated value. Also verify there are no mismatched direct DOM edits.

### 46. Why does the form remain pristine after adding a row?

That is the documented behavior of structural operations. Call **markAsDirty()** if the user action should count as an edit.

### 47. Why does a saved payload omit a disabled row?

The enabled parent's aggregate **value** excludes disabled controls. Use **getRawValue()** if the payload intentionally needs them, or re-enable the row if it should take part in the normal form.

### 48. Why does patchValue() not show all incoming rows?

It does not construct controls. Add or reconcile the necessary child controls before applying the incoming values.

### 49. Which status should I check before saving?

Check the parent form's **invalid** and, when async validation exists, **pending** states before saving. If submission fails validation, call **markAllAsTouched()** when the interface should reveal errors.

### 50. What should I review before shipping a dynamic form?

Confirm that row structure and types are consistent, all structural changes use the Angular API, validation covers both row-level and collection-level rules, disabled values are handled deliberately, repeated fields have accessible labels, and add/remove actions have the correct button type.

## Official references

- [Reactive forms and dynamic FormArray controls](https://angular.dev/guide/forms/reactive-forms)
- [FormArray API](https://angular.dev/api/forms/FormArray)
- [Strictly typed reactive forms](https://angular.dev/guide/forms/typed-forms)
- [Building dynamic forms](https://angular.dev/guide/forms/dynamic-forms)
