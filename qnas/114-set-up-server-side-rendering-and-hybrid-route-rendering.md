# Set Up Server-Side Rendering and Hybrid Route Rendering - Q&A

## 1. How do I add SSR to an existing Angular application?

Run ng add @angular/ssr from the application workspace with the CLI version aligned to the project. Review the generated package, bootstrap files, server routes, and build configuration. File names and setup can differ across Angular versions.

## 2. What does the SSR schematic generate?

It can add a server bootstrap, server application configuration, server route configuration, build and serve target changes, and a server integration. Inspect the actual workspace after running it; do not assume every project receives exactly the same structure.

## 3. What are Angular server routes?

ServerRoute entries map URL paths to rendering modes and, where supported, related configuration such as prerender parameters, response status, or headers. They let a hybrid application choose rendering behavior by route.

## 4. What are the route rendering mode values?

Current Angular supports Client, Server, and Prerender. Client renders in the browser, Server renders for each request, and Prerender generates HTML at build time.

## 5. Why should specific routes come before a wildcard?

A wildcard can match paths not covered by more specific patterns. Put specific routes first so they receive the intended mode before a catch-all behavior applies.

## 6. How are server routes connected to the application?

The generated server application configuration commonly uses provideServerRendering with withRoutes(serverRoutes). Keep the generated providers and bootstrap aligned with the current Angular version.

## 7. How does getPrerenderParams work?

For a parameterized route configured for Prerender, getPrerenderParams returns the parameter combinations Angular should generate at build time. A route such as articles/:slug can return one object for each known slug.

## 8. Can getPrerenderParams use inject?

Yes, when called synchronously before the first await inside its injection context. Store the injected service first, then await its data. Do not call inject after an await or inside an unrelated callback.

## 9. What kind of data belongs in a prerender route?

Use public, build-time data that is suitable to share among all visitors. Do not place secrets, session data, or user-specific content into generated static HTML.

## 10. What if a requested prerender path was not generated?

Choose a prerender fallback. Current Angular documentation describes Server, Client, and None strategies, with Server as the default when no fallback is specified. The correct choice depends on whether the deployment has a rendering server and how unknown paths should behave.

## 11. Can a static host use Server fallback?

No. A static file host cannot perform request-time Angular rendering. For static-only deployments, generate all supported routes or configure a client-side fallback that the host can serve. Do not configure a server fallback without a server runtime.

## 12. What is the difference between outputMode static and request-time SSR?

Static output generates static assets and prerendered HTML without an Angular server file for handling requests. Request-time SSR produces server output that must run in a compatible server or platform runtime. Choose a build and deployment shape that supports the selected route modes.

## 13. How do I verify an SSR route?

Request the URL directly in a fresh browser navigation or with an HTTP client. Inspect the returned HTML, server response status, headers, and logs. Internal client navigation alone does not prove the server can render the route.

## 14. How do I verify prerendering?

Build the project and inspect the generated output for the expected HTML path. Request that path directly from the local preview or deployment. Also check how the host handles trailing slashes and unknown paths.

## 15. Does ng serve prove production SSR is configured?

It can help verify development behavior, but it does not prove the production server bundle is deployed correctly. Build using the production configuration and run the output through the platform's supported runtime or preview.

## 16. Can I set status codes and response headers per route?

Server route definitions support status and headers for configured server routes in current Angular. Use them for the intended HTTP behavior and verify the response at the deployed boundary. Avoid placing secrets or user data in headers.

## 17. Can prerender and SSR routes coexist?

Yes. A hybrid application can prerender stable pages, server-render routes that need fresh request-time content, and client-render other routes. Ensure the output and host support every selected mode.

## 18. What happens if a prerender route has too many parameter values?

The build can take longer and generate a larger deployment. Reassess whether every path needs prebuilt HTML. Options include limiting static paths or using a suitable runtime-rendering fallback when the deployment supports it.

## 19. What should I check when deployment works for the home page but not deep links?

Check the static host's rewrite rules, generated route files, server fallback, and route mode configuration. Direct requests to a route need to reach either its prerendered file or the Angular rendering server.

## 20. Does a successful server response mean hydration works?

No. It confirms that the server produced a response. The browser must still initialize the client application and connect it to the rendered DOM. Verify interactions and hydration behavior separately.

## 21. Can server rendering safely use shared caching?

Only when the response is safe to share. Public HTML can often be cached with an appropriate policy. User-specific responses require a careful cache boundary or private/no-store handling so one visitor cannot receive another visitor's HTML.

## 22. How should I preserve CLI-generated files during an upgrade?

Review Angular migration guidance and compare generated files against the new version. Avoid copying an old custom server setup over the generated current one without understanding API and runtime changes.

## 23. What if getPrerenderParams needs private authentication?

That route is usually a poor fit for public prerendering. Build-time HTML is shared output. Use an appropriate request-time or client-side flow for user-specific data and protect the response and cache accordingly.

## 24. What should I verify before choosing a deployment platform?

Confirm that it supports the build's output type, direct route requests, static asset serving, request-time execution if SSR routes exist, required Node/runtime versions, and the desired cache behavior.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="113-client-rendering-server-rendering-and-prerendering.md">113 - Client Rendering, Server Rendering, and Prerendering</a></td><td><a href="../lessons/114-set-up-server-side-rendering-and-hybrid-route-rendering.md">Lesson</a></td><td><a href="../examples/114-set-up-server-side-rendering-and-hybrid-route-rendering.md">Example</a></td><td><a href="115-hydration-event-replay-and-incremental-hydration.md">115 - Hydration, Event Replay, and Incremental Hydration</a></td>
</tr>
  </tbody>
</table>
