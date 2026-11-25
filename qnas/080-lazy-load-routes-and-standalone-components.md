# Lazy-load routes and standalone components: Q&A

## 1. What does lazy loading mean for an Angular route?

It means the router downloads route code when it is needed for navigation instead of including that code in the initial eager route bundle. The bundler usually emits separate JavaScript chunks for dynamic imports.

## 2. What is the difference between `component` and `loadComponent`?

`component` references a class through a normal import, so the component is eagerly available with the route configuration. `loadComponent` is an async loader, commonly using `import()`, so the component code is requested when that route is about to activate.

## 3. What is the difference between `loadComponent` and `loadChildren`?

- `loadComponent` lazily loads one routed component.
- `loadChildren` lazily loads a child route configuration, which can contain several related routes.

Use `loadComponent` for an isolated screen and `loadChildren` for a feature with its own route tree.

## 4. Does a component loaded by `loadComponent` need to be standalone?

Yes. The router's `loadComponent` property lazy-loads a standalone component. Current Angular components are standalone by default, but an explicit `standalone: true` helps show the requirement and supports older Angular versions that defaulted to non-standalone.

## 5. Can a standalone feature be lazy-loaded without an NgModule?

Yes. Export a `Routes` array from a feature route file and return it from `loadChildren`:

~~~ts
{
  path: 'admin',
  loadChildren: () =>
    import('./admin/admin.routes').then(module => module.ADMIN_ROUTES),
}
~~~

An NgModule is not required for this standalone route-array pattern.

## 6. Does `import()` return a component directly?

It returns a promise for the imported module. Select the named component from that module:

~~~ts
loadComponent: () =>
  import('./reports/reports.page').then(module => module.ReportsPage)
~~~

If the file uses a default export, the module import can be returned directly when it resolves to the component.

## 7. Does writing a dynamic import guarantee the exact chunk layout?

It creates an async import boundary for the build system, but the production bundler determines the emitted chunks. Build the app and inspect its output and network requests to verify the actual split.

## 8. Should the home page also be lazy-loaded?

It depends on the app. Angular's guidance generally favors eager loading primary landing pages and lazy-loading other pages or features. Measure the initial bundle and first-navigation experience before choosing.

## 9. How large should a lazy route boundary be?

Choose a meaningful feature or screen boundary. A substantial admin area with several routes is a good candidate for `loadChildren`. An isolated heavy screen can use `loadComponent`. Splitting every tiny component can add requests and navigation delay without meaningfully shrinking the initial bundle.

## 10. Can I nest lazy-loaded routes?

Yes, but deep nesting can create sequential downloads before the final page appears. Keep lazy boundaries aligned with real feature boundaries and check the first navigation into deeply nested areas.

## 11. What is route preloading?

Preloading downloads lazy route configurations in the background after initial navigation, before a visitor opens them. It can shorten the first navigation delay while spending extra bandwidth and memory.

## 12. Is preloading enabled by default?

No. The router's default preloading strategy is `NoPreloading`. Only load the lazy route configuration when navigation needs it unless you configure another strategy.

## 13. What does `PreloadAllModules` do?

It preloads all lazy-loaded route modules after initial navigation. It is useful only when its bandwidth and memory costs fit the application. Measure it, especially for large apps or users on limited connections.

## 14. Does `PreloadAllModules` necessarily preload every `loadComponent` chunk?

Do not assume so. The built-in strategy is documented for lazy-loaded route modules/configurations. Check Angular's current behavior for the version and route pattern you use, and inspect the network/build output.

## 15. How is route lazy loading different from `@defer`?

Route lazy loading defers whole screens or route configurations until navigation. `@defer` defers eligible template dependencies within a component that is already active, often based on viewport visibility or interaction.

## 16. Can I use dependency injection inside a lazy loader?

Yes. Angular runs `loadComponent` and `loadChildren` loader functions in the current route's injection context. The loader can use `inject()` for providers available at that route, inherited from a parent, or provided globally.

## 17. Does lazy loading protect a feature from unauthorized users?

No. Lazy-loaded code still runs in the visitor's browser and can be downloaded or invoked. Use route guards to shape navigation and enforce authorization in the server/API for protected data and operations.

## 18. Will users see a delay the first time they open a lazy route?

Possibly. The browser may need to request and parse a new chunk. Show navigation feedback if that delay is noticeable. Preloading may reduce the wait at the cost of earlier network use.

## 19. What if a lazy chunk fails to load?

The navigation can fail. Handle router navigation errors at the app level and provide a recovery message. Also account for deployment and caching: an already-open app may request a chunk that no longer exists after a release.

## 20. Does the component need its own imports after lazy loading?

Yes. A standalone component still declares the components, directives, and pipes used in its template in its own `imports` array. Lazy loading controls when its code is downloaded; it does not change template import rules.

## 21. How should I confirm that lazy loading improved the app?

Compare production build output and browser network behavior. Check the initial transferred JavaScript, whether the feature chunk is fetched on navigation, the delay on first and later visits, and the effect of any preloading. Avoid relying on source syntax alone.

## 22. Where can I read the official guidance?

- [Lazy-loaded routes and performance](https://angular.dev/best-practices/performance/lazy-loaded-routes)
- [Route loading strategies](https://angular.dev/guide/routing/loading-strategies)
- [Preloading and route behavior](https://angular.dev/guide/routing/customizing-route-behavior)
- [Standalone component imports](https://angular.dev/guide/components/importing)

## Related material

- [Lesson: Lazy-load routes and standalone components](../lessons/080-lazy-load-routes-and-standalone-components.md)
- [Example: Lazy-load routes and standalone components](../examples/080-lazy-load-routes-and-standalone-components.md)
- [Resolvers, route data, and loading a page before activation](../lessons/079-resolvers-route-data-and-loading-a-page-before-activation.md)
