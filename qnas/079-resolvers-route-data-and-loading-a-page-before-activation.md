# Resolvers, route data, and loading a page before activation: Q&A

## 1. What is an Angular route resolver?

A resolver is a function that supplies data for a route before Angular activates its destination component. The router waits for its result, then places that value in the route's resolved data.

## 2. When should I use a resolver?

Use one when a page needs the result before its first meaningful render, such as a product detail route that cannot display without the product. Optional or below-the-fold content can often load after activation with a component or store loading state.

## 3. What is the functional resolver type?

Use `ResolveFn<T>` from `@angular/router`:

~~~ts
export const productResolver: ResolveFn<Product> = route => {
  return inject(ProductCatalog).getById(
    route.paramMap.get('productId')!,
  );
};
~~~

The function receives the target route snapshot and router state.

## 4. What can a resolver return?

A resolver can return a value directly, a `Promise`, or an `Observable`. It can also return a `RedirectCommand` to redirect the current navigation. Its type is represented by `MaybeAsync<T | RedirectCommand>`.

## 5. How do I attach a resolver to a route?

Add it to the route's `resolve` map. The map key becomes the route-data key:

~~~ts
{
  path: 'products/:productId',
  component: ProductDetailPage,
  resolve: {product: productResolver},
}
~~~

The `product` key is where the component reads the resolved value.

## 6. How does the router know which parameter to pass?

Read the target route snapshot's parameter map. For a path `products/:productId`, read `route.paramMap.get('productId')`. Check for a missing value when the route structure does not guarantee it.

## 7. How does the component access resolved data?

Two common options are:

- Read `ActivatedRoute.data` or `ActivatedRoute.snapshot.data`.
- Enable `withComponentInputBinding()` and declare a component input matching the resolver key.

For example, a resolver key named `product` can bind to `product = input.required<Product>()`.

## 8. Does component input binding only include resolver results?

No. With `withComponentInputBinding()`, router state can also bind matching inputs from path parameters, query parameters, static route data, and resolved data. Use distinct keys to avoid unintended collisions. The router sets a matching input to `undefined` when there is no corresponding route value.

## 9. Does a resolver show a loading indicator inside the destination component?

Not by itself. The destination component is not activated until its resolver data is ready. The rest of the app can show navigation progress, for example by observing router navigation state. If the destination should render immediately with skeletons or an in-page spinner, use a non-blocking data-loading pattern instead.

## 10. Do resolvers run before route guards?

No. Relevant guards run first. Resolvers run only after the guards allow navigation, so the router does not resolve destination data for a navigation that was rejected by a guard.

## 11. What is the execution order for nested routes?

Parent route resolvers run before child route resolvers. A child resolver can read resolved parent data. This dependency is sequential and can create a network waterfall, so avoid nesting requests when the child does not need the parent's result.

## 12. Do resolvers automatically cache data?

No. A resolver only supplies data for navigation. Use an appropriate service or store cache when the application needs reuse, invalidation, or refresh after mutations.

## 13. Can a resolver redirect when a record is missing?

Yes. Return a `RedirectCommand`:

~~~ts
return new RedirectCommand(router.parseUrl('/not-found'));
~~~

Returning a redirect makes the router cancel the current target navigation and proceed to the specified destination.

## 14. What happens if a resolver throws or its request errors?

The navigation fails with a `NavigationError` unless the application handles it. Handle a known case in the resolver, such as mapping a 404 to a not-found redirect, or centralize navigation failures with `withNavigationErrorHandler` or a router event handler.

Do not turn every server error into “not found.” Distinguish missing records, authorization failures, and unexpected network or server errors.

## 15. What if an observable completes without emitting?

It does not supply the route data. Ensure an observable resolver emits a value or deliberately redirect/error. Do not use an empty stream as a substitute for a route result.

## 16. Can a resolver access injected services?

Yes. A functional resolver runs in an injection context, so it can call `inject()` synchronously to get a service. If it uses `await`, capture injected dependencies before awaiting.

## 17. Should I fetch every page request in the resolver?

No. Resolve data that must be available before activation. Fetch later or optional sections after activation so one slow secondary request does not hold up the whole route.

## 18. What are route resources, and how are they different?

Route resources integrate Angular resource APIs with route state and can support reactive parameters, parallel loading across matched routes, in-place reloads, and non-blocking loading states. The official routing docs currently mark route resources as developer preview, so check their stability before using them in a production dependency.

A traditional resolver blocks route activation until its data is ready. Choose according to whether the data is a prerequisite for the destination.

## 19. Does resolved data enforce authorization?

No. The resolver runs in the client application. The backend must still authorize every request and protect every record. A route guard or resolver does not prevent someone from calling an API directly.

## 20. What should I check when debugging a resolver?

Check that the route key matches the component input or route-data lookup, the route parameter name is correct, the service emits or resolves a value, and errors are handled. Also inspect router navigation events to see whether the navigation completed, was canceled, or failed.

## 21. Can the resolver also set a route title?

Yes. Route configuration supports a static title or a title resolver. Keep title resolution focused on metadata needed for the route, and avoid duplicating expensive data requests when the page's main resolver already fetches that record.

## 22. Where are the official references?

- [Angular data resolvers guide](https://angular.dev/guide/routing/data-resolvers)
- [ResolveFn API](https://angular.dev/api/router/ResolveFn)
- [withComponentInputBinding API](https://angular.dev/api/router/withComponentInputBinding)
- [withNavigationErrorHandler API](https://angular.dev/api/router/withNavigationErrorHandler)
- [Route resources guide](https://angular.dev/guide/routing/data-fetching-with-resources)

## Related material

- [Lesson: Resolvers, route data, and loading a page before activation](../lessons/079-resolvers-route-data-and-loading-a-page-before-activation.md)
- [Example: Resolvers, route data, and loading a page before activation](../examples/079-resolvers-route-data-and-loading-a-page-before-activation.md)
- [Route guards and access control boundaries](../lessons/078-route-guards-and-access-control-boundaries.md)
