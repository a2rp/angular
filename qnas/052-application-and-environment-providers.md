# Q&A: Application and environment providers

These answers focus on configuring providers in a standalone Angular application. See the [lesson](../lessons/052-application-and-environment-providers.md) for the full guide and the [example](../examples/052-application-and-environment-providers.md) for a runnable setup.

## 1. What is ApplicationConfig?

ApplicationConfig is the configuration object accepted by standalone bootstrap. Its providers array contains providers and EnvironmentProviders that should be available to the root component and its descendants.

~~~ts
export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient()],
};
~~~

## 2. Where does bootstrapApplication() get the root providers?

Pass ApplicationConfig as its second argument. Angular creates the application’s root EnvironmentInjector during bootstrap and registers those providers there.

~~~ts
bootstrapApplication(AppComponent, appConfig);
~~~

## 3. Does bootstrapApplication() work with a non-standalone root component?

No. The root component passed to bootstrapApplication() must be standalone. NgModule-based applications typically bootstrap through their root NgModule instead.

## 4. Should all providers go in main.ts?

No. For a small app, passing providers inline is fine. As configuration grows, place it in app.config.ts and import that object from main.ts. This keeps bootstrap code short and makes the application wiring easier to review.

## 5. Is ApplicationConfig the same as a component’s providers array?

No. ApplicationConfig.providers configures the root EnvironmentInjector. Component.providers configures an ElementInjector for that component instance and its eligible descendants. Their accepted provider types and sharing scopes differ.

## 6. How is @Service() different from ApplicationConfig.providers?

In current Angular, @Service() can automatically register a service in the root EnvironmentInjector. ApplicationConfig.providers is an explicit application wiring point for services, values, and framework features that need configuration. A service marked autoProvided: false can be manually provided in the app config.

Existing applications may use @Injectable({ providedIn: 'root' }) for root registration.

## 7. When should I use an application-level provider?

Use application scope when the service or feature is needed broadly across the application, such as HTTP, routing, public runtime settings, or a shared logger. Choose route or component scope when only that feature or component tree should receive the provider.

## 8. What is a provideX() function?

It is a public configuration function that registers the providers needed by an Angular feature or library. Examples include provideRouter() and provideHttpClient(). These functions usually accept typed feature options and return an environment provider bundle.

Use the function rather than copying internal provider registrations from a framework package.

## 9. What does EnvironmentProviders mean?

EnvironmentProviders is a wrapper type for providers intended for an EnvironmentInjector. It prevents environment-only provider bundles from being passed accidentally to Component.providers. Treat the value as an opaque bundle and pass it to an API that accepts environment providers.

## 10. What is the difference between Provider and EnvironmentProviders?

Provider is a regular DI registration, such as a class provider or useValue mapping. EnvironmentProviders wraps a group of registrations intended for an EnvironmentInjector. ApplicationConfig.providers accepts either type. A component injector does not accept environment-only bundles.

## 11. Why does provideHttpClient() go in ApplicationConfig?

provideHttpClient() returns environment-level providers for HttpClient and its configuration. Register it at application or another supported environment scope so services and components can inject the configured client.

~~~ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
  ],
};
~~~

## 12. How do I configure HTTP interceptors?

Pass them through the documented interceptor feature function to provideHttpClient(). Use functional interceptors with withInterceptors() when that matches the application’s design.

~~~ts
provideHttpClient(
  withInterceptors([authInterceptor, loggingInterceptor]),
)
~~~

Import the APIs from @angular/common/http and register this provider bundle in an environment injector.

## 13. How do I register routes in a standalone application?

Pass the route table to provideRouter() from @angular/router and put the result in ApplicationConfig.providers.

~~~ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
  ],
};
~~~

Then use RouterOutlet and other router directives in the relevant standalone component imports.

## 14. What does makeEnvironmentProviders() do?

It wraps ordinary providers in an EnvironmentProviders bundle. Use it when creating a reusable provideX() function so TypeScript helps prevent callers from adding that environment-only configuration to a component injector.

## 15. How do I build a reusable provider helper?

Create a function that accepts the options consumers are allowed to configure, then return makeEnvironmentProviders() containing the token/value and the service class.

