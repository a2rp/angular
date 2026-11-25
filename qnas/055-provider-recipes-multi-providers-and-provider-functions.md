# Q&A: Provider recipes, multi providers, and provider functions

These answers cover the ways Angular providers map tokens to values, how multiple registrations contribute to one token, and how libraries can expose a typed configuration API. See the [lesson](../lessons/055-provider-recipes-multi-providers-and-provider-functions.md) for the full guide and the [example](../examples/055-provider-recipes-multi-providers-and-provider-functions.md) for a runnable feature setup.

## 1. What does a provider do?

A provider tells an injector which runtime token to match and how to obtain the value for that token. The recipe can use a class, an existing value, a factory, an alias, or multiple contributions.

## 2. What is the class shorthand?

When a class provides itself under its own token, put the class in the providers array:

~~~ts
providers: [SearchClient]
~~~

This is shorthand for providing SearchClient with useClass: SearchClient.

## 3. When should I use useClass?

Use it when consumers inject one token but the injector should instantiate a particular implementation, such as a production logger or a test logger.

~~~ts
providers: [
  { provide: AppLogger, useClass: ConsoleLogger },
]
~~~

## 4. Does useClass create the same instance as a separate provider for that class?

Not necessarily. If the class token and an abstraction token both have useClass registrations for the same implementation, each provider record can construct its own instance. Use useExisting when both tokens must resolve to one instance.

## 5. When should I use useValue?

Use it for an already-created value such as configuration, a constant, a callback, or a test double.

~~~ts
{ provide: APP_LABEL, useValue: 'Customer portal' }
~~~

Angular returns that value; it does not clone or freeze it.

## 6. When should I use useFactory?

Use it when the injected value must be created from runtime configuration or other dependencies.

~~~ts
{
  provide: SEARCH_ENDPOINT,
  useFactory: () => {
    const config = inject(API_SETTINGS);
    return new URL('/search', config.apiOrigin);
  },
}
~~~

If callers need an operation to run repeatedly, put that operation in a service method. A provider factory creates the dependency value for its injector; it is not a per-call callback.

## 7. Can useFactory call inject()?

Yes. Angular invokes provider factories in an injection context. The factory can call inject() to resolve dependencies available in that provider's injector.

Do not call inject() later from an arbitrary event handler or ordinary function outside an injection context.

## 8. What does the deps array do?

It declares tokens Angular should resolve and pass as arguments to the factory:

~~~ts
{
  provide: SEARCH_ENDPOINT,
  useFactory: (config: ApiSettings) =>
    new URL('/search', config.apiOrigin),
  deps: [API_SETTINGS],
}
~~~

The deps form makes the factory's DI dependencies explicit in provider metadata. The inject() form is another valid style for Angular factories.

## 9. When should I use useExisting?

Use it to make another token an alias for an already registered token:

~~~ts
providers: [
  ConsoleLogger,
  { provide: AppLogger, useExisting: ConsoleLogger },
]
~~~

Injecting either token resolves to the same ConsoleLogger instance.

## 10. What is the difference between useExisting and useClass?

useExisting performs an alias lookup to another provider. useClass tells Angular to create an instance of a class for this provider record. If two tokens each useClass the same class, they can resolve to separate instances.

## 11. What is a multi provider?

A multi provider allows multiple provider records to contribute values for one token in the same injector. Angular returns the contributions as an array.

~~~ts
{ provide: STARTUP_TASKS, useValue: initializeCache, multi: true }
{ provide: STARTUP_TASKS, useValue: initializeTelemetry, multi: true }
~~~

## 12. What type should a multi token have?

Type it as the array consumers receive, for example InjectionToken<readonly StartupTask[]>:

~~~ts
export const STARTUP_TASKS =
  new InjectionToken<readonly StartupTask[]>('STARTUP_TASKS');
~~~

Each multi provider contributes one StartupTask value.

## 13. Can a multi provider use different provider recipes?

Yes. A contribution can use useValue, useClass, useFactory, or useExisting, as long as all records for that token in the injector use multi: true and each recipe produces the item type expected by the array.

