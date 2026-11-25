# 82. Route-level providers and route configuration — Examples

This example uses the same catalog pages and API service under two route branches:

- /catalog uses the main catalog API.
- /catalog-preview uses the preview catalog API.

Each route provides its own API root, label, and CatalogApi instance. The route shell and child pages can inject those dependencies; unrelated routes cannot.

The example assumes an API is available at the configured paths and returns JSON matching the Book interface below. Replace those paths or adapt the service to your backend.

## Project files

~~~text
src/
├── main.ts
└── app/
    ├── app.config.ts
    ├── app.component.ts
    ├── app.component.html
    ├── app.routes.ts
    ├── catalog.tokens.ts
    ├── catalog-api.service.ts
    ├── catalog-shell.component.ts
    ├── catalog-shell.component.html
    ├── catalog-books.component.ts
    ├── catalog-books.component.html
    ├── catalog-book.component.ts
    ├── catalog-book.component.html
    └── not-found.component.ts
~~~

## 1. Define configuration tokens and the model

### src/app/catalog.tokens.ts

~~~ts
import { InjectionToken } from '@angular/core';

export const CATALOG_API_ROOT =
  new InjectionToken<string>('CATALOG_API_ROOT');

export const CATALOG_LABEL =
  new InjectionToken<string>('CATALOG_LABEL');

export interface Book {
  id: string;
  title: string;
  author: string;
}
~~~

The tokens have no global default. The routes that use the catalog feature supply both values.

## 2. Create a route-scoped API service

### src/app/catalog-api.service.ts

~~~ts
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import type { Observable } from 'rxjs';
import { CATALOG_API_ROOT, type Book } from './catalog.tokens';

@Injectable()
export class CatalogApi {
  private readonly http = inject(HttpClient);
  private readonly apiRoot = inject(CATALOG_API_ROOT);

  listBooks(): Observable<Book[]> {
    return this.http.get<Book[]>(this.apiRoot + '/books');
  }

  getBook(id: string): Observable<Book> {
    const encodedId = encodeURIComponent(id);
    return this.http.get<Book>(this.apiRoot + '/books/' + encodedId);
  }
}
~~~

This service intentionally does not use providedIn: 'root'. It reads a token whose value changes by route, so each route provides the service and token in the same injector scope.

## 3. Configure application providers

### src/app/app.config.ts

HttpClient and the router are configured application-wide. The catalog API root and label are configured separately on each catalog route.

~~~ts
import { provideHttpClient } from '@angular/common/http';
import type { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    provideRouter(routes),
  ],
};
~~~

### src/main.ts

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch((error: unknown) => console.error(error));
~~~

## 4. Create the route table

### src/app/app.routes.ts

The helper function creates two route definitions with the same child routes and components. Each parent route has its own providers and configuration values.

~~~ts
import type { Route, Routes } from '@angular/router';
import { CatalogApi } from './catalog-api.service';
import { CatalogBookComponent } from './catalog-book.component';
import { CatalogBooksComponent } from './catalog-books.component';
import { CatalogShellComponent } from './catalog-shell.component';
import { CATALOG_API_ROOT, CATALOG_LABEL } from './catalog.tokens';
import { NotFoundComponent } from './not-found.component';

function catalogRoute(
  path: string,
  label: string,
  apiRoot: string,
): Route {
  return {
    path,
    component: CatalogShellComponent,
    providers: [
      CatalogApi,
      { provide: CATALOG_LABEL, useValue: label },
      { provide: CATALOG_API_ROOT, useValue: apiRoot },
    ],
    children: [
      { path: '', pathMatch: 'full', redirectTo: 'books' },
      { path: 'books', component: CatalogBooksComponent },
      { path: 'book/:id', component: CatalogBookComponent },
    ],
  };
}

export const routes: Routes = [
  catalogRoute('catalog', 'Main catalog', '/api/catalog'),
  catalogRoute(
    'catalog-preview',
    'Catalog preview',
    '/api/catalog-preview',
  ),
  { path: '', pathMatch: 'full', redirectTo: 'catalog' },
  { path: '**', component: NotFoundComponent },
];
~~~

The child routes use the parent route's injector. Both the shell and child pages can resolve the same route-scoped CatalogApi instance and configuration tokens.

## 5. Create the shell that reads route-scoped configuration

### src/app/catalog-shell.component.ts

~~~ts
import { Component, inject } from '@angular/core';
import { RouterLink, RouterOutlet } from '@angular/router';
import { CATALOG_LABEL } from './catalog.tokens';

@Component({
  standalone: true,
  selector: 'app-catalog-shell',
  imports: [RouterLink, RouterOutlet],
  templateUrl: './catalog-shell.component.html',
})
export class CatalogShellComponent {
  readonly label = inject(CATALOG_LABEL);
}
~~~

### src/app/catalog-shell.component.html

~~~html
<header>
  <h1>{{ label }}</h1>
  <nav aria-label="Catalog pages">
    <a routerLink="books">Browse books</a>
    <a routerLink="book/42">Featured book</a>
  </nav>
</header>

<main>
  <router-outlet></router-outlet>
</main>
~~~

The links are relative to the catalog shell route. The same shell works under either catalog URL because it receives its label from the active route injector.

## 6. Consume the route-scoped API from a child page

### src/app/catalog-books.component.ts

~~~ts
import { AsyncPipe } from '@angular/common';
import { Component, inject } from '@angular/core';
import { RouterLink } from '@angular/router';
import { CatalogApi } from './catalog-api.service';

