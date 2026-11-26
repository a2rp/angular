# URL Parameters, Headers, Response Types, and Credentials — Q&A

## URL parameters

### 1. How do I add a query parameter to an HttpClient request?

Pass a params object in the request options:

~~~ts
this.http.get<Product[]>('/api/products', {
  params: { category: 'books', page: 2 }
});
~~~

Angular serializes these values into the URL query string.

### 2. When should I use HttpParams instead of an object?

Use an object for a small, static set of values. Use HttpParams when values are conditional, repeated, built in multiple steps, or need a custom parameter encoder.

### 3. Why did calling params.set not change my original HttpParams?

HttpParams is immutable. set, append, and delete return a new instance. Reassign the returned value:

~~~ts
params = params.set('page', 2);
~~~

### 4. What is the difference between set and append?

set replaces the values for a parameter name with one value. append adds another value for that name, which creates a repeated query parameter.

### 5. How do I send repeated query values?

Use append more than once or pass a supported array in the params object:

~~~ts
const params = new HttpParams()
  .append('tag', 'angular')
  .append('tag', 'http');
~~~

Confirm that the backend expects repeated keys rather than a comma-separated value.

### 6. Does Angular URL-encode query values?

Yes. HttpParams uses Angular's URL parameter codec by default. When the server requires a different encoding convention, provide a custom HttpParameterCodec.

### 7. Should I place an API key or private value in query params?

Generally no. URLs can be recorded in browser history, logs, analytics, and referrer data. Follow the API's security design and use an appropriate header or request body for sensitive values.

## Headers

### 8. How do I set a request header?

Pass a headers object:

~~~ts
this.http.get('/api/products', {
  headers: { 'X-Client-Version': '2' }
});
~~~

For reusable or incremental construction, use HttpHeaders.

### 9. Are HttpHeaders mutable?

No. HttpHeaders is immutable. Methods such as set and append return a new instance. Assign the result when building headers over multiple steps.

### 10. What is the difference between HttpHeaders.set and append?

set replaces the value for a header name. append adds another value. Use the operation that matches the server's expected header format.

### 11. Should I always set Content-Type manually?

No. The correct content type depends on the request body and API contract. Angular generally serializes JSON objects and supplies an appropriate content type. Avoid setting a conflicting value or setting a content type for a body that does not need one.

### 12. Can an application set any browser request header?

No. Browsers control or restrict some headers. Server CORS policy may also determine which non-simple headers are allowed. Check browser and server behavior instead of assuming every header object entry will be sent.

## Response types and observe

### 13. What does responseType change?

responseType tells HttpClient how to interpret the response body. JSON is the default; text returns a string, blob returns a Blob, and arraybuffer returns an ArrayBuffer.

### 14. Does get<Product>() validate the JSON body?

No. The generic type is a compile-time assertion. It does not validate the received data at runtime. Use runtime validation when the application needs to prove an external response matches a schema.

### 15. What does observe change?

observe controls what kind of value the Observable emits:

- body (default): the response body.
- response: the full HttpResponse, including status and headers.
- events: the request lifecycle event stream.

### 16. When should I use observe: 'response'?

Use it when the caller needs response metadata such as status or headers in addition to the body.

### 17. Is observe: 'events' the same as observe: 'response'?

No. events emits lifecycle events and may emit multiple values. response emits the final full HttpResponse. Filter an event stream for HttpEventType.Response before consuming its final body. Progress handling is covered in lesson 088.

### 18. Why does TypeScript reject an extracted responseType or observe option?

Those literal values select HttpClient overloads. An extracted object may widen the value to string. Preserve the literal type:

~~~ts
const options = {
  responseType: 'text' as const
};
~~~

Apply the same idea to observe when extracting it.

## Credentials and cross-origin requests

### 19. What does credentials: 'same-origin' mean?

It sends browser credentials only for requests to the same origin. It is the default behavior of the Fetch credentials option.

### 20. When should I use credentials: 'include'?

Use include only when a trusted cross-origin API is intended to receive cookies or other browser credentials and its CORS and cookie configuration permits that access. The server must allow the exact requesting origin and credentialed requests.

### 21. What does credentials: 'omit' do?

It requests that credentials not be sent. Use it when the API should not receive cookies or other browser-managed credentials.

### 22. What is withCredentials?

It is a compatibility option used by older HttpClient code. Current Angular documentation says it takes precedence over credentials if both are supplied. Avoid mixing the two options; choose the policy explicitly.

### 23. Does credentials: 'include' automatically forward browser cookies during SSR?

No. Angular's current HTTP guide says credentials: 'include' does not automatically forward cookies from the incoming browser request during server-side rendering. SSR forwarding must be deliberately implemented and restricted to trusted destinations.

### 24. Why can a credentialed cross-origin request still fail?

Client credentials settings do not configure the server. Browser cookie policy, cookie attributes, CORS response headers, origin rules, and server authentication must all align. Inspect the browser's network and console details and server configuration.

## Practical reminders

| Need | Option or class |
| --- | --- |
| Simple query values | params object |
| Conditional or repeated query values | HttpParams |
| Simple request headers | headers object |
| Incremental header construction | HttpHeaders |
| JSON body | responseType: 'json' (default) |
| Plain text body | responseType: 'text' |
| Binary body | responseType: 'blob' or 'arraybuffer' |
| Status and headers | observe: 'response' |
| Lifecycle and progress events | observe: 'events' |
| Same-origin credentials | credentials: 'same-origin' |
| Cross-origin credentials when explicitly required | credentials: 'include' |

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="086-build-typed-http-requests-and-api-services.md">086 - Build Typed HTTP Requests and API Services</a></td><td><a href="../lessons/087-url-parameters-headers-response-types-and-credentials.md">Lesson</a></td><td><a href="../examples/087-url-parameters-headers-response-types-and-credentials.md">Example</a></td><td><a href="088-upload-and-download-progress-events.md">088 - Upload and Download Progress Events</a></td>
</tr>
  </tbody>
</table>
