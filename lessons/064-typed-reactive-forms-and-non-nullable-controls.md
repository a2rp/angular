# Typed reactive forms and non-nullable controls

Angular's reactive forms are strictly typed by default. TypeScript can check the shape and values of controls in a **FormGroup**, provide editor autocomplete, and catch many mistakes before the application runs.

A **FormControl** is nullable by default because calling **reset()** can set its value to **null**. The **nonNullable** option changes both the control's type and its reset behavior. This lesson explains how to choose the correct type rather than removing nullability without considering what reset should do.

## Learning outcomes

After this lesson, you should be able to:

- Explain how Angular infers a control's value type.
- Choose between **FormControl<T>** and **FormControl<T | null>**.
- Understand why an initial string control is nullable by default.
- Use **nonNullable: true** for values that should reset to their initial value.
- Build typed groups with **NonNullableFormBuilder**.
- Recognize that a non-nullable builder does not alter a control constructed separately.
- Handle optional values in **FormGroup.value** and use **getRawValue()** when needed.
- Model optional controls and recognize when **Untyped** APIs give up type safety.

## Typed reactive forms by default

Typed reactive forms have been the default since Angular 14. When a **FormGroup** is built from typed controls, TypeScript tracks both the control names and their value types.

~~~ts
import { FormControl, FormGroup } from '@angular/forms';

const loginForm = new FormGroup({
  email: new FormControl(''),
  password: new FormControl(''),
});

loginForm.controls.email.setValue('asha@example.com');
loginForm.controls.email.setValue(null); // allowed by the inferred type
~~~

The email control is inferred as **FormControl<string | null>**, not **FormControl<string>**. Its value can become **null** when it is reset, so TypeScript includes that possibility.

Types apply to reactive forms. Template-driven forms do not currently provide the same typed control-tree API.

## Why FormControl is nullable by default

A regular control can be reset without a value:

~~~ts
const email = new FormControl('asha@example.com');

email.reset();

console.log(email.value); // null
~~~

That runtime behavior is why the type includes **null**. The type is describing what the control can actually hold, not just what the user initially typed.

Use a nullable control when an empty or absent value is meaningful. When initializing with **null**, provide a wider generic type if the control should later accept a string:

~~~ts
const nickname = new FormControl<string | null>(null);

nickname.setValue('Ash');
nickname.setValue(null);
~~~

Without the explicit generic, **new FormControl(null)** is inferred too narrowly as a control whose value type is only **null**.

## Make a control non-nullable

Use **nonNullable: true** when the control should always keep a value of its declared type and a no-argument reset should restore the initial value:

~~~ts
const displayName = new FormControl('Asha Rao', {
  nonNullable: true,
});

displayName.setValue('Mina'); // valid
// displayName.setValue(null); // TypeScript error

displayName.reset();
console.log(displayName.value); // 'Asha Rao'
~~~

This option changes both compile-time typing and runtime reset behavior:

| Control option | Value type | No-argument reset |
| --- | --- | --- |
| Default | **T \| null** | **null** |
| **nonNullable: true** | **T** | The control's initial default value |

Do not set non-nullable only to silence a TypeScript error. Decide whether **null** is a valid state in the application, and make the reset behavior match that decision.

You can still reset to an explicit value:

~~~ts
displayName.reset('Guest');
~~~

For a non-nullable control, that value becomes the current control value; a later no-argument reset returns to the control's default value.

## Type controls whose initial value is null

An initial **null** value alone cannot tell TypeScript which non-null type may be used later:

~~~ts
const missingType = new FormControl(null);
// missingType.setValue('Ash'); // Error: the inferred type is only null
~~~

Specify the intended union:

~~~ts
const optionalNickname = new FormControl<string | null>(null);
optionalNickname.setValue('Ash');
optionalNickname.setValue(null);
~~~

This is different from a non-nullable empty string control. Use **FormControl<string | null>** when both a string and no value are part of the model.

## Type a FormGroup

The control tree is typed from its members:

~~~ts
const profileForm = new FormGroup({
  displayName: new FormControl('', { nonNullable: true }),
  email: new FormControl('', { nonNullable: true }),
  nickname: new FormControl<string | null>(null),
});

profileForm.controls.displayName.setValue('Asha');
profileForm.controls.nickname.setValue(null);

// TypeScript catches misspelled control names and incompatible values.
// profileForm.controls.emali;
// profileForm.controls.email.setValue(42);
~~~

When a group contains a non-nullable control and a nullable control, each keeps its own type. The group's type also records the known control names.

Validators do not make a value non-nullable, and non-nullable typing does not prove that a value is valid. A **FormControl<string>** can still contain an empty string or a badly formatted email unless the application checks the validators and form status.

## Understand FormGroup.value and getRawValue()

A typed group's **value** is partial because disabled controls are excluded from an enabled group's aggregate value. A control's key can therefore be **undefined** in the group's **value** even when the group declares that control.

~~~ts
const profileForm = new FormGroup({
  displayName: new FormControl('', { nonNullable: true }),
  email: new FormControl('', { nonNullable: true }),
});

const currentValue = profileForm.value;
// currentValue.displayName can be undefined.
// currentValue.email can be undefined.

const completeValue = profileForm.getRawValue();
// completeValue.displayName is string.
// completeValue.email is string.
~~~

