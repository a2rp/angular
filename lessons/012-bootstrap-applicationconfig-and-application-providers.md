# 12. Bootstrap, ApplicationConfig, and application providers

A browser needs an entry point before it can render an Angular application. In a modern standalone application, that entry point is usually the call to `bootstrapApplication` in `main.ts`. This lesson explains how that call starts the root component, how `ApplicationConfig` supplies application-wide providers, and how a dependency token can make a shared value available throughout the app.

## Learning goals

After this lesson, you should be able to:

- Explain what `bootstrapApplication` starts and what it returns.
- Describe the role of the root component and `ApplicationConfig`.
- Register application-wide dependencies and framework features.
- Provide a typed value with an `InjectionToken`.
- Distinguish application providers from component template imports.
- Recognize when an existing NgModule provider setup can be bridged into a standalone app.

## 1. The bootstrap entry point

A typical standalone app starts from a small entry file:

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { App } from './app/app';
import { appConfig } from './app/app.config';

bootstrapApplication(App, appConfig).catch((error) => {
  console.error(error);
});
```

The first argument is the root component. Angular creates it and renders its template as the application’s root. The root component must be standalone. The optional second argument is an `ApplicationConfig`, which can configure providers and other application-level behavior.

`bootstrapApplication` returns a `Promise<ApplicationRef>`. The promise resolves after Angular bootstraps the application. Handling a rejection is useful because a startup failure can otherwise leave a blank page with only a console error.

Some projects keep the configuration inline:

```ts
bootstrapApplication(App, {
  providers: [],
});
```

Other projects export it from `app.config.ts`, which keeps startup and configuration separate. Follow the project’s generated structure rather than creating duplicate configuration files.

## 2. ApplicationConfig

`ApplicationConfig` is Angular’s application-level configuration object. Its `providers` array can contain regular providers and `EnvironmentProviders` returned by framework setup functions.

```ts
import { ApplicationConfig } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [],
};
```

Providers registered here are available to the root component and its descendants, unless a nearer injector supplies a different value for the same token. Application configuration is a suitable place for shared services, router setup, HTTP client setup, and application-wide values.

This is different from a component’s `imports` array. Component imports make standalone components, directives, and pipes usable in that component’s template. Application providers configure dependency injection and framework behavior.

## 3. Application-wide providers

A provider tells Angular how to supply a value when code asks for a dependency token. A class provider can often use shorthand:

```ts
providers: [SessionService]
```

The longer form makes the token and implementation explicit:

```ts
providers: [
  { provide: SessionService, useClass: SessionService },
]
```

Other common provider forms include:

- `useValue`: supply an already-created value.
- `useClass`: create an instance of a class.
- `useFactory`: call a function to create a value.
- `useExisting`: alias one token to another.

Use the form that communicates the setup clearly. A value provider is useful for application settings or other objects that should be shared.

## 4. InjectionToken for non-class dependencies

JavaScript interfaces and TypeScript type aliases are erased when TypeScript compiles to JavaScript. They cannot be used as runtime dependency-injection tokens. Use an `InjectionToken` when you want to inject a value described by an interface or another non-class type.

```ts
import { InjectionToken } from '@angular/core';

export interface AppSettings {
  appTitle: string;
  apiBaseUrl: string;
}

export const APP_SETTINGS =
  new InjectionToken<AppSettings>('app.settings');
```

The generic type provides compile-time checking. The token object is the runtime identity Angular uses for lookup. The descriptive string helps identify the token in diagnostics; it is not the lookup key by itself.

Provide a value for that token in the application configuration:

```ts
import { ApplicationConfig } from '@angular/core';
import { APP_SETTINGS } from './app-settings';

export const appConfig: ApplicationConfig = {
  providers: [
    {
      provide: APP_SETTINGS,
      useValue: {
        appTitle: 'Task Tracker',
        apiBaseUrl: '/api',
      },
    },
  ],
};
```

A component or service can inject the token:

```ts
import { Component, inject } from '@angular/core';
import { APP_SETTINGS } from './app-settings';

