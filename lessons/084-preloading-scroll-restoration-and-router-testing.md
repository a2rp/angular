# 84. Preloading, scroll restoration, and router testing

## What you will learn

After this lesson, you should be able to:

- Explain how preloading works with lazy route configuration.
- Choose between no preloading, preloading every lazy route, and selective preloading.
- Configure anchor scrolling and scroll-position restoration.
- Know how scroll restoration behaves during normal navigation and browser back/forward.
- Use Angular's RouterTestingHarness to test routed components and navigation.
- Test route parameters, redirects, and guards through a real router configuration.
- Recognize when a router test needs a custom host component.

## 1. Lazy loading and preloading solve different timing problems

Lazy loading splits route code so it can be requested when needed rather than being part of the application's initial code load. A lazy route commonly uses loadChildren:

~~~ts
{
  path: 'reports',
  loadChildren: () =>
    import('./reports/reports.routes').then((module) => module.REPORT_ROUTES),
}
~~~

Without preloading, the route configuration and its code are loaded when a user first navigates to that lazy route. That can make the first visit slower.

Preloading asks Angular to load lazy route code in the background after the initial navigation. It can make a later first visit to a feature faster, at the cost of downloading and holding more code.

Preloading does not mean that every route component is activated, nor does it mean that the feature's API data has been fetched. It loads lazy route code according to a strategy.

## 2. Built-in preloading strategies

Angular provides two built-in strategies:

- NoPreloading is the default. Lazy routes load when the user navigates to them.
- PreloadAllModules requests all lazy-loaded route modules after initial navigation.

Configure a built-in strategy with withPreloading:

~~~ts
import {
  PreloadAllModules,
  provideRouter,
  withPreloading,
} from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withPreloading(PreloadAllModules),
    ),
  ],
};
~~~

Use NoPreloading when you want to avoid background route downloads. Use PreloadAllModules when the app is small enough and the expected navigation benefit justifies its extra network and memory use.

Preloading still preserves the lazy route boundary for initial loading. It does not make all feature code part of the initial application navigation, though the browser may fetch those lazy chunks shortly afterward.

## 3. Selectively preload feature routes

For a larger application, routes can opt into preloading through route data. A custom PreloadingStrategy decides which route load callbacks to run.

~~~ts
import { Injectable } from '@angular/core';
import {
  PreloadingStrategy,
  Route,
} from '@angular/router';
import { Observable, of } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class SelectivePreloadingStrategy
  implements PreloadingStrategy {
  preload(
    route: Route,
    load: () => Observable<any>,
  ): Observable<any> {
    return route.data?.['preload'] === true
      ? load()
      : of(null);
  }
}
~~~

Mark only the desired lazy routes:

~~~ts
export const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () =>
      import('./dashboard/dashboard.routes')
        .then((module) => module.DASHBOARD_ROUTES),
    data: { preload: true },
  },
  {
    path: 'reports',
    loadChildren: () =>
      import('./reports/reports.routes')
        .then((module) => module.REPORT_ROUTES),
    data: { preload: false },
  },
  {
    path: 'admin',
    loadChildren: () =>
      import('./admin/admin.routes')
        .then((module) => module.ADMIN_ROUTES),
  },
];
~~~

Register the strategy instead of PreloadAllModules:

~~~ts
provideRouter(
  routes,
  withPreloading(SelectivePreloadingStrategy),
)
~~~

A missing preload flag is treated as false in this implementation. This makes the opt-in policy clear: a route is loaded in the background only when it explicitly has preload set to true.

Preloading is a product and performance choice. Consider connection quality, data usage, initial application work, and how likely users are to visit each feature. For a very large feature, selective preloading may be preferable to downloading everything immediately.

## 4. Configure scrolling during router navigation

Angular can manage the browser viewport when the route changes. Configure it with withInMemoryScrolling:

