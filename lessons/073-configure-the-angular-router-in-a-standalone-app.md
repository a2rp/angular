# Configure the Angular Router in a standalone app

Angular Router maps browser URLs to application views. In a standalone application, define a route table and register it once in the application-level providers with `provideRouter()` from `@angular/router`.

This lesson covers the initial configuration: route objects, the application provider setup, default and not-found routes, and the minimum outlet needed to render the active route. The next lesson explores links and outlets in more detail.

## Learning outcomes

After this lesson, you should be able to:

- Define a typed route table with `Routes`.
- Map URL paths to standalone components.
- Register the route table through `provideRouter()` in `ApplicationConfig`.
- Configure a root path, redirect, and wildcard fallback.
- Explain route order and why an empty-path redirect needs `pathMatch: 'full'`.
- Identify the minimum root-template requirement for routed content to appear.

## Router setup in a standalone application

Angular Router is provided by the separate `@angular/router` package. A standalone application does not need an NgModule to configure it. Its setup has three parts:

1. Define a `Routes` array.
2. Pass that array to `provideRouter(routes)` in the application providers.
3. Put a `<router-outlet>` in a component template so the currently matched route has a place to render.

Angular CLI projects normally keep these pieces in `src/app/app.routes.ts` and `src/app/app.config.ts`. The generated `main.ts` bootstraps the root component with the application configuration.

## Define the route table

A route maps a URL path to a component or to another routing behavior. A small route table can look like this:

~~~ts
import {Routes} from '@angular/router';
import {AboutPage} from './about-page';
import {HomePage} from './home-page';
import {NotFoundPage} from './not-found-page';

export const routes: Routes = [
  {
    path: '',
    pathMatch: 'full',
    redirectTo: 'home',
  },
  {
    path: 'home',
    component: HomePage,
    title: 'Home',
  },
  {
    path: 'about',
    component: AboutPage,
    title: 'About',
  },
  {
    path: '**',
    component: NotFoundPage,
    title: 'Page not found',
  },
];
~~~

Route `path` values are application paths without a leading slash. A route path of `'about'` matches the browser URL `/about`.

The `component` property identifies what to render for a match. The optional `title` gives the activated page a document title; defining useful titles also helps people using assistive technology understand which page is active.

## Register the router with `provideRouter()`

Import the routes into the standalone application's configuration and register the router in its providers:

~~~ts
import {ApplicationConfig} from '@angular/core';
import {provideRouter} from '@angular/router';

import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
  ],
};
~~~

If the generated `app.config.ts` already has providers for other features, keep them and add `provideRouter(routes)` to the existing array. Do not replace the generated application configuration just to add routing.

The CLI bootstrap already passes `appConfig` to `bootstrapApplication()`. A manually bootstrapped standalone application uses the same provider setup:

~~~ts
bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes)],
});
~~~

Register one router at the application level. Individual route components do not each call `provideRouter()`.

## Understand default and fallback routes

The empty path `''` represents the application root. It can render a home component directly or redirect to a named route. When redirecting from an empty path, use `pathMatch: 'full'` so the redirect applies to the exact root URL rather than acting as a prefix match.

A wildcard path `'**'` catches URLs not matched by earlier routes. Put it last. The router checks routes in order and uses the first matching route, so a broad route placed earlier can prevent a more specific route from being reached.

For example, if an app later has both `users/new` and `users/:id`, place the fixed `users/new` route before the parameterized route. Detailed route parameters and navigation behavior are covered in later chapters.

## Provide a rendering location

The root component must import `RouterOutlet` and include its directive in the template:

~~~ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet],
  template: '<router-outlet />',
})
export class AppComponent {}
~~~

When the URL matches a route, Angular creates that route's component at the outlet. The outlet itself remains available for later navigations. A complete application can keep shared layout elements around the outlet.

This is the minimum rendering setup. The next lesson covers how to add declarative links with `RouterLink` and active-state styling with `RouterLinkActive`.

## Common configuration errors

| Error | Result | Fix |
| --- | --- | --- |
| Calling `provideRouter()` in a route component | Router providers are registered at the wrong scope or multiple times | Register it once in application providers |
| Forgetting `provideRouter(routes)` | The route table is not configured in the application | Add it to the existing `ApplicationConfig.providers` |
| Omitting `RouterOutlet` | Routes may match, but routed components have no rendering location | Import `RouterOutlet` and include `<router-outlet>` |
| Writing `path: '/about'` | Route path syntax is incorrect for the route table | Use `path: 'about'`; the browser URL still begins with `/` |
| Putting `'**'` before regular routes | The wildcard can capture paths that should match later entries | Keep the wildcard route last |
| Redirecting `''` without `pathMatch: 'full'` | The empty path can match as a prefix and redirect unexpectedly | Add `pathMatch: 'full'` to an exact root redirect |
| Replacing all existing app providers | Other configured application services may stop being provided | Add the router provider alongside the existing providers |

## Deployment note

Client-side routing changes the URL without requesting a new application document for every in-app navigation. A production web server must also serve the Angular application entry point for valid client routes when someone opens or refreshes a deep URL directly. Configure that server fallback as part of deployment; the router provider alone cannot configure the web server.

## Setup checklist

- [ ] The route table is typed as `Routes`.
- [ ] Every component route imports its component.
- [ ] Paths have no leading slash.
- [ ] The empty-path route is intentional and uses an exact match when redirecting.
- [ ] The wildcard route appears last.
- [ ] `provideRouter(routes)` is in the existing application providers.
- [ ] The root template includes `RouterOutlet`.
- [ ] Each route has a meaningful title where appropriate.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="072-signal-forms-schemas-validation-and-custom-controls.md">072 - Signal Forms: schemas, validation, and custom controls</a></td><td><a href="../qnas/073-configure-the-angular-router-in-a-standalone-app.md">Q&amp;A</a></td><td><a href="../examples/073-configure-the-angular-router-in-a-standalone-app.md">Example</a></td><td><a href="074-routeroutlet-routerlink-and-active-route-links.md">074 - RouterOutlet, RouterLink, and active route links</a></td>
</tr>
  </tbody>
</table>