@Component({
  selector: 'app-header',
  template: '<h1>{{ settings.appTitle }}</h1>',
})
export class Header {
  readonly settings = inject(APP_SETTINGS);
}
```

The provider is configured once at the application level, and the value can be injected by descendants that need it.

## 5. Configure framework features with provider functions

Modern Angular features commonly expose provider functions. Add their results to the application providers array:

```ts
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
  ],
};
```

These functions configure dependency-injection providers for the router and HTTP client. They are not template imports. For example, a component that uses `<router-outlet>` still imports `RouterOutlet` in its own component metadata, even though `provideRouter(routes)` is configured for the application.

Angular CLI templates can change over time. Inspect the existing generated `app.config.ts` and keep any providers your project already needs when adding another one.

## 6. Services provided in root

A service decorated with `providedIn: 'root'` is available from the application’s root injector without adding the class to `app.config.ts`:

```ts
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class SessionService {}
```

Use application configuration when a dependency requires an explicit value or feature setup, such as `provideRouter(routes)`. Use `providedIn: 'root'` for a service whose own declaration can describe its default application-wide availability. Avoid registering the same dependency in multiple places without a reason: a closer provider can create a separate instance for its injector scope.

Provider scopes and instance behavior are covered in more depth in the dependency-injection lessons.

## 7. Existing NgModule providers

When integrating an older library that still exposes providers through an NgModule, `importProvidersFrom` can collect those providers for application configuration:

```ts
import { importProvidersFrom } from '@angular/core';
import { SomeLegacyModule } from 'some-legacy-library';

export const appConfig: ApplicationConfig = {
  providers: [
    importProvidersFrom(SomeLegacyModule),
  ],
};
```

This is a compatibility bridge for provider configuration. It does not make an NgModule necessary for a new standalone app, and it does not import the module’s components into a standalone component template. Prefer a library’s dedicated standalone provider function when it offers one.

## 8. Change detection and generated configuration

Use the configuration created for your Angular version as the starting point. Newer Angular versions use zoneless change detection by default, so copying older examples that add a Zone.js configuration provider can be unnecessary or can change behavior. Add change-detection configuration only when the project’s Angular version and requirements call for it.

## Common mistakes

- Passing a non-standalone component as the root to `bootstrapApplication`.
- Treating `app.config.ts` as required; it is a useful organization choice, while the config object itself can also be inline.
- Putting a service in a component’s `imports` array. Services are injected through providers, not template imports.
- Putting `provideRouter(routes)` in a component’s `imports` array. The router setup is an application provider; template directives such as `RouterOutlet` are component imports.
- Trying to inject a TypeScript interface directly instead of creating an `InjectionToken`.
- Replacing the generated provider array and accidentally deleting existing configuration.
- Placing private credentials in browser-delivered configuration. Anything shipped to a browser can be inspected by users; client-side settings are not secrets.
- Adding a legacy provider configuration without checking whether the library has a current provider function.

## Practical checklist

- [ ] The root component passed to `bootstrapApplication` is standalone.
- [ ] The application bootstraps from the project’s actual entry file.
- [ ] Shared application providers are registered in one clear place.
- [ ] Non-class values use a stable runtime token such as `InjectionToken`.
- [ ] Template dependencies are imported by the component whose template uses them.
- [ ] Existing generated providers are preserved.
- [ ] Browser configuration contains no secrets.

## Next lesson

Continue with [Components, metadata, and component boundaries](013-components-metadata-and-component-boundaries.md).

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="011-run-serve-build-and-inspect-an-angular-application.md">011 - Run, Serve, Build, and Inspect an Angular Application</a></td><td><a href="../qnas/012-bootstrap-applicationconfig-and-application-providers.md">Q&amp;A</a></td><td><a href="../examples/012-bootstrap-applicationconfig-and-application-providers.md">Example</a></td><td><a href="013-components-metadata-and-component-boundaries.md">013 - 13. Components, metadata, and component boundaries</a></td>
</tr>
  </tbody>
</table>
