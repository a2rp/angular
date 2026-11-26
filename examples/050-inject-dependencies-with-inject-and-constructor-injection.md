# Inject dependencies with `inject()` and constructor injection — example

This standalone Angular example compares field injection with constructor injection in one small catalog feature. The product list uses `inject()`; the summary receives the catalog and an `InjectionToken` through its constructor. Both components read the same root-provided catalog service, so adding an item in the list updates the summary.

The service classes use `@Service()`, which is available in current Angular v22. The example also includes the configuration provider and app bootstrap so the token is actually resolvable.

## What this example demonstrates

- Injecting class tokens into a component with `inject()`.
- Injecting class tokens with constructor parameters.
- Injecting a typed `InjectionToken` with `inject()` and with `@Inject()`.
- Sharing one root-provided service instance across sibling components.
- Keeping `inject()` calls in valid injection contexts.
- Registering configuration at the application injector.

## File layout

~~~text
src/
├── main.ts
└── app/
    ├── app.config.ts
    ├── app.component.ts
    ├── app.component.html
    ├── api-config.token.ts
    ├── product.model.ts
    ├── audit-logger.service.ts
    ├── product-api.service.ts
    ├── product-catalog.service.ts
    ├── product-list.component.ts
    ├── product-list.component.html
    ├── product-summary.component.ts
    └── product-summary.component.html
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

## 2. Create a typed token for API configuration

An interface gives TypeScript a shape for the config, but Angular needs a runtime token. Define and export one `InjectionToken` object:

~~~ts
// src/app/api-config.token.ts
import { InjectionToken } from '@angular/core';

export interface ApiConfig {
  readonly baseUrl: string;
  readonly environmentName: string;
}

export const API_CONFIG = new InjectionToken<ApiConfig>('api.config');
~~~

The string is a debugging label. Consumers and providers must import the same `API_CONFIG` object; creating another token with the same label would create a different key.

## 3. Register the token in application configuration

~~~ts
// src/app/app.config.ts
import type { ApplicationConfig } from '@angular/core';
import { API_CONFIG } from './api-config.token';

export const appConfig: ApplicationConfig = {
  providers: [
    {
      provide: API_CONFIG,
      useValue: {
        baseUrl: '/api',
        environmentName: 'local demo',
      },
    },
  ],
};
~~~

This local configuration value does not make a network request. It gives the API service and summary component a typed value from Angular DI.

## 4. Create a logger and a local API service

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

~~~ts
// src/app/product-api.service.ts
import { inject, Service } from '@angular/core';
import { API_CONFIG } from './api-config.token';
import type { Product } from './product.model';

@Service()
export class ProductApiService {
  private readonly config = inject(API_CONFIG);
  readonly endpoint = this.config.baseUrl + '/products';

  private readonly initialProducts: readonly Product[] = [
    { id: 1, name: 'Notebook', price: 4.5 },
    { id: 2, name: 'Pen', price: 1.25 },
  ];

  getProducts(): Product[] {
    return this.initialProducts.map((product) => ({ ...product }));
  }
}
~~~

The API service demonstrates `inject()` inside a service and a token-backed configuration value. It returns an in-memory fixture so the example works without a backend. A real API service could use `HttpClient` while keeping its HTTP details out of the view components.

## 5. Create the shared catalog service

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
  readonly apiEndpoint = this.api.endpoint;
  readonly productCount = computed(() => this.products().length);
  readonly totalValue = computed(() =>
    this.products().reduce((total, product) => total + product.price, 0),
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

The catalog service composes the API and logger with `inject()`. It owns the feature state and exposes a readonly signal and computed values. Both components will resolve this same root-provided service instance because neither declares a local provider for it.

## 6. Use `inject()` in the product list component

~~~ts
// src/app/product-list.component.ts
import { Component, inject } from '@angular/core';
import { ProductCatalogService } from './product-catalog.service';

@Component({
  selector: 'app-product-list',
  standalone: true,
  templateUrl: './product-list.component.html',
})
export class ProductListComponent {
  readonly catalog = inject(ProductCatalogService);
}
~~~

~~~html
<!-- src/app/product-list.component.html -->
<section aria-labelledby="products-title">
  <h2 id="products-title">Products</h2>

  <button type="button" (click)="catalog.addSampleProduct()">
    Add sample product
  </button>

  @if (catalog.products().length === 0) {
    <p>No products are loaded.</p>
  } @else {
    <ul>
      @for (product of catalog.products(); track product.id) {
        <li>{{ product.name }} - {{ product.price }}</li>
      }
    </ul>
  }
</section>
~~~

The class injects once when Angular creates the component. Its click handler calls the stored catalog dependency. It does not call `inject()` at event time.

## 7. Use constructor injection and `@Inject()` in the summary component

~~~ts
// src/app/product-summary.component.ts
import { Component, Inject } from '@angular/core';
import { API_CONFIG } from './api-config.token';
import type { ApiConfig } from './api-config.token';
import { ProductCatalogService } from './product-catalog.service';

@Component({
  selector: 'app-product-summary',
  standalone: true,
  templateUrl: './product-summary.component.html',
})
export class ProductSummaryComponent {
  constructor(
    readonly catalog: ProductCatalogService,
    @Inject(API_CONFIG) readonly config: ApiConfig,
  ) {}
}
~~~

~~~html
<!-- src/app/product-summary.component.html -->
<section aria-labelledby="summary-title" aria-live="polite">
  <h2 id="summary-title">Catalog summary</h2>
  <p>Products: {{ catalog.productCount() }}</p>
  <p>Total value: {{ catalog.totalValue() }}</p>
  <p>API endpoint: {{ catalog.apiEndpoint }}</p>
  <p>Configured environment: {{ config.environmentName }}</p>
</section>
~~~

The constructor asks Angular for two tokens: the class token `ProductCatalogService` and the explicit token `API_CONFIG`. The catalog token is identified by its class; `@Inject(API_CONFIG)` identifies the non-class token. Both results are stored as parameter properties.

## 8. Compose the two consumers

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
  <h1>Dependency injection in a product catalog</h1>
  <app-product-summary></app-product-summary>
  <app-product-list></app-product-list>
</main>
~~~

The root component also uses field injection and loads the local products during construction. The list and summary are siblings, but both resolve the same root catalog. Clicking **Add sample product** should increase the list and update the summary count and total.

## 9. Bootstrap with the configured provider

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, appConfig).catch((error: unknown) => {
  console.error(error);
});
~~~

