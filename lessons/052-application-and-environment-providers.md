# Application and environment providers

A standalone Angular application needs a place to register services and framework features that belong to the application as a whole. The root EnvironmentInjector is created during bootstrap, and its providers are available to the root component and its descendants unless a nearer provider overrides them.

ApplicationConfig is the typed configuration object for standalone bootstrap. Its providers array can contain ordinary providers and EnvironmentProviders returned by Angular’s provider functions. This lesson explains how to assemble that configuration, why Angular uses provideX functions, how to create reusable environment-provider helpers, and how existing NgModule providers can be bridged into a standalone application.

## Learning outcomes

After this lesson, you should be able to:

- Explain how bootstrapApplication creates the application’s root EnvironmentInjector.
- Build and pass an ApplicationConfig object.
- Register application-wide values and services in the root environment.
- Use provider functions such as provideRouter() and provideHttpClient().
- Explain the purpose of EnvironmentProviders and makeEnvironmentProviders().
- Build a typed reusable provideX() helper.
- Use provideAppInitializer() for work that must finish before application initialization completes.
- Import providers from a legacy NgModule when migrating.
- Choose between application, route, and component provider scopes.

## Where application providers live

In a standalone application, bootstrapApplication() renders a standalone root component and accepts an optional ApplicationConfig object. The providers in that configuration are registered in the root EnvironmentInjector. The root component and its descendants can inject them.

A minimal bootstrap call can provide services directly:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { LoggingService } from './app/logging.service';

bootstrapApplication(AppComponent, {
  providers: [LoggingService],
});
~~~

This works for a small application. As setup grows, put configuration in an app.config.ts file so the bootstrap entry point stays short and the application wiring is easy to review.

In current Angular, @Service() can register a service in the root EnvironmentInjector. Existing applications may use @Injectable({ providedIn: 'root' }) for the same broad scope. ApplicationConfig is especially useful when wiring framework features, runtime values, or providers that need explicit configuration.

## Define ApplicationConfig

ApplicationConfig is a stable type whose providers property accepts Provider and EnvironmentProviders entries. A reusable configuration file can then be passed to bootstrapApplication().

~~~ts
// src/app/app.config.ts
import type { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';
import { LoggingService } from './logging.service';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    LoggingService,
  ],
};
~~~

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig).catch((error: unknown) => {
  console.error(error);
});
~~~

The route table, router, HTTP client, and logging service are now configured at application scope. Components can inject services such as LoggingService, and HttpClient can be injected after provideHttpClient() is registered.

In an NgModule-based application, application providers are typically registered on the root NgModule instead. Do not add an NgModule solely to configure providers in a standalone application.

## Register values and implementations

Application providers can register a service class or map a token to a value, class, factory, or alias. Use an InjectionToken when the value is not represented by a runtime class, such as a configuration object or a TypeScript interface.

~~~ts
// src/app/runtime-config.ts
import { InjectionToken } from '@angular/core';

export interface RuntimeConfig {
  readonly apiBaseUrl: string;
  readonly buildLabel: string;
}

export const RUNTIME_CONFIG =
  new InjectionToken<RuntimeConfig>('runtime.config');
~~~

~~~ts
// src/app/app.config.ts
import type { ApplicationConfig } from '@angular/core';
import { RUNTIME_CONFIG } from './runtime-config';
import type { RuntimeConfig } from './runtime-config';

const runtimeConfig: RuntimeConfig = {
  apiBaseUrl: 'https://api.example.test',
  buildLabel: 'development',
};

export const appConfig: ApplicationConfig = {
  providers: [
    { provide: RUNTIME_CONFIG, useValue: runtimeConfig },
  ],
};
~~~

~~~ts
// Inject from a service or component created by Angular
import { inject, Service } from '@angular/core';
import { RUNTIME_CONFIG } from './runtime-config';

@Service()
export class CatalogApi {
  private readonly config = inject(RUNTIME_CONFIG);

  readonly baseUrl = this.config.apiBaseUrl;
}
~~~

The configuration object above is public application data. Do not place passwords, private API keys, or other secrets in browser-delivered configuration; users can inspect the compiled application.

Use useClass when selecting an implementation, useFactory when a value needs to be created from other providers, and useExisting when another token should point to the same instance. Use multi: true only for tokens designed to collect multiple registrations. Those provider forms use the same injector rules from the previous lesson.

## Provider functions

