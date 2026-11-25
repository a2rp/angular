# Route parameters, snapshots, and reactive parameters: example

This standalone example displays product details based on a path parameter. It demonstrates a route declaration, links that build parameterized URLs, a reactive detail component, and a snapshot-based alternative for a one-time read.

## File layout

~~~text
src/
├── app/
│   ├── pages/
│   │   ├── product-detail-page.ts
│   │   └── product-list-page.ts
│   ├── app.component.ts
│   ├── app.config.ts
│   └── app.routes.ts
└── main.ts
~~~

## 1. Define the parameterized route

Create `src/app/app.routes.ts`:

~~~ts
import {Routes} from '@angular/router';

import {ProductDetailPage} from './pages/product-detail-page';
import {ProductListPage} from './pages/product-list-page';

export const routes: Routes = [
  {
    path: 'products',
    component: ProductListPage,
    title: 'Products',
  },
  {
    path: 'products/:productId',
    component: ProductDetailPage,
    title: 'Product details',
  },
  {
    path: '**',
    redirectTo: 'products',
  },
];
~~~

The colon in `:productId` declares a named path parameter. A URL such as `/products/42` activates `ProductDetailPage` with `productId` equal to the string `42`.

## 2. Register routes and bootstrap the app

In `src/app/app.config.ts`:

~~~ts
import {ApplicationConfig} from '@angular/core';
import {provideRouter} from '@angular/router';

import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)],
};
~~~

The CLI-generated `src/main.ts` should bootstrap the root component with this configuration:

~~~ts
import {bootstrapApplication} from '@angular/platform-browser';

import {AppComponent} from './app/app.component';
import {appConfig} from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch((error: unknown) => console.error(error));
~~~

## 3. Add the root outlet

Create `src/app/app.component.ts`:

~~~ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  template: `
    <main>
      <router-outlet />
    </main>
  `,
})
export class AppComponent {}
~~~

The router renders either the product list or the selected product detail component at the outlet.

## 4. Link to products

Create `src/app/pages/product-list-page.ts`:

~~~ts
import {Component} from '@angular/core';
import {RouterLink} from '@angular/router';

@Component({
  selector: 'app-product-list-page',
  standalone: true,
  imports: [RouterLink],
  template: `
    <h1>Products</h1>

    <ul>
      @for (product of products; track product.id) {
        <li>
          <a [routerLink]="['/products', product.id]">
            {{ product.name }}
          </a>
        </li>
      }
    </ul>
  `,
})
export class ProductListPage {
  readonly products = [
    {id: '42', name: 'Desk lamp'},
    {id: '99', name: 'Travel mug'},
  ];
}
~~~

The command array builds a URL from a fixed route prefix and the selected product's ID. The IDs remain strings because route parameter values are strings.

## 5. React to route parameter changes

Create `src/app/pages/product-detail-page.ts`:

~~~ts
import {Component, computed, inject} from '@angular/core';
import {toSignal} from '@angular/core/rxjs-interop';
import {ActivatedRoute, RouterLink} from '@angular/router';
import {map} from 'rxjs';

const productNames: Record<string, string> = {
  '42': 'Desk lamp',
  '99': 'Travel mug',
};

@Component({
  selector: 'app-product-detail-page',
  standalone: true,
  imports: [RouterLink],
  template: `
    <a routerLink="/products">Back to products</a>

    @if (productId(); as id) {
      <h1>Product details</h1>
      <p>Route ID: {{ id }}</p>

      @if (productName(); as name) {
        <p>Product: {{ name }}</p>
      } @else {
        <p>No product was found for ID {{ id }}.</p>
      }

      <a [routerLink]="['/products', '99']">Open product 99</a>
    } @else {
      <p>Waiting for the route parameter...</p>
    }
  `,
})
export class ProductDetailPage {
  private readonly route = inject(ActivatedRoute);

  readonly productId = toSignal(
    this.route.paramMap.pipe(
      map((paramMap) => paramMap.get('productId')),
    ),
    {initialValue: null},
  );

  readonly productName = computed(() => {
    const id = this.productId();
    return id === null ? null : productNames[id] ?? null;
  });
}
~~~

`ActivatedRoute.paramMap` emits a parameter map as the active route state changes. `toSignal` turns the selected value into a signal, so the template and computed product name update when the ID changes. The map lookup can return no result, so the template displays a not-found message for an unknown ID.

## 6. Compare with a snapshot read

If a component only needs the parameter once when it is created, it can read the snapshot:

~~~ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-one-time-product-detail',
  standalone: true,
  template: '<p>Product ID: {{ productId ?? "missing" }}</p>',
})
export class OneTimeProductDetailPage {
  private readonly route = inject(ActivatedRoute);

  readonly productId =
    this.route.snapshot.paramMap.get('productId');
}
~~~

This is a different implementation of the detail page, shown for comparison. Use either the reactive component or the snapshot version for a route. A field initialized from a snapshot will not itself change if the router reuses the component for another product ID; the reactive version handles that case.

## 7. Try the example

Start the development server:

~~~sh
ng serve
~~~

Then check:

1. Open `/products` and select Desk lamp. The URL should become `/products/42`.
2. Confirm the detail page shows the ID `42` and the name Desk lamp.
3. Select Open product 99. The URL and displayed ID/name should update to `99` and Travel mug.
4. Open `/products/unknown` directly. The route matches, but the component shows that no product exists for that ID.
5. Use Back and Forward to check browser history.
6. Open `/products/` without an ID. The parameterized route does not match; the fallback sends the visitor to `/products`.

## Key points

- `:productId` declares a path parameter named `productId`.
- `paramMap.get()` returns `string | null`; validate and handle the result.
- A snapshot is a one-time view of route state.
- The `paramMap` observable lets a component respond when parameters change.
- URL parameters are visible and untrusted; do not put secrets in them.
- A valid-looking ID can still refer to a missing record.

## Official references

- [Read route state](https://angular.dev/guide/routing/read-route-state)
- [ActivatedRoute API](https://angular.dev/api/router/ActivatedRoute)
- [ParamMap API](https://angular.dev/api/router/ParamMap)
