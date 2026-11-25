# 38. Q&A: Computed signals and derived values

## Computed signal fundamentals

### 1. What is a computed signal?

It is a read-only signal whose value is derived from other signals. Create one with `computed(() => ...)`, read it by calling it, and let Angular track the source signals read by the calculation.

### 2. How do I create one?

Import `computed` from `@angular/core` and return a value from its callback, for example `const doubled = computed(() => count() * 2)`. The callback should synchronously derive a value from signal inputs.

### 3. How do I read a computed signal?

Call it like any other signal: `doubled()`. In a template, use `{{ doubled() }}`. Reading a computed in a template also makes that view a reactive consumer of the computed value.

### 4. Does a computed signal have to depend on another signal?

It is intended to derive a value from signals. If it reads no signals, its result has no reactive signal dependency and will not update merely because a plain variable changed. For a constant, use a normal constant instead.

### 5. Is a computed signal writable?

No. It returns a read-only `Signal<T>`, so it has no `.set()` or `.update()` method. Change its source signal and let the derivation produce a new result.

### 6. Why is a computed signal read-only?

Its value is determined by its sources. Allowing an unrelated manual write would create a second value that could disagree with the derivation. If the dependent value must also be manually writable, choose a different state design or consider `linkedSignal()`.

### 7. Does a computed value update automatically when a source signal changes?

Angular invalidates the cached value when a tracked dependency changes. The next read calculates the value from current sources. Consumers that read it reactively can then refresh based on the change.

## Laziness, caching, and dependencies

### 8. What does “lazy” mean for `computed()`?

Angular does not need to execute the derivation until the computed signal is first read. Creating the computed signal alone does not require calculating its result immediately.

### 9. What does “memoized” mean?

After a computed value has been calculated, Angular caches it. Additional reads can reuse the cached value while its dependencies remain unchanged instead of rerunning the calculation each time.

### 10. Does the computed callback run immediately when a dependency changes?

The cached result becomes invalid when a dependency changes. Angular can wait until the computed is read again before recalculating it; computed values are lazy.

### 11. How does Angular know which signals a computed depends on?

It tracks signals that the callback actually reads during its latest execution. Calling `price()` inside the callback adds `price` as a dependency; merely referring to an unrelated signal outside the executed code does not.

### 12. Are dependencies conditional?

Yes. If a computed reads `user()` only when `showUser()` is true, then `user` is tracked only during executions that take that branch. When the branch changes, dependencies can be added or removed.

### 13. Are reads after an asynchronous boundary tracked?

No. Reactive tracking applies while the computed callback executes synchronously. A computed callback should not return a Promise or perform asynchronous work; use a suitable resource, Observable, Promise, or service workflow for async data.

### 14. Will a change to a plain class property invalidate a computed signal?

No. Angular only tracks signal reads. If the computed uses `this.taxRate` as a regular property, changing that property does not notify Angular. Make it signal state if it should be a reactive dependency.

## Designing derived state

### 15. What kinds of values should be computed?

Synchronous values determined by current signals, such as totals, filtered or sorted lists, labels, enabled-state flags, and validation messages. Keep the original source values separately and derive these results from them.

### 16. Why not store the same derived value in a writable signal?

It duplicates state. Every place that changes a source would also have to update the duplicate correctly. A computed value follows its sources automatically and avoids synchronization bugs.

### 17. Can I derive a value from more than one signal?

Yes. Read each source inside the callback, for example `computed(() => price() * quantity() * (1 + taxRate()))`. All signals read by that execution become dependencies.

### 18. Can one computed signal use another computed signal?

Yes. A computed is itself a read-only signal. Calling it inside another computed creates a dependency chain, such as `subtotal()` being read by `total`.

### 19. Can a computed return an array or object?

Yes. It can return any value. For filtered collections, return a new array from methods such as `filter()` or `map()` rather than mutating the source array.

### 20. Can I update a computed signal with `.set()` after a user action?

No. Update the writable source signal that represents the user's action. The computed signal will derive the new result. If the value must be user-editable independently, it should not be modeled as an ordinary computed value.

### 21. Should I use a template method or `computed()` for a derived display value?

Use a computed when the result depends on signals and can be reused as reactive state. A method can be called whenever Angular checks a template that uses it; a computed caches its result until its dependencies invalidate it. Keep either calculation pure and reasonably small.

### 22. Can I use a computed signal for a current time or random number?

Not by itself. `Date.now()` and `Math.random()` are not signals, so Angular has no dependency to notify it when they change. Put changing inputs into signals and update them from an appropriate timer or action if the UI must react.

## Purity and side effects

### 23. Can a computed callback write to another signal?

No. A computed derivation should read dependencies and return a value. Signal writes during computed evaluation are disallowed because they introduce side effects into a derivation.

### 24. Can I use an effect to copy a computed value into another signal?

Usually do not. If the second value is purely derived from other signals, model it as a computed signal. Angular recommends computed values for derivation and effects for appropriate synchronization with non-signal APIs.

### 25. Can a computed signal make an HTTP request or subscribe to an Observable?

No. A computed is synchronous and may be lazy or cached, so it is not a reliable place to start work. Use a resource, Observable, Promise, or service workflow for async data, and use effects only for appropriate side effects.

### 26. Can a computed callback update the DOM directly?

No. Keep DOM synchronization out of derived calculations. Use Angular template and host bindings where possible, or an appropriate effect/after-render API when integrating an imperative external library.

### 27. Can I log from a computed callback while debugging?

You can temporarily log to understand when a derivation runs, but do not rely on logging there for application behavior. Lazy evaluation and memoization mean the callback may not run at every source update.

## Troubleshooting

### 28. Why do I get a “set does not exist” TypeScript error?

That is expected for a computed signal. Its type is read-only. Write to one of its writable source signals instead.

### 29. Why did the computed not change after I changed a property?

The changed value may be a plain property rather than a signal, or the computed may not read that signal in its current conditional branch. Check that the relevant signal is actually called during the current computation.

### 30. Why did the computed callback run fewer times than I expected?

Computed values are lazy and memoized. It runs when needed after a dependency changes, not necessarily immediately on every source write. Do not use its execution count to drive application logic.

### 31. Why does changing a signal not affect an array returned by my computed?

Check that the computed reads the signal that owns the array and that the array was updated with a new reference. Mutating the same array in place may not notify consumers using the default equality behavior.

### 32. How do computed signals relate to `OnPush` templates?

When an `OnPush` component template reads a computed signal, Angular tracks that dependency. When its underlying tracked sources change, Angular can mark the component for checking and refresh the rendered value.

### 33. Where is the complete working example for computed signals?

See [Example 38](../examples/038-computed-signals-and-derived-values.md). The concepts and design choices are explained in [Lesson 38](../lessons/038-computed-signals-and-derived-values.md).

## Further reading

- [Angular Signals guide](https://angular.dev/guide/signals)
- [computed API](https://angular.dev/api/core/computed)
- [Signal writes in a disallowed context](https://angular.dev/errors/NG0600)
- [Effects guide](https://angular.dev/guide/signals/effect)
- [Linked signals](https://angular.dev/guide/signals/linked-signal)