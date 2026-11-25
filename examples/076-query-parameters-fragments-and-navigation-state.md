# Query parameters, fragments, and navigation state: example

This standalone example builds a small product catalog whose category, sorting, and page selection live in the URL. It also demonstrates a fragment link to the results section and optional navigation state when opening checkout.

A sample URL looks like:

~~~text
/products?category=lighting&sort=price&page=1#results
~~~

Here, the route is `/products`, the query string controls catalog view state, and the fragment points to the results section.

## File layout

~~~text
src/
├── app/
│   ├── pages/
│   │   ├── checkout-page.ts
│   │   └── product-catalog-page.ts
│   ├── app.component.ts
│   ├── app.config.ts
│   └── app.routes.ts
└── main.ts
~~~

## 1. Configure routes and anchor scrolling

Create `src/app/app.routes.ts`:

~~~ts
import {Routes} from '@angular/router';

import {CheckoutPage} from './pages/checkout-page';
import {ProductCatalogPage} from './pages/product-catalog-page';

export const routes: Routes = [
  {
    path: 'products',
    component: ProductCatalogPage,
    title: 'Products',
  },
  {
    path: 'checkout',
    component: CheckoutPage,
    title: 'Checkout',
  },
  {
    path: '',
    pathMatch: 'full',
    redirectTo: 'products',
  },
  {
    path: '**',
    redirectTo: 'products',
  },
];
~~~

Configure router scrolling in `src/app/app.config.ts`:

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

Angular's in-memory anchor scrolling is disabled by default. Enabling it tells the router to scroll to an element whose `id` matches the URL fragment.

The CLI-generated `src/main.ts` should bootstrap the root component with this configuration:

~~~ts
import {bootstrapApplication} from '@angular/platform-browser';

import {AppComponent} from './app/app.component';
import {appConfig} from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch((error: unknown) => console.error(error));
~~~

## 2. Add the root outlet

Create `src/app/app.component.ts`:

~~~ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  template: '<router-outlet />',
})
export class AppComponent {}
~~~

## 3. Build a URL-driven catalog

Create `src/app/pages/product-catalog-page.ts`:

~~~ts
import {Component, computed, inject} from '@angular/core';
import {toSignal} from '@angular/core/rxjs-interop';
import {
  ActivatedRoute,
  Router,
  RouterLink,
} from '@angular/router';

interface Product {
  id: string;
  name: string;
  category: 'lighting' | 'kitchen';
  price: number;
}

@Component({
  selector: 'app-product-catalog-page',
  standalone: true,
  imports: [RouterLink],
  template: `
    <div id="top">
      <h1>Product catalog</h1>
      <p>Filters and sorting are stored in the URL.</p>
    </div>

    <nav aria-label="Catalog filters">
      <a
        routerLink="/products"
        [queryParams]="{category: 'all', sort: sort(), page: 1}"
        queryParamsHandling="merge"
        fragment="results"
      >
        All products
      </a>

      <a
        routerLink="/products"
        [queryParams]="{category: 'lighting', sort: sort(), page: 1}"
        queryParamsHandling="merge"
        fragment="results"
      >
        Lighting
      </a>

      <a
        routerLink="/products"
        [queryParams]="{category: 'kitchen', sort: sort(), page: 1}"
        queryParamsHandling="merge"
        fragment="results"
      >
        Kitchen
      </a>
    </nav>

    <label>
      Sort by
      <select [value]="sort()" (change)="setSort($event)">
        <option value="name">Name</option>
        <option value="price">Price</option>
      </select>
    </label>

    <p>
      Current URL state:
      category={{ category() }},
      sort={{ sort() }},
      page={{ currentPage() }},
      fragment={{ fragment() ?? 'none' }}
    </p>

    <a
      routerLink="/checkout"
      [state]="{source: 'catalog'}"
    >
      Continue to checkout
    </a>

    <section id="results" aria-labelledby="results-heading">
      <h2 id="results-heading">Results</h2>

      @if (visibleProducts().length > 0) {
        <ul>
          @for (product of visibleProducts(); track product.id) {
            <li>
              {{ product.name }} - {{ product.category }} - \${{ product.price }}
            </li>
          }
        </ul>
      } @else {
        <p>No products match this category.</p>
      }

      <p>Page {{ currentPage() }} of {{ totalPages() }}</p>

      <nav aria-label="Product pages">
        @if (currentPage() > 1) {
          <a
            [routerLink]="['/products']"
            [queryParams]="{
              category: category(),
              sort: sort(),
              page: currentPage() - 1
            }"
            queryParamsHandling="merge"
            fragment="results"
          >
            Previous page
          </a>
        }

        @if (currentPage() < totalPages()) {
          <a
            [routerLink]="['/products']"
            [queryParams]="{
              category: category(),
              sort: sort(),
              page: currentPage() + 1
            }"
            queryParamsHandling="merge"
            fragment="results"
          >
            Next page
          </a>
        }
      </nav>
    </section>
  `,
})
export class ProductCatalogPage {
  private readonly route = inject(ActivatedRoute);
  private readonly router = inject(Router);

  private readonly query = toSignal(this.route.queryParamMap, {
    initialValue: null,
  });

  readonly fragment = toSignal(this.route.fragment, {
    initialValue: null,
  });

