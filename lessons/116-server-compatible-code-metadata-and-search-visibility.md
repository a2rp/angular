# Server-Compatible Code, Metadata, and Search Visibility

Code that runs during server rendering or prerendering cannot assume a browser exists. Globals such as window, navigator, location, or browser-only libraries can fail during a server request or build. Server-compatible code keeps initial rendering consistent across the server and browser, then initializes genuinely browser-only behavior at the right lifecycle point.

Search visibility also depends on what a crawler receives. Angular can render useful page titles and metadata into the server or prerendered HTML, but metadata is only one part of discoverability.

## Learning outcomes

- Identify browser-only APIs and packages that can break SSR or prerendering.
- Keep server and browser output structurally consistent for hydration.
- Isolate platform-specific behavior behind services or providers.
- Use render hooks carefully for browser-only DOM work.
- Set route titles and metadata that appear in the initial HTML.
- Understand the limits of metadata and client-side rendering for search visibility.

## 1. Do not assume browser globals exist

Server rendering and prerendering execute application code outside a browser. These globals may be unavailable:

- window
- document
- navigator
- location
- HTMLElement and layout properties such as offsetWidth

A library can also access these globals at module initialization time, before a component can check which platform it is running on. Verify that SSR dependencies support server use, and avoid importing browser-only code in a path evaluated during server rendering.

## 2. Keep server and browser markup consistent

Hydration expects the browser DOM to correspond to the structure rendered on the server. Avoid using a platform check in a template to produce different initial content:

~~~html
@if (isBrowser) {
  <p>Browser-only content</p>
} @else {
  <p>Server-only content</p>
}
~~~

This pattern can create a mismatch and layout shift. Render a consistent initial structure, then update it through a deliberate client-side state change after the initial render when necessary.

## 3. Isolate platform-specific behavior

When browser and server behavior differ, keep the difference behind an injected service or provider. The component can depend on a shared abstraction while each platform supplies a suitable implementation.

This makes the platform boundary explicit, avoids scattering runtime checks through templates, and allows server rendering to return a safe result without requiring browser APIs.

Prefer platform-specific providers for logic that is fundamentally different on the server and browser. Do not create a server substitute that pretends to support DOM layout or browser storage.

## 4. Use render hooks for browser-only DOM work

Angular's afterNextRender and afterEveryRender callbacks run on browser platforms and are skipped during server rendering. They can be used for DOM work such as initializing a chart library that requires a real element.

However, afterNextRender does not guarantee that a component is fully hydrated before the callback runs. Read and write the DOM carefully, prefer Angular bindings, and follow the render-hook phases supported by the Angular version.

Do not use render hooks as a place to render different initial markup for the server and client. Keep the initial DOM compatible, then enhance it in the browser.

## 5. Route titles

Define a meaningful title for every important route. Angular Router updates the document title from the active route's title property.

A title can be a static string or a resolver when the page name depends on route data. A custom TitleStrategy can apply a consistent application suffix or compose a title from route information.

A useful title is concise, distinct for each page, and descriptive to both the user and browser history. Avoid making every route use the same generic title.

## 6. Meta descriptions and social metadata

Angular's Meta service can add or update standard meta tags and social sharing metadata such as Open Graph properties. For a public article, metadata might include:

- A concise description of the page.
- An Open Graph title, description, and preview image.
- A canonical URL when the deployment has multiple equivalent URLs.

Populate metadata from trusted page data, and ensure route-specific values are applied before the server or prerendered response is produced. Set a safe default in the document shell and replace it for individual routes.

A meta description is a summary shown by some search results; search engines may choose different text. Open Graph metadata supports link previews in participating platforms. Neither guarantees ranking or a particular preview.

## 7. Metadata must reach the initial response

If metadata matters for a route, inspect the HTML returned by the server or generated during prerendering. Metadata created only after a client-side API call may not be present in the initial response.

For SSR, provide the data needed for the title and tags as part of the server render. For prerendering, make the information available during the build. Do not place private or personalized values into public generated HTML.

## 8. Search visibility is broader than metadata

A robust search-friendly page also needs:

- Useful, crawlable page content in the initial HTML where appropriate.
- Unique route URLs and meaningful links between pages.
- Correct title and description metadata.
- A sensible status code and redirect behavior.
- Canonical URL handling where duplicate routes exist.
- Fast, accessible, mobile-friendly content.

Search engines vary in their ability and willingness to execute client JavaScript. SSR or prerendering can make content available in HTML, but search results are not guaranteed.

## 9. Common mistakes

- Reading window or localStorage during top-level module evaluation.
- Calling browser APIs in constructors that also run during SSR.
- Rendering completely different server and client templates.
- Assuming afterNextRender runs only after hydration is complete.
- Applying all routes the same title and description.
- Updating metadata after the response has already been sent.
- Including user-specific metadata in shared prerendered output.
- Treating metadata as a guarantee of ranking or social preview behavior.

## 10. A verification workflow

1. Build or serve the application with its SSR/prerender configuration.
2. Request the route directly, not only through client navigation.
3. Inspect the returned HTML for page content, title, and meta tags.
4. Check the browser console for hydration mismatch errors.
5. Exercise browser-only enhancements after the initial render.
6. Verify that static output contains only public, build-time data.
7. Test status codes, redirects, and canonical URL behavior at the deployed host.

## Key takeaways

- SSR and prerender code must not assume browser globals are available.
- Keep initial server and browser DOM structure consistent.
- Isolate platform-specific behavior behind providers or services.
- Use render hooks carefully; browser-only does not necessarily mean post-hydration.
- Titles and metadata should be route-specific and present in the initial HTML.
- Search visibility depends on content, links, accessibility, status behavior, performance, and metadata; no single API guarantees it.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="115-hydration-event-replay-and-incremental-hydration.md">115 - Hydration, Event Replay, and Incremental Hydration</a></td><td><a href="../qnas/116-server-compatible-code-metadata-and-search-visibility.md">Q&amp;A</a></td><td><a href="../examples/116-server-compatible-code-metadata-and-search-visibility.md">Example</a></td><td><a href="117-security-xss-sanitization-csp-and-trusted-content.md">117 - Security: XSS, Sanitization, CSP, and Trusted Content</a></td>
</tr>
  </tbody>
</table>
