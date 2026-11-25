# 83. Page titles, redirects, wildcard routes, and not-found pages

## What you will learn

After this lesson, you should be able to:

- Set a document title for each route.
- Use a resolver when a route title depends on route state.
- Customize how the application composes page titles.
- Redirect a URL to another route, including legacy paths.
- Choose between prefix and full path matching for redirects.
- Create a not-found page with a wildcard route.
- Order routes so specific routes are matched before fallbacks.
- Understand what an Angular not-found route does and does not do.

## 1. Give routes useful page titles

A route can define a title:

~~~ts
export const routes: Routes = [
  { path: '', component: HomePage, title: 'Home' },
  { path: 'about', component: AboutPage, title: 'About' },
  { path: 'products', component: ProductsPage, title: 'Products' },
];
~~~

When a route activates, Angular updates the document title. This is the text shown in the browser tab and used by assistive technology to identify the page. It complements the visible page heading; it does not replace an h1 in the page content.

Use a concise title that describes the current page. A site-wide title strategy can add a consistent application name or other shared formatting.

## 2. Use a resolver for a dynamic title

A route title may be a string or a resolver that returns a string. Use a ResolveFn when the title should be derived from route parameters, query parameters, or data available during route activation.

~~~ts
import type { ResolveFn } from '@angular/router';

export const productTitle: ResolveFn<string> = (route) => {
  const productId = route.paramMap.get('id');
  return productId ? 'Product ' + productId : 'Product details';
};

export const routes: Routes = [
  {
    path: 'products/:id',
    component: ProductPage,
    title: productTitle,
  },
];
~~~

The title resolver receives route information and returns the string Angular should use as that route's title. A resolver can also inject dependencies when it runs in Angular's injection context. Keep title resolution focused; load complex page data with a data resolver or another suitable data-loading pattern rather than making title logic do unrelated work.

## 3. Customize title composition with TitleStrategy

Angular's default title strategy uses the deepest primary route title it finds. A custom TitleStrategy is useful when the app needs a standard suffix, prefix, or another consistent title format.

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
    const title = routeTitle
      ? routeTitle + ' | Learning Notes'
      : 'Learning Notes';

    this.documentTitle.setTitle(title);
  }
}
~~~

Register it with the TitleStrategy token in application providers:

~~~ts
import { provideRouter, TitleStrategy } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    { provide: TitleStrategy, useClass: AppTitleStrategy },
  ],
};
~~~

The default strategy follows the primary outlet. A title set only on a named auxiliary route does not replace the primary page title by default. If the application needs titles that combine parallel outlet branches, define that policy in a custom strategy.

Do not set document.title independently in many components unless there is a specific reason. Route titles keep the behavior close to the route configuration; a strategy centralizes formatting.

## 4. Redirect routes

A redirect sends a matched URL to another URL instead of activating a component:

~~~ts
const routes: Routes = [
  { path: 'articles', redirectTo: '/blog' },
  { path: 'blog', component: BlogPage, title: 'Blog' },
];
~~~

A redirect is useful for:

- Choosing the default page for the application.
- Keeping old bookmarks working after a route rename.
- Mapping a legacy URL pattern to its replacement.
- Sending a user to an appropriate route based on an explicit redirect rule.

The target can be absolute or relative. A target beginning with a slash is absolute from the application root. A target without a leading slash is resolved relative to the route that matched.

## 5. Preserve path parameters when redirecting

A redirect can carry route parameters into the destination:

~~~ts
{
  path: 'legacy-user/:id',
  redirectTo: '/users/:id',
},
{
  path: 'users/:id',
  component: UserPage,
  title: 'User profile',
}
~~~

A visit to /legacy-user/42 redirects to /users/42. Use matching parameter names where values need to be carried forward.

Redirects are URL routing rules, not component navigation code. If the destination depends on information that is available only after a component loads or data resolves, use an appropriate guard or application flow instead.

## 6. Understand pathMatch

The pathMatch option controls how much of the URL a route must match. It matters especially on redirects.

- prefix is the default. The route path only needs to match the beginning of the remaining URL.
- full requires the entire remaining URL path to match.

For a root redirect, use full:

~~~ts
{ path: '', pathMatch: 'full', redirectTo: '/home' }
~~~

An empty path is a prefix of every URL. Without full, the root redirect can keep matching after its destination is selected, causing a redirect loop or intercepting routes that should have matched.

A prefix redirect can be useful when an entire old path prefix has moved:

~~~ts
{ path: 'news', redirectTo: 'articles' }
~~~

With the default prefix matching, /news redirects to /articles and deeper paths under /news can retain their remaining segments under the new prefix. Use full when only one exact path should redirect.

## 7. Conditional redirects

In current Angular routing, redirectTo can also be a RedirectFunction. It receives partial route-matching information and returns a string or UrlTree.

~~~ts
import type { RedirectFunction } from '@angular/router';