~~~ts
import {
  provideRouter,
  withInMemoryScrolling,
} from '@angular/router';

provideRouter(
  routes,
  withInMemoryScrolling({
    scrollPositionRestoration: 'enabled',
    anchorScrolling: 'enabled',
  }),
)
~~~

The scrollPositionRestoration option has three values:

- disabled is the default; Angular does not manage scroll restoration.
- top scrolls to the top after every navigation.
- enabled restores a saved position on backward navigation. On other navigations, it scrolls to the URL fragment when one is present, otherwise it scrolls to the top.

Use enabled for the common pattern where forward navigation begins at the top or at a fragment, while browser back restores the previous position.

The top option is simpler, but it also resets scrolling when navigation changes only query parameters. Choose it only when that behavior is desired.

## 5. Enable anchor scrolling

A fragment identifies an element in the page, such as /guide#installation. Anchor scrolling is disabled by default. When enabled, the router scrolls to the matching element on navigation to a URL with a fragment.

~~~ts
withInMemoryScrolling({
  anchorScrolling: 'enabled',
})
~~~

A RouterLink can set the fragment:

~~~html
<a routerLink="/guide" fragment="installation">
  Installation
</a>

<section id="installation">
  <h2>Installation</h2>
</section>
~~~

Anchor scrolling does not run on browser popstate navigation. Back and forward navigation use the saved scroll position when restoration is enabled; if there is no saved position, the router's configured restoration behavior applies.

If a fixed header covers the target, configure a scroll offset with ViewportScroller:

~~~ts
import { inject, Injectable } from '@angular/core';
import { ViewportScroller } from '@angular/common';

@Injectable({ providedIn: 'root' })
export class ScrollOffsetSetup {
  private readonly viewportScroller = inject(ViewportScroller);

  constructor() {
    this.viewportScroller.setOffset([0, 72]);
  }
}
~~~

Ensure this setup service is actually instantiated, or set the offset from an application initialization point. The offset is the top and left pixel distance used when scrolling to an anchor.

## 6. Test routed behavior with RouterTestingHarness

RouterTestingHarness is an Angular test utility for navigating a real configured Router and inspecting the activated component. A typical test:

1. Configure TestBed with the standalone components and provideRouter.
2. Create the harness with RouterTestingHarness.create().
3. Navigate with navigateByUrl().
4. Assert on the activated component, rendered DOM, URL, or navigation result.

~~~ts
import { TestBed } from '@angular/core/testing';
import { provideRouter } from '@angular/router';
import { RouterTestingHarness } from '@angular/router/testing';
import { AboutPage, HomePage } from './pages';

describe('application routes', () => {
  it('renders the page selected by the URL', async () => {
    TestBed.configureTestingModule({
      imports: [HomePage, AboutPage],
      providers: [
        provideRouter([
          { path: 'home', component: HomePage },
          { path: 'about', component: AboutPage },
        ]),
      ],
    });

    const harness = await RouterTestingHarness.create();
    await harness.navigateByUrl('/about', AboutPage);

    expect(harness.routeNativeElement?.textContent)
      .toContain('About');
  });
});
~~~

The optional component type passed to navigateByUrl makes the expected route component explicit. The harness also exposes routeNativeElement for assertions against the rendered routed view.

Use your project's configured test runner for describe, it, and expect. Angular CLI projects may be configured with different test runners, so the runner imports and setup can vary even though the router harness usage is similar.

## 7. Test route parameters and the resulting URL

A routed component may read values from the URL. Test it by navigating to a representative URL rather than mocking ActivatedRoute by hand.

~~~ts
await harness.navigateByUrl('/users/42', UserPage);

expect(harness.routeNativeElement?.textContent)
  .toContain('User 42');

expect(TestBed.inject(Router).url).toBe('/users/42');
~~~

This exercises the route configuration, parameter parsing, component activation, and rendering together. Include invalid or unusual parameter values when they matter to the feature.

## 8. Test redirects and guards by observing navigation

A redirect is best tested by navigating to its source URL and asserting the destination URL and component:

~~~ts
await harness.navigateByUrl('/legacy-profile/42', ProfilePage);

expect(TestBed.inject(Router).url).toBe('/profile/42');
expect(harness.routeNativeElement?.textContent)
  .toContain('Profile');
~~~

A guard should be tested by checking both outcomes: navigation allowed, and navigation redirected or rejected. Provide a fake dependency when the guard depends on an authentication or permission service. Keep the real Router configuration in the test so that the test covers the actual navigation result.

## 9. Test nested routes as an integration

A nested route test needs the parent component to render a RouterOutlet. Import the standalone parent and child components, configure the parent with children, and navigate to the nested URL.

~~~ts
provideRouter([
  {
    path: 'settings',
    component: SettingsShell,
    children: [
      { path: 'profile', component: ProfilePage },
    ],
  },
])
~~~

Then navigate to /settings/profile and assert that the shell and profile content both appear in the harness-rendered view. This checks the integration among the route tree, parent outlet, and child component.

## 10. When the harness is not enough

RouterTestingHarness is designed for common routed-component tests using the primary outlet. A test involving named outlets, a specific application layout, or multiple complex outlets may need a custom host component with the necessary RouterOutlet elements.

Do not replace the Router with a mock for navigation tests. A real configured router catches errors in route definitions, outlet placement, redirects, and URL construction that a mocked navigation method would miss.

## 11. Keep router tests focused

A useful router test checks the observable behavior that matters to a user:

- Which component appears for a URL.
- Whether route parameters are read correctly.
- Whether a redirect reaches the expected destination.
- Whether a guard allows or redirects navigation.
- Whether a nested child renders inside its parent.
- Whether the resulting URL contains the expected route state.

Avoid testing private router implementation details or asserting every router event when a component, URL, or navigation result expresses the behavior more directly.

## 12. Common mistakes

- **Expecting every lazy route to preload by default:** the default strategy is NoPreloading.
- **Preloading every feature in a very large app without considering cost:** background loading uses network and memory.
- **Thinking preloading fetches API records:** it loads route code; data fetching is separate.
- **Enabling anchor scrolling but not providing a fragment target:** the URL fragment must correspond to an element ID.
- **Expecting anchor scrolling during browser back/forward:** popstate uses stored scroll restoration behavior.
- **Using top restoration when query-only navigation should preserve position:** top resets on all navigation.
- **Mocking the Router for route integration tests:** this skips matching, activation, outlet, and URL behavior.
- **Forgetting to import standalone components into TestBed:** routed standalone components must be available to the testing module.
- **Using the harness for complex named-outlet layouts without a host:** create a custom host with the required outlets.
- **Assuming all Angular projects use the same test runner:** use the runner already configured by the project.

## Key takeaways

- NoPreloading is the default; PreloadAllModules preloads lazy route modules after initial navigation.
- A custom PreloadingStrategy can use route data to selectively load features.
- In-memory scrolling can restore positions, scroll to the top, or follow a URL fragment.
- Anchor scrolling is disabled by default and does not run for popstate navigation.
- RouterTestingHarness navigates the real router and helps assert routed component behavior.
- Keep route tests focused on URLs, rendered content, parameters, redirects, guards, and outlets.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="083-page-titles-redirects-wildcard-routes-and-not-found-pages.md">083 - 83. Page titles, redirects, wildcard routes, and not-found pages</a></td><td><a href="../qnas/084-preloading-scroll-restoration-and-router-testing.md">Q&amp;A</a></td><td><a href="../examples/084-preloading-scroll-restoration-and-router-testing.md">Example</a></td><td><a href="085-configure-httpclient-with-providehttpclient.md">085 - Configure HttpClient with provideHttpClient</a></td>
</tr>
  </tbody>
</table>
