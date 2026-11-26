# 82. Route-level providers and route configuration — Q&A

Questions and answers for [Lesson 82](../lessons/082-route-level-providers-and-route-configuration.md).

## Fundamentals

### 1. What are route-level providers?

They are dependencies registered in a route's providers array. Angular makes them available to that route and its descendants through a route EnvironmentInjector.

### 2. What injector does Angular create for a route with providers?

Angular creates an EnvironmentInjector for the route and uses it for the route and its children. The route's providers belong to that environment-injector scope.

### 3. Does a route need a component to provide dependencies?

No. A componentless route can provide dependencies to its child routes. It can group feature routes and shared configuration without adding a parent component view.

### 4. Who can inject a provider registered on a parent route?

The parent route's content and its descendants can resolve it, subject to Angular's injector hierarchy and any nearer provider for the same token.

### 5. Can a parent component inject a provider from one of its child routes?

No. Dependency resolution looks from the requester toward its ancestor injectors. It does not search down into child route scopes.

### 6. Can sibling routes use each other's route providers?

No. Siblings do not inherit providers from one another. Put a provider on their nearest shared ancestor if they should share it.

### 7. Do route-level providers apply to all routes in the application?

No. They apply within the route branch that owns the provider and its descendants. Unrelated route branches do not inherit them.

## Comparing provider scopes

### 8. What is the difference between a root provider and a route provider?

A root provider belongs to the application's root environment injector and is broadly available. A route provider belongs to the injector for one route branch and is available there and below.

### 9. What is the difference between a route provider and a component provider?

A route provider belongs to an EnvironmentInjector associated with the route. A component provider belongs to the component's ElementInjector and is available to that component and applicable descendants.

### 10. Which provider wins when a component and route both provide the same token?

Angular checks the element-injector hierarchy for the requesting component before its environment-injector hierarchy. A nearer component provider can therefore override the route provider for that component subtree.

### 11. Is a route provider one instance per navigation?

Do not assume that leaving and re-entering a route creates a fresh service instance. Provider scope is tied to an injector. If a feature needs an explicit reset or per-navigation state, design that lifecycle directly.

### 12. Can the same service class be provided on two sibling routes?

Yes. Each route can register the class and its own configuration. Consumers under each branch resolve their branch's provider.

~~~ts
{
  path: 'catalog',
  providers: [
    CatalogApi,
    { provide: CATALOG_API_ROOT, useValue: '/api/catalog' },
  ],
}
~~~

A sibling route can provide the same CatalogApi class with a different CATALOG_API_ROOT value.

## Tokens and route-specific configuration

### 13. Why use an InjectionToken for route configuration?

It gives a typed key for a value that participates in dependency injection, such as an API base path, feature configuration, or adapter implementation. It also avoids coupling a service to one hard-coded value.

### 14. If a token is route-provided, can a root-provided service automatically read that route value?

Not reliably. A root-provided service is created in the root injector's context. A token available only in a child route injector is not visible upward to that root service.

Register the consuming service at the route scope too, or pass the route-specific value through an explicit API that fits the design.

### 15. Can a route provider override a value also provided at the root?

Yes. For requests made within that route branch, the route injector is nearer than the root injector, so its provider is resolved first unless an even nearer element injector provides the same token.

### 16. What provider forms can be used on a route?

A route's providers can include Angular Provider entries and EnvironmentProviders. Common provider forms include a class, a value provider, a class alias, or a factory provider.

### 17. Can a route provider depend on another route provider?

Yes. Providers in the same route injector can depend on other providers available from that injector or its ancestors. Keep configuration dependencies explicit and ensure the provider is registered where its consumers can resolve it.

### 18. Can a route provider's value depend on a URL parameter?

A static useValue does not change automatically when the URL parameter changes. For data derived from route state, use ActivatedRoute in the route component or an appropriate resolver/data-loading pattern. Do not assume the route provider factory receives each activated parameter as a fresh configuration.

