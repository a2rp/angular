# FormArray and dynamic form structures

A **FormArray** manages an ordered collection of Angular form controls whose number can change while the application is running. Use it for repeatable fields such as phone numbers, skills, addresses, order lines, or survey answers.

A **FormGroup** organizes controls by known names. A **FormArray** organizes controls by numeric position. A **FormRecord** organizes controls by dynamic string keys. Choosing the right container keeps both the form model and its TypeScript types aligned with the data.

## Learning outcomes

After this lesson, you should be able to:

- Explain when to use **FormArray**, **FormGroup**, or **FormRecord**.
- Create a typed **FormArray** of controls or repeated groups.
- Add, insert, replace, remove, and clear controls through the public API.
- Bind an array of controls to a reactive-form template.
- Validate individual entries and the array as a whole.
- Explain how disabled controls affect **value** and **getRawValue()**.
- Set array values safely and understand how **reset()** differs from removing controls.

## Choose the right form container

| Container | Shape | Typical use |
| --- | --- | --- |
| **FormControl** | One value | Name, email, checkbox |
| **FormGroup** | Object with known keys | A profile with name and email |
| **FormArray** | Ordered list of controls with a changing length | A list of skills or repeated address groups |
| **FormRecord** | Object with dynamic string keys and a consistent control type | A set of settings whose names are not known in advance |

A **FormArray** is dynamic in length, but its child controls should have a consistent control type. That type can itself be a **FormGroup** or another **FormArray**. For example, a list of addresses can be a **FormArray** of address **FormGroup** controls.

If each item has several fields, model each row as a **FormGroup** and put those groups in the array. This preserves names such as `street` and `city` inside each row while the outer collection remains ordered.

## Create a typed FormArray of controls

Import the reactive forms types needed by the component and add **ReactiveFormsModule** to the component imports:

~~~ts
import { Component, inject } from '@angular/core';
import {
  FormArray,
  FormBuilder,
  FormControl,
  ReactiveFormsModule,
  Validators,
} from '@angular/forms';

@Component({
  selector: 'app-skill-list',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: './skill-list.component.html',
})
export class SkillListComponent {
  private readonly fb = inject(FormBuilder).nonNullable;

  readonly skills = this.fb.array<FormControl<string>>([
    this.fb.control('TypeScript', Validators.required),
  ]);

  addSkill(): void {
    this.skills.push(this.fb.control('', Validators.required));
    this.skills.markAsDirty();
  }

  removeSkill(index: number): void {
    if (index < 0 || index >= this.skills.length) {
      return;
    }

    this.skills.removeAt(index);
    this.skills.markAsDirty();
  }

  clearSkills(): void {
    this.skills.clear();
    this.skills.markAsDirty();
  }
}
~~~

The explicit **FormControl<string>** element type makes the array homogeneous and non-nullable. The injected non-nullable builder creates controls whose no-argument reset restores their initial value.

**push()** appends one control. Current Angular versions also allow passing an array of controls to **push()** when adding several at once. **insert()** adds a control at a chosen index, **setControl()** replaces the control at an index, **removeAt()** removes one, and **clear()** removes every control.

These methods register controls with the form tree and update the array's value and validation state. Do not mutate **skills.controls** directly with native array methods such as **push()** or **splice()**. Direct mutation bypasses Angular's control registration and change notifications.

### Structural changes and dirty state

Angular does not mark a **FormArray** dirty merely because **push()**, **insert()**, **setControl()**, or **removeAt()** was called. If the change represents a user's edit and your UI relies on dirty state, call **markAsDirty()** explicitly, as in the component above.

A programmatic change made while loading saved data may intentionally leave the form pristine. Dirty state should represent the user interaction policy your application needs.

## Put the array inside a FormGroup

Most forms combine fixed fields with a dynamic list. Put the array in a parent group and access it through the group's typed controls:

~~~ts
readonly profileForm = this.fb.group({
  fullName: ['', Validators.required],
  skills: this.fb.array<FormControl<string>>([
    this.fb.control('TypeScript', Validators.required),
  ]),
});

get skills(): FormArray<FormControl<string>> {
  return this.profileForm.controls.skills;
}
~~~

The getter provides a convenient typed reference to the array. It avoids repeatedly looking up a control by string and casting the result.

The form's value has an object shape, and its **skills** member is an ordered array. The number and order of values correspond to the registered child controls, subject to disabled-control behavior explained below.

## Bind a FormArray in the template

Use **formArrayName** to connect a nested array to its parent **FormGroup**. Since array controls do not have string names, bind each child by its numeric index through **formControlName**:

