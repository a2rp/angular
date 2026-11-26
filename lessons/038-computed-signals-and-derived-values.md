# 38. Computed signals and derived values

A computed signal represents a value that can be calculated from other signals. Create one with `computed()`, read it by calling it, and let Angular keep the result in sync with the signals it actually uses. Computed signals are read-only: update their source signals and let the derived value recalculate.

Computed values are useful for totals, filtered collections, labels, validation state, and other synchronous calculations. They help keep one source of truth instead of storing both original state and a second copy that must be manually synchronized.

## Learning goals

After this lesson, you should be able to:

- Create and read a computed signal with `computed()`.
- Explain why computed signals are read-only, lazy, and memoized.
- Build a derived value from one or more writable signals.
- Understand how Angular tracks conditional, dynamic dependencies.
- Avoid effects or duplicate writable state for synchronous derivations.
- Know when a value should be computed, stored as writable state, or loaded asynchronously.

## Create a computed signal

Import `computed` from `@angular/core`. Provide a function that reads source signals and returns the derived value:

~~~ts
import { computed, signal } from '@angular/core';

const price = signal(12);
const quantity = signal(3);

const subtotal = computed(() => price() * quantity());

console.log(subtotal()); // 36
~~~

The function calls `price()` and `quantity()`. Angular tracks those signal reads and knows that `subtotal` depends on them. When either source changes, the computed value becomes invalid and will be recalculated the next time it is read.

A computed signal is also read by calling it:

~~~ts
quantity.set(4);
console.log(subtotal()); // 48
~~~

The computed value updates because its dependencies changed. Do not assign to it or try to call `.set()` or `.update()` on it.

## Keep source state and derived state separate

Suppose an order has a price and quantity. Store those inputs as writable signals and derive the total:

~~~ts
readonly price = signal(12);
readonly quantity = signal(3);
readonly subtotal = computed(() => this.price() * this.quantity());
~~~

If tax is also a source value, derive the total including tax:

~~~ts
readonly taxRate = signal(0.08);
readonly total = computed(
  () => this.subtotal() * (1 + this.taxRate()),
);
~~~

The dependency graph is `price` and `quantity` → `subtotal` → `total`, plus `taxRate` → `total`. If `quantity` changes, Angular invalidates the values that depend on it. The next read of `subtotal` or `total` obtains results based on the latest source values.

Do not keep a writable `subtotal` alongside `price` and `quantity`, then remember to update it in every method. That duplicates state and creates opportunities for the values to disagree. Derive it once from the source signals.

## Computed signals are read-only

`computed()` returns a `Signal<T>`, not a `WritableSignal<T>`:

~~~ts
const count = signal(1);
const doubled = computed(() => count() * 2);

// Read the derived value.
console.log(doubled());

// These do not exist on a computed signal:
// doubled.set(4);
// doubled.update((value) => value + 1);
~~~

Change the source signal instead:

~~~ts
count.set(2);
console.log(doubled()); // 4
~~~

Read-only here is an API property. The derived value can still change when its dependencies change; the consumer simply cannot write a separate value into it.

## Computed values are lazy and memoized

A computed function is lazy: Angular does not need to calculate its value until it is first read. Once evaluated, Angular caches the result. Reading it again while its dependencies are unchanged returns the cached result without rerunning the derivation.

If a dependency changes, Angular marks the cached value invalid. It recalculates the value when a consumer next reads it. It does not need to eagerly run every derivation at the moment a source is written.

~~~ts
const amount = signal(5);
const doubled = computed(() => {
  console.log('Calculating doubled amount');
  return amount() * 2;
});

// The calculation first runs here.
console.log(doubled());

// This read can use the cached result.
console.log(doubled());

amount.set(6);

// The calculation runs again when the invalidated value is read.
console.log(doubled());
~~~

Do not use a computed derivation for work that must happen as an unconditional side effect. Its laziness and memoization mean there is no promise that it will run just because a source changed. Use an appropriate event or effect for imperative side effects; effects are covered in a later lesson.

## Dependencies are tracked dynamically

Angular tracks the signals read during the latest execution of the computed function. When a branch changes, the set of dependencies can change too:

~~~ts
const showDetails = signal(false);
const accountName = signal('Asha');

const summary = computed(() => {
  if (showDetails()) {
    return `Account: ${accountName()}`;
  }

  return 'Details are hidden';
});
~~~

When `showDetails()` is false, this computation does not read `accountName()`, so `accountName` is not a dependency of the current result. If details are enabled, the computed function runs again, reads `accountName()`, and starts tracking it. If details are hidden again, that dependency can be removed.

This makes conditional derived state efficient and accurate. Only the signals read in the latest execution are dependencies.

## Use computed values in components and templates

A component can keep source state writable and expose derived values for its template:

~~~ts
import { Component, computed, signal } from '@angular/core';

