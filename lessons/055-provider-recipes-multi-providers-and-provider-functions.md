# Provider recipes, multi providers, and provider functions

Angular providers map a runtime token to the value Angular should inject. A class shorthand is convenient for the default case, but applications and libraries often need to select an implementation, supply a configuration value, construct a dependency from other providers, alias an old token, or collect contributions from multiple features.

This lesson compares the main provider recipes, explains multi providers, and shows how a typed provideX() function can package a feature's provider setup behind a small public API.

## Learning outcomes

After this lesson, you should be able to:

- Choose between class shorthand, useClass, useValue, useFactory, and useExisting.
- Explain why useExisting preserves an instance while useClass constructs an implementation.
- Register a typed multi-provider token and inject the resulting array.
- Avoid mixing regular and multi providers for the same token in one injector.
- Build a public provider function with typed options.
- Use makeEnvironmentProviders when a helper belongs in an environment injector.
- Compose optional feature behavior with withX() functions.

## What a provider declares

A provider has an identifier and a recipe for obtaining its value. The identifier is usually a class or InjectionToken. The recipe may be a class, a value, a factory, or an alias.

For a class that provides itself, use the concise class form:

~~~ts
providers: [SearchClient]
~~~

It is equivalent to:

~~~ts
providers: [
  { provide: SearchClient, useClass: SearchClient },
]
~~~

Use the shorthand when the class token is the right identifier and the default class is the implementation.

## useClass: choose an implementation

Use useClass when consumers inject an abstraction but the injector should create a particular class.

~~~ts
export abstract class AppLogger {
  abstract log(message: string): void;
}

@Service({ autoProvided: false })
export class ConsoleLogger extends AppLogger {
  log(message: string): void {
    console.log(message);
  }
}

providers: [
  { provide: AppLogger, useClass: ConsoleLogger },
]
~~~

A test or another application configuration can use a different class for the same token:

~~~ts
providers: [
  { provide: AppLogger, useClass: SilentLogger },
]
~~~

Providing a class is not the same as aliasing an existing token. Two class providers can create two instances even if they use the same implementation class.

## useValue: supply an existing value

Use useValue for constants, configuration objects, functions, or test doubles that have already been created.

~~~ts
export const APP_LABEL =
  new InjectionToken<string>('APP_LABEL');

providers: [
  { provide: APP_LABEL, useValue: 'Customer portal' },
]
~~~

For structured configuration, pair the token with an interface and validate untrusted runtime data before providing it. See [InjectionToken and typed application configuration](054-injectiontoken-and-typed-application-configuration.md).

A useValue provider returns the supplied value. Angular does not clone or freeze it.

## useFactory: construct a value from dependencies

Use useFactory when a dependency must be derived or constructed with runtime configuration.

~~~ts
export const SEARCH_ENDPOINT =
  new InjectionToken<URL>('SEARCH_ENDPOINT');

providers: [
  {
    provide: SEARCH_ENDPOINT,
    useFactory: () => {
      const settings = inject(API_SETTINGS);
      return new URL('/search', settings.apiOrigin);
    },
  },
]
~~~

Angular invokes the factory in an injection context, so the factory can call inject(). This keeps the dependency lookup next to the factory that needs it.

A factory can also declare dependencies with the deps array:

~~~ts
{
  provide: SEARCH_ENDPOINT,
  useFactory: (settings: ApiSettings) =>
    new URL('/search', settings.apiOrigin),
  deps: [API_SETTINGS],
}
~~~

Use one style consistently within a codebase. The deps form is useful when dependencies should be explicit in provider metadata; inject() is concise and convenient in current standalone applications.

A factory normally runs when the injector first needs that token, then the injector caches its result for that provider record. A factory is not automatically a per-call function. If callers need repeated work, inject a service with a method that performs that work.

## useExisting: alias a token to an existing provider

Use useExisting when two tokens should resolve to the same instance.

~~~ts
providers: [
  ConsoleLogger,
  { provide: AppLogger, useExisting: ConsoleLogger },
]
~~~

Injecting ConsoleLogger and AppLogger now returns the instance registered as ConsoleLogger.

Do not substitute useClass here:

~~~ts
providers: [
  ConsoleLogger,
  { provide: AppLogger, useClass: ConsoleLogger },
]
~~~

This can construct one instance for ConsoleLogger and another for AppLogger. useExisting performs an alias lookup; useClass configures a class to instantiate for that token.

Aliases are useful during API migrations when new code uses a new token while legacy callers still inject the old token.

## Multi providers collect contributions

A multi provider lets several provider records contribute values for one token in the same injector. Angular returns those values as an array.

~~~ts
export type StartupTask = () => void;

export const STARTUP_TASKS =
  new InjectionToken<readonly StartupTask[]>('STARTUP_TASKS');

providers: [
  {
    provide: STARTUP_TASKS,
    useValue: initializeCache,
    multi: true,
  },
  {
    provide: STARTUP_TASKS,
    useValue: initializeTelemetry,
    multi: true,
  },
]
~~~

A consumer can inject the array and run each task:

~~~ts
@Service({ autoProvided: false })
export class StartupRunner {
  private readonly tasks = inject(STARTUP_TASKS);

  run(): void {
    for (const task of this.tasks) {
      task();
    }
  }
}
~~~

The token's type should describe the collection returned by injection, such as readonly StartupTask[]. Each provider contributes one value; Angular assembles the array.

### Keep registrations consistent

All records for a multi token in a given injector should use multi: true. Do not combine a multi registration and a regular single-value registration for the same token in that injector. That setup has incompatible shapes and can produce an invalid multi-provider error.

