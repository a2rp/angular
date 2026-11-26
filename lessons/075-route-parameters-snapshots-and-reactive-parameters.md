# Route parameters, snapshots, and reactive parameters

Route parameters put identifying values in a URL path. Angular matches a path such as `products/42` against a route pattern such as `products/:productId` and makes the value available to the routed component.

Angular exposes route state through `ActivatedRoute`. A snapshot is useful when a component only needs the value at the moment it is created. The reactive `paramMap` observable is appropriate when the same routed component can remain active while its parameter changes.

## Learning outcomes

After this lesson, you should be able to:

- Declare a named route parameter with a colon in a route path.
- Read a parameter from `ActivatedRoute.snapshot.paramMap`.
- Read parameter changes from `ActivatedRoute.paramMap`.
- Explain why a property initialized from a snapshot can become stale.
- Handle a missing value safely and validate route values before using them.
- Distinguish path parameters from query parameters.

## Declare a route parameter

A route parameter is a named segment in a route path. Prefix its name with a colon:

~~~ts
import {Routes} from '@angular/router';

export const routes: Routes = [
  {
    path: 'products/:productId',
    component: ProductDetailPage,
  },
];
~~~

For this route, `/products/42` matches and supplies the string `42` for `productId`. A value such as `/products/desk-lamp` supplies `desk-lamp` instead. The parameter name in the route configuration must match the key used to read it.

A URL parameter is part of the address and can be bookmarked, shared, or entered directly. It is visible to the user, so do not put passwords, access tokens, or private data in it. Treat values from the URL as untrusted input: validate their shape and check that the requested resource is available.

## Get the active route with `ActivatedRoute`

`ActivatedRoute` is provided to a routed component and describes the route currently associated with it. Inject it with Angular's `inject` function:

~~~ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-product-detail',
  template: '<h1>Product detail</h1>',
})
export class ProductDetailPage {
  private readonly route = inject(ActivatedRoute);
}
~~~

The active route exposes both a snapshot and observable state. For path parameters, the map forms are usually convenient:

- `route.snapshot.paramMap` reads the parameters in the current snapshot.
- `route.paramMap` is an observable that emits parameter maps as the active route state changes.
- `paramMap.get(name)` returns a string or `null`.
- `paramMap.has(name)` checks whether the key exists.
- `paramMap.getAll(name)` returns all values for a key.

Use the map API when a parameter may be absent. It makes the null case explicit instead of assuming every route value exists.

## Read a snapshot once

A route snapshot represents route state at one point in navigation. Read its parameter map when a component is created if the component only needs the initial value:

~~~ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-product-detail',
  template: '<h1>Product: {{ productId ?? "not selected" }}</h1>',
})
export class ProductDetailPage {
  private readonly route = inject(ActivatedRoute);

  readonly productId: string | null =
    this.route.snapshot.paramMap.get('productId');
}
~~~

A snapshot is simple and avoids a subscription. It is a good fit when the routed component is replaced whenever the relevant parameter changes, or when the value is only needed for a one-time decision.

Do not confuse the snapshot with a live stream. If you copy a snapshot value into a field once and Angular later reuses that component for a different parameter, the field you assigned does not update itself.

## React to parameter changes

Angular can reuse the same component instance when navigation changes only a parameter for the same route. For example, moving from `/products/42` to `/products/99` may keep the product-detail component active. A value copied once from the snapshot can then continue to show `42`.

Use `route.paramMap` when the displayed data must follow route changes. One modern option is to map the observable to the value you need and convert it to a signal with `toSignal`:

~~~ts
import {Component, computed, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';
import {toSignal} from '@angular/core/rxjs-interop';
import {map} from 'rxjs';

@Component({
  selector: 'app-product-detail',
  template: '<h1>Product: {{ productId() ?? "not selected" }}</h1>',
})
export class ProductDetailPage {
  private readonly route = inject(ActivatedRoute);

  readonly productId = toSignal(
    this.route.paramMap.pipe(
      map((paramMap) => paramMap.get('productId')),
    ),
    {initialValue: null},
  );
}
~~~

The resulting signal updates whenever the route parameter map emits. The initial value is `null`, which keeps the type honest and gives the template a safe state before a value is available. `toSignal` subscribes to the observable and automatically cleans up with the component's injection context.

You can also keep the value as an observable and use Angular's `AsyncPipe` in the template. Whichever approach you choose, use the live route state when the component must respond to parameter changes.

## Use the parameter safely

Route parameters are strings. A parameter that looks numeric is still a string until the application parses and validates it:

~~~ts
const rawId = this.route.snapshot.paramMap.get('productId');

if (rawId === null || !/^\d+$/.test(rawId)) {
  // Show an invalid or not-found state.
  return;
}

const productId = Number(rawId);
~~~

For a real application, validation should match the identifier format your data layer expects. IDs may be numeric, UUIDs, slugs, or another string format; do not convert every ID to a number by default.

After validating its format, use the identifier to request or select the matching data. A syntactically valid ID can still refer to a record that does not exist, so handle the not-found result as well.

## Path parameters and query parameters

A path parameter identifies a segment in the route pattern:

- Route pattern: `products/:productId`
- URL: `/products/42`
- Read it with: `route.paramMap.get('productId')`

A query parameter follows `?` and does not change which path pattern matches:

- URL: `/products?sort=price&page=2`
- Read it with: `route.queryParamMap.get('sort')`

Path parameters commonly identify a resource; query parameters commonly represent optional view state such as sorting or filtering. Angular exposes them through different maps. A later lesson covers query parameters in detail.

## Common mistakes

- **Using a different key name:** `:productId` must be read as `productId`, not `id`.
- **Expecting a number:** parameter values are strings; parse and validate them when needed.
- **Ignoring the nullable result:** `get()` returns `string | null`. Handle the missing case instead of using a non-null assertion without a reason.
- **Using a snapshot for a live value:** a field initialized once from the snapshot will not itself react to later route changes. Use `paramMap` for reactive behavior.
- **Treating URL values as trusted:** validate them and handle IDs that do not identify an existing resource.
- **Confusing path and query parameters:** read path values from `paramMap` and query values from `queryParamMap`.
- **Putting secrets in a route:** URLs can be visible in browser history, logs, screenshots, and referrer data.

## Choosing between snapshot and reactive state

Use a snapshot for a one-time read when the component's lifetime and use case make that sufficient. Use the `paramMap` observable when a routed component should update as navigation changes its parameters. In either case, keep the value nullable until it has been checked, validate it for the domain, and provide a clear invalid or not-found state.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="074-routeroutlet-routerlink-and-active-route-links.md">074 - RouterOutlet, RouterLink, and active route links</a></td><td><a href="../qnas/075-route-parameters-snapshots-and-reactive-parameters.md">Q&amp;A</a></td><td><a href="../examples/075-route-parameters-snapshots-and-reactive-parameters.md">Example</a></td><td><a href="076-query-parameters-fragments-and-navigation-state.md">076 - Query parameters, fragments, and navigation state</a></td>
</tr>
  </tbody>
</table>
