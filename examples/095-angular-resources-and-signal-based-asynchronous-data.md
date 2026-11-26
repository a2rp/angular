# Angular Resources and Signal-Based Asynchronous Data: Examples

These examples show three ways to load read-only data into a signal-based view. Use each example independently in an Angular v22 project. Resource APIs had different stability and availability in earlier Angular releases; verify your project's version before copying them.

## 1. Load a selected profile with resource

This example has a signal that stores the selected user ID. Changing the selection starts a new request. Clearing it pauses the Resource.

### user-profile.component.ts

~~~ts
import { Component, resource, signal } from '@angular/core';

interface User {
  id: string;
  displayName: string;
  email: string;
}

function parseUser(value: unknown): User {
  if (typeof value !== 'object' || value === null) {
    throw new Error('The user response is invalid.');
  }

  const record = value as Record<string, unknown>;
  const id = record['id'];
  const displayName = record['displayName'];
  const email = record['email'];

  if (
    typeof id !== 'string'
    || typeof displayName !== 'string'
    || typeof email !== 'string'
  ) {
    throw new Error('The user response is invalid.');
  }

  return { id, displayName, email };
}

async function loadUser(id: string, abortSignal: AbortSignal): Promise<User> {
  const response = await fetch('/api/users/' + encodeURIComponent(id), {
    signal: abortSignal
  });

  if (!response.ok) {
    throw new Error('The user could not be loaded.');
  }

  const body: unknown = await response.json();
  return parseUser(body);
}

@Component({
  selector: 'app-user-profile',
  standalone: true,
  templateUrl: './user-profile.component.html'
})
export class UserProfileComponent {
  readonly selectedUserId = signal<string | undefined>('user-42');

  readonly user = resource<User, string>({
    params: () => this.selectedUserId(),
    loader: ({ params, abortSignal }) => loadUser(params, abortSignal)
  });

  selectUser(id: string): void {
    this.selectedUserId.set(id);
  }

  clearSelection(): void {
    this.selectedUserId.set(undefined);
  }

  reload(): void {
    this.user.reload();
  }
}
~~~

### user-profile.component.html

~~~html
<nav aria-label="Choose a user">
  <button type="button" (click)="selectUser('user-42')">User 42</button>
  <button type="button" (click)="selectUser('user-84')">User 84</button>
  <button type="button" (click)="clearSelection()">Clear selection</button>
</nav>

@if (user.hasValue()) {
  <section aria-labelledby="profile-heading" [attr.aria-busy]="user.isLoading()">
    @if (user.isLoading()) {
      <p role="status">Refreshing profile...</p>
    }

    <h1 id="profile-heading">{{ user.value().displayName }}</h1>
    <p>{{ user.value().email }}</p>
    <button type="button" (click)="reload()">Refresh profile</button>
  </section>
} @else if (user.isLoading()) {
  <p role="status">Loading profile...</p>
} @else if (user.error()) {
  <div role="alert">
    <p>The profile could not be loaded.</p>
    <button type="button" (click)="reload()">Try again</button>
  </div>
} @else {
  <p>Select a user to view the profile.</p>
}
~~~

### What to notice

- selectedUserId is read in params, so changing it triggers a new load.
- Clearing the selection returns undefined from params, so the resource becomes idle.
- abortSignal is passed to fetch. If the selection changes during a pending read, fetch can stop the old request.
- parseUser validates the data shape instead of trusting a compile-time type assertion.
- hasValue() guards every profile value read.
- reload() refreshes the current selection.

A native fetch loader may need an absolute URL or an injected base URL when the application renders on the server. If you need Angular interceptors or HttpClient testing support, use httpResource instead.

## 2. Search with httpResource

httpResource is useful when a reactive signal controls an HTTP GET request and the request should go through the configured Angular HTTP stack.

### product-search.component.ts

~~~ts
import { httpResource } from '@angular/common/http';
import { Component, signal } from '@angular/core';

interface Product {
  id: string;
  name: string;
  price: number;
}

