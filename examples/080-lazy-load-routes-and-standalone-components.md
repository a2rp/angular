# Lazy-load routes and standalone components: example

This standalone example keeps the home page eager, lazy-loads an About page with `loadComponent`, and lazy-loads an admin route tree with `loadChildren`. The admin shell and its pages are then loaded individually with `loadComponent`.

The route configuration uses a normal dynamic import. In a production build, the bundler can emit separate chunks for those imports and the router requests them as their route boundaries are reached.

## File layout

~~~text
src/
├── main.ts
└── app/
    ├── app.component.ts
    ├── app.config.ts
    ├── app.routes.ts
    ├── home.page.ts
    ├── not-found.page.ts
    ├── about/
    │   └── about.page.ts
    └── admin/
        ├── admin.routes.ts
        ├── admin-shell.page.ts
        ├── dashboard.page.ts
        └── users.page.ts
~~~

Create a standalone Angular application, then add or adapt these files.

## 1. Keep the primary home page eager

Create `src/app/home.page.ts`:

~~~ts
import {Component} from '@angular/core';
import {RouterLink} from '@angular/router';

@Component({
  selector: 'app-home-page',
  standalone: true,
  imports: [RouterLink],
  template: '<main><h1>Home</h1><p>This primary page is eagerly available.</p><nav aria-label="Explore features"><a routerLink="/about">Open the lazy About page</a><a routerLink="/admin">Open the lazy Admin feature</a></nav></main>',
})
export class HomePage {}
~~~

The home page is a top-level import in the main route file, so it is part of the eager route code.

## 2. Add the standalone pages that will load later

Create `src/app/about/about.page.ts`:

~~~ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-about-page',
  standalone: true,
  template: '<main><h1>About</h1><p>This standalone page was loaded through loadComponent.</p></main>',
})
export class AboutPage {}
~~~

Create `src/app/not-found.page.ts`:

~~~ts
import {Component} from '@angular/core';
import {RouterLink} from '@angular/router';

@Component({
  selector: 'app-not-found-page',
  standalone: true,
  imports: [RouterLink],
  template: '<main><h1>Page not found</h1><p>No route matches this URL.</p><a routerLink="/">Return home</a></main>',
})
export class NotFoundPage {}
~~~

## 3. Define the admin route tree in its own file

Create `src/app/admin/admin.routes.ts`:

~~~ts
import type {Routes} from '@angular/router';

export const ADMIN_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./admin-shell.page').then(module => module.AdminShellPage),
    children: [
      {
        path: '',
        pathMatch: 'full',
        redirectTo: 'dashboard',
      },
      {
        path: 'dashboard',
        loadComponent: () =>
          import('./dashboard.page').then(module => module.AdminDashboardPage),
      },
      {
        path: 'users',
        loadComponent: () =>
          import('./users.page').then(module => module.AdminUsersPage),
      },
    ],
  },
];
~~~

The main router does not import this file directly. It will request it through `loadChildren` when an `/admin` URL is matched.

Create `src/app/admin/admin-shell.page.ts`:

~~~ts
import {Component} from '@angular/core';
import {RouterLink, RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-admin-shell',
  standalone: true,
  imports: [RouterLink, RouterOutlet],
  template: '<main><h1>Admin</h1><nav aria-label="Admin"><a routerLink="dashboard">Dashboard</a><a routerLink="users">Users</a></nav><router-outlet /></main>',
})
export class AdminShellPage {}
~~~

Create `src/app/admin/dashboard.page.ts`:

~~~ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-admin-dashboard-page',
  standalone: true,
  template: '<section><h2>Admin dashboard</h2><p>This screen has its own lazy component loader.</p></section>',
})
export class AdminDashboardPage {}
~~~

Create `src/app/admin/users.page.ts`:

~~~ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-admin-users-page',
  standalone: true,
  template: '<section><h2>Admin users</h2><p>This screen is loaded only when its child route activates.</p></section>',
})
export class AdminUsersPage {}
~~~

