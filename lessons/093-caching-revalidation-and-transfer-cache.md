# Caching, Revalidation, and Transfer Cache

Caching can reduce repeated network work, but several different mechanisms are involved. HTTP caches follow server response headers and validators. Angular's SSR transfer cache reuses selected server-rendered HttpClient responses during browser hydration. An application-level cache is custom state managed by application code.

## Learning outcomes

After this lesson, you should be able to:

- Distinguish browser HTTP caching from Angular SSR transfer caching.
- Explain Cache-Control, ETag, conditional requests, and 304 responses.
- Select an appropriate Fetch cache mode for an HttpClient request.
- Configure and filter Angular's HTTP transfer cache.
- Exclude private or user-specific responses from transfer caching.
- Recognize when explicit application-cache invalidation is needed.

## 1. Three different caches

| Mechanism | What it does | Typical lifetime |
| --- | --- | --- |
| Browser or intermediary HTTP cache | Reuses or revalidates responses according to HTTP caching headers | Controlled by HTTP semantics, browser behavior, and intermediaries |
| Angular HTTP transfer cache | Serializes eligible SSR HttpClient responses into the rendered page so hydration can reuse them | Initial SSR-to-browser hydration, until the browser app becomes stable |
| Application-level cache | Stores data in a service, signal store, RxJS stream, or other app state | Defined by application code |

These mechanisms are related but not interchangeable. For example, a Fetch cache option controls interaction with the browser's HTTP cache; it does not configure the Angular SSR transfer-cache filter.

## 2. Set HTTP caching policy on the server

The server response should express whether a response can be cached and how it should be revalidated. A public, versioned resource may be cached for a period:

~~~http
HTTP/1.1 200 OK
Cache-Control: public, max-age=300
ETag: "catalog-v42"
Content-Type: application/json
~~~

For private data that may be stored by the client but must be revalidated before reuse, the server can use an appropriate private revalidation policy:

~~~http
HTTP/1.1 200 OK
Cache-Control: private, no-cache
ETag: "profile-v8"
Content-Type: application/json
~~~

The backend must choose directives based on the data and deployment. Shared caches should not reuse one user's private response for another user.

- max-age allows a cached response to be reused while it is fresh.
- no-cache does not mean "never store". It requires validation before a stored response is reused.
- no-store tells caches not to store the response.
- private prevents shared caches from storing the response as a shared entry; it may still be stored by a private client cache unless another directive prevents it.

For highly sensitive data, the backend may choose stricter directives such as private, no-store. This does not erase responses already stored elsewhere; caching policy should be designed consistently across the origin, browser, SSR, and any CDN.

## 3. Revalidate with ETag

An ETag is a server-selected validator for a representation. When a stored response needs revalidation, the browser can send its validator in If-None-Match:

~~~http
GET /api/catalog HTTP/1.1
If-None-Match: "catalog-v42"
~~~

If the representation has not changed, the server can reply:

~~~http
HTTP/1.1 304 Not Modified
ETag: "catalog-v42"
Cache-Control: public, max-age=300
~~~

A 304 response has no replacement body. The HTTP cache can reuse the previously stored body with the updated metadata. If the representation changed, the server returns a new 200 response with the updated body and validator.

Last-Modified and If-Modified-Since provide a time-based alternative. ETag is often more precise when the server can produce a useful validator.

Normally, let the browser and server perform HTTP revalidation. Do not add If-None-Match manually unless the application has a specific protocol and handles the validator and 304 response correctly.

## 4. Choose a Fetch cache mode when needed

With Angular's default Fetch backend, HttpClient request options support the Fetch cache option. It controls the browser HTTP cache interaction, not Angular's SSR transfer cache:

~~~ts
this.http.get<CatalogItem[]>('/api/catalog', {
  cache: 'no-cache'
});
~~~

Common Fetch cache modes include:

| Mode | General behavior |
| --- | --- |
| default | Use the browser's normal HTTP cache behavior |
| no-cache | Revalidate a stored response with the network before reusing it |
| no-store | Do not use or update the browser HTTP cache for this request |
| force-cache | Prefer a matching cached response, even when stale |
| only-if-cached | Use a matching cached response or fail if one is unavailable; subject to request mode constraints |

Use a mode that matches the resource and server policy. no-cache is a revalidation behavior, not the same as no-store. Prefer server Cache-Control headers to define the resource's durable policy.

## 5. Angular HTTP transfer cache during SSR and hydration

When Angular renders a page on the server, HttpClient can cache eligible responses, serialize them into the initial HTML, and let the browser reuse them while hydrating. This can prevent the browser from repeating a request already made during server rendering.

The transfer cache is intended for the initial SSR-to-browser handoff. Angular stops using the transferred entries after the browser application becomes stable. It is not a general-purpose persistent client cache.

