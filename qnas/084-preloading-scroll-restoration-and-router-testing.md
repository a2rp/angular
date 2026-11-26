# Preloading, Scroll Restoration, and Router Testing — Q&A

## Preloading

### 1. What is route preloading?

Route preloading loads a lazy route's code after the application has started and the router is idle enough to do so. The route remains lazy for the initial application load, but its code may be ready sooner when the user later navigates to it.

### 2. How is preloading different from lazy loading?

Lazy loading defers downloading and evaluating route code until needed. Preloading is an optional follow-up that fetches some lazy route code in the background after startup. A route can be lazy-loaded without being preloaded.

### 3. What is Angular Router's default preloading behavior?

The default is NoPreloading. Lazy routes load when navigation first requests them. To preload routes, configure a preloading strategy with the router.

### 4. How do I preload every eligible lazy route?

Configure PreloadAllModules with withPreloading:

~~~ts
provideRouter(
  routes,
  withPreloading(PreloadAllModules)
)
~~~

This is convenient for small applications, but it can download code a user never visits. Consider bandwidth, startup network activity, and memory before enabling it broadly.

### 5. Does PreloadAllModules make every route eager?

No. The route still uses its lazy loading declaration. Preloading requests the lazy route code after startup; it does not turn that route into an eagerly imported route in the initial bundle.

### 6. How can I preload only selected routes?

Use a custom PreloadingStrategy and mark routes with route data:

~~~ts
@Injectable({ providedIn: 'root' })
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<unknown>): Observable<unknown> {
    return route.data?.['preload'] === true ? load() : of(null);
  }
}
~~~

~~~ts
const routes: Routes = [
  {
    path: 'reports',
    loadChildren: () => import('./reports/reports.routes').then(m => m.REPORT_ROUTES),
    data: { preload: true }
  }
];
~~~

~~~ts
provideRouter(routes, withPreloading(SelectivePreloadingStrategy))
~~~

The Observable type returned by a strategy can depend on the Angular version and the route loader type in the application. Keep the strategy aligned with the installed Angular Router types.

### 7. Can I preload based on user role, network, or application state?

Yes. A custom strategy can consult an injected service or browser capability before calling the supplied load function. Keep the decision predictable, avoid loading protected data as part of preloading, and remember that route-code preloading does not replace authorization checks in guards or on the server.

### 8. Does preloading execute route guards or resolvers?

Preloading is about fetching lazy route code. It is not a successful navigation and should not be treated as running a route's normal activation flow. Guards and resolvers still need to be correct when the user actually navigates.

### 9. How do I know whether preloading is useful?

Measure it. Compare route navigation latency and network requests before and after enabling a strategy. Prefer selectively preloading routes likely to be used soon, and consider user data, device constraints, and connection quality.

### 10. What if preloading fails?

The application should still handle a later navigation failure gracefully. Check the network and console, verify the lazy import path and build output, and ensure any custom strategy returns the loader Observable when appropriate. Do not assume that a background preload guarantees a route is available.

## Scroll behavior

### 11. What does scrollPositionRestoration do?

It configures the router's scroll position behavior after navigation. With enabled, Angular restores a saved position when navigating back or forward; for a new navigation it scrolls to a fragment when configured and present, otherwise it scrolls to the top. With top, it goes to the top after each navigation. The default disabled setting leaves scroll position unchanged.

~~~ts
provideRouter(
  routes,
  withInMemoryScrolling({
    scrollPositionRestoration: 'enabled',
    anchorScrolling: 'enabled'
  })
)
~~~

### 12. What is anchor scrolling?

Anchor scrolling moves the viewport to an element identified by the URL fragment, such as /guide#installation. Enable it with anchorScrolling: 'enabled'. The element needs a matching id in the rendered page.

### 13. Does anchor scrolling run when the user presses Back or Forward?

No. Angular's built-in anchor scrolling does not run on popstate navigation. When scrollPositionRestoration is enabled, the router restores the saved scroll position for that history entry instead.

### 14. What is the difference between disabled and top?

disabled preserves the current viewport position. top moves to the top for each navigation. enabled restores a saved position on backward or forward navigation and uses the new-navigation behavior for other navigations.

### 15. How do I account for a fixed header when scrolling to anchors?

Use ViewportScroller's offset, for example in the root component:

~~~ts
constructor(viewportScroller: ViewportScroller) {
  viewportScroller.setOffset([0, 72]);
}
~~~

Choose an offset that matches the actual fixed header size, and verify it at responsive breakpoints. An offset cannot fix an anchor that does not exist yet when scrolling occurs.