The admin shell must contain a `RouterOutlet` because its child route component renders inside it. The standalone shell imports `RouterOutlet` and `RouterLink` for its template.

## 4. Configure eager and lazy routes

Create `src/app/app.routes.ts`:

~~~ts
import type {Routes} from '@angular/router';

import {HomePage} from './home.page';
import {NotFoundPage} from './not-found.page';

export const routes: Routes = [
  {
    path: '',
    pathMatch: 'full',
    component: HomePage,
    title: 'Home',
  },
  {
    path: 'about',
    loadComponent: () =>
      import('./about/about.page').then(module => module.AboutPage),
    title: 'About',
  },
  {
    path: 'admin',
    loadChildren: () =>
      import('./admin/admin.routes').then(module => module.ADMIN_ROUTES),
  },
  {
    path: '**',
    component: NotFoundPage,
    title: 'Page not found',
  },
];
~~~

The root route and not-found component are eager imports. The About page has a standalone component boundary. The Admin feature has a separate route-configuration boundary followed by standalone component boundaries within its route tree.

## 5. Register the router and bootstrap

Create `src/app/app.component.ts`:

~~~ts
import {Component} from '@angular/core';
import {RouterLink, RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterLink, RouterOutlet],
  template: '<header><nav aria-label="Main navigation"><a routerLink="/">Home</a><a routerLink="/about">About</a><a routerLink="/admin">Admin</a></nav></header><router-outlet />',
})
export class AppComponent {}
~~~

Create `src/app/app.config.ts`:

~~~ts
import type {ApplicationConfig} from '@angular/core';
import {provideRouter} from '@angular/router';

import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)],
};
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

This configuration uses the router's default no-preloading behavior. The Admin route configuration loads when the visitor first navigates to `/admin`.

## 6. Try each boundary and inspect the build

Run the Angular development server and navigate through the app:

1. Open `/`. The eager Home page appears.
2. Open About. Angular loads the standalone page through `loadComponent`.
3. Open Admin. Angular first loads `admin.routes.ts` through `loadChildren`, then loads the shell and default dashboard through their component loaders.
4. Open Users from the Admin shell. The Users page is loaded when that child route activates.
5. Build for production with `ng build` and inspect the emitted files under `dist/`. Confirm the initial output and lazy chunks rather than assuming a particular chunk count or filename.

If the first visit to Admin feels slow, consider an appropriate preloading strategy for its lazy route configuration. Preloading uses extra bandwidth and memory; decide from build and usage measurements. Angular's built-in `PreloadAllModules` is documented for lazy-loaded route modules/configurations. Do not assume it preloads every `loadComponent` chunk.

## What this example does not do

- It does not implement authentication or authorization. A lazy Admin URL is still client code; use guards for navigation policy and enforce access to protected data in the API.
- It does not use `@defer`. That feature defers template content within an already active component rather than splitting an entire route.
- It does not apply a preloading strategy. Add one only after deciding its bandwidth and first-navigation trade-off.

## Related material

- [Lesson: Lazy-load routes and standalone components](../lessons/080-lazy-load-routes-and-standalone-components.md)
- [Q&A: Lazy-load routes and standalone components](../qnas/080-lazy-load-routes-and-standalone-components.md)
- [Resolvers, route data, and loading a page before activation](../lessons/079-resolvers-route-data-and-loading-a-page-before-activation.md)
- Angular lazy-loaded routes guide
- Angular route loading strategies
- Standalone component imports

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="079-resolvers-route-data-and-loading-a-page-before-activation.md">079 - Resolvers, route data, and loading a page before activation</a></td><td><a href="../lessons/080-lazy-load-routes-and-standalone-components.md">Lesson</a></td><td><a href="../qnas/080-lazy-load-routes-and-standalone-components.md">Q&amp;A</a></td><td><a href="081-nested-routes-child-outlets-and-auxiliary-routes.md">081 - 81. Nested routes, child outlets, and auxiliary routes</a></td>
</tr>
  </tbody>
</table>
