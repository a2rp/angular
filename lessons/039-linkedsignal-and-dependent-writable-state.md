# 39. `linkedSignal` and dependent writable state

A `computed()` signal is read-only. A regular `signal()` is writable, but it does not automatically reset when another signal it was initialized from changes. `linkedSignal()` bridges those needs: it creates a writable signal whose value is initialized and recalculated from reactive source state, while still allowing a user or component method to change it between source updates.

This is useful when a value normally follows another value but can be temporarily chosen or edited. Examples include the selected item in a changing option list, an editable form draft derived from loaded data, or a default that should be recalculated when its configuration changes.

`linkedSignal()` is stable starting in Angular 20. Use it when the project version supports the API.

## Learning goals

After this lesson, you should be able to:

- Explain when `linkedSignal()` is useful compared with `signal()` and `computed()`.
- Create a linked signal with a simple computation.
- Manually write to a linked signal with `set()` or `update()`.
- Use the source/computation form to preserve or repair a value when its source changes.
- Read the previous source and previous linked value safely.
- Recognize when a linked signal resets and when the manual value persists.
- Avoid copying derived state through an `effect()`.

## The problem: writable state can become stale

Suppose a user selects a shipping option from a list. The list can later change, but the selected option is also writable:

~~~ts
const shippingOptions = signal(['Ground', 'Air', 'Sea']);
const selectedOption = signal(shippingOptions()[0]);

selectedOption.set('Sea');
shippingOptions.set(['Email', 'Will Call']);
~~~

`selectedOption` still contains `'Sea'`, even though that value is no longer available. A plain signal has no relationship to the list after its initial value is chosen.

A `computed()` value can track the list, but a computed value cannot be manually changed to reflect the user's choice. `linkedSignal()` is for state that should be recalculated when its source changes and remain writable in the meantime.

## Create a linked signal from a computation

Pass a computation function that reads the source signal:

~~~ts
import { linkedSignal, signal } from '@angular/core';

const shippingOptions = signal(['Ground', 'Air', 'Sea']);
const selectedOption = linkedSignal(() => shippingOptions()[0] ?? null);

console.log(selectedOption()); // 'Ground'

selectedOption.set('Sea');
console.log(selectedOption()); // 'Sea'

shippingOptions.set(['Email', 'Will Call']);
console.log(selectedOption()); // 'Email'
~~~

The linked signal starts with the computation result. The user can then set a different value. When the source dependency changes, Angular runs the computation again and replaces the linked signal's value with that result.

A `linkedSignal()` returns a writable signal, so it supports the same read, `set()`, and `update()` operations as other writable signals. Its computed relationship determines what happens when a tracked source changes.

## Preserve a still-valid choice

The simple form resets to its computed default whenever a dependency changes. If a user's current selection should be kept when it still exists in a refreshed list, use the source/computation form and inspect the previous state:

~~~ts
interface ShippingOption {
  id: number;
  name: string;
}

const shippingOptions = signal<ShippingOption[]>([
  { id: 1, name: 'Ground' },
  { id: 2, name: 'Air' },
  { id: 3, name: 'Sea' },
]);

const selectedOption = linkedSignal<ShippingOption[], ShippingOption | null>({
  source: shippingOptions,
  computation: (newOptions, previous) => {
    const previousId = previous?.value?.id;

    return (
      newOptions.find((option) => option.id === previousId) ??
      newOptions[0] ??
      null
    );
  },
});
~~~

The explicit type arguments are `<Source, Destination>`: here the source is an array of options and the linked value is one option or `null`. The `previous` argument is optional because there is no prior linked value during the initial computation.

This calculation looks up the previously selected option by its stable `id` in the new list. If it still exists, it returns the new object from the refreshed list. If it was removed, the calculation selects the first available option. If the list is empty, it returns `null`.

The `previous` object contains:

- `previous.source`: the previous source value.
- `previous.value`: the previous value of the linked signal.

Use optional chaining because `previous` is undefined during initial computation and the previous linked value may itself be `null`.

## Manual updates and source updates

A linked signal is writable. A user action can update it directly:

~~~ts
function chooseShipping(optionId: number): void {
  const option = shippingOptions().find((item) => item.id === optionId);

  if (option) {
    selectedOption.set(option);
  }
}
~~~

If the source does not change, that manual choice remains the current value. When the source changes, the linked signal runs its computation with the new source and can preserve, replace, or clear the choice according to your policy.

Think through both transitions:

1. **User changes the linked value:** `set()` or `update()` changes the current writable value.
2. **A tracked source changes:** the computation recalculates the linked value, using the new source and optionally the previous state.

The computation should return a valid destination value for every possible source state, including empty lists, deleted records, or missing selections.

## Sources and reactive dependencies

In the source/computation form, `source` identifies the main signal value passed to the computation. The computation can also read other signals. Those reads become reactive dependencies too, so a change to any of them can cause recomputation.

