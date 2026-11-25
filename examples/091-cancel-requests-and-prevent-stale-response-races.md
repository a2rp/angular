# Cancel Requests and Prevent Stale Response Races: Examples

This chapter uses a search box to demonstrate canceling obsolete requests. It also shows lifecycle cleanup, route-driven reads, operator selection, and the boundary between client cancellation and server-side work.

## 1. Complete latest-query-wins search component

The component below normalizes user input, waits briefly before making a request, and uses switchMap to replace the previous timer or HTTP request as soon as a new query arrives.

### search.component.ts

~~~ts
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { AsyncPipe } from '@angular/common';
import { Component, inject } from '@angular/core';
import { FormControl, ReactiveFormsModule } from '@angular/forms';
import {
  catchError,
  distinctUntilChanged,
  map,
  Observable,
  of,
  startWith,
  switchMap,
  timer
} from 'rxjs';

interface SearchResult {
  id: string;
  title: string;
  description: string;
}

type SearchState =
  | { kind: 'idle'; query: string }
  | { kind: 'loading'; query: string }
  | { kind: 'success'; query: string; results: SearchResult[] }
  | { kind: 'error'; query: string; message: string };

function getSearchErrorMessage(error: unknown): string {
  if (!(error instanceof HttpErrorResponse)) {
    return 'The search could not be completed. Please try again.';
  }

  if (error.status === 0) {
    return 'The service could not be reached. Check your connection and try again.';
  }

  if (error.status >= 500) {
    return 'The search service is temporarily unavailable.';
  }

  return 'The search could not be completed. Please try again.';
}

@Component({
  selector: 'app-search',
  standalone: true,
  imports: [AsyncPipe, ReactiveFormsModule],
  templateUrl: './search.component.html'
})
export class SearchComponent {
  private readonly http = inject(HttpClient);

  readonly query = new FormControl('', { nonNullable: true });

  readonly state$: Observable<SearchState> = this.query.valueChanges.pipe(
    startWith(this.query.value),
    map((value) => value.trim()),
    distinctUntilChanged(),
    switchMap((query) => {
      if (query.length < 2) {
        return of<SearchState>({ kind: 'idle', query });
      }

      return timer(250).pipe(
        switchMap(() => this.http.get<SearchResult[]>('/api/search', {
          params: { q: query }
        })),
        map((results): SearchState => ({
          kind: 'success',
          query,
          results
        })),
        startWith({ kind: 'loading', query } satisfies SearchState),
        catchError((error: unknown): Observable<SearchState> =>
          of({
            kind: 'error',
            query,
            message: getSearchErrorMessage(error)
          })
        )
      );
    })
  );
}
~~~

### search.component.html

~~~html
<label for="search-query">Search</label>
<input
  id="search-query"
  type="search"
  [formControl]="query"
  autocomplete="off"
  placeholder="Enter at least two characters">

@if (state$ | async; as state) {
  @switch (state.kind) {
    @case ('idle') {
      <p>Enter at least two characters to search.</p>
    }

    @case ('loading') {
      <p role="status">Searching for {{ state.query }}…</p>
    }

    @case ('error') {
      <p role="alert">{{ state.message }}</p>
    }

    @case ('success') {
      @if (state.results.length === 0) {
        <p>No results found for {{ state.query }}.</p>
      } @else {
        <ul>
          @for (result of state.results; track result.id) {
            <li>
              <h2>{{ result.title }}</h2>
              <p>{{ result.description }}</p>
            </li>
          }
        </ul>
      }
    }
  }
}
~~~

The timer is inside switchMap, so a new distinct input cancels the previous timer or active request immediately. The new request starts only after the user pauses for 250 milliseconds. The async pipe owns the template subscription and unsubscribes when the view is destroyed.

The UI uses explicit idle, loading, success, and error states. That keeps a failed search distinct from a successful search with zero results.

## 2. Show that inner teardown runs

The finalize operator runs when the request Observable completes, errors, or is unsubscribed. Add it inside the inner request when investigating cleanup:

~~~ts
import { finalize } from 'rxjs';

switchMap(() =>
  this.http.get<SearchResult[]>('/api/search').pipe(
    finalize(() => {
      console.debug('Search request subscription ended');
    })
  )
)
~~~

This log proves the subscription was finalized, but by itself it does not tell you whether the request succeeded, failed, or was canceled. Use it for diagnostics, not as a user-facing cancellation signal.

## 3. Keep the outer input stream alive after an error

This version catches the error inside the request branch. A later query can still start another request:

~~~ts
queries$.pipe(
  switchMap((query) =>
    this.http.get<SearchResult[]>('/api/search', {
      params: { q: query }
    }).pipe(
      catchError(() => of([] as SearchResult[]))
    )
  )
);
~~~

This concise fallback does not distinguish request failure from a successful empty response. In a real interface, return an explicit error state as in the complete component above.

Avoid catching at the outside if the returned replacement would terminate future input handling:

~~~ts
queries$.pipe(
  switchMap((query) =>
    this.http.get<SearchResult[]>('/api/search', {
      params: { q: query }
    })
  ),
  catchError(() => of([] as SearchResult[]))
);
~~~

When that outer catch replaces the failed stream with a one-value fallback, the resulting stream completes. Later queries may no longer be processed.

## 4. Load data for the currently selected route

A route parameter can change while the component stays alive. switchMap makes each selected ID replace the previous read.

~~~ts
import { HttpClient } from '@angular/common/http';
import { ActivatedRoute } from '@angular/router';
import { inject } from '@angular/core';
import { map, of, switchMap } from 'rxjs';

interface User {
  id: string;
  name: string;
}

class UserDetailModel {
  private readonly route = inject(ActivatedRoute);
  private readonly http = inject(HttpClient);

  readonly user$ = this.route.paramMap.pipe(
    map((params) => params.get('id')),
    switchMap((id) => {
      if (!id) {
        return of(null);
      }

      return this.http.get<User>('/api/users/' + encodeURIComponent(id));
    })
  );
}
~~~

In a component template, the async pipe can subscribe to user$ and release the request when that view is destroyed.

## 5. Tie an imperative subscription to component destruction

Prefer the async pipe or toSignal for values rendered by a component. For a subscription that performs an imperative side effect, use takeUntilDestroyed.

~~~ts
import { Component, DestroyRef, inject } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { NotificationService } from './notification.service';

@Component({
  selector: 'app-notifications',
  template: ''
})
export class NotificationsComponent {
  private readonly destroyRef = inject(DestroyRef);
  private readonly notifications = inject(NotificationService);

  listen(): void {
    this.notifications.messages$
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe((message) => {
        this.showMessage(message);
      });
  }

  private showMessage(message: string): void {
    // Update component-owned UI state.
  }
}
~~~

Passing DestroyRef is useful here because listen can run later from a button handler. Calling takeUntilDestroyed() without an argument is appropriate inside an injection context such as a constructor or field initializer.

## 6. Select the operator for the operation

Use the operator that matches what should happen to work already in progress:

~~~ts
// Latest search replaces old search.
searchTerms$.pipe(
  switchMap((term) => searchApi.search(term))
);

// Every write is queued and processed in order.
saveActions$.pipe(
  concatMap((action) => saveApi.save(action))
);

// Independent reads can run concurrently.
ids$.pipe(
  mergeMap((id) => userApi.getUser(id))
);

// Ignore repeated submit clicks while one save is active.
submitClicks$.pipe(
  exhaustMap(() => orderApi.createOrder())
);
~~~

The snippets illustrate operator choice; each needs the corresponding service and operator imports in its own file. exhaustMap can prevent duplicate clicks in the current client stream, but it is not a substitute for server-side idempotency or validation.

## 7. Cancellation is not rollback

For a read request, dropping an obsolete client response is usually harmless. For a write, the server may already have applied the action even if the client unsubscribes before receiving the response.

For operations such as creating an order:

- Do not assume unsubscribing means the order was not created.
- Use a server-supported idempotency key when retries or duplicate submissions are possible.
- Use concatMap if every queued client action should be sent in order.
- Use a dedicated server-side cancellation endpoint or protocol if the server operation itself must stop.

## 8. Request sequence example

Imagine the user types three values before any response arrives:

| Input | What switchMap does |
| --- | --- |
| ang | Starts its delay, then its request |
| angu | Unsubscribes the ang delay or request and starts a new delay |
| angular | Unsubscribes the angu delay or request and starts the newest delay |
| Latest request returns | Emits the state for angular |
| Old server work finishes later | Its canceled client subscription does not update this stream |

The old request might still consume server resources if the backend had already started processing it. Cancellation ensures the obsolete result is not delivered through this client subscription.

## Checklist

- Use one Observable pipeline for changing inputs instead of starting one subscription per keystroke.
- Use switchMap when the newest input supersedes earlier reads.
- Put the debounce timer inside switchMap when old work should be canceled immediately.
- Catch errors inside the inner request branch if future inputs should still work.
- Represent idle, loading, success, and error states distinctly.
- Use async pipe or toSignal for view subscriptions.
- Use takeUntilDestroyed for imperative subscriptions tied to a component lifecycle.
- Choose concatMap, mergeMap, or exhaustMap when their semantics fit better.
- Treat client abort as cleanup, not server rollback.
