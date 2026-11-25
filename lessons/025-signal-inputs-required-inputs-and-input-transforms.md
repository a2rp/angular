# 25. Signal inputs, required inputs, and input transforms

A component input is a value supplied by its parent. Angular's signal-based input API declares those values with `input()` and exposes each one as a read-only signal. Use `input.required()` when the parent must provide a value, and an input transform when an incoming representation should be normalized into the type the component uses.

~~~ts
import { Component, computed, input } from '@angular/core';

@Component({
  selector: 'app-user-badge',
  standalone: true,
  template: '<p>{{ label() }} — {{ isActive() ? "Active" : "Inactive" }}</p>',
})
export class UserBadge {
  readonly label = input('Guest');
  readonly isActive = input.required<boolean>();
  readonly displayName = computed(() => this.label().trim());
}
~~~

The parent controls the inputs. The component reads their current values by calling the signals.

## Learning goals

After this lesson, you should be able to:

- Declare optional signal inputs with and without a default value.
- Declare required inputs and understand how Angular checks their use.
- Read an input signal in TypeScript and a component template.
- Derive values from inputs with `computed`.
- Add an input alias without changing the TypeScript property name.
- Use pure transforms to normalize values at the component boundary.
- Apply Angular's `booleanAttribute` and `numberAttribute` transforms.
- Explain input transform types and when a transform is inappropriate.
- Recognize that decorator-based `@Input` remains supported.
- Distinguish a one-way input from a writable model input.

## 1. Inputs define a component's parent-to-child API

A parent binds values to a child component's inputs:

~~~html
<app-user-badge [label]="currentUser.name" [isActive]="currentUser.active" />
~~~

The child declares which values it accepts. Inputs are public component API: their names and expected types affect every template that uses the component. Keep names clear, document required values through the API, and avoid aliases unless they solve a real compatibility or naming problem.

Signal inputs are declared with the `input` function from `@angular/core`. Angular recognizes calls to `input()` and `input.required()` in component and directive class field initializers as input declarations.

## 2. Optional signal inputs

An optional input can have a default value:

~~~ts
readonly size = input(16);
~~~

The property is an input signal whose value is a number. Read it by calling it:

~~~ts
const currentSize = this.size();
~~~

You can also declare an optional input without providing a default:

~~~ts
readonly caption = input<string>();
~~~

In this form, the input value may be `undefined` until the parent provides it. Handle that possibility in TypeScript and templates:

~~~html
<p>{{ caption() ?? 'No caption provided' }}</p>
~~~

A default value is useful when the component has a natural fallback. Use the no-default form when the absence of a value is meaningful and should remain explicit.

## 3. Required inputs

Use `input.required<T>()` when the component cannot be used correctly without a value:

~~~ts
readonly product = input.required<Product>();
~~~

A required input does not need an artificial initial value. Angular checks component usage in templates and reports an error when a required input is missing:

~~~html
<!-- The required product input is missing -->
<app-product-card />
~~~

Provide the input in the parent template:

~~~html
<app-product-card [product]="featuredProduct" />
~~~

Required inputs improve the component contract and template checking. They do not add `undefined` to the signal's type: if the input is required to be a `Product`, code that reads it receives a `Product`.

When creating a component dynamically through a programmatic API, make sure its required inputs are set through that API. The template compiler can only check uses it can see in templates.

## 4. Read an input signal

Signal inputs are read-only. In TypeScript, call the signal to read its current value:

~~~ts
readonly firstName = input.required<string>();

greeting(): string {
  return 'Hello, ' + this.firstName();
}
~~~

In the component's own template, call it as well:

~~~html
<p>Hello, {{ firstName() }}.</p>
~~~

Do not assign to an input signal. The parent owns the bound value; when the parent's binding changes, Angular updates the child input signal. If the child must propose or emit a new value as part of a two-way component API, use a model input, covered in a later chapter.

## 5. Derive values with computed

Use `computed` when a value is derived from one or more inputs:

~~~ts
import { computed, input } from '@angular/core';

readonly firstName = input.required<string>();
readonly lastName = input.required<string>();

readonly fullName = computed(() =>
  [this.firstName(), this.lastName()].join(' ')
);
~~~

The computed signal updates when either input changes. This keeps derived state connected to its source values without copying inputs into separate mutable fields.

A computed value is a good fit when the result has a different meaning from its inputs, such as a formatted label, a filtered view, or a calculated total. Do not use an input transform to hold derived state.

## 6. Alias an input

An alias changes the public name used in templates while preserving the class property name:

~~~ts
readonly value = input(0, { alias: 'sliderValue' });
~~~

A parent binds using the alias:

~~~html
<app-slider [sliderValue]="volume" />
~~~

Inside the component, TypeScript still reads `this.value()`. Aliases can help preserve an existing template API during a rename or avoid a collision with a native DOM property. Avoid aliases when the class and template names can naturally match.

## 7. Transform an incoming value

A transform normalizes the value as Angular writes it to the input. The value accepted by the template can be wider than the value the component reads.

~~~ts
function trimLabel(value: string | undefined): string {
  return value?.trim() ?? '';
}

@Component({
  selector: 'app-labeled-value',
  standalone: true,
  template: '<span>{{ label() }}</span>',
})
export class LabeledValue {
  readonly label = input('', { transform: trimLabel });
}
~~~

