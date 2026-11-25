# Q&A: Bootstrap, ApplicationConfig, and Application Providers

## Startup and the root component

### 1. What does bootstrapping an Angular application mean?

Bootstrapping is the startup process that creates the Angular application, establishes its root injector, creates the root component, and starts rendering the application.

### 2. What does bootstrapApplication do?

The standalone bootstrap API starts an application with a standalone root component and an optional application configuration. It returns a Promise that resolves to an ApplicationRef when startup succeeds.

### 3. What are the arguments to bootstrapApplication?

The first argument is the standalone root component class. The optional second argument is an ApplicationConfig object. The API also accepts an optional bootstrap context for advanced server-rendering scenarios; most browser entry files do not need to supply it.

### 4. Does the root component have to be standalone?

Yes. bootstrapApplication expects a standalone root component. A root component intended to be bootstrapped from an NgModule follows a different, module-based startup path.

### 5. Where is bootstrapApplication usually called?

In the application's entry file, commonly src/main.ts. That file imports the root component and, when the project uses a separate config file, the exported appConfig.

### 6. Why does bootstrapApplication return a Promise?

Application initialization can be asynchronous. The Promise represents completion or failure of bootstrap, so startup errors can be caught and reported.

### 7. Should the bootstrap Promise be handled?

It is good practice to handle rejection so startup failures are visible in diagnostics. The CLI-generated entry file may already include a catch handler; preserve the project's generated pattern.

## Application configuration

### 8. What is ApplicationConfig?

ApplicationConfig is the type of configuration passed to bootstrapApplication. Its providers property configures dependencies and framework features for the application.

### 9. Is app.config.ts required?

No. It is a common place to organize and export application configuration, but a configuration object can also be written inline at the bootstrap call. Follow the structure already used by the project.

### 10. What can go in ApplicationConfig.providers?

The array accepts regular dependency providers and EnvironmentProviders, including values returned by Angular feature setup functions such as provideRouter and provideHttpClient.

### 11. What is an application-level provider?

It is a dependency configured for the application's environment injector. Components and services under the application can resolve it, unless a closer injector provides its own value for the same token.

### 12. What is the difference between application providers and component imports?

Application providers configure dependency injection and framework services. Component imports make standalone components, directives, and pipes available to a particular component template.

### 13. Should I put a service in a component's imports array?

No. A service is injected through Angular's dependency injection system. A component imports template dependencies, not injectable service classes.

### 14. Should I put provideRouter in a component's imports array?

No. Call provideRouter(routes) in application providers. If a standalone component template uses router-outlet or a router directive, that component also imports the corresponding directive or standalone export.

## Providers and tokens

### 15. What is a provider?

A provider tells Angular how to supply a value for a dependency-injection token. It may associate a token with a class, an existing value, a factory, or another token.

### 16. What does the class-provider shorthand mean?

Putting a class in a providers array is shorthand for providing that class as both the token and implementation. For example, SessionService is shorthand for a class provider whose provide and useClass values are SessionService.

### 17. What is useValue for?

useValue supplies a value that already exists, such as an application settings object or a test double. Angular returns that configured value when the token is injected.

### 18. What is useClass for?

useClass tells Angular which class to instantiate for a token. It is useful when the token and chosen implementation differ, such as selecting a mock or alternate implementation.

### 19. What are useFactory and useExisting?

useFactory calls a function to produce a dependency, potentially using other injected dependencies. useExisting makes one token an alias for another token's instance.

### 20. What is an InjectionToken?

An InjectionToken is a runtime object that can identify a dependency that has no suitable class token, such as a configuration object, a function, or a primitive value.

### 21. Why can't I inject a TypeScript interface directly?

Interfaces and type aliases are erased during compilation, so they do not exist as runtime JavaScript values. Angular needs a runtime token for dependency lookup; an InjectionToken supplies one.

### 22. What does the generic type on InjectionToken do?

