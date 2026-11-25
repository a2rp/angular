# Lazy Loading and Deferrable Views with defer — Q&A

## 1. What is lazy loading?

Lazy loading downloads application code only when a feature is needed instead of putting all of it into the startup bundle. Angular routing can lazy-load route components or child route trees, and deferrable views can lazy-load a section inside a page.

## 2. What is the difference between lazy routes and @defer?

A lazy route is a page or route-tree boundary. Angular requests its code when navigation activates that route. An @defer block is a section within a component template that loads eligible dependencies when a trigger occurs.

## 3. What do loadComponent and loadChildren do?

loadComponent lazily loads a route's component. loadChildren lazily loads its child route configuration, which can itself contain additional lazy components or children. Both commonly use dynamic import expressions.

## 4. Should every route be lazy-loaded?

No. Keep primary landing pages and routes required immediately available as appropriate. Lazy loading every tiny route can add network round trips and complexity. Decide from startup bundle evidence, route use, and first-navigation experience.

## 5. What is a deferrable view?

A deferrable view is an Angular template section declared with @defer. Angular can split eligible standalone components, directives, and pipes in that block into separate chunks and load them on demand.

## 6. What trigger does @defer use by default?

The default trigger is idle, meaning the browser can load the content when it becomes idle. Use an explicit trigger when the user flow needs a different timing, such as when a section enters the viewport or a user requests it.

## 7. What triggers are available?

Common built-in triggers include idle, viewport, interaction, hover, immediate, and timer. A when trigger loads when a condition becomes truthy. Trigger details and available options can evolve, so consult the Angular documentation for the project's version.

## 8. Does a when block return to its placeholder when the condition becomes false?

No. It is a one-time transition. Once the condition becomes truthy and the deferred content loads, a later false value does not restore the placeholder.

## 9. What are @placeholder, @loading, and @error for?

- @placeholder shows content before the defer trigger occurs.
- @loading shows content while deferred dependencies are loading.
- @error shows a fallback if the deferred loading fails.

These states improve clarity and resilience. Keep their dependencies lightweight because the state content is not deferred by the same block.

## 10. Can I omit the placeholder?

Yes, but the block will show no content before it loads unless a placeholder is provided. Certain triggers or usages may require a template reference element or placeholder. A visible stable placeholder is often better for layout and user understanding.

## 11. What does the minimum parameter do?

A minimum display duration can prevent the placeholder or loading state from flashing briefly when a chunk loads very quickly. Use it sparingly; an unnecessarily long minimum can make a fast feature feel slower.

## 12. What does the after parameter do in @loading?

It delays showing the loading state after loading begins. This can avoid displaying a spinner for a very short wait. It does not delay the load itself.

## 13. Which dependencies can Angular defer?

Components, directives, and pipes in the defer block can be split when they meet Angular's defer requirements. In particular, a dependency must be standalone, and it must not be referenced outside defer blocks in the same file or through a view query that makes it eager.

## 14. Do all transitive dependencies have to be standalone?

No. The directly deferred dependencies need to meet the standalone requirement. Their transitive dependencies may still be declared in an NgModule and can participate in the deferred chunk.

## 15. Why did @defer not create a separate chunk?

Check whether the deferred dependency is referenced elsewhere in the same file, imported through a barrel that groups modules, or is not standalone. Also inspect the production build output; development HMR may fetch defer chunks eagerly and obscure normal trigger behavior.

## 16. What is a barrel file and why can it affect splitting?

A barrel file re-exports symbols from other files, commonly through an index file. Importing a deferred component through a barrel can cause the bundler to group exports together. Import directly from the component file and verify the output chunk.

## 17. What is prefetching?

Prefetching downloads deferred code before the display trigger fires, using a separate prefetch trigger. For example, code can be fetched while the browser is idle and shown after a user interaction. Prefetching can reduce perceived wait time but still consumes bandwidth and cache resources.

## 18. Does prefetching also load the data the component needs?

Not automatically. Prefetch triggers concern deferred code. The component's API data and other runtime resources need their own loading strategy.

## 19. Can @defer harm CLS?

Yes. If a large visible section is replaced by content with a different size, the page can shift. Avoid deferring above-the-fold content when possible. If necessary, reserve a stable area and measure layout shifts.

## 20. How should I make deferred content accessible?

Provide meaningful status text for important loading transitions and consider a concise polite live region. Ensure keyboard focus remains predictable when content replaces a placeholder. Avoid announcements for purely decorative changes and avoid repeatedly announcing the entire section.

## 21. How does @defer behave with SSR or static generation?

By default, server-rendered or statically generated output contains the placeholder (or nothing) for a defer block; the trigger runs on the client. Hydration features can alter this for configured blocks. Confirm behavior with the rendering strategy used by the app.

## 22. Why does development behavior differ from production?

With hot module replacement, defer chunks may be fetched eagerly to support updates. Development tooling can therefore override configured trigger timing. Verify chunk splitting and network behavior using a production build or a suitable preview.

## 23. How can I verify that a split worked?

Build the app using its production configuration, inspect emitted chunks, then use the browser Network panel to see when the chunk is requested. Verify both the startup bundle and the first-use experience.

## 24. When should I use a route boundary versus a defer block?

Use route lazy loading when a distinct page or child route is not needed until navigation. Use @defer for a substantial optional section inside an already-loaded page. Avoid splitting tiny pieces that create more requests and complexity than they save.

## 25. What is a reliable loading strategy?

Keep critical content promptly available, defer optional code at a meaningful boundary, show stable placeholder/loading/error states, avoid nested request waterfalls, and measure both initial load and first feature use.

## Further reading

- [Angular deferrable views](https://angular.dev/guide/templates/defer)
- [Angular lazy-loaded routes](https://angular.dev/best-practices/performance/lazy-loaded-routes)
- [Angular performance best practices](https://angular.dev/best-practices/performance)