~~~html
<form [formGroup]="profileForm" (ngSubmit)="save()">
  <label for="full-name">Full name</label>
  <input id="full-name" formControlName="fullName" autocomplete="name" />

  <fieldset formArrayName="skills">
    <legend>Skills</legend>

    @for (skill of skills.controls; track skill; let i = $index) {
      <div class="skill-row">
        <label [for]="'skill-' + i">Skill {{ i + 1 }}</label>
        <input
          [id]="'skill-' + i"
          type="text"
          [formControlName]="i"
          autocomplete="off"
        />
        <button
          type="button"
          [attr.aria-label]="'Remove skill ' + (i + 1)"
          (click)="removeSkill(i)"
        >
          Remove
        </button>
      }
    }
  </fieldset>

  <button type="button" (click)="addSkill()">Add a skill</button>
  <button type="submit" [disabled]="profileForm.invalid">Save profile</button>
</form>
~~~

The **formArrayName** directive locates the array in the parent group. The repeated input uses the current index as its **formControlName**. Tracking each control object gives Angular a stable identity for a row when the list changes.

Buttons inside a form default to submit buttons unless their type is specified. Mark the add and remove buttons **type="button"** so they do not submit the form. Each repeated input needs a label associated with a unique **id**; the remove button also has an accessible name that identifies the row.

For a top-level array with no parent group, Angular also provides **FormArrayDirective** and the **[formArray]** binding. The nested **formArrayName** approach is usually clearer when the array is one part of a larger form.

## Use a FormArray of repeated FormGroup rows

When every entry has several fields, put a group at each index. This example models skills with a name and level:

~~~ts
import { Component, inject } from '@angular/core';
import {
  FormArray,
  FormBuilder,
  FormControl,
  FormGroup,
  ReactiveFormsModule,
  Validators,
} from '@angular/forms';

type SkillLevel = 'beginner' | 'intermediate' | 'advanced';

type SkillControls = {
  name: FormControl<string>;
  level: FormControl<SkillLevel>;
};

type SkillGroup = FormGroup<SkillControls>;

@Component({
  selector: 'app-profile-editor',
  standalone: true,
  imports: [ReactiveFormsModule],
  templateUrl: './profile-editor.component.html',
})
export class ProfileEditorComponent {
  private readonly fb = inject(FormBuilder).nonNullable;

  readonly profileForm = this.fb.group({
    fullName: ['', Validators.required],
    skills: this.fb.array<SkillGroup>([this.createSkill()]),
  });

  get skills(): FormArray<SkillGroup> {
    return this.profileForm.controls.skills;
  }

  private createSkill(): SkillGroup {
    return this.fb.group({
      name: ['', Validators.required],
      level: this.fb.control<SkillLevel>('beginner'),
    });
  }

  addSkill(): void {
    this.skills.push(this.createSkill());
    this.skills.markAsDirty();
  }

  removeSkill(index: number): void {
    if (index < 0 || index >= this.skills.length) {
      return;
    }

    this.skills.removeAt(index);
    this.skills.markAsDirty();
  }

  save(): void {
    if (this.profileForm.invalid) {
      this.profileForm.markAllAsTouched();
      return;
    }

    const profile = this.profileForm.getRawValue();
    console.log(profile);
  }
}
~~~

The factory method creates the same group shape for every row. It also keeps row initialization and validators in one place. Since the level control uses a union type, TypeScript rejects values outside the three allowed levels in code.

Bind each group with **[formGroupName]** and its numeric index:

~~~html
<form [formGroup]="profileForm" (ngSubmit)="save()">
  <label for="full-name">Full name</label>
  <input id="full-name" formControlName="fullName" autocomplete="name" />

  <fieldset formArrayName="skills">
    <legend>Skills</legend>

    @for (skillGroup of skills.controls; track skillGroup; let i = $index) {
      <fieldset [formGroupName]="i">
        <legend>Skill {{ i + 1 }}</legend>

        <label [for]="'skill-name-' + i">Name</label>
        <input
          [id]="'skill-name-' + i"
          formControlName="name"
          autocomplete="off"
        />

        <label [for]="'skill-level-' + i">Level</label>
        <select [id]="'skill-level-' + i" formControlName="level">
          <option value="beginner">Beginner</option>
          <option value="intermediate">Intermediate</option>
          <option value="advanced">Advanced</option>
        </select>

        <button
          type="button"
          [attr.aria-label]="'Remove skill ' + (i + 1)"
          (click)="removeSkill(i)"
        >
          Remove skill
        </button>
      </fieldset>
    }
  </fieldset>

  <button type="button" (click)="addSkill()">Add skill</button>
  <button type="submit">Save profile</button>
