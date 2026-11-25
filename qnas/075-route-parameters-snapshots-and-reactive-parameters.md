# Route parameters, snapshots, and reactive parameters: Q&A

## 1. What is a route parameter?

A route parameter is a named value in a URL path segment. In a route pattern such as `products/:productId`, the colon introduces a parameter named `productId`. A URL such as `/products/42` supplies the value `42`.

## 2. How do I declare a route parameter?

Put a colon and parameter name in the route path:

~~~ts
{
  path: 'products/:productId',
  component: ProductDetailPage,
}
~~~

The spelling after the colon is the key used to retrieve the value from the activated route.

## 3. How do I get the currently activated route?

Inject `ActivatedRoute` into a routed component:

~~~ts
private readonly route = inject(ActivatedRoute);
~~~

Angular provides an `ActivatedRoute` for the route associated with the component, including its parameters, query parameters, route data, and snapshot.

## 4. How do I read a route parameter from a snapshot?

Read the key from `route.snapshot.paramMap`:

~~~ts
const productId = this.route.snapshot.paramMap.get('productId');
~~~

The result is a `string | null`. Handle the missing case instead of assuming the value exists.

## 5. What is a route snapshot?

A snapshot is a static view of route state at one point in navigation. It is useful for a one-time read, such as when a component only needs the parameter value at creation.

A field assigned from a snapshot does not automatically update just because the route later changes.

## 6. When is a snapshot enough?

A snapshot is enough when the component only needs the value for its current activation and does not need to react to later changes to that parameter. It keeps the code simple and does not require an observable subscription.

## 7. When should I use `route.paramMap` instead?

Use `route.paramMap` when the component should react if its route parameters change while it remains active. Angular may reuse a component when only a parameter changes, so code that copies the initial snapshot into a field may keep showing the old value.

## 8. How do I read parameter changes as an observable?

Use `paramMap` and map it to the value the component needs:

~~~ts
readonly productId$ = this.route.paramMap.pipe(
  map((paramMap) => paramMap.get('productId')),
);
~~~

The stream emits when the active route's parameter map changes. Use the observable in the template with `AsyncPipe` or convert it to a signal with `toSignal`.

## 9. How can I convert a route parameter observable to a signal?

Use `toSignal` from `@angular/core/rxjs-interop`:

~~~ts
readonly productId = toSignal(
  this.route.paramMap.pipe(
    map((paramMap) => paramMap.get('productId')),
  ),
  {initialValue: null},
);
~~~

The signal tracks later emissions, and `initialValue: null` represents the period before a string value is available. Create `toSignal` in an injection context, such as a component field initializer.

## 10. Does a route parameter arrive as a number if the URL contains digits?

No. Route parameters are strings. If the application needs a number, first check that the value is present and matches the expected format, then convert it. Do not assume all IDs are numeric; many applications use UUIDs or slugs.

## 11. What do `ParamMap.get()`, `has()`, and `getAll()` return?

- `get(name)` returns one value as `string | null`.
- `has(name)` reports whether the map contains the key.
- `getAll(name)` returns all values for a key as a string array.

The map API makes missing values explicit.

## 12. What if the parameter is missing?

A route pattern that requires a parameter normally will not match a URL missing that segment. However, `ParamMap.get()` remains nullable by type. Keep the missing-value handling because it protects against route changes, optional data, or assumptions that do not hold in every context.

## 13. Why might a value initialized from a snapshot become stale?

Angular's router can reuse a routed component when navigation changes only a parameter on the same route. A field initialized once from `route.snapshot.paramMap` does not get reassigned automatically. A reactive `route.paramMap` stream updates with the route state.

## 14. Are route parameters the same as query parameters?

No. Route parameters are part of the path and are declared in the route pattern, for example `/products/42`. Query parameters follow `?` and do not change the path pattern, for example `/products?sort=price`. Angular exposes them through `paramMap` and `queryParamMap` respectively.

## 15. How can I distinguish path and query parameters in code?

~~~ts
const id = this.route.snapshot.paramMap.get('productId');
const sort = this.route.snapshot.queryParamMap.get('sort');
~~~

Use the map that matches where the value appears in the URL.

## 16. Should sensitive information be stored in a route parameter?

No. URL values can appear in browser history, server logs, screenshots, and referrer information. Never put passwords, access tokens, or private data in route parameters.

## 17. Is a syntactically valid parameter guaranteed to identify a real record?

No. A value can have a valid format and still refer to a resource that does not exist or that the visitor cannot access. Validate its format, handle data lookup failures, and enforce authorization on the server.

## 18. How should an observable subscription be managed?

Prefer the `AsyncPipe` or `toSignal` for values displayed by a component; they integrate with the component lifecycle. If subscribing manually, manage the subscription lifetime with an appropriate cleanup strategy so destroyed components do not keep unnecessary work alive.

## 19. What if my component uses the wrong parameter key?

The key must match the route declaration. For `path: 'products/:productId'`, read `productId`. Reading `id` returns `null` unless the route also defines an `id` parameter.

## 20. Can a component read a parameter from a parent route?

Route parameters are associated with the activated route tree. If a parameter belongs to a parent route, inspect the appropriate parent route state or configure component input binding as appropriate for the application. Do not assume every child route's own parameter map contains every ancestor value.

## 21. What is component input binding for route values?

The optional `withComponentInputBinding()` router feature binds matching route values, query parameters, and route data to component inputs. It is an alternative API for suitable applications. This chapter uses `ActivatedRoute` to make the snapshot and reactive parameter sources explicit.

## 22. What should I test when a parameterized page is added?

Check that:

1. A valid URL matches the parameterized route.
2. The component reads the parameter with the correct key.
3. Missing or malformed values show a suitable state.
4. Navigating from one ID to another updates the view when the component is reused.
5. A valid-looking but unknown ID shows a not-found state.
6. Query parameters are read from `queryParamMap` rather than `paramMap`.

## 23. Where can I verify the behavior?

See Angular's official [route state guide](https://angular.dev/guide/routing/read-route-state), [ActivatedRoute API](https://angular.dev/api/router/ActivatedRoute), [ParamMap API](https://angular.dev/api/router/ParamMap), and [component input binding API](https://angular.dev/api/router/withComponentInputBinding).