## Guards, resolvers, and route data

### 19. Can a functional guard inject a route-scoped service?

Yes, when the guard runs within that route provider subtree. A functional guard may call inject synchronously in the injection context Angular establishes for it.

### 20. Why can inject not be called inside an arbitrary callback?

inject depends on Angular having established an injection context. That context is synchronous; it is not automatically preserved after an asynchronous boundary or in an unrelated callback. Inject dependencies at the beginning of the guard or function.

### 21. What is the difference between route providers and route data?

Providers register injectable dependencies. Route data stores metadata associated with a route, such as a title, breadcrumb label, or analytics key. Components typically read route data through ActivatedRoute.

### 22. Should a value be a provider or route data?

Use a provider if it should be injected or configure a service. Use route data if it describes the route and should be read as metadata. Static labels and identifiers are often route data; an API client is a provider.

### 23. Can route providers be used with lazy-loaded children?

Yes. The route injector applies to the route and its children. For a lazy-loaded NgModule, Angular uses that injector as the parent of the loaded module's injector.

### 24. Are lazy loading and route-level providers the same feature?

No. Lazy loading controls when route code is loaded. Route-level providers control where dependencies are available. They can be used together, but one does not imply the other.

## Design and troubleshooting

### 25. Are route providers a security boundary?

No. They organize client-side dependencies; they do not prevent a user from entering a URL or calling a backend endpoint. Use guards for client navigation and enforce authorization on the server.

### 26. Should secrets be stored in route providers?

No. Browser-side code and configuration are available to the user. Do not put secret credentials or private keys in client-side providers.

### 27. Why does a route child report a missing provider?

Check whether the dependency is registered on the route itself or an ancestor route, whether the component is actually activated within that route branch, and whether a provider token is imported from the same definition.

### 28. Why does a route-scoped service use the wrong configuration?

Common causes include:

- The service is only provided in root and is constructed there.
- The route provider uses a different token instance than the service imports.
- A nearer component provider overrides the expected service.
- The consumer is activated outside the route branch with the configuration.

### 29. Why do two consumers receive different instances of a service?

The same class may be provided at several injector levels. Angular resolves the nearest provider for each consumer. A component-level provider can create an instance different from the route-scoped or root instance.

### 30. Where should a service be provided when several child pages need to share state?

Put it on their nearest shared route ancestor if the state belongs to that route feature. Use component providers if it should be limited to a component subtree, or root if it should be shared application-wide.

### 31. Does a route provider protect API calls made by its service?

No. It can configure the client-side service, but backend endpoints still need their own authentication and authorization checks.

### 32. What should I inspect when an injection error occurs in a route?

1. Identify the requesting component, guard, or resolver.
2. Trace the activated route back to its ancestors.
3. Confirm the provider is on the route or ancestor shared by that requester.
4. Check for a nearer component provider that changes the selected instance.
5. Confirm the token and service are imported from the intended module.
6. Check whether the consuming service is actually instantiated in the route scope.

## Quick review

- **Application provider:** broadly available from the application environment injector.
- **Route provider:** available to one route and its descendants.
- **Component provider:** owned by a component's element-injector subtree.
- **Nearest provider:** selected first according to Angular's injector hierarchy.
- **Route data:** metadata, not an injectable service registration.
- **Route scope:** code organization, not backend authorization.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="081-nested-routes-child-outlets-and-auxiliary-routes.md">081 - 81. Nested routes, child outlets, and auxiliary routes</a></td><td><a href="../lessons/082-route-level-providers-and-route-configuration.md">Lesson</a></td><td><a href="../examples/082-route-level-providers-and-route-configuration.md">Example</a></td><td><a href="083-page-titles-redirects-wildcard-routes-and-not-found-pages.md">083 - 83. Page titles, redirects, wildcard routes, and not-found pages</a></td>
</tr>
  </tbody>
</table>
