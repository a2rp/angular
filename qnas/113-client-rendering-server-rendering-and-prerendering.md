# Client Rendering, Server Rendering, and Prerendering — Q&A

## 1. What does CSR mean?

Client-side rendering creates the page in the visitor's browser after the application JavaScript loads and runs. Angular uses CSR by default.

## 2. What does SSR mean?

Server-side rendering creates HTML on a server for each incoming request and sends it to the browser. The response can contain rendered page content before the browser finishes loading the client application.

## 3. What does SSG or prerendering mean?

Static-site generation prerenders pages at build time and writes HTML files. The host can then serve those files as static assets, often through a CDN.

## 4. What is hybrid rendering?

Hybrid rendering means using more than one rendering mode in the same application. Angular can select Client, Server, or Prerender modes by route.

## 5. Which rendering mode does Angular use by default?

Angular applications use client-side rendering by default. SSR or prerendering must be configured for the routes that need them.

## 6. When is CSR a good fit?

CSR can be a good fit for authenticated dashboards, interactive applications, and products that can be served from static hosting. It may be suitable when most useful content is behind sign-in and an initial server-rendered page is not a requirement.

## 7. When is SSR a good fit?

SSR can fit routes that need fresh content on each request or benefit from returning populated HTML immediately. It requires a server or platform that can run the rendering workload and access the required data.

## 8. When is prerendering a good fit?

Prerender stable pages whose data and routes are known at build time, such as documentation or public course descriptions. Prerendering is less suitable for pages whose content is specific to the visitor or changes too frequently for rebuilds.

## 9. Does SSR make a page interactive before JavaScript loads?

No. SSR sends rendered HTML, but the client application still needs to load and attach behavior. Hydration connects the client-side application to server-rendered HTML.

## 10. Does SSR always improve performance?

No. It may improve content visibility, but the server response can be slow, data fetching can be expensive, and the browser still needs to load client code for interaction. Measure the complete user experience.

## 11. Does SSR automatically improve SEO?

No. It can make page content available in the initial HTML, which may help crawlers and link previews. Search visibility also depends on metadata, content, links, performance, accessibility, and crawler behavior.

## 12. What is the main freshness trade-off with prerendering?

The HTML reflects data available when the build ran. Changes are not reflected until the page is regenerated and deployed, unless a configured fallback or other data mechanism is used.

## 13. Does SSR always use current data?

Not necessarily. The server can use caches, and responses may be stale depending on the cache policy. Rendering per request does not guarantee a fresh backend response if that request is served from a cache.

## 14. Can prerendered pages include dynamic route parameters?

Yes, if the build process can determine which parameter values to generate. For example, a list of known article IDs can produce one static HTML file per article. Unknown paths need an appropriate fallback or will not have a generated page.

## 15. What is the main operational cost of SSR?

The app needs a server or compatible runtime for request-time rendering. Teams must monitor it, scale it, secure it, and handle render or data failures. Server capacity also affects response time and cost.

## 16. Can a prerendered app run without a Node server?

Yes. If configured for static output and all required routes are prerendered, the output can be served by a static file host or CDN. Confirm the Angular version and build configuration used by the project.

## 17. Should every route use the same rendering mode?

No. Public documentation, fresh search results, and private dashboards may have different needs. Choose a mode for each route based on its content, personalization, freshness, and hosting constraints.

## 18. Is personalized content safe to prerender?

Usually not. Build-time HTML is shared static output and should not contain secrets or visitor-specific information. Personal data must be loaded and handled with an appropriate runtime and cache boundary.

## 19. What is a shared-cache risk with SSR?

A shared cache can return one rendered response to multiple visitors. If the HTML includes user-specific data and the cache key or directives are not correct, another user may receive it. Keep personalized pages out of public shared caches unless the response is safely partitioned.

## 20. Can a hybrid app use CSR for some routes and SSR for others?

Yes. Angular's server route configuration is designed to select a rendering mode by path. Configure route patterns carefully, including fallback routes, and verify the generated output and deployed server behavior.

## 21. How do I choose between SSR and prerendering for public content?

If the content is stable and known at build time, prerendering may be simpler and cheaper to serve. If it changes frequently or depends on request-time context, SSR may be more appropriate. Consider freshness, build cost, server cost, and caching.

## 22. What measurements should I compare when changing rendering modes?

Measure server response timing, when the main content appears, client JavaScript startup, interactivity, route-specific field data, and server resource use. The relevant measures depend on the route and what problem the change is intended to solve.

## 23. What are common rendering strategy mistakes?

Common mistakes include enabling SSR for every route without need, prerendering personalized content, caching private HTML publicly, assuming SSR removes the need for client JavaScript, and forgetting to rebuild stale prerendered pages.

## 24. What is a reasonable hybrid example?

A documentation section can be prerendered, a frequently changing public catalog can be SSR-rendered, and a signed-in workspace can be client-rendered. These are starting points; product needs and measurements should decide.

## Further reading

- [Angular rendering strategies](https://angular.dev/guide/routing/rendering-strategies)
- [Angular prerendering and hybrid rendering](https://angular.dev/guide/prerendering)
- [Angular SSR](https://angular.dev/guide/ssr)