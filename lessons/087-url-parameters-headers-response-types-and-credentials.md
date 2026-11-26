# URL Parameters, Headers, Response Types, and Credentials

HttpClient request options let an application shape a request and choose what kind of response stream it receives. This lesson covers URL query parameters, request headers, responseType, observe, and credential handling. Upload and download progress events are covered in the next lesson.

## Learning outcomes

After this lesson, you should be able to:

- Add query parameters without manually concatenating a query string.
- Use HttpParams and HttpHeaders without losing their immutable updates.
- Choose a JSON, text, Blob, or ArrayBuffer response type.
- Decide whether a request should emit its body, full response, or HTTP event stream.
- Configure same-origin or cross-origin credentials with care.
- Keep option literals precise so TypeScript selects the intended HttpClient overload.

## 1. Add query parameters with an object

For simple query values, pass an object in the params option:

~~~ts
this.http.get<Product[]>('/api/products', {
  params: {
    category: 'books',
    page: 2,
    inStock: true
  }
});
~~~

Angular serializes the values into the request URL, conceptually:

~~~text
/api/products?category=books&page=2&inStock=true
~~~

Use the API's actual parameter names and meanings. Query parameters are visible in URLs and may appear in browser history, server logs, analytics, and monitoring systems. Do not put secrets or sensitive personal data in a URL.

## 2. Use HttpParams for controlled construction

HttpParams is useful when parameters are conditional, repeated, or constructed across multiple steps:

~~~ts
import { HttpParams } from '@angular/common/http';

let params = new HttpParams()
  .set('category', 'books')
  .set('page', 1);

if (includeArchived) {
  params = params.set('archived', true);
}

params = params.append('tag', 'angular');
params = params.append('tag', 'typescript');

return this.http.get<Product[]>('/api/products', { params });
~~~

HttpParams is immutable. Each set, append, or delete call returns a new instance. Reassign the result or the original instance remains unchanged.

- set replaces the values associated with a key.
- append adds another value for the key.
- get reads the first value.
- getAll returns every value for a key.
- delete removes a key or a particular value.

Angular encodes parameter names and values by default. Use a custom HttpParameterCodec only when the server requires a different encoding convention.

## 3. Set request headers

An object is convenient for a small number of headers:

~~~ts
this.http.get<Product[]>('/api/products', {
  headers: {
    'X-Client-Version': '2'
  }
});
~~~

For reusable construction or multiple operations, use HttpHeaders:

~~~ts
import { HttpHeaders } from '@angular/common/http';

const baseHeaders = new HttpHeaders()
  .set('X-Client-Version', '2')
  .set('Accept-Language', 'en');

const requestHeaders = baseHeaders.set('X-Request-Mode', 'preview');

return this.http.get<Product[]>('/api/products', {
  headers: requestHeaders
});
~~~

HttpHeaders is immutable too. set and append return a new HttpHeaders instance. set replaces the existing value for a name; append adds a value. Reassign the returned instance when building headers over multiple statements.

Do not set headers such as Content-Type blindly. The right content type depends on the body and backend contract. For JSON requests, Angular generally serializes objects and sets an appropriate content type. Browser-managed headers may also be restricted.

## 4. Choose a responseType

HttpClient assumes JSON by default. For a non-JSON body, set responseType:

~~~ts
const statusText$ = this.http.get('/api/status.txt', {
  responseType: 'text'
});
~~~

Common response types are:

| responseType | Body value |
| --- | --- |
| json | Parsed JSON; default |
| text | string |
| blob | Blob |
| arraybuffer | ArrayBuffer |

Examples:

~~~ts
const report$ = this.http.get('/api/report.csv', {
  responseType: 'text'
});

const image$ = this.http.get('/api/image', {
  responseType: 'blob'
});

const fileBytes$ = this.http.get('/api/archive', {
  responseType: 'arraybuffer'
});
~~~

The generic type for a JSON request and responseType for non-JSON data are different concepts. A generic such as get<Product>() does not validate the JSON at runtime.

## 5. Keep responseType as a literal

The responseType value determines the TypeScript overload and resulting Observable type. A direct object literal retains its value:

~~~ts
this.http.get('/api/status.txt', {
  responseType: 'text'
});
~~~

When extracting options into a variable, preserve the literal with as const:

