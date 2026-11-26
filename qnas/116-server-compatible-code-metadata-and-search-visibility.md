# Server-Compatible Code, Metadata, and Search Visibility - Q&A

## 1. Why can browser code fail during SSR?

Angular runs application code in a server environment while rendering SSR routes or prerendering pages. Browser globals and DOM capabilities may not exist there. Code that reads window, document, navigator, location, or layout properties can therefore throw.

## 2. Is checking isPlatformBrowser in a component enough?

It can guard some imperative code, but it does not make a browser-only library safe if the library accesses browser globals during module import. It also should not be used in a template to render different initial server and browser markup because that can cause hydration mismatch.

## 3. What is a better way to separate server and browser behavior?

Put the platform-specific behavior behind an injected abstraction or platform-specific provider. Components then depend on one service API while the server and browser can supply appropriate implementations.

## 4. When should I use afterNextRender?

Use afterNextRender for browser-only work that must happen after Angular renders, such as initializing a DOM-dependent library. It is skipped on the server. Still take care: Angular does not guarantee that every component is fully hydrated before the callback runs.

## 5. Can I use afterNextRender to replace my whole template on the browser?

That is usually a poor pattern. It can cause a visible change and hydration mismatch. Render a compatible initial structure on both platforms and use a focused client enhancement when necessary.

## 6. Why are different server and browser templates risky?

Hydration expects the browser DOM to match the server-rendered structure. Different conditions, such as a browser check that displays one tree on the server and another in the client, can cause Angular to find nodes it did not expect.

## 7. How do I add a title for each route?

Use the route title property. Angular Router updates the document title when the route activates. A route title can be a string or a resolver for a title that depends on route data.

## 8. What is TitleStrategy?

TitleStrategy is the Angular Router abstraction used to update the document title after navigation. The default strategy uses the deepest primary route title. A custom strategy can add an app suffix or compose a title from route data.

## 9. What does Angular's Meta service do?

The Meta service can add, update, retrieve, and remove HTML meta tags. It can manage description and social-sharing properties such as Open Graph tags. Route-specific metadata should be set from trusted page data.

## 10. How should I create a meta description?

Write a concise, accurate summary for the specific page. Avoid copying one description to every route. Search engines may choose other text, so a meta description does not guarantee the exact search snippet.

## 11. How do I add Open Graph metadata?

Use Angular's Meta service to update properties such as og:title, og:description, and og:image. Ensure the values are correct for the page and present in the HTML returned to the social crawler.

## 12. Does metadata need to be in the initial HTML?

For the most reliable visibility to crawlers and link preview tools, include it in the server-rendered or prerendered response. Metadata added only after a client API request may not be visible to every consumer.

## 13. Does SSR guarantee search ranking?

No. SSR can make page content and metadata available in the initial HTML, but ranking depends on many factors such as content relevance, links, accessibility, performance, crawlability, and search engine systems. No Angular API guarantees ranking.

## 14. What should I inspect to verify server-rendered metadata?

Request the route directly and inspect the document response or view-source output. Check the title, description, social properties, canonical link if used, and the expected page content. A browser's live DOM after client rendering may not reveal what the initial server response contained.

## 15. Can prerendered HTML contain user-specific metadata?

It should not. Prerendered output is generated at build time and commonly shared among visitors. Do not include personal data, tokens, or private account values in it.

## 16. Is it safe to access localStorage in a constructor?

No. A constructor can execute during server rendering, where localStorage is unavailable. Move the read to browser-only code after rendering or put storage behind a browser-specific service. Ensure any resulting template update is hydration-safe.

## 17. Can a third-party chart library break SSR?

Yes, if it reads window, document, canvas, or other browser capabilities during import or initialization. Check the library's SSR support. Dynamically load it in a browser-only lifecycle when appropriate and keep server rendering independent of the library.

## 18. What are platform-specific providers?

They are provider configurations that supply different implementations of the same abstraction on different platforms. They centralize platform differences and keep application components from scattering environment checks through templates.

## 19. Should I use document directly to update meta tags?

Prefer Angular's Title and Meta services for their intended responsibilities. Direct DOM manipulation can be harder to keep server-compatible and hydration-safe. Use DOM APIs only for behavior that cannot be expressed through Angular services or templates.

## 20. How does a canonical URL help?

A canonical URL identifies the preferred URL for equivalent or duplicate content. It can help search systems understand which URL should represent the content. Set it to the correct public URL and avoid constructing it from untrusted request headers.

## 21. What is the difference between metadata and page content?

Metadata describes the page in the document head. The actual page content, headings, meaningful links, status code, and accessibility still matter. Metadata cannot replace useful content.

## 22. What if SSR works locally but fails after deployment?

Check dependencies for browser assumptions, CDN transformations, proxy behavior, route data access, and the actual deployed server output. Confirm the initial document response and browser hydration separately.

## 23. Can afterNextRender read layout safely?

It runs on the browser after a render, but DOM reads can still trigger layout and the callback may run before hydration completes. Use the explicit read/write phases available in the project's Angular version, avoid unnecessary layout reads, and do not mutate Angular-owned DOM unexpectedly.

## 24. What is a sound server-compatibility workflow?

1. Build or serve with the SSR/prerender configuration.
2. Request each route directly.
3. Check server logs for browser-global errors.
4. Inspect returned content and metadata.
5. Verify client hydration and interactive behavior.
6. Check that generated pages contain no visitor-specific data.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="115-hydration-event-replay-and-incremental-hydration.md">115 - Hydration, Event Replay, and Incremental Hydration</a></td><td><a href="../lessons/116-server-compatible-code-metadata-and-search-visibility.md">Lesson</a></td><td><a href="../examples/116-server-compatible-code-metadata-and-search-visibility.md">Example</a></td><td><a href="117-security-xss-sanitization-csp-and-trusted-content.md">117 - Security: XSS, Sanitization, CSP, and Trusted Content</a></td>
</tr>
  </tbody>
</table>
