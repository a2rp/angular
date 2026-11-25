# Resolvers, route data, and loading a page before activation: example

This standalone example loads a product before activating its detail page. It demonstrates:

- A typed `ResolveFn<Product | RedirectCommand>` that reads a path parameter.
- Route configuration with a named resolver key.
- Resolved data bound to a required component input.
- A 404 redirect to a not-found route and a centralized handler for unexpected navigation errors.
- A top-level progress message while the router waits for the resolver.

The example expects a same-origin API endpoint, `GET /api/products/:productId`, that returns a product as JSON. A sample response is shown below. The server must still enforce access to its API data.

## File layout

~~~text
src/
├── main.ts
└── app/
    ├── app.component.ts
    ├── app.config.ts
    ├── app.routes.ts
    ├── pages/
    │   └── message.page.ts
    └── products/
        ├── product-detail.page.ts
        ├── product.model.ts
        ├── product.resolver.ts
        └── product-api.service.ts
~~~

Create a standalone Angular application with Angular Router and `HttpClient`, then add or adapt these files.

## 1. Define the resolved data type

Create `src/app/products/product.model.ts`:

~~~ts
export interface Product {
  id: string;
  name: string;
  description: string;
  price: number;
}
~~~

For example, `GET /api/products/p-1001` can return:

~~~json
{
  "id": "p-1001",
  "name": "Desk lamp",
  "description": "An adjustable lamp for a work desk.",
  "price": 39.5
}
~~~

## 2. Create the API service

Create `src/app/products/product-api.service.ts`:

~~~ts
import {HttpClient} from '@angular/common/http';
import {inject, Injectable} from '@angular/core';
import type {Observable} from 'rxjs';

import type {Product} from './product.model';

@Injectable({providedIn: 'root'})
export class ProductApiService {
  private readonly http = inject(HttpClient);

  getById(productId: string): Observable<Product> {
    const encodedId = encodeURIComponent(productId);
    return this.http.get<Product>('/api/products/' + encodedId);
  }
}
~~~

The backend should return an HTTP 404 when the product does not exist. It should return other appropriate status codes for authentication, authorization, or server failures.

## 3. Resolve the product before activation

Create `src/app/products/product.resolver.ts`:

~~~ts
import {HttpErrorResponse} from '@angular/common/http';
import {inject} from '@angular/core';
import {RedirectCommand, Router} from '@angular/router';
import type {ResolveFn} from '@angular/router';
import {catchError, of, throwError} from 'rxjs';

import {ProductApiService} from './product-api.service';
import type {Product} from './product.model';

export const productResolver: ResolveFn<Product | RedirectCommand> = route => {
  const api = inject(ProductApiService);
  const router = inject(Router);
  const productId = route.paramMap.get('productId');

  if (!productId) {
    return new RedirectCommand(router.parseUrl('/not-found'));
  }

  return api.getById(productId).pipe(
    catchError((error: unknown) => {
      if (error instanceof HttpErrorResponse && error.status === 404) {
        return of(new RedirectCommand(router.parseUrl('/not-found')));
      }

      return throwError(() => error);
    }),
  );
};
~~~

The resolver captures injected services synchronously, reads the target route parameter, and returns the HTTP observable. For a 404 it returns a redirect result. Other errors propagate to the router's navigation error handling rather than being mislabeled as missing data.

## 4. Display the resolved product

Create `src/app/products/product-detail.page.ts`:

~~~ts
import {CurrencyPipe} from '@angular/common';
import {Component, input} from '@angular/core';

import type {Product} from './product.model';

@Component({
  selector: 'app-product-detail',
  standalone: true,
  imports: [CurrencyPipe],
  template: '<article><h1>{{ product().name }}</h1><p>{{ product().description }}</p><p>Price: {{ product().price | currency }}</p></article>',
})
export class ProductDetailPage {
  readonly product = input.required<Product>();
}
~~~

The input name `product` must match the key assigned in the route's `resolve` object. The component only activates after that input has a resolved value.

## 5. Add a simple message page for alternate routes

Create `src/app/pages/message.page.ts`:

~~~ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-message-page',
  standalone: true,
  template: '<main><h1>{{ title }}</h1><p>{{ description }}</p></main>',
})
export class MessagePage {
  private readonly route = inject(ActivatedRoute);

  get title(): string {
    return this.route.snapshot.data['title'] ?? 'Message';
  }

  get description(): string {
    return this.route.snapshot.data['description'] ?? '';
  }
}
~~~

## 6. Configure routes and navigation error handling

Create `src/app/app.routes.ts`:

~~~ts
import type {Routes} from '@angular/router';

