# Build Typed HTTP Requests and API Services — Q&A

## Typed data and request methods

### 1. What does HttpClient's generic type argument do?

A generic such as get<Product>() tells TypeScript to treat the response body as Product. It improves editor support and compile-time checking, but it is not runtime validation. Angular does not compare the returned JSON with the Product interface.

### 2. What type should I use when the response shape is unknown?

Use unknown and validate or transform the value before treating it as a trusted application type. Avoid any unless there is a specific, justified reason to opt out of type checking.

### 3. What is the difference between a response interface and a request type?

A response interface describes what the application expects to receive. A request type describes the fields the API accepts for an operation. A create request may omit a server-generated id, while an update request may allow only a subset of fields.

### 4. Should I use the same type for every operation on a resource?

Only when the API contract truly uses the same shape. Separate types for create, update, list, and response bodies make differences explicit and prevent accidentally requiring server-generated fields in client input.

### 5. Which HttpClient method should I use?

Use the method that matches the API operation: get for retrieval, post for creation or an API-defined operation, put for replacement, patch for partial update, and delete for deletion. Confirm the backend contract; method names alone do not specify every server behavior.

### 6. Does PATCH always mean that every field is optional?

No. PATCH commonly represents a partial update, but the exact accepted document and field semantics depend on the API. Define the TypeScript request type to match the server's contract.

### 7. What is the request body in a post, put, or patch call?

It is the second argument in the common convenience method signatures, for example post<T>(url, body). The backend defines the accepted fields, encoding, and response shape.

### 8. Does delete<T>() tell the server what type to delete?

No. The generic describes the expected response body. It does not alter the server's deletion semantics or request payload.

## Observables and subscriptions

### 9. When is an HttpClient request sent?

HttpClient returns a cold Observable. Creating the Observable does not send a request; subscribing starts it.

### 10. Does subscribing twice send two requests?

Normally yes. Each subscription to a cold HttpClient Observable makes an independent request. If the result should be shared, choose an explicit sharing or caching strategy instead of assuming the Observable is cached.

### 11. Should an API service subscribe to its own request?

Usually no. Return the Observable and let the caller choose how to compose or consume it. A service may subscribe when it deliberately owns a long-lived process, but that is a different responsibility and requires clear lifecycle handling.

### 12. What is the difference between subscribe and the async pipe?

subscribe is imperative: code handles next, error, and completion callbacks. The async pipe consumes an Observable in a template and manages its subscription with the view lifecycle. Choose based on whether the operation is view data or requires an imperative side effect.

### 13. Does unsubscribing undo a server operation?

Unsubscribing can abort an in-flight client request when the backend supports cancellation. It does not guarantee that a server which already received the request has rolled back its work. Treat cancellation as stopping the client from waiting or continuing, not as a transaction rollback.

### 14. Why should I avoid subscribing multiple times in a component and template?

The component subscription and the template async pipe are separate subscriptions. With a cold HttpClient Observable, both can send separate requests. Share the result deliberately or expose it in a way that has one intended subscriber.

## API service design

### 15. What belongs in an API service?

Endpoint construction, HttpClient calls, API request/response types, and transformations that belong to the API boundary are common responsibilities. UI-specific notifications, dialogs, and page navigation usually belong in a feature layer.

### 16. Should the API service return Observable or Promise?

HttpClient returns Observable, which composes naturally with RxJS and Angular templates. Convert to a Promise only when the caller specifically needs Promise-based control flow, for example with firstValueFrom.

### 17. Should an API service catch every error?

Not necessarily. Let errors propagate when the caller needs to choose the UI response or recovery. Translate an error in the service when that translation is a stable part of its public contract. Avoid turning all errors into success-shaped empty data.

### 18. Should components build endpoint URLs?

A small application may start that way, but a focused API service keeps endpoint paths and request methods consistent. Components can focus on user actions and rendering rather than transport details.

### 19. Is getById implemented by downloading the full collection and searching it?

Usually a backend exposes a direct item endpoint such as /api/products/42. Call that endpoint when available. Downloading the full collection to find one item is wasteful unless that is genuinely the API contract or a deliberate client-side cache strategy.

### 20. How should I build an endpoint URL from an id?

Use the endpoint structure defined by the API, for example baseUrl + '/' + id. URL encoding and query parameters require care; use Angular's URL parameter APIs for query values, which are covered in the next lesson.

## Response handling and testing

### 21. Does the generic type select text, Blob, or ArrayBuffer responses?

No. The generic describes a JSON body type. Non-JSON responses use the responseType option, such as text, blob, or arraybuffer. The option affects the return type and must retain a literal type when stored in a variable.

### 22. How do I access status and response headers?

By default, HttpClient returns the response body. Use observe: 'response' when the full HttpResponse is needed. The observe option changes the return type and should remain a literal value in extracted options.

### 23. Where should I test an API service?

Use Angular's HTTP testing utilities to capture requests, assert method and URL, and flush controlled responses. Do not depend on a real network for an isolated service test. Detailed HTTP testing patterns are covered in a later lesson.

### 24. How do I know the server's JSON really matches my TypeScript interface?

The generic alone cannot establish that. Validate at runtime when the application needs stronger guarantees, especially at external data boundaries. Validation may be a small explicit function or a runtime schema tool.

## Quick reference

| Task | Typical call |
| --- | --- |
| Read a list | http.get<Item[]>('/api/items') |
| Read one item | http.get<Item>('/api/items/' + id) |
| Create a resource | http.post<Item>('/api/items', input) |
| Replace a resource | http.put<Item>('/api/items/' + id, input) |
| Partially update | http.patch<Item>('/api/items/' + id, changes) |
| Delete a resource | http.delete<void>('/api/items/' + id) |

The response type should match the actual API response. In particular, do not assume a delete endpoint returns no content unless the backend contract says so.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="085-configure-httpclient-with-providehttpclient.md">085 - Configure HttpClient with provideHttpClient</a></td><td><a href="../lessons/086-build-typed-http-requests-and-api-services.md">Lesson</a></td><td><a href="../examples/086-build-typed-http-requests-and-api-services.md">Example</a></td><td><a href="087-url-parameters-headers-response-types-and-credentials.md">087 - URL Parameters, Headers, Response Types, and Credentials</a></td>
</tr>
  </tbody>
</table>