Multi providers contribute within an injector. Dependency resolution still follows the injector hierarchy: a closer provider for the token can shadow an ancestor registration. Do not assume that arrays automatically merge across root, route, and component scopes.

Multi providers are useful for plugin lists, validation rules, startup hooks, or other extension points where multiple independent packages contribute values to a shared token. Use a normal array in one useValue when one owner already has the entire collection.

## Provider functions define a public configuration API

A library can export a provideX() function instead of asking every application to know its internal tokens and provider records.

~~~ts
import { makeEnvironmentProviders } from '@angular/core';
import type {
  EnvironmentProviders,
  Provider,
} from '@angular/core';

export interface AuditOptions {
  readonly label: string;
  readonly includeTime?: boolean;
}

export function provideAudit(
  options: AuditOptions,
): EnvironmentProviders {
  const providers: Provider[] = [
    { provide: AUDIT_OPTIONS, useValue: options },
    AuditService,
  ];

  return makeEnvironmentProviders(providers);
}
~~~

The application uses the feature through its public API:

~~~ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(),
    provideAudit({
      label: 'Storefront',
      includeTime: true,
    }),
  ],
};
~~~

A well-designed provider function:

- Accepts typed, documented options.
- Hides private tokens and internal implementation classes.
- Registers every provider the feature needs.
- Returns EnvironmentProviders when it is designed for application or route environments.
- Documents the scope and any optional capabilities it enables.

The place where the caller registers the returned value determines its environment scope. ApplicationConfig.providers makes the feature available from the root environment; Route.providers scopes it to that route and its children.

## Compose optional features with withX functions

A provideX function can accept feature options returned by withX functions. Each option describes a focused addition without exposing the feature's internal provider records.

~~~ts
export interface AuditFeature {
  readonly providers: Provider[];
}

export function withConsoleOutput(): AuditFeature {
  return {
    providers: [
      ConsoleAuditSink,
      { provide: AUDIT_SINK, useExisting: ConsoleAuditSink },
    ],
  };
}

export function withRemoteOutput(
  endpoint: URL,
): AuditFeature {
  return {
    providers: [
      { provide: AUDIT_ENDPOINT, useValue: endpoint },
      RemoteAuditSink,
      { provide: AUDIT_SINK, useExisting: RemoteAuditSink },
    ],
  };
}

export function provideAudit(
  options: AuditOptions,
  ...features: AuditFeature[]
): EnvironmentProviders {
  return makeEnvironmentProviders([
    { provide: AUDIT_OPTIONS, useValue: options },
    AuditService,
    ...features.flatMap((feature) => feature.providers),
  ]);
}
~~~

A caller can choose the feature set in one place:

~~~ts
provideAudit(
  { label: 'Storefront' },
  withConsoleOutput(),
)
~~~

Design feature options so invalid or contradictory combinations are difficult to express. For example, two different output sinks might be mutually exclusive; represent that choice as one typed option or a discriminated union instead of accepting both silently.

Do not recreate Angular's internal provider arrays in application code. Use public functions such as provideRouter() and provideHttpClient() when configuring Angular features, and use your library's public provideX API for library features.

## Choose a recipe

| Need | Provider recipe |
| --- | --- |
| A class provides itself under its own token | Class shorthand or useClass |
| Select an implementation for an abstraction | useClass |
| Supply an existing constant, config object, or test double | useValue |
| Build a value using runtime options or other dependencies | useFactory |
| Make two tokens resolve to the exact same instance | useExisting |
| Collect values from independent registrations | multi: true |
| Package a feature's public setup | A typed provideX() function |
| Add optional capabilities to that setup | Typed withX() feature functions |

## Common mistakes

- **Using useClass to create an alias:** useExisting is the recipe that shares the exact existing instance.
- **Assuming a factory runs every time the token is injected:** injector results are cached for the provider record. Put repeated work in a method instead.
- **Mixing multi and regular registrations for one token in the same injector:** use the same multi setting for every registration of that token in that injector.
- **Typing a multi token as one item:** injected values are an array, so the token type should describe the array.
- **Expecting multi values to merge across injector levels:** normal hierarchy lookup still selects a nearer provider. Design the extension point at one intended scope.
- **Returning raw providers from an environment-only helper:** return makeEnvironmentProviders(...) with an EnvironmentProviders type to communicate and enforce the intended context.
- **Exporting internal tokens unnecessarily:** keep implementation details private and expose a typed configuration function.
- **Allowing contradictory feature options:** constrain combinations with the API shape and validate options when needed.

## Continue learning

- Previous: [InjectionToken and typed application configuration](054-injectiontoken-and-typed-application-configuration.md)
- Q&A: [Provider recipes, multi providers, and provider functions](../qnas/055-provider-recipes-multi-providers-and-provider-functions.md)
- Example: [Provider recipes, multi providers, and provider functions](../examples/055-provider-recipes-multi-providers-and-provider-functions.md)
- Next: [Injection contexts and resource lifetimes](056-injection-contexts-and-resource-lifetimes.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="054-injectiontoken-and-typed-application-configuration.md">054 - InjectionToken and typed application configuration</a></td><td><a href="../qnas/055-provider-recipes-multi-providers-and-provider-functions.md">Q&amp;A</a></td><td><a href="../examples/055-provider-recipes-multi-providers-and-provider-functions.md">Example</a></td><td><a href="056-injection-contexts-and-resource-lifetimes.md">056 - Injection contexts and resource lifetimes</a></td>
</tr>
  </tbody>
</table>