## 14. Can I mix multi: true and a regular provider for the same token?

Do not mix them in the same injector. Multi providers produce an array; a regular provider produces one value. Combining the incompatible shapes can cause an invalid multi-provider error.

## 15. Do multi-provider arrays merge across injector levels?

Do not assume they do. Angular resolves a token through its injector hierarchy and uses the nearest matching provider. If an extension list must include values from several features, compose those contributions at the intended shared injector.

## 16. Does multi: true make providers global?

No. It collects contributions within the injector where those records are registered. The injector's location still determines which consumers can resolve the token.

## 17. Should I use multi providers or one array with useValue?

Use multi providers when independent features need to contribute values without one central owner editing a shared array. Use one array value when a single owner already knows the complete collection.

## 18. How should a provider function be named?

Use the provideX pattern, such as provideAudit() or provideSearch(). The function name should communicate which feature or dependency bundle it configures.

## 19. What belongs in a provideX function?

Accept the feature's typed public options, then register the tokens, services, and feature wiring needed to implement them. Keep internal tokens and classes private when consumers do not need them.

## 20. Why return EnvironmentProviders?

Use EnvironmentProviders when a helper is designed for application or route environments. It communicates the intended injector context and helps prevent accidental placement in a component's providers array.

## 21. Where can I register the result of a provider function?

Register it in ApplicationConfig.providers for application-wide scope, or in Route.providers for a route feature and its children, if the helper returns EnvironmentProviders.

The registration location determines the scope.

## 22. Can an environment-provider helper go in Component.providers?

No. EnvironmentProviders are designed for environment injectors. Use ordinary providers in component metadata for component-scoped dependencies.

## 23. What are withX functions?

They are small option functions such as withConsoleOutput() or withRetry(options) that describe optional features. A provideX function can accept those options and compose their providers into one feature configuration.

## 24. How should a helper handle conflicting options?

Design its TypeScript API so invalid combinations are difficult to express. Use a discriminated union for mutually exclusive choices, and validate runtime values when compile-time types are insufficient.

## 25. Should I export every token used by a provider helper?

No. Export only the tokens that consumers are meant to provide or inject directly. Keep internal configuration and implementation tokens private when the provideX function is the supported public API.

## 26. How do I test a provider recipe?

Configure TestBed with the same token and scope the code under test uses. Replace a class with a fake through useClass, supply a fixed value with useValue, or override the provider using TestBed APIs. Testing the consumer alone does not prove the production app registered the right environment scope.

## 27. Is a provider factory invoked once for the whole application?

It is cached by its injector's provider record after the injector first resolves it. A different injector scope can have a distinct provider record and value. Do not use a factory when you need a new value for every method call.

## 28. Can multiple provideX helpers register the same token?

They can, but the API must define how that is supposed to work. For an extension list, use a multi token consistently. For a single configuration token, define whether one helper replaces another and avoid accidental duplicate registrations.

## 29. Are provideRouter() and provideHttpClient() examples of provider functions?

Yes. They are public configuration functions that package the framework providers and options needed for routing or HTTP. Prefer these public APIs instead of copying framework internals into application code.

## 30. Which service decorator should I use for a manually registered service?

In Angular v22.1 and newer, use @Service({ autoProvided: false }) when the class should only be registered through the provider helper or providers array. In projects using Injectable, use Injectable() without providedIn for the same manual-registration intent.

## Further reading

- [The lesson](../lessons/055-provider-recipes-multi-providers-and-provider-functions.md)
- [The example](../examples/055-provider-recipes-multi-providers-and-provider-functions.md)
- [Defining dependency providers](https://angular.dev/guide/di/defining-dependency-providers)
- [Provider recipes and provider identifiers](https://angular.dev/guide/di/dependency-injection-providers)
- [makeEnvironmentProviders API](https://angular.dev/api/core/makeEnvironmentProviders)
- [Invalid multi-provider error](https://angular.dev/errors/NG0209)
- [InjectionToken and typed application configuration](../lessons/054-injectiontoken-and-typed-application-configuration.md)