### 16. Why does an anchor sometimes fail to scroll?

Check that the URL contains the expected fragment, the target element has the same id, the target is in the rendered DOM when navigation completes, and the scrolling feature is enabled. Also inspect sticky headers, nested scrolling containers, and custom code that changes scroll position.

### 17. Does Angular Router manage scrolling inside every scrollable container?

No. Router in-memory scrolling is intended for the document viewport. A nested element with overflow scrolling usually needs application-specific logic and tests.

## Router testing

### 18. What does RouterTestingHarness do?

RouterTestingHarness provides a test-friendly way to create a router, navigate to a URL, and inspect the component activated in the primary outlet. It reduces manual setup for common route tests.

### 19. How do I test a route with RouterTestingHarness?

Configure the test with provideRouter and the route components, create the harness, navigate, and assert the activated component or rendered content:

~~~ts
TestBed.configureTestingModule({
  imports: [HomePage, AboutPage],
  providers: [
    provideRouter([
      { path: '', component: HomePage },
      { path: 'about', component: AboutPage }
    ])
  ]
});

const harness = await RouterTestingHarness.create();
await harness.navigateByUrl('/about', AboutPage);

expect(harness.routeNativeElement?.textContent).toContain('About');
~~~

Use the test runner and assertion syntax already configured by the project; Angular projects may use different runners.

### 20. What does navigateByUrl(url, ExpectedComponent) verify?

It navigates through the router and checks that the routed component has the expected type. Assertions on routeNativeElement can then verify visible output. Add explicit assertions for details such as route data, parameters, or service calls when those are part of the behavior being tested.

### 21. How do I test route parameters?

Navigate with a URL containing the parameter, then assert the rendered component behavior or its injected ActivatedRoute data. Prefer checking user-visible behavior over coupling every test to private implementation details.

### 22. How do I test redirects, guards, or resolvers?

Define the relevant route configuration and dependencies in the test, navigate to the starting URL, and assert the final route or visible result. For guards, cover both allowed and denied outcomes. For resolvers, verify the component receives the resolved data and that failure behavior is intentional.

### 23. When do I need a host component instead of RouterTestingHarness?

RouterTestingHarness is designed for common primary-outlet routing. Complex layouts, named outlets, or a specific parent component's router outlets may need a normal TestBed host component with RouterOutlet and explicit router setup.

### 24. Should a router test use the real browser?

Most routing behavior can be tested with Angular's test environment. Use a browser-level end-to-end test when the behavior depends on actual browser history, viewport scrolling, layout, focus, or user interaction across the whole application.

### 25. How should I test scroll restoration?

Separate configuration and application logic from browser-specific behavior. Verify the configured router options and any ViewportScroller interaction in unit-level tests where practical; use a browser test to confirm actual back/forward restoration, anchor position, fixed-header offset, and nested scrolling behavior.

### 26. Should preloading be tested with a live network?

No. Unit-test a custom strategy by controlling its loader and dependencies, and verify whether it invokes the loader for selected routes. Use browser or integration checks only when validating actual bundling and network behavior.

### 27. Why can a routing test pass while navigation still fails in production?

A test may omit providers, lazy chunk delivery, base URL configuration, server fallback behavior, deployment path handling, or browser history conditions. Test routing logic in isolation, then verify build and deployment assumptions in the environment where the application runs.

## Quick decision guide

| Need | Suitable approach |
| --- | --- |
| Keep all lazy routes on demand | NoPreloading |
| Warm all lazy routes after startup | PreloadAllModules, after considering network and memory cost |
| Warm only chosen routes | Custom PreloadingStrategy and route data |
| Restore Back or Forward scroll position | scrollPositionRestoration: 'enabled' |
| Reset viewport to the top after navigation | scrollPositionRestoration: 'top' |
| Navigate to URL fragments | anchorScrolling: 'enabled' and matching element ids |
| Test a normal routed component | RouterTestingHarness |
| Test named outlets or full browser viewport behavior | Host-component or browser-level test as appropriate |

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="083-page-titles-redirects-wildcard-routes-and-not-found-pages.md">083 - 83. Page titles, redirects, wildcard routes, and not-found pages</a></td><td><a href="../lessons/084-preloading-scroll-restoration-and-router-testing.md">Lesson</a></td><td><a href="../examples/084-preloading-scroll-restoration-and-router-testing.md">Example</a></td><td><a href="085-configure-httpclient-with-providehttpclient.md">085 - Configure HttpClient with provideHttpClient</a></td>
</tr>
  </tbody>
</table>