Angular feature packages expose functions such as provideRouter() and provideHttpClient(). They register the set of providers required by a feature and accept typed options or feature functions.

~~~ts
import {
  provideHttpClient,
  withInterceptors,
} from '@angular/common/http';
import { provideRouter, withInMemoryScrolling } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withInMemoryScrolling({
        anchorScrolling: 'enabled',
      }),
    ),
    provideHttpClient(
      withInterceptors([authInterceptor]),
    ),
  ],
};
~~~

Use the package’s provideX() function instead of copying its internal providers. A provider function keeps implementation details inside that package, gives configuration a typed API, and can compose optional features. Names and available feature functions depend on the installed Angular package version; check that package’s current API when configuring an app.

The same pattern appears in application configuration for hydration, animations, change detection, and other framework features. Only register the features the application actually needs.

## What EnvironmentProviders means

EnvironmentProviders is a wrapper type for providers that belong in an EnvironmentInjector. Angular uses it as a guard so environment-only providers are not accidentally added to a component’s providers array.

ApplicationConfig.providers accepts both Provider and EnvironmentProviders. Component providers are a different scope: they belong to an ElementInjector. If TypeScript reports that an EnvironmentProviders value is not valid in a component’s providers array, do not cast the type to suppress the error. Register it in application or other supported environment configuration.

A provider function commonly returns EnvironmentProviders. Treat that result as an opaque bundle: call the function and pass the result to the correct environment configuration point.

## Create a reusable provideX() helper

A library or application feature can export a function that groups its providers behind a small, typed configuration API. Use makeEnvironmentProviders() to mark the group as environment-only.

~~~ts
// src/app/search/search.providers.ts
import {
  InjectionToken,
  makeEnvironmentProviders,
  Service,
} from '@angular/core';

export interface SearchOptions {
  readonly endpoint: string;
  readonly maxResults: number;
}

export const SEARCH_OPTIONS =
  new InjectionToken<SearchOptions>('search.options');

@Service({ autoProvided: false })
export class SearchClient {
  // Inject SEARCH_OPTIONS when implementing the client.
}

export function provideSearch(options: SearchOptions) {
  return makeEnvironmentProviders([
    { provide: SEARCH_OPTIONS, useValue: options },
    SearchClient,
  ]);
}
~~~

A consuming application can configure the feature in one line:

~~~ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideSearch({
      endpoint: 'https://search.example.test',
      maxResults: 20,
    }),
  ],
};
~~~

A good provider helper owns the token and implementation details, accepts only the options consumers should control, documents its scope, and returns the provider bundle. Do not expose or duplicate private internal tokens just to make the helper configurable.

## Initialize application data before startup completes

Use provideAppInitializer() when the application must load or prepare data during startup. The initializer runs in an injection context. If it returns a Promise or Observable, Angular waits for that asynchronous work to finish before initialization completes.

~~~ts
// src/app/runtime-settings.service.ts
import { HttpClient } from '@angular/common/http';
import { inject, Service, signal } from '@angular/core';
import { firstValueFrom } from 'rxjs';
import type { RuntimeConfig } from './runtime-config';

@Service({ autoProvided: false })
export class RuntimeSettings {
  private readonly http = inject(HttpClient);
  readonly config = signal<RuntimeConfig | null>(null);

  async load(): Promise<void> {
    const config = await firstValueFrom(
      this.http.get<RuntimeConfig>('/assets/runtime-config.json'),
    );
    this.config.set(config);
  }
}
~~~

~~~ts
// Add to ApplicationConfig.providers
import { inject, provideAppInitializer } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';
import { RuntimeSettings } from './runtime-settings.service';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    RuntimeSettings,
    provideAppInitializer(() => inject(RuntimeSettings).load()),
  ],
};
~~~

The initializer and components resolve the same root-scoped RuntimeSettings instance. Once load() resolves, the runtime config is available from its signal. The file must be served as a public asset; it is not a secure place for secrets.

The older APP_INITIALIZER token is deprecated in current Angular; use provideAppInitializer() for new code. Keep startup work small and bounded: if an initializer never resolves or completes, application initialization will not finish.

## Bring providers from an NgModule during migration

importProvidersFrom() extracts providers from an NgModule and its imported modules so a standalone application can use them in an environment injector.

~~~ts
import { importProvidersFrom } from '@angular/core';
import { LegacyAnalyticsModule } from './legacy-analytics/legacy-analytics.module';