Start the Angular CLI development server with the project's existing script, usually `ng serve`. The page should display two initial products, the local API endpoint and configured environment. Click the button and confirm that both the product list and summary update; the console should show the catalog log entry.

## 10. Compare the two styles side by side

The list component uses a field initializer:

~~~ts
readonly catalog = inject(ProductCatalogService);
~~~

The summary component requests the same class dependency through its constructor:

~~~ts
constructor(readonly catalog: ProductCatalogService) {}
~~~

They have the same provider resolution and share the same instance. The syntax does not decide whether a service is global or local; provider and injector scope do. The summary additionally uses `@Inject(API_CONFIG)` because a TypeScript interface is not a runtime token.

## 11. Use the legacy decorator in Angular v17-v21 projects

For a project that supports `inject()` but predates the Angular v22 `@Service()` decorator, replace each service's decorator and import with `@Injectable({ providedIn: 'root' })`:

~~~ts
import { inject, Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class ProductApiService {
  private readonly config = inject(API_CONFIG);
  // Keep the remaining class members unchanged.
}
~~~

Make the equivalent change in `AuditLoggerService` and `ProductCatalogService`. Keep the `inject()` fields; they are supported with `@Injectable()` in modern Angular versions. For still older Angular releases, use the injection APIs and template syntax available in that release.

## Troubleshooting

### NG0201: No provider for `API_CONFIG`

Check that `appConfig` includes the `API_CONFIG` provider and that `main.ts` passes `appConfig` to `bootstrapApplication`. Confirm both files import the same token object from `api-config.token.ts`.

### NG0203: `inject()` called outside an injection context

Keep calls in a field initializer or in the constructor body while Angular is creating the instance. Inject once, then use the stored field in handlers and lifecycle hooks.

### The constructor token is not resolving

Confirm the constructor parameter has `@Inject(API_CONFIG)` and that a provider exists for that exact token. The TypeScript interface is useful for typing, but Angular cannot resolve it by itself.

### The summary does not change after clicking

Check that neither child component provides `ProductCatalogService` locally. A nearer provider would create a different instance and split the state. Also confirm the button calls `addSampleProduct()` on the injected catalog.

### `@Service()` cannot be imported

The example's service files target current Angular v22. In Angular v17-v21, replace `@Service()` with `@Injectable({ providedIn: 'root' })` and update the imports. Check the installed package version before using the current decorator.

## Continue learning

- Lesson: [Inject dependencies with inject and constructor injection](../lessons/050-inject-dependencies-with-inject-and-constructor-injection.md)
- Q&A: [Inject dependencies with inject and constructor injection](../qnas/050-inject-dependencies-with-inject-and-constructor-injection.md)
- Next lesson: [Hierarchical dependency injection](../lessons/051-hierarchical-dependency-injection.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="049-services-and-the-injectable-pattern.md">049 - Services and the injectable pattern</a></td><td><a href="../lessons/050-inject-dependencies-with-inject-and-constructor-injection.md">Lesson</a></td><td><a href="../qnas/050-inject-dependencies-with-inject-and-constructor-injection.md">Q&amp;A</a></td><td><a href="051-hierarchical-dependency-injection.md">051 - Hierarchical dependency injection</a></td>
</tr>
  </tbody>
</table>
