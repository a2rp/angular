# Q&A: Error handling, cleanup, and takeUntilDestroyed

These answers review Observable failures, recovery, retries, and Angular subscription lifetimes. See the [lesson](../lessons/060-error-handling-cleanup-and-takeuntildestroyed.md) for the full explanation and the [example](../examples/060-error-handling-cleanup-and-takeuntildestroyed.md) for a runnable demo.

## Observable outcomes

### 1. What are the terminal notifications of an Observable?

An Observable can end by notifying **complete** or **error**. Both are terminal for that subscription. An error means the work failed; completion means the source finished normally.

### 2. Is unsubscription the same as completion?

No. Unsubscription means the consumer stopped listening. It can happen because a view was destroyed or because **switchMap()** replaced an inner stream. The subscriber does not receive a normal completion callback just because it unsubscribed, though teardown and **finalize()** still run.

### 3. Can an Observable continue emitting after it sends an error?

No. An error terminates that subscription. To continue with a defined policy, an operator such as **catchError()** must replace the failed source with another Observable.

### 4. Does a failed Observable automatically retry?

No. Errors are terminal unless the pipeline explicitly retries or replaces the source. **retry()** resubscribes; **catchError()** switches to a recovery Observable.

## Recovering from failures

### 5. What does catchError() do?

**catchError()** receives the error and returns a replacement Observable. That replacement may emit a fallback value, complete without emitting, or fail again.

### 6. Does catchError() resume the original Observable where it failed?

No. It subscribes to the replacement Observable returned by the callback. The original failed subscription has ended; recovery does not resume it from the point of failure.

### 7. What does returning of(fallback) mean?

**of(fallback)** emits that value and then completes. The downstream value type must include the fallback, and the consumer should handle it intentionally.

### 8. What does returning EMPTY mean?

**EMPTY** completes immediately without emitting a value. Use it only when completing without a result is a valid outcome. Otherwise the UI may remain without a success or error state.

### 9. How do I rethrow an error from catchError()?

Return **throwError(() => error)**. This creates a replacement Observable that errors when subscribed, allowing a higher layer to handle the failure.

### 10. Is a subscribe error callback enough to recover?

No. A **subscribe({ error })** callback can report an unhandled failure at the subscription boundary, but it does not replace the failed Observable or make it continue. Put recovery in the pipeline with **catchError()**.

### 11. Can tap({ error }) handle an error?

No. **tap({ error })** is for a side effect such as logging or metrics. It does not recover the source. Use **catchError()** to replace or rethrow the error.

### 12. Where should catchError() go in a switchMap() request pipeline?

Inside the inner Observable when one request failure should be handled while later outer events are still accepted:

~~~ts
terms$.pipe(
  switchMap((term) =>
    search(term).pipe(
      catchError(() => of([])),
    ),
  ),
);
~~~

The failed request becomes an empty result, and the outer input subscription stays active.

### 13. What happens if catchError() is after switchMap()?

It handles errors from the flattened output. Returning a fallback replaces that output stream; if the fallback completes, the original outer source is no longer observed. This is useful when an error should end the overall workflow, but may be wrong for a long-lived input stream.

### 14. Can an outer catchError() restart the source?

It can return a new Observable, including another subscription to a source, but that is an explicit replacement policy. It does not automatically continue the original failed subscription. Be careful to avoid accidental infinite resubscription loops.

### 15. Should I catch errors in both a service and a component?

Only when each layer has a distinct responsibility. A service may add context or translate a known API response; a component may choose a screen state. Logging and swallowing the same error at every layer hides where recovery actually occurs.

## HTTP and UI state

### 16. How does Angular HttpClient report request failures?

It emits an **HttpErrorResponse** through the Observable error channel. Successful responses are emitted as values. A failed request does not arrive as a normal response value.

### 17. What does status 0 usually mean for HttpClient?

It commonly indicates a network, connection, or browser-level failure where an HTTP response status was not available. It is not proof that the server did nothing.

### 18. What does a nonzero HttpErrorResponse status mean?

The server or an intermediary returned an HTTP response with that status. Handle expected statuses according to application policy; for example, authentication and validation errors usually need a different response from temporary server failures.

### 19. Can I trust the error response body to match my TypeScript interface?

No. The body comes from outside the application and is runtime data. Narrow or validate it before reading fields, and show only messages designed to be safe for users.

### 20. Should I show error.message or the whole error object to users?

Usually no. Raw errors may expose internal details or be confusing. Log useful diagnostic context in an appropriate channel and show a concise message with a recovery action.

### 21. What is a useful loading, success, and error model?

A discriminated union such as **{ kind: 'loading' }**, **{ kind: 'success'; data: T }**, and **{ kind: 'error'; message: string }** makes the possible UI states explicit. The template can render one branch for each state.

### 22. What is the difference between a fallback value and an error state?

A fallback value participates in the success-value path and must be distinguishable from real data. An error state communicates that the operation failed. Choose based on whether the application can safely treat the fallback as valid output.

## Retry policy

### 23. What does retry({ count: 2 }) mean?

It allows up to two resubscriptions after errors, which means up to three total source attempts: the initial attempt plus two retries.

### 24. Does retry() rerun a cold HttpClient request?

Yes. Subscribing again to an Angular **HttpClient** Observable sends another request. Use retry only when repeating the operation is part of a safe and intentional policy.