export const appConfig: ApplicationConfig = {
  providers: [
    importProvidersFrom(LegacyAnalyticsModule),
  ],
};
~~~

This is a migration bridge for provider configuration. It does not make NgModule-declared components available in a standalone component template; import standalone components directly or use the appropriate component-import mechanism. Do not use importProvidersFrom() to move component-scoped providers into the application root unintentionally.

## Application scope, route scope, and component scope

| Registration point | Injector scope | Typical purpose |
| --- | --- | --- |
| Root @Service() or ApplicationConfig.providers | Root EnvironmentInjector | App-wide services and framework setup |
| A route’s providers | Route EnvironmentInjector | Services needed by a route feature and its guards/resolvers |
| A component or directive’s providers | ElementInjector | State isolated to that component instance or subtree |

Application-level providers are available broadly and may be overridden by a nearer provider. Route providers belong to a feature environment. Component providers create an element-level instance. Choose based on who should share the value and which lifecycle should own it.

Chapter 053 covers component-scoped and route-scoped providers in detail.

## Common mistakes

### Registering an environment provider on a component

Functions such as provideRouter() return environment-level configuration. Put them in ApplicationConfig.providers or another supported environment configuration point, not in Component.providers.

### Treating ApplicationConfig as a component config

ApplicationConfig configures the root environment during bootstrap. Component metadata configures a component’s element injector. The two provider arrays have different scopes and accepted types.

### Copying framework internals into providers

Manually reproducing the internal router or HTTP provider list is brittle. Use the public provideRouter() or provideHttpClient() API and its documented feature functions.

### Putting everything in the root configuration

Root providers are appropriate for app-wide services and infrastructure. Component-specific state should remain component-scoped; feature-specific services may belong to a route. Overly broad scope can cause unintended shared state.

### Using importProvidersFrom() as a shortcut for every dependency

Use it when an existing NgModule provides behavior that is not yet migrated. It brings providers into an environment injector; it does not import declarations into a standalone component template.

### Treating browser configuration as secret

Values sent to the browser are visible to users. Use server-side APIs for private credentials, and keep client configuration limited to public settings.

## Practical checklist

Before adding application providers, check:

- Is this provider needed across the application or only within a feature/component?
- Does the package already offer a public provideX() function?
- Is the value a runtime class token or should it use an InjectionToken?
- Does an EnvironmentProviders result go into an environment injector?
- Is any environment configuration exposed to the browser safe to reveal?
- Does startup work need to block bootstrap, or can it load later?
- Is importProvidersFrom() temporarily bridging a legacy NgModule?

## Key takeaways

- bootstrapApplication() accepts ApplicationConfig for a standalone root component.
- ApplicationConfig.providers configures providers available to the root component and its descendants.
- EnvironmentProviders marks a provider bundle for environment injector configuration and guards against component-scope misuse.
- Use public provideX() functions for router, HTTP, and other framework features.
- makeEnvironmentProviders() is useful for reusable environment-level provider helpers.
- provideAppInitializer() runs startup work in an injection context and waits for a returned Promise or Observable.
- importProvidersFrom() can bridge NgModule providers into a standalone application’s environment injector.
- Root, route, and component scopes have different sharing and lifecycle behavior.

## Further reading

- [Defining dependency providers](https://angular.dev/guide/di/defining-dependency-providers)
- [Hierarchical injectors](https://angular.dev/guide/di/hierarchical-dependency-injection)
- [ApplicationConfig API](https://angular.dev/api/core/ApplicationConfig)
- [bootstrapApplication API](https://angular.dev/api/platform-browser/bootstrapApplication)
- [EnvironmentProviders API](https://angular.dev/api/core/EnvironmentProviders)
- [makeEnvironmentProviders API](https://angular.dev/api/core/makeEnvironmentProviders)
- [importProvidersFrom API](https://angular.dev/api/core/importProvidersFrom)
- [provideAppInitializer API](https://angular.dev/api/core/provideAppInitializer)
- [provideHttpClient API](https://angular.dev/api/common/http/provideHttpClient)
- [provideRouter API](https://angular.dev/api/router/provideRouter)

## Continue learning

- Previous: [Hierarchical dependency injection](051-hierarchical-dependency-injection.md)
- Q&A: [Application and environment providers](../qnas/052-application-and-environment-providers.md)
- Example: [Application and environment providers](../examples/052-application-and-environment-providers.md)
- Next: [Component-scoped and route-scoped providers](053-component-scoped-and-route-scoped-providers.md)
