# 37. Q&A: Writable signals and updating signal state

## Creating and reading signals

### 1. What is a writable signal?

It is a callable value created with `signal()` that stores state and exposes methods to replace it. Read it by calling it, then use `set()` or `update()` to write a new value.

### 2. How do I create a writable signal?

Import `signal` from `@angular/core` and give it an initial value: `const count = signal(0)`. The initial value normally determines its TypeScript type.

### 3. Why do I write `count()` instead of `count`?

A signal is read by calling its getter function. `count()` returns the current value and lets Angular track the read when it happens in a template or another reactive context.

### 4. How do I make a signal that can be empty?

Include the empty state in its type, such as `signal<string | undefined>(undefined)`. Then check for that state before using the value as a string.

### 5. Should every component property be a signal?

No. Use a signal for state that changes and whose consumers need to react to that change. A constant, a value used only once, or an ordinary local variable may not need a signal.

### 6. Can I read a signal in an Angular template?

Yes. Call it in the expression, for example `{{ count() }}` or `[disabled]="count() >= limit"`. Angular tracks template reads and can refresh the view when the signal changes.

### 7. What does a template read do in an `OnPush` component?

Angular tracks the signal as a dependency of that component's view. When the signal changes, Angular marks the component for checking on the next update cycle.

## Choosing a write operation

### 8. When should I use `set()`?

Use `set(nextValue)` when you already have the complete value to store, such as resetting a counter with `count.set(0)` or assigning a response object after a request completes.

### 9. When should I use `update()`?

Use `update(current => nextValue)` when the next value is calculated from the current value. A counter increment is `count.update(value => value + 1)`.

### 10. Can `set()` and `update()` both change the same signal?

Yes. They are two ways to write a writable signal. Use the one that most clearly expresses the state transition; do not pick one based only on a stylistic rule.

### 11. How do I toggle a boolean signal?

Use `enabled.update(current => !current)`. This makes the operation explicitly depend on the current value.

### 12. Should the callback passed to `update()` be asynchronous?

No. `update()` expects a synchronous function that returns the next signal value. Do asynchronous work first, then call `set()` with the result or call `update()` with a synchronous transformation.

### 13. Should an `update()` callback make a request or write to another signal?

No. Keep the updater synchronous and focused on returning the next value. Start requests, log actions, or coordinate other state in the event handler or service method that decided to perform the update.

### 14. Can I write to a signal from a button click handler?

Yes. Event handlers and component or service methods are normal places for state changes. The restriction is against writing while Angular is evaluating a template or while a pure derivation such as `computed()` is being evaluated.

### 15. What happens if I call `set()` with the same value?

Signals use an equality check before notifying reactive consumers. By default, Angular uses `Object.is()`, so setting the same primitive value or the same object reference is treated as unchanged. Custom equality functions are an advanced option covered separately.

## Updating arrays and objects

### 16. Why should I avoid mutating the object stored in a signal?

Mutation preserves the same reference. The signal's default equality check may therefore see no change and consumers may not update as intended. Make a new object and set or return that reference instead.

### 17. How do I update one field of an object signal?

Use `update()` and copy the current object: `profile.update(current => ({ ...current, city: 'Jaipur' }))`. The spread preserves fields that were not changed.

### 18. How do I update a nested object field?

Copy each object on the path to the changed field. For example, copy the outer preferences object and its `notifications` object, then replace only `productUpdates`.

### 19. Is object spread a deep copy?

No. Spread is shallow. It copies the outer object's properties, but nested object references remain shared unless you copy those nested objects too.

### 20. How do I add an item to an array signal?

Return a new array, for example `tasks.update(current => [...current, newTask])`. Do not use `push()` on the array returned by `tasks()`.

### 21. How do I remove an item from an array signal?

Use `filter()` to return a new array: `tasks.update(current => current.filter(task => task.id !== id))`.

### 22. How do I update an item inside an array signal?

Use `map()` to create a new array and return a new object for the matching item, for example `task.id === id ? { ...task, done: true } : task`.

### 23. Can I mutate an array and then call `set()` with that same array?

Avoid it. Although `set()` is called, the reference is unchanged, so the default equality comparison can treat the signal as unchanged. Replace the array with a new one instead.

### 24. Do I always need immutable updates for signal values?

You can store any JavaScript value, but immutable updates are the straightforward way to produce a changed reference and keep state transitions visible. They are especially important for arrays and objects observed by default equality.

## Readonly signals and boundaries

### 25. What does `asReadonly()` return?

It returns a read-only signal view. Consumers can call it to read the latest value, but the view does not expose `set()` or `update()`.

### 26. Does a readonly signal stop its value from changing?

No. It reflects changes made through the original writable signal. Readonly means that this reference does not provide write methods; the value is not frozen in time.

### 27. Does `asReadonly()` deep-freeze objects and arrays?

No. It does not protect nested values from direct JavaScript mutation. Keep state immutable and avoid giving consumers mutable objects they are expected to edit directly.

### 28. Why keep a writable signal private in a service?

It protects the state boundary. Consumers can read the signal, while the service controls which operations can update it through methods such as `increment()` or `reset()`.

### 29. Can I call `.set()` on a computed signal or an input signal?

No. Computed signals and `input()` signals are read-only APIs. A local signal created with `signal()` is writable. Component input state belongs to the parent; use a suitable output or model input API when two-way communication is intended.

## Troubleshooting and design

### 30. Why does Angular report that `set` or `update` does not exist?

You may be calling a write method on a read-only signal, such as a computed signal, an input signal, or an `asReadonly()` view. Keep writes on the original `WritableSignal` or expose an operation through the owner.

### 31. Why did the UI not update after changing a nested property?

The object or array reference may not have changed. Return a new reference for the signal value and for any nested objects on the path that changed.

### 32. Should I use `update()` when the next state comes from an HTTP response?

Usually use `set(response)` because the response is already the complete next value. If it must be merged with current state, use a synchronous `update()` that returns a new merged object after the response arrives.

### 33. Is a signal the same as an Observable?

No. A signal is synchronously read as its current value and updated directly. An Observable represents a stream of emissions and supports RxJS operators. Use the abstraction that matches the state or stream you have.

### 34. What should I use for state derived from other signals?

Use a computed signal rather than manually copying the derived value into another writable signal. Computed signals are read-only derivations and are introduced in [Lesson 38](../lessons/038-computed-signals-and-derived-values.md).

### 35. Where is the complete working example for writable signals?

See [Example 37](../examples/037-writable-signals-and-updating-signal-state.md). The full concepts and code decisions are in [Lesson 37](../lessons/037-writable-signals-and-updating-signal-state.md).

## Further reading

- [Angular Signals guide](https://angular.dev/guide/signals)
- [WritableSignal API](https://angular.dev/api/core/WritableSignal)
- [Signal essentials](https://angular.dev/essentials/signals)
- [Signal writes in a disallowed context](https://angular.dev/errors/NG0600)