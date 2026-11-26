# Cancel Requests and Prevent Stale Response Races: Q&A

## 1. What is a stale response?

A stale response belongs to an older user action or input. For example, a search for ang may finish after a later search for angular and replace the newer results with outdated ones.

## 2. Why can responses arrive out of order?

Network latency, server load, caching, and request size vary. The order in which requests start does not guarantee the order in which their responses arrive.

## 3. What does HttpClient return?

Angular HttpClient methods return cold RxJS Observables. The request begins when a subscriber subscribes. A separate subscription to the same Observable generally sends a separate backend request.

## 4. What happens when I unsubscribe from an in-flight HttpClient request?

Angular aborts the client-side in-flight request when its subscription is unsubscribed. Operators such as switchMap can unsubscribe from an obsolete inner request automatically.

## 5. Does aborting a request undo server-side work?

No. The server may already have received and processed the request before the client aborts. Unsubscription stops the client from waiting for or handling that result; it is not a transaction rollback.

## 6. How does switchMap help with stale responses?

switchMap switches to the Observable produced by the newest source value. It unsubscribes from the previous inner Observable, so an old HttpClient result no longer reaches the downstream UI stream.

## 7. When should I use switchMap?

Use it when the newest value supersedes earlier work, such as typeahead search, filters, selected tabs, route parameters, or loading a detail view for the current selection.

## 8. Why is switchMap a good fit for a search box?

A newer query makes an older query's result irrelevant. switchMap cancels the prior timer or request and only emits values from the newest query branch.

## 9. Where should the debounce delay go if I want to cancel immediately on typing?

Put a timer or debounce inside the switchMap branch. A new source value then reaches switchMap immediately and cancels the previous branch, while the new request waits for the delay.

If debounceTime is placed before switchMap, the new value is held back until the debounce period ends. The old request can stay active during that period.

## 10. What does distinctUntilChanged do?

It prevents consecutive equal values from starting duplicate inner work. Normalize input first if whitespace or casing should not count as a meaningful change.

## 11. Does switchMap cancel every kind of Observable?

It unsubscribes from the previous inner Observable. Whether that teardown actually stops external work depends on the Observable's teardown behavior. Angular HttpClient connects teardown to aborting its request.

## 12. What if an older request already returned before the next query?

Its response has already been emitted. switchMap cannot undo a value the UI has already processed, but it prevents later emissions from an unsubscribed prior branch. Usually the previous one-shot HTTP request has completed by then.

## 13. Where should catchError go in a switchMap pipeline?

Put catchError inside the inner request branch when one failed request should display an error but later inputs should still work. An outer catchError that replaces the whole input stream with a fallback can complete the stream and stop future queries.

## 14. Should I return an empty array after a request error?

Only if that behavior is intentional. An empty array can mean a successful search with no matches, so it can hide the difference between an error and a valid empty result. A state union with idle, loading, success, and error variants is clearer.

## 15. Can cancellation trigger the request's error callback?

When the subscription itself is unsubscribed, the subscriber is closed and does not receive a normal error notification from that canceled subscription. Do not treat expected switchMap cleanup as a user-visible request failure.

## 16. How do switchMap, concatMap, mergeMap, and exhaustMap differ?

- switchMap cancels the previous inner subscription and uses the latest one.
- concatMap queues inner work and processes it in order.
- mergeMap runs inner work concurrently.
- exhaustMap ignores new source values while the current inner work is active.

Choose based on whether new input replaces, queues, runs beside, or is ignored during current work.

## 17. Should switchMap be used for POST requests?

Not by default. Unsubscribing from a POST does not prove the server did not apply it. If each submission must be processed, use an operator and API design that preserve that requirement, such as concatMap plus server-side idempotency where appropriate.

## 18. Is switchMap itself a stale-response guard for imperative subscriptions?

It helps when requests are composed into one Observable chain. Starting a new subscription manually for each input bypasses that coordination and can allow out-of-order callbacks. Prefer one stream with switchMap, or implement an explicit request identity guard if an imperative design is unavoidable.

## 19. What does takeUntilDestroyed do?

Angular's takeUntilDestroyed operator completes a stream and unsubscribes when its component, directive, or other Angular context is destroyed. This is useful for imperative subscriptions tied to that lifecycle.

## 20. Can I call takeUntilDestroyed from any method?

Calling it without an argument requires an Angular injection context. In a later event handler or ordinary method, inject DestroyRef and pass it explicitly to takeUntilDestroyed.

## 21. Do I need takeUntilDestroyed for every HttpClient request?

Not always. HttpClient requests usually complete after one response, and the async pipe or toSignal manages its own subscription. Cleanup still matters if a component is destroyed before a response, and it is especially important for long-lived streams.

## 22. How does the async pipe help?

The async pipe subscribes to an Observable used by the template and unsubscribes when the view is destroyed or when the bound Observable reference changes. For a single view subscription, this is often simpler than a manual subscribe call.

## 23. What happens if two parts of a view subscribe to the same cold HttpClient Observable?

Each subscription can issue its own request. If the request should be shared, deliberately share the stream or derive it once and bind it once. Sharing changes cancellation semantics because one consumer unsubscribing does not necessarily stop work still needed by another consumer.

## 24. Can shareReplay prevent cancellation?

It depends on the sharing configuration and whether other subscribers remain active. With a non-ref-counted shared source, the source may stay subscribed after all view consumers unsubscribe. Understand the chosen sharing policy before assuming view teardown cancels the backend request.

## 25. How should loading state behave when requests are canceled?

Model loading as part of the current query branch. When a new query arrives, the prior branch is replaced and the new branch can emit its own loading state. Do not let a late callback from an old request clear the current request's spinner.

## 26. What if the user clears the search box?

Map the empty or too-short query to an idle state without issuing a request. Because that value enters switchMap, it also cancels the prior timer or in-flight request.

## 27. Does route navigation cancel a request automatically?

If the request subscription is owned by the destroyed view, such as through the async pipe, destroying that view unsubscribes and cancels the client request. A service-level subscription that outlives the view will continue unless it is explicitly scoped or canceled.

## 28. How can I verify that an old result is not rendered?

Use a controlled scenario where request A starts, request B starts afterward, and B responds first. Confirm that A cannot replace B's state if it completes later. Also verify that a failing request does not prevent a subsequent query from working.

## 29. Should every old request always be canceled?

No. Cancellation is appropriate when old work is no longer useful. Some operations need all results, ordered completion, or explicit server-side cancellation semantics. Select the operator and API behavior to match the operation.

## 30. What is the main rule for request cancellation?

Use switchMap for latest-value-wins reads, lifecycle-bound cleanup for manual subscriptions, and explicit state for loading and errors. Treat client cancellation as client-side cleanup, not as proof that the server has undone a mutation.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="090-http-errors-retry-policy-and-timeout-handling.md">090 - HTTP Errors, Retry Policy, and Timeout Handling</a></td><td><a href="../lessons/091-cancel-requests-and-prevent-stale-response-races.md">Lesson</a></td><td><a href="../examples/091-cancel-requests-and-prevent-stale-response-races.md">Example</a></td><td><a href="092-authentication-cookies-and-xsrf-protection.md">092 - Authentication, Cookies, and XSRF Protection</a></td>
</tr>
  </tbody>
</table>
