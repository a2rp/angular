# Q&A: Signal inputs, required inputs, and input transforms

## Signal input basics

### 1. What is a component input?

An input is a value a parent supplies to a child component through Angular's template binding system. The child declares the input as part of its public API.

### 2. What does Angular's input function declare?

`input()` declares an input property and returns an input signal. The component reads the latest value by calling that signal.

### 3. How do I read a signal input in TypeScript?

Call it like a signal: `this.title()`. Do not read it as a plain property value.

### 4. How do I read a signal input in the component template?

Call the input signal in the template, for example `{{ title() }}`.

### 5. Can the child assign a new value to an input signal?

No. Input signals are read-only. The parent owns the bound value. Use internal writable state for child-owned changes or a model input when two-way binding is part of the component API.

### 6. Does a signal input update when the parent binding changes?

Yes. Angular updates the input signal with the latest value from the parent binding. Computations that read it can then update accordingly.

### 7. Where may input() be called?

Declare it in a component or directive class field initializer. Angular statically analyzes input declarations, so do not call it from a method, constructor body, loop, or conditional branch.

### 8. Is input() a general-purpose signal constructor?

No. Use `signal()` for writable internal state, `computed()` for derived state, and `input()` for values supplied by a parent.

## Optional and required inputs

### 9. How do I declare an optional input with a default?

Give `input()` an initial value, such as `readonly size = input(16)`. The input signal exposes the default value unless Angular supplies another input value.

### 10. How do I declare an optional input that may be undefined?

Use a type argument without an initial value, such as `readonly caption = input<string>()`. The value can be undefined, so the component should handle that case.

### 11. How can a template handle an optional input?

Use a fallback expression such as `{{ caption() ?? 'No caption' }}`, or derive a fallback with a computed signal.

### 12. How do I declare a required signal input?

Use `input.required<T>()`, for example `readonly user = input.required<User>()`. A required input does not need a fake initial value.

### 13. What happens if a required input is omitted from a component use in a template?

Angular's template checking reports an error because the required component API is incomplete.

### 14. Does a required input's signal type include undefined?

Not automatically. `input.required<User>()` exposes a signal of `User` rather than `User | undefined`.

### 15. Can a required input be null?

Only if its declared type allows null. Required means the caller must provide the input; it does not automatically prohibit null when the type includes null.

### 16. Does the template compiler check required inputs set through dynamic component creation?

No, the template compiler cannot inspect arbitrary programmatic creation calls. Code creating a component dynamically must set its required inputs through the appropriate component API.

### 17. When should an input be required instead of optional?

Make it required when the component cannot operate correctly without the value and callers should always provide it. Use an optional input when absence is valid and the component has a clear fallback or behavior for it.

## Reading and deriving values

### 18. Can I use an input signal in computed?

Yes. A computed signal can read one or more input signals and derive a value:

~~~ts
readonly fullName = computed(() =>
  [this.firstName(), this.lastName()].join(' ')
);
~~~

### 19. Why use computed instead of copying an input into a field?

A computed value stays derived from the current input values. Copying an input into mutable state can create duplicate state that becomes stale unless carefully synchronized.

### 20. Can the child mutate an object received as an input?

The input signal itself is read-only, but that does not necessarily make the object inside it deeply immutable. Treat parent-owned input objects as read-only and communicate requested changes through outputs or a model input instead of mutating shared data.

### 21. What if a component needs to change an input value from user interaction?

A normal input is parent-to-child and read-only. Use a model input if the component should both receive and write the value through two-way binding, or emit an event so the parent updates its own state.

## Aliases and compatibility

### 22. What does an input alias do?

It changes the public name used in the template while keeping the TypeScript property name. For example, `value = input(0, { alias: 'sliderValue' })` is used as `[sliderValue]` by the parent and as `this.value()` in the component.

### 23. When is an alias useful?

An alias can preserve an existing template API during a property rename or avoid a collision with a native DOM property. Avoid aliases without a clear compatibility or naming reason.

