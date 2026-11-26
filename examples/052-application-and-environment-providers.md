# Application and environment providers - example

This standalone Angular example shows how to configure application-level providers:

- Provide HttpClient and a runtime settings service in the root EnvironmentInjector.
- Use provideAppInitializer() to load public runtime configuration before the app finishes initialization.
- Package a configurable SearchClient behind a typed provideSearch() helper.
- Inject both services into the root component.

The code uses the current Angular v22 @Service() API. In older Angular projects, use the supported @Injectable() form for service classes and keep the same provider registration approach. The example expects the standard Angular CLI public directory.

## File layout

~~~text
public/
└── runtime-config.json
src/
├── main.ts
└── app/
    ├── app.config.ts
    ├── app.component.ts
    ├── app.component.html
    ├── runtime-config.ts
    ├── runtime-settings.service.ts
    └── search.providers.ts
~~~

## 1. Add public runtime configuration

This JSON file is served as a static asset. It can contain public settings such as a public API base URL or a display name. Never put private credentials in a file delivered to the browser.

~~~json
// public/runtime-config.json
{
  "appName": "Angular Environment Provider Demo",
  "publicApiBaseUrl": "https://api.example.test"
}
~~~

## 2. Define the runtime config type

~~~ts
// src/app/runtime-config.ts
export interface RuntimeConfig {
  readonly appName: string;
  readonly publicApiBaseUrl: string;
}
~~~

## 3. Create a service to load and hold runtime settings

The service uses HttpClient to fetch the JSON file and stores the response in a signal. It is marked autoProvided: false because app.config.ts will explicitly register the service in the root application environment.

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
      this.http.get<RuntimeConfig>('/runtime-config.json'),
    );
    this.config.set(config);
  }
}
~~~

## 4. Create a reusable environment provider

SearchOptions and its token define the public configuration for a small SearchClient. provideSearch() returns an EnvironmentProviders bundle so the service and its options are registered together at environment scope.

~~~ts
// src/app/search.providers.ts
import {
  InjectionToken,
  makeEnvironmentProviders,
  Service,
  inject,
} from '@angular/core';
import type { EnvironmentProviders } from '@angular/core';

export interface SearchOptions {
  readonly endpoint: string;
  readonly maxResults: number;
}

const SEARCH_OPTIONS =
  new InjectionToken<SearchOptions>('search.options');

@Service({ autoProvided: false })
export class SearchClient {
  private readonly options = inject(SEARCH_OPTIONS);

  readonly endpoint = this.options.endpoint;
  readonly maxResults = this.options.maxResults;
}

export function provideSearch(
  options: SearchOptions,
): EnvironmentProviders {
  return makeEnvironmentProviders([
    { provide: SEARCH_OPTIONS, useValue: options },
    SearchClient,
  ]);
}
~~~

SEARCH_OPTIONS is intentionally private to this file. Application code configures the feature through provideSearch() and injects SearchClient instead of depending on the token’s implementation details.

## 5. Assemble the root ApplicationConfig

The initializer injects RuntimeSettings and returns its Promise. Angular waits for that Promise before application initialization completes. The same RuntimeSettings provider is later injected by AppComponent.

~~~ts
// src/app/app.config.ts
import {
  inject,
  provideAppInitializer,
} from '@angular/core';
import type { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';
import { provideSearch } from './search.providers';
import { RuntimeSettings } from './runtime-settings.service';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    RuntimeSettings,
    provideSearch({
      endpoint: 'https://search.example.test',
      maxResults: 10,
    }),
    provideAppInitializer(() => inject(RuntimeSettings).load()),
  ],
};
~~~

The endpoint and result limit above are public application settings. A different build or deployment can supply different public values. Do not use this mechanism to hide credentials.

## 6. Display the configured services

The root component injects RuntimeSettings and SearchClient from the root EnvironmentInjector. The control-flow block shows a loading message until the initializer has populated the runtime config.

~~~ts
// src/app/app.component.ts
import { Component, inject } from '@angular/core';
import { RuntimeSettings } from './runtime-settings.service';
import { SearchClient } from './search.providers';

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
})
export class AppComponent {
  readonly settings = inject(RuntimeSettings);
  readonly search = inject(SearchClient);
}
~~~

~~~html
<!-- src/app/app.component.html -->
<main>
  <h1>Application and environment providers</h1>

  @if (settings.config(); as config) {
    <section>
      <h2>{{ config.appName }}</h2>
      <p>Public API: {{ config.publicApiBaseUrl }}</p>
    </section>
  } @else {
    <p>Loading runtime configuration…</p>
  }

  <section>
    <h2>Search feature</h2>
    <p>Endpoint: {{ search.endpoint }}</p>
    <p>Maximum results: {{ search.maxResults }}</p>
  </section>
</main>
~~~

## 7. Bootstrap the standalone application

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig).catch((error: unknown) => {
  console.error('Application bootstrap failed', error);
});
~~~

## 8. Run it

Use an Angular standalone application, add the files above, and run:

~~~sh
ng serve
~~~

Open the local URL printed by Angular CLI. If your project serves static assets from a different directory, put runtime-config.json there and update the URL in RuntimeSettings.

## Expected behavior

The page should show:

- Angular Environment Provider Demo and the public API URL loaded from runtime-config.json.
- The search endpoint and maximum result count supplied through provideSearch().
- No loading message once the initializer has completed.

If the JSON request fails, the initializer’s Promise rejects and bootstrap fails. In a production app, decide deliberately how to handle unavailable configuration: show a meaningful startup error, provide a safe fallback, or retry according to the application’s requirements.

## What happens in the injector

| Token or service | Registration | Scope |
| --- | --- | --- |
| HttpClient | provideHttpClient() in app.config.ts | Root EnvironmentInjector |
| RuntimeSettings | app.config.ts providers array | Root EnvironmentInjector |
| SearchOptions | Returned by provideSearch() | Root EnvironmentInjector |
| SearchClient | Returned by provideSearch() | Root EnvironmentInjector |

Both AppComponent and the initializer resolve the same root RuntimeSettings instance. SearchClient receives the options registered by the helper. None of these environment providers belongs in a component’s providers array.

## Optional: bridge a legacy NgModule

If a still-needed library exposes providers only through an NgModule, import them into the environment configuration:

~~~ts
import { importProvidersFrom } from '@angular/core';
import { LegacyAnalyticsModule } from './legacy-analytics/legacy-analytics.module';

export const appConfig: ApplicationConfig = {
  providers: [
    importProvidersFrom(LegacyAnalyticsModule),
    provideHttpClient(),
    RuntimeSettings,
    provideSearch({
      endpoint: 'https://search.example.test',
      maxResults: 10,
    }),
    provideAppInitializer(() => inject(RuntimeSettings).load()),
  ],
};
~~~

This is a provider migration bridge. It does not make the NgModule’s components or directives available in AppComponent’s template.

## Optional: configure router providers

A routed application can add provideRouter() to the same root configuration. Keep feature-specific route providers in the route configuration; chapter 053 explains their scope.

~~~ts
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
  ],
};
~~~

The snippets in the optional sections show how to extend the app configuration; do not paste multiple declarations of appConfig into the same file.

## Common adjustments

- If every feature should share SearchClient, keep provideSearch() in ApplicationConfig.providers.
- If search configuration belongs only to a route, configure that feature in the route’s environment instead; see chapter 053.
- If a local component needs a separate service instance, provide that class on the component instead of creating another root provider.
- If runtime configuration is optional, decide what defaults and error behavior are safe before making the initializer non-blocking.
- Keep public runtime configuration free of secrets.
- Use a distinct InjectionToken object for each independent configuration contract.

## Related files

- Lesson: [Application and environment providers](../lessons/052-application-and-environment-providers.md)
- Q&A: [Application and environment providers](../qnas/052-application-and-environment-providers.md)
- Next lesson: [Component-scoped and route-scoped providers](../lessons/053-component-scoped-and-route-scoped-providers.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="051-hierarchical-dependency-injection.md">051 - Hierarchical dependency injection</a></td><td><a href="../lessons/052-application-and-environment-providers.md">Lesson</a></td><td><a href="../qnas/052-application-and-environment-providers.md">Q&amp;A</a></td><td><a href="053-component-scoped-and-route-scoped-providers.md">053 - Component-scoped and route-scoped providers</a></td>
</tr>
  </tbody>
</table>