By default, Angular transfer-caches eligible HEAD, QUERY, and GET requests without credential headers or credentialed request settings. Angular skips requests or responses with cache-control directives that prohibit transfer caching, including no-store, no-cache, or private; it also skips responses with Set-Cookie and requests using Fetch cache modes no-store or no-cache.

## 6. Exclude user-specific endpoints

A response such as an account profile is personalized. Do not place it in an HTML transfer cache that may be reused or shared incorrectly.

~~~ts
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import {
  provideClientHydration,
  withHttpTransferCacheOptions
} from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    provideClientHydration(
      withHttpTransferCacheOptions({
        filter: (request) => !request.url.includes('/api/profile')
      })
    )
  ]
};
~~~

The filter is an additional application policy. The server should also return correct caching directives for private responses, and deployment infrastructure must not cache personalized rendered HTML as if it were public.

A request can opt out individually:

~~~ts
this.http.get<UserProfile>('/api/profile', {
  transferCache: false
});
~~~

Angular also excludes several credentialed and private cases by default. Treat those defaults as safeguards, not as a reason to opt private responses into transfer caching. If changing the filter or defaults, check what data is serialized into the HTML and whether the entire response is safe for that page and user.

## 7. Do not casually include authentication data

By default, transfer caching excludes requests with Authorization, Proxy-Authorization, or Cookie headers, as well as request settings that send credentials. Responses with Set-Cookie are excluded. This helps avoid embedding user-specific data into transferred HTML.

Avoid enabling includeRequestsWithAuthHeaders or includeRequestsWithCredentials globally unless you have verified that every affected response is safe to reuse. Likewise, avoid enabling includeNonCacheableRequests to override no-store, no-cache, private, or Set-Cookie safeguards without a carefully reviewed reason.

If response headers are needed on the hydrated response, include only a deliberate allowlist. Angular includes no response headers by default:

~~~ts
provideClientHydration(
  withHttpTransferCacheOptions({
    includeHeaders: ['ETag', 'Cache-Control'],
    filter: (request) => request.url.startsWith('/api/public/')
  })
)
~~~

Never include authentication tokens or other sensitive response headers in the transfer cache.

## 8. POST requests and transfer caching

POST is not transfer-cached by default. Angular can be configured to include POST requests, for example for a GraphQL read query sent using POST, but only do so when the operation is read-only, idempotent, and safe to reuse between server and browser rendering.

~~~ts
provideClientHydration(
  withHttpTransferCacheOptions({
    includePostRequests: true,
    filter: (request) =>
      request.url.includes('/graphql') && request.method === 'POST'
  })
)
~~~

Do not enable POST transfer caching for mutations, such as creating an order or updating an account. A cache must not change the number or timing of server-side effects.

## 9. Separate HTTP caching from application data refresh

The Fetch cache mode does not clear an application-level signal, service map, or shareReplay stream. Likewise, a cached browser response and an SSR transfer entry have separate eligibility and lifetime rules.

After a mutation, decide how the visible application state should be refreshed:

- Update the local state with the server's returned representation.
- Invalidate and refetch the affected resource.
- Refetch related list and detail data when the backend response may have changed them.

The server's cache validators and directives still govern the HTTP response. Avoid adding an unbounded in-memory cache when HTTP caching already meets the requirement.

## Common mistakes

### Treating no-cache as no-store

no-cache allows storage but requires revalidation before reuse. Use no-store when the response must not be stored by caches.

### Assuming transfer cache is persistent

Angular transfer cache is for reusing eligible SSR responses during initial browser hydration. It does not replace a long-lived application cache.

### Caching personalized SSR responses globally

The transfer cache is serialized into rendered HTML. A CDN or server cache that reuses personalized HTML across users can expose data. Exclude private endpoints and configure rendered-page caching carefully.

### Opting credentialed requests into transfer cache without reviewing them

Credentialed responses can be user-specific. Do not enable credential or auth-header caching globally just to reduce requests.

### Including sensitive headers

Transferred HTML is delivered to the browser. Do not serialize authorization tokens or sensitive headers in the transfer cache.

### Expecting cache modes to invalidate all caches

Fetch cache options affect the browser HTTP cache. They do not automatically clear Angular transfer entries or app-managed state.

### Enabling POST caching for mutations

Transfer-cached POST requests must represent safe, idempotent reads. Do not include state-changing requests.

## Key idea

Choose server HTTP directives and validators for browser and intermediary caches. Use Angular transfer cache only to reuse safe SSR responses during hydration, and exclude personalized data. Keep browser HTTP caching, Angular transfer caching, and application-state caching as distinct tools.

## Related references

- Q&A: qnas/093-caching-revalidation-and-transfer-cache.md
- Examples: examples/093-caching-revalidation-and-transfer-cache.md
- Angular SSR and HttpClient transfer cache: https://angular.dev/best-practices/performance/ssr
- Angular Fetch cache options: https://angular.dev/guide/http/making-requests
- MDN HTTP caching and revalidation: https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching
