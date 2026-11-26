# Preloading, Scroll Restoration, and Router Testing — Examples

This chapter collects small examples for warming lazy routes, configuring document scrolling, and testing routed components. The snippets use standalone Angular APIs. Adapt imports and test-runner syntax to the Angular version and setup in your project.

## 1. Define routes that can be preloaded selectively

Mark only the lazy routes that should be warmed in the background:

~~~ts
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./home/home.page').then(module => module.HomePage)
  },
  {
    path: 'guide',
    loadChildren: () =>
      import('./guide/guide.routes').then(module => module.GUIDE_ROUTES),
    data: { preload: true }
  },
  {
    path: 'reports',
    loadChildren: () =>
      import('./reports/reports.routes').then(module => module.REPORT_ROUTES),
    data: { preload: false }
  },
  {
    path: '**',
    redirectTo: ''
  }
];
~~~

A route declared with loadChildren is lazy. The data flag is an application decision used by the custom strategy below; Angular does not assign special meaning to a property named preload.

Example lazy route files:

~~~ts
// guide/guide.routes.ts
import { Routes } from '@angular/router';

export const GUIDE_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./guide-home.page').then(module => module.GuideHomePage)
  },
  {
    path: 'installation',
    loadComponent: () =>
      import('./installation.page').then(module => module.InstallationPage)
  }
];
~~~

~~~ts
// reports/reports.routes.ts
import { Routes } from '@angular/router';

export const REPORT_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./reports-home.page').then(module => module.ReportsHomePage)
  }
];
~~~

## 2. Add a selective preloading strategy

The router gives a PreloadingStrategy each eligible route and a function that starts its lazy load. Call that function only for routes chosen by the application:

~~~ts
// selective-preloading.strategy.ts
import { Injectable } from '@angular/core';
import {
  PreloadingStrategy,
  Route
} from '@angular/router';
import { Observable, of } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    if (route.data?.['preload'] === true) {
      return load();
    }

    return of(null);
  }
}
~~~

Wire the strategy into the router:

~~~ts
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter, withPreloading } from '@angular/router';
import { routes } from './app.routes';
import { SelectivePreloadingStrategy } from './selective-preloading.strategy';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withPreloading(SelectivePreloadingStrategy)
    )
  ]
};
~~~

If every lazy route should be warmed, use PreloadAllModules instead:

~~~ts
provideRouter(
  routes,
  withPreloading(PreloadAllModules)
)
~~~

Preloading downloads route code. It does not make guarded pages public and does not replace authorization checks in route guards, APIs, or the server.

## 3. Configure restoration and anchor scrolling

Enable history restoration and URL fragment scrolling together:

~~~ts
import {
  provideRouter,
  withInMemoryScrolling
} from '@angular/router';

provideRouter(
  routes,
  withInMemoryScrolling({
    scrollPositionRestoration: 'enabled',
    anchorScrolling: 'enabled'
  })
)
~~~

With this configuration, a new navigation uses a URL fragment when available and otherwise scrolls to the top. Back and Forward navigation restores the saved position. Anchor scrolling does not take over on popstate navigation.

Other common scrollPositionRestoration values:

~~~ts
withInMemoryScrolling({
  scrollPositionRestoration: 'top'
})
~~~

~~~ts
withInMemoryScrolling({
  scrollPositionRestoration: 'disabled'
})
~~~

Use top when every navigation should begin at the top. Use disabled when the router should leave the viewport where it is.

## 4. Link to a fragment and provide a target

Use RouterLink with a fragment:

~~~html
<a [routerLink]="['/guide']" fragment="installation">
  Read installation instructions
</a>
~~~

The rendered route must contain an element with the matching id:

~~~html
<main>
  <h1>Guide</h1>
  <section>
    <h2 id="installation">Installation</h2>
    <p>Install the application dependencies before starting the development server.</p>
  </section>
</main>
~~~

A normal URL can also point directly to the fragment:

~~~text
/guide#installation
~~~

## 5. Offset anchors below a fixed header

When a fixed header covers the target after scrolling, configure the viewport offset at the application root:

~~~ts
import { Component } from '@angular/core';
import { ViewportScroller } from '@angular/common';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  template: '<app-header /><router-outlet />'
})
export class AppComponent {
  constructor(viewportScroller: ViewportScroller) {
    viewportScroller.setOffset([0, 72]);
  }
}
~~~

The first number is the horizontal offset and the second is the vertical offset, in pixels. Match it to the actual header height. If the target is inserted asynchronously, make sure it exists before relying on fragment scrolling.

## 6. Test a basic route with RouterTestingHarness

The harness creates a router test environment and exposes the component rendered in the primary outlet:

~~~ts
import { TestBed } from '@angular/core/testing';
import { provideRouter } from '@angular/router';
import { RouterTestingHarness } from '@angular/router/testing';
import { AboutPage } from './about.page';
import { HomePage } from './home.page';

describe('application routes', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HomePage, AboutPage],
      providers: [
        provideRouter([
          { path: '', component: HomePage },
          { path: 'about', component: AboutPage }
        ])
      ]
    });
  });

  it('renders the requested page', async () => {
    const harness = await RouterTestingHarness.create();

    await harness.navigateByUrl('/about', AboutPage);

    expect(harness.routeNativeElement?.textContent).toContain('About');
  });
});
~~~

Use the test runner configured in the project. The example uses familiar Jasmine-style describe, it, and expect syntax; the route setup and harness APIs are the important parts.

## 7. Test route parameters through visible behavior

Given a component that reads an id from ActivatedRoute, navigate to a concrete URL and assert what the page displays:

~~~ts
const routes = [
  { path: 'users/:id', component: UserPage }
];

TestBed.configureTestingModule({
  imports: [UserPage],
  providers: [provideRouter(routes)]
});

const harness = await RouterTestingHarness.create();
const page = await harness.navigateByUrl('/users/42', UserPage);

expect(page.userId()).toBe('42');
expect(harness.routeNativeElement?.textContent).toContain('User 42');
~~~

The exact accessor depends on the page implementation. Prefer an assertion on meaningful page behavior so the test describes the contract rather than private router details.

## 8. Test a guard's allowed and denied outcomes

Configure the guard dependencies with the test's desired state, navigate, and assert the resulting page or redirect. Test both outcomes:

~~~ts
const routes = [
  { path: 'login', component: LoginPage },
  {
    path: 'admin',
    canActivate: [adminGuard],
    component: AdminPage
  }
];

TestBed.configureTestingModule({
  imports: [LoginPage, AdminPage],
  providers: [
    provideRouter(routes),
    { provide: AuthService, useValue: { isAdmin: () => false } }
  ]
});

const harness = await RouterTestingHarness.create();
await harness.navigateByUrl('/admin', LoginPage);

expect(harness.routeNativeElement?.textContent).toContain('Sign in');
~~~

Add another test with an authorized AuthService and assert that AdminPage is activated. The guard remains a navigation rule; preloading route code must not be used as an access-control mechanism.

## 9. Test a redirect

The harness can assert the component activated after a redirect:

~~~ts
TestBed.configureTestingModule({
  imports: [HomePage],
  providers: [
    provideRouter([
      { path: '', component: HomePage },
      { path: 'start', redirectTo: '' }
    ])
  ]
});

const harness = await RouterTestingHarness.create();
await harness.navigateByUrl('/start', HomePage);

expect(harness.routeNativeElement?.textContent).toContain('Home');
~~~

For more complex redirect rules, also assert the final router URL if it is part of the behavior your application promises.

## 10. Verify the selective strategy's decision logic

Keep the decision simple enough to reason about. When unit-testing the strategy directly, use a route with the preload data flag and a loader spy:

~~~ts
const strategy = new SelectivePreloadingStrategy();
const route: Route = {
  path: 'guide',
  data: { preload: true }
};
const load = jasmine.createSpy('load').and.returnValue(of('loaded'));

strategy.preload(route, load).subscribe();

expect(load).toHaveBeenCalledTimes(1);
~~~

And verify an unmarked route is left alone:

~~~ts
const strategy = new SelectivePreloadingStrategy();
const route: Route = { path: 'reports' };
const load = jasmine.createSpy('load').and.returnValue(of('loaded'));

strategy.preload(route, load).subscribe();

expect(load).not.toHaveBeenCalled();
~~~

These snippets assume Jasmine spies. If the project uses Vitest or Jest, replace the spy and assertion helpers with that runner's equivalents. Ensure the file imports Route and of in a real test.

## 11. Browser verification checklist for scroll behavior

Unit tests can verify configuration and application decisions. Use a real browser session to check viewport behavior:

1. Open a long page and scroll well below the top.
2. Navigate to a different route, then use Back and confirm the earlier position returns.
3. Use Forward and confirm the next route's saved position returns.
4. Open a URL containing a fragment and confirm its element is visible below any fixed header.
5. Navigate between routes that have a nested overflow container and verify that container's behavior separately.
6. Repeat at a narrow viewport if the header height or layout changes.

The browser check matters because document layout, rendering timing, and native history are not fully represented by a component-level test.

## 12. Common failure checks

- If no route is preloaded, confirm withPreloading is registered and the route uses a lazy loader.
- If an unmarked route is being loaded, inspect the custom strategy and the route's data.
- If the browser downloads every route chunk, confirm the strategy selected and inspect the production build's network requests.
- If a fragment does not move the viewport, confirm anchorScrolling is enabled and the target id matches the URL fragment.
- If Back does not restore scroll, confirm scrollPositionRestoration is enabled and that application code does not overwrite the position.
- If a harness test cannot find a component, confirm the URL, route configuration, imports, guard results, and expected component type.
- If the test runner rejects a matcher or spy, adapt only the test syntax to the runner configured by the project.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="083-page-titles-redirects-wildcard-routes-and-not-found-pages.md">083 - 83. Page titles, redirects, wildcard routes, and not-found pages</a></td><td><a href="../lessons/084-preloading-scroll-restoration-and-router-testing.md">Lesson</a></td><td><a href="../qnas/084-preloading-scroll-restoration-and-router-testing.md">Q&amp;A</a></td><td><a href="085-configure-httpclient-with-providehttpclient.md">085 - Configure HttpClient with provideHttpClient</a></td>
</tr>
  </tbody>
</table>
