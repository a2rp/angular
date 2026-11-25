# Query parameters, fragments, and navigation state

A route path identifies which view to activate. Query parameters, URL fragments, and navigation state carry additional context around a navigation.

For a URL such as `/products?sort=price&page=2#results`:

- `/products` is the route path.
- `sort=price&page=2` are query parameters.
- `results` is the fragment.
- Angular navigation state, if supplied, is separate from the visible URL.

Use query parameters for optional, shareable view state such as filters, sorting, and pagination. Use a fragment to identify a section of the current page. Use navigation state for supplementary data attached to one browser-history navigation when it does not belong in the URL.

## Learning outcomes

After this lesson, you should be able to:

- Read query parameters and fragments from `ActivatedRoute`.
- React to query or fragment changes using the route observables.
- Add query parameters and fragments with `RouterLink` or `Router.navigate()`.
- Choose whether to replace, merge, or preserve current query parameters.
- Configure Angular's optional fragment scrolling.
- Explain how navigation state differs from URL state.

## Query parameters represent optional URL state

Query parameters follow `?` in a URL. They do not change the route path that matched. For example, `/products?sort=price&page=2` still matches the `products` route. Query values are strings in the URL and can be bookmarked, copied, or shared.

Common uses include:

- A search term, category, or filter.
- A selected page in a paginated list.
- A sort order or display mode.
- A tab selection when that selection should be linkable.

Avoid putting secrets or private information in query parameters. URLs can appear in browser history, logs, screenshots, and referrer information.

## Read query parameters with `ActivatedRoute`

Inject `ActivatedRoute` into the routed component. Use `queryParamMap` to read a value safely:

~~~ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-product-list',
  template: '<p>Sort order: {{ sort }}</p>',
})
export class ProductListPage {
  private readonly route = inject(ActivatedRoute);

  readonly sort =
    this.route.snapshot.queryParamMap.get('sort') ?? 'name';
}
~~~

`queryParamMap.get()` returns `string | null`. Supply a default for absent values and validate values before using them. For example, accept only known sort keys rather than allowing any URL string to control application logic.

A snapshot is a point-in-time read. When the same component should update as the URL query changes, use the observable `queryParamMap`. For a signal-based component, map that stream and convert it with `toSignal`:

~~~ts
import {Component, computed, inject} from '@angular/core';
import {toSignal} from '@angular/core/rxjs-interop';
import {ActivatedRoute} from '@angular/router';
import {map} from 'rxjs';

@Component({
  selector: 'app-product-list',
  template: '<p>Sort order: {{ sort() }}</p>',
})
export class ProductListPage {
  private readonly route = inject(ActivatedRoute);

  private readonly sortParam = toSignal(
    this.route.queryParamMap.pipe(
      map((params) => params.get('sort')),
    ),
    {initialValue: null},
  );

  readonly sort = computed(() => {
    const value = this.sortParam();
    return value === 'price' || value === 'name' ? value : 'name';
  });
}
~~~

This example responds when the user changes the query string while staying on the same route. A snapshot-based field would not automatically update.

## Add query parameters with `RouterLink`

Bind a query parameter object to an anchor:

~~~html
<a
  routerLink="/products"
  [queryParams]="{sort: 'price', page: 1}"
>
  Products by price
</a>
~~~

This creates a URL like `/products?sort=price&page=1`. Query parameters are replaced by the provided object by default; other current parameters are not automatically retained.

To add or update selected keys while keeping other current values, use `queryParamsHandling="merge"`:

~~~html
<a
  routerLink="/products"
  [queryParams]="{sort: 'price'}"
  queryParamsHandling="merge"
>
  Sort by price
</a>
~~~

If the current URL is `/products?category=books&page=2`, merging `sort=price` produces `/products?category=books&page=2&sort=price`. If the new object contains a key already present, the new value wins.

Use `queryParamsHandling="preserve"` when the next navigation should keep the current query parameters and discard any new query parameter object. Use the default replacement behavior when the new navigation should define the complete query state.

## Update query state programmatically

Use `Router.navigate()` when navigation follows an application action rather than a link:

~~~ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute, Router} from '@angular/router';

@Component({
  selector: 'app-product-list',
  template: '...',
})
export class ProductListPage {
  private readonly route = inject(ActivatedRoute);
  private readonly router = inject(Router);

  setSort(sort: 'name' | 'price'): void {
    void this.router.navigate([], {
      relativeTo: this.route,
      queryParams: {sort, page: 1},
      queryParamsHandling: 'merge',
    });
  }
}
~~~

An empty command array keeps the current route. `relativeTo` makes that intent explicit. Resetting `page` to `1` when sort order changes is a product decision; the router only updates the URL values you provide.