@Component({
  standalone: true,
  selector: 'app-catalog-books',
  imports: [AsyncPipe, RouterLink],
  templateUrl: './catalog-books.component.html',
})
export class CatalogBooksComponent {
  readonly books$ = inject(CatalogApi).listBooks();
}
~~~

### src/app/catalog-books.component.html

~~~html
<section aria-labelledby="books-heading">
  <h2 id="books-heading">Books</h2>

  @if (books$ | async; as books) {
    <ul>
      @for (book of books; track book.id) {
        <li>
          <a [routerLink]="['../book', book.id]">
            {{ book.title }} — {{ book.author }}
          </a>
        </li>
      } @empty {
        <li>No books are available.</li>
      }
    </ul>
  } @else {
    <p>Loading books…</p>
  }
</section>
~~~

The list page does not need to know which API root is active. It injects CatalogApi, and the route-scoped service already has the correct configuration.

## 7. Read a route parameter and load one book

### src/app/catalog-book.component.ts

~~~ts
import { AsyncPipe } from '@angular/common';
import { Component, inject } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { filter, map, switchMap } from 'rxjs';
import { CatalogApi } from './catalog-api.service';

@Component({
  standalone: true,
  selector: 'app-catalog-book',
  imports: [AsyncPipe],
  templateUrl: './catalog-book.component.html',
})
export class CatalogBookComponent {
  private readonly route = inject(ActivatedRoute);
  private readonly catalogApi = inject(CatalogApi);

  readonly book$ = this.route.paramMap.pipe(
    map((params) => params.get('id')),
    filter((id): id is string => id !== null),
    switchMap((id) => this.catalogApi.getBook(id)),
  );
}
~~~

### src/app/catalog-book.component.html

~~~html
@if (book$ | async; as book) {
  <article>
    <h2>{{ book.title }}</h2>
    <p>By {{ book.author }}</p>
    <p>Book ID: {{ book.id }}</p>
  </article>
} @else {
  <p>Loading book…</p>
}
~~~

The component gets the id from its own route and the API client from the parent route's provider scope. When route parameters change, the stream requests the book for the current id.

## 8. Add root navigation

### src/app/app.component.ts

~~~ts
import { Component } from '@angular/core';
import { RouterLink, RouterOutlet } from '@angular/router';

@Component({
  standalone: true,
  selector: 'app-root',
  imports: [RouterLink, RouterOutlet],
  templateUrl: './app.component.html',
})
export class AppComponent {}
~~~

### src/app/app.component.html

~~~html
<nav aria-label="Catalog configuration">
  <a routerLink="/catalog/books">Main catalog</a>
  <a routerLink="/catalog-preview/books">Preview catalog</a>
</nav>

<router-outlet></router-outlet>
~~~

The root navigation is outside both route branches. It can link to them, but it does not inherit either branch's CatalogApi or configuration tokens.

## 9. Add a not-found page

### src/app/not-found.component.ts

~~~ts
import { Component } from '@angular/core';
import { RouterLink } from '@angular/router';

@Component({
  standalone: true,
  selector: 'app-not-found',
  imports: [RouterLink],
  template: '<main><h1>Page not found</h1><a routerLink="/catalog">Open the main catalog</a></main>',
})
export class NotFoundComponent {}
~~~

The wildcard route is last in the route table so it does not take precedence over the catalog branches.

## 10. Try both route scopes

Use a backend that responds with Book JSON at these endpoints, or adjust the API root values:

~~~text
GET /api/catalog/books
GET /api/catalog/books/:id

GET /api/catalog-preview/books
GET /api/catalog-preview/books/:id
~~~

Then navigate manually:

1. Open /catalog/books. The shell label is “Main catalog”; CatalogApi requests /api/catalog/books.
2. Open a book. The URL becomes /catalog/book/:id, and the detail page uses the same route-scoped API client.
3. Open /catalog-preview/books. The same components run under the preview route, with a different label and API root.
4. Open an unknown URL. The wildcard route renders the not-found page.
5. Inspect the route tree in Angular DevTools to see the providers on each catalog route.

These are manual navigation steps. This Markdown example does not include a backend or automated test suite.

## Scope map

~~~text
App EnvironmentInjector
├── Catalog route injector
│   ├── CATALOG_API_ROOT = /api/catalog
│   ├── CATALOG_LABEL = Main catalog
│   ├── CatalogApi
│   ├── CatalogShellComponent
│   └── catalog child pages
└── Catalog preview route injector
    ├── CATALOG_API_ROOT = /api/catalog-preview
    ├── CATALOG_LABEL = Catalog preview
    ├── CatalogApi
    ├── CatalogShellComponent
    └── catalog child pages
~~~

Each route branch uses the same classes but resolves the tokens and service from its own route injector. The route-specific values are not application-wide defaults.

## Common adjustments

- **Use a static mock instead of HTTP:** replace CatalogApi methods with observables of local Book data; keep the route providers to demonstrate isolated configuration.
- **Share a root logger:** keep LoggerService provided in root and inject it into CatalogApi; leave only the API root route-scoped.
- **Add a route-specific guard:** place the guard on a catalog route and inject an access service provided on that route or an ancestor.
- **Add a shared parent for more paths:** move providers to the nearest parent route shared by all consumers.
- **Reset feature state on explicit user action:** add a reset method or create a new feature workflow; do not depend on leaving and revisiting the URL to recreate the route injector.

## What this example demonstrates

- A route providers array creates an injection scope for that route branch.
- Descendant components can inject services registered by their parent route.
- Sibling route branches can configure the same service class differently.
- A service that consumes route-specific configuration must be instantiated in the route scope.
- Root navigation can enter a route subtree but does not inherit dependencies from it.
