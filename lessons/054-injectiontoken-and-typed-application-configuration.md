# InjectionToken and typed application configuration

TypeScript interfaces, object types, strings, functions, and generic types are useful descriptions, but most of them do not exist at runtime. Angular dependency injection needs a runtime key that a provider and an injection request can share. InjectionToken supplies that key and connects it to a TypeScript type.

This lesson shows how to define tokens for configuration and other non-class values, provide typed values during application setup, create a default token factory, and build a reusable environment-provider helper. It also explains token identity, runtime validation, and safe handling of configuration delivered to the browser.

## Learning outcomes

After this lesson, you should be able to:

- Explain why an interface cannot be used directly as a DI token.
- Create and export a typed InjectionToken.
- Provide a value with useValue and inject it with the correct TypeScript type.
- Explain why token identity matters more than its description string.
- Use a token factory for a root-level default.
- Build a typed provider function using makeEnvironmentProviders.
- Distinguish compile-time typing from runtime validation.

## Why Angular needs an InjectionToken

Angular can use a class as a token because the class exists at runtime. An interface is erased when TypeScript compiles to JavaScript, so Angular cannot use that interface as a runtime lookup key.

This will not work as intended:

~~~ts
export interface ApiSettings {
  readonly apiOrigin: string;
}

// Interfaces do not exist at runtime.
// Angular cannot resolve ApiSettings as a DI token.
~~~

Create an InjectionToken for the interface instead:

~~~ts
import { InjectionToken } from '@angular/core';

export interface ApiSettings {
  readonly apiOrigin: string;
  readonly requestTimeoutMs: number;
}

export const API_SETTINGS =
  new InjectionToken<ApiSettings>('API_SETTINGS');
~~~

The generic type parameter connects the runtime token to the expected TypeScript value. Calling inject(API_SETTINGS) returns an ApiSettings value to TypeScript-aware code.

The description string is for debugging. It is not the identity of the token. Angular matches the exact InjectionToken object instance.

~~~ts
// api-settings.token.ts
export const API_SETTINGS =
  new InjectionToken<ApiSettings>('API_SETTINGS');
~~~

Every provider and consumer should import this exported instance. Creating another InjectionToken with the same type and description creates a different key and will not match the original provider.

## Provide typed application configuration

An application-wide settings object can be registered in ApplicationConfig.providers. Give the object an explicit type or use satisfies so TypeScript checks its shape.

~~~ts
// src/app/api-settings.ts
import type { ApiSettings } from './api-settings.token';

export const apiSettings = {
  apiOrigin: 'https://api.example.test',
  requestTimeoutMs: 8000,
} satisfies ApiSettings;
~~~

~~~ts
// src/app/app.config.ts
import type { ApplicationConfig } from '@angular/core';
import { API_SETTINGS } from './api-settings.token';
import { apiSettings } from './api-settings';

export const appConfig: ApplicationConfig = {
  providers: [
    { provide: API_SETTINGS, useValue: apiSettings },
  ],
};
~~~

~~~ts
// A component or service
import { inject } from '@angular/core';
import { API_SETTINGS } from './api-settings.token';

export class SearchClient {
  private readonly settings = inject(API_SETTINGS);

  buildEndpoint(path: string): URL {
    return new URL(path, this.settings.apiOrigin);
  }
}
~~~

The token types the injection point. The satisfies clause checks that the configured object supplies the required fields while preserving its inferred type. A plain annotation is also valid:

~~~ts
const apiSettings: ApiSettings = {
  apiOrigin: 'https://api.example.test',
  requestTimeoutMs: 8000,
};
~~~

Use readonly properties for configuration that consumers should read but not mutate. This communicates intent to TypeScript; readonly does not freeze the object at runtime.

## Token identity and provider matching

Angular compares InjectionToken objects by identity. The description helps identify a token in errors, but two tokens with the same description are not interchangeable.

~~~ts
const FIRST_SETTINGS =
  new InjectionToken<ApiSettings>('API_SETTINGS');
const SECOND_SETTINGS =
  new InjectionToken<ApiSettings>('API_SETTINGS');

// FIRST_SETTINGS and SECOND_SETTINGS are different runtime keys.
~~~

Keep each token in a shared module and import it from both the provider and consumer. Do not construct tokens inside functions that can run more than once.

## Use a token factory for a default value

An InjectionToken may define a factory:

~~~ts
export const DEFAULT_LANGUAGE =
  new InjectionToken<string>('DEFAULT_LANGUAGE', {
    factory: () => 'en',
  });
~~~

A token with a factory is provided in the root environment by default, so it does not need a separate provider entry. A factory can inject other dependencies because Angular evaluates it in an injection context.

~~~ts
import { inject, InjectionToken } from '@angular/core';
import { API_SETTINGS } from './api-settings.token';

export const API_ORIGIN =
  new InjectionToken<string>('API_ORIGIN', {
    factory: () => inject(API_SETTINGS).apiOrigin,
  });
~~~

Prefer a token factory for a stable default that should be available throughout the root environment. Use an explicit provider when the application's bootstrap or feature configuration must choose the value.

Do not assume a root token factory will read a route-local override. The factory belongs to the token's root provider; a factory that needs route-specific configuration should itself be provided in that route's environment.

## Provide an environment-scoped configuration helper

A library or feature can export a function that accepts typed options and returns providers. This hides the internal token and service wiring from application bootstrap.

~~~ts
// src/app/search.providers.ts
import {
  EnvironmentProviders,
  InjectionToken,
  makeEnvironmentProviders,
  Service,
  inject,
} from '@angular/core';

export interface SearchConfig {
  readonly apiOrigin: string;
  readonly requestTimeoutMs: number;
}