function parseProducts(value: unknown): Product[] {
  if (!Array.isArray(value)) {
    throw new Error('The product response is invalid.');
  }

  return value.map((item): Product => {
    if (typeof item !== 'object' || item === null) {
      throw new Error('The product response is invalid.');
    }

    const record = item as Record<string, unknown>;
    const id = record['id'];
    const name = record['name'];
    const price = record['price'];

    if (
      typeof id !== 'string'
      || typeof name !== 'string'
      || typeof price !== 'number'
    ) {
      throw new Error('The product response is invalid.');
    }

    return { id, name, price };
  });
}

@Component({
  selector: 'app-product-search',
  standalone: true,
  templateUrl: './product-search.component.html'
})
export class ProductSearchComponent {
  readonly query = signal('');

  readonly products = httpResource<Product[]>(
    () => {
      const query = this.query().trim();

      if (query.length < 2) {
        return undefined;
      }

      return {
        url: '/api/products',
        params: { q: query }
      };
    },
    { parse: parseProducts }
  );

  updateQuery(event: Event): void {
    const target = event.target;

    if (target instanceof HTMLInputElement) {
      this.query.set(target.value);
    }
  }

  reload(): void {
    this.products.reload();
  }
}
~~~

If HttpClient is not already configured, add provideHttpClient() to the application providers:

~~~ts
import type { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient()]
};
~~~

Keep any interceptors and HTTP features in that same HttpClient configuration.

### product-search.component.html

~~~html
<label for="product-query">Search products</label>
<input
  id="product-query"
  type="search"
  [value]="query()"
  (input)="updateQuery($event)">

@if (products.hasValue()) {
  <section aria-labelledby="results-heading" [attr.aria-busy]="products.isLoading()">
    <h1 id="results-heading">Results</h1>

    @if (products.isLoading()) {
      <p role="status">Updating results...</p>
    }

    @if (products.value().length === 0) {
      <p>No products match this search.</p>
    } @else {
      <ul>
        @for (product of products.value(); track product.id) {
          <li>
            <strong>{{ product.name }}</strong>
            <span>{{ product.price }}</span>
          </li>
        }
      </ul>
    }

    <button type="button" (click)="reload()">Refresh results</button>
  </section>
} @else if (products.isLoading()) {
  <p role="status">Loading products...</p>
} @else if (products.error()) {
  <p role="alert">Products could not be loaded. Try again later.</p>
} @else {
  <p>Enter at least two characters to search.</p>
}
~~~

### What to notice

- A query shorter than two characters produces no HTTP request.
- When the query changes, httpResource issues a request for the new query and cancels an outstanding request when possible.
- The request runs through HttpClient, so configured interceptors apply.
- parseProducts validates the JSON before it becomes the resource value.
- A successful empty array is rendered as no matches; an HTTP or validation error is rendered as an error.
- This is a read-only GET. Use HttpClient directly for a submit action that changes server data.

## 3. Adapt an Observable service with rxResource

Use rxResource when the data layer already exposes Observables. It keeps the resource-style status and value signals in the component.

### product-api.service.ts

~~~ts
import { HttpClient } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import type { Observable } from 'rxjs';

export interface Product {
  id: string;
  name: string;
}

@Injectable({ providedIn: 'root' })
export class ProductApi {
  private readonly http = inject(HttpClient);

  getById(id: string): Observable<Product> {
    return this.http.get<Product>(
      '/api/products/' + encodeURIComponent(id)
    );
  }
}
~~~

### product-detail.component.ts

~~~ts
import { Component, inject, signal } from '@angular/core';
import { rxResource } from '@angular/core/rxjs-interop';
import { ProductApi } from './product-api.service';

@Component({
  selector: 'app-product-detail',
  standalone: true,
  templateUrl: './product-detail.component.html'
})
export class ProductDetailComponent {
  private readonly productApi = inject(ProductApi);

  readonly productId = signal('product-7');

  readonly product = rxResource({
    params: () => this.productId(),
    stream: ({ params }) => this.productApi.getById(params)
  });

  selectProduct(id: string): void {
    this.productId.set(id);
  }

  reload(): void {
    this.product.reload();
  }
}
~~~

### product-detail.component.html

~~~html
<nav aria-label="Choose a product">
  <button type="button" (click)="selectProduct('product-7')">Product 7</button>
  <button type="button" (click)="selectProduct('product-9')">Product 9</button>
