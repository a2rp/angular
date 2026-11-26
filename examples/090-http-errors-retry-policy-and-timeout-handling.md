# HTTP Errors, Retry Policy, and Timeout Handling: Examples

These examples build on lesson 090. They show a bounded retry policy for transient GET failures, per-attempt timeouts, safe error messages, and why a mutation needs an explicit idempotency strategy before it can be retried.

## 1. Add a bounded retry operator

Create a reusable operator for requests that are safe to repeat. This policy retries network or timeout failures, HTTP 408, and server errors. It does not retry ordinary client errors such as 400 or 404.

~~~ts
// retry-transient.operator.ts
import { HttpErrorResponse } from '@angular/common/http';
import {
  MonoTypeOperatorFunction,
  retry,
  throwError,
  timer
} from 'rxjs';

function isRetryable(error: unknown): error is HttpErrorResponse {
  return error instanceof HttpErrorResponse
    && (
      error.status === 0
      || error.status === 408
      || error.status >= 500
    );
}

export function retryTransient<T>(
  maxRetries = 2
): MonoTypeOperatorFunction<T> {
  return retry<T>({
    count: maxRetries,
    delay: (error, retryNumber) => {
      if (!isRetryable(error)) {
        return throwError(() => error);
      }

      const backoffMs = Math.min(
        300 * 2 ** (retryNumber - 1),
        5000
      );
      const jitterMs = Math.floor(Math.random() * 200);

      return timer(backoffMs + jitterMs);
    }
  });
}
~~~

The original request is not included in maxRetries. A value of 2 allows up to three attempts: the initial request and two retries. The delay grows between attempts and includes a small random jitter to reduce synchronized retry bursts.

This policy deliberately leaves HTTP 429 out. If an API returns 429, follow its contract and honor Retry-After where provided instead of applying an unrelated fixed delay. The exact retryable statuses should be agreed with the API.

## 2. Use the policy for a GET request

Each retry resubscribes to the HttpClient Observable and sends another backend request. The timeout below applies to each attempt; it does not cap the combined time spent across all attempts and delays.

~~~ts
// product.service.ts
import { HttpClient } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { Observable } from 'rxjs';
import { retryTransient } from './retry-transient.operator';

export interface Product {
  id: number;
  name: string;
  price: number;
}

@Injectable({ providedIn: 'root' })
export class ProductService {
  private readonly http = inject(HttpClient);

  getProducts(): Observable<Product[]> {
    return this.http.get<Product[]>('/api/products', {
      timeout: 5000
    }).pipe(
      retryTransient(2)
    );
  }
}
~~~

With a 5-second timeout and two retries, the request flow can take longer than 15 seconds because retry delays and interceptor work add time. If the whole user operation needs a deadline, implement that separately at the application level.

## 3. Map technical errors to useful UI messages

Keep backend payloads and internal details out of user-facing messages. Use the status and other stable metadata to select a short action the user can take.

~~~ts
// http-error-message.ts
import { HttpErrorResponse } from '@angular/common/http';

export function toHttpErrorMessage(error: unknown): string {
  if (!(error instanceof HttpErrorResponse)) {
    return 'Something went wrong. Please try again.';
  }

  if (error.status === 0) {
    return 'The server could not be reached. Check your connection and try again.';
  }

  if (error.status === 401) {
    return 'Your session may have expired. Sign in and try again.';
  }

  if (error.status === 403) {
    return 'You do not have permission to perform this action.';
  }

  if (error.status === 404) {
    return 'The requested item could not be found.';
  }

  if (error.status === 429) {
    return 'Too many requests. Wait a moment before trying again.';
  }

  if (error.status >= 500) {
    return 'The service is temporarily unavailable. Please try again later.';
  }

  return 'The request could not be completed. Check the information and try again.';
}
~~~

A status of 0 can mean a network-level problem or a timeout, so the client may not be able to tell which happened. Avoid displaying raw stack traces, server response bodies, or internal URLs to users.

## 4. Handle an error at the page boundary

This example keeps the service focused on making the request. The page decides what fallback to display. Returning an empty array keeps the Observable alive with a usable value, so keep a separate error state to avoid making failure look like a successful empty result.

~~~ts
// product-list.component.ts
import { Component, inject, signal } from '@angular/core';
import { catchError, of } from 'rxjs';
import { Product, ProductService } from './product.service';
import { toHttpErrorMessage } from './http-error-message';

@Component({
  selector: 'app-product-list',
  templateUrl: './product-list.component.html'
})
export class ProductListComponent {
  private readonly productService = inject(ProductService);

  readonly products = signal<Product[] | null>(null);
  readonly errorMessage = signal('');

