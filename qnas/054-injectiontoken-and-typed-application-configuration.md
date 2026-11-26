# Q&A: InjectionToken and typed application configuration

These answers focus on runtime DI keys for values that do not have a class, and on safely typing configuration provided to an Angular application. See the [lesson](../lessons/054-injectiontoken-and-typed-application-configuration.md) for the full explanation and the [example](../examples/054-injectiontoken-and-typed-application-configuration.md) for a reusable configuration helper.

## 1. What is an InjectionToken?

It is a runtime object Angular can use as a key in dependency injection. Its generic type parameter connects that key to the value's TypeScript type.

~~~ts
export const API_SETTINGS =
  new InjectionToken<ApiSettings>('API_SETTINGS');
~~~

## 2. Why can't I inject an interface directly?

An interface is erased when TypeScript compiles to JavaScript. Angular needs a runtime key, so create an InjectionToken for the interface and inject that token instead.

## 3. When should I use InjectionToken instead of a class token?

Use it when the dependency is not a runtime class, such as an interface-shaped configuration object, string, number, function, array, or generic type. Use a class token when the class itself is an appropriate runtime identity.

## 4. What does the generic type parameter do?

It gives TypeScript the expected type when a consumer injects the token. For example, InjectionToken<ApiSettings> makes inject(API_SETTINGS) return an ApiSettings value at compile time.

The generic does not validate the value at runtime.

## 5. Does the description string identify the token?

No. It is a debugging description. Angular matches the exact InjectionToken object instance, not its description.

## 6. Why must I import the same token instance?

A provider and consumer need the same runtime key. Two InjectionToken objects created separately are different keys, even if both use the same generic type and description. Export one token from a shared file and import it wherever needed.

## 7. What happens if a token is not provided?

If no provider or token factory supplies it in the consumer's injector hierarchy, Angular throws a missing-provider error. A generic parameter does not automatically create a value.

## 8. How do I provide a typed value with useValue?

Create the token and register its value in the appropriate providers array:

~~~ts
providers: [
  { provide: API_SETTINGS, useValue: apiSettings },
]
~~~

ApplicationConfig.providers is a common location for app-wide settings. Route providers can supply a feature-specific value.

## 9. How can TypeScript check the configuration object's shape?

Give the object an explicit type or use the satisfies operator:

~~~ts
const apiSettings = {
  apiOrigin: 'https://api.example.test',
  requestTimeoutMs: 8000,
} satisfies ApiSettings;
~~~

The compiler checks required properties and their types while preserving the object's inferred type.

## 10. Does readonly make the config immutable at runtime?

No. readonly prevents reassignment through TypeScript-checked code. It does not freeze the object or validate values at runtime. Use runtime freezing only if the application specifically needs that behavior.

## 11. Can an InjectionToken define its own default value?

Yes. Supply a factory in the token options. A token with a factory is provided in the root environment by default.

~~~ts
export const DEFAULT_LANGUAGE =
  new InjectionToken<string>('DEFAULT_LANGUAGE', {
    factory: () => 'en',
  });
~~~

## 12. Can an InjectionToken factory inject other dependencies?

Yes. Angular calls the factory in an injection context, so it can use inject() to resolve dependencies.

~~~ts
export const API_ORIGIN =
  new InjectionToken<string>('API_ORIGIN', {
    factory: () => inject(API_SETTINGS).apiOrigin,
  });
~~~

## 13. What is the difference between a token factory and useFactory?

A token factory is part of the InjectionToken definition and provides that token by default in the root environment. A useFactory provider is registered explicitly in a providers array and can be scoped where that provider is registered.

Use a token factory for a safe default. Use useFactory when application or feature wiring should explicitly control how the value is created.

## 14. Can a token factory read a route-specific override?

A factory on a root-provided token runs in the root token's injector context. Do not assume it will read a value that exists only in a route injector. If a token must vary by route, register an explicit provider in that route's providers array or use a route-scoped provider helper.

## 15. What does makeEnvironmentProviders do?