### 24. Does an alias change how the component reads the signal?

No. TypeScript continues using the class property name. The alias changes the public template binding name.

### 25. Does Angular still support decorator-based @Input?

Yes. The decorator-based `@Input` API remains supported. Angular recommends signal inputs for new projects, but existing projects can continue using or migrate from `@Input` incrementally.

### 26. Is the parent binding syntax different for @Input and signal inputs?

No. Both APIs use the same component binding syntax, such as `[title]="pageTitle"`. The difference is how the component declares and reads the value.

## Input transforms

### 27. What is an input transform?

A transform is a function Angular applies when it writes an incoming value to an input. It can normalize a wider range of caller values into a consistent type used by the component.

### 28. When does Angular run an input transform?

Angular runs it as it sets the input value. The component reads the transformed result from its input signal.

### 29. What type does the input signal expose when a transform is used?

The signal exposes the transform's return type. For a transform from a string or undefined to a string, the component reads a string.

### 30. How does Angular determine which values a parent may bind to a transformed input?

The transform function's parameter type determines the accepted binding type, while its return type determines the type the component reads.

### 31. Should input transforms be pure?

Yes. A transform should return a result based on its argument without side effects or dependence on changing external state. Impure transforms can make input updates unpredictable.

### 32. Must a transform be statically analyzable?

Yes. Angular needs to analyze the transform declaration at build time. Use a stable function reference; do not choose a transform dynamically or conditionally.

### 33. What is a good use for an input transform?

Normalize an incoming representation at the component boundary, such as trimming optional text or converting a boolean-like HTML attribute value into a boolean.

### 34. What should I use for a value derived from multiple inputs?

Use `computed`. Transforms normalize one incoming value; derived state combines or reinterprets values.

### 35. What does booleanAttribute do?

It converts common HTML-style boolean attribute values to a boolean. Attribute presence generally means true, while the literal string `"false"` is treated as false.

### 36. What does numberAttribute do?

It attempts to parse an incoming value as a number. If parsing fails, the result is `NaN`. Add explicit validation when the application needs a finite value or a specific range.

### 37. Can a required input also have a transform?

Yes. A required input can use transform options. The caller must still provide it, and the component reads the transformed value.

### 38. Can I use a transform to validate all external data?

A transform can normalize its input, but it is not automatically comprehensive domain validation. Check validity and constraints explicitly where the application requires them.

## Common errors and design choices

### 39. Why is a required input showing an error in a template?

The component is being used without binding every required input. Add the missing binding, or make the input optional if the component can legitimately work without it.

### 40. Why is my transformed input's parent binding rejected by TypeScript?

The value may not match the transform function's parameter type. Expand that parameter type to represent the values callers should be allowed to provide, while keeping the transform's return type aligned with what the component expects.

### 41. Why is the input signal blank when I omit the binding?

An optional input declared without a default can be undefined. Add a default to `input()` or handle undefined in the template or computed logic.

### 42. Why can't I call set() on an input signal?

Input signals are read-only from the child. Use `signal()` for writable internal state or `model()` when a value is intentionally writable and synchronized with a parent.

### 43. Should I use a setter to trim an input?

Prefer a pure input transform for simple boundary normalization. Avoid expensive or side-effecting setters; Angular can set inputs more than once.

### 44. Do required inputs replace runtime validation?

No. Required inputs catch missing bindings in checked templates. They do not validate untrusted API data, dynamic values, or business rules such as string length and numeric range.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="024-built-in-directives-and-structural-directive-syntax.md">024 - 24. Built-in directives and structural directive syntax</a></td><td><a href="../lessons/025-signal-inputs-required-inputs-and-input-transforms.md">Lesson</a></td><td><a href="../examples/025-signal-inputs-required-inputs-and-input-transforms.md">Example</a></td><td><a href="026-component-outputs-and-custom-events.md">026 - 26. Component outputs and custom events</a></td>
</tr>
  </tbody>
</table>