export const promoRedirect: RedirectFunction = (snapshot) => {
  return snapshot.queryParams['campaign'] === 'spring'
    ? '/offers/spring'
    : '/offers';
};
~~~

A redirect function runs during route matching, before the router has a complete activated route state. It should use only the information available at that stage. It may use inject synchronously in its injection context, but it should not assume resolved route data or a fully activated component.

Use a conditional redirect for simple URL selection. For multi-step authorization or decisions that need resolved application data, choose a guard or another explicit navigation workflow.

## 8. Wildcard routes and not-found pages

The wildcard path, written as two asterisks, matches URLs that did not match an earlier route. A common use is rendering a not-found page:

~~~ts
export const routes: Routes = [
  { path: '', pathMatch: 'full', redirectTo: '/home' },
  { path: 'home', component: HomePage, title: 'Home' },
  { path: 'about', component: AboutPage, title: 'About' },
  { path: '**', component: NotFoundPage, title: 'Page not found' },
];
~~~

Place the wildcard after the routes it should fall back from. Angular uses first-match-wins route matching. If the wildcard appears before a valid page route, it can capture that URL and prevent the intended route from being considered.

A useful not-found page should:

- Clearly explain that the requested page was not found.
- Offer a practical way to continue, such as a link to the home page.
- Use a meaningful document title.
- Keep the message understandable without relying only on color or an icon.

A wildcard page handles Angular client-side routing. It does not by itself make the web server return an HTTP 404 status. Production server and SSR status handling are separate concerns.

## 9. Wildcards inside child routes

A feature can have a wildcard among its children:

~~~ts
{
  path: 'admin',
  component: AdminShell,
  children: [
    { path: 'users', component: AdminUsersPage },
    { path: 'teams', component: AdminTeamsPage },
    { path: '**', component: AdminNotFoundPage },
  ],
}
~~~

This catches unrecognized paths under the admin route, such as /admin/unknown. A root-level wildcard catches paths that do not match the application's top-level routes.

Choose the fallback level that matches the user experience. A feature-specific not-found page can preserve its shell and navigation; the root not-found route handles the rest of the app.

## 10. Route order and specificity

Angular tries routes in configuration order and uses the first route that matches. Put specific routes before broad routes and place wildcard routes last.

~~~ts
export const routes: Routes = [
  { path: 'products/new', component: NewProductPage },
  { path: 'products/:id', component: ProductPage },
  { path: 'products', component: ProductsPage },
  { path: '**', component: NotFoundPage },
];
~~~

Here, /products/new should be matched by the static route before the parameter route. A parameter route such as products/:id could otherwise interpret new as an id.

Order is not a ranking system in which Angular later compares all matches. Once a route matches and its remaining segments can be handled, later route definitions are not used for that navigation.

## 11. Redirects, guards, and not-found routes serve different purposes

- A redirect rewrites a URL to another route.
- A guard allows or blocks a navigation, or can return a UrlTree to send navigation elsewhere.
- A wildcard route handles paths that no earlier route accepted.
- A not-found component communicates that there is no matching page.

Do not use a wildcard redirect to the home page when a real not-found experience is desired. Redirecting every unknown URL can hide broken links and gives users no indication that the requested page does not exist.

Client-side guards and redirects are also not server authorization. Protected data and operations must be authorized by the backend.

## 12. Common mistakes

- **Root redirect without full matching:** the empty prefix can match every URL and create a loop.
- **Wildcard placed too early:** it captures paths before their specific routes are reached.
- **Static route placed after a parameter route:** a word such as new may be interpreted as a parameter.
- **Redirect target has the wrong base:** decide whether the target should be root-absolute or relative.
- **Legacy parameter omitted:** include the relevant parameter in the destination pattern.
- **Expecting a not-found component to set an HTTP status:** the wildcard changes the client-side view; server status handling is separate.
- **Using an empty page title everywhere:** define useful route titles so tabs and assistive technology identify the current page.
- **Confusing the document title with the visible heading:** set both when the page needs both.
- **Assuming auxiliary route titles are included by default:** the default strategy uses the deepest primary route title.

## Key takeaways

- Route title metadata updates the browser document title during navigation.
- A ResolveFn can derive a title from the matched route.
- TitleStrategy centralizes title formatting; the default follows the primary route branch.
- Redirects can be absolute, relative, parameterized, or conditional.
- Root empty-path redirects should use full path matching.
- Wildcard routes are fallbacks and belong after specific route definitions.
- A client-side not-found page does not itself set an HTTP 404 response.

## Further reading

- [Define routes: redirects, titles, and wildcards](https://angular.dev/guide/routing/define-routes)
- [Redirecting routes](https://angular.dev/guide/routing/redirecting-routes)
- [TitleStrategy API](https://angular.dev/api/router/TitleStrategy)
- [RedirectFunction API](https://angular.dev/api/router/RedirectFunction)
