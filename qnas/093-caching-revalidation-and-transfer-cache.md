# Caching, Revalidation, and Transfer Cache: Q&A

## 1. Why use caching?

Caching can reduce repeated network transfers and latency. The right policy depends on whether data is public, personalized, frequently changing, or safe to reuse.

## 2. Is browser HTTP caching the same as Angular transfer cache?

No. Browser HTTP caching follows HTTP cache semantics and server headers. Angular transfer cache reuses eligible HttpClient responses made during server rendering while the browser hydrates the initial page.

## 3. Is Angular transfer cache a persistent client cache?

No. It is for the SSR-to-browser handoff. Angular uses transferred entries during initial hydration and stops using them after the browser application becomes stable.

## 4. What is an application-level cache?

It is data retained in application-managed state, such as a service, signal store, or RxJS stream. Its invalidation and lifetime are controlled by application code.

## 5. What does Cache-Control: max-age do?

It allows a response to be considered fresh for the specified number of seconds. While fresh, a cache may reuse it without asking the origin again.

## 6. Does Cache-Control: no-cache mean that the response is never stored?

No. It can be stored, but a cache must validate it before reusing it. Use no-store when caches should not store the response.

## 7. What does Cache-Control: private mean?

It prevents shared caches from storing a response as a shared entry. It can still be stored by a private client cache unless another directive prevents it.

## 8. What is an ETag?

An ETag is a server-provided validator for a representation. It lets a client or cache ask whether the stored representation is still current.

## 9. How does If-None-Match work?

A client sends the validator it has in an If-None-Match request header. If the representation is unchanged, the server can return 304 Not Modified; if it changed, the server returns a new representation.

## 10. Does a 304 response contain the new body?

No. A 304 response indicates that the cached representation can be reused. The HTTP cache combines the stored body with the validation response metadata.

## 11. Should I manually add If-None-Match in Angular?

Usually no. Let the browser's HTTP cache and the server handle normal validation. Manually managing validators requires correctly handling 304 responses and cached bodies.

## 12. What does Angular's Fetch cache option control?

It controls how an HttpClient request interacts with the browser HTTP cache when using the Fetch backend. It does not configure Angular SSR transfer-cache filtering or a service-level data cache.

## 13. Is Fetch cache: no-cache the same as no-store?

No. no-cache means revalidate before reusing a stored response. no-store avoids using or updating the browser's HTTP cache for that request.

## 14. What does force-cache do?

It prefers a matching cached response, even if stale. Use it only when serving stale data is acceptable for the resource.

## 15. What does only-if-cached do?

It uses a matching cached response or fails when none is available, subject to Fetch request mode constraints. It is not a general offline strategy by itself.

## 16. Which methods does Angular transfer cache by default?

Current Angular documentation lists eligible HEAD, QUERY, and GET requests by default. These must also meet the credential and cache-control eligibility rules.

## 17. Which requests or responses does Angular transfer cache exclude by default?

It excludes requests with Authorization, Proxy-Authorization, or Cookie headers, credentialed request settings, and non-cacheable Cache-Control directives such as no-store, no-cache, or private. It also excludes responses with Set-Cookie and requests using Fetch cache modes no-store or no-cache.

## 18. How can I disable transfer caching for one request?

Set transferCache: false in the HttpClient request options.

## 19. How can I exclude a group of endpoints?

Use the filter option in withHttpTransferCacheOptions within provideClientHydration. For example, exclude profile or account endpoints from transfer caching.

## 20. Does transfer cache include response headers by default?

No. No response headers are included by default. If the hydrated response needs a header, explicitly allowlist it with includeHeaders.

## 21. Should I include authentication headers in transfer cache?

No, not unless you have proved that the responses are safe to reuse and cannot expose user-specific data. Angular excludes credential-bearing requests by default for this reason.

## 22. Can I enable transfer caching for credentialed requests?

The configuration supports it, but enabling it can transfer personalized data into rendered HTML. Only opt in when the response is safe to cache and reuse across the server-to-browser boundary.

## 23. Can I include POST requests in transfer cache?

Angular allows POST transfer caching to be enabled. Use it only for idempotent, read-only operations such as certain GraphQL queries. Do not cache mutations.

## 24. Why might the browser repeat an SSR request during hydration?

The request may be ineligible for transfer caching, excluded by a filter, marked non-cacheable, credentialed, or not identical in the server and browser environments. Check the server request and the hydration transfer-cache configuration.

## 25. Does Cache-Control: private automatically exclude an Angular transfer-cache entry?

Angular's current transfer-cache behavior treats private as a directive that excludes the request or response from transfer caching. It also has separate request-level options such as transferCache: false.

## 26. Can I use no-cache to prevent an SSR response from appearing in transfer cache?

Angular excludes no-cache transfer-cache requests and responses by default. However, no-cache still permits storage in HTTP caches when revalidated; it is not the same as a universal no-store directive.

## 27. Why is caching personalized SSR HTML dangerous?

The transfer cache is serialized into the HTML delivered to the browser. If a server or CDN reuses personalized HTML for a different user, one user's data may be exposed. Exclude personalized data and configure shared HTML caching carefully.

## 28. Does an HTTP cache mode invalidate a signal or RxJS cache?

No. Browser Fetch cache modes do not clear application-managed values or Angular transfer-cache entries. Invalidate or update application state explicitly after mutations.

## 29. What should happen after a successful mutation?

Update the relevant local state from the server response or invalidate and refetch the affected resources. Consider related lists and detail views that may also have changed.

## 30. How should I choose a policy for private data?

Ask the backend to define the response's HTTP Cache-Control policy. Exclude user-specific responses from SSR transfer caching unless reuse is explicitly safe, and avoid shared caching of personalized rendered HTML.

## 31. Does adding ETag guarantee the browser always sends a conditional request?

No. A conditional request occurs when the cache needs revalidation. If a response is still fresh under its cache policy, the browser may reuse it without contacting the server.

## 32. Can a service worker or Cache API affect these results?

Yes. Service workers and application code can provide additional caching behavior. They are separate from Angular transfer cache and must have their own cache scope, freshness, privacy, and invalidation policies.

## 33. What should I inspect when a response seems stale?

Inspect the Network panel for cache indicators, request and response Cache-Control headers, ETag or Last-Modified validators, and whether the request came from the network, a browser cache, a service worker, or transfer cache.

## 34. What is the main rule for safe caching?

Cache public data according to server-defined HTTP policy. Revalidate data when freshness matters. Treat transfer cache as a short-lived hydration optimization, and keep personalized responses out of shared or cross-user caches.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="092-authentication-cookies-and-xsrf-protection.md">092 - Authentication, Cookies, and XSRF Protection</a></td><td><a href="../lessons/093-caching-revalidation-and-transfer-cache.md">Lesson</a></td><td><a href="../examples/093-caching-revalidation-and-transfer-cache.md">Example</a></td><td><a href="094-loading-empty-success-and-error-ui-states.md">094 - Loading, Empty, Success, and Error UI States</a></td>
</tr>
  </tbody>
</table>