  readonly category = computed(() => {
    const value = this.query()?.get('category');
    return value === 'lighting' || value === 'kitchen' ? value : 'all';
  });

  readonly sort = computed(() => {
    const value = this.query()?.get('sort');
    return value === 'price' ? 'price' : 'name';
  });

  readonly page = computed(() => {
    const rawPage = this.query()?.get('page');
    const parsedPage = Number(rawPage);
    return Number.isInteger(parsedPage) && parsedPage > 0 ? parsedPage : 1;
  });

  private readonly products: Product[] = [
    {id: 'lamp', name: 'Desk lamp', category: 'lighting', price: 45},
    {id: 'pendant', name: 'Pendant light', category: 'lighting', price: 80},
    {id: 'mug', name: 'Travel mug', category: 'kitchen', price: 18},
    {id: 'kettle', name: 'Electric kettle', category: 'kitchen', price: 55},
    {id: 'sconce', name: 'Wall sconce', category: 'lighting', price: 62},
  ];

  readonly sortedProducts = computed(() => {
    const category = this.category();
    const sort = this.sort();

    const matching = this.products.filter(
      (product) => category === 'all' || product.category === category,
    );

    return [...matching].sort((left, right) => {
      return sort === 'price'
        ? left.price - right.price
        : left.name.localeCompare(right.name);
    });
  });

  readonly totalPages = computed(() => {
    return Math.max(1, Math.ceil(this.sortedProducts().length / 2));
  });

  readonly currentPage = computed(() => {
    return Math.min(this.page(), this.totalPages());
  });

  readonly visibleProducts = computed(() => {
    const start = (this.currentPage() - 1) * 2;
    return this.sortedProducts().slice(start, start + 2);
  });

  setSort(event: Event): void {
    const target = event.target;
    if (!(target instanceof HTMLSelectElement)) {
      return;
    }

    const value = target.value;
    if (value !== 'name' && value !== 'price') {
      return;
    }

    void this.router.navigate([], {
      relativeTo: this.route,
      queryParams: {
        sort: value,
        page: 1,
      },
      queryParamsHandling: 'merge',
      fragment: 'results',
    });
  }
}
~~~

The component reads `queryParamMap` and `fragment` as reactive signals. It validates the category, sort key, and page instead of assuming URL values are valid. Changing the sort preserves other query keys and resets the page to 1. Category and pagination links use `RouterLink` so their state can be shared and restored from the URL.

The `fragment` is also read from the activated route. The filters and page links point to `#results`, and the matching element is present as `id="results"`.

## 4. Read optional navigation state at checkout

Create `src/app/pages/checkout-page.ts`:

~~~ts
import {Component, inject} from '@angular/core';
import {Router, RouterLink} from '@angular/router';

@Component({
  selector: 'app-checkout-page',
  standalone: true,
  imports: [RouterLink],
  template: `
    <h1>Checkout</h1>

    @if (source) {
      <p>Opened from: {{ source }}</p>
    } @else {
      <p>No navigation source was supplied.</p>
    }

    <a routerLink="/products">Return to products</a>
  `,
})
export class CheckoutPage {
  private readonly router = inject(Router);

  readonly source =
    this.router.currentNavigation()?.extras.state?.['source'] ?? null;
}
~~~

The catalog link passes `{source: 'catalog'}` as navigation `state`. The checkout page reads it while navigation is in progress. If someone opens `/checkout` directly, the navigation may have no such value; that is why the page provides a fallback and does not depend on the optional state to function.

Use query parameters instead if the context must be in a shareable URL. Navigation state is supplementary browser-history state, not secure storage.

## 5. Try the example

Start the development server:

~~~sh
ng serve
~~~

Then verify the behavior:

1. Open `/products?category=lighting&sort=price&page=1#results` directly. The app should show lighting products sorted by price and scroll to the results section.
2. Select a category. The category should change in the URL while the route remains `/products`.
3. Change the sort order. Other query keys should remain and the page should reset to 1.
4. Use Next page. The `page` query value should change, and the visible products should update.
5. Copy the URL into a new tab. The same category, sort, page, and fragment should be restored.
6. Select Continue to checkout. The checkout page should show `catalog` as its source.
7. Open `/checkout` directly. The page should still render and show its no-source fallback.
8. Use browser Back and Forward. Confirm the URL-driven catalog state follows the selected history entry.

## Key points

- Query parameters are useful for optional state that should survive sharing and refresh.
- Query values are strings and should be validated before use.
- Query parameters replace by default; choose `merge` or `preserve` when needed.
- Fragments use `#` and can identify an element by its `id`.
- Angular anchor scrolling is opt-in through `withInMemoryScrolling`.
- Navigation `state` is stored with a browser history entry but is absent from the URL.
- Optional navigation state must not be required for a direct page load.

## Official references

- [Read route state](https://angular.dev/guide/routing/read-route-state)
- [Navigate to routes](https://angular.dev/guide/routing/navigate-to-routes)
- [RouterLink API](https://angular.dev/api/router/RouterLink)
- [NavigationExtras API](https://angular.dev/api/router/NavigationExtras)
- [QueryParamsHandling API](https://angular.dev/api/router/QueryParamsHandling)
- [InMemoryScrollingOptions API](https://angular.dev/api/router/InMemoryScrollingOptions)
