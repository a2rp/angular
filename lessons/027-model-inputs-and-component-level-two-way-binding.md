# 27. Model inputs and component-level two-way binding

A model input lets a component receive a value from its parent and publish changes to that value back to the parent. It is useful for interactive controls whose main job is to edit a value, such as a slider, checkbox, date picker, or custom select.

Declare a model with `model()` from `@angular/core`. It returns a writable signal and exposes a matching input/output pair. The component reads the current value by calling the model signal and changes it with `.set()` or `.update()`.

~~~ts
import { Component, model } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <button type="button" (click)="decrement()">−</button>
    <span>{{ count() }}</span>
    <button type="button" (click)="increment()">+</button>
  `,
})
export class Counter {
  readonly count = model(0);

  increment(): void {
    this.count.update((current) => current + 1);
  }

  decrement(): void {
    this.count.update((current) => current - 1);
  }
}
~~~

A parent can bind a writable value with Angular's two-way binding syntax:

~~~html
<app-counter [(count)]="quantity" />
~~~

The syntax `[(count)]` combines a property binding with an event binding. Angular passes the parent's `quantity` value into the child's `count` model and updates `quantity` when the child changes its model.

## Learning goals

After this lesson, you should be able to:

- Explain how a model input combines an input and a change output.
- Declare optional, initialized, and required models with `model()`.
- Read and update a model signal with `()`, `.set()`, and `.update()`.
- Bind a child model to parent state with `[(modelName)]`.
- Use a writable signal or a plain property as the parent binding target.
- Decide when a model input is clearer than a one-way input and a separate output.

## What a model input provides

A model input is a writable signal exposed at the component boundary as both:

1. An input, so a parent can provide the initial or current value.
2. A corresponding output, whose name is the model input name followed by `Change`.

For example, `count = model(0)` exposes `count` and `countChange`. The model input can be changed from inside the component with `.set()` or `.update()`. Angular emits the corresponding change event when the model is written, allowing a parent two-way binding to receive the new value.

A model signal is different from a regular signal declared with `signal()`: `model()` also makes the value part of the component's public input/output API. It is different from `input()`, whose returned signal is read-only inside the child.

## Declare initialized, optional, and required models

Use an initial value when the component has a sensible default:

~~~ts
readonly checked = model(false);
readonly pageSize = model(20);
~~~

TypeScript infers the model type from the initial value. You can also specify a generic explicitly:

~~~ts
readonly selectedId = model<string | null>(null);
~~~

Without an initial value, `model<T>()` starts as `undefined`, so the type includes `undefined`:

~~~ts
readonly selectedId = model<string>();
~~~

Use `model.required<T>()` when the consumer must provide the value:

~~~ts
readonly value = model.required<number>();
~~~

Angular reports a template usage that omits a required model binding as a build-time error. A required model has no default value, so the parent must supply it.

As with other Angular initializer APIs, declare `model()` as a component or directive class property initializer. Do not create it in a method, constructor, or conditional block.

## Read and update the model

A model is a signal. Read its current value by calling it:

~~~ts
const currentCount = this.count();
~~~

Replace its value with `.set()`:

~~~ts
this.count.set(10);
~~~

Derive a new value from the current one with `.update()`:

~~~ts
this.count.update((current) => current + 1);
~~~

Updates made by the child are sent through the model's corresponding change output. With a parent two-way binding, the parent target is updated as well. The child can render its own current value from the same signal, for example `{{ count() }}` in a template.

## Bind the model from a parent

Angular's component two-way binding syntax combines `[input]` and `(inputChange)`:

~~~html
<app-counter [(count)]="quantity" />
~~~

Conceptually, this is similar to:

~~~html
<app-counter [count]="quantity" (countChange)="quantity = $event" />
~~~

Use the two-way form when the parent wants to provide the value and accept updates under one binding. The parent target must be something Angular can update.

### Bind to a plain property

A regular writable property works:

~~~ts
export class App {
  quantity = 3;
}
~~~

~~~html
<app-counter [(count)]="quantity" />
<p>Quantity in the parent: {{ quantity }}</p>
~~~

### Bind to a signal

A writable signal also works. Bind the signal itself, not the result of calling it:

~~~ts
import { signal } from '@angular/core';

export class App {
  quantity = signal(3);
}
~~~

~~~html
<app-counter [(count)]="quantity" />
<p>Quantity in the parent: {{ quantity() }}</p>
~~~

The binding expression is `quantity`, the signal instance. Call `quantity()` in places where the template needs to read and display the signal's current value.

## Use one-way binding or a separate output when appropriate

Two-way binding is convenient, but it is not required for every child value. If the parent only supplies a value and the child should not write it back, use a regular input:

~~~ts
readonly label = input.required<string>();
~~~

If the child reports an action or a richer event rather than editing one shared value, use a regular output:

~~~ts
readonly saved = output<SaveResult>();
~~~

Use a model when the component's public contract is specifically “parent supplies this value; component may update it.” Prefer separate inputs and outputs when the events have their own meaning, need different payloads, or should trigger explicit parent logic rather than synchronize one value.

## Aliases and limits

A model can be marked required or assigned an alias, similar to a standard input. For example:

~~~ts
readonly value = model(0, { alias: 'sliderValue' });
~~~

The public model name can then be used in the parent template while the TypeScript member remains `value`. Keep aliases rare so the class and template API remain easy to match.

Model inputs do not support input transforms. If you need to normalize external input data, decide whether the value should instead be represented by a regular input and an explicit output, or normalize it in a well-defined part of the component's own update logic.

Model inputs are available in components and directives. A `model()` call is a compiler-recognized initializer API; keep it at class property initialization.

## Relationship to `[(ngModel)]`

The `[()]` punctuation is Angular's general two-way binding syntax. For a component model, write `[(count)]` using that component model's public name. `[(ngModel)]` is a separate forms directive for form controls and requires the forms setup appropriate to the application. Do not add `ngModel` just to bind one component model.

## Common mistakes

- **Using `input()` and trying to write to it from the child:** input signals are read-only. Use `model()` when the child must publish changes to that same value.
- **Calling `model()` from a method:** declare it as a class property initializer.
- **Binding to `quantity()` instead of `quantity`:** when the parent target is a signal, pass the writable signal instance to `[(count)]`; call the signal only to read its value.
- **Expecting `model()` to create two-way binding without a parent binding:** the parent must use `[(count)]` or explicitly bind `[count]` and `(countChange)`.
- **Forgetting the generated output name:** a `count` model has a `countChange` output.
- **Using a model for every event:** choose regular outputs for semantic events that are not edits to a shared value.
- **Expecting input transforms to work on a model:** model inputs do not support transforms.
- **Using `[(ngModel)]` on a component model:** bind to the component's model property, such as `[(count)]`.

## Practical checklist

Before adding a model input, confirm that:

- The component genuinely needs to both receive and update the same value.
- The model type and optional/default/required behavior are intentional.
- The component updates it with `.set()` or `.update()`.
- The parent binds a writable property or writable signal using `[(modelName)]` when it needs synchronization.
- If the parent uses a signal, the binding passes the signal instance, while reads call the signal.
- A regular input/output pair would not communicate the contract more clearly.
- No input transform is being assumed for the model.

## Further reading

- [Model inputs in the component inputs guide](https://angular.dev/guide/components/inputs#model-inputs)
- [Two-way binding](https://angular.dev/guide/templates/two-way-binding)
- [`model()` API](https://angular.dev/api/core/model)
- [`ModelSignal` API](https://angular.dev/api/core/ModelSignal)