~~~ts
const options = signal(['Small', 'Medium', 'Large']);
const fallbackIndex = signal(0);

const selectedSize = linkedSignal({
  source: options,
  computation: (availableSizes) =>
    availableSizes[fallbackIndex()] ?? null,
});
~~~

Here, a change to either `options` or `fallbackIndex` can recompute the linked value. Keep the callback synchronous and focused on selecting or preparing the destination state.

## When to use `signal`, `computed`, or `linkedSignal`

| Need | Use | Can it be manually written? | What happens when sources change? |
| --- | --- | --- | --- |
| Independent state | `signal()` | Yes | It stays as written until explicitly updated |
| A value that must always be derived | `computed()` | No | It recalculates from tracked sources |
| Dependent state that should be writable between source changes | `linkedSignal()` | Yes | Its computation recalculates from tracked sources |

For example:

- Use `computed(() => subtotal() * taxRate())` for a total that must always be derived.
- Use `signal('')` for a search string that is independently edited.
- Use `linkedSignal(() => options()[0] ?? null)` for a selected option that defaults from a list, can be changed by the user, and must be reset or repaired when that list changes.

Avoid using an `effect()` to watch a source and copy a value into a separate signal. That makes a state relationship indirect and can introduce timing and synchronization problems. Use `computed()` when the value is never independently writable, and `linkedSignal()` when it is both dependent and writable. Use effects for suitable side effects that synchronize signal state with non-signal APIs, not as a general state-copy mechanism.

## Expose a readonly view when appropriate

A service can keep the linked signal private and expose a readonly view together with operations that define how consumers may choose values:

~~~ts
import { Injectable, linkedSignal, signal } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class ShippingState {
  private readonly _options = signal<ShippingOption[]>([]);

  private readonly _selected = linkedSignal<ShippingOption[], ShippingOption | null>({
    source: this._options,
    computation: (options, previous) =>
      options.find((option) => option.id === previous?.value?.id) ??
      options[0] ??
      null,
  });

  readonly options = this._options.asReadonly();
  readonly selected = this._selected.asReadonly();

  choose(optionId: number): void {
    const option = this._options().find((item) => item.id === optionId);

    if (option) {
      this._selected.set(option);
    }
  }
}
~~~

Consumers can read `selected()` but cannot call write methods on the readonly view. The service still controls updates through `choose()`. `asReadonly()` does not deep-freeze object values, so continue to avoid mutating option objects in place.

## Advanced: customize writes to update the source

By default, `set()` and `update()` write to the linked signal's own current value. In advanced cases, you can provide a `set` option to translate a write back to the source of truth:

~~~ts
const celsius = signal(0);

const fahrenheit = linkedSignal(
  () => (celsius() * 9) / 5 + 32,
  {
    set: (nextFahrenheit) => {
      celsius.set(((nextFahrenheit - 32) * 5) / 9);
    },
  },
);

fahrenheit.set(212);
console.log(celsius()); // 100
console.log(fahrenheit()); // 212
~~~

The setter option receives the new destination value and a `rawSet` function. It can update the source so the linked computation produces the corresponding destination value. Use `rawSet` only when you intentionally need to write the linked value directly without relying on a source recomputation. This customization is not needed for ordinary linked state; it should preserve a clear source-of-truth rule.

## Common mistakes

- Using a plain signal initialized from another signal and expecting it to stay synchronized.
- Using `computed()` for a value that the user must be able to change directly.
- Using `linkedSignal()` for a value that should never be manually written; use `computed()` instead.
- Copying source state into a destination signal with an `effect()`.
- Assuming a manual linked-signal value will survive every source change unchanged.
- Returning a stale object from `previous.value` instead of finding its corresponding object in the new source list.
- Ignoring missing selections or an empty source collection.
- Forgetting that the `previous` argument can be undefined on first computation.
- Mutating source objects or arrays in place instead of replacing them immutably.
- Assuming that `linkedSignal()` is available in Angular versions before 20.
- Performing asynchronous work or side effects inside the computation callback.

## Key takeaways

- `linkedSignal()` creates a writable signal whose value is recalculated when its tracked dependencies change.
- A manual update can override the current value until the linked computation runs again.
- The simple form recalculates from a function; the source/computation form can inspect the new source and previous state.
- Use stable identifiers to preserve a valid selection across refreshed object lists.
- Choose `computed()` for read-only derivation and `linkedSignal()` for dependent state that must remain writable.
- Avoid effects for copying signal state from one signal to another.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="038-computed-signals-and-derived-values.md">038 - 38. Computed signals and derived values</a></td><td><a href="../qnas/039-linkedsignal-and-dependent-writable-state.md">Q&amp;A</a></td><td><a href="../examples/039-linkedsignal-and-dependent-writable-state.md">Example</a></td><td><a href="040-effects-cleanup-and-when-not-to-use-effects.md">040 - 40. Effects, cleanup, and when not to use effects</a></td>
</tr>
  </tbody>
</table>
