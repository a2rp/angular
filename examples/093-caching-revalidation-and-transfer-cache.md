# Caching, Revalidation, and Transfer Cache: Examples

These examples show how browser HTTP caching, HTTP validators, and Angular's SSR transfer cache work together while remaining separate mechanisms.

## 1. Define a server cache policy for public catalog data

The backend can cache a public catalog response for five minutes and provide an ETag validator:

~~~http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: public, max-age=300
ETag: "catalog-v42"

[{"id":"p-1","name":"Notebook"}]
~~~

When the response becomes stale, the browser may revalidate it:

~~~http
GET /api/catalog HTTP/1.1
If-None-Match: "catalog-v42"
~~~

If the catalog is unchanged, the server returns:

~~~http
HTTP/1.1 304 Not Modified
Cache-Control: public, max-age=300
ETag: "catalog-v42"
~~~

The 304 has no replacement body. The HTTP cache reuses its stored response body. If the catalog changed, the server returns a new 200 response with the new body and ETag.

The browser performs this conditional-request flow from the HTTP metadata. Angular application code usually just requests the resource:

~~~ts
this.http.get<CatalogItem[]>('/api/catalog');
~~~

## 2. Choose a browser Fetch cache mode for one request

Angular's default Fetch backend supports the Fetch cache option. Use it only when the request needs behavior different from the browser's normal HTTP cache policy.

~~~ts
// Revalidate a stored response before reusing it.
this.http.get<CatalogItem[]>('/api/catalog', {
  cache: 'no-cache'
});

// Do not use or update the browser HTTP cache for this request.
this.http.get<LiveStatus>('/api/live-status', {
  cache: 'no-store'
});

// Prefer a matching cached response, even if it is stale.
// Use only when stale content is acceptable.
this.http.get<HelpArticle[]>('/api/help', {
  cache: 'force-cache'
});
~~~

These options do not clear Angular transfer-cache entries or application-managed data. The backend's response Cache-Control headers remain important.

## 3. Configure Angular transfer cache to exclude account data

Angular uses transfer cache to reuse eligible server-rendered HttpClient responses during initial browser hydration. Exclude user-specific endpoints with a filter.

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
        filter: (request) =>
          !request.url.includes('/api/profile')
          && !request.url.includes('/api/account')
      })
    )
  ]
};
~~~

The filter controls Angular's transfer cache. The backend should also return appropriate HTTP cache headers for private responses. If the rendered HTML itself is cached by a CDN or reverse proxy, configure that layer so personalized HTML is not reused between users.

## 4. Opt out of transfer cache for a sensitive request

A request can opt out individually:

~~~ts
getAccountSettings(): Observable<AccountSettings> {
  return this.http.get<AccountSettings>('/api/account/settings', {
    transferCache: false
  });
}
~~~

A backend can also send a private or non-cacheable response policy:

~~~http
Cache-Control: private, no-store
~~~

Angular's transfer-cache defaults skip several non-cacheable responses and requests. Use both server policy and the request option when the data is sensitive, and review any CDN or service-worker caching separately.

## 5. Include only needed response headers in transfer cache

No response headers are included in transfer-cache entries by default. If the hydrated client needs a safe header for a public endpoint, allowlist only that header:

~~~ts
provideClientHydration(
  withHttpTransferCacheOptions({
    includeHeaders: ['ETag'],
    filter: (request) => request.url.startsWith('/api/public/')
  })
)
~~~

Do not include authentication tokens, session data, or other sensitive headers. Transferred data is serialized into the HTML sent to the browser.

## 6. Avoid transfer-caching authenticated responses

Angular excludes requests with auth or cookie headers and credentialed requests by default. Do not override that policy just to reduce request counts.

If a team considers enabling credentialed transfer caching, first prove that the affected responses are not user-specific and cannot be reused across requests or users. A private profile endpoint should stay excluded:

~~~ts
provideClientHydration(
  withHttpTransferCacheOptions({
    filter: (request) => !request.url.includes('/api/profile')
  })
)
~~~

If a response varies by the authenticated user, the server-side rendered HTML and every cache in front of it must also respect that user boundary.

## 7. Cache a read-only POST only when its contract is safe

Some GraphQL servers accept read-only queries using POST. Transfer caching can be enabled for POST requests, but only when the endpoint and request are safe and idempotent. Use a dedicated read-only endpoint or a strict request filter:

~~~ts
provideClientHydration(
  withHttpTransferCacheOptions({
    includePostRequests: true,
    filter: (request) =>
      request.url.endsWith('/graphql/public-query')
      && request.method === 'POST'
  })
)
~~~

This assumes the endpoint accepts only read-only queries. Do not use this configuration for a general GraphQL endpoint that can also execute mutations. Never enable POST transfer caching for operations that create, update, or delete server data.

## 8. Keep app-level state refresh explicit

HTTP caching and transfer caching do not know when an application-level signal or service cache should be invalidated after a mutation. Update or refresh that state deliberately.

~~~ts
saveProduct(product: Product): Observable<Product> {
  return this.http.put<Product>(
    '/api/products/' + encodeURIComponent(product.id),
    product
  ).pipe(
    tap((saved) => {
      this.productStore.update(saved);
    })
  );
}
~~~

Alternatively, clear the affected app-level entry and refetch it. Consider related lists or detail views that may also have changed.

## 9. Compare the three layers in a request flow

| Layer | Example behavior | Controlled by |
| --- | --- | --- |
| Browser HTTP cache | Reuses a fresh response or validates an ETag | Response headers and Fetch cache mode |
| Angular transfer cache | Reuses an eligible SSR response during hydration | Hydration configuration and transferCache request option |
| Application state cache | Returns data retained in a service or store | Application invalidation and refresh logic |

If a UI appears stale, identify which layer supplied the value before changing cache settings.

## 10. Verify caching behavior manually

Use browser developer tools and server logs to inspect:

- Response Cache-Control and ETag headers.
- If-None-Match requests and 304 responses after a response becomes stale.
- Whether the browser reports a network response, memory/disk cache, or service worker.
- Whether the server made an SSR request and whether hydration repeated it.
- Whether account or profile data is absent from transfer-cached HTML.
- Whether a mutation updates or invalidates application-managed state.

A 304 response is not a failed request. It is a successful validation that allows the cache to reuse its stored representation.

## Checklist

- Let the server define HTTP freshness and revalidation policy.
- Use ETag or Last-Modified validators when useful.
- Remember that no-cache means revalidate, while no-store means do not store.
- Treat Fetch cache modes, Angular transfer cache, and app-level caches separately.
- Keep user-specific and credentialed responses out of transfer cache by default.
- Use transferCache: false or a filter for sensitive endpoints.
- Allowlist only non-sensitive headers for transfer-cache entries.
- Enable POST transfer caching only for safe, idempotent read operations.
- Refresh application-managed state after mutations.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="092-authentication-cookies-and-xsrf-protection.md">092 - Authentication, Cookies, and XSRF Protection</a></td><td><a href="../lessons/093-caching-revalidation-and-transfer-cache.md">Lesson</a></td><td><a href="../qnas/093-caching-revalidation-and-transfer-cache.md">Q&amp;A</a></td><td><a href="094-loading-empty-success-and-error-ui-states.md">094 - Loading, Empty, Success, and Error UI States</a></td>
</tr>
  </tbody>
</table>