~~~ts
export function provideSearch(options: SearchOptions) {
  return makeEnvironmentProviders([
    { provide: SEARCH_OPTIONS, useValue: options },
    SearchClient,
  ]);
}
~~~

Keep internal tokens and implementation details private when possible.

## 16. Can I call makeEnvironmentProviders() in a component’s providers?

Do not. Its result is specifically typed for environment injectors. Put it in ApplicationConfig.providers or another documented environment configuration point. Do not bypass the type check with a cast.

## 17. Does application configuration make a provider impossible to override?

No. A component or route can register a nearer provider for the same token, and hierarchical lookup can resolve that closer provider in its scope. Use that deliberately because it may create a different instance for those consumers.

## 18. Can application configuration contain an InjectionToken value?

Yes. Use an InjectionToken for non-class runtime values such as a typed configuration object, function, or primitive. Provide the exact exported token object and inject that same object in consumers.

## 19. Is browser runtime configuration safe for secrets?

No. Any value shipped to the browser can be inspected by users. Keep browser configuration limited to public settings. Private credentials and privileged operations belong on a server.

## 20. What does provideAppInitializer() do?

It registers a function that runs during application initialization. The function runs in an injection context, so it can call inject(). If it returns a Promise or Observable, Angular waits for the asynchronous work to resolve or complete before initialization finishes.

## 21. Does provideAppInitializer() save the value returned by an HTTP request?

Not by itself. The initializer’s returned Promise or Observable controls when initialization is considered complete. Store the result in a configuration service or another state container if components need to read it later.

## 22. Can inject() be called inside an initializer callback?

Yes. provideAppInitializer() invokes the callback in an injection context. Call inject() synchronously inside that callback and return the asynchronous work. Do not wait for an asynchronous callback and then call inject() after the context has ended.

## 23. Is APP_INITIALIZER still the preferred API?

No. APP_INITIALIZER is deprecated in current Angular. Use provideAppInitializer() for new code.

## 24. What does importProvidersFrom() do?

It collects providers from an NgModule and its imported NgModules for use in an environment injector. It is useful when gradually migrating a project or library from NgModules to standalone configuration.

## 25. Does importProvidersFrom() import components into a standalone template?

No. It imports providers, not component or directive declarations for a standalone component template. Import standalone components directly, or use the appropriate component-import mechanism for the legacy declaration.

## 26. Should I use importProvidersFrom() in every standalone app?

No. Prefer native standalone APIs and provideX() functions for new configuration. Use importProvidersFrom() when you still need providers exposed by an existing NgModule.

## 27. How do I know if a provider is root, route, or component scoped?

Check where it is registered:

- @Service() or ApplicationConfig.providers: root EnvironmentInjector.
- Route.providers: a route environment for that feature.
- Component.providers: an ElementInjector associated with a component instance.

The nearest provider on the lookup path wins. See hierarchical dependency injection.

## 28. What should I check when a provider is missing?

Confirm that the provider is registered in an injector visible to the requester, that the same InjectionToken object is used at both registration and injection, and that an environment provider is not placed in a component providers array. Also check whether a nearer provider shadows the value you expected.

## 29. Does the order of services in ApplicationConfig.providers control initialization order?

Do not use array ordering as a general service initialization mechanism. Provider registration makes tokens resolvable; provideAppInitializer() is the explicit API for asynchronous work that must finish during startup.

## 30. What is the simplest rule for choosing a provider function or direct provider?

Use a public provideX() function when a framework or library offers one for a feature. Use direct class/value/factory providers for your own app-level tokens and services. Use makeEnvironmentProviders() to package a reusable environment-only configuration.

## Continue learning

- Lesson: [Application and environment providers](../lessons/052-application-and-environment-providers.md)
- Example: [Application and environment providers](../examples/052-application-and-environment-providers.md)
- Next lesson: [Component-scoped and route-scoped providers](../lessons/053-component-scoped-and-route-scoped-providers.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="051-hierarchical-dependency-injection.md">051 - Hierarchical dependency injection</a></td><td><a href="../lessons/052-application-and-environment-providers.md">Lesson</a></td><td><a href="../examples/052-application-and-environment-providers.md">Example</a></td><td><a href="053-component-scoped-and-route-scoped-providers.md">053 - Component-scoped and route-scoped providers</a></td>
</tr>
  </tbody>
</table>
