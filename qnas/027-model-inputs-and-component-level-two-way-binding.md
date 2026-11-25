# 27. Q&A: Model inputs and component-level two-way binding

## 1. What is a model input?

A model input is a writable signal exposed by a component or directive as an input/output pair. The parent can provide its value, and the component can update it. When the parent uses two-way binding, those updates are written back to the parent's target.

## 2. How do I declare a model input?

Import `model` from `@angular/core` and declare it as a class property initializer:

~~~ts
readonly count = model(0);
~~~

This creates a writable model signal with an initial value of zero.

## 3. What does `model()` return?

It returns a `ModelSignal<T>`, which behaves as a writable signal and is also registered as the component's input/output contract. You can read it by calling it, and update it with `.set()` or `.update()`.

## 4. How is `model()` different from `input()`?

A signal input created with `input()` is read-only inside the child. A model created with `model()` is writable inside the child and exposes a corresponding change output so a parent can receive updates.

## 5. What input and output names does `model()` expose?

For a model named `count`, Angular exposes an input named `count` and generates an output named `countChange`. You do not declare a separate `output()` for that same model.

## 6. When does the model's change output emit?

When the component writes a value with the model signal's `.set()` or `.update()` method, Angular emits through the generated change output. With a parent two-way binding, the parent's target receives that value.

## 7. How do I read the model value inside the component?

Call the model signal like any other signal:

~~~ts
const currentValue = this.count();
~~~

In a template, render it as `{{ count() }}`.

## 8. How do I update the model to a known value?

Use `.set()`:

~~~ts
this.count.set(10);
~~~

## 9. How do I update the model based on its current value?

Use `.update()`:

~~~ts
this.count.update((current) => current + 1);
~~~

This form is useful for incrementing, toggling, or deriving the next value.

## 10. How do I declare a model with an initial value?

Pass the default value to `model()`:

~~~ts
readonly checked = model(false);
~~~

The initial value also lets TypeScript infer the model's type.

## 11. How do I declare a model that starts as `undefined`?

Call `model<T>()` without an initial value:

~~~ts
readonly selectedId = model<string>();
~~~

Its value type includes `undefined`, because the initial value is undefined.

## 12. How do I make a model required?

Use `model.required<T>()`:

~~~ts
readonly value = model.required<number>();
~~~

The component consumer must bind the model. Angular reports a missing required model as a template/build-time error.

## 13. What does component two-way binding look like?

Use the `[()]` syntax with the model's public name:

~~~html
<app-counter [(count)]="quantity" />
~~~

The shorthand binds the parent's value into `count` and handles the generated `countChange` output.

## 14. What is the expanded form of `[(count)]`?

For a plain writable parent property, it is conceptually equivalent to:

~~~html
<app-counter [count]="quantity" (countChange)="quantity = $event" />
~~~

The two-way form combines the property and event bindings into one expression.

## 15. Can the parent's two-way binding target be a plain property?

Yes. Angular can update a writable property such as `quantity = 3` when the child emits a new model value.

## 16. Can the parent's two-way binding target be a signal?

Yes. Pass the writable signal instance:

~~~html
<app-counter [(count)]="quantity" />
~~~

If `quantity` is a signal, do not write `quantity()` in the binding. Call it in expressions that read and display its current value, such as `{{ quantity() }}`.

## 17. Why do I pass the signal instance instead of `quantity()`?

The two-way binding needs a writable target it can update. `quantity` is the writable signal instance; `quantity()` is only the current value returned by reading that signal.

## 18. Does `model()` make a component two-way bound automatically?

No. The component must declare a model, and the parent must bind it with `[(modelName)]` to synchronize both directions. Without that parent binding, there is no parent target to update.

## 19. Can the parent listen to the generated change output directly?

Yes. A `count` model exposes `countChange`, so a parent may write `[count]="quantity" (countChange)="handleCountChange($event)"`. Use `[(count)]` when the behavior is simply to keep the two values synchronized; use the explicit output binding when the parent needs to handle the change itself.

## 20. Should every input be a model input?

No. Use a regular input when the parent supplies a value and the child should not write it back. Use a model when the component's contract explicitly allows it to edit that same value.

## 21. When is a model input a good fit?

It is a good fit for interactive controls whose purpose is to edit a value, such as a custom slider, checkbox, date picker, or select control. The child can implement the interaction while the parent can still own or observe the value.

## 22. When is a separate output better than a model?

Use a separate output for a meaningful event that is not simply an edit to one shared value, such as `saved`, `submitted`, or `selectionCommitted`. Outputs let the parent decide what action to take and can carry event-specific data.

## 23. Can I combine a model input with other inputs and outputs?

Yes. A component can use regular inputs for configuration, model inputs for values it edits, and outputs for other semantic events. Give each public property a clear purpose.

## 24. Can I use `model()` in a directive?

Yes. Components and directives can declare model inputs. Consumers bind to the directive's model on the element where the directive is applied.

## 25. Where must I call `model()`?

Declare it in a component or directive class property initializer. It is a compiler-recognized API. Do not call it in a constructor, method, or conditional branch.

## 26. Can model inputs use input transforms?

No. Angular model inputs do not support input transforms. If you need to normalize incoming data, consider a regular input plus a separate output, or perform normalization in an explicit part of your component's update logic.

## 27. Can a model have an alias?

Yes. `model()` accepts an alias option similar to a standard input. The TypeScript property keeps its code name, while the alias is the public template-facing name. Use aliases only when they help with compatibility or avoid a naming collision.

## 28. Does `[(count)]` mean the same thing as `[(ngModel)]`?

They use the same general `[()]` two-way binding punctuation, but they are different APIs. `[(count)]` binds to a component model named `count`. `[(ngModel)]` uses Angular Forms' `NgModel` directive and its forms setup.

## 29. Do I need `FormsModule` to use a component model?

No. A component model declared with `model()` uses Angular's component binding API. `FormsModule` is needed for `[(ngModel)]`, not for `[(count)]` on a component model.

## 30. Can I write to the parent-owned model from the child without a user action?

Technically the component can call `.set()` or `.update()` whenever its intended behavior requires it. Keep the public contract clear and update the model only when the component is meant to change that value.

## 31. How does a child render the same value it updates?

Read the model signal in its template, for example `<span>{{ count() }}</span>`. The signal provides the current value to the child view and participates in Angular's reactive rendering.

## 32. Can I use `asReadonly()` with a model signal?

`ModelSignal` provides signal behavior, including a read-only view through `.asReadonly()`. Expose a read-only signal to internal helpers when they only need to observe the value; keep the model itself as the public two-way API.

## 33. What is the main difference between `[(value)]="value"` and `[value]="value"`?

The square-bracket form sends a value into the child. The `[()]` form also handles the child's matching change output and writes changes back to the parent target.

## 34. Does changing a regular `input()` property inside the child provide two-way binding?

No. Signal inputs are read-only in the child. For two-way component value editing, use a model input or explicitly design an input and output pair.

## 35. Is `model()` stable for production use?

Yes. Angular documents the `model()` initializer API as stable since Angular v19. Check the compatibility requirements of the Angular version used by your project.

## 36. Where can I read the official documentation?

See Angular's [model inputs guide](https://angular.dev/guide/components/inputs#model-inputs), [two-way binding guide](https://angular.dev/guide/templates/two-way-binding), [`model()` API reference](https://angular.dev/api/core/model), and [`ModelSignal` API reference](https://angular.dev/api/core/ModelSignal).
