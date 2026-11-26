# Services and the injectable pattern — example

This standalone Angular example demonstrates service responsibilities and dependency injection with a small product catalog. A root-provided catalog service coordinates a data source and a logger. The summary and list are sibling components: both inject the same catalog service, and the product count updates when the list adds a product.

The primary version uses `@Service()`, available in current Angular v22. The compatibility section shows how to adapt the services for projects on earlier Angular releases.

## What this example demonstrates

- A focused data service, a feature-state service, and a logging service.
- Service-to-service dependency injection using `inject()`.
- A root-provided instance shared by sibling components.
- A button interaction that updates state through the shared service.
- A component-scoped provider variant for intentionally isolated state.

## File layout

~~~text
src/
└── app/
    ├── app.component.ts
    ├── app.component.html
    ├── product.model.ts
    ├── product-api.service.ts
    ├── audit-logger.service.ts
    ├── product-catalog.service.ts
    ├── product-summary.component.ts
    ├── product-summary.component.html
    ├── product-list.component.ts
    └── product-list.component.html
~~~

## 1. Define the product type

~~~ts
// src/app/product.model.ts
export interface Product {
  readonly id: number;
  readonly name: string;
  readonly price: number;
}
~~~

Keeping the data shape in one place helps the API service, catalog, and components agree about what a product contains.

## 2. Add a data source service

~~~ts
// src/app/product-api.service.ts
import { Service } from '@angular/core';
import type { Product } from './product.model';

@Service()
export class ProductApiService {
  private readonly initialProducts: readonly Product[] = [
    { id: 1, name: 'Notebook', price: 4.5 },
    { id: 2, name: 'Pen', price: 1.25 },
    { id: 3, name: 'Desk lamp', price: 18.75 },
  ];

  getProducts(): Product[] {
    // Return copies so callers cannot mutate the fixture objects.
    return this.initialProducts.map((product) => ({ ...product }));
  }
}
~~~

This is an in-memory fixture, so the example works without a server. In a real application, this class is a natural boundary for HTTP requests and response mapping. Returning copies also keeps the sample's source data from being accidentally changed by a consumer.

## 3. Add a logging service

~~~ts
// src/app/audit-logger.service.ts
import { Service } from '@angular/core';

@Service()
export class AuditLoggerService {
  info(message: string): void {
    console.info('[catalog]', message);
  }
}
~~~

Logging has its own small responsibility. The catalog can use it without directly implementing console output or requiring its consumers to know about logging.

## 4. Let a catalog service compose the dependencies

~~~ts
// src/app/product-catalog.service.ts
import { computed, inject, Service, signal } from '@angular/core';
import { AuditLoggerService } from './audit-logger.service';
import { ProductApiService } from './product-api.service';
import type { Product } from './product.model';

@Service()
export class ProductCatalogService {
  private readonly api = inject(ProductApiService);
  private readonly logger = inject(AuditLoggerService);

  private readonly productState = signal<readonly Product[]>([]);
  readonly products = this.productState.asReadonly();

  readonly productCount = computed(() => this.products().length);

  readonly totalValue = computed(() =>
    this.products().reduce((sum, product) => sum + product.price, 0),
  );

  load(): void {
    const products = this.api.getProducts();
    this.productState.set(products);
    this.logger.info('Loaded ' + products.length + ' products.');
  }

  addSampleProduct(): void {
    const nextId = this.products().reduce(
      (highestId, product) => Math.max(highestId, product.id),
      0,
    ) + 1;

    const product: Product = {
      id: nextId,
      name: 'New product ' + nextId,
      price: 9.99,
    };

    this.productState.update((products) => [...products, product]);
    this.logger.info('Added ' + product.name + '.');
  }
}
~~~

The catalog receives its API and logger through Angular DI. It owns the feature's product state and exposes a readonly signal plus computed values. Components can read the catalog but cannot call `set()` or `update()` on its public product signal; state changes go through the catalog's methods.

`@Service()` provides each of these classes from the root injector by default. The catalog and its two dependencies are created as needed and reused by consumers resolving them from that same root injector.

## 5. Build a summary component

~~~ts
// src/app/product-summary.component.ts
import { CurrencyPipe } from '@angular/common';
import { Component, inject } from '@angular/core';
import { ProductCatalogService } from './product-catalog.service';

@Component({
  selector: 'app-product-summary',
  standalone: true,
  imports: [CurrencyPipe],
  templateUrl: './product-summary.component.html',
})
export class ProductSummaryComponent {
  readonly catalog = inject(ProductCatalogService);
}
~~~

~~~html
<!-- src/app/product-summary.component.html -->
<section aria-labelledby="catalog-summary-title">
  <h2 id="catalog-summary-title">Catalog summary</h2>
  <p>Products: {{ catalog.productCount() }}</p>
  <p>Total value: {{ catalog.totalValue() | currency }}</p>
</section>
~~~

## 6. Build a list component that changes shared state

~~~ts
// src/app/product-list.component.ts
import { CurrencyPipe } from '@angular/common';
import { Component, inject } from '@angular/core';
import { ProductCatalogService } from './product-catalog.service';

@Component({
  selector: 'app-product-list',
  standalone: true,
  imports: [CurrencyPipe],
  templateUrl: './product-list.component.html',
})
export class ProductListComponent {
  readonly catalog = inject(ProductCatalogService);
}
~~~

