# HTTP Errors, Retry Policy, and Timeout Handling

HTTP requests can fail because of a network problem, a timeout, or an error response from the server. Angular reports request failures through the HttpClient Observable's error channel as HttpErrorResponse. RxJS operators can retry selected failures, but retries must be bounded and appropriate for the operation.

## Learning outcomes

After this lesson, you should be able to:

- Distinguish network failures, timeouts, and backend error responses.
- Read the useful parts of HttpErrorResponse without exposing unsafe details to users.
- Set Angular's per-request timeout option.
- Add a bounded retry policy with RxJS.
- Retry transient failures while avoiding unsafe repeated mutations.
- Use catchError to translate or rethrow an error at the right boundary.

## 1. HttpErrorResponse

HttpClient emits a response through the Observable error channel when a request fails. The error is an HttpErrorResponse, which includes status, headers, URL, and an error payload where available.

~~~ts
import { HttpErrorResponse } from '@angular/common/http';

function describeError(error: HttpErrorResponse): string {
  if (error.status === 0) {
    return 'A network or timeout failure occurred.';
  }

  return 'The server returned HTTP ' + error.status + '.';
}
~~~

Three broad failure classes are useful to distinguish:

| Failure | Typical signal |
| --- | --- |
| Network or connection failure | status is 0; no usable backend response |
| Request timeout | status is 0; the request exceeded its configured timeout |
| Backend failure | nonzero HTTP status such as 400, 404, or 500 |

The exact error payload depends on the backend and failure. Treat error.error as external data, not as a trusted object with guaranteed fields.

## 2. Set a request timeout

The HttpClient timeout option is a number of milliseconds:

~~~ts
this.http.get<Product[]>('/api/products', {
  timeout: 5000
});
~~~

If the backend request does not complete within the limit, Angular aborts it and emits an error. In current Angular, a timeout error is represented as an HttpErrorResponse with status 0.

Angular's timeout option applies to the backend request itself. It does not include time spent in interceptor logic before or after that backend request. A timeout is not a guarantee that the whole application flow, including all interceptor work, has finished within the same duration.

If retries are enabled, each retry makes another backend attempt and is subject to that attempt's timeout. The total elapsed time can therefore be longer than one timeout value.

## 3. Retry only selected transient failures

RxJS retry resubscribes to the source Observable after an error. HttpClient Observables are cold, so each resubscription sends a new backend request and runs the interceptor chain again.

A simple retry(count) retries every error and can repeat permanently failing requests. Use a delay function to retry only conditions that may recover:

~~~ts
import { HttpErrorResponse } from '@angular/common/http';
import { retry, throwError, timer } from 'rxjs';

function isRetryable(error: unknown): error is HttpErrorResponse {
  return error instanceof HttpErrorResponse
    && (error.status === 0 || error.status >= 500);
}

const products$ = this.http.get<Product[]>('/api/products', {
  timeout: 5000
}).pipe(
  retry({
    count: 2,
    delay: (error, retryCount) => {
      if (!isRetryable(error)) {
        return throwError(() => error);
      }

      const delayMs = Math.min(250 * 2 ** (retryCount - 1), 4000);
      return timer(delayMs);
    }
  })
);
~~~

With count: 2, RxJS allows two retries after the original attempt, for up to three attempts total. The increasing delay reduces rapid repeated traffic. Add random jitter when many clients might retry at the same time.

This is an example policy, not a universal rule. Some APIs have specific retryable status codes or a Retry-After response header. Follow the server contract.

## 4. Be careful retrying mutations

A GET request is generally safe to retry because it is intended to retrieve data. A timed-out POST may have reached and changed the server even though the client never received the response. Retrying it can create duplicate records or repeat an action.

For state-changing operations:

- Retry only when the operation is idempotent or the server supports an idempotency key.
- Confirm that the API contract allows retrying the specific method.
- Do not assume a client timeout means the server did nothing.
- Consider a status lookup or reconciliation flow for uncertain outcomes.

The HTTP method alone does not prove an endpoint is safe to retry; the server's behavior matters.

## 5. Use catchError to translate or rethrow

catchError can translate an error into a deliberate fallback or rethrow it:

~~~ts
import { HttpErrorResponse } from '@angular/common/http';
import { catchError, of, throwError } from 'rxjs';

const product$ = this.http.get<Product>('/api/products/42').pipe(
  catchError((error: HttpErrorResponse) => {
    if (error.status === 404) {
      return of(null);
    }

    return throwError(() => error);
  })
);
~~~

This changes the resulting type because the stream can now emit Product or null. Only use a fallback when it has a clear meaning in the application. Converting every error to an empty list can hide a real outage as if the server returned valid empty data.

## 6. Put retry before final error handling

A common pipeline retries first, then maps the final failure:

~~~ts
return this.http.get<Product[]>('/api/products', {
  timeout: 4000
}).pipe(
  retry({
    count: 2,
    delay: retryDelay
  }),
  catchError(error => {
    this.logger.warn('Product request failed', error);
    return throwError(() => error);
  })
);
~~~

If catchError converts the error to a successful fallback before retry sees it, retry has no error to retry. Put operators in the order that matches the intended policy.

## 7. Keep user messages separate from diagnostic details

Technical error fields can contain URLs, server payloads, or implementation details. Log or report only what the application's policy allows. Show a safe message that helps the user recover:

~~~ts
this.productApi.getAll().subscribe({
  next: products => this.products.set(products),
  error: error => {
    this.logger.error('Could not load products', error);
    this.errorMessage.set('Products could not be loaded. Try again.');
  }
});
~~~

Avoid displaying a raw stack trace or unfiltered server message to users.

## 8. Errors and interceptor event streams

An interceptor's next(req) returns an Observable of HttpEvents. Successful responses appear as HttpEventType.Response events. Failures generally arrive through the Observable's error channel, so handle them with catchError or an error callback rather than expecting a normal response event.

Retries can run the interceptor chain again. Interceptors should be designed with that in mind: avoid non-idempotent body mutation and duplicate side effects.

## 9. Common mistakes

### Retrying every error forever

A missing route, invalid request, or authentication failure will not usually be fixed by repeating it. Always set a finite retry count and select retryable errors.

### Retrying a write after a timeout without checking semantics

The server may have completed the write even though the response was lost. Use an idempotency strategy or reconcile the result.

### Treating status 0 as a server status code

Status 0 usually indicates a network-level or timeout failure with no normal HTTP response. It is not an HTTP status returned by the backend.

### Assuming the timeout bounds the entire chain

Angular's request timeout only applies to the backend request. Interceptor delays and other application work are outside that bound.

### Showing error.error directly

The payload is backend-controlled and may be missing, malformed, or unsafe for presentation. Map it to an intentional user-facing message.

### Forgetting retry resubscribes

Each retry sends a new HTTP request and runs the interceptor chain again. Account for repeated side effects and request bodies.

## Key idea

Treat an HTTP failure as data in an error channel, distinguish transient failures from permanent ones, and use finite retries only when repeating the operation is safe. Angular's timeout bounds each backend attempt, not the entire interceptor chain.

## Related references

- Q&A: qnas/090-http-errors-retry-policy-and-timeout-handling.md
- Examples: examples/090-http-errors-retry-policy-and-timeout-handling.md
- Angular HTTP errors and timeouts: https://angular.dev/guide/http/making-requests
- Angular interceptors and retrying: https://angular.dev/guide/http/interceptors
- RxJS retry operator: https://rxjs.dev/api/index/function/retry
- RxJS RetryConfig: https://rxjs.dev/api/operators/RetryConfig