It gives TypeScript a type for the value expected from the token, improving checking at compile time. The generic does not create a runtime type check.

### 23. Is the InjectionToken description string its identity?

No. The token object itself is the identity used for lookup. The description helps humans recognize the token in debugging output.

### 24. What happens if a token has no provider?

Angular reports a dependency injection error when code requests that token and no provider can be found in the relevant injector hierarchy. Add the provider at the intended scope or use a service's providedIn declaration.

## Scope and service registration

### 25. What does providedIn: 'root' do?

It makes a service injectable through the root injector without requiring the class to be listed in ApplicationConfig.providers. It is a common default for app-wide services.

### 26. When should I use providedIn: 'root' versus app.config.ts?

Use providedIn: 'root' when a service can declare its own default root availability. Use application configuration for feature setup functions, explicit values, and provider choices that belong to application startup.

### 27. Do application providers and providedIn: 'root' conflict?

They can overlap if the same token is registered in both places. Avoid duplicate registrations unless the scope or replacement is intentional; a provider in a closer injector may result in a different instance.

### 28. Does an application provider automatically create one instance everywhere?

It supplies a value from the application injector to consumers that resolve through that injector. A nearer component, route, or other injector can provide the same token and create a separate scoped value.

### 29. Should every injectable class be added to ApplicationConfig.providers?

No. Services using providedIn are available without an explicit provider entry. Add an explicit provider when a token needs a value or implementation mapping, or when a framework feature requires configuration.

## Framework features and compatibility

### 30. Why do Angular APIs use functions such as provideRouter?

Provider functions bundle feature setup into a readable configuration call and return providers for Angular's injector. They are the standard configuration style for many standalone APIs.

### 31. Does provideHttpClient make an HTTP request?

No. It configures the HTTP client for injection. A service still needs to inject HttpClient and call its request methods to perform network work.

### 32. Does provideRouter make router-outlet available in every template?

It configures the router service and routes. A standalone component still needs to import RouterOutlet if its template uses router-outlet.

### 33. What is importProvidersFrom for?

It collects providers from existing NgModule-based libraries so they can be registered in standalone application or environment configuration. Prefer a library's dedicated provider function when available.

### 34. Does importProvidersFrom import NgModule components into a standalone template?

No. It collects providers. It does not add components, directives, or pipes to a component's template scope.

### 35. Should a new standalone app be built around NgModules?

Usually not. Standalone components and functional provider APIs are the modern default for new Angular applications. Use module-based APIs when maintaining or integrating code that still requires them.

## Versions, security, and troubleshooting

### 36. Should I copy older Zone.js provider examples into a new app?

Not automatically. Angular defaults and generated configuration vary by version; newer Angular versions use zoneless change detection by default. Check the installed Angular version and generated config before changing change-detection providers.

### 37. Can I put an API secret in an application settings object?

No. Browser-delivered configuration is visible to users. Keep private credentials on a trusted server; values such as a public API base URL are not secret merely because they are stored in a config file.

### 38. What should I check if an injected settings token fails?

Confirm that the same exported token object is used by both the provider and consumer, that a provider exists in an injector reachable from the consumer, and that no import accidentally creates a second token object.

### 39. What should I check if a standalone component reports an unknown template element?

Check the component's imports for the template dependency, such as RouterOutlet. Application providers configure services; they do not automatically add template dependencies.

### 40. What should I check after editing app.config.ts?

Preserve providers already generated for the project, verify each imported API and path, and confirm the bootstrap entry passes the intended appConfig. Then run the project's normal development or build command when you are ready to verify it.

## Further reading

- [bootstrapApplication API](https://angular.dev/api/platform-browser/bootstrapApplication)
- [ApplicationConfig API](https://angular.dev/api/core/ApplicationConfig)
- [Define dependency providers](https://angular.dev/guide/di/defining-dependency-providers)
- [Dependency injection providers](https://angular.dev/guide/di/dependency-injection-providers)