const SEARCH_CONFIG =
  new InjectionToken<SearchConfig>('SEARCH_CONFIG');

@Service({ autoProvided: false })
class SearchClient {
  private readonly config = inject(SEARCH_CONFIG);

  buildUrl(query: string): URL {
    const url = new URL('/search', this.config.apiOrigin);
    url.searchParams.set('q', query);
    return url;
  }
}

export function provideSearch(
  config: SearchConfig,
): EnvironmentProviders {
  return makeEnvironmentProviders([
    { provide: SEARCH_CONFIG, useValue: config },
    SearchClient,
  ]);
}
~~~

~~~ts
// src/app/app.config.ts
import type { ApplicationConfig } from '@angular/core';
import { provideSearch } from './search.providers';

export const appConfig: ApplicationConfig = {
  providers: [
    provideSearch({
      apiOrigin: 'https://api.example.test',
      requestTimeoutMs: 8000,
    }),
  ],
};
~~~

The helper's parameter is checked against SearchConfig. The EnvironmentProviders return type communicates that the bundle belongs in an environment injector, such as ApplicationConfig.providers or Route.providers. makeEnvironmentProviders also helps prevent accidentally placing the bundle in a component's providers array.

The location where the helper is registered determines its scope:

- Register it in ApplicationConfig.providers when the whole application should share the configuration and service.
- Register it in Route.providers when only that route feature and its children should use it.

This is one form of Angular's provideX configuration pattern. Chapter 055 covers provider recipes and feature options in more detail.

## Choose between useValue, a token factory, and useFactory

| Pattern | Use it when | Where the value comes from |
| --- | --- | --- |
| InjectionToken with useValue | The app or feature already has a configuration value | A value assembled by the caller |
| InjectionToken factory | A token has a safe default that can be derived in an injection context | A token-owned factory, root-provided by default |
| Provider with useFactory | The caller needs explicit wiring for how a dependency is constructed | A registered factory provider |

A provider factory can inject dependencies in its factory body:

~~~ts
{
  provide: SEARCH_ENDPOINT,
  useFactory: () => {
    const config = inject(API_SETTINGS);
    return new URL('/search', config.apiOrigin);
  },
}
~~~

For most typed configuration objects, useValue is easiest to read. Use a factory when the value must be derived from other providers or created at runtime.

## Compile-time types are not runtime validation

A type annotation, interface, generic, or satisfies expression is removed or erased from the JavaScript output. It cannot validate data received from a server, a JSON file, local storage, a URL, or another untrusted runtime source.

Treat external data as unknown and validate it before providing it:

~~~ts
const rawConfig: unknown = await response.json();
const validatedConfig = parseApiSettings(rawConfig);

providers: [
  { provide: API_SETTINGS, useValue: validatedConfig },
]
~~~

The parseApiSettings function should check the actual runtime structure and reject invalid values. A cast such as rawConfig as ApiSettings only suppresses a compiler warning; it does not validate anything.

Values shipped to the browser are public. Never place passwords, private API keys, signing secrets, or credentials in a token, build-time environment file, HTML, or downloaded runtime configuration.

## Testing and replacing configuration

A token makes configuration easy to replace in a test:

~~~ts
TestBed.configureTestingModule({
  providers: [
    {
      provide: API_SETTINGS,
      useValue: {
        apiOrigin: 'https://api.test',
        requestTimeoutMs: 10,
      } satisfies ApiSettings,
    },
  ],
});
~~~

Use the same exported token that production code injects. A test-only token with the same description will not override the production token.

ApplicationConfig and route configuration can also register a different provider for the same token at different scopes. As covered in [component-scoped and route-scoped providers](053-component-scoped-and-route-scoped-providers.md), the nearest matching provider wins.

## Common mistakes

- **Injecting an interface directly:** interfaces do not exist at runtime. Create an InjectionToken for the interface type.
- **Creating two tokens with the same description:** Angular matches token object identity, not the description string.
- **Defining a token but never providing it:** create a provider or define a token factory that supplies a default.
- **Assuming the generic validates data:** TypeScript types do not validate JSON or runtime values.
- **Using a type assertion as validation:** a cast changes what TypeScript believes, not the value received at runtime.
- **Leaking secrets into browser configuration:** anything delivered to client code can be inspected by users.
- **Returning Provider[] from an environment-only helper:** prefer makeEnvironmentProviders and EnvironmentProviders when a helper is meant for application or route injectors.
- **Putting a token factory at root for route-specific configuration:** root factories resolve in the root injector context. Register an explicit route-level provider if the value varies by route.

## Further reading

- [InjectionToken API](https://angular.dev/api/core/InjectionToken)
- [Defining dependency providers](https://angular.dev/guide/di/defining-dependency-providers)
- [Injection context](https://angular.dev/guide/di/dependency-injection-context)
- [makeEnvironmentProviders API](https://angular.dev/api/core/makeEnvironmentProviders)
- [ApplicationConfig API](https://angular.dev/api/core/ApplicationConfig)
- [bootstrapApplication API](https://angular.dev/api/platform-browser/bootstrapApplication)
- [Hierarchical dependency injection](https://angular.dev/guide/di/hierarchical-dependency-injection)

## Continue learning

- Previous: [Component-scoped and route-scoped providers](053-component-scoped-and-route-scoped-providers.md)
- Q&A: [InjectionToken and typed application configuration](../qnas/054-injectiontoken-and-typed-application-configuration.md)
- Example: [InjectionToken and typed application configuration](../examples/054-injectiontoken-and-typed-application-configuration.md)
- Next: [Provider recipes, multi providers, and provider functions](055-provider-recipes-multi-providers-and-provider-functions.md)