If a query parameter should be the only query state, omit `queryParamsHandling` or set it to `'replace'`. If unrelated filters should remain, choose `'merge'`. Choose deliberately so stale values do not unexpectedly affect the new view.

## Use a fragment to identify a page section

A fragment follows `#` and commonly identifies an element in the current page:

~~~html
<a routerLink="/guides" fragment="accessibility">
  Accessibility section
</a>

<section id="accessibility">
  <h2>Accessibility</h2>
  ...
</section>
~~~

The resulting URL is `/guides#accessibility`. You can also set a fragment programmatically:

~~~ts
void this.router.navigate(['/guides'], {
  fragment: 'accessibility',
});
~~~

The active route exposes the current fragment through `ActivatedRoute.fragment` and `ActivatedRoute.snapshot.fragment`. The observable is useful when the component needs to respond to fragment changes.

A URL fragment is not sent to the web server in an HTTP request. It is client-side navigation context for the document or application.

## Configure router fragment scrolling

Having a fragment in the URL does not by itself guarantee that Angular scrolls to the matching element. Angular's in-memory anchor scrolling is disabled by default. Enable it with `withInMemoryScrolling`:

~~~ts
import {ApplicationConfig} from '@angular/core';
import {
  provideRouter,
  withInMemoryScrolling,
} from '@angular/router';

import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withInMemoryScrolling({
        anchorScrolling: 'enabled',
      }),
    ),
  ],
};
~~~

When enabled, Angular scrolls to the element whose `id` matches the fragment after navigation. Back/Forward history navigation uses scroll restoration behavior rather than anchor scrolling. If the target content is rendered later or is not present, the application may need a different scrolling approach.

## Navigation state is separate from the URL

Navigation extras can attach a `state` object to a navigation:

~~~ts
void this.router.navigate(['/checkout'], {
  state: {source: 'cart'},
});
~~~

This value is not part of the URL. Angular writes it to the current browser history entry's `history.state` along with a router-generated `navigationId`. It can be useful for optional context, such as recording where a navigation originated. Do not use it for information that must be shareable, bookmarkable, or required to render a directly opened URL. Do not put secrets in it.

During an in-progress navigation, the target component can read the current navigation extras:

~~~ts
import {Component, inject} from '@angular/core';
import {Router} from '@angular/router';

@Component({
  selector: 'app-checkout',
  template: '<h1>Checkout</h1>',
})
export class CheckoutPage {
  private readonly router = inject(Router);

  readonly source =
    this.router.currentNavigation()?.extras.state?.['source'] ?? null;
}
~~~

The current navigation is only available while navigation is in progress. For data that must remain in the current history entry after navigation, the browser exposes `history.state`; Angular also includes its own navigation identifier there, so the whole object is not a stable application data object to compare by identity.

Angular also provides an `info` navigation extra for transient information about one navigation. Unlike `state`, `info` is not written to browser history. It is an advanced option for short-lived navigation context, such as choosing a transition animation. Keep the distinction clear:

| Value | In the URL? | Stored in browser history? | Typical use |
| --- | --- | --- | --- |
| Query parameters | Yes | As part of URL | Search, filters, sorting, pagination |
| Fragment | Yes | As part of URL | A section or anchor within a page |
| Navigation `state` | No | Yes, in `history.state` | Optional context for a history entry |
| Navigation `info` | No | No | Temporary context for the active navigation |

## Common mistakes

- **Reading a query key from `paramMap`:** use `queryParamMap` for values after `?`.
- **Assuming query values have TypeScript types:** they are URL strings; parse and validate them.
- **Expecting a query update to preserve other values by default:** use `queryParamsHandling="merge"` when you need to keep them.
- **Using `preserve` and expecting new keys to apply:** preserve keeps current query state and ignores supplied new values.
- **Expecting every fragment link to scroll automatically:** enable `anchorScrolling: 'enabled'` in router scrolling options.
- **Putting required page state only in navigation `state`:** direct links and bookmarks do not carry it; use the URL or load required data from a durable source.
- **Treating navigation state as secure storage:** it is client-side browser state, not an authorization mechanism.
- **Comparing `history.state` objects by identity:** Angular adds a navigation ID, so treat it as a bag of data and read the specific key you need.

## Official references

- [Read route state](https://angular.dev/guide/routing/read-route-state)
- [Navigate to routes](https://angular.dev/guide/routing/navigate-to-routes)
- [ActivatedRoute API](https://angular.dev/api/router/ActivatedRoute)
- [RouterLink API](https://angular.dev/api/router/RouterLink)
- [NavigationExtras API](https://angular.dev/api/router/NavigationExtras)
- [QueryParamsHandling API](https://angular.dev/api/router/QueryParamsHandling)
- [InMemoryScrollingOptions API](https://angular.dev/api/router/InMemoryScrollingOptions)
