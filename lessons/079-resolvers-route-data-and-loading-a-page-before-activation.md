# Resolvers, route data, and loading a page before activation

A route resolver loads essential data during navigation, before Angular activates the destination component. This is useful when the page cannot render meaningfully without a record, such as a product detail page that needs its product before showing the page.

Resolvers make navigation wait for that data. Use them for information that is required at first render, not for every request a page may ever make. If a section can appear immediately with a loading state, load it after activation or consider a reactive resource approach.

## Learning outcomes

After this lesson, you should be able to:

- Write a functional resolver with the `ResolveFn<T>` type.
- Read route parameters and return data from an injected service.
- Attach one or more resolvers under a route's `resolve` property.
- Read resolved values through `ActivatedRoute` or component input binding.
- Handle missing records, redirects, and resolver failures deliberately.
- Explain that a resolver blocks activation and that nested parent-to-child resolvers can add sequential wait time.

## When to use a resolver

Use a resolver when the destination page needs its data to be ready before its component is activated. Examples include:

- A detail route that requires a valid record.
- A page where several pieces of essential route data must be present together.
- Initial data needed for a consistent server-rendered route.

A resolver is a navigation prerequisite, not a general-purpose data-fetching layer. It does not automatically cache data, refresh it after a mutation, or replace the page's normal loading and error states for later requests.

If the information is optional, large, or below the fold, it is often better to activate the component first and show a loading state while that part loads. Angular's newer route resources support reactive and non-blocking data loading, but the official documentation currently marks route resources as developer preview. Check their release status before depending on them in a stable application.

## Create a functional resolver

A resolver is a function typed with `ResolveFn<T>`. It receives the target `ActivatedRouteSnapshot` and `RouterStateSnapshot`, and it may use dependency injection.

~~~ts
import {inject} from '@angular/core';
import type {
  ActivatedRouteSnapshot,
  ResolveFn,
  RouterStateSnapshot,
} from '@angular/router';

import {ProductCatalog} from './product-catalog';
import type {Product} from './product';

export const productResolver: ResolveFn<Product> = (
  route: ActivatedRouteSnapshot,
  _state: RouterStateSnapshot,
) => {
  const catalog = inject(ProductCatalog);
  const productId = route.paramMap.get('productId');

  if (!productId) {
    throw new Error('The product route requires a productId.');
  }

  return catalog.getById(productId);
};
~~~

The service can return a product directly, a `Promise<Product>`, or an `Observable<Product>`. The router waits for the resolver result before activating the route. A resolver that needs an asynchronous check can also return a `RedirectCommand` to send the navigation elsewhere.

Call `inject()` while the resolver function is running in Angular's injection context. If an async resolver uses `await`, capture injected dependencies before the first `await`.

## Attach resolved data to a route

Put resolver functions in the route's `resolve` map. The key becomes the name under which the result is stored in the route data:

~~~ts
import type {Routes} from '@angular/router';

import {ProductDetailPage} from './product-detail.page';
import {productResolver} from './product.resolver';

export const routes: Routes = [
  {
    path: 'products/:productId',
    component: ProductDetailPage,
    resolve: {
      product: productResolver,
    },
  },
];
~~~

For a visit to `/products/42`, the router passes `42` as the `productId` parameter. It runs the resolver, waits for the result, then activates `ProductDetailPage` with the resolved value under the `product` key.

A route may have multiple resolver keys:

~~~ts
resolve: {
  product: productResolver,
  recommendations: recommendationsResolver,
}
~~~

Keep each resolved value focused on data the initial route actually requires. Avoid making unrelated resolver work a prerequisite for the page.

## Read the result in the component

### Bind resolved keys to component inputs

Configure `withComponentInputBinding()` with the router and define an input whose name matches the resolver key:

~~~ts
import {provideRouter, withComponentInputBinding} from '@angular/router';

import {routes} from './app.routes';

export const appConfig = {
  providers: [
    provideRouter(routes, withComponentInputBinding()),
  ],
};
~~~

~~~ts
import {Component, input} from '@angular/core';

import type {Product} from './product';

@Component({
  selector: 'app-product-detail',
  standalone: true,
  template: '<h1>{{ product().name }}</h1><p>{{ product().description }}</p><p>Price: {{ product().price }}</p>',
})
export class ProductDetailPage {
  readonly product = input.required<Product>();
}
~~~

The input name `product` matches the key in `resolve`. This keeps route data explicit in the component's interface. Component input binding can also bind other router state, such as path parameters and query parameters. Give inputs distinct names so a route value does not unintentionally override another value with the same key. A matching input with no value is set to `undefined`; use a required input or a deliberate default when the value must exist.

