# 39. Q&A: `linkedSignal` and dependent writable state

## `linkedSignal` basics

### 1. What is `linkedSignal()`?

It creates a writable signal whose initial and refreshed values come from a reactive computation. Unlike `computed()`, the returned signal can also be manually updated with `set()` or `update()`.

### 2. Which Angular versions include stable `linkedSignal()`?

The API is stable starting in Angular 20. If a project uses an earlier Angular version, the API may be unavailable or have a different status; check the documentation for that version before using it.

### 3. How is `linkedSignal()` different from `signal()`?

`signal(initialValue)` holds independent writable state. `linkedSignal(computation)` also holds writable state, but it recalculates its value when the signals read by its computation change.

### 4. How is `linkedSignal()` different from `computed()`?

A computed signal is read-only and always represents its derivation. A linked signal is writable and can be manually changed between source updates; the computation runs again when its dependencies change.

### 5. What does the simple computation form do?

`linkedSignal(() => options()[0] ?? null)` initializes the linked value from the options signal. If that source changes, the computation runs again and the linked value is reset to the returned option or `null`.

### 6. Is a linked signal writable?

Yes. It returns a `WritableSignal`, so you can read it, call `.set()` with a value, or call `.update()` with a synchronous updater.

### 7. Does a manual value stay forever after calling `.set()`?

No. It stays current while the tracked sources remain unchanged. When a dependency changes, the computation runs again and determines the next linked value. That may preserve or replace the manual choice, depending on the computation.

### 8. What is a common use case for a linked signal?

A selected item that should normally come from a changing list but can be changed by the user. When the list refreshes, the computation can keep the selection if it remains valid or choose a fallback if it was removed.

### 9. When should I avoid `linkedSignal()`?

Avoid it for values that are purely derived and should never be written; use `computed()`. Also avoid it for independent state with no relationship to another signal; use `signal()`.

## Sources and recomputation

### 10. What does `source` mean in the object form?

It identifies the signal-like source whose current value is passed as the first argument to `computation`. For example, `source: this.options` passes the current options array to the callback.

### 11. Can the computation read signals other than `source`?

Yes. Any signals read inside the computation are reactive dependencies too. A change to one of those signals can recalculate the linked value.

### 12. Is the `computation` callback synchronous?

Yes. It must synchronously return the next linked value. Do not use it for asynchronous requests, timers, subscriptions, or other side effects.

### 13. Does a manual call to `.set()` rerun the computation immediately?

It writes the linked value directly according to the linked signal's write behavior. The computation is used to initialize and respond to changed dependencies. If you customize the setter option, you can map writes back to a source instead.

### 14. Can the source be a computed signal or an input signal?

Yes. The source can be another signal, including a computed or component input signal. The linked signal recalculates when the source changes.

### 15. Should I use an effect to copy a source into a writable destination signal?

Usually not. Copying signal state with an effect creates an indirect synchronization path. Prefer `computed()` for read-only derivation or `linkedSignal()` for dependent state that must also be writable.

## Previous state and preserving choices

### 16. Why does the source/computation form accept `previous`?

It lets the calculation use both the new source and the linked signal's prior state. This is useful for preserving a user's selection if the new source still contains the matching item.

### 17. What is in the `previous` argument?

It has `previous.source`, the old source value, and `previous.value`, the old linked-signal value. The whole `previous` argument is optional because it is absent during initial computation.

### 18. Why do examples explicitly provide two generic types?

The generic parameters are `<Source, Destination>`. The two-type form that inspects `previous` often needs both types to be stated so TypeScript can check the source and returned linked value correctly.

### 19. Why should I compare option IDs instead of object references?

A refreshed list usually contains new object instances. The same logical choice can have a different object reference, so compare a stable identifier and return the matching object from the new list.

### 20. What should the computation return if the previously selected item disappeared?

Choose an explicit fallback. A common policy is the first item in the new list, with `null` if the list is empty. The destination type must allow every possible result, including `null` if needed.

### 21. Why should I use optional chaining on `previous`?

The initial computation has no previous state, so `previous` can be undefined. Also, the previous linked value may be `null`. Access values such as an ID safely with `previous?.value?.id`.

### 22. Can the computation preserve the previous destination even when it is not valid for the new source?

It can, but usually it should not. The point of the relationship is to keep the destination consistent with its source. Validate the previous value against the new source and choose a defined fallback if it is no longer valid.

## Writing and custom setters

### 23. Can I call `.update()` on a linked signal?

Yes. It is writable. Use `update()` when the next linked value depends on its current value, and `set()` when the complete next value is already known.

### 24. Can a linked signal update a nested property in a parent source object?

Yes. It can be configured so writes immutably update the parent source and the linked value is then recalculated from that source. The linked-signal guide includes this pattern; keep one clear source of truth.

### 25. What does the optional `set` configuration do?

It customizes how writes to the linked signal are handled. The callback receives the requested value and a `rawSet` function; it can translate the write into an update to the source instead of only changing the linked value.

### 26. What is `rawSet` for?

It directly writes the linked signal's internal value as part of a custom setter. Use it only when the desired write should not rely on recomputation from the source. Most linked-signal use cases do not need this advanced option.

### 27. Is a custom setter needed for preserving a list selection?

No. Preserving or replacing a selection when the source list changes belongs in the `computation` callback. A custom setter is for changing what happens when code manually calls `.set()` or `.update()` on the linked signal.

### 28. Can I expose a linked signal as readonly?

Yes. Use `asReadonly()` for consumers that may read the linked value but should not change it directly. Expose operation methods on the owner to control allowed writes.

### 29. Does `asReadonly()` freeze objects stored in a linked signal?

No. It hides the signal's write methods but does not deep-freeze object values. Keep source and destination updates immutable.

## Troubleshooting and decisions

### 30. Why does the linked value change after the user selected something else?

A tracked source or another signal read by the computation changed, so the computation returned a new value. Decide whether your policy should reset, preserve by key, or clear the selection when that happens.

### 31. Why did my linked value not update when I expected?

Check that the relevant value is a signal and is read inside the linked computation. Plain properties are not reactive dependencies. Also check that the actual signal value or reference changed.

### 32. Can a linked signal keep a selection when an options list refreshes but the options have new objects?

Yes. Use the source/computation form, read the previous ID, find that ID in the new list, and return the new matching object. If no match exists, return the defined fallback.

### 33. Is `linkedSignal()` an alternative to two-way binding or `model()`?

Not automatically. `linkedSignal()` models dependent writable state inside a component or service. For a child component API that intentionally accepts and emits a parent-updated value, an input/output or model-input design may be more appropriate.

### 34. Can a linked signal be used for asynchronous data?

It can depend on signal state that represents async results, but its computation itself is synchronous. Use a resource, Observable, Promise, or service to load data, then derive or link local state from the resulting signal values.

### 35. Where is the complete working example for this chapter?

See [Example 39](../examples/039-linkedsignal-and-dependent-writable-state.md). The concepts and design trade-offs are explained in [Lesson 39](../lessons/039-linkedsignal-and-dependent-writable-state.md).

## Further reading

- [Angular linkedSignal guide](https://angular.dev/guide/signals/linked-signal)
- [linkedSignal API](https://angular.dev/api/core/linkedSignal)
- [Effects guide](https://angular.dev/guide/signals/effect)
- [Signal writes in a disallowed context](https://angular.dev/errors/NG0600)