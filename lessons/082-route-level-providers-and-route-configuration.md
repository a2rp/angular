# 82. Route-level providers and route configuration

## What you will learn

After this lesson, you should be able to:

- Explain the providers property on a route.
- Scope services and configuration tokens to a route branch.
- Compare route, application, and component provider scopes.
- Understand how Angular resolves dependencies through injector hierarchies.
- Configure route-specific services for different feature areas.
- Use route-scoped services in child pages and functional guards.
- Avoid common provider-scope mistakes.

## 1. Routes can configure dependency injection

A route maps a URL pattern to a view and can also configure dependencies for that section of the application. Its providers array supplies dependencies to that route and its children.

~~~ts
import type { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: 'admin',
    providers: [
      AdminAuditService,
      { provide: ADMIN_API_ROOT, useValue: '/api/admin' },
    ],
    children: [
      { path: 'users', component: AdminUsersPage },
      { path: 'teams', component: AdminTeamsPage },
    ],
  },
  { path: 'account', component: AccountPage },
];
~~~

The admin route and its descendants can inject AdminAuditService and ADMIN_API_ROOT. The unrelated account route does not inherit those providers.

Angular creates an EnvironmentInjector for a route with providers and uses it for that route and its children. This gives a route branch its own dependency scope between the application environment and narrower component scopes.

A route can provide dependencies even when it has no component of its own. A componentless parent can group child routes and their shared providers.

## 2. Provider visibility follows the route tree

Think of route providers as a scope attached to a branch:

~~~text
Application EnvironmentInjector
├── /admin route EnvironmentInjector
│   ├── /admin/users
│   └── /admin/teams
└── /account
~~~

A route provider is available to that route and its descendants. A child route can use a provider from an ancestor route. A parent cannot inject a provider that exists only on one of its child routes, and sibling routes do not inherit one another's providers.

For example, a component activated under /admin/users can inject services provided by /admin. A component outside the admin branch cannot.

This is useful for feature-specific configuration, API clients, state, or services that should be shared within one route section without becoming application-wide.

## 3. Angular resolves the closest applicable provider

Angular's dependency injection is hierarchical. For a component request, Angular checks the component's element injector and its ancestors. If the token is not found there, Angular checks the applicable EnvironmentInjector hierarchy, which may include route and application injectors.

When a token is provided at several levels, the nearer provider wins. A component-level provider can override a route-level instance for that component and its component descendants.

~~~ts
@Component({
  standalone: true,
  selector: 'app-admin-users',
  providers: [AdminAuditService],
  template: '<h1>Users</h1>',
})
export class AdminUsersPage {}
~~~

Here, the page and its component descendants use the component-scoped AdminAuditService. Other components in the admin route subtree can still use the route-scoped service.

Choose the scope intentionally:

- Use application providers for dependencies shared broadly across the app.
- Use route providers for dependencies shared within one route branch.
- Use component providers for dependencies owned by one rendered component subtree.

## 4. Provide route-specific configuration with a token

An InjectionToken lets a service depend on configuration without hard-coding it.

~~~ts
import { inject, Injectable, InjectionToken } from '@angular/core';

export const CATALOG_API_ROOT =
  new InjectionToken<string>('CATALOG_API_ROOT');

@Injectable()
export class CatalogApi {
  private readonly apiRoot = inject(CATALOG_API_ROOT);

  getBooksUrl(): string {
    return this.apiRoot + '/books';
  }
}
~~~

Register both the service and its configuration on the route:

~~~ts
{
  path: 'catalog',
  component: CatalogShell,
  providers: [
    CatalogApi,
    { provide: CATALOG_API_ROOT, useValue: '/api/catalog' },
  ],
  children: [
    { path: 'books', component: CatalogBooksPage },
  ],
}
~~~

CatalogBooksPage and other descendants of this route can inject CatalogApi. The service receives the route's CATALOG_API_ROOT value.

The service is deliberately registered in the route's providers array. If a service that reads route-specific configuration is provided only in the root injector, its factory is created in the root scope and cannot read a token that exists only in a child route injector. Provide the service itself at the route scope when it needs that route's configuration.

## 5. Use the same service class with different route configuration

Sibling route branches can provide the same class with different values:

~~~ts
const catalogChildren: Routes = [
  { path: '', pathMatch: 'full', redirectTo: 'books' },
  { path: 'books', component: CatalogBooksPage },
];

export const routes: Routes = [
  {
    path: 'catalog',
    component: CatalogShell,
    providers: [
      CatalogApi,
      { provide: CATALOG_API_ROOT, useValue: '/api/catalog' },
    ],
    children: catalogChildren,
  },
  {
    path: 'catalog-preview',
    component: CatalogShell,
    providers: [
      CatalogApi,
      { provide: CATALOG_API_ROOT, useValue: '/api/catalog-preview' },
    ],
    children: catalogChildren,
  },
];
~~~

Under /catalog, CatalogApi receives /api/catalog. Under /catalog-preview, it receives /api/catalog-preview. Each route branch has its own route EnvironmentInjector, so the service and token are resolved within that branch.

This pattern can support feature variants, preview modes, or different API roots. Keep the configuration explicit so it is clear which route owns each value.