It wraps provider definitions as EnvironmentProviders. This communicates that the provider bundle belongs in an environment injector and helps prevent accidentally putting it in a component's providers array.

## 16. When should a provideX helper return EnvironmentProviders?

Use that return type when the helper is designed for ApplicationConfig.providers or Route.providers. Build the bundle with makeEnvironmentProviders().

~~~ts
export function provideSearch(config: SearchConfig): EnvironmentProviders {
  return makeEnvironmentProviders([
    { provide: SEARCH_CONFIG, useValue: config },
    SearchClient,
  ]);
}
~~~

A component-only helper may instead use component provider patterns appropriate for an ElementInjector.

## 17. Does the helper determine whether configuration is global or route-scoped?

The registration location determines scope. Register the same environment-provider helper in ApplicationConfig.providers for application-wide dependencies, or in a Route.providers array for a route feature.

## 18. What is a provider function?

It is a function that accepts typed options and returns the provider configuration a feature needs. A good helper keeps internal tokens private and gives the caller a small, documented configuration API.

The next [provider recipes lesson](../lessons/055-provider-recipes-multi-providers-and-provider-functions.md) explores this pattern further.

## 19. How do I validate JSON configuration?

Receive external data as unknown and pass it through a runtime parser or schema validator before providing it. TypeScript interfaces and satisfies are compile-time tools; they do not inspect data returned from JSON.parse, fetch, local storage, or a URL.

## 20. Does a type assertion validate external data?

No. A cast such as rawConfig as ApiSettings only tells TypeScript to trust the value. It does not verify that apiOrigin exists or that requestTimeoutMs is a number.

## 21. Is browser configuration secret?

No. Values delivered to browser code can be inspected by users. Keep passwords, private API keys, signing secrets, and credentials on trusted servers.

## 22. Is InjectionToken the same as a build-time environment variable?

No. A token is an Angular DI key. A build-time environment file is a tooling convention for selecting values during a build. A token can provide a value that was selected by build configuration, bootstrap logic, or runtime loading.

Neither approach makes a value secret if it is included in browser-delivered code.

## 23. Can I call inject(API_SETTINGS) from an ordinary event handler?

Not unless the code is running in an Angular injection context. Inject in a component or service constructor/field initializer, a provider factory, a token factory, or another supported injection context. For an event handler, store the injected value on the class and use that field.

## 24. How do I make an optional token dependency?

Use inject with the optional option when absence is expected:

~~~ts
const analytics = inject(ANALYTICS_CLIENT, { optional: true });
~~~

The result can be null. Give that case explicit behavior instead of assuming a client exists.

## 25. How do I replace a token value in a test?

Provide the same token with a test value in TestBed:

~~~ts
TestBed.configureTestingModule({
  providers: [
    { provide: API_SETTINGS, useValue: testSettings },
  ],
});
~~~

This overrides the token for the test injector. A separate test InjectionToken with the same description would not replace the production token.

## 26. Can a route override app-wide configuration?

Yes. Register a provider for the same token on the route. Consumers in that route subtree use the nearer route value; consumers elsewhere continue resolving the application value. Keep such overrides intentional and document how the feature differs.

## 27. Should a configuration token be exported publicly?

Export the token when app consumers or other packages need to provide or inject it. Keep implementation-only tokens private inside the feature package. A public token is part of the feature's API and should have a stable, documented contract.

## 28. When should configuration be passed as a component input instead?

Use an input when a parent configures one component instance through the template. Use a DI token when the value is a dependency shared through an injector scope, used by services, or configured at the application or route level.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="053-component-scoped-and-route-scoped-providers.md">053 - Component-scoped and route-scoped providers</a></td><td><a href="../lessons/054-injectiontoken-and-typed-application-configuration.md">Lesson</a></td><td><a href="../examples/054-injectiontoken-and-typed-application-configuration.md">Example</a></td><td><a href="055-provider-recipes-multi-providers-and-provider-functions.md">055 - Provider recipes, multi providers, and provider functions</a></td>
</tr>
  </tbody>
</table>