~~~ts
const textOptions = {
  responseType: 'text' as const
};

this.http.get('/api/status.txt', textOptions);
~~~

If TypeScript widens the option to a general string, it cannot reliably determine which response type the request returns.

## 6. Choose what the Observable emits with observe

By default, HttpClient emits the response body:

~~~ts
this.http.get<Product[]>('/api/products');
~~~

Use observe: 'response' when status, headers, or other full response metadata is needed:

~~~ts
this.http.get<Product[]>('/api/products', {
  observe: 'response'
}).subscribe(response => {
  console.log(response.status);
  console.log(response.headers.get('ETag'));
  console.log(response.body);
});
~~~

Use observe: 'events' for the lower-level event stream:

~~~ts
this.http.get('/api/report', {
  observe: 'events',
  responseType: 'text'
});
~~~

An event stream can contain multiple event kinds, not just the final response. Progress reporting and event filtering are covered in lesson 088.

Like responseType, observe affects the returned type. Keep it as a literal when storing options:

~~~ts
const responseOptions = {
  observe: 'response' as const
};

this.http.get<Product[]>('/api/products', responseOptions);
~~~

## 7. Send credentials to the right origin

With the Fetch backend, the credentials option controls whether browser credentials such as cookies are included:

~~~ts
this.http.get<UserProfile>('/api/profile', {
  credentials: 'same-origin'
});
~~~

Available values:

| credentials | Behavior |
| --- | --- |
| same-origin | Send credentials only to the same origin; default |
| include | Include credentials for cross-origin requests too |
| omit | Do not send credentials |

Cross-origin cookie requests also depend on browser cookie rules and server CORS configuration. The server must explicitly allow the requesting origin and credentialed access; a wildcard origin is not valid for credentialed CORS responses.

Only use include for a trusted API that is configured to receive credentials. Do not send authentication cookies or authorization data to a destination derived from untrusted input.

### Legacy withCredentials option

withCredentials is supported for compatibility with older HttpClient code:

~~~ts
this.http.get<UserProfile>('https://api.example.com/profile', {
  withCredentials: true
});
~~~

Current Angular documentation says withCredentials takes precedence over credentials when both are set. Avoid specifying both because it obscures the actual policy. Prefer the Fetch credentials option when supported by the project's Angular version.

For server-side rendering, credentials: 'include' does not automatically forward cookies from the incoming browser request. SSR code should handle forwarded credentials deliberately and only for trusted destinations.

## 8. Combine options in a request

Options can be used together:

~~~ts
const params = new HttpParams()
  .set('page', 1)
  .set('pageSize', 20);

const headers = new HttpHeaders()
  .set('X-Client-Version', '2');

return this.http.get<Product[]>('/api/products', {
  params,
  headers,
  observe: 'response',
  credentials: 'same-origin'
});
~~~

The resulting Observable emits an HttpResponse<Product[]> body, along with response status and headers.

## 9. Common mistakes

### Forgetting HttpParams is immutable

Calling params.set(...) without storing its returned value does not update params. Assign the returned instance.

### Using append when the API expects one value

append creates repeated values for a key. Use set when a new value should replace the old one.

### Treating observe: 'events' as a response body

An event stream contains request lifecycle events. Filter it for the final HttpResponse before treating it as a response body.

### Mixing observe and responseType values through widened strings

Keep extracted values literal with as const, or use a helper that returns an appropriately typed option object.

### Enabling cross-origin credentials without server support

The browser and server both enforce credential and CORS rules. Check the API origin, cookie attributes, and explicit server CORS configuration; do not weaken security controls to hide a configuration error.

### Putting private values in query strings

URLs are widely recorded. Use an appropriate request body or header according to the API contract and security design.

## Key idea

Use params for query data, headers for request metadata, responseType for the response body's representation, and observe for the level of response detail. Configure credentials only when the destination and server policy are known.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="086-build-typed-http-requests-and-api-services.md">086 - Build Typed HTTP Requests and API Services</a></td><td><a href="../qnas/087-url-parameters-headers-response-types-and-credentials.md">Q&amp;A</a></td><td><a href="../examples/087-url-parameters-headers-response-types-and-credentials.md">Example</a></td><td><a href="088-upload-and-download-progress-events.md">088 - Upload and Download Progress Events</a></td>
</tr>
  </tbody>
</table>
