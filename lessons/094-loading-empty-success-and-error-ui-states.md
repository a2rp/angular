# Loading, Empty, Success, and Error UI States

A request-driven view has more than two outcomes. It can be idle before a request starts, loading, successful with data, successful with no data, or failed. Modeling these states explicitly helps the template show the right content and prevents errors from looking like empty results.

## Learning outcomes

After this lesson, you should be able to:

- Model asynchronous UI states with a discriminated union.
- Distinguish a successful empty result from an error.
- Render each state with Angular template control flow.
- Give loading and error feedback accessibly.
- Keep error messages safe and actionable.
- Preserve existing data during refresh when the product design calls for it.
- Keep retry actions available without terminating future requests.

## 1. Model the states explicitly

A boolean such as isLoading cannot describe whether data is empty, successful, or failed. Several independent booleans can also create impossible combinations, such as isLoading and hasError both being true.

Use one status field whose value determines the rest of the state:

~~~ts
export type LoadState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'empty' }
  | { status: 'error'; message: string; retryable: boolean };
~~~

A discriminated union gives each state a clear shape:

| State | Meaning | Typical UI |
| --- | --- | --- |
| idle | No request has started yet | Prompt or no content |
| loading | A request is in progress | Progress indicator |
| success | A request completed with usable data | Data view |
| empty | A request succeeded but returned no items | Helpful empty-state message |
| error | The request failed | Error message and possible retry action |

An empty state is a successful result with no content. A failed request is not an empty result, even if the UI has no data to show.

## 2. Convert a request into a UI state stream

The request layer can map successful values and errors into the union:

~~~ts
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { catchError, map, of, startWith } from 'rxjs';
import type { Observable } from 'rxjs';

interface Product {
  id: string;
  name: string;
}

function getSafeMessage(error: unknown): string {
  if (error instanceof HttpErrorResponse && error.status === 0) {
    return 'The service could not be reached. Check your connection and try again.';
  }

  if (error instanceof HttpErrorResponse && error.status >= 500) {
    return 'The service is temporarily unavailable. Try again later.';
  }

  return 'Products could not be loaded. Please try again.';
}

@Injectable({ providedIn: 'root' })
export class ProductService {
  private readonly http = inject(HttpClient);

  loadProducts(): Observable<LoadState<Product[]>> {
    return this.http.get<Product[]>('/api/products').pipe(
      map((products): LoadState<Product[]> =>
        products.length === 0
          ? { status: 'empty' }
          : { status: 'success', data: products }
      ),
      startWith({ status: 'loading' } as LoadState<Product[]>),
      catchError((error: unknown) =>
        of<LoadState<Product[]>>({
          status: 'error',
          message: getSafeMessage(error),
          retryable: true
        })
      )
    );
  }
}
~~~

The loading state is emitted before the HTTP response. A successful empty array becomes empty, while an HTTP failure becomes error. The UI does not have to infer the outcome from a missing value.

This example marks the error retryable for simplicity. A real application should derive retry availability from its API contract and the operation, and should not offer a retry for every possible failure.

## 3. Render the states explicitly

Angular's @switch template block makes each state visible in one place:

~~~html
@if (state$ | async; as state) {
  <section
    aria-labelledby="products-heading"
    [attr.aria-busy]="state.status === 'loading'">

    <h1 id="products-heading">Products</h1>

    @switch (state.status) {
      @case ('idle') {
        <p>Choose a category to view products.</p>
      }

      @case ('loading') {
        <p role="status">Loading products…</p>
      }

      @case ('success') {
        <ul>
          @for (product of state.data; track product.id) {
            <li>{{ product.name }}</li>
          }
        </ul>
      }

      @case ('empty') {
        <p>No products are available in this category yet.</p>
      }

      @case ('error') {
        <div role="alert">
          <p>{{ state.message }}</p>
          @if (state.retryable) {
            <button type="button" (click)="reload()">Try again</button>
          }
        </div>
      }
    }
  </section>
}
~~~

The async pipe subscribes to state$ and cleans up its view subscription when the view is destroyed. A status message can announce progress without moving keyboard focus. An alert is suitable for an important error message; do not put every routine state change in an assertive announcement.