~~~html
<!-- src/app/product-list.component.html -->
<section aria-labelledby="catalog-products-title">
  <h2 id="catalog-products-title">Products</h2>

  <button type="button" (click)="catalog.addSampleProduct()">
    Add sample product
  </button>

  @if (catalog.products().length === 0) {
    <p>No products have been loaded.</p>
  } @else {
    <ul>
      @for (product of catalog.products(); track product.id) {
        <li>
          {{ product.name }} — {{ product.price | currency }}
        </li>
      }
    </ul>
  }
</section>
~~~

## 7. Compose both consumers in the root component

~~~ts
// src/app/app.component.ts
import { Component, inject } from '@angular/core';
import { ProductCatalogService } from './product-catalog.service';
import { ProductListComponent } from './product-list.component';
import { ProductSummaryComponent } from './product-summary.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ProductSummaryComponent, ProductListComponent],
  templateUrl: './app.component.html',
})
export class AppComponent {
  private readonly catalog = inject(ProductCatalogService);

  constructor() {
    this.catalog.load();
  }
}
~~~

~~~html
<!-- src/app/app.component.html -->
<main>
  <h1>Product catalog</h1>
  <app-product-summary />
  <app-product-list />
</main>
~~~

The root component loads the initial fixture once when it is constructed. Both child components resolve the same root-provided catalog instance. When the list adds a product, the catalog's signal changes and the summary recomputes its product count and total value.

## 8. Confirm the app bootstrap

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

Run the project with its existing Angular CLI development script, usually `ng serve`. The app should initially show three products. Select **Add sample product** and check that the list grows, the summary count increases, and the total changes. The browser console should show the catalog log messages.

## 9. Try a component-scoped instance

The current example intentionally shares catalog state from the root injector. To see how a local provider changes that behavior, add `providers: [ProductCatalogService]` to a component that owns an isolated catalog subtree. The component and its descendants will resolve that local instance instead of the root one.

If you use `@Service({ autoProvided: false })` on a class, remember that automatic root provision is disabled; you must register it in a `providers` array at the desired scope. Use this only when separate state is intentional. Providing the service at the wrong boundary can make sibling views disagree about their data.

## 10. Adapt the services for Angular versions before v22

For a project that does not have `@Service()`, replace the decorator and import in each service file with `@Injectable({ providedIn: 'root' })`. The service code uses `inject()`, which is supported with `@Injectable()` in modern Angular versions before v22 as well.

For example, change the top of `product-catalog.service.ts` to:

~~~ts
import { computed, inject, Injectable, signal } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class ProductCatalogService {
  // Keep the rest of the class unchanged.
}
~~~

Make the equivalent decorator/import change in `product-api.service.ts` and `audit-logger.service.ts`. Do not use `@Service()` in a release where that symbol is unavailable.

## 11. Make the data source use HTTP

The included API service is deliberately local so the example runs without a backend. In a real app, `ProductApiService` can inject Angular's `HttpClient` and return typed HTTP results. Configure the HTTP client once at the application boundary with `provideHttpClient()` in the application providers, then let the catalog decide how API data updates the feature state.

Keep the HTTP request implementation in the API-facing service. Keep catalog-specific state and operations in `ProductCatalogService`. This separation makes it easier to replace the local fixture, provide a fake API in a test, or reuse the feature behavior without placing transport details in a template component.

## What to observe

- Both sibling components inject the same `ProductCatalogService` because it is provided at root.
- The list's click handler changes state by calling a catalog method.
- The summary responds because its values are computed from the catalog's signal.
- `ProductApiService` and `AuditLoggerService` are injected into the catalog, showing service-to-service DI.
- Adding a component-level provider intentionally shadows a parent provider and creates a scoped instance.
- A service class is not inherently global; provider placement establishes visibility and instance scope.

## Troubleshooting

### Angular reports NG0201 for a service

Check that the class has a root provider or a provider at the correct scope. If using `@Injectable()` with no `providedIn`, register it explicitly. If using `@Service({ autoProvided: false })`, add it to a `providers` array.

### `inject()` reports NG0203

Call `inject()` in a field initializer or another valid injection context, not in an event handler or ordinary method. This example injects each dependency once into a field.

### The summary does not update

Check that the list calls `catalog.addSampleProduct()` and that both components import and inject the same `ProductCatalogService` token. Look for a component or route provider that may be creating a second catalog instance.

### A pipe is unknown in a standalone template

Standalone components must import the pipe used in their own template. This example imports `CurrencyPipe` in both components that display currency values.

## Continue learning

- Lesson: [Services and the injectable pattern](../lessons/049-services-and-the-injectable-pattern.md)
- Q&A: [Services and the injectable pattern](../qnas/049-services-and-the-injectable-pattern.md)
- Next lesson: [Inject dependencies with inject and constructor injection](../lessons/050-inject-dependencies-with-inject-and-constructor-injection.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="048-inspect-signals-and-components-with-angular-devtools.md">048 - Inspect signals and components with Angular DevTools</a></td><td><a href="../lessons/049-services-and-the-injectable-pattern.md">Lesson</a></td><td><a href="../qnas/049-services-and-the-injectable-pattern.md">Q&amp;A</a></td><td><a href="050-inject-dependencies-with-inject-and-constructor-injection.md">050 - Inject dependencies with `inject()` and constructor injection</a></td>
</tr>
  </tbody>
</table>
