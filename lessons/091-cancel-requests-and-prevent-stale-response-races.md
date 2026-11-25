# Cancel Requests and Prevent Stale Response Races

Search boxes, filters, route changes, and rapidly updated controls can start several HTTP requests before earlier ones finish. If an older request responds last, its data can overwrite the result for the user's newest input. Angular and RxJS give us ways to cancel obsolete client requests and make the latest result authoritative.

## Learning outcomes

After this lesson, you should be able to:

- Explain how unsubscribing affects an in-flight HttpClient request.
- Use switchMap for latest-value-wins request flows.
- Place debounce and error handling so new input cancels old work promptly.
- Choose between switchMap, concatMap, mergeMap, and exhaustMap based on intent.
- Clean up manual subscriptions when a component is destroyed.
- Recognize that client-side cancellation does not roll back server-side work.

## 1. Why stale responses happen

Suppose a user types three queries in quick succession:

| Query | Request starts | Response arrives |
| --- | --- | --- |
| ang | 0 ms | 900 ms |
| angu | 100 ms | 500 ms |
| angular | 200 ms | 350 ms |

If the application starts every request independently, responses can arrive in a different order from the requests. A naive subscription may show the result for ang after the result for angular has already appeared.

This is a race condition: the final UI depends on response timing instead of the latest user intent.

## 2. HttpClient Observables and cancellation

HttpClient returns cold Observables. The request begins when something subscribes. Unsubscribing from an in-progress HttpClient Observable aborts the client-side request. This is why the async pipe and RxJS operators that unsubscribe from an inner Observable can clean up old requests.

Cancellation is not a rollback guarantee. The server may already have received and processed a request before the client aborts its connection. Treat cancellation as stopping client-side work and ignoring an obsolete result, not as proof that server-side work never happened.

## 3. Use switchMap for latest-value-wins flows

switchMap maps each source value to an inner Observable. When a new source value arrives, it unsubscribes from the previous inner Observable and subscribes to the new one. This makes it a good fit when only the result for the newest search, filter, or route parameter matters.

~~~ts
import { HttpClient } from '@angular/common/http';
import { FormControl } from '@angular/forms';
import {
  catchError,
  distinctUntilChanged,
  map,
  of,
  startWith,
  switchMap,
  timer
} from 'rxjs';

interface SearchResult {
  id: string;
  title: string;
}

type SearchState =
  | { kind: 'idle'; query: string }
  | { kind: 'loading'; query: string }
  | { kind: 'success'; query: string; results: SearchResult[] }
  | { kind: 'error'; query: string };

class SearchModel {
  private readonly http = inject(HttpClient);

  readonly state$ = this.searchControl.valueChanges.pipe(
    startWith(this.searchControl.value),
    map((value) => value.trim()),
    distinctUntilChanged(),
    switchMap((query) => {
      if (query.length < 2) {
        return of({ kind: 'idle' as const, query });
      }

      return timer(250).pipe(
        switchMap(() => http.get<SearchResult[]>('/api/search', {
          params: { q: query }
        })),
        map((results): SearchState => ({
          kind: 'success',
          query,
          results
        })),
        startWith({ kind: 'loading', query } satisfies SearchState),
        catchError(() => of<SearchState>({ kind: 'error', query }))
      );
    })
  );
}
~~~

This sketch focuses on the stream. In a component, searchControl can be a non-nullable FormControl, and the model can be a component or a service.

The timer is inside switchMap. As soon as a new distinct query arrives, switchMap cancels the previous timer or HTTP request, then starts a fresh 250 ms delay. Putting debounceTime before switchMap would wait for the debounce period before notifying switchMap, so the previous HTTP request could remain active while the user is typing.

In application code, let TypeScript infer the stream type where possible. If a helper is extracted, declare its return type as Observable<SearchState> rather than using any. The expanded example file contains a complete component version.

## 4. Keep errors inside the inner request

Place catchError inside the switchMap request branch when later source values should continue to work after one request fails. If catchError is placed outside switchMap and replaces the whole stream with a fallback, that outer stream can complete; future input may no longer start requests.

~~~ts
queries$.pipe(
  switchMap((query) =>
    http.get<SearchResult[]>('/api/search', {
      params: { q: query }
    }).pipe(
      catchError(() => of([] as SearchResult[]))
    )
  )
);
~~~

This fallback keeps the input stream alive, but an empty array does not say whether the search succeeded with no matches or failed. For user interfaces, a discriminated state such as idle, loading, success, and error communicates the difference more clearly.

## 5. Choose the flattening operator by intent

The right operator depends on what should happen when a new value arrives while work is active.

| Operator | Behavior when new source value arrives | Typical use |
| --- | --- | --- |
| switchMap | Unsubscribe from the previous inner Observable; use the newest one | Search, filters, route-driven reads |
| concatMap | Queue the new work until prior work completes | Ordered writes or sequential steps |
| mergeMap | Run inner Observables concurrently | Independent requests where every result matters |
| exhaustMap | Ignore new source values while current work is active | Prevent repeated submit actions |

Do not choose switchMap merely because it is familiar. If every write must be processed, cancelling the client's subscription to an earlier write can be the wrong application behavior. Consider concatMap or a server-supported idempotency strategy.

## 6. Bind subscriptions to component lifetime

Prefer the async pipe or toSignal for component data streams because they manage the subscription lifecycle. If an imperative subscription is necessary, use takeUntilDestroyed so it is unsubscribed when the component is destroyed.

~~~ts
import { Component, inject } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { SearchService } from './search.service';

@Component({
  selector: 'app-search',
  template: ''
})
export class SearchComponent {
  private readonly search = inject(SearchService);

  constructor() {
    this.search.notifications$
      .pipe(takeUntilDestroyed())
      .subscribe((message) => {
        this.showNotification(message);
      });
  }

  private showNotification(message: string): void {
    // Update component-owned UI state.
  }
}
~~~

Calling takeUntilDestroyed() without an argument requires an Angular injection context, such as a constructor. If the subscription is created later in an event handler or another method, inject DestroyRef and pass it to takeUntilDestroyed(this.destroyRef).

For an ordinary HttpClient request, automatic completion after one response means a manual cleanup operator is often unnecessary solely to prevent a long-lived subscription. Cleanup is still valuable when a component can be destroyed before the response arrives, and it is essential for long-lived streams such as events, WebSockets, or application state.

## 7. Search component with explicit UI states

The examples file includes a full standalone component. Its stream uses switchMap to cancel the prior timer or request, catches errors inside the request branch, and exposes the result as a state Observable for the async pipe.

~~~ts
readonly state$ = this.searchControl.valueChanges.pipe(
  startWith(this.searchControl.value),
  map((value) => value.trim()),
  distinctUntilChanged(),
  switchMap((query) => {
    if (query.length < 2) {
      return of<SearchState>({ kind: 'idle', query });
    }

    return timer(250).pipe(
      switchMap(() => http.get<SearchResult[]>('/api/search', {
        params: { q: query }
      })),
      map((results): SearchState => ({
        kind: 'success',
        query,
        results
      })),
      startWith<SearchState>({ kind: 'loading', query }),
      catchError(() => of<SearchState>({ kind: 'error', query }))
    );
  })
);
~~~

A template can render each state separately. This avoids showing stale results as if they belong to the latest query and gives the user clear loading and error feedback.

## Common mistakes

### Starting independent subscriptions for every keystroke

Each subscription may issue a separate request, and results can update the UI out of order. Use one stream with switchMap for latest-value-wins reads.

### Putting debounce before switchMap when immediate cancellation matters

The old request remains subscribed until the debounced value is emitted. Put a timer or debounce inside the switchMap branch when each raw input should cancel prior work right away.

### Placing catchError outside the request branch

Replacing the outer stream with a fallback may stop future queries from being processed. Catch request errors inside the inner Observable when the source should remain active.

### Treating an aborted request as a server rollback

The server may already have received the request. Cancellation controls the client subscription and transport; it does not undo a mutation that the server has processed.

### Using switchMap for every kind of operation

Latest-wins is correct for searches and many reads. It may be wrong when every submitted write or queued operation must complete.

## Key idea

Use switchMap when newer input makes older request results irrelevant. HttpClient unsubscription can abort an in-flight client request, while the stream structure prevents stale results from reaching the UI. Choose another operator when the application must preserve every operation, and never assume client cancellation rolls back server-side effects.

## Related references

- Q&A: qnas/091-cancel-requests-and-prevent-stale-response-races.md
- Examples: examples/091-cancel-requests-and-prevent-stale-response-races.md
- Angular making HTTP requests and cancelling subscriptions: https://angular.dev/guide/http/making-requests
- Angular takeUntilDestroyed: https://angular.dev/ecosystem/rxjs-interop/take-until-destroyed
- RxJS switchMap: https://rxjs.dev/api/operators/switchMap