The parent can pass a string or `undefined`. Angular applies `trimLabel`, and the component reads a string from `label()`. The transform runs when Angular sets the input, not whenever the component happens to read the signal.

Input transforms should be pure and statically analyzable. A pure transform returns a result from its argument without changing external state or performing side effects. Declare the transform as a stable function and do not select it conditionally at runtime.

Use a transform for boundary normalization, such as trimming text or coercing a string attribute to a boolean. Use `computed` for derived values and an effect or another explicit workflow for side effects.

## 8. Use booleanAttribute and numberAttribute

Angular provides common transforms in `@angular/core`:

~~~ts
import {
  booleanAttribute,
  Component,
  input,
  numberAttribute,
} from '@angular/core';

@Component({
  selector: 'app-notice',
  standalone: true,
  template: '@if (!hidden()) { <aside>{{ message() }}</aside> }',
})
export class Notice {
  readonly hidden = input(false, { transform: booleanAttribute });
  readonly message = input.required<string>();
  readonly retryCount = input(0, { transform: numberAttribute });
}
~~~

The boolean transform follows HTML-style boolean attribute conventions: the presence of the attribute normally means true, while the literal string `"false"` is treated as false.

The number transform attempts to parse a numeric value. If parsing fails, it produces `NaN`. If the component requires a valid number within a particular range, validate it deliberately; a transform does not automatically make arbitrary input valid.

## 9. Transform types

The transform function's parameter type determines which values a template may supply. The value returned by the transform determines the type exposed by the input signal.

For example, a normalizer can accept a string or `undefined` and always return a string:

~~~ts
function normalizeLabel(value: string | undefined): string {
  return value?.trim() ?? '';
}

readonly label = input('', { transform: normalizeLabel });
~~~

The component can safely call string methods on `label()`, while template callers may pass the broader accepted input type. This is useful for HTML-like values that can be absent or arrive as strings.

Do not annotate a transform parameter more narrowly than the values callers are allowed to bind. Angular's template type checking uses the transform's accepted type to check the parent binding.

## 10. Required inputs with transforms

A required input can also have a transform. The input remains required from the caller's perspective, while the component reads the transformed result:

~~~ts
function normalizeName(value: string): string {
  return value.trim();
}

readonly name = input.required({ transform: normalizeName });
~~~

Use this combination when the value must be supplied but still needs normalization. A required declaration is not a replacement for runtime validation of external data; it ensures a component use provides the input in checked templates.

## 11. Signal inputs and decorator-based @Input

The decorator-based API remains supported:

~~~ts
import { Input } from '@angular/core';

@Input({ required: true }) title = '';
~~~

The parent-facing binding syntax is the same, but the class member is a normal property rather than an input signal. Angular recommends signal inputs for new projects, while existing applications can continue using `@Input` and can migrate incrementally.

Avoid declaring two inputs with the same public name. Choose one declaration style for each component property and follow the conventions used by the surrounding codebase.

## 12. Input declarations are static

Declare `input()` calls directly in component or directive field initializers:

~~~ts
readonly title = input('Untitled');
readonly record = input.required<RecordData>();
~~~

Do not call `input()` inside a method, a constructor, a conditional branch, or a loop. Angular analyzes input declarations statically to build the component's input API.

Signal input functions are for declaring input properties; they are not general-purpose signal constructors. Use `signal()` for writable internal state and `computed()` for values derived from other signals.

## 13. Common mistakes

### Reading the property without calling it

An input signal is a function-like signal. Use `this.title()` in TypeScript and `title()` in the template to read its value.

### Trying to write to an input

Inputs are controlled by the parent and are read-only in the child. Use local state for internal changes or a model input when the component needs a two-way binding contract.

### Giving a required input a fake fallback

`input.required<T>()` exists so the component can require a value without an artificial default. Do not use a placeholder just to silence a missing-value concern.

### Forgetting the parent binding

If a required input is missing at a template use site, Angular reports an error. Add the binding or reconsider whether the input should actually be optional.

### Using a transform for derived state

Transforms normalize an incoming value. If the result combines inputs or has a different semantic meaning, prefer `computed`.

### Making a transform impure

Do not mutate component state, access the DOM, perform I/O, or depend on changing external state in a transform. Keep it pure and deterministic.

### Assuming numberAttribute validates a domain value

It parses a numeric representation but can return `NaN`. Add explicit validation when the application requires a finite value, a minimum, or a maximum.

### Adding aliases without a reason

Aliases add another name developers must understand. Use one when preserving a public API or avoiding a real naming conflict, not as a default naming convention.

## Practical checklist

Before exposing a component input, check that:

- The input is optional or required for a clear reason.
- Optional inputs have an appropriate default or handle `undefined`.
- The component reads signal inputs by calling them.
- Required values are bound at every checked template use site.
- A transform performs only pure boundary normalization.
- The transform parameter accepts every intended parent-binding type.
- `computed` is used for values derived from inputs.
- Aliases are deliberate and consistent with the component's public API.
- Two-way interaction uses a model input rather than writing to a read-only input.

## Further reading

- [Accepting data with input properties](https://angular.dev/guide/components/inputs)
- [input API](https://angular.dev/api/core/input)
- [InputSignalWithTransform API](https://angular.dev/api/core/InputSignalWithTransform)
- [Migration to signal inputs](https://angular.dev/reference/migrations/signal-inputs)