@Component({
  selector: 'app-order-summary',
  standalone: true,
  template: `
    <p>Unit price: {{ price() }}</p>
    <p>Quantity: {{ quantity() }}</p>
    <p>Subtotal: {{ subtotal() }}</p>
    <p>Total: {{ total() }}</p>
    <button type="button" (click)="addItem()">Add one item</button>
  `,
})
export class OrderSummaryComponent {
  readonly price = signal(12);
  readonly quantity = signal(1);
  readonly taxRate = signal(0.08);

  readonly subtotal = computed(() => this.price() * this.quantity());
  readonly total = computed(
    () => this.subtotal() * (1 + this.taxRate()),
  );

  addItem(): void {
    this.quantity.update((current) => current + 1);
  }
}
~~~

The template calls each signal. `subtotal` and `total` are read-only derived values, while the button updates the writable `quantity` signal. Angular tracks signal reads in a component template, including computed-signal reads.

## Derive a filtered collection

Computed signals can return arrays and other objects. For example, derive the visible tasks from a writable list and a writable search query:

~~~ts
interface Task {
  id: number;
  title: string;
  done: boolean;
}

readonly tasks = signal<Task[]>([
  { id: 1, title: 'Read signals', done: true },
  { id: 2, title: 'Build a small demo', done: false },
]);
readonly query = signal('');

readonly visibleTasks = computed(() => {
  const normalizedQuery = this.query().trim().toLowerCase();

  return this.tasks().filter((task) =>
    task.title.toLowerCase().includes(normalizedQuery),
  );
});
~~~

Update the writable source signals to affect the result:

~~~ts
this.query.set('demo');

this.tasks.update((current) => [
  ...current,
  { id: 3, title: 'Review computed signals', done: false },
]);
~~~

The derivation does not mutate the task list. It returns a filtered array for readers of `visibleTasks()`.

## Keep derivations pure and synchronous

A computed function should calculate and return a value from the signals it reads. Do not write to another signal from inside it:

~~~ts
// Avoid: a computed derivation must not update other state.
// readonly doubled = computed(() => {
//   const result = this.count() * 2;
//   this.total.set(result);
//   return result;
// });
~~~

Instead, make the derived value itself the computed signal:

~~~ts
readonly doubled = computed(() => this.count() * 2);
~~~

Avoid requests, subscriptions, timers, logging for application behavior, DOM writes, and other side effects inside a computed function. Keep it synchronous, deterministic, and free of writes. Computed signals are for derived state, not for starting work.

## Choose the right state tool

| Need | Use | Why |
| --- | --- | --- |
| State that changes because of user actions or application operations | `signal()` | It has `set()` and `update()` |
| A synchronous value determined entirely by other signals | `computed()` | It stays in sync without duplicate writes |
| A dependent value that recalculates from another signal but can also be changed manually | `linkedSignal()` | It combines a source relationship with writable state; covered next |
| A response that arrives asynchronously | Resource, Observable, Promise, or service workflow | Computed functions are synchronous and do not fetch data |
| A side effect that synchronizes with a non-signal API | An event handler or, where appropriate, `effect()` | Side effects should not be hidden in a derivation |

If you need a value that is initially derived but can be changed independently while staying linked to a source, `computed()` is intentionally not writable. Use a writable state design or the `linkedSignal()` API introduced in the next lesson.

## Common mistakes

- Forgetting to call a computed signal, such as displaying `total` instead of `total()`.
- Trying to call `.set()` or `.update()` on a computed signal.
- Copying a computed value into a second writable signal with an effect.
- Reading a plain variable inside `computed()` and expecting changes to that variable to invalidate the result.
- Performing a request or other side effect inside the computation.
- Assuming computed functions run eagerly every time a dependency changes.
- Assuming a signal referenced in one branch is always a dependency, even when that branch was not taken.
- Mutating an array or object source in place and expecting reference equality to detect the change.
- Using a computed signal for a value that must remain independently editable.

## Key takeaways

- `computed()` creates a read-only signal from a synchronous derivation.
- Read computed values by calling them, just like other signals.
- Angular lazily evaluates and memoizes computed values until a dependency changes.
- Only signal reads made during the latest execution are tracked as dependencies.
- Keep computed functions pure: derive and return values; do not write state or perform side effects.
- Derive synchronous values instead of storing duplicate writable state.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="037-writable-signals-and-updating-signal-state.md">037 - 37. Writable signals and updating signal state</a></td><td><a href="../qnas/038-computed-signals-and-derived-values.md">Q&amp;A</a></td><td><a href="../examples/038-computed-signals-and-derived-values.md">Example</a></td><td><a href="039-linkedsignal-and-dependent-writable-state.md">039 - 39. `linkedSignal` and dependent writable state</a></td>
</tr>
  </tbody>
</table>
