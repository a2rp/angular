# Query parameters, fragments, and navigation state: Q&A

## 1. What is a query parameter?

A query parameter is optional URL state written after `?`. For example, `/products?sort=price&page=2` has the query values `sort=price` and `page=2`. It does not change which route path matches.

## 2. What is a fragment?

A fragment is the part of the URL after `#`, such as `results` in `/products?sort=price#results`. It commonly identifies a section of a page. The fragment is client-side URL state and is not sent to the server in an HTTP request.

## 3. How are path parameters, query parameters, and fragments different?

- A path parameter is a named segment in the route pattern, such as `:productId` in `products/:productId`.
- A query parameter is optional key/value state after `?`, often used for filtering or pagination.
- A fragment follows `#` and typically names a section or anchor.

Angular exposes these values through different route properties.

## 4. How do I read a query parameter once?

Read it from the route snapshot's query parameter map:

~~~ts
const sort = this.route.snapshot.queryParamMap.get('sort');
~~~

The result is `string | null`. Provide a default and validate values before using them.

## 5. How do I respond when a query parameter changes?

Use the reactive `ActivatedRoute.queryParamMap` observable. Map it to the value the component needs and use `AsyncPipe` or convert it to a signal with `toSignal`. A field read once from the snapshot will not automatically change when the URL is updated.

## 6. Are query parameter values numbers or booleans?

They are URL strings when read from the route. Convert and validate them explicitly. For example, check that a page string is a positive integer before using it as a page number.

## 7. How can I add query parameters with a link?

Bind an object to `queryParams`:

~~~html
<a
  routerLink="/products"
  [queryParams]="{sort: 'price', page: 1}"
>
  Sort by price
</a>
~~~

This creates a URL like `/products?sort=price&page=1`.

## 8. What is the default query parameter handling behavior?

The new query parameter object replaces the current query state by default. If the current URL is `/products?category=books&page=2` and a navigation provides only `sort=price`, the old keys are not automatically retained.

## 9. What does `queryParamsHandling="merge"` do?

It combines the provided keys with the current query parameters. If the same key appears in both places, the new value takes precedence.

For example, merging `sort=price` into `?category=books&page=2` yields `?category=books&page=2&sort=price`.

## 10. What does `queryParamsHandling="preserve"` do?

It keeps the current query parameters and discards new query parameter values supplied for that navigation. Use it when the current query state should remain as-is, not when you need to add or update a key.

## 11. When should I choose replace, merge, or preserve?

- **Replace** when the new navigation should define the complete query state. This is the default.
- **Merge** when you are updating a few keys and want unrelated keys to remain.
- **Preserve** when all current query parameters should carry to the destination and no new values should be applied.

Choose based on the feature's behavior; preserving stale filters can be just as surprising as accidentally dropping useful ones.

## 12. How can I update query parameters from TypeScript?

Inject `Router` and call `navigate()`. An empty commands array keeps the current route; `relativeTo` makes the route context explicit:

~~~ts
void this.router.navigate([], {
  relativeTo: this.route,
  queryParams: {sort: 'price'},
  queryParamsHandling: 'merge',
});
~~~

## 13. How do I add a fragment to a router link?

Use the `fragment` input:

~~~html
<a routerLink="/guides" fragment="accessibility">
  Accessibility section
</a>
~~~

The resulting URL is `/guides#accessibility`. The page should have an element with a matching `id` if the fragment is intended to identify a section.

## 14. How do I set a fragment programmatically?

Pass `fragment` in the navigation extras:

~~~ts
void this.router.navigate(['/guides'], {
  fragment: 'accessibility',
});
~~~

The route also exposes its current fragment as `route.fragment` and `route.snapshot.fragment`.

## 15. Does setting a fragment automatically scroll to the element?

Not necessarily. Angular's in-memory anchor scrolling is disabled by default. Enable it with `withInMemoryScrolling({anchorScrolling: 'enabled'})` in the router providers. The matching element must exist in the rendered page.

## 16. What happens to fragment scrolling on Back and Forward?

When Angular's scrolling feature is enabled, anchor scrolling does not run on a `popstate` navigation. The router uses its scroll-position restoration behavior instead. Test history navigation separately from clicking a fragment link.

## 17. What is Angular navigation `state`?

`state` is developer-provided data attached to a navigation. It is not placed in the URL. Angular writes it into the browser history entry's `history.state`, together with an Angular-generated `navigationId`.

## 18. When should I use navigation `state` rather than query parameters?

Use query parameters for state that should be visible, copyable, bookmarkable, and available when a URL is opened directly. Use navigation `state` for supplementary context attached to a particular browser-history entry when it does not belong in the URL. Do not make a page depend on navigation state if the page must work from a direct link.

## 19. Is navigation `state` private or secure?

No. It is client-side browser history data, not a security boundary. Do not use it to store secrets or to replace server-side authorization.

## 20. How can the destination read navigation state during navigation?

During an in-progress navigation, inject `Router` and read `currentNavigation()?.extras.state`:

~~~ts
readonly source =
  this.router.currentNavigation()?.extras.state?.['source'] ?? null;
~~~

The current navigation is only available while navigation is in progress. After completion, state is in the current history entry's `history.state`.

## 21. What is the difference between navigation `state` and `info`?

`state` is persisted in browser history. `info` is transient information for the current navigation and is not written to history. It can be useful for short-lived behavior such as deciding which transition animation to use.

## 22. Why should application code not compare `history.state` objects by identity?

Angular adds a `navigationId` to the history state. The object therefore may not be reference-equal to the object supplied by application code. Read the specific application key you need instead.

## 23. Which route API should I use for query parameters and fragments?

- Use `snapshot.queryParamMap` for a one-time query parameter read.
- Use `queryParamMap` for reactive query changes.
- Use `snapshot.fragment` for a one-time fragment read.
- Use `fragment` for reactive fragment changes.

These are distinct from `paramMap` and `params`, which represent route-specific path or matrix parameters.

## 24. What are common mistakes with query strings and fragments?

- Reading a query key from `paramMap` instead of `queryParamMap`.
- Assuming a query string value already has a numeric or boolean type.
- Expecting query parameters to merge by default.
- Using `preserve` while expecting new query keys to be applied.
- Expecting fragment scrolling without enabling anchor scrolling.
- Depending on navigation `state` for a page that must work from a shared or direct URL.
- Treating URL or history state as secure storage.

## 25. Where can I verify the APIs?

See Angular's [route state guide](https://angular.dev/guide/routing/read-route-state), [RouterLink API](https://angular.dev/api/router/RouterLink), [NavigationExtras API](https://angular.dev/api/router/NavigationExtras), [QueryParamsHandling API](https://angular.dev/api/router/QueryParamsHandling), and [in-memory scrolling options](https://angular.dev/api/router/InMemoryScrollingOptions).
