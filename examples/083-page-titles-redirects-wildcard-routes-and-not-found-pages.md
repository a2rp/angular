# 83. Page titles, redirects, wildcard routes, and not-found pages - Examples

This example configures route titles, a dynamic product title, old URL redirects, a conditional promo redirect, and a not-found page. It also adds a custom title strategy that appends the application name.

## Project files

~~~text
src/
├── main.ts
└── app/
    ├── app.config.ts
    ├── app.component.ts
    ├── app.routes.ts
    ├── app-title.strategy.ts
    └── pages.ts
~~~

## 1. Create page components

### src/app/pages.ts

The components are small so the routing behavior stays easy to see. In a larger app, place each component in its own file.

~~~ts
import { Component, inject } from '@angular/core';
import { ActivatedRoute, RouterLink } from '@angular/router';

@Component({
  standalone: true,
  template: '<h1>Home</h1><p>Welcome to the learning notes.</p>',
})
export class HomePage {}

@Component({
  standalone: true,
  template: '<h1>About</h1><p>About this learning project.</p>',
})
export class AboutPage {}

@Component({
  standalone: true,
  imports: [RouterLink],
  template: '<h1>Products</h1><p>Choose a product page.</p><a routerLink="/products/new">Create a product</a><a routerLink="/products/42">Open product 42</a>',
})
export class ProductListPage {}

@Component({
  standalone: true,
  template: '<h1>Create a product</h1>',
})
export class NewProductPage {}

@Component({
  standalone: true,
  template: '<h1>Product {{ id }}</h1><p>Product detail page.</p>',
})
export class ProductPage {
  readonly id =
    inject(ActivatedRoute).snapshot.paramMap.get('id') ?? 'unknown';
}

@Component({
  standalone: true,
  template: '<h1>Articles</h1><p>Article index page.</p>',
})
export class ArticleListPage {}

@Component({
  standalone: true,
  template: '<h1>Article {{ slug }}</h1>',
})
export class ArticlePage {
  readonly slug =
    inject(ActivatedRoute).snapshot.paramMap.get('slug') ?? 'unknown';
}

@Component({
  standalone: true,
  template: '<h1>Offers</h1><p>Browse current offers.</p>',
})
export class OffersPage {}

@Component({
  standalone: true,
  template: '<h1>Email offer</h1><p>This page was selected from a campaign link.</p>',
})
export class EmailOfferPage {}

@Component({
  standalone: true,
  imports: [RouterLink],
  template: '<main><h1>Page not found</h1><p>The address does not match a page in this application.</p><a routerLink="/home">Return to the home page</a></main>',
})
export class NotFoundPage {}
~~~

The product and article pages read their current path parameters from ActivatedRoute. Their route titles use the same route information to describe the page in the browser tab.

## 2. Define route titles, redirects, and fallbacks

### src/app/app.routes.ts

~~~ts
import type {
  RedirectFunction,
  ResolveFn,
  Routes,
} from '@angular/router';
import {
  AboutPage,
  ArticleListPage,
  ArticlePage,
  EmailOfferPage,
  HomePage,
  NewProductPage,
  NotFoundPage,
  OffersPage,
  ProductListPage,
  ProductPage,
} from './pages';

const productTitle: ResolveFn<string> = (route) => {
  const id = route.paramMap.get('id');
  return id ? 'Product ' + id : 'Product details';
};

const articleTitle: ResolveFn<string> = (route) => {
  const slug = route.paramMap.get('slug');
  return slug ? 'Article: ' + slug : 'Article';
};

const campaignRedirect: RedirectFunction = (snapshot) => {
  return snapshot.queryParams['source'] === 'email'
    ? '/offers/email'
    : '/offers';
};

export const routes: Routes = [
  { path: '', pathMatch: 'full', redirectTo: '/home' },

  { path: 'home', component: HomePage, title: 'Home' },
  { path: 'about', component: AboutPage, title: 'About' },

  {
    path: 'products/new',
    component: NewProductPage,
    title: 'Create product',
  },
  {
    path: 'products/:id',
    component: ProductPage,
    title: productTitle,
  },
  {
    path: 'products',
    component: ProductListPage,
    title: 'Products',
  },

  {
    path: 'legacy-products/:id',
    redirectTo: '/products/:id',
  },

  { path: 'news', redirectTo: 'articles' },
  { path: 'articles', component: ArticleListPage, title: 'Articles' },
  {
    path: 'articles/:slug',
    component: ArticlePage,
    title: articleTitle,
  },

  {
    path: 'promo',
    pathMatch: 'full',
    redirectTo: campaignRedirect,
  },
  { path: 'offers', component: OffersPage, title: 'Offers' },
  {
    path: 'offers/email',
    component: EmailOfferPage,
    title: 'Email offer',
  },

  {
    path: '**',
    component: NotFoundPage,
    title: 'Page not found',
  },
];
~~~

The order is intentional:

