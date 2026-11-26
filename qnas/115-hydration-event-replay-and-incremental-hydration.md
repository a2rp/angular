# Hydration, Event Replay, and Incremental Hydration - Q&A

## 1. What is hydration?

Hydration is the client-side process that reuses server-rendered HTML and attaches Angular behavior, state, and event listeners to the existing DOM.

## 2. Is server-side rendering the same as hydration?

No. SSR renders HTML on the server for a request. Hydration runs in the browser afterward to make that HTML interactive. A server response can be correct even if client hydration later fails.

## 3. Does a prerendered page also need hydration?

If it is an interactive Angular application, yes. Prerendering creates HTML at build time; hydration connects the client application to that HTML in the browser.

## 4. Why reuse the server DOM instead of rendering again?

Reusing the DOM can avoid flicker, duplicate rendering work, and layout shifts that happen when the browser discards the server content and recreates it.

## 5. How is hydration enabled?

Angular CLI SSR setup commonly configures provideClientHydration for the app. Custom setups can add the provider manually. Check the generated configuration to avoid duplicate providers.

## 6. What is Event Replay?

Event Replay captures supported user events that happen before Angular finishes hydration and replays them after listeners are ready. It can prevent early clicks or other interactions from being lost.

## 7. Does Event Replay capture every possible browser event?

No. It supports a set of native events and interactions. Verify the specific controls and events your application relies on instead of assuming every arbitrary event is captured.

## 8. How do I enable Event Replay?

With current Angular, incremental hydration enables Event Replay. If incremental hydration is disabled but early event capture is still needed, use the withEventReplay hydration feature supported by the project's Angular version.

## 9. What is incremental hydration?

Incremental hydration leaves selected server-rendered sections dehydrated initially and hydrates them when a configured trigger fires. It uses @defer boundaries and can delay loading the client dependencies for a section.

## 10. Does incremental hydration remove server-rendered content until it is interactive?

No. For an incremental hydration block, Angular can render the main deferred content on the server while keeping its client-side section dehydrated. The HTML can be visible before the section's code is loaded.

## 11. How is hydrate on interaction different from on interaction?

A hydrate trigger controls when an SSR-rendered block becomes hydrated on the initial client load. A regular defer trigger controls when the block's content loads during client rendering, such as after a later client-side navigation.

## 12. What hydration triggers are available?

Angular currently documents idle, viewport, interaction, hover, immediate, timer, when, and never hydration triggers. Availability and exact syntax depend on Angular version; consult the current guide.

## 13. Why should a placeholder still be provided?

The placeholder can be used when the block is reached during a later client-side navigation, where regular defer behavior applies. It also provides a fallback before a block loads and can help reserve space.

## 14. What happens with hydrate never?

The section remains static during the initial SSR load and its subtree is not hydrated then. A later client-side render can still load and hydrate the defer block according to its normal triggers.

## 15. Can a child hydration trigger fire before its parent?

Angular's view hierarchy requires parent boundaries to hydrate before a nested child. If a nested child is triggered, Angular hydrates the necessary dehydrated ancestors first.

## 16. What causes hydration mismatch errors?

Common causes include invalid HTML nesting, direct DOM changes before hydration, third-party scripts that modify markup, CDN HTML transformations, browser extensions, or different server and client rendering conditions.

## 17. Why is direct DOM mutation risky before hydration?

Angular expects the DOM tree produced by its server rendering. If code inserts, removes, or moves nodes before the client hydrates, the actual structure may not match Angular's expected structure.

## 18. Is ngSkipHydration a normal fix?

No. It is an escape hatch for a specific incompatible component subtree. Use it only on a component host and as a temporary workaround while removing the underlying incompatibility.

## 19. Does ngSkipHydration skip just one element?

It skips hydration for the component host and its subtree. Angular recreates that area in the browser, which can add work and cause visible changes.

## 20. How do I debug a hydration error?

Inspect the browser console, validate the rendered HTML, search for direct DOM manipulation and third-party transformations, compare server and client build settings, then use Angular DevTools to inspect hydration status. Reproduce on the actual SSR or prerendered route.

## 21. Can a CDN cause hydration problems?

Yes. Some CDN features alter HTML, whitespace, or comments before the browser receives it. If errors occur only in production, compare the server output with the HTML delivered through the CDN and disable transformations that change Angular's expected DOM.

## 22. What is the performance trade-off of incremental hydration?

It can reduce the client JavaScript required up front, but a section may be visible before its behavior is ready. Choose a trigger based on user expectations and measure when that section becomes interactive.

## 23. Should every section hydrate on interaction?

No. Critical controls should become interactive promptly. Static or below-the-fold content may be suitable for delayed hydration. Select triggers based on how and when users need each section.

## 24. How should I verify hydration works?

Load an SSR or prerendered route directly, inspect the HTML, wait for client startup, and interact with important controls. Test early interactions during startup if Event Replay is enabled. Check console output and layout stability.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="114-set-up-server-side-rendering-and-hybrid-route-rendering.md">114 - Set Up Server-Side Rendering and Hybrid Route Rendering</a></td><td><a href="../lessons/115-hydration-event-replay-and-incremental-hydration.md">Lesson</a></td><td><a href="../examples/115-hydration-event-replay-and-incremental-hydration.md">Example</a></td><td><a href="116-server-compatible-code-metadata-and-search-visibility.md">116 - Server-Compatible Code, Metadata, and Search Visibility</a></td>
</tr>
  </tbody>
</table>