The template assumes that the component exposes state$ and reload(). The complete example file shows a standalone component and its request stream.

## 4. Keep messages useful and safe

Show a brief message that helps the user decide what to do next. Do not place raw server response bodies, stack traces, internal URLs, or sensitive information directly in the page.

Different failures can deserve different next steps:

- A network interruption may suggest checking the connection.
- An expired session may require signing in again.
- A validation error may identify a field that needs correction.
- A temporary server failure may offer a retry.
- A forbidden action may explain that the user lacks permission.

Use the server's documented error contract to select a message. If the response contains field-level validation errors, map only the expected fields and messages into the form; treat response data as untrusted input.

## 5. Decide what happens to old data during refresh

When a user reloads a view, choose deliberately whether to clear the current data or keep it visible while fresh data loads.

- Clear-then-load is simple, but can cause a blank screen during every refresh.
- Keep-then-refresh avoids a blank screen, but the UI should say that the displayed data may be stale.
- If refresh fails while old data remains, show a refresh error without pretending the old data is the newest result.

One explicit model might add refreshing and refreshError states that carry the previous data:

~~~ts
type RefreshState<T> =
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'empty' }
  | { status: 'refreshing'; data: T }
  | { status: 'refreshError'; data: T; message: string }
  | { status: 'error'; message: string };
~~~

Do not silently render old data under a success label if the user needs to know it may be stale.

## 6. Keep later retry actions working

If a user can trigger repeated loads, catch each request error inside the request branch. That way one failure becomes an error state without permanently ending the outer reload stream.

~~~ts
reloadClicks$.pipe(
  startWith(void 0),
  switchMap(() =>
    this.http.get<Product[]>('/api/products').pipe(
      map((products): LoadState<Product[]> =>
        products.length === 0
          ? { status: 'empty' }
          : { status: 'success', data: products }
      ),
      startWith({ status: 'loading' } as LoadState<Product[]>),
      catchError((error: unknown) =>
        of<LoadState<Product[]>>({
          status: 'error',
          message: getSafeMessage(error),
          retryable: true
        })
      )
    )
  )
);
~~~

If catchError is placed outside switchMap and replaces the entire stream with a fallback, future click values may no longer start requests. The inner catchError keeps the trigger stream alive.

## 7. Make retry intentional

A retry button should call the same load action and show a fresh loading state. Prevent accidental duplicate submissions when they could create duplicate mutations. For GET reads, a retry is commonly safe; for state-changing requests, follow the API's idempotency contract.

Avoid infinite retries that keep the view in a perpetual loading state. Give users clear feedback if the retry fails again.

## Common mistakes

### Using one boolean for every possible state

A loading flag cannot distinguish empty success from failure. Use a state union or another explicit state model.

### Showing an empty state after every error

This falsely tells users the request succeeded with no results. Render an error state separately.

### Rendering every branch at once

Keep conditionals mutually exclusive so loading, empty, success, and error content cannot conflict.

### Showing raw backend errors

Backend payloads can contain implementation details or unsafe text. Map them to intentional user-facing messages.

### Hiding stale data without a product decision

Decide whether refresh clears old content or labels it as stale. Avoid showing stale content as if it were the new result.

### Letting one error stop future retry actions

For a stream of refresh triggers, catch the request error inside the request branch so later triggers are still handled.

## Key idea

Model each meaningful asynchronous outcome explicitly. A successful empty response, an error, and a request in progress are different states with different UI. Use an accessible template for each state and make retry and stale-data behavior deliberate.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="093-caching-revalidation-and-transfer-cache.md">093 - Caching, Revalidation, and Transfer Cache</a></td><td><a href="../qnas/094-loading-empty-success-and-error-ui-states.md">Q&amp;A</a></td><td><a href="../examples/094-loading-empty-success-and-error-ui-states.md">Example</a></td><td><a href="095-angular-resources-and-signal-based-asynchronous-data.md">095 - Angular Resources and Signal-Based Asynchronous Data</a></td>
</tr>
  </tbody>
</table>