- The root redirect uses full matching so it only handles the empty root path.
- The static products/new path appears before products/:id.
- The product list follows its more specific product paths.
- The wildcard route is last so valid routes can match first.
- The news redirect uses the default prefix matching, so /news/story is redirected to /articles/story.

The productTitle and articleTitle resolvers create route titles from path parameters. The promo redirect chooses a destination from query parameters available during matching.

## 3. Add a custom title strategy

### src/app/app-title.strategy.ts

~~~ts
import { inject, Injectable } from '@angular/core';
import { Title } from '@angular/platform-browser';
import {
  RouterStateSnapshot,
  TitleStrategy,
} from '@angular/router';

@Injectable()
export class AppTitleStrategy extends TitleStrategy {
  private readonly documentTitle = inject(Title);

  override updateTitle(snapshot: RouterStateSnapshot): void {
    const routeTitle = this.buildTitle(snapshot);
    const formattedTitle = routeTitle
      ? routeTitle + ' | Angular Notes'
      : 'Angular Notes';

    this.documentTitle.setTitle(formattedTitle);
  }
}
~~~

The inherited buildTitle method returns the title selected from the router snapshot. The custom strategy adds a consistent Angular Notes suffix and updates the document through Angular's Title service.

## 4. Register the router and title strategy

### src/app/app.config.ts

~~~ts
import type { ApplicationConfig } from '@angular/core';
import { provideRouter, TitleStrategy } from '@angular/router';
import { AppTitleStrategy } from './app-title.strategy';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    { provide: TitleStrategy, useClass: AppTitleStrategy },
  ],
};
~~~

### src/app/app.component.ts

~~~ts
import { Component } from '@angular/core';
import { RouterLink, RouterOutlet } from '@angular/router';

@Component({
  standalone: true,
  selector: 'app-root',
  imports: [RouterLink, RouterOutlet],
  template: '<nav aria-label="Main navigation"><a routerLink="/home">Home</a><a routerLink="/about">About</a><a routerLink="/products">Products</a></nav><router-outlet></router-outlet>',
})
export class AppComponent {}
~~~

### src/main.ts

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch((error: unknown) => console.error(error));
~~~

## 5. Try the routes

Navigate manually and check both the visible page and the browser tab title:

| URL | Expected view | Expected document title |
| --- | --- | --- |
| / | Home | Home \| Angular Notes |
| /home | Home | Home \| Angular Notes |
| /about | About | About \| Angular Notes |
| /products | Product list | Products \| Angular Notes |
| /products/new | Create product | Create product \| Angular Notes |
| /products/42 | Product 42 | Product 42 \| Angular Notes |
| /legacy-products/42 | Redirects to product 42 | Product 42 \| Angular Notes |
| /news/story | Redirects to /articles/story | Article: story \| Angular Notes |
| /promo?source=email | Email offer | Email offer \| Angular Notes |
| /promo | Offers | Offers \| Angular Notes |
| /unknown-path | Not found | Page not found \| Angular Notes |

The redirect destination supplies the activated component and title. The redirect route itself does not display a page.

## 6. What this example does not configure

The wildcard route displays a client-side not-found view. It does not set an HTTP 404 status by itself. Configure the web server or SSR response handling separately if the deployed application must return a not-found HTTP status.

The example also does not implement product or article APIs. The page components show route state only; connect them to application data services as needed.

## Troubleshooting checklist

- If every URL returns Home, check that the empty redirect uses full path matching.
- If /products/new shows Product new, move the static route before the parameter route.
- If an unknown URL opens a valid page, check whether an earlier broad route captured it.
- If the browser title never changes, make sure the router is registered and the custom strategy is provided under TitleStrategy.
- If a redirect goes to the wrong section, make the destination root-absolute with a leading slash or adjust its relative target intentionally.
- If /news/story does not preserve the rest of the path, check that the redirect uses prefix matching and the destination route handles the remaining segments.
- If the visible heading is missing, add it to the component; a document title does not render page content.
- If the server must report 404, configure server or SSR status behavior in addition to the Angular wildcard route.

## What this example demonstrates

- Static route titles update the document title.
- ResolveFn can create route titles from parameters.
- TitleStrategy can apply shared title formatting.
- Redirects support default navigation, legacy paths, prefix mapping, and a conditional destination.
- Route ordering determines which matching configuration is used.
- A final wildcard route renders a useful not-found page.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="082-route-level-providers-and-route-configuration.md">082 - 82. Route-level providers and route configuration</a></td><td><a href="../lessons/083-page-titles-redirects-wildcard-routes-and-not-found-pages.md">Lesson</a></td><td><a href="../qnas/083-page-titles-redirects-wildcard-routes-and-not-found-pages.md">Q&amp;A</a></td><td><a href="084-preloading-scroll-restoration-and-router-testing.md">084 - 84. Preloading, scroll restoration, and router testing</a></td>
</tr>
  </tbody>
</table>
