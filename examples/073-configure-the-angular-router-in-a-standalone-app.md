# Example: Configure the Angular Router in a standalone app

Wire a small standalone Angular application to three routes: a root redirect, a home page, an about page, and a not-found page. This demonstrates the initial router configuration. Declarative links and active-link styles are covered in the next chapter.

The snippets use the common Angular CLI files under `src/app/`. Match the import paths and root component class to your project's actual filenames.

## 1. Add routed page components

Create `src/app/home-page.ts`:

~~~ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-home-page',
  standalone: true,
  template: `
    <main>
      <h2>Home</h2>
      <p>This view is rendered for the /home route.</p>
    </main>
  `,
})
export class HomePage {}
~~~

Create `src/app/about-page.ts`:

~~~ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-about-page',
  standalone: true,
  template: `
    <main>
      <h2>About</h2>
      <p>This view is rendered for the /about route.</p>
    </main>
  `,
})
export class AboutPage {}
~~~

Create `src/app/not-found-page.ts`:

~~~ts
import {Component} from '@angular/core';

@Component({
  selector: 'app-not-found-page',
  standalone: true,
  template: `
    <main>
      <h2>Page not found</h2>
      <p>The URL does not match a page in this application.</p>
    </main>
  `,
})
export class NotFoundPage {}
~~~

## 2. Define the routes

Create or update `src/app/app.routes.ts`:

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

Notice that route paths do not start with a slash. Angular matches the root URL `/` against the empty path and redirects it to `/home`. The exact path match keeps that redirect scoped to the root. The wildcard is last so the named routes get a chance to match first.

## 3. Register the router in the application config

Create or update `src/app/app.config.ts`:

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

If the CLI generated other providers in this file, keep them and add `provideRouter(routes)` to the existing providers array.

The generated bootstrap file should already pass `appConfig` to `bootstrapApplication()`. Keep that connection. A typical bootstrap looks like this:

~~~ts
bootstrapApplication(App, appConfig)
  .catch((error: unknown) => console.error(error));
~~~

The exact root component import and class name depend on the project. You do not need to call `provideRouter()` from a routed page.

## 4. Add the outlet to the root component

Update the root standalone component so it imports `RouterOutlet` and provides a place for the active route:

~~~ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  template: `
    <header>
      <h1>Router setup demo</h1>
      <p>Open /home or /about in the address bar.</p>
    </header>

    <router-outlet />
  `,
})
export class App {}
~~~

If your CLI project already has a root component, update that component instead of creating a second root. Keep its existing styles and other imports as needed.

## 5. Navigate and observe

Start the application with `ng serve`, then open these URLs:

| URL | Expected result |
| --- | --- |
| `/` | Redirects to `/home` and displays the home page |
| `/home` | Displays the home page |
| `/about` | Displays the about page |
| `/does-not-exist` | Displays the not-found page |

In this initial setup you can type the URLs directly. The next lesson adds in-app navigation links and active-link state.

## 6. Understand what the configuration does

- `routes` describes how URL paths map to views.
- `provideRouter(routes)` installs the router at application scope.
- `RouterOutlet` marks where the activated route component appears.
- `title` updates the document title for a matched route.
- `pathMatch: 'full'` prevents the empty-path redirect from acting as a prefix.
- `'**'` is a last-resort route and belongs after the ordinary routes.

The Angular Router handles client navigation, but production hosting must also serve the application's entry document for valid deep URLs. That server fallback is configured separately.

## Troubleshooting checklist

If a route does not appear:

1. Confirm the route component import path and exported class name.
2. Confirm `provideRouter(routes)` is in the application providers.
3. Confirm the CLI bootstrap still uses the app config.
4. Confirm the root component imports `RouterOutlet` and includes `<router-outlet />`.
5. Check that route paths have no leading slash.
6. Keep the wildcard after all normal paths.
7. If direct refresh of a deep URL fails only after deployment, configure the web server's application fallback.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="072-signal-forms-schemas-validation-and-custom-controls.md">072 - Signal Forms: schemas, validation, and custom controls</a></td><td><a href="../lessons/073-configure-the-angular-router-in-a-standalone-app.md">Lesson</a></td><td><a href="../qnas/073-configure-the-angular-router-in-a-standalone-app.md">Q&amp;A</a></td><td><a href="074-routeroutlet-routerlink-and-active-route-links.md">074 - RouterOutlet, RouterLink, and active route links</a></td>
</tr>
  </tbody>
</table>
