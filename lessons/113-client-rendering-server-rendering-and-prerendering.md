# Client Rendering, Server Rendering, and Prerendering

Angular can create page HTML in the browser, render it on a server for each request, or generate HTML at build time. A hybrid application can choose a different mode for each route. The right choice depends on the route's data, freshness, interactivity, SEO needs, and deployment environment. Server rendering is not automatically better for every application.

## Learning outcomes

- Explain client-side rendering (CSR), server-side rendering (SSR), and prerendering/static-site generation (SSG).
- Compare their request and build-time behavior.
- Choose a rendering mode based on route requirements.
- Understand freshness, hosting, caching, and operational trade-offs.

## 1. Client-side rendering

With CSR, the server serves static application files and the browser downloads and executes JavaScript to render the page. Angular uses CSR by default.

CSR can fit highly interactive applications where users sign in before viewing most content, search indexing of the initial response is not important, and a static host or service worker can serve the application. The server does not render Angular for each route request.

The trade-off is that visitors may see little content until JavaScript has downloaded, parsed, and run. The initial experience depends on the browser device, network, and any additional requests made by the application.

## 2. Server-side rendering

With SSR, a server renders the requested route into HTML for each request and returns that HTML to the browser. This can show meaningful content before client JavaScript finishes loading. SSR can fit frequently changing public content or a route whose initial HTML depends on request-time data.

SSR adds server work and operational requirements. Hosting must support the Angular rendering runtime. Server capacity, data latency, caching, and failure handling become part of the page experience.

Server rendering does not make an interactive page work without browser JavaScript. The client application still needs to load and connect its behavior to the returned HTML. Hydration is covered in chapter 115.

## 3. Prerendering and static-site generation

Prerendering creates HTML at build time and writes it as static files. It is useful for content known when the build runs and shared by visitors, such as documentation, public course pages, or stable product information.

Prerendered files can be delivered by static hosting or a CDN without running Angular on each request. Content can become stale until a new build is deployed. Parameterized pages need a known set of paths or an appropriate fallback.

## 4. Compare the modes

| Mode | When HTML is created | Good fit | Main trade-off |
| --- | --- | --- | --- |
| CSR | In the visitor's browser | Authenticated dashboards and interactive apps served as static assets | Main content may wait for JavaScript |
| SSR | For each request on a server | Fresh or request-specific initial content | Server runtime and request cost |
| Prerender/SSG | At build time | Public pages with known content | Updates require a rebuild; paths must be generated |
| Hybrid | Per route, using a combination | Apps with public content and interactive or private areas | More configuration and deployment paths |

## 5. Angular hybrid rendering

Angular server route configuration supports Client, Server, and Prerender modes. A hybrid application could prerender documentation routes, use SSR for public search results that need fresh content, and keep a signed-in workspace client-rendered.

The actual route setup is covered in chapter 114. See Angular's [rendering strategies guide](https://angular.dev/guide/routing/rendering-strategies).

## 6. Data freshness and caching

Think about when data is selected:

- CSR commonly fetches data in the browser after the app starts.
- SSR can fetch data while rendering each request, subject to server-side cache rules.
- Prerendering uses build-time data, so the deployed HTML can become stale until rebuilt.

Caching changes practical freshness and cost. Public HTML may be cached at a CDN. Personalized responses need careful cache boundaries. Do not serve user-specific rendered HTML to other users through a shared public cache.

## 7. Performance and search visibility

SSR and SSG can return content sooner in HTML, but the result still depends on server response time, data latency, HTML size, JavaScript startup, and image delivery. Rendering content on a server does not automatically guarantee strong search visibility; metadata, links, content quality, accessibility, caching, and crawler behavior also matter.

Measure the route's real user experience before and after changing rendering mode. Include both the first response and the interactive experience.

## 8. Hosting and operational impact

- CSR can typically be served from static hosting.
- Prerendered output can be deployed as static files, often through a CDN.
- SSR needs a server or compatible platform runtime for routes configured for request-time rendering.
- Hybrid deployments need correct static asset and server routing behavior.

SSR adds monitoring, scaling, security updates, and failure handling. Prerendering many routes can increase build time and output size. Include those costs in the decision.

## 9. Route-by-route decision process

For each route, ask:

1. Must the first response contain the main content?
2. Is the content identical for every visitor at build time?
3. How fresh must it be?
4. Does rendering require user-specific credentials or request context?
5. Can the platform run an Angular rendering server?
6. Which response and interaction metrics will show whether the mode helped?

Choose CSR when browser rendering fits. Choose prerendering for stable content known at build time. Choose SSR when a fresh server-rendered response benefits the route and the server can produce it safely and reliably. Combine modes when route requirements differ.

## Common mistakes

- Enabling SSR everywhere without checking if request-time rendering is useful.
- Prerendering visitor-specific information.
- Sharing personalized SSR output through a public cache.
- Assuming server-rendered HTML is interactive before client JavaScript is ready.
- Forgetting that prerendered content can become stale.
- Choosing a mode without checking the hosting platform.

## Key takeaways

- CSR renders in the browser; SSR renders per request; prerendering renders at build time.
- Angular can select a rendering mode per route.
- SSR and SSG can improve initial content delivery, but do not guarantee better performance or SEO.
- Match data freshness and caching to whether content is public or user-specific.
- Choose from route requirements, measurements, and hosting capability.

## Further reading

- [Angular rendering strategies](https://angular.dev/guide/routing/rendering-strategies)
- [Angular server and hybrid rendering](https://angular.dev/guide/prerendering)
- [Angular SSR guide](https://angular.dev/guide/ssr)
- [Server-side rendering performance guide](https://angular.dev/best-practices/performance/ssr)