Use **value** when the application should use only enabled controls. Use **getRawValue()** when the operation intentionally requires every declared control, including disabled ones. A disabled field is excluded from validation too, so do not disable a required field merely to bypass a validity check.

The nullable values inside controls are a separate issue from optional keys in **FormGroup.value**:

- A nullable control may contain **null**.
- A disabled control may be absent from the parent's **value**, represented by **undefined** at that key.
- **getRawValue()** includes disabled control values, but a nullable control can still contain **null**.

## Use NonNullableFormBuilder

For a group with many non-nullable fields, **NonNullableFormBuilder** avoids repeating **{ nonNullable: true }** on every automatically created control.

~~~ts
import { Component, inject } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';

@Component({
  selector: 'app-account-form',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: '',
})
export class AccountFormComponent {
  private readonly fb = inject(FormBuilder).nonNullable;

  readonly accountForm = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(2)]],
    email: ['', [Validators.required, Validators.email]],
  });
}
~~~

The automatically constructed **name** and **email** controls are non-nullable and reset to their initial empty strings.

A non-nullable builder does not change a **FormControl** that you created yourself:

~~~ts
private readonly fb = inject(FormBuilder).nonNullable;

readonly profileForm = this.fb.group({
  displayName: '',
  nickname: new FormControl<string | null>(null),
});
~~~

Here **displayName** is non-nullable, but **nickname** remains nullable because the control was explicitly constructed with that type. The builder only controls the nullability of controls it constructs from simple configuration values.

You can inject **NonNullableFormBuilder** directly where that is a better fit for the application. It provides the non-nullable version of the builder methods.

## Optional controls in a group

Most groups describe controls that are always present. If a control may be added or removed at runtime, mark that control optional in the controls type:

~~~ts
interface AccountControls {
  email: FormControl<string>;
  recoveryEmail?: FormControl<string>;
}

const account = new FormGroup<AccountControls>({
  email: new FormControl('', { nonNullable: true }),
  recoveryEmail: new FormControl('', { nonNullable: true }),
});

account.removeControl('recoveryEmail');
account.addControl(
  'recoveryEmail',
  new FormControl('', { nonNullable: true }),
);
~~~

TypeScript allows adding and removing only controls represented as optional in the group type. Use **FormArray** for a dynamic ordered list of repeated controls, and **FormRecord** when the group has open-ended string keys with a consistent control type. These collection APIs are covered in later lessons.

## Untyped forms and migration

Angular still provides **UntypedFormControl**, **UntypedFormGroup**, **UntypedFormArray**, and related APIs for older code or for a gradual migration. They behave like their earlier untyped counterparts and use **any**, so TypeScript cannot verify control names or value shapes.

Prefer typed APIs for new reactive forms. During migration, replace untyped controls incrementally, fix type errors by choosing accurate nullable or non-nullable types, and keep **any** limited to boundaries where the value truly is unknown.

## Common mistakes

- **Assuming an initial string means a non-nullable control:** by default, reset can set the control to **null**.
- **Using new FormControl(null) without a generic:** TypeScript infers a control that only accepts null; specify the intended union type.
- **Adding nonNullable just to satisfy the compiler:** this also changes reset behavior, so confirm that behavior is correct.
- **Assuming validators change the type:** a required validator does not turn **string | null** into **string**. Check form validity before using the value.
- **Assuming every key is present in FormGroup.value:** disabled controls can be omitted. Account for **undefined** or use **getRawValue()** deliberately.
- **Assuming getRawValue removes null:** it includes disabled controls, but preserves each control's own nullable type.
- **Assuming NonNullableFormBuilder rewrites existing controls:** a preconstructed nullable control stays nullable.
- **Using FormGroup for arbitrary runtime keys:** use **FormRecord** for open-ended keys and **FormArray** for ordered dynamic collections.
- **Using Untyped APIs without a reason:** they discard useful compile-time checks.

## A practical decision guide

Choose a nullable **FormControl<T | null>** when:

- **null** represents a meaningful missing, cleared, or not-yet-chosen value.
- A control is explicitly cleared to **null** as part of the form flow.
- The data source itself may return **null**.

Choose a non-nullable **FormControl<T>** when:

- The control always has a value of type **T** in the component model.
- Resetting should restore its initial value.
- An empty string, zero, or false is a suitable empty value for the field.

The form type should model actual values and reset behavior. Validation is a separate question: even a well-typed value still needs to satisfy the form's rules.

## Next steps

- Continue with [FormArray and dynamic form structures](065-formarray-and-dynamic-form-structures.md).
- Review the [Q&A](../qnas/064-typed-reactive-forms-and-non-nullable-controls.md).
- Work through the [complete example](../examples/064-typed-reactive-forms-and-non-nullable-controls.md).
- Revisit [Reactive forms with FormControl and FormGroup](063-reactive-forms-with-formcontrol-and-formgroup.md).

## Official references

- [Strictly typed reactive forms](https://angular.dev/guide/forms/typed-forms)
- [FormControl API](https://angular.dev/api/forms/FormControl)
- [FormBuilder API](https://angular.dev/api/forms/FormBuilder)
- [NonNullableFormBuilder API](https://angular.dev/api/forms/NonNullableFormBuilder)
- [FormGroup API](https://angular.dev/api/forms/FormGroup)
