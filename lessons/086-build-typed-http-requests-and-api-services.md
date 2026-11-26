# Build Typed HTTP Requests and API Services

Angular's HttpClient offers methods for common HTTP operations such as GET, POST, PUT, PATCH, and DELETE. Each method returns an RxJS Observable. A TypeScript type argument can describe the expected JSON body, and an API service can keep endpoint details out of components.

Types make application code easier to read and help catch mistakes during development. They do not prove that a remote server actually sent data matching the declared type.

## Learning outcomes

After this lesson, you should be able to:

- Choose an HttpClient method for a common API operation.
- Describe a request body and expected response with TypeScript types.
- Build an injectable API service that returns Observables.
- Explain when an HTTP request is sent and why repeated subscriptions matter.
- Distinguish compile-time typing from runtime response validation.

## 1. HttpClient request methods

HttpClient methods map naturally to common HTTP methods:

| Angular method | Common use |
| --- | --- |
| get<T>() | Read a resource or collection |
| post<T>() | Create a resource or perform an operation |
| put<T>() | Replace a resource representation |
| patch<T>() | Partially update a resource |
| delete<T>() | Delete a resource or receive a deletion result |
| request<T>() | Use a method or configuration that does not fit a convenience method |

The exact API contract belongs to the server. HTTP method names suggest common semantics, but an application's backend may impose additional rules.

## 2. Type the data your application expects

Define an interface or type that describes the JSON shape the application intends to use:

~~~ts
export interface Product {
  id: number;
  name: string;
  price: number;
}
~~~

Use it as a generic argument when requesting a response body:

~~~ts
getProducts(): Observable<Product[]> {
  return this.http.get<Product[]>('/api/products');
}
~~~

The generic argument tells TypeScript to treat the returned body as Product[]. Angular does not inspect the JSON at runtime to confirm its fields or types. A malformed response can still cause runtime errors later.

When the response shape is uncertain, start from unknown and validate or map it before treating it as a trusted domain value. Avoid using any as a way to silence uncertainty.

## 3. Type request and response data separately

Creation and update requests do not always have the same shape as a persisted resource. Model each API contract clearly:

~~~ts
export interface CreateProductRequest {
  name: string;
  price: number;
}

export interface UpdateProductRequest {
  name?: string;
  price?: number;
}
~~~

A create operation sends one shape and may receive the newly created Product:

~~~ts
createProduct(input: CreateProductRequest): Observable<Product> {
  return this.http.post<Product>('/api/products', input);
}
~~~

A full replacement and a partial update have different intent:

~~~ts
replaceProduct(id: number, product: Product): Observable<Product> {
  return this.http.put<Product>('/api/products/' + id, product);
}

updateProduct(id: number, changes: UpdateProductRequest): Observable<Product> {
  return this.http.patch<Product>('/api/products/' + id, changes);
}
~~~

The server determines whether PUT replaces a full resource and which fields PATCH accepts. Use types that describe the actual API contract rather than assuming every backend follows the same convention.

## 4. Put API operations in an injectable service

An API service can centralize URL construction and request methods:

~~~ts
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { Observable } from 'rxjs';

export interface Product {
  id: number;
  name: string;
  price: number;
}

export interface CreateProductRequest {
  name: string;
  price: number;
}

export interface UpdateProductRequest {
  name?: string;
  price?: number;
}

@Injectable({ providedIn: 'root' })
export class ProductApiService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = '/api/products';

  getAll(): Observable<Product[]> {
    return this.http.get<Product[]>(this.baseUrl);
  }

  getById(id: number): Observable<Product> {
    return this.http.get<Product>(this.baseUrl + '/' + id);
  }

  create(input: CreateProductRequest): Observable<Product> {
    return this.http.post<Product>(this.baseUrl, input);
  }

  replace(id: number, product: Product): Observable<Product> {
    return this.http.put<Product>(this.baseUrl + '/' + id, product);
  }

  update(id: number, changes: UpdateProductRequest): Observable<Product> {
    return this.http.patch<Product>(this.baseUrl + '/' + id, changes);
  }

  remove(id: number): Observable<void> {
    return this.http.delete<void>(this.baseUrl + '/' + id);
  }
}
~~~

This method assumes the backend exposes a dedicated item endpoint. Use the endpoint contract your backend actually provides.

## 5. Return the Observable from the service

An API service generally returns the HttpClient Observable rather than subscribing internally:

~~~ts
getAll(): Observable<Product[]> {
  return this.http.get<Product[]>(this.baseUrl);
}
~~~

This lets the caller decide how to consume the result. It also keeps view state, navigation, and user feedback out of a low-level API service.

A component can subscribe directly when it must react imperatively:

~~~ts
this.productApi.getAll().subscribe({
  next: products => this.products.set(products),
  error: error => this.errorMessage.set('Could not load products.')
});
~~~

For template-driven display, an Observable exposed to the template can be consumed with Angular's async pipe. The pipe manages subscription lifetime as the view is created and destroyed.

## 6. HttpClient Observables are cold

Creating an HttpClient Observable does not send the request by itself. The request begins when something subscribes to it.

~~~ts
const products$ = this.http.get<Product[]>('/api/products');
// No request has been sent yet.

products$.subscribe();
// A request is sent.
~~~

Each independent subscription to a cold HttpClient Observable normally creates a separate backend request:

~~~ts
const products$ = this.http.get<Product[]>('/api/products');

products$.subscribe();
products$.subscribe(); // A second request is made.
~~~

Do not subscribe multiple times accidentally when the desired behavior is one request. Share or otherwise coordinate the stream only when the application actually needs that behavior; caching and replay policy should be explicit.

Unsubscribing from an in-flight request aborts the client-side request when supported by the backend. This is useful when a view no longer needs the result, but it does not guarantee that a server which already received the request has undone its work.

## 7. Responses are not runtime-validated by generics

This code:

~~~ts
this.http.get<Product>('/api/products/1')
~~~

does not parse and validate the server response against the Product interface. It types the result as Product for compile-time use. Interfaces are erased from JavaScript output and cannot validate JSON at runtime.

For external or unreliable data, request an unknown value and validate its shape at a boundary before using it:

~~~ts
this.http.get<unknown>('/api/products/1')
~~~

Validation may be implemented with explicit checks or a runtime schema library. The right choice depends on application requirements. Never treat a TypeScript generic as a security check or as proof that a server response is correct.

## 8. Keep service responsibilities clear

A focused API service can:

- Build endpoint URLs.
- Call HttpClient methods.
- Express request and response contracts.
- Return the resulting Observable.

A feature component can:

- Trigger an operation in response to user interaction.
- Present loading, success, and error states.
- Choose when to subscribe or expose an Observable to its template.

A service may also transform data when that transformation is part of its API boundary. Keep UI-specific behavior, such as opening dialogs or displaying toast notifications, outside a generic transport service.

## 9. Handle errors at the appropriate boundary

HttpClient reports request failures through the Observable error channel. The API service can let the error propagate, or translate it when that is a stable part of the service contract. The feature that knows what the user is doing is often best positioned to choose the message and recovery action.

Avoid converting every error to an empty list or a success-shaped value without a clear reason. That can make a failed request look like valid empty data. Detailed error and retry patterns are covered later.

## 10. Common mistakes

### Assuming the generic validates JSON

It does not. Treat remote responses as external data and validate them when runtime certainty is required.

### Subscribing inside every API method

Doing so hides control flow and makes callers unable to compose requests naturally. Return the Observable unless the service explicitly owns a long-lived process.

### Reusing one Observable and assuming it is cached

An HttpClient Observable is normally cold. Multiple subscriptions send multiple requests unless a deliberate sharing strategy changes that behavior.

### Using POST for every operation

Choose methods based on the API contract and intended operation. GET, POST, PUT, PATCH, and DELETE are not interchangeable merely because each can be represented as a request.

### Returning a fake success value after failure

Preserve the error or translate it deliberately. Do not make an unsuccessful request indistinguishable from valid data.

## Key idea

HttpClient gives Angular applications typed request methods that return Observables. Use request and response types to communicate the expected API contract, keep calls in focused injectable services, and remember that TypeScript typing does not validate remote JSON at runtime.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="085-configure-httpclient-with-providehttpclient.md">085 - Configure HttpClient with provideHttpClient</a></td><td><a href="../qnas/086-build-typed-http-requests-and-api-services.md">Q&amp;A</a></td><td><a href="../examples/086-build-typed-http-requests-and-api-services.md">Example</a></td><td><a href="087-url-parameters-headers-response-types-and-credentials.md">087 - URL Parameters, Headers, Response Types, and Credentials</a></td>
</tr>
  </tbody>
</table>
