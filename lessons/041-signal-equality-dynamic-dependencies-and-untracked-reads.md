# 41. Signal equality, dynamic dependencies, and untracked reads

Signals notify reactive consumers when their value changes. Equality determines whether a proposed value counts as a change. Dependency tracking determines which signal reads can cause a `computed()` or `effect()` to run again. `untracked()` lets code read a signal without adding that dependency to the current reactive computation.

These rules help Angular avoid unnecessary work, but they also mean a comparator or an untracked read can hide a change your UI needs. Use both deliberately.

## Learning goals

After this lesson, you should be able to:

- Explain the default `Object.is()` equality used by signals.
- Provide a custom equality function for writable and computed signals.
- Avoid mutations and comparators that hide meaningful changes.
- Describe how a computed signal's dependency list changes between runs.
- Use `untracked()` for incidental reads without creating reactive dependencies.
- Diagnose why a consumer did or did not rerun after a signal update.

## Default signal equality

Signals use `Object.is()` to compare a proposed value with their current value unless you provide another comparison function. If the values are equal, Angular treats the update as unchanged and does not notify downstream consumers.

~~~ts
import { signal } from '@angular/core';

const quantity = signal(2);
quantity.set(2); // Same primitive value: no change notification.
quantity.set(3); // Different primitive value: consumers are notified.
~~~

For objects and arrays, `Object.is()` checks identity, not their contents:

~~~ts
const settings = signal({ theme: 'light', fontSize: 16 });

settings.set({ theme: 'light', fontSize: 16 });
// A new object identity, so the signal treats this as a change.
~~~

Conversely, mutating an object and setting that same reference does not produce a change:

~~~ts
const settings = signal({ theme: 'light', fontSize: 16 });
const current = settings();
current.theme = 'dark';
settings.set(current); // Same reference: Object.is() returns true.
~~~

Signal values should normally be updated immutably. Create a new object or array so identity changes and Angular can notify consumers:

~~~ts
settings.update((current) => ({ ...current, theme: 'dark' }));
~~~

A read-only signal is not deeply immutable. `asReadonly()` prevents callers from using `.set()` and `.update()` through that reference, but it does not prevent mutation of an object returned by the signal. Keep update ownership clear and treat stored objects as immutable.

## Custom equality functions

Pass an `equal` function in the options when the default comparison is too strict for the value's intended meaning. It receives the previous and proposed values and returns `true` when they should count as equal.

~~~ts
interface SearchOptions {
  query: string;
  category: string;
}

const options = signal<SearchOptions>(
  { query: '', category: 'all' },
  {
    equal: (previous, next) =>
      previous.query === next.query &&
      previous.category === next.category,
  },
);

options.set({ query: '', category: 'all' });
// The fields are equal, so downstream consumers need not be notified.
~~~

Custom equality can also be used with `computed()`:

~~~ts
const normalizedQuery = computed(
  () => rawQuery().trim().toLowerCase(),
  { equal: (previous, next) => previous === next },
);
~~~

In this example, changing `rawQuery` from `'Angular'` to `' angular '` invalidates the computation, but the normalized result is still `'angular'`. The equality function can prevent downstream consumers from being notified about an equivalent result.

An equality function defines which changes Angular is allowed to ignore. It should be pure, deterministic, and cheap. Do not compare only an ID when consumers also need changes to a name, status, or other field. If the comparator says two values are equal, Angular can suppress the update, leaving dependent UI based on the prior value. Prefer the default comparison unless you have a clear semantic reason and have considered every consumer.

## Dynamic dependencies

Angular tracks signal reads made while evaluating a reactive consumer, such as a `computed()` derivation or an `effect()` callback. The dependency set reflects the signals read during the most recent evaluation. A branch that is not taken does not add its signals as dependencies.

~~~ts
import { computed, signal } from '@angular/core';

const showCount = signal(false);
const count = signal(0);

const label = computed(() => {
  if (showCount()) {
    return `Count: ${count()}`;
  }

  return 'Count is hidden';
});
~~~

When `showCount()` is false, evaluating `label()` reads only `showCount`. Updating `count` does not invalidate `label` because it is not currently a dependency. Once `showCount` becomes true and `label()` is evaluated again, the derivation reads `count` and begins tracking it. If `showCount` later becomes false and `label()` is reevaluated, `count` is removed from the dependency set.

