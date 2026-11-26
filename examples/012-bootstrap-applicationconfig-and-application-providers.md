# Examples: Bootstrap, ApplicationConfig, and Application Providers

This walkthrough configures an Angular standalone application with a typed settings token, the router, and the HTTP client. A routed standalone page injects the app title. The sample shows how startup configuration and template imports work together.

The code uses the common CLI layout with files such as src/main.ts, src/app/app.ts, and src/app/app.html. Generated filenames can differ between Angular versions; keep the project's existing layout and adapt the paths.

## 1. Check the workspace

Run commands from the Angular workspace root, the directory containing angular.json:

~~~sh
ng version
~~~

Use the application created in the previous chapters, or a disposable standalone Angular app. Inspect its existing main entry point and app config before editing so that existing providers are preserved.

## 2. Define an app settings token

Create src/app/app-settings.ts:

~~~ts
import { InjectionToken } from '@angular/core';

export interface AppSettings {
  appTitle: string;
  apiBaseUrl: string;
}

export const APP_SETTINGS =
  new InjectionToken<AppSettings>('app.settings');
~~~

The interface documents the value shape and provides compile-time checking. The InjectionToken is the runtime key Angular uses for injection.

## 3. Create a routed page

Create src/app/home.page.ts:

~~~ts
import { Component, inject } from '@angular/core';
import { APP_SETTINGS } from './app-settings';

@Component({
  selector: 'app-home-page',
  standalone: true,
  template: '<main><h1>{{ settings.appTitle }}</h1><p>Application settings were injected from the root configuration.</p></main>',
})
export class HomePage {
  readonly settings = inject(APP_SETTINGS);
}
~~~

This standalone page injects the settings token. Its template does not need to import the token; tokens are used by TypeScript injection code, not as template directives.

## 4. Declare routes

Create src/app/app.routes.ts:

~~~ts
import { Routes } from '@angular/router';
import { HomePage } from './home.page';

export const routes: Routes = [
  { path: '', component: HomePage },
  { path: '**', redirectTo: '' },
];
~~~

The empty path displays HomePage. The wildcard route redirects other paths to the page.

## 5. Configure application providers

Create or update src/app/app.config.ts:

~~~ts
import { provideHttpClient } from '@angular/common/http';
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { APP_SETTINGS, type AppSettings } from './app-settings';
import { routes } from './app.routes';

const settings: AppSettings = {
  appTitle: 'Task Tracker',
  apiBaseUrl: '/api',
};

export const appConfig: ApplicationConfig = {
  providers: [
    { provide: APP_SETTINGS, useValue: settings },
    provideRouter(routes),
    provideHttpClient(),
  ],
};
~~~

The settings value, router setup, and HTTP client setup are available through the application's injector. Keep any providers already present in your generated app.config.ts and merge these entries into its providers array.

The sample registers HttpClient so services can inject it. A request will only succeed when an API server or development proxy is available at the configured URL.

## 6. Make the root component render the router

Update the root component, commonly src/app/app.ts:

~~~ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  templateUrl: './app.html',
})
export class App {}
~~~

Update its template, commonly src/app/app.html:

~~~html
<router-outlet />
~~~

There are two different kinds of setup here:

- provideRouter(routes) configures router services and routes at application scope.
- RouterOutlet is imported by App because its template uses the router-outlet element.

If the CLI-generated root component uses inline template syntax, keep that style and include RouterOutlet in its imports.

## 7. Bootstrap the app

Update src/main.ts to pass the root component and app config to bootstrapApplication:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { App } from './app/app';
import { appConfig } from './app/app.config';

bootstrapApplication(App, appConfig).catch((error) => {
  console.error(error);
});
~~~

Use the root component's actual exported class name and path. Some workspaces already have the bootstrap call; if so, confirm it passes the intended appConfig instead of adding a second startup call.

## 8. Run and inspect

From the workspace root, start the app:

~~~sh
ng serve
~~~

Open the local address printed by the CLI. The page should show “Task Tracker”. This confirms that bootstrap started the root component, the router rendered HomePage, and HomePage resolved APP_SETTINGS from the application injector.

Then build the app:

~~~sh
ng build
~~~

Resolve compilation errors before continuing. If the app uses different generated filenames or a custom project target, follow its actual workspace configuration.

## 9. Experiment with provider behavior

Try each change in a disposable branch or practice app, then restore the working setup:

1. Change appTitle in app.config.ts and observe the page update.
2. Temporarily remove the APP_SETTINGS provider. The page should report that Angular cannot find a provider for that token.
3. Temporarily remove RouterOutlet from the root component's imports. The template should report an unknown router-outlet dependency.
4. Temporarily remove provideRouter(routes). The router outlet no longer has the router configuration it needs.
5. Add a second APP_SETTINGS provider at a closer component scope and observe that the closer injector's value can take precedence for that component subtree.

These are separate layers: application providers configure services and values, while component imports define template dependencies.

## 10. Keep configuration safe

The sample's title and relative API URL are public client settings. Do not put passwords, private API keys, or database credentials in the app settings token or any other file bundled for the browser. Browser code and its downloaded configuration can be inspected by users.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="011-run-serve-build-and-inspect-an-angular-application.md">011 - Run, Serve, Build, and Inspect an Angular Application</a></td><td><a href="../lessons/012-bootstrap-applicationconfig-and-application-providers.md">Lesson</a></td><td><a href="../qnas/012-bootstrap-applicationconfig-and-application-providers.md">Q&amp;A</a></td><td><a href="013-components-metadata-and-component-boundaries.md">013 - 13. Components, metadata, and component boundaries</a></td>
</tr>
  </tbody>
</table>