  loadProducts(): void {
    this.errorMessage.set('');

    this.productService.getProducts().pipe(
      catchError((error: unknown) => {
        this.errorMessage.set(toHttpErrorMessage(error));
        return of([] as Product[]);
      })
    ).subscribe((products) => {
      this.products.set(products);
    });
  }
}
~~~

~~~html
<!-- product-list.component.html -->
@if (errorMessage()) {
  <p role="alert">{{ errorMessage() }}</p>
}

@if (products(); as items) {
  @for (product of items; track product.id) {
    <p>{{ product.name }} - {{ product.price }}</p>
  }
}
~~~

The example uses Angular's built-in control-flow syntax. In a project using a different Angular version, use the template syntax supported by that project. In production UI, also consider loading state, cancellation, and whether an empty fallback is appropriate.

If the error should continue to a higher-level handler, rethrow it instead of replacing it with a fallback:

~~~ts
import { catchError, throwError } from 'rxjs';

return this.productService.getProducts().pipe(
  catchError((error: unknown) => {
    this.logger.error('Product request failed', error);
    return throwError(() => error);
  })
);
~~~

## 5. Do not blindly retry a POST

A timeout does not prove that the server did not process a request. For example, an order may have been created successfully while the response was lost. Repeating the POST could create a second order.

~~~ts
// order.service.ts
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { Observable } from 'rxjs';

export interface NewOrder {
  productId: number;
  quantity: number;
}

export interface Order {
  id: string;
  productId: number;
  quantity: number;
}

@Injectable({ providedIn: 'root' })
export class OrderService {
  private readonly http = inject(HttpClient);

  createOrder(order: NewOrder, idempotencyKey: string): Observable<Order> {
    const headers = new HttpHeaders({
      'Idempotency-Key': idempotencyKey
    });

    return this.http.post<Order>('/api/orders', order, {
      headers,
      timeout: 8000
    });
  }
}
~~~

Only send an idempotency key if the server explicitly supports it and documents its behavior. The server must recognize repeated requests with the same key and return the original result instead of applying the mutation again. The client alone cannot make a POST idempotent.

## 6. Inspect Retry-After for a rate-limited response

Some APIs use HTTP 429 or 503 with a Retry-After header. Its value can be a number of seconds or an HTTP date. This helper parses either form; production code should also apply a maximum wait and follow the API's documented behavior.

~~~ts
import { HttpErrorResponse } from '@angular/common/http';

export function getRetryAfterMs(
  error: HttpErrorResponse,
  now = Date.now()
): number | null {
  const value = error.headers.get('Retry-After');

  if (!value) {
    return null;
  }

  const seconds = Number(value);

  if (Number.isFinite(seconds) && seconds >= 0) {
    return seconds * 1000;
  }

  const retryAt = Date.parse(value);

  if (Number.isNaN(retryAt)) {
    return null;
  }

  return Math.max(0, retryAt - now);
}
~~~

Do not automatically retry forever when a header is absent or malformed. A retry count, maximum delay, cancellation path, and user feedback still matter.

## 7. Example request timeline

For a GET with a 5-second timeout and two retries, a possible sequence is:

| Attempt | Result | Next step |
| --- | --- | --- |
| Initial request | Network failure after 1 second | Wait for the first backoff, then retry |
| Retry 1 | HTTP 503 after 2 seconds | Wait for the second backoff, then retry |
| Retry 2 | HTTP 200 after 1 second | Emit the successful response |
| Any attempt | HTTP 404 | Do not retry; send the error to the caller |

If all permitted attempts fail, the final error reaches the next operator or subscriber. Put catchError after retry when the fallback should run once after retries are exhausted. Put it before retry only when converting the error to a normal value is intentional; a recovered value does not trigger another retry.

## Checklist

- Set a timeout appropriate for the operation.
- Remember that the timeout applies to each backend attempt.
- Retry only a small, finite number of times.
- Retry only failures that the API treats as transient.
- Use exponential backoff and consider jitter.
- Respect Retry-After when the API contract requires it.
- Do not retry mutations unless they are idempotent or the server supports idempotency keys.
- Keep technical details out of end-user messages.
- Distinguish a fallback state from a successful empty result.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="089-functional-interceptors-and-request-context.md">089 - Functional Interceptors and Request Context</a></td><td><a href="../lessons/090-http-errors-retry-policy-and-timeout-handling.md">Lesson</a></td><td><a href="../qnas/090-http-errors-retry-policy-and-timeout-handling.md">Q&amp;A</a></td><td><a href="091-cancel-requests-and-prevent-stale-response-races.md">091 - Cancel Requests and Prevent Stale Response Races</a></td>
</tr>
  </tbody>
</table>