</form>
~~~

The nesting mirrors the model: the parent **FormGroup** contains a **FormArray** named **skills**, and each array element is a **FormGroup** with **name** and **level** controls.

## Validate rows and the collection

Validators can be attached at different levels:

- Put a validator on an individual **FormControl** when a value needs its own rule, such as a required skill name.
- Put a validator on a row **FormGroup** when multiple fields in that row must agree.
- Put a validator on the **FormArray** when the collection as a whole needs a rule, such as a maximum number of entries or no duplicate values.

Child validity contributes to the status of the parent array and form. A single invalid child makes the enabled parent form invalid. Use **markAllAsTouched()** after a rejected submit when the interface should reveal the field errors.

The minimum and maximum length validators can be applied to a collection. Keep the distinction between *required item values* and *required collection size* clear. A required validator on a child text control does not require that the array contain a child. If the business rule requires at least one row, apply and test a collection-level validator that expresses that rule.

Validation should explain the error to the user. For a dynamic list, show which row is invalid and identify missing or malformed values, rather than presenting only a generic form-level failure.

## Read values and account for disabled controls

For an enabled **FormArray**, **value** contains values for its enabled child controls. Disabled children are omitted from an enabled parent's aggregate value. Use **getRawValue()** when the operation intentionally needs values from disabled children too.

~~~ts
const enabledSkills = this.skills.value;
const everySkillIncludingDisabled = this.skills.getRawValue();
~~~

The raw value includes all child values but does not change the controls' enabled state. A disabled control is excluded from validation as well as from the enabled parent's aggregate value. Do not disable a field merely to avoid validating input that should still be required.

## Set values, patch values, and reset

A value update does not create or remove controls. The control structure must exist before assigning values.

**setValue()** is strict: provide a value for every control in the array, in the expected order. A mismatch throws, which helps catch structure errors.

~~~ts
this.skills.setValue(['TypeScript', 'Angular']);
~~~

**patchValue()** accepts a partial array of values and updates the controls it can match. It is useful when applying partial data, but its permissiveness can make mismatched structures less obvious.

~~~ts
this.skills.patchValue(['TypeScript']);
~~~

If incoming data should define the number of rows, construct or reconcile the controls first, then set or patch the values. Avoid assuming that **setValue()** will build missing controls.

Calling **reset()** resets existing children and marks descendants pristine and untouched; it does not remove the controls from the array. Call **clear()** when the intent is to remove all rows. If the form should return to a predefined initial list, rebuild or reset the array to that intended structure.

## Common mistakes

- **Using FormGroup for an unknown number of indexed rows:** use **FormArray** for an ordered collection.
- **Using FormArray for open-ended named keys:** use **FormRecord** for dynamic string keys.
- **Mixing unrelated child control types in a typed array:** design the array around one consistent child-control type, such as repeated groups with the same shape.
- **Using an index as a string key:** the child is selected by array index; bind it with **[formControlName]="i"** or **[formGroupName]="i"**.
- **Calling native array methods on controls:** use Angular's **push()**, **insert()**, **setControl()**, **removeAt()**, and **clear()** methods.
- **Expecting add/remove methods to set dirty state:** call **markAsDirty()** when the structural edit is a user change.
- **Expecting setValue() or patchValue() to create rows:** create the controls before assigning values.
- **Expecting reset() to empty the array:** reset existing controls; use **clear()** to remove them.
- **Ignoring disabled controls when preparing a payload:** choose **value** or **getRawValue()** deliberately.
- **Creating repeated unlabeled inputs:** give each field an associated label and each repeated action a useful accessible name.

## Next steps

- Continue with [Synchronous validators and validation state](066-synchronous-validators-and-validation-state.md).
- Review the [Q&A](../qnas/065-formarray-and-dynamic-form-structures.md).
- Work through the [complete example](../examples/065-formarray-and-dynamic-form-structures.md).
- Revisit [Typed reactive forms and non-nullable controls](064-typed-reactive-forms-and-non-nullable-controls.md).

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="064-typed-reactive-forms-and-non-nullable-controls.md">064 - Typed reactive forms and non-nullable controls</a></td><td><a href="../qnas/065-formarray-and-dynamic-form-structures.md">Q&amp;A</a></td><td><a href="../examples/065-formarray-and-dynamic-form-structures.md">Example</a></td><td><a href="066-synchronous-validators-and-validation-state.md">066 - Synchronous validators and validation state</a></td>
</tr>
  </tbody>
</table>