The same rule applies to effects:

~~~ts
const usePrimary = signal(true);
const primaryColor = signal('blue');
const backupColor = signal('gray');

effect(() => {
  const color = usePrimary() ? primaryColor() : backupColor();
  console.log('Selected color:', color);
});
~~~

The effect initially tracks `usePrimary` and `primaryColor`. After it runs with `usePrimary()` false, it tracks `usePrimary` and `backupColor` instead. Changes to the inactive color do not rerun the effect.

`computed()` is lazy and memoized: its derivation runs when the computed is read and needs a current value. Once invalidated by a dependency, it recalculates the next time it is read. This is why a dependency added by a branch becomes active when the computation is reevaluated, rather than merely because some other signal changed elsewhere.

## Read without tracking with `untracked()`

`untracked()` executes a callback in a non-tracking context and returns that callback's result. It reads the signal's current value normally; it simply does not register the read as a dependency of the surrounding reactive computation.

~~~ts
import { effect, signal, untracked } from '@angular/core';

const currentUser = signal('Ada');
const diagnosticCounter = signal(0);

effect(() => {
  const user = currentUser();
  const counter = untracked(diagnosticCounter);

  console.log(`User changed: ${user}; diagnostic count is ${counter}`);
});
~~~

This effect reruns when `currentUser` changes, but not when only `diagnosticCounter` changes. If it runs for another reason, it reads the latest counter value at that time.

`untracked()` also accepts a callback, which is useful when calling an external function that might read signals internally:

~~~ts
effect(() => {
  const user = currentUser();

  untracked(() => {
    analytics.logUser(user);
  });
});
~~~

Any signal reads made inside `analytics.logUser()` will not become dependencies of this effect. This is useful when those reads are incidental. If a value should cause the effect to rerun, read it normally outside `untracked()`.

## Equality and dependency tracking work together

A consumer reruns only when one of its tracked dependencies is considered changed. There are two separate questions:

1. **Did the source signal accept the proposed value as a change?** Its equality function decides.
2. **Is this source signal a current dependency of the consumer?** The most recent reactive evaluation decides.

For example, updating an inactive branch's signal will not rerun a consumer because it is not a dependency. Updating an active signal with a value considered equal will not notify consumers either. When debugging a missing update, inspect both the equality comparison and the current dependency path.

## Common mistakes

- **Mutating an object and setting the same reference:** default identity equality sees no change. Replace the object immutably.
- **Using deep equality everywhere:** deep comparisons can be expensive and may hide updates consumers need. Add one only when measurements and semantics justify it.
- **Ignoring fields in a comparator:** a field changes but Angular treats the entire value as equal, so consumers may not receive the new value.
- **Assuming every signal read is always a dependency:** conditional branches make dependencies dynamic.
- **Expecting an inactive branch to rerun a computation:** a signal that was not read in the latest evaluation cannot invalidate that consumer.
- **Using `untracked()` for a real dependency:** the consumer will not rerun when only that value changes.
- **Expecting `untracked()` to cache a value:** it reads the current value at that moment; it only suppresses dependency registration.

## Key takeaways

- Default signal equality uses `Object.is()`: primitive values compare by value semantics, objects and arrays by identity.
- Use immutable updates for object and array state.
- An equality function defines which proposed values count as changes; keep it correct for every consumer.
- Computed signals and effects track only reads made during their latest synchronous evaluation.
- Conditional branches can add and remove dependencies as the consumer reruns.
- `untracked()` reads current values without adding dependencies to the surrounding reactive computation.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="040-effects-cleanup-and-when-not-to-use-effects.md">040 - 40. Effects, cleanup, and when not to use effects</a></td><td><a href="../qnas/041-signal-equality-dynamic-dependencies-and-untracked-reads.md">Q&amp;A</a></td><td><a href="../examples/041-signal-equality-dynamic-dependencies-and-untracked-reads.md">Example</a></td><td><a href="042-rxjs-interop-with-tosignal-and-toobservable.md">042 - 42. RxJS interop with `toSignal()` and `toObservable()`</a></td>
</tr>
  </tbody>
</table>
