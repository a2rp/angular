# Error handling, cleanup, and takeUntilDestroyed

Asynchronous work can succeed, fail, complete without a value, or be cancelled before it finishes. A reliable Angular UI handles each outcome deliberately. RxJS provides operators for recovering from errors and cleaning up work; Angular provides lifecycle-aware tools such as **takeUntilDestroyed()**.

## Learning outcomes

After this lesson, you should be able to:

- Distinguish an Observable error from normal completion and unsubscription.
- Recover from an error with **catchError()** and choose where recovery belongs.
- Use bounded **retry()** only when repeating the operation is safe.
- Represent loading, success, and error states in the UI.
- Explain what **finalize()** does on success, failure, and cancellation.
- Choose **AsyncPipe**, **toSignal()**, or **takeUntilDestroyed()** for subscription ownership.
- Handle Angular **HttpErrorResponse** without exposing sensitive server details.

## The three Observable endings

An Observable can notify its observer of values, then end in one of two ways:

1. **Complete:** the source finished normally.
2. **Error:** the source failed. An error is terminal for that subscription; it will not emit more values or later complete.

Unsubscription is different. It means the consumer no longer wants notifications. It may happen explicitly, when **switchMap()** replaces an inner stream, or when an Angular view is destroyed. A cancelled stream does not send a normal completion notification to its observer.

This distinction matters for UI behavior and cleanup. A completed request may show its result. An errored request needs a recovery or error state. A cancelled request should stop updating a view that no longer needs the result.

## Handle errors with catchError()

**catchError()** listens for the error notification and replaces the failed source with another Observable. The replacement might emit a fallback value, complete without a value, or rethrow the error.

~~~ts
import { catchError, of } from 'rxjs';

const profile$ = this.http.get<Profile>('/api/profile').pipe(
  catchError((error: unknown) => {
    console.error('Could not load the profile', error);
    return of<Profile | null>(null);
  }),
);
~~~

This example turns a request failure into a normal emission of **null**. The subscriber now needs to handle that fallback as part of its value type. Do not return a fallback merely to silence an error; choose a value that the rest of the application can interpret safely.

To preserve the failure for a caller while performing a side effect, return a new error Observable:

~~~ts
import { catchError, throwError } from 'rxjs';

const profile$ = this.http.get<Profile>('/api/profile').pipe(
  catchError((error: unknown) => {
    this.logger.error('Profile request failed', error);
    return throwError(() => error);
  }),
);
~~~

The error callback passed to **throwError()** creates the error when the replacement Observable is subscribed. Handle the error once at the layer that can make a useful decision. Avoid logging the same failure in every service, interceptor, and component.

A final **subscribe({ error })** handler is useful as a last boundary for reporting an unhandled failure from an imperative subscription. It does not transform the stream or let it continue. Prefer **catchError()** in the pipeline when the stream needs a defined fallback or user-facing state.

## Put recovery at the right level

With a long-lived input stream that starts one request per value, catch an individual request error inside the flattening operator:

~~~ts
import { catchError, of, switchMap } from 'rxjs';

const results$ = this.searchTerms$.pipe(
  switchMap((term) =>
    this.searchApi.search(term).pipe(
      catchError((error: unknown) => {
        this.logger.warn('Search failed', error);
        return of<SearchResult[]>([]);
      }),
    ),
  ),
);
~~~

Each request failure becomes an empty result for that query. The outer **searchTerms$** stream remains subscribed, so a later search term can start another request.

If **catchError()** is placed after **switchMap()**, it handles an error from the flattened result. Returning a fallback can complete the replacement stream; it does not automatically resubscribe to the original input stream. This placement is appropriate when one failure should end the whole workflow or when the entire output has one recovery policy. For interactive input streams, inner recovery is often the intended behavior.

## Model request state explicitly

A screen should make its request state visible instead of leaving stale content or swallowing errors. A discriminated union keeps the possible states clear:

~~~ts
import { catchError, map, of, startWith } from 'rxjs';

