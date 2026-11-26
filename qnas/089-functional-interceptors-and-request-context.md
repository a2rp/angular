# Functional Interceptors and Request Context - Q&A

## Interceptors

### 1. What is an HttpClient interceptor?

An interceptor is middleware for requests made through HttpClient. It can inspect or clone an outgoing request, call the next handler, and transform the resulting HTTP event stream.

### 2. How do I activate a functional interceptor?

Register it with withInterceptors inside provideHttpClient:

~~~ts
provideHttpClient(
  withInterceptors([loggingInterceptor])
)
~~~

Defining or importing the function without registering it does not place it in the HTTP pipeline.

### 3. In what order do interceptors run?

Outgoing requests pass through the configured functional interceptors in the listed order. The response event stream returns through the chain in reverse order.

### 4. What does next(req) do?

It passes the request to the next interceptor in the chain, or to the backend if no interceptor remains. The returned Observable carries HTTP events back to the caller and through response transformations.

### 5. Does every interceptor have to call next?

Most do, but Angular allows advanced interceptors to synthesize a response or use another source without forwarding to the backend. If an interceptor short-circuits, later interceptors and the backend do not process that request.

### 6. Can an interceptor inspect the final response?

Yes. Apply RxJS operators to the Observable returned by next. For an event stream, check event.type because it can include progress and other events before the final HttpEventType.Response.

### 7. Why are functional interceptors recommended?

Angular recommends them because their ordering is more predictable, especially in complex configurations. Existing class-based interceptors can still be enabled through the DI-based compatibility setup.

### 8. How do I configure class-based interceptors?

Register provideHttpClient(withInterceptorsFromDi()) and provide the classes using the HTTP_INTERCEPTORS multi-provider. For new code, consider functional interceptors with withInterceptors.

## Request immutability and injection

### 9. Can an interceptor directly change HttpRequest headers?

No. HttpRequest and HttpHeaders are immutable. Clone the request and set the changed headers on the clone.

### 10. Does req.clone deeply copy the request body?

No. The body can contain a mutable object and is not protected from deep mutations. Avoid changing an object body in place; construct a new value so retries or repeated processing do not observe accidental mutations.

### 11. Can an interceptor inject a service?

Yes. A functional interceptor runs in the injection context of the injector that registered it, so it can call inject while the interceptor function is executing.

### 12. Can I call inject inside a later timer or arbitrary Observable callback?

Do not rely on that. Resolve the dependency synchronously while the interceptor is executing, capture the service or needed value, and use it in later callbacks.

### 13. Does a global auth interceptor safely attach a token to all requests?

No. Restrict authorization headers to the intended API origin or path. A token should not be sent to third-party or user-controlled URLs.

## Request context

### 14. What is HttpContext for?

HttpContext is a typed client-side metadata map attached to an HTTP request. Interceptors can use it to decide how to process that request, such as whether it should display a global loading indicator.

### 15. Is HttpContext sent to the server?

No. Context values are not automatically serialized into a query parameter, header, or request body. They remain metadata inside Angular's HTTP pipeline.

### 16. How does HttpContextToken provide type safety?

Create a HttpContextToken<T> and use that token with HttpContext.set and HttpContext.get. The token's generic type tells TypeScript what value is expected.

### 17. Why does the token constructor take a function?

The function supplies the default value for a request that does not explicitly set the token. It also allows object or array defaults to be created separately for each request.

### 18. How do I override a context value for one request?

Create or update the request's HttpContext:

~~~ts
const context = new HttpContext()
  .set(SKIP_GLOBAL_LOADING, true);

return this.http.get('/api/jobs/status', { context });
~~~

That value affects interceptors that read the token for this request; it does not change server behavior.

### 19. Is HttpContext immutable like HttpHeaders?

No. HttpContext is mutable and shared between cloned requests unless a different context is supplied. Changes can remain visible if the request is retried.

### 20. Should context hold authentication tokens or authorization decisions?

No. Context is request metadata for client-side behavior, not a security boundary. Store authentication according to the application's security design and enforce authorization on the server.

### 21. Can two tokens with the same name conflict?

HttpContext uses token identity as the key. Import and reuse the same token instance when an interceptor and request service need to share a setting; creating separate token instances creates separate keys.

## Loading indicators and retries

### 22. Why use context to skip a global spinner?

Some requests, such as quiet background refreshes, may not need to change the global loading UI. A typed token lets the request opt out without adding a server header or special URL convention.

### 23. Should a loading interceptor use finalize?

finalize runs when the returned Observable completes, errors, or is unsubscribed. It is useful for balancing a begin/end loading counter. For concurrent requests, use a counter or request tracking rather than a single boolean that one request can clear while another is active.

### 24. Will the interceptor run again if the Observable is subscribed to twice?

HttpClient Observables are cold. Each subscription sends a new request and traverses the configured interceptor chain.

### 25. Why does idempotency matter in interceptors?

A retry can cause the same logical request to pass through interceptors again. Cloning and setting a header consistently avoids accidental duplicate header values. In-place mutation of a body can accumulate changes across attempts.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="088-upload-and-download-progress-events.md">088 - Upload and Download Progress Events</a></td><td><a href="../lessons/089-functional-interceptors-and-request-context.md">Lesson</a></td><td><a href="../examples/089-functional-interceptors-and-request-context.md">Example</a></td><td><a href="090-http-errors-retry-policy-and-timeout-handling.md">090 - HTTP Errors, Retry Policy, and Timeout Handling</a></td>
</tr>
  </tbody>
</table>