</nav>

@if (product.hasValue()) {
  <article aria-labelledby="product-heading" [attr.aria-busy]="product.isLoading()">
    @if (product.isLoading()) {
      <p role="status">Refreshing product...</p>
    }

    <h1 id="product-heading">{{ product.value().name }}</h1>
    <p>Product ID: {{ product.value().id }}</p>
    <button type="button" (click)="reload()">Refresh product</button>
  </article>
} @else if (product.isLoading()) {
  <p role="status">Loading product...</p>
} @else if (product.error()) {
  <p role="alert">The product could not be loaded.</p>
} @else {
  <p>Select a product.</p>
}
~~~

This Observable must emit a value or an error before it completes. An Observable that completes empty cannot provide a resource value. Add retry, mapping, or other RxJS operators in the service or stream when the feature needs those policies.

## 4. Represent changing signal values with a streaming Resource

A promise loader resolves one value for a request. A streaming Resource instead observes a Signal of ResourceStreamItem values. The signal can publish a new value later.

~~~ts
import { Component, resource, signal } from '@angular/core';
import type { ResourceStreamItem } from '@angular/core';

@Component({
  selector: 'app-live-price',
  standalone: true,
  template: `
    @if (price.hasValue()) {
      <p>Latest price: {{ price.value() }}</p>
    } @else if (price.error()) {
      <p role="alert">Price updates are unavailable.</p>
    }
  `
})
export class LivePriceComponent {
  private readonly updates = signal<ResourceStreamItem<number>>({
    value: 0
  });

  readonly price = resource({
    stream: () => this.updates
  });

  publishPrice(nextPrice: number): void {
    this.updates.set({ value: nextPrice });
  }

  publishFailure(): void {
    this.updates.set({ error: new Error('Price feed failed.') });
  }
}
~~~

The escaped template markup above represents a normal component template string. In a real feature, an injected WebSocket or server-sent-event service could update the signal. Manage the connection lifecycle and reconnection behavior in that service. The next chapter covers live connections in detail.

## 5. Pick the tool from the data source

| Situation | Recommended starting point |
| --- | --- |
| One Promise-based read with custom loading logic | resource |
| Reactive HTTP GET that needs interceptors | httpResource |
| Existing service returns an Observable | rxResource |
| A form submits a mutation | HttpClient directly |
| Every event must be processed with custom stream policy | RxJS service or a carefully designed stream resource |

Do not choose a Resource only because a method name resembles the desired feature. Consider whether cancellation, eager requests, mutations, validation, and stream behavior fit the task.

## Common mistakes

### Using a made-up API response without validation

A generic response type does not check the actual JSON. Treat decoded JSON as unknown and validate the fields the UI relies on.

### Reading the value before checking hasValue()

The value may be undefined or may throw in an error state. Guard reads with hasValue().

### Sending a request for invalid input

Return undefined from the reactive request function until the input is valid.

### Using a Resource for a mutation

A changed request can cancel an in-progress load. Send user-triggered mutations through HttpClient and handle their result as an explicit action.

### Assuming an Observable always provides a value

rxResource needs the stream to emit or error before completion. Avoid empty completion unless it is converted into a meaningful value or error.

### Leaving live connections open

A Resource exposes state; it does not define the whole socket lifecycle. A service must release connections and define reconnect behavior.

## Key idea

Use resource for custom promise reads, httpResource for reactive HttpClient reads, and rxResource for Observable-backed data. In every case, make params explicit, validate external data where needed, and render loading, empty, success, and error states deliberately.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="094-loading-empty-success-and-error-ui-states.md">094 - Loading, Empty, Success, and Error UI States</a></td><td><a href="../lessons/095-angular-resources-and-signal-based-asynchronous-data.md">Lesson</a></td><td><a href="../qnas/095-angular-resources-and-signal-based-asynchronous-data.md">Q&amp;A</a></td><td><a href="096-server-sent-events-websockets-and-live-updates.md">096 - Server-Sent Events, WebSockets, and Live Updates</a></td>
</tr>
  </tbody>
</table>
