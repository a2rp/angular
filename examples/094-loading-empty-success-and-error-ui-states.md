# Loading, Empty, Success, and Error UI States: Examples

This standalone product-list feature turns one HttpClient request into explicit UI states. The template can render idle/loading/success/empty/error separately and offer a retry only when the example policy marks the error retryable.

## 1. Define the state and error-message helpers

~~~ts
// product-list.state.ts
import { HttpErrorResponse } from '@angular/common/http';

export interface Product {
  id: string;
  name: string;
  price: number;
}

export type ProductListState =
  | { status: 'loading' }
  | { status: 'success'; products: Product[] }
  | { status: 'empty' }
  | { status: 'error'; message: string; retryable: boolean };

export function isRetryable(error: unknown): boolean {
  return error instanceof HttpErrorResponse
    && (
      error.status === 0
      || error.status === 429
      || error.status >= 500
    );
}

export function toProductErrorMessage(error: unknown): string {
  if (!(error instanceof HttpErrorResponse)) {
    return 'Products could not be loaded. Please try again.';
  }

  if (error.status === 0) {
    return 'The service could not be reached. Check your connection and try again.';
  }

  if (error.status === 401) {
    return 'Your session may have expired. Sign in and try again.';
  }

  if (error.status === 403) {
    return 'You do not have permission to view these products.';
  }

  if (error.status === 429) {
    return 'Too many requests. Wait a moment before trying again.';
  }

  if (error.status >= 500) {
    return 'The service is temporarily unavailable. Try again later.';
  }

  return 'Products could not be loaded. Please try again.';
}
~~~

Error bodies are backend-controlled. This helper selects safe text from stable status information instead of rendering the raw response.

## 2. Build a request stream with retry action

~~~ts
// product-list.component.ts
import { AsyncPipe, CurrencyPipe } from '@angular/common';
import { HttpClient } from '@angular/common/http';
import { Component, inject } from '@angular/core';
import type { Observable } from 'rxjs';
import {
  Subject,
  catchError,
  map,
  of,
  startWith,
  switchMap
} from 'rxjs';
import { isRetryable, toProductErrorMessage } from './product-list.state';
import type { Product, ProductListState } from './product-list.state';

@Component({
  selector: 'app-product-list',
  standalone: true,
  imports: [AsyncPipe, CurrencyPipe],
  templateUrl: './product-list.component.html'
})
export class ProductListComponent {
  private readonly http = inject(HttpClient);
  private readonly reloadRequests = new Subject<void>();

  readonly state$: Observable<ProductListState> = this.reloadRequests.pipe(
    startWith(void 0),
    switchMap(() =>
      this.http.get<Product[]>('/api/products').pipe(
        map((products): ProductListState =>
          products.length === 0
            ? { status: 'empty' }
            : { status: 'success', products }
        ),
        startWith({ status: 'loading' } as ProductListState),
        catchError((error: unknown) =>
          of<ProductListState>({
            status: 'error',
            message: toProductErrorMessage(error),
            retryable: isRetryable(error)
          })
        )
      )
    )
  );

  reload(): void {
    this.reloadRequests.next();
  }
}
~~~

The first startWith triggers the initial request. Each reload click creates a new request. switchMap replaces any previous request stream, which is useful when an older read should no longer determine the view.

The inner catchError converts one request failure to an error state while keeping the reloadRequests stream available for future attempts.

## 3. Render each state in the template

~~~html
<!-- product-list.component.html -->
@if (state$ | async; as state) {
  <section
    aria-labelledby="products-heading"
    [attr.aria-busy]="state.status === 'loading'">

    <h1 id="products-heading">Products</h1>

    @switch (state.status) {
      @case ('loading') {
        <p role="status">Loading products…</p>
      }

      @case ('success') {
        <ul>
          @for (product of state.products; track product.id) {
            <li>
              <span>{{ product.name }}</span>
              <span>{{ product.price | currency }}</span>
            </li>
          }
        </ul>
      }

      @case ('empty') {
        <p>No products are available yet.</p>
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

role=status is appropriate for a routine loading announcement. role=alert makes an important error announcement available to assistive technology. Keep focus where it is unless the interaction requires the user to move to a correction.

## 4. See each possible outcome

The endpoint can produce different UI states:

| API result | Stream state | What the user sees |
| --- | --- | --- |
| Request is in progress | loading | A loading status |
| 200 with one or more products | success | The product list |
| 200 with an empty array | empty | A successful empty message |
| Network or server failure | error | A safe message and, when allowed, a retry button |
| Retry button clicked | loading | A new request starts |

An empty array is not an error. A failed HTTP request is not an empty array.

## 5. Preserve previous data while refreshing

Some views should keep old content visible while refreshing. In that case, use a separate refreshing state that explicitly carries the previous data:

~~~ts
type ProductRefreshState =
  | { status: 'loading' }
  | { status: 'success'; products: Product[] }
  | { status: 'empty' }
  | { status: 'refreshing'; products: Product[] }
  | { status: 'refreshError'; products: Product[]; message: string }
  | { status: 'error'; message: string };
~~~

The template can keep rendering products for refreshing and refreshError, while also showing a non-blocking status that the data may be stale. Do not show retained data under a normal success state if the user needs to know that a refresh failed.

## 6. Keep form validation separate from page-load state

A form has its own states and field errors. Do not collapse request loading, form validity, and server validation into one unrelated boolean.

~~~ts
interface FormSubmissionState {
  status: 'idle' | 'submitting' | 'success' | 'error';
  message?: string;
  fieldErrors?: Record<string, string>;
}
~~~

Map only documented server validation fields to form controls. Unknown fields should not be used as arbitrary HTML or user-facing text.

## 7. Manual review checklist

Without changing the API, review the UI by considering these responses:

- A non-empty product array should render the success branch.
- An empty array should render the empty branch.
- A network failure should render the error branch with a connection message.
- A 401 or 403 should render the corresponding safe access message.
- A retryable failure should show the retry control.
- A non-retryable failure should not offer a misleading retry.
- Clicking retry should show loading and allow the next outcome to render.
- Loading and error announcements should not unexpectedly move keyboard focus.

## Common mistakes

### Returning an empty array from catchError

This makes a failure look like a successful empty result. Return a typed error state instead.

### Keeping old and new requests active without a policy

For a latest-view-wins read, switchMap cancels the previous subscription when a new reload trigger arrives. For operations where every action must complete, choose an operator that preserves those operations.

### Letting one error disable retry forever

Catch the request error inside the switchMap branch, so the outer reload trigger remains active.

### Marking every error retryable

Some errors need a different action, such as signing in again or correcting input. Base retryability on the API contract and operation.

### Announcing every state assertively

Routine loading progress should not interrupt users. Use a polite status for progress and reserve alert behavior for important errors.

## Key idea

Represent the request lifecycle as typed, mutually exclusive states. Render loading, successful data, empty success, and failure separately; keep retry and stale-data behavior intentional and accessible.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="093-caching-revalidation-and-transfer-cache.md">093 - Caching, Revalidation, and Transfer Cache</a></td><td><a href="../lessons/094-loading-empty-success-and-error-ui-states.md">Lesson</a></td><td><a href="../qnas/094-loading-empty-success-and-error-ui-states.md">Q&amp;A</a></td><td><a href="095-angular-resources-and-signal-based-asynchronous-data.md">095 - Angular Resources and Signal-Based Asynchronous Data</a></td>
</tr>
  </tbody>
</table>