type LoadState =
  | { kind: 'loading' }
  | { kind: 'success'; profile: Profile }
  | { kind: 'error'; message: string };

const state$ = this.profileApi.load().pipe(
  map((profile): LoadState => ({ kind: 'success', profile })),
  catchError((error: unknown) =>
    of<LoadState>({
      kind: 'error',
      message: 'The profile could not be loaded. Try again.',
    }),
  ),
  startWith<LoadState>({ kind: 'loading' }),
);
~~~

The component can render loading, success, and error branches separately. Keep technical diagnostics in logging and show a concise, actionable message to the user. Do not display raw exception text, stack traces, internal URLs, or server response bodies unless they are deliberately designed as safe public messages.

Angular **HttpClient** reports request failures as **HttpErrorResponse** values in the Observable error channel. A network or connection failure commonly has status **0**; an HTTP response error carries the backend status code. The response body is untrusted input, even if an interface describes its expected shape. Validate or narrow it before using server-provided fields.

~~~ts
import { HttpErrorResponse } from '@angular/common/http';

function describeHttpFailure(error: unknown): string {
  if (error instanceof HttpErrorResponse) {
    if (error.status === 0) {
      return 'The server could not be reached. Check your connection and try again.';
    }

    if (error.status === 401) {
      return 'Sign in again to continue.';
    }
  }

  return 'Something went wrong. Please try again.';
}
~~~

Use the application's authentication flow for authorization failures, and avoid treating every status as the same retryable problem.

## Retry only when repeating work is safe

**retry()** resubscribes to the source after an error. With **retry({ count: 2 })**, RxJS makes at most two resubscriptions, for up to three total attempts. A delay can space out retries:

~~~ts
import { retry, timer } from 'rxjs';

const data$ = this.http.get<Data>('/api/data').pipe(
  retry({
    count: 2,
    delay: (_error, retryCount) => timer(retryCount * 300),
  }),
);
~~~

Retries can be useful for short-lived connection problems or transient service failures. They are not a universal error handler. Before retrying, consider:

- Is the failure likely to be temporary?
- Is the operation safe to repeat?
- Is the retry count bounded?
- Does the user need to know that the operation is still pending?
- Could retrying duplicate a write or repeat a side effect?

A new subscription to **HttpClient** sends a new request. Retrying a read is often safer than automatically repeating a payment, message send, or other write. Server-side idempotency keys and an explicit product policy may be needed for retryable writes. Also remember that a cold source can emit earlier values again after a retry; downstream consumers must not mistake a repeated emission for a new unique event.

Put **retry()** before **catchError()** when the recovery handler should run only after the retry policy is exhausted:

~~~ts
const result$ = this.api.load().pipe(
  retry({ count: 2 }),
  catchError((error: unknown) => of(makeErrorState(error))),
);
~~~

An unbounded retry can keep a failed UI loading indefinitely and create unnecessary traffic. Use bounded retries, an intentional delay, and an observable user state.

## Use finalize() for cleanup

**finalize()** runs its callback when the source completes, errors, or is unsubscribed. It is useful for releasing resources or recording that a particular operation ended:

~~~ts
import { finalize } from 'rxjs';

const request$ = this.api.load().pipe(
  finalize(() => this.logger.debug('Request subscription ended')),
);
~~~

It does not recover from errors, and it does not tell you by itself whether the stream succeeded. Use **map()**, **catchError()**, and completion handling for the outcome; use **finalize()** for cleanup that must run for every ending path.

Be careful with a shared boolean loading flag when requests can overlap. If an older request is cancelled, its **finalize()** can set the flag to false while a newer request is still running. Prefer a state stream scoped to each request, a request counter, or a single latest-request policy such as **switchMap()**.

## Give subscriptions an owner

A subscription should have a clear lifetime. Choose the owner that matches how the value is used:

- **AsyncPipe** subscribes for a template and unsubscribes when that view is destroyed.
- **toSignal()** converts an Observable to a signal and manages its subscription with the owning Angular context by default.
- **takeUntilDestroyed()** completes an imperative subscription when its Angular context is destroyed.
- Finite sources such as a one-shot **HttpClient** request normally complete on their own, but cancellation on component destruction can still prevent obsolete work and callbacks.

