# Lazy-load routes and standalone components

Lazy loading defers route code until a visitor navigates to that part of the app. This can reduce the JavaScript needed for the initial page, especially when an application has large features that most visitors do not open immediately.

Angular provides two main route properties for this:

- `loadComponent` lazily imports the component for one route.
- `loadChildren` lazily imports a child route configuration.

Use eager loading for the primary page when it should be ready immediately. Lazy-load substantial or less frequently visited areas. Lazy loading changes when code is downloaded; it does not protect the code or its data from a visitor.

## Learning outcomes

After this lesson, you should be able to:

- Explain the difference between eager `component`, lazy `loadComponent`, and lazy `loadChildren`.
- Split a standalone screen into an on-demand JavaScript chunk.
- Put a feature's route tree in its own file and lazy-load it.
- Choose route boundaries that balance initial bundle size and later navigation speed.
- Explain how preloading trades background network use for faster first navigation.
- Distinguish route lazy loading from template-level `@defer`.

## Eager and lazy route components

A route using `component` with a normal top-level import is eager: its component is included with the code containing the route configuration.

~~~ts
import {ReportsPage} from './reports/reports.page';

const routes = [
  {
    path: 'reports',
    component: ReportsPage,
  },
];
~~~

A `loadComponent` route uses a dynamic import. The router requests the component code when that route is about to become active:

~~~ts
const routes = [
  {
    path: 'reports',
    loadComponent: () =>
      import('./reports/reports.page').then(module => module.ReportsPage),
  },
];
~~~

The loaded component must be standalone. Standalone components are the default for components created with current Angular versions; keeping `standalone: true` explicit in a teaching example also makes the requirement clear.

The dynamic import returns a promise. The router waits for that import before activating the component. Build tools such as Angular's bundler can turn the import boundary into a separate JavaScript chunk requested on demand.

## Lazy-load a feature route tree

Use `loadChildren` when a feature has its own group of child routes. With a standalone app, the imported file can export a `Routes` array:

~~~ts
// app.routes.ts
export const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () =>
      import('./admin/admin.routes').then(module => module.ADMIN_ROUTES),
  },
];
~~~

~~~ts
// admin/admin.routes.ts
import type {Routes} from '@angular/router';

export const ADMIN_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./admin-shell.page').then(module => module.AdminShellPage),
    children: [
      {
        path: '',
        pathMatch: 'full',
        redirectTo: 'dashboard',
      },
      {
        path: 'dashboard',
        loadComponent: () =>
          import('./dashboard.page').then(module => module.AdminDashboardPage),
      },
      {
        path: 'users',
        loadComponent: () =>
          import('./users.page').then(module => module.AdminUsersPage),
      },
    ],
  },
];
~~~

There are two boundaries here. The router first loads the admin route configuration when matching an `/admin` URL. The admin shell and its child screens are themselves loaded through `loadComponent` as their routes activate. Each page component that uses this approach must be standalone.

Legacy applications can also use `loadChildren` to load an NgModule. The standalone route-array pattern is appropriate for new standalone applications; an NgModule is not required just to create a lazy feature boundary.

## Choose a useful boundary

Lazy loading helps when it removes meaningful code from the initial request. It can hurt when every tiny screen becomes a separate request or when nested lazy boundaries create a long sequence of downloads before a page appears.

A useful starting point is:

- Keep the primary landing route eager if it is needed immediately.
- Lazy-load substantial features such as administration, analytics, or an editor.
- Use `loadComponent` for an isolated standalone destination.
- Use `loadChildren` when a feature has several related routes.
- Avoid deeply nested lazy boundaries unless they match real feature boundaries and measurements support them.

The first navigation to a lazy route may take longer because its chunk has not been downloaded yet. Later navigations usually reuse the loaded code for the lifetime of the application.

## Preloading trade-offs

The default router strategy is no preloading: lazy route configurations are loaded when navigation needs them. Angular also provides `PreloadAllModules` for preloading lazy-loaded route modules after initial navigation. Preloading can make a later visit feel faster, but it spends bandwidth and memory before the visitor asks for that feature. It is a trade-off, not a free speed-up.

~~~ts
import {PreloadAllModules, provideRouter, withPreloading} from '@angular/router';

export const appConfig = {
  providers: [
    provideRouter(routes, withPreloading(PreloadAllModules)),
  ],
};
~~~

Measure the production build and actual navigation experience before enabling broad preloading. The built-in strategy is described in terms of lazy-loaded route modules; do not assume every standalone `loadComponent` boundary is preloaded the same way. For selective feature-route preloading, a custom strategy can use route metadata.

## Route lazy loading versus `@defer`

Route lazy loading divides code at navigation boundaries. The user must navigate to the route before its component code is needed.

`@defer` divides eligible template dependencies within a component. It can load a section later based on a trigger such as viewport visibility or user interaction, without changing the route. Use route lazy loading for whole screens or features and `@defer` for optional parts of an already active screen.

## Loading feedback and failures

A lazy route can have a visible delay the first time it loads. The router can expose navigation state so the app shell can display a progress indicator. Keep that feedback accessible and avoid leaving the visitor unsure whether the link worked.

A dynamic import can fail, for example when a network connection drops or a deployment removes a chunk still referenced by an already-open app. Handle navigation errors at the application level and provide a useful recovery path. During deployment, configure asset caching and version rollout so old app shells do not point to missing chunks.

Lazy-loaded code is still client code. Do not place secrets in a lazy feature and do not use lazy loading as authorization. Protect route navigation with guards where appropriate, and enforce data access in the server/API.

## Verify the split

A source-level dynamic import indicates a lazy boundary, but the production build shows what the bundler actually emitted. Build the app and inspect the output:

- Is the initial bundle smaller than when all route components are imported directly?
- Did a feature become a separate chunk?
- Does opening the feature request that chunk?
- Did excessive splitting or nested lazy loading add too much navigation delay?
- Would measured preloading improve the experience for common routes?

Avoid optimizing only for a smaller initial bundle if it makes the routes people need feel slow.

## Related material

- [Resolvers, route data, and loading a page before activation](079-resolvers-route-data-and-loading-a-page-before-activation.md)
- [Nested routes, child outlets, and auxiliary routes](081-nested-routes-child-outlets-and-auxiliary-routes.md)
- [Route-level providers and route configuration](082-route-level-providers-and-route-configuration.md)
- Angular lazy-loaded routes guide
- Angular route loading strategies
- Angular route preloading guidance
- Angular standalone component imports

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="079-resolvers-route-data-and-loading-a-page-before-activation.md">079 - Resolvers, route data, and loading a page before activation</a></td><td><a href="../qnas/080-lazy-load-routes-and-standalone-components.md">Q&amp;A</a></td><td><a href="../examples/080-lazy-load-routes-and-standalone-components.md">Example</a></td><td><a href="081-nested-routes-child-outlets-and-auxiliary-routes.md">081 - 81. Nested routes, child outlets, and auxiliary routes</a></td>
</tr>
  </tbody>
</table>