### 25. Which failures are reasonable candidates for retry?

Some transient network interruptions or temporary service failures may be candidates. Validation errors, most authentication failures, and other deterministic client errors usually need correction or user action rather than repetition.

### 26. Is retrying a GET always harmless?

A GET is intended to be safe by HTTP semantics, but applications and servers can still have unexpected side effects or rate limits. Consider the endpoint's actual contract, the retry count, and the user experience.

### 27. Can I automatically retry a POST or payment request?

Do not assume it is safe. The server may have processed the first request even if the client did not receive its response. Use a server-supported idempotency key or another explicit deduplication strategy before retrying a write.

### 28. Can retry cause duplicate values downstream?

Yes. Resubscribing can repeat emissions that occurred before the source failed. Downstream code must account for repeated values or place side effects so retries do not duplicate unintended work.

### 29. Why should retry counts be bounded?

An unbounded retry can leave the UI waiting forever, keep network traffic running, and make recovery invisible. Set an appropriate attempt limit and show a meaningful state when it is exhausted.

### 30. Why put retry() before catchError()?

Then the retry operator sees source errors first. **catchError()** handles the failure only after the retry policy is exhausted. If **catchError()** replaces the source first, the downstream retry operator may see only the replacement's behavior instead of the original error.

### 31. What is retry backoff?

Backoff inserts a delay before another attempt, often increasing with each retry. It reduces rapid repeated traffic during an outage. The delay and maximum attempts should fit the application's latency and service policies.

## Cleanup and ownership

### 32. What does finalize() do?

It runs its callback when the subscription completes, errors, or is unsubscribed. Use it for cleanup and operation-level logging that must happen on every ending path.

### 33. Does finalize() mean the request succeeded?

No. It runs for success, failure, and cancellation. Track the outcome separately with emitted state, error handling, or completion logic.

### 34. Does finalize() recover from an error?

No. It is a teardown callback, not an error-recovery operator. Use **catchError()** for recovery.

### 35. Does finalize() run when switchMap() cancels an inner request?

Yes. Cancellation unsubscribes from the prior inner subscription, so its **finalize()** callback runs.

### 36. Why can finalize() break a shared loading boolean?

A cancelled older request can run its cleanup after a newer request has set loading to true. If the old callback sets the shared flag to false, the UI says it is idle while the newer request is still active. Scope loading state to each request, use a counter, or choose a single latest-request policy.

### 37. When should I use AsyncPipe?

Use **AsyncPipe** when an Observable is primarily consumed by a template. The pipe manages the view subscription and unsubscribes when the view is destroyed.

### 38. When should I use toSignal()?

Use **toSignal()** when a component needs to read Observable values as a signal. It manages its subscription with the current Angular injection context by default; provide an appropriate **Injector** when converting outside a context.

### 39. When should I use takeUntilDestroyed()?

Use it to bind an imperative Observable subscription to an Angular destruction context, commonly for component or directive side effects. It completes the output when that context is destroyed and tears down upstream work.

### 40. Do all HttpClient subscriptions need takeUntilDestroyed()?

A normal one-shot **HttpClient** request completes after its response, so its subscription is finite. Lifecycle cleanup can still be useful if the component may disappear before the response and should not continue obsolete work or update old state.

### 41. Why can takeUntilDestroyed() be called with no argument sometimes?

Inside an Angular injection context, Angular can find the current **DestroyRef**. In a later method call where there is no active injection context, pass a **DestroyRef** that was injected earlier.

### 42. Where should takeUntilDestroyed() be placed after switchMap()?

Usually after **switchMap()** and the rest of the composed pipeline. When the context is destroyed, teardown then reaches the active inner subscription as well as the outer source.

### 43. Is takeUntilDestroyed() stable in every Angular version?

It is stable since Angular 19. Check the API and compatibility of the specific Angular version in an older application before adopting it.

### 44. What if I subscribe to an Observable that never completes?

Give it a clear owner. Use **AsyncPipe** or **toSignal()** for view data, or **takeUntilDestroyed()** for imperative work owned by an Angular context. Otherwise the subscription can outlive the component.

### 45. Does unsubscribing from HttpClient roll back server work?

No. It can abort the client's in-progress request, but the server may already have received or processed it. Treat cancellation as client-side lifecycle control, not transaction rollback.

## References

- [RxJS catchError() API](https://rxjs.dev/api/operators/catchError)
- [RxJS retry() API](https://rxjs.dev/api/index/function/retry)
- [RxJS finalize() API](https://rxjs.dev/api/operators/finalize)
- [Angular HttpClient guide](https://angular.dev/guide/http/making-requests)
- [Angular takeUntilDestroyed() API](https://angular.dev/api/core/rxjs-interop/takeUntilDestroyed)

## Continue learning

- Previous: [Flattening operators, cancellation, and concurrency](../lessons/059-flattening-operators-cancellation-and-concurrency.md)
- Lesson: [Error handling, cleanup, and takeUntilDestroyed](../lessons/060-error-handling-cleanup-and-takeuntildestroyed.md)
- Example: [Error handling, cleanup, and takeUntilDestroyed](../examples/060-error-handling-cleanup-and-takeuntildestroyed.md)
- Next: [Choose template-driven, reactive, or signal forms](../lessons/061-choose-template-driven-reactive-or-signal-forms.md)
