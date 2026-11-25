# Angular Resources and Signal-Based Asynchronous Data

Signals such as signal and computed hold synchronous values. Server data arrives later, so a component also needs a way to represent the request, its current value, loading, and failure. Angular resources connect that asynchronous work to signal-based UI.

This chapter uses the current Angular v22 resource APIs. Check the Angular version used by your project: in earlier Angular releases, some resource APIs were experimental or unavailable.

## Learning outcomes

After this lesson, you should be able to:

- Explain what problem a Resource solves and when it fits.
- Build a resource with reactive params and an asynchronous loader.
- Pause a request by returning undefined from params.
- Read resource status, value, error, and loading state safely.
- Use cancellation and reload for read operations.
- Choose between resource, httpResource, rxResource, and HttpClient.
- Avoid using cancellable resources for mutations.
- Consider runtime response validation and SSR data privacy.

## 1. Why use a Resource?

A signal is synchronous: reading it immediately returns its current value. A promise or Observable represents work that may finish later. A Resource joins these ideas: it runs asynchronous work and exposes its result and request state through signals.

A Resource is useful when the view should react to a changing input such as a selected user ID, search term, or route parameter. The Resource tracks the signals read in its params function. When those values change, it starts work for the new params and exposes the new request state.

A Resource does not replace every asynchronous API. Use it when the UI needs a signal-friendly representation of a read. Use HttpClient directly for writes, progress-event workflows, or request orchestration that needs explicit RxJS operators.

## 2. Choose the appropriate API

| API | Best fit | Data source |
| --- | --- | --- |
| resource | A one-result asynchronous read with a custom loader | Promise or async function |
| httpResource | A reactive HTTP read that should use Angular's HTTP stack | HttpClient request |
| rxResource | A resource-shaped interface over an existing Observable API | RxJS Observable |
| HttpClient directly | Mutations, progress events, and explicit request workflows | HttpClient Observable |
| A live stream API | Values that keep changing after the initial response | Resource stream, rxResource, or an application-specific Observable |

Current Angular documentation marks resource, httpResource, and rxResource stable since Angular v22.0. Confirm the API status for the exact Angular version in your application before adopting it.

## 3. Create a resource with params and loader

The resource function accepts a params computation and an asynchronous loader. The loader runs when the params value changes. In this example, selecting a user ID causes a new read:

~~~ts
import { Component, resource, signal } from '@angular/core';

interface User {
  id: string;
  displayName: string;
}

function parseUser(value: unknown): User {
  if (typeof value !== 'object' || value === null) {
    throw new Error('The user response is invalid.');
  }

  const record = value as Record<string, unknown>;
  const id = record['id'];
  const displayName = record['displayName'];

  if (typeof id !== 'string' || typeof displayName !== 'string') {
    throw new Error('The user response is invalid.');
  }

  return { id, displayName };
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
  selector: 'app-user-card',
  standalone: true,
  templateUrl: './user-card.component.html'
})
export class UserCardComponent {
  readonly userId = signal<string | undefined>('user-42');

  readonly user = resource<User, string>({
    params: () => this.userId(),
    loader: ({ params, abortSignal }) => loadUser(params, abortSignal)
  });

  selectUser(id: string): void {
    this.userId.set(id);
  }

  clearSelection(): void {
    this.userId.set(undefined);
  }

  reload(): void {
    this.user.reload();
  }
}
~~~

The type arguments describe the loaded value and the params value. The type of user.value() is User or undefined until a value is available. The parser treats the JSON body as unknown and checks the fields at runtime. A TypeScript generic alone does not validate data returned by a server.

The loader receives an object containing params, abortSignal, and previous status information. The loader should use params for the request it is loading. Avoid reading unrelated changing signals inside the loader; keep request dependencies explicit in params.

## 4. Reactive params and pausing work

The params function is reactive. Angular tracks signal reads made while evaluating it.

~~~ts
readonly searchTerm = signal('');

readonly search = resource({
  params: () => {
    const query = this.searchTerm().trim();
    return query.length >= 2 ? { query } : undefined;
  },
  loader: ({ params, abortSignal }) =>
    loadSearchResults(params.query, abortSignal)
});
~~~

Returning undefined means there is no valid request. The loader does not run and the resource enters the idle state. This is useful for a required selection, a search minimum length, or a feature that is not enabled yet.

Return undefined itself when the request should be paused. Returning an object such as { query: undefined } is still a defined params value and may start the loader.

## 5. Read resource state safely