import {MessagePage} from './pages/message.page';
import {ProductDetailPage} from './products/product-detail.page';
import {productResolver} from './products/product.resolver';

export const routes: Routes = [
  {
    path: 'products/:productId',
    component: ProductDetailPage,
    resolve: {
      product: productResolver,
    },
    title: 'Product details',
  },
  {
    path: 'not-found',
    component: MessagePage,
    title: 'Product not found',
    data: {
      title: 'Product not found',
      description: 'No product exists for the requested ID.',
    },
  },
  {
    path: 'load-error',
    component: MessagePage,
    title: 'Unable to load this page',
    data: {
      title: 'Unable to load this page',
      description: 'The request failed. Please try again later.',
    },
  },
  {
    path: '',
    pathMatch: 'full',
    redirectTo: 'products/p-1001',
  },
  {
    path: '**',
    component: MessagePage,
    title: 'Page not found',
    data: {
      title: 'Page not found',
      description: 'No route matches this URL.',
    },
  },
];
~~~

Create `src/app/app.config.ts`. This enables resolved values to bind to component inputs and configures a central handler for unexpected navigation errors:

~~~ts
import {provideHttpClient} from '@angular/common/http';
import {inject} from '@angular/core';
import type {ApplicationConfig} from '@angular/core';
import {
  provideRouter,
  Router,
  withComponentInputBinding,
  withNavigationErrorHandler,
} from '@angular/router';

import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    provideRouter(
      routes,
      withComponentInputBinding(),
      withNavigationErrorHandler(error => {
        console.error('Navigation failed:', error);
        const router = inject(Router);
        void router.navigateByUrl('/load-error');
      }),
    ),
  ],
};
~~~

A not-found redirect is a deliberate resolver result and does not need to be treated as an unexpected navigation error. Other failures are logged and sent to the generic load-error page.

## 7. Show navigation feedback while data loads

Create `src/app/app.component.ts`:

~~~ts
import {Component, computed, inject} from '@angular/core';
import {Router, RouterLink, RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterLink, RouterOutlet],
  template: '<header><nav aria-label="Main navigation"><a routerLink="/products/p-1001">Desk lamp</a><a routerLink="/products/p-missing">Missing product example</a></nav></header>@if (isNavigating()) { <p role="status">Loading page data…</p> }<router-outlet />',
})
export class AppComponent {
  private readonly router = inject(Router);

  readonly isNavigating = computed(
    () => this.router.currentNavigation() !== null,
  );
}
~~~

Create `src/main.ts`:

~~~ts
import {bootstrapApplication} from '@angular/platform-browser';

import {AppComponent} from './app/app.component';
import {appConfig} from './app/app.config';

bootstrapApplication(AppComponent, appConfig).catch(error => {
  console.error(error);
});
~~~

The destination component stays inactive while the resolver is pending. The app-level message gives feedback during that navigation. For a slower or nonessential request, prefer activating the page and showing a loading state within it instead of blocking the route.

## 8. Try the flows

Run the Angular development server with the API available at the same origin:

1. Open `/products/p-1001`. The app requests the product, then displays its detail page after the resolver completes.
2. Open `/products/p-missing` and have the API respond with HTTP 404. The resolver redirects to `/not-found`.
3. Have the API return a server error for another product. The resolver lets the error propagate and the global handler navigates to `/load-error`.
4. Add a delay to the API response. Observe the app-level loading message while the router waits.
5. Confirm that the API denies unauthorized requests even if called without using the Angular route.

## Common mistakes

- Starting optional page requests in resolvers and making all navigation wait for them.
- Using a resolver key that does not match the component input or route-data lookup.
- Treating every API error as a missing record.
- Returning an empty observable instead of data, a redirect, or an error.
- Forgetting to configure `provideHttpClient()` for an `HttpClient` service.
- Assuming resolved data replaces server-side authorization or automatically stays fresh after edits.

## Related material

- [Lesson: Resolvers, route data, and loading a page before activation](../lessons/079-resolvers-route-data-and-loading-a-page-before-activation.md)
- [Q&A: Resolvers, route data, and loading a page before activation](../qnas/079-resolvers-route-data-and-loading-a-page-before-activation.md)
- [Route guards and access control boundaries](../lessons/078-route-guards-and-access-control-boundaries.md)
- [Angular data resolvers guide](https://angular.dev/guide/routing/data-resolvers)
- [ResolveFn API](https://angular.dev/api/router/ResolveFn)
- [withComponentInputBinding API](https://angular.dev/api/router/withComponentInputBinding)
- [withNavigationErrorHandler API](https://angular.dev/api/router/withNavigationErrorHandler)
