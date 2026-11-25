# Q&A: Configure the Angular Router in a standalone app

These questions review the first setup of Angular routing in a standalone application. See the [lesson](../lessons/073-configure-the-angular-router-in-a-standalone-app.md) and the [complete setup example](../examples/073-configure-the-angular-router-in-a-standalone-app.md).

## Router configuration

### 1. What does Angular Router do?

It matches the current application URL against configured routes and activates the component or behavior associated with the first matching route.

### 2. Which package contains the Angular Router?

`@angular/router`. It is separate from `@angular/core` and can be used by standalone applications or NgModule-based applications.

### 3. What is the basic standalone router setup?

Define a `Routes` array, register it with `provideRouter(routes)` in the application providers, and include a `RouterOutlet` in a template where routed components can render.

### 4. What is `Routes`?

It is the Angular type for a collection of route definitions. Import it from `@angular/router` and use it to type the route table.

### 5. What does a component route usually contain?

At minimum, a `path` and a `component`:

~~~ts
{path: 'about', component: AboutPage}
~~~

### 6. Should route paths start with a slash?

No. Define `path: 'about'`. The browser URL for that route is still `/about`.

### 7. Where are routes commonly stored in a CLI project?

Usually in `src/app/app.routes.ts`. The exact filenames can vary with project options or later edits.

### 8. What does `provideRouter(routes)` do?

It registers the router providers and route table for the application. It returns providers intended for an application-level provider configuration.

### 9. Where should `provideRouter(routes)` go in a standalone CLI app?

In the `providers` array in the existing `ApplicationConfig`, commonly in `src/app/app.config.ts`.

### 10. Should each route component call `provideRouter()`?

No. Configure the application router once at the application level.

### 11. Do I need to replace all providers in `app.config.ts` to add routing?

No. Add `provideRouter(routes)` alongside the existing providers so other application features keep their configuration.

### 12. How does the CLI app receive `appConfig`?

The generated bootstrap code passes the application configuration to `bootstrapApplication()`. Preserve that generated connection when adding routing.

### 13. What is the difference between `provideRouter()` and `RouterModule.forRoot()`?

`provideRouter()` is the provider-based setup used by standalone applications. `RouterModule.forRoot()` is associated with NgModule-based setup.

## Paths, order, and fallback behavior

### 14. What does `path: ''` represent?

It represents the application root path, typically the URL `/`.

### 15. How do you redirect the root path to `/home`?

Use a route like:

~~~ts
{path: '', pathMatch: 'full', redirectTo: 'home'}
~~~

### 16. Why is `pathMatch: 'full'` useful on an empty-path redirect?

The default path matching strategy is prefix matching. An empty path is a prefix of every URL, so an exact match prevents the redirect from unexpectedly applying to other routes.

### 17. What does `pathMatch: 'full'` mean?

It requires the whole remaining URL to match the route path before the route is selected.

### 18. What does `path: '**'` mean?

It is a wildcard route that can catch a URL not matched by earlier routes. It is commonly used for a not-found component.

### 19. Where should a wildcard route go?

At the end of the route array. Angular uses a first-match-wins strategy, so earlier routes should get a chance to match first.

### 20. Why does route order matter?

Angular stops at the first matching route. A broad or wildcard route placed too early can capture URLs that a later, more specific route should handle.

### 21. When should the fixed path `users/new` come before `users/:id`?

When both patterns exist. The static route should precede a parameter route so the literal segment `new` is not treated as an ID.

### 22. What is the `title` property for on a route?

It sets the document title when that route is active. Meaningful titles help users, including people using assistive technology, identify the current page.

### 23. Are guards, resolvers, and parameters required for the initial router setup?

No. Those are additional route features. Start with path-to-component mapping and add advanced behavior when the application needs it.

## Rendering and navigation

### 24. Why is `RouterOutlet` needed?

It marks the location in a template where the currently activated route component is rendered. Without an outlet, the route may match but there is no normal outlet location to show that routed view.

### 25. Does `RouterOutlet` disappear after a route component renders?

No. The outlet remains as the anchor for routed content; the activated component is rendered adjacent to it in the DOM.

### 26. Which component should import the primary `RouterOutlet`?

Usually the root standalone component. Add `RouterOutlet` to that component's `imports` and include `<router-outlet />` in its template.

### 27. Does this setup also teach declarative navigation links?

Only at a basic orientation level. The next chapter covers `RouterLink` and active route links in detail.

### 28. What is the role of `RouterLink`?

It is a directive that makes a link navigate through Angular Router. Use a real `<a>` element for navigation and bind the route through `routerLink`.

### 29. Why can a deep URL work after in-app navigation but fail on refresh?

In-app navigation is handled by the client router. A direct browser request also reaches the web server, which must be configured to serve the Angular app entry point for valid client routes.

### 30. Does `provideRouter()` configure that web-server fallback?

No. It configures routing inside the Angular application. Server fallback rules are a separate deployment concern.

## Troubleshooting

### 31. What happens if the route table is not passed to `provideRouter()`?

The standalone app does not have that route configuration registered, so expected navigation and route activation will not be set up.

### 32. Why might no routed view appear even though the route is configured?

Check that the active route's component is declared correctly and that the root template contains an imported `RouterOutlet`.

### 33. Why should you not put a leading slash in a route's `path`?

Route path definitions are relative path patterns. Angular builds the browser URL from them; the path config itself should be written without the leading slash.

### 34. What happens if the wildcard is before a normal route?

The wildcard may match first and prevent the normal route from being reached. Move the wildcard to the end.

### 35. What should you do when adding the router to an existing standalone app?

Import the routes into the existing app config, add `provideRouter(routes)` to its providers, and include `RouterOutlet` in the root component's imports and template. Keep unrelated providers intact.

## Review checklist

- [ ] Route objects use paths without leading slashes.
- [ ] Routes are imported where the application config is defined.
- [ ] `provideRouter(routes)` is registered once at application scope.
- [ ] A root outlet provides the rendering location.
- [ ] An empty-path redirect uses `pathMatch: 'full'`.
- [ ] The wildcard route is last.
- [ ] Production hosting is configured to serve the app for deep client routes.

## References

- [Angular routing overview](https://angular.dev/guide/routing)
- [Define routes](https://angular.dev/guide/routing/define-routes)
- [Show routes with outlets](https://angular.dev/guide/routing/show-routes-with-outlets)
- [provideRouter() API](https://angular.dev/api/router/provideRouter)
- [Router reference](https://angular.dev/guide/routing/router-reference)
