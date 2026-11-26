# 42. RxJS interop with `toSignal()` and `toObservable()` - Q&A

## `toSignal()` and Observable sources

### 1. What does `toSignal()` do?

It subscribes to an Observable and exposes its latest emitted value as a read-only Angular Signal. Read the returned signal with function-call syntax, such as `latestValue()`.

### 2. Does `toSignal()` subscribe immediately or on the first read?

It subscribes immediately when `toSignal()` is created. This can start work such as an HTTP request or a hot stream subscription before the Signal is ever read.

### 3. Why should I not call `toSignal()` repeatedly for the same Observable?

Each call creates another subscription. That can duplicate side effects, requests, timers, and work. Create one bridge in a stable location and reuse the returned Signal.

### 4. What value does a signal from an asynchronous Observable have before the first emission?

Without an explicit initial value, the signal can be `undefined` until the Observable emits. Its type reflects that. With `{ initialValue: value }`, it returns that value until the first emission.

### 5. When should I provide `initialValue`?

When the stream might not emit immediately and the UI needs a deliberate value before the first emission. Choose a value whose meaning is clear, or represent the pre-emission state with `undefined` or a richer loading-state model.

### 6. What does `requireSync: true` do?

It asserts that the Observable emits synchronously during subscription. If it does, the returned Signal type does not include `undefined` and no initial value is required. If the source does not emit synchronously, Angular throws at runtime.

### 7. Which sources are suitable for `requireSync: true`?

Sources with a real synchronous first-emission guarantee, such as an RxJS `BehaviorSubject`. Do not use it for HTTP requests, timers, or pipelines that may defer their first emission.

### 8. Does every operator preserve a synchronous emission guarantee?

No. Operators can change timing. For example, a pipeline that schedules or delays notifications may no longer emit synchronously. Use `requireSync` only when the entire source pipeline guarantees a synchronous first value.

### 9. Where must I create `toSignal()`?

By default, it needs an injection context, commonly a component or service constructor or field initializer. If creating it later, pass an `Injector` captured from an injection context.

### 10. How is the subscription cleaned up?

By default, Angular ties the subscription to a `DestroyRef` from the creation injection context. It unsubscribes when that context is destroyed, such as when the owning component is destroyed.

### 11. What does `{ manualCleanup: true }` mean for `toSignal()`?

It disables automatic `DestroyRef` teardown. The subscription then persists until the Observable completes. It is most appropriate for streams that complete naturally; it is a poor fit for a never-ending stream because `toSignal()` does not return the subscription for manual disposal.

### 12. What happens if the source Observable errors?

The error is thrown when the resulting Signal is read. If the error should be handled in the UI, handle or convert it to a value in the Observable pipeline before calling `toSignal()`.

### 13. What happens when the source Observable completes?

The Signal continues to return the last value emitted before completion. Completion does not reset the Signal.

### 14. Can I provide a custom equality function to `toSignal()`?

Yes. Its `equal` option compares successive Observable emissions. When two values are considered equal, the resulting Signal does not update and downstream computations or effects need not rerun. Ensure the comparator includes every field that consumers care about.

### 15. Is `toSignal()` the same as using `AsyncPipe`?

Both can expose Observable values to a template and manage subscriptions with an owner lifetime. `toSignal()` creates a reusable Signal that can also be read in component code and used in computed derivations. Use whichever fits the component; avoid creating both bridges for the same data without a reason.

## `toObservable()` and signal sources

### 16. What does `toObservable()` do?

It exposes a Signal's current state as an RxJS Observable, so RxJS operators and Observable-based APIs can respond to that state.

### 17. Does `toObservable()` produce an event for every call to `.set()`?

Not necessarily. It publishes signal state through an effect. If the signal is set multiple times before the effect runs, updates can be coalesced and only the final stabilized value is emitted. Use an Observable or `Subject` when every event occurrence matters.

### 18. Is the first Observable value synchronous?

The latest value may be delivered synchronously to a subscriber if it is available. Subsequent updates are asynchronous because the signal value is propagated through an effect. Code should not rely on every notification being synchronous.

### 19. Why might a subscriber see only the final value after several synchronous writes?

Signals represent current state, and `toObservable()` publishes after the effect observes that state. Multiple writes before stabilization can collapse into the final value. This is expected for state bridging, but unsuitable for an event log that must preserve every intermediate occurrence.

### 20. Does `toObservable()` need an injection context?

Yes, by default. It creates an effect to monitor the Signal. Create it in an injection context or provide an `Injector` in its options.

### 21. Can I use RxJS operators after `toObservable()`?

Yes. For example, `debounceTime()` can wait for typing to settle, `distinctUntilChanged()` can skip repeated values, and `switchMap()` can start work for the newest query while unsubscribing from the previous inner stream.

### 22. Does `toObservable()` turn a Signal into an event stream?

It turns state into an Observable view, not an event log. It publishes the latest state and can coalesce updates. If the domain is events such as clicks or audit events, model the events directly with RxJS.

## Choosing and combining the bridges

### 23. When should I use `toSignal()`?

When an Observable is the source of truth and Angular component or template code needs a synchronous read of its latest value.

### 24. When should I use `toObservable()`?

When a Signal is the source of truth and an Observable-based API or RxJS pipeline should react to its current state.

### 25. Is Signal → Observable → Signal a good default pattern?

No. Each bridge adds scheduling, subscription, and lifetime considerations. Use both only when there is a real integration boundary that requires the intermediate Observable pipeline.

### 26. How do I debounce a Signal-backed search input?

Convert the query Signal once with `toObservable()`, then apply RxJS operators such as `debounceTime()`, `distinctUntilChanged()`, and `switchMap()`. Convert the final result stream with one `toSignal()` if the template needs synchronous reads.

### 27. What is the purpose of `switchMap()` in a search pipeline?

It switches to the Observable returned for the latest query and unsubscribes from the previous inner Observable. This is often appropriate for replaceable requests where an older query's response is no longer needed.

### 28. When should I keep the data as an Observable instead of calling `toSignal()`?

Keep it as an Observable when the consumer is already Observable-based, when you need stream operators throughout the consumer, or when subscription timing/event semantics matter more than synchronous state reads. Convert at the integration boundary instead of converting everything.

### 29. When should I use a resource API instead of the bridge pipeline?

If the main task is asynchronous data loading from reactive parameters and the application benefits from built-in loading, error, and value states, Angular's `resource` or `rxResource` may be a better fit. Check their current APIs and compatibility for the Angular version in the project.

### 30. What checklist helps avoid common interop bugs?

- Is the Observable or Signal the source of truth?
- Will the Observable emit synchronously, or does it need an `initialValue`?
- Is `requireSync` guaranteed by the entire source pipeline?
- Is this bridge created once in an appropriate injection context?
- Who owns teardown, and should the stream complete naturally?
- Are errors handled before the Signal is read?
- Do I need every event, or only the latest state?
- Does the conversion preserve the timing semantics the consumer expects?

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="041-signal-equality-dynamic-dependencies-and-untracked-reads.md">041 - 41. Signal equality, dynamic dependencies, and untracked reads</a></td><td><a href="../lessons/042-rxjs-interop-with-tosignal-and-toobservable.md">Lesson</a></td><td><a href="../examples/042-rxjs-interop-with-tosignal-and-toobservable.md">Example</a></td><td><a href="043-angular-rendering-and-change-detection.md">043 - 43. Angular rendering and change detection</a></td>
</tr>
  </tbody>
</table>