A Resource exposes signal-based state:

| Property | Meaning |
| --- | --- |
| value() | The current value. It throws when the Resource is in the error state. |
| hasValue() | Whether it currently has a valid value; also narrows the TypeScript type. |
| status() | One of idle, loading, reloading, resolved, local, or error. |
| isLoading() | Whether a load or reload is in progress. |
| error() | The most recent loader error, if any. |
| reload() | Starts the loader again for the current params. |

Guard a value read with hasValue(). This prevents an error-state value read from throwing and lets TypeScript know the value is present.

~~~html
@if (user.hasValue()) {
  <article [attr.aria-busy]="user.isLoading()">
    @if (user.isLoading()) {
      <p role="status">Refreshing the profile...</p>
    }

    <h2>{{ user.value().displayName }}</h2>
    <p>User ID: {{ user.value().id }}</p>
  </article>
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

The order distinguishes an initial load from a refresh that still has a usable value. During a reload of the same params, the status is reloading and the previous value remains available. During a new params load, the old value should not be treated as the new user's data.

Use a generic user-facing message unless the API provides a documented, safe error contract. Do not render raw exception messages or arbitrary server response text.

### Resource statuses

| Status | Typical situation | Value behavior |
| --- | --- | --- |
| idle | Params are undefined; no loader runs | Undefined |
| loading | Params changed and a new load started | Undefined |
| reloading | reload() requested another read for the same params | Previous value remains available |
| resolved | The loader completed successfully | Resolved value |
| local | The value was changed with a writable Resource API | Locally set value |
| error | The loader failed | Reading value() throws |

A defaultValue can provide an initial value instead of undefined. Choose it carefully: an empty array used as a default may be a useful placeholder, but by itself it does not tell the UI whether the server successfully returned no results. Use status when that distinction matters.

## 6. Cancellation and stale requests

Angular aborts an in-progress resource load when its params change or the resource is destroyed. The loader receives an AbortSignal so that APIs that support cancellation can stop the underlying work. The fetch example passes this signal to fetch.

Cancellation helps prevent a slow response for an old selection from becoming the visible result for a newer selection. It does not undo server work that has already happened. A request can reach the server before the client aborts it, so this behavior is another reason to reserve resources for reads.

If a custom asynchronous API ignores AbortSignal, Angular can stop treating its result as the current load, but the underlying operation may continue consuming resources. Use an API that honors cancellation when available.

## 7. Use httpResource for reactive HTTP reads

httpResource is a reactive wrapper around HttpClient. It makes a GET request eagerly, updates when signals used to build the request change, exposes resource state signals, and uses Angular's HTTP stack, including configured interceptors.

~~~ts
import { httpResource } from '@angular/common/http';
import { Component, signal } from '@angular/core';

interface Product {
  id: string;
  name: string;
}

@Component({
  selector: 'app-product-search',
  standalone: true,
  templateUrl: './product-search.component.html'
})
export class ProductSearchComponent {
  readonly query = signal('');

  readonly products = httpResource<Product[]>(() => {
    const query = this.query().trim();

    if (query.length < 2) {
      return undefined;
    }

    return {
      url: '/api/products',
      params: { q: query }
    };
  });
}
~~~

The request factory returns undefined until the query has at least two characters, so no HTTP request is made before then. For normal JSON responses, httpResource parses JSON by default. Its parse option can validate or transform the raw response before the resource exposes it.

A generic type such as httpResource<Product[]>() is compile-time information; it does not prove that the server actually returned a Product array. Validate important API responses at runtime.

The template can render the request states:

~~~html
<label>
  Search products
  <input
    type="search"
    [value]="query()"
    (input)="query.set($any($event.target).value)">
</label>

@if (products.hasValue()) {
  @if (products.isLoading()) {
    <p role="status">Updating results...</p>
  }

  @if (products.value().length === 0) {
    <p>No products match this search.</p>
  } @else {
    <ul>
      @for (product of products.value(); track product.id) {
        <li>{{ product.name }}</li>
      }
    </ul>
  }
} @else if (products.isLoading()) {
  <p role="status">Loading products...</p>
} @else if (products.error()) {
  <p role="alert">Products could not be loaded.</p>
} @else {
  <p>Enter at least two characters to search.</p>
}
~~~

In a real component, prefer a typed event handler over $any when your project style requires strict template typing. The example keeps focus on how the resource state drives the UI.

Use provideHttpClient in application configuration when setting up or customizing HttpClient. If an application already provides it, httpResource uses that configured client.

## 8. Use rxResource with an Observable-based service

rxResource is useful when a service already exposes an Observable API and the view should consume its result through resource state signals. It uses a stream factory instead of a promise loader:

~~~ts
import { HttpClient } from '@angular/common/http';
import { Component, inject, signal } from '@angular/core';
import { rxResource } from '@angular/core/rxjs-interop';

interface Product {
  id: string;
  name: string;
}

@Component({
  selector: 'app-product-detail',
  standalone: true,
  templateUrl: './product-detail.component.html'
})
export class ProductDetailComponent {
  private readonly http = inject(HttpClient);
  readonly productId = signal('product-7');

  readonly product = rxResource({
    params: () => this.productId(),
    stream: ({ params }) =>
      this.http.get<Product>(
        '/api/products/' + encodeURIComponent(params)
      )
  });
}
~~~

The stream callback is called for each new params value. The returned Observable must emit a value or an error before it completes. An Observable that completes without either, such as EMPTY, cannot provide a resource value.

rxResource does not replace RxJS. Use RxJS operators for debounce, combining sources, retries, or workflows where every event matters. Use the resource state API when the result should be presented as the current signal-based value of a view.

## 9. Resources and continuously changing data

A promise loader returns one result per request. For data that keeps changing, Angular's resource API also has a streaming form whose stream loader returns a Promise of a Signal. rxResource can consume an Observable source. A service may also keep its Observable API directly when it needs richer backpressure, buffering, reconnect, or event-processing behavior.

The streaming loader returns a Signal<ResourceStreamItem<T>> or a Promise of one. Each signal value is a ResourceStreamItem: either { value: T } or { error: Error }. A resource can provide the latest value and status to the view, but it does not decide your reconnection policy, event ordering, message retention, or offline behavior. Those remain application decisions. The next lesson covers live connections in more depth.

## 10. SSR, caching, and sensitive data

A resource can use an id to transfer a resolved server-rendered value to the browser during hydration. The id must be unique in the application and identical on the server and client.

Transferred values are serialized into the HTML. Do not enable resource transfer caching for user-specific data when the rendered HTML might be shared or cached across users. Follow the application's authentication and cache policy, and review the SSR transfer-cache behavior for the HTTP request as well.

## Common mistakes

### Reading value() without a guard

The value signal throws while the resource is in an error state. Check hasValue() before reading it.

### Treating a default empty value as a successful response

A default value can exist while the request is still idle or loading. Use the status to distinguish a placeholder from a server-confirmed empty result.

### Putting request dependencies only inside the loader

The resource reacts to params. Build params from the signals that define the request so changes reliably start a new load.

### Returning an object that contains undefined to pause a request

Only an undefined params result pauses loading. An object with an undefined field is still a request value.

### Using a resource for POST, PUT, or DELETE

A changing request or destroyed resource can abort a load. Use HttpClient and an explicit action flow for mutations.

### Assuming cancellation reverses server-side work

AbortSignal can cancel supported client work, but it cannot guarantee that the server did not already receive or process the request.

### Trusting a TypeScript response type as runtime validation

HTTP JSON is untrusted input. Validate fields that the application depends on, either with a parser or a schema library.

### Using a promise loader for a long-lived stream

A one-result loader is a poor fit for continuous updates. Use a streaming resource, rxResource, or an Observable service according to the behavior required.

## Key idea

Resources make asynchronous reads reactive: params determine what to load, resource signals describe the current result, and cancellation helps keep the view aligned with the latest request. Use the right resource variant for the data source, guard value reads, and keep mutations in explicit HTTP workflows.

## Related references

- Q&A: qnas/095-angular-resources-and-signal-based-asynchronous-data.md
- Examples: examples/095-angular-resources-and-signal-based-asynchronous-data.md
- Angular async reactivity with resources: https://angular.dev/guide/signals/resource
- Angular resource API: https://angular.dev/api/core/resource`r`n- Angular streaming resource loader API: https://angular.dev/api/core/ResourceStreamingLoader
- Angular reactive data fetching with httpResource: https://angular.dev/guide/http/http-resource
- Angular httpResource API: https://angular.dev/api/common/http/httpResource
- Angular RxJS interop and rxResource: https://angular.dev/guide/signals/rxjs-interop
- Angular rxResource API: https://angular.dev/api/core/rxjs-interop/rxResource
- Angular version compatibility: https://angular.dev/reference/versions