~~~ts
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

this.route.paramMap.pipe(
  switchMap((params) => this.pageApi.load(params.get('id'))),
  takeUntilDestroyed(),
).subscribe((page) => {
  this.pageState.set(page);
});
~~~

When **takeUntilDestroyed()** is called in an injection context, Angular can use the current **DestroyRef**. In a method invoked later, there may be no current injection context. Inject and pass the **DestroyRef** explicitly:

~~~ts
import { DestroyRef, inject } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

export class SearchComponent {
  private readonly destroyRef = inject(DestroyRef);

  startListening(): void {
    this.searchEvents$.pipe(
      takeUntilDestroyed(this.destroyRef),
    ).subscribe((event) => this.handle(event));
  }

  // Other component members are omitted.
}
~~~

Place **takeUntilDestroyed()** after operators that create inner subscriptions, such as **switchMap()**, so destroying the context tears down the composed chain and its active inner work. The API is stable since Angular 19; check compatibility when maintaining an older Angular application.

## A practical decision guide

- Need to show a recoverable request failure? Map the failure to an explicit UI state with **catchError()**.
- Need the input stream to accept another event after one request fails? Handle the error inside that event's inner Observable.
- Need to repeat a request? Use a bounded **retry()** policy only after checking transience and repeat safety.
- Need cleanup on every termination path? Use **finalize()**.
- Need a template subscription? Prefer **AsyncPipe** or **toSignal()**.
- Need imperative component work? Use **takeUntilDestroyed()** with the owning context.
- Need a final safety boundary for a manual subscription? Provide an **error** handler, while keeping recovery decisions in the Observable pipeline.

## Common mistakes

- **Treating an error like completion:** an error is terminal for that subscription unless an operator replaces it.
- **Using catchError() only to log:** logging does not recover; return a deliberate fallback or rethrow.
- **Catching outside a long-lived input pipeline:** one request failure can replace or end the whole outer workflow.
- **Retrying every error:** validation, authentication, and many client errors will not improve through repetition.
- **Retrying a write without idempotency:** the first attempt may have reached the server even if the client saw a failure.
- **Using unbounded retry:** the UI and network may remain busy forever.
- **Confusing finalize() with success:** it runs after success, failure, and unsubscription.
- **Using finalize() to toggle a shared loading boolean:** cancellation from an older request can clear a newer request's loading state.
- **Calling takeUntilDestroyed() outside an injection context with no DestroyRef:** pass an explicitly injected **DestroyRef**.
- **Manually subscribing to template data without a lifetime:** use **AsyncPipe**, **toSignal()**, or **takeUntilDestroyed()** as appropriate.
- **Displaying raw error details:** keep internal diagnostics separate from safe user-facing messages.

## Continue learning

- Previous: [Flattening operators, cancellation, and concurrency](059-flattening-operators-cancellation-and-concurrency.md)
- Q&A: [Error handling, cleanup, and takeUntilDestroyed](../qnas/060-error-handling-cleanup-and-takeuntildestroyed.md)
- Example: [Error handling, cleanup, and takeUntilDestroyed](../examples/060-error-handling-cleanup-and-takeuntildestroyed.md)
- Next: [Choose template-driven, reactive, or signal forms](061-choose-template-driven-reactive-or-signal-forms.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="059-flattening-operators-cancellation-and-concurrency.md">059 - Flattening operators, cancellation, and concurrency</a></td><td><a href="../qnas/060-error-handling-cleanup-and-takeuntildestroyed.md">Q&amp;A</a></td><td><a href="../examples/060-error-handling-cleanup-and-takeuntildestroyed.md">Example</a></td><td><a href="061-choose-template-driven-reactive-or-signal-forms.md">061 - Choose template-driven, reactive, or signal forms</a></td>
</tr>
  </tbody>
</table>