### Read route data with `ActivatedRoute`

You can also read resolved values from the route's `data`. The observable form reacts when the route data changes:

~~~ts
import {Component, computed, inject} from '@angular/core';
import {toSignal} from '@angular/core/rxjs-interop';
import {ActivatedRoute} from '@angular/router';

import type {Product} from './product';

@Component({
  selector: 'app-product-detail',
  standalone: true,
  template: '<h1>{{ product().name }}</h1>',
})
export class ProductDetailPage {
  private readonly route = inject(ActivatedRoute);
  private readonly data = toSignal(this.route.data, {requireSync: true});

  readonly product = computed(
    () => this.data()['product'] as Product,
  );
}
~~~

`route.snapshot.data` is convenient when a component only needs the current resolved snapshot. Use the observable or input-binding approach if the component must respond when the route's resolved data changes.

## Redirect when essential data cannot be loaded

A resolver can return a `RedirectCommand`. This is useful when the route has a valid alternative, such as sending an unknown product to a not-found page.

~~~ts
import {inject} from '@angular/core';
import {HttpErrorResponse} from '@angular/common/http';
import {RedirectCommand, Router} from '@angular/router';
import type {ResolveFn} from '@angular/router';
import {catchError, of} from 'rxjs';

import {ProductCatalog} from './product-catalog';
import type {Product} from './product';

export const productResolver: ResolveFn<Product | RedirectCommand> = route => {
  const catalog = inject(ProductCatalog);
  const router = inject(Router);
  const productId = route.paramMap.get('productId');

  if (!productId) {
    return new RedirectCommand(router.parseUrl('/not-found'));
  }

  return catalog.getById(productId).pipe(
    catchError((error: unknown) => {
      if (error instanceof HttpErrorResponse && error.status === 404) {
        return of(new RedirectCommand(router.parseUrl('/not-found')));
      }

      throw error;
    }),
  );
};
~~~

This example treats an HTTP 404 as a missing product and lets other failures propagate. Decide how the application should handle network failures and server errors rather than turning every failure into “not found.” A resolver error becomes a navigation error; an application can handle navigation errors centrally with `withNavigationErrorHandler`, subscribe to `NavigationError`, or handle a specific case in the resolver.

## Understand resolver order and navigation time

Angular runs route guards first. Resolvers run only after all relevant guards allow navigation. This avoids fetching protected page data for a navigation the guards already reject.

Resolvers on a parent route run before resolvers on child routes. Child resolvers can read resolved parent data, but this creates a sequential dependency: the child waits for the parent. A long parent request followed by a long child request can make navigation feel slow. If the child does not depend on the parent's result, avoid nesting resolver work unnecessarily.

The router does not activate the destination component until its required resolver values are ready. Give users feedback when the delay may be noticeable, for example with a top-level navigation progress indicator. Set reasonable request timeouts and keep resolver work small.

## Errors, empty results, and cancellation

Plan for the cases where data cannot be resolved:

- Return a `RedirectCommand` when there is a meaningful alternate destination.
- Let unexpected failures reach a centralized navigation error handler, or handle them explicitly in the resolver.
- Ensure observable-based data sources emit a value. An observable that completes without emitting cannot supply the required route data.
- Do not return a placeholder object that looks like valid data when the request failed.
- Keep backend authorization in the API. Resolving a route record does not make that record private or secure.

## Resolver or component loading?

| Situation | Prefer |
| --- | --- |
| The page cannot render without a record | Resolver |
| Optional content can appear after the page | Component or store loading state |
| A request should be reactive to changing signal parameters and can show a loading state | Route resources, after checking their release status |
| Data must be authorized | Server/API authorization, whether fetched by a resolver or component |

A resolver can improve the initial page state, but it also blocks navigation. Choose based on what the page needs before its first meaningful render.

## Related material

- [Route guards and access control boundaries](078-route-guards-and-access-control-boundaries.md)
- [Lazy-load routes and standalone components](080-lazy-load-routes-and-standalone-components.md)
- [Official Angular data resolvers guide](https://angular.dev/guide/routing/data-resolvers)
- [ResolveFn API](https://angular.dev/api/router/ResolveFn)
- [withComponentInputBinding API](https://angular.dev/api/router/withComponentInputBinding)
- [withNavigationErrorHandler API](https://angular.dev/api/router/withNavigationErrorHandler)
- [Angular route resources guide](https://angular.dev/guide/routing/data-fetching-with-resources)