## 6. Route providers and root providers

A service marked with providedIn: 'root' is available from the application's root environment injector:

~~~ts
@Injectable({ providedIn: 'root' })
export class LoggerService {}
~~~

A root provider is appropriate when the same application-wide instance and behavior should be shared across routes. A route can also provide the same class for a more specific instance:

~~~ts
{
  path: 'admin',
  providers: [LoggerService],
  children: [/* admin routes */],
}
~~~

Requests in the admin branch resolve its route provider before reaching the root provider. Other branches continue to use the root provider.

Configuration tokens need the same care. If a root-provided service must read a value that changes by route, adding that token only to a route does not move the root service into that route's injector. Register the service at the route scope too, or pass the route-specific value through an explicit method or another suitable abstraction.

## 7. Use route providers from a functional guard

A functional guard can use inject while Angular invokes it in an injection context:

~~~ts
import { inject } from '@angular/core';
import type { CanActivateFn } from '@angular/router';

export const canOpenAdmin: CanActivateFn = () => {
  const access = inject(AdminAccessService);
  return access.canOpenAdmin();
};
~~~

If the guard runs on a route within the admin provider subtree, it can use the route-scoped AdminAccessService.

Calls to inject must happen synchronously while Angular has established an injection context. Do not call inject later from an arbitrary callback or after an asynchronous boundary. Inject the dependency first, then use the injected value.

## 8. Route providers are different from route data

Route providers register injectable dependencies. Route data attaches metadata to a route and can be read through ActivatedRoute.

~~~ts
{
  path: 'reports',
  component: ReportsPage,
  data: { title: 'Reports', section: 'analytics' },
}
~~~

Use route data for descriptive values such as a title, breadcrumb label, or feature identifier. Use a provider when a class, token, or configured service must participate in dependency injection. They solve different problems.

## 9. Route providers and lazy routes

Route providers can be used with routes that load child route definitions. Angular uses the route's EnvironmentInjector for that route and its children; for a lazy-loaded module, it is also the parent of the loaded module's injector.

~~~ts
{
  path: 'workspace',
  providers: [WorkspaceSession],
  loadChildren: () =>
    import('./workspace/workspace.routes').then((module) => module.WORKSPACE_ROUTES),
}
~~~

This makes the route a natural place for dependencies shared by a lazy feature. Lazy loading and provider scope are separate concerns: lazy loading controls when route code is loaded, while providers control where dependencies are available.

## 10. Choose a provider location

| Need | Typical location |
| --- | --- |
| One shared service for the whole app | providedIn: 'root' or application configuration |
| A service/configuration shared by a route section and its descendants | Route providers |
| A service instance owned by one component and its descendants | Component providers |
| Static route metadata such as a title or breadcrumb key | Route data |
| Data that must be ready before route activation | A resolver or another explicit data-loading pattern |

Avoid widening a provider's scope just to resolve a missing-provider error. Identify which route or component needs the dependency, then provide it at the narrowest shared ancestor that can supply all intended consumers.

## 11. Route providers are not access control

A route-scoped service can organize feature dependencies, but provider scope does not protect data or secure an API. A user can navigate directly to a URL or call a backend endpoint outside the Angular UI.

Use router guards for client-side navigation decisions and enforce authorization on the server for protected data and operations. Do not put secrets in browser-side provider values; client code and configuration are available to the user.

## 12. Common mistakes

- **Providing only the token, not the route-configured service:** a root-created service cannot see a value that exists only in a child route injector.
- **Putting a provider on a child instead of a shared parent:** sibling components then do not share that provider scope.
- **Expecting a parent to inject a child's provider:** dependency resolution walks toward ancestors, not down into child route scopes.
- **Confusing route and component providers:** route providers belong to the EnvironmentInjector hierarchy; component providers belong to the ElementInjector hierarchy.
- **Providing the same class at several levels without realizing it:** the nearest provider is selected, so different consumers may get different instances.
- **Using route data as if it registered a service:** metadata is not an injectable provider.
- **Treating route scope as a security boundary:** client-side provider scope does not authorize a server request.
- **Assuming a new service is constructed for every navigation:** provider scope belongs to an injector; do not rely on route re-entry to reset state.

## Key takeaways

- A route's providers apply to that route and its descendants through a route EnvironmentInjector.
- Sibling route branches do not inherit each other's route-level dependencies.
- The closest applicable provider wins; a component provider can override a route provider.
- Register a service at route scope when it must inject route-specific configuration.
- Route data is metadata; route providers supply dependencies.
- Route-level injection improves feature organization, but it does not replace server-side authorization.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="081-nested-routes-child-outlets-and-auxiliary-routes.md">081 - 81. Nested routes, child outlets, and auxiliary routes</a></td><td><a href="../qnas/082-route-level-providers-and-route-configuration.md">Q&amp;A</a></td><td><a href="../examples/082-route-level-providers-and-route-configuration.md">Example</a></td><td><a href="083-page-titles-redirects-wildcard-routes-and-not-found-pages.md">083 - 83. Page titles, redirects, wildcard routes, and not-found pages</a></td>
</tr>
  </tbody>
</table>
