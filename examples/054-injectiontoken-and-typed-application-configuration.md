# InjectionToken and typed application configuration - example

This standalone app uses a typed InjectionToken to provide public search settings, then packages the token and a SearchClient into a reusable provideSearch() helper. The app component injects both and builds a URL from the configured API origin.

The helper returns EnvironmentProviders, so it can be registered in application or route configuration. This example registers it at the application level. The route-scoped alternative is shown near the end.

## Create the project

Create a standalone Angular project:

~~~sh
ng new angular-injection-token-demo --standalone --style=css
cd angular-injection-token-demo
~~~

The example targets Angular v22.1 and newer because it uses the Service decorator. For projects using Injectable, use Injectable() without providedIn for the manually provided SearchClient.

## File layout

~~~text
src/
├── main.ts
└── app/
    ├── app.component.ts
    ├── app.component.html
    ├── app.config.ts
    └── search/
        ├── search-config.ts
        ├── search-settings.ts
        ├── search-client.ts
        └── search.providers.ts
~~~

## 1. Define a typed configuration contract and token

~~~ts
// src/app/search/search-config.ts
import { InjectionToken } from '@angular/core';

export interface SearchConfig {
  readonly appName: string;
  readonly apiOrigin: string;
  readonly requestTimeoutMs: number;
  readonly features: {
    readonly suggestions: boolean;
    readonly advancedFilters: boolean;
  };
}

export const SEARCH_CONFIG =
  new InjectionToken<SearchConfig>('SEARCH_CONFIG');
~~~

SearchConfig describes the object at compile time. SEARCH_CONFIG is the runtime key Angular uses for dependency injection. Export and reuse this same token instance wherever it is provided or injected.

## 2. Create public application settings

~~~ts
// src/app/search/search-settings.ts
import type { SearchConfig } from './search-config';

export const searchSettings = {
  appName: 'Typed Search Demo',
  apiOrigin: 'https://api.example.test',
  requestTimeoutMs: 8000,
  features: {
    suggestions: true,
    advancedFilters: false,
  },
} satisfies SearchConfig;
~~~

The satisfies clause checks the configuration shape and field types. It does not validate external JSON at runtime. These settings are bundled into the browser app, so they must not contain credentials or private secrets.

## 3. Create a service that consumes the token

~~~ts
// src/app/search/search-client.ts
import { inject, Service } from '@angular/core';
import { SEARCH_CONFIG } from './search-config';

@Service({ autoProvided: false })
export class SearchClient {
  private readonly config = inject(SEARCH_CONFIG);

  get requestTimeoutMs(): number {
    return this.config.requestTimeoutMs;
  }

  buildSearchUrl(query: string): URL {
    const url = new URL('/search', this.config.apiOrigin);
    url.searchParams.set('q', query);
    return url;
  }
}
~~~

The service reads its settings through DI. It does not import the concrete application settings file, so the implementation can be reused with a different typed configuration.

## 4. Package the token and service in an environment provider helper

~~~ts
// src/app/search/search.providers.ts
import { makeEnvironmentProviders } from '@angular/core';
import type { EnvironmentProviders } from '@angular/core';
import { SEARCH_CONFIG } from './search-config';
import { SearchClient } from './search-client';
import type { SearchConfig } from './search-config';

export function provideSearch(
  config: SearchConfig,
): EnvironmentProviders {
  return makeEnvironmentProviders([
    { provide: SEARCH_CONFIG, useValue: config },
    SearchClient,
  ]);
}
~~~

The helper imports the same SEARCH_CONFIG instance that SearchClient injects. makeEnvironmentProviders marks the bundle for environment injectors and helps prevent it from being placed in a component's providers array by mistake.

## 5. Register the helper in ApplicationConfig

~~~ts
// src/app/app.config.ts
import type { ApplicationConfig } from '@angular/core';
import { provideSearch } from './search/search.providers';
import { searchSettings } from './search/search-settings';

export const appConfig: ApplicationConfig = {
  providers: [
    provideSearch(searchSettings),
  ],
};
~~~

ApplicationConfig.providers creates app-wide configuration for this standalone application. The provider helper keeps the setup in one named call rather than exposing the internal token and service wiring at every call site.

## 6. Inject the settings and service in the root component

~~~ts
// src/app/app.component.ts
import { Component, inject, signal } from '@angular/core';
import { SEARCH_CONFIG } from './search/search-config';
import { SearchClient } from './search/search-client';

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
})
export class AppComponent {
  protected readonly settings = inject(SEARCH_CONFIG);
  protected readonly searchClient = inject(SearchClient);
  protected readonly searchUrl = signal('');

  protected buildUrl(query: string): void {
    this.searchUrl.set(
      this.searchClient.buildSearchUrl(query).toString(),
    );
  }
}
~~~

~~~html
<!-- src/app/app.component.html -->
<main>
  <h1>{{ settings.appName }}</h1>
  <p>API origin: <code>{{ settings.apiOrigin }}</code></p>
  <p>Configured request timeout: {{ searchClient.requestTimeoutMs }} ms</p>
  <p>
    Suggestions:
    {{ settings.features.suggestions ? 'enabled' : 'disabled' }}
  </p>
  <p>
    Advanced filters:
    {{ settings.features.advancedFilters ? 'enabled' : 'disabled' }}
  </p>

  <label for="search-query">Search query</label>
  <input #query id="search-query" type="search" />
  <button type="button" (click)="buildUrl(query.value)">
    Build search URL
  </button>

  @if (searchUrl()) {
    <p>Generated URL: <code>{{ searchUrl() }}</code></p>
  }
</main>
~~~

The app displays values from the injected configuration. Enter a query and select **Build search URL** to see the SearchClient use the configured API origin. The example builds a URL only; it does not send a network request.

## 7. Bootstrap the standalone application

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig).catch((error: unknown) => {
  console.error(error);
});
~~~

## 8. Run the app

~~~sh
ng serve
~~~

Open the local address printed by the Angular CLI. The app name, API origin, timeout, and feature flags come from the single typed settings object.

## Optional: register the helper for one route feature

If only one route subtree needs SearchClient, add the helper to that route's providers instead of the root ApplicationConfig:

~~~ts
const routes: Routes = [
  {
    path: 'catalog',
    providers: [
      provideSearch(catalogSearchSettings),
    ],
    loadComponent: () =>
      import('./catalog.page').then((module) => module.CatalogPage),
  },
];
~~~

The same token and helper can be used in an environment scope. This route's configuration is available to its route content and child routes. See [component-scoped and route-scoped providers](../lessons/053-component-scoped-and-route-scoped-providers.md) for the injector behavior.

## What this example demonstrates

| Code | Purpose |
| --- | --- |
| SearchConfig | Compile-time shape for application settings |
| SEARCH_CONFIG | Runtime DI token with the SearchConfig type |
| satisfies SearchConfig | Compile-time check that the settings have the required fields |
| provideSearch(config) | Reusable function that registers the token and service |
| makeEnvironmentProviders() | Marks the provider bundle for environment injector configuration |
| ApplicationConfig.providers | Makes the settings and service available to the application |

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="053-component-scoped-and-route-scoped-providers.md">053 - Component-scoped and route-scoped providers</a></td><td><a href="../lessons/054-injectiontoken-and-typed-application-configuration.md">Lesson</a></td><td><a href="../qnas/054-injectiontoken-and-typed-application-configuration.md">Q&amp;A</a></td><td><a href="055-provider-recipes-multi-providers-and-provider-functions.md">055 - Provider recipes, multi providers, and provider functions</a></td>
</tr>
  </tbody>
</table>
