# HTTP Errors, Retry Policy, and Timeout Handling — Q&A

## Understanding failures

### 1. How does HttpClient report a failed request?

The request Observable emits an error, usually an HttpErrorResponse. It does not emit a successful response body for that failed request.

### 2. What does status 0 mean?

Status 0 generally indicates a network-level failure or timeout where there is no normal HTTP response from the backend. It is not a status code returned by the server.

### 3. How are backend errors different?

A backend error has the HTTP status returned by the server, such as 400, 404, or 500. The HttpErrorResponse also includes response metadata and an error payload when one is available.

### 4. Can I assume error.error always has a particular shape?

No. It is external data and can be absent or vary by failure and backend. Validate or safely map fields before using them.

### 5. Should I display error.message or error.error directly?

Usually not. These values can include implementation details or backend-controlled content. Log diagnostics according to policy and show a deliberate, safe user-facing message.

## Timeouts

### 6. How do I set an Angular HttpClient timeout?

Pass timeout in milliseconds in the request options:

~~~ts
this.http.get('/api/config', { timeout: 3000 });
~~~

If the backend request takes longer, Angular aborts the request and emits an error.

### 7. Does Angular's timeout include interceptor time?

No. The HttpClient timeout option applies to the backend HTTP request itself. Time spent in interceptor logic is outside that timeout.

### 8. Does a timeout prove the server did not process the request?

No. It means the client did not receive a completed response within the configured time. The server may have received or even completed the operation.

### 9. Does RxJS retry repeat the timeout for each attempt?

Yes. Each retry resubscribes to the cold HttpClient Observable and starts another backend request, which gets its own configured timeout. Total elapsed time can exceed one timeout value.

## Retry behavior

### 10. What does retry(2) mean?

It permits two resubscriptions after the original attempt, so there can be up to three attempts total. The retry count does not include the initial attempt.

### 11. What does retry do to a cold HttpClient Observable?

It resubscribes after a failure. Each resubscription sends a new backend request and runs the interceptor chain again.

### 12. Does retry only repeat requests that failed before reaching the server?

No. The client may not know whether the server received or processed a request. This uncertainty is especially important for writes and timeouts.

### 13. Which errors should usually be retried?

Potential candidates include temporary network failures and some server-side failures. The correct policy depends on the API. Client errors such as invalid input or missing resources usually need correction rather than repeated requests.

### 14. Should I retry status 429?

Only according to the API's rate-limit policy. If the server provides Retry-After, respect that guidance rather than applying an unrelated short delay. Avoid retrying in a way that increases rate pressure.

### 15. Why use a delay or exponential backoff?

A delay reduces rapid repeated traffic while a service or network is recovering. Exponential backoff increases the pause after successive failures. Add jitter when many clients might retry at the same time.

### 16. Should every HTTP request have automatic retries?

No. Choose retries per request or use a carefully scoped policy. Some operations are not safe to repeat, and some failures are permanent.

### 17. Is retry safe for GET?

GET is generally intended to be safe and idempotent, so retry is commonly appropriate for selected transient failures. Still respect rate limits, user impact, and the particular API's behavior.

### 18. Is retry safe for POST?

Not automatically. The server may process the first POST even if the response is lost. A retry can create duplicates or repeat an action. Use an idempotency key or a server-defined reconciliation flow where supported.

### 19. What about PUT and DELETE?

HTTP semantics often define PUT and DELETE as idempotent, but actual APIs can differ. Verify how the server implements the operation before retrying it automatically.

### 20. Why should retry have a finite count?

Without a limit, retry can resubscribe indefinitely and create ongoing traffic. Use a finite retry count and stop on permanent failures.

## Error operator ordering

### 21. Does catchError retry a request?

No. catchError handles an error and may return a fallback or rethrow. Use retry for resubscription, and place it before final catchError handling if the intent is to retry first.

### 22. Why did retry not run after catchError?

catchError may have transformed the error into a successful fallback value. retry only sees errors that remain in the stream.

### 23. Should I return an empty list for every failed list request?

Usually no. An error is different from a valid empty response. Converting every failure to an empty list can hide outages or authorization problems as normal data.

### 24. Can errors be caught in a component subscribe call?

Yes. An error callback can update the UI. For reusable transformations or shared behavior, catchError can be used in the Observable pipeline. Choose the layer that understands the desired recovery.

## Quick reference

| Situation | Typical handling |
| --- | --- |
| Invalid request or validation failure | Show a useful correction; do not retry automatically |
| 401 or 403 | Follow the application's authentication/authorization flow |
| 404 | Show not-found behavior if meaningful |
| Network failure or timeout | Consider a bounded retry for safe operations |
| 5xx response | Consider a limited retry based on API policy |
| 429 response | Respect Retry-After or server rate-limit guidance |
| Timed-out write with unknown outcome | Reconcile or use idempotency support before repeating |

## Related references

- Lesson: lessons/090-http-errors-retry-policy-and-timeout-handling.md
- Examples: examples/090-http-errors-retry-policy-and-timeout-handling.md
- Angular HTTP errors and timeouts: https://angular.dev/guide/http/making-requests
- Angular interceptors: https://angular.dev/guide/http/interceptors
- RxJS retry operator: https://rxjs.dev/api/index/function/retry
- RxJS RetryConfig: https://rxjs.dev/api/operators/RetryConfig
