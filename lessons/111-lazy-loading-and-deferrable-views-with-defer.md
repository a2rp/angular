# Lazy Loading and Deferrable Views with defer

Lazy loading keeps code out of the startup path until the user needs it. Angular routing can load a route component or child route tree on navigation. Deferrable views can load a section inside an already-open page when a trigger occurs.

Both approaches can reduce the initial JavaScript, but they move some work to a later moment. Choose the moment carefully: a feature that appears late can feel slower or shift the layout if its space and loading state are not planned.

## Learning outcomes

After this lesson, you should be able to:

- Explain the difference between route lazy loading and a deferred view.
- Configure lazy route components and child route trees.
- Use an Angular @defer block with placeholder, loading, and error states.
- Select idle, viewport, interaction, or condition-based triggers.
- Understand prefetching and the conditions required for dependencies to split.
- Avoid common layout, accessibility, and waterfall problems.

## 1. Route-level lazy loading

A route declared with a direct component reference is included with the eagerly imported route configuration. For a feature that does not need to be available on the first screen, use loadComponent or loadChildren with a dynamic import.

~~~ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./home/home-page').then((module) => module.HomePage),
  },
  {
    path: 'reports',
    loadChildren: () => import('./reports/reports.routes'),
  },
];
~~~

A route file can export its child route collection:

~~~ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./reports-page').then((module) => module.ReportsPage),
  },
  {
    path: 'detail/:id',
    loadComponent: () =>
      import('./report-detail-page').then((module) => module.ReportDetailPage),
  },
];
~~~

The router loads the route chunk when navigation requires it. This can reduce startup code, but users may wait for a request when they first visit that feature. Keep the primary landing page and critical initial content available promptly, and check the generated build output to confirm the split.

See [Angular's lazy-loaded routes guide](https://angular.dev/best-practices/performance/lazy-loaded-routes).

## 2. Deferrable views

The @defer block splits eligible dependencies from the containing component and loads them when its trigger fires. It is useful for a section that is not needed immediately, such as comments below an article or an optional analytics panel.

~~~html
@defer {
  <app-recommendations />
} @placeholder {
  <section aria-label="Recommendations">
    Recommendations will appear here.
  </section>
} @loading (after 100ms; minimum 300ms) {
  <p aria-live="polite">Loading recommendations…</p>
} @error {
  <p>Recommendations could not be loaded.</p>
}
~~~

The placeholder, loading, and error sections make the states explicit. Their own dependencies are not deferred by this block, so keep those states lightweight.

## 3. Requirements for a real code split

A component, directive, or pipe used in the deferred block can be split only when Angular can isolate its code. The dependency must be standalone, and the same dependency must not be referenced eagerly elsewhere in the same file or through a view query that requires it. Non-standalone dependencies are not deferred by the block.

Import the deferred component directly from its own file. A barrel import can cause bundlers to group unrelated exports together, so the expected chunk may not be created. Always verify the production build output instead of assuming a split happened.

## 4. Choose a trigger that fits the user flow

Angular supports built-in triggers such as:

- **idle:** load when the browser is idle; this is the default trigger.
- **viewport:** load when the placeholder or referenced element enters the viewport.
- **interaction:** load after a user interacts with a referenced element.
- **hover:** load when a user hovers or focuses the placeholder or referenced element.
- **immediate:** load as soon as the rest of the view has rendered.
- **timer:** load after a specified duration.
- **when:** load when an expression becomes truthy.

An example of an interaction trigger:

~~~html
<button #showComments type="button">Show comments</button>

@defer (on interaction(showComments)) {
  <app-comments />
} @placeholder {
  <p>Comments are not loaded yet.</p>
}
~~~

A when trigger is a one-time transition: when its expression becomes truthy, Angular loads the deferred content. If it later becomes false, the block does not revert to the placeholder.

Do not use a timer simply because it is easy. Pick a trigger that preserves the user's expectation and does not postpone content that should already be visible.

## 5. Prefetch before display

A prefetch trigger can download the deferred code before the main trigger displays it. For example, an application may prefetch when the browser is idle and display only when the user interacts. This can make the interaction faster while keeping the initial page smaller, but it still uses network and cache resources.

Prefetching code does not guarantee the server data required by the component has also been fetched. Plan data loading and error handling separately.

## 6. Prevent layout shifts and accessibility gaps

Avoid deferring content that is visible in the initial viewport unless the placeholder reserves the same space and the design accounts for the transition. A large placeholder-to-content height change can worsen CLS.

Screen readers may announce the placeholder but not automatically announce the arriving deferred content. Use a concise live region when the transition conveys important information. Avoid duplicating announcements for decorative content. Keep keyboard focus predictable when a user-triggered defer block replaces a control or placeholder.

## 7. Avoid nested loading waterfalls

If nested deferred sections all load on the same trigger, the application can create a cascade of chunk requests and data requests. Each section should have a distinct reason to load. Avoid splitting code so finely that a normal user action requires several dependent network round trips.

Use route-level lazy loading for page-sized feature boundaries and @defer for meaningful sections within a page. The separation is a design aid, not a rule that every component must be deferred.

## 8. Development and server rendering behavior

With hot module replacement enabled, development tooling may fetch defer chunks eagerly, so the browser behavior may not match the production trigger timing. Verify actual chunk behavior with a production build or a suitable preview.

For server-side rendering and static generation, defer blocks normally render their placeholder (or nothing if no placeholder exists) on the server; triggers run on the client. A later chapter covers hydration options that can change this behavior.

## 9. A practical loading workflow

1. Inspect the initial production bundle and route chunks.
2. Identify code that is not required for the first useful view.
3. Choose a route boundary or a view-level section boundary.
4. Add a loader state, error state, and reserved layout space where needed.
5. Trigger loading at a user-appropriate time.
6. Verify the split and runtime request in build output and browser network tools.
7. Measure startup and first-use behavior again.

## Key takeaways

- Lazy routes defer entire page features until navigation; @defer postpones eligible sections within a page.
- Route and defer loaders create later requests, so consider first-use latency.
- Standalone dependencies, direct imports, and no eager references are important for defer splitting.
- Add stable placeholders and accessible loading or error states.
- Avoid deferring above-the-fold content in a way that causes layout shift.
- Inspect a production build to confirm the intended chunks exist.

## Further reading

- [Deferrable views](https://angular.dev/guide/templates/defer)
- [Lazy-loaded routes](https://angular.dev/best-practices/performance/lazy-loaded-routes)
- [Angular performance best practices](https://angular.dev/best-practices/performance)
- [Angular template control flow](https://angular.dev/guide/templates/control-flow)