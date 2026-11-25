# Q&A: Flattening operators, cancellation, and concurrency

These answers explain higher-order Observables and how **switchMap()**, **mergeMap()**, **concatMap()**, and **exhaustMap()** manage inner work. See the [lesson](../lessons/059-flattening-operators-cancellation-and-concurrency.md) for the full explanation and the [example](../examples/059-flattening-operators-cancellation-and-concurrency.md) for a runnable Angular demo.

## 1. What is a higher-order Observable?

It is an Observable whose values are themselves Observables or other supported asynchronous inputs. Its type is often written as **Observable<Observable<T>>**.

For example, mapping a stream of search terms to HTTP request Observables creates a higher-order stream.

## 2. What is an inner Observable?

It is an Observable returned by a projection from an outer Observable value. In a search stream, each query can create one inner HTTP request Observable.

## 3. Why does map() not finish a request pipeline by itself?

**map()** changes each outer value into the value returned by the projection. If the projection returns an Observable, the outer stream emits that Observable as a value. **map()** does not subscribe to or flatten it.

Use a flattening operator when the desired result should emit the inner Observable's values.

## 4. What does flattening mean here?

Flattening connects inner Observables to the outer result and forwards their values according to a chosen policy. The result stream emits the inner values instead of emitting Observable objects.

## 5. What do switchMap(), mergeMap(), concatMap(), and exhaustMap() have in common?

Each maps an outer value to an inner Observable and flattens the inner output. They differ in how they treat active inner streams and new outer values.

## 6. What is switchMap()?

**switchMap()** unsubscribes from the previous inner Observable when a new outer value arrives, then subscribes to the newest inner Observable. Only the latest inner stream continues contributing values.

## 7. When is switchMap() a good choice?

Use it when a newer value makes previous work irrelevant, such as type-ahead searches, route parameter changes, or refreshing a preview for the latest selection.

## 8. Does switchMap() cancel the underlying work?

It unsubscribes from the prior inner Observable. Whether that stops the underlying operation depends on the source. Angular **HttpClient** aborts an in-progress client request when its subscription is cancelled.

## 9. Does cancelling an HTTP request guarantee that the server did nothing?

No. The request may already have reached or been processed by the server. Client cancellation is not a transaction rollback and does not undo server-side effects.

## 10. Should I use switchMap() for writes?

Use it only if discarding the previous client subscription matches the product requirements. For ordered writes, **concatMap()** may fit better; for independent writes, **mergeMap()** may fit. The backend must still define how concurrent or repeated writes behave.

## 11. What is mergeMap()?

**mergeMap()** starts an inner Observable for each outer value and allows those inner streams to overlap. It forwards their values as they arrive.

## 12. Does mergeMap() preserve source order?

No. Inner streams can complete or emit at different speeds, so output order follows inner emissions, not necessarily the order of the outer values.

## 13. What is mergeMap()'s default concurrency?

RxJS **mergeMap()** allows unlimited concurrent inner subscriptions by default. Its optional concurrency argument limits the number that are active at once.

~~~ts
const uploads$ = files$.pipe(
  mergeMap((file) => upload(file), 3),
);
~~~

Here at most three upload Observables are active; later files wait for an available slot.

## 14. Does a concurrency limit mean mergeMap() runs inner work in order?

No. A limit controls how many inner streams can be active, not the order in which they finish or emit. Use **concatMap()** when only one inner stream should run at a time in source order.

## 15. When is mergeMap() appropriate?

Use it when operations are independent and may run concurrently, such as parallel reads or uploads where result ordering does not matter. Add a concurrency limit if producing work faster than it can finish could create too many active operations.

## 16. What is concatMap()?

**concatMap()** subscribes to one inner Observable at a time. It waits for the current inner to complete before subscribing to the next, preserving the order in which outer values are handled.

## 17. Does concatMap() discard new outer values while busy?

No. It queues them and processes them sequentially after the active inner completes.

## 18. Can concatMap() build up a queue?

Yes. If outer values arrive faster than inner Observables finish, pending values accumulate. A very fast or endless source with slow operations can create long delays and memory growth.

## 19. Is concatMap() the same as mergeMap() with concurrency 1?

In behavior, **concatMap(project)** is equivalent to **mergeMap(project, 1)**. **concatMap()** communicates the sequential intent directly.

## 20. When is concatMap() appropriate?

Use it when order matters and each operation should finish before the next starts, such as a queue of document edits or sequential steps in a workflow.

## 21. What is exhaustMap()?

**exhaustMap()** starts an inner Observable for the first outer value. It ignores further outer values while that inner Observable is active. Once the inner completes, it can accept another outer value.

## 22. Does exhaustMap() queue values that arrive while busy?

No. Those values are discarded by the operator; they do not run later.

## 23. When is exhaustMap() useful?

Use it to ignore repeated triggers during an active operation, such as preventing double submission of a login or checkout form.

## 24. How is exhaustMap() different from disabling a button?

A disabled button improves the user interface, but events can still come from other sources or code paths. **exhaustMap()** applies the ignore-while-busy rule to the stream itself. A clear loading state and suitable UI behavior are still important.

## 25. How do I choose the operator?

Ask what should happen when a second event arrives while work A is still active:

- Cancel A and keep the latest event? Use **switchMap()**.
- Keep A and run B concurrently? Use **mergeMap()**.
- Keep A and queue B? Use **concatMap()**.
- Keep A and discard B? Use **exhaustMap()**.

## 26. What happens if a switchMap() inner Observable never completes?

It remains active until a new outer value arrives or the outer/result subscription is cancelled. A newer outer value unsubscribes from it and replaces it.

## 27. What happens if a concatMap() inner Observable never completes?

The next queued inner Observables never start. The output remains blocked on that active inner unless the stream is cancelled.

## 28. What happens if an exhaustMap() inner Observable never completes?

The operator keeps ignoring future outer values because the accepted inner is still active. Make sure long-running inner work has a deliberate completion or cancellation path.

## 29. What happens if a mergeMap() inner Observable never completes?

That inner remains active. Other inner streams may still start and emit, but the flattened output cannot complete while an active inner remains.

## 30. When does the flattened output complete?

It generally waits for the outer source to complete and for the active inner work to finish. **concatMap()** can also have queued inners left to process. A never-ending outer or inner stream can keep the result active.

## 31. What happens if an inner Observable errors?

Unless the error is handled, it errors the flattened output and ends that subscription. If a long-lived outer stream should continue after one request fails, handle that request's error inside its projection. The next chapter covers error handling.

## 32. How does switchMap() help with type-ahead search?

A new query replaces the previous request subscription. Angular **HttpClient** can abort an in-progress request when unsubscribed, so an obsolete response is less likely to update the current view.

Use **debounceTime()** to wait for typing to pause and **distinctUntilChanged()** to avoid requests for identical consecutive queries. These operators complement **switchMap()**; they do not replace its cancellation policy.

## 33. Does switchMap() wait for a debounce period before cancelling the previous request?

Only if the stream is ordered with a debounce operator before **switchMap()**. For example, **query$.pipe(debounceTime(250), switchMap(...))** waits for the debounce period before sending the debounced value downstream. That means a prior inner remains subscribed until the next debounced value arrives.

Choose the order based on the desired behavior and consider when the previous request should be replaced.

## 34. Can I use switchMap() with a timer or animation?

Yes. The inner Observable can be any suitable stream, including **timer()**, an event stream, or an HTTP request. A new outer value switches away from the current inner stream.

## 35. Does subscribing to a higher-order Observable automatically handle component destruction?

No. Flattening operators define how outer and inner streams relate. The final component-owned subscription still needs an owner. Prefer **AsyncPipe** or **toSignal()** for template data, and use **takeUntilDestroyed()** for imperative component subscriptions.

## 36. What does takeUntilDestroyed() do?

It completes the piped Observable when the associated Angular context is destroyed, which unsubscribes the resulting subscription and its active inner work. In an injection context it can find the current destruction reference; otherwise pass a **DestroyRef** explicitly.

## 37. Can I manually subscribe inside switchMap()?

Avoid nested manual subscriptions for ordinary request flows. Return the inner Observable from the projection so the flattening operator can manage its subscription, cancellation, errors, and completion as one pipeline.

~~~ts
const results$ = searchTerms$.pipe(
  switchMap((term) => searchApi.search(term)),
);
~~~

## 38. How can I tell which inner stream was cancelled?

In a focused demo, place **finalize()** inside the projected Observable and track whether it emitted or completed. In production, use appropriate logging or request instrumentation without adding side effects that change the stream's behavior.

## 39. Can I change a flattening operator without changing the inner request function?

Yes. The projection can remain the same while the operator changes the policy. That can change cancellation, concurrency, ordering, and which values are ignored or queued, so review those behaviors before swapping operators.

## 40. What is the key difference between combining and flattening?

Combination operators join values from independent streams, such as **combineLatest()** and **withLatestFrom()** from the previous chapter. Flattening operators handle a source that emits other Observables and decide how to subscribe to those inner streams.

## Further reading

- [RxJS switchMap() API](https://rxjs.dev/api/operators/switchMap)
- [RxJS mergeMap() API](https://rxjs.dev/api/operators/mergeMap)
- [RxJS concatMap() API](https://rxjs.dev/api/operators/concatMap)
- [RxJS exhaustMap() API](https://rxjs.dev/api/operators/exhaustMap)
- [Angular HttpClient request cancellation](https://angular.dev/guide/http/making-requests)
- [Angular takeUntilDestroyed() API](https://angular.dev/api/core/rxjs-interop/takeUntilDestroyed)
- [Angular RxJS cleanup guide](https://angular.dev/ecosystem/rxjs-interop/take-until-destroyed)

## Continue learning

- Lesson: [Flattening operators, cancellation, and concurrency](../lessons/059-flattening-operators-cancellation-and-concurrency.md)
- Example: [Flattening operators, cancellation, and concurrency](../examples/059-flattening-operators-cancellation-and-concurrency.md)
- Next lesson: [Error handling, cleanup, and takeUntilDestroyed](../lessons/060-error-handling-cleanup-and-takeuntildestroyed.md)
