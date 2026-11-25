# Q&A: Inject dependencies with `inject()` and constructor injection

These answers cover the two common ways Angular classes request dependencies. Current Angular v22 has `@Service()` for root-provided services that use `inject()`. Constructor injection remains supported in components and in services marked with `@Injectable()`.

## 1. What does dependency injection mean?

Dependency injection means a class receives the values it needs from an outside system instead of constructing those values itself. In Angular, the injector resolves a requested token using registered providers and supplies the result to a component, directive, or service created by Angular.

This separates a consumer from the details of how a dependency is created. It also gives an application or test a place to choose a provider implementation.

## 2. What is a dependency token?

A token is the runtime key Angular uses to find a provider. A class can be its own token, so requesting `ProductApiService` asks the injector for a provider registered for that class. `InjectionToken<T>` creates a token for values such as configuration objects, functions, or interfaces.

The TypeScript type helps the compiler check the value, while the runtime token tells Angular what to resolve.

## 3. What is the difference between providing and injecting?

Providing makes a token available in an injector and tells Angular what value or implementation it represents. Injecting requests that token from a consumer. A dependency can be injected only when an appropriate provider is available from the injector used for that request.

Changing a class from constructor injection to `inject()` does not create a provider and does not change the provider's scope.

## 4. Where can I use `inject()`?

Use `inject()` in an injection context, commonly:

- A field initializer of a component, directive, or DI-created service.
- The constructor body of a class Angular is currently creating.
- A provider or `InjectionToken` factory.
- A framework callback documented to run in an injection context, such as a functional route guard.

The class must be created through Angular DI for its field initialization or constructor to have the expected injector available.

## 5. Can I call `inject()` inside a constructor?

Yes. During construction of a class that Angular is creating, `inject()` is valid in the constructor body:

~~~ts
@Component({ selector: 'app-example', template: '' })
export class ExampleComponent {
  private readonly router: Router;

  constructor() {
    this.router = inject(Router);
  }
}
~~~

Most classes can use a field initializer instead, which is shorter. Constructor-body injection is useful when the constructor already contains setup that needs the dependency.

## 6. Can `inject()` be called from a normal method?

Usually no. After Angular has finished constructing the instance, an event handler, ordinary method, or lifecycle hook such as `ngOnInit` no longer runs in the class creation context. Calling `inject()` there can throw NG0203.

Request the dependency once in a field initializer or constructor, then call methods on the stored field from later code.

## 7. Why does `inject()` work in a field initializer?

Angular creates the component, directive, or service through DI. The field initializer runs as part of that creation, while the injector context is available. Angular can therefore resolve the requested token and assign it to the field.

A manually constructed class created with `new` does not get an Angular injector context just because one of its fields calls `inject()`.

## 8. What does constructor injection look like?

Declare dependencies as constructor parameters. Angular resolves class tokens and passes their values when it constructs the DI-managed class:

~~~ts
@Component({ selector: 'app-example', template: '' })
export class ExampleComponent {
  constructor(private readonly router: Router) {}
}
~~~

The TypeScript parameter property both declares a field and assigns the argument to it. The component can then use `this.router` in ordinary methods.

## 9. Can components use constructor injection?

Yes. Components, directives, and services created by Angular can use either constructor injection or `inject()` field injection. `@Component()` already makes the component a DI participant; it does not need an additional `@Injectable()` decorator.

For services, use an Angular service decorator and provider configuration appropriate to the chosen injection style.

## 10. Can a service use constructor injection with `@Service()`?

No. Current Angular v22's `@Service()` is designed for services that receive dependencies with `inject()`. If a service should use constructor parameters, mark it with `@Injectable()` and configure its provider, commonly with `providedIn: 'root'`.

For an earlier Angular version without `@Service()`, the established `@Injectable({ providedIn: 'root' })` pattern is the standard choice for a root service.

## 11. What is the difference between `inject()` and constructor injection?

They use the same injector and provider resolution. Their syntax and tradeoffs differ:

- `inject()` keeps a dependency request in a field initializer and avoids a constructor used only for assigning fields.
- Constructor injection shows dependencies together in the constructor signature and supports ordinary parameter decorators such as `@Optional()`.
- `@Service()` services use `inject()`; `@Injectable()` services can use either style.
- Components can use either style.

Follow the class decorator's requirements and the repository's style. Avoid mixing styles without a specific reason.

## 12. Do both styles return the same service instance?

They resolve the same token through the injector used at that point. If both consumers resolve a root-provided token from the same root injector and no closer provider overrides it, they receive the same root instance. The injection syntax itself does not create a new instance.

A component or route provider can create a closer instance regardless of which syntax the consumer uses. Injector scope determines reuse.

## 13. What is an `InjectionToken` for?

An `InjectionToken<T>` gives Angular a runtime token for a value that is not naturally represented by a class. Common uses include configuration objects, API URLs, feature flags, function values, and interfaces.

TypeScript interfaces are erased when JavaScript is emitted, so Angular cannot resolve an interface as a runtime token. Create one `InjectionToken` for the interface and provide that token.

## 14. How do I inject an `InjectionToken` with `inject()`?

Pass the exported token object to `inject()`:

~~~ts
readonly apiConfig = inject(API_CONFIG);
~~~

The token must have a provider in an injector visible to the consumer, unless the token itself has a factory that supplies a default value.

## 15. How do I inject an `InjectionToken` through a constructor?

Use `@Inject(TOKEN)` on the constructor parameter because TypeScript type information alone cannot identify an interface or primitive token at runtime:

~~~ts
constructor(@Inject(API_CONFIG) private readonly config: ApiConfig) {}
~~~

`@Inject()` selects the token for that parameter; it does not register a provider for it.

## 16. Does the string in `new InjectionToken('api.config')` identify the token?

No. The string is a human-readable debugging label. Angular uses the identity of the `InjectionToken` object itself. Export one token constant, then import that exact constant wherever it is provided or injected.

Creating a second `new InjectionToken('api.config')` produces a different token, even though the labels match.

## 17. Can I inject a TypeScript interface directly?

No. Interfaces and type aliases do not exist at runtime, so they cannot be DI tokens. Define an `InjectionToken<InterfaceName>` and map it to an implementation with a provider, for example:

~~~ts
export const DATA_READER = new InjectionToken<DataReader>('data.reader');

providers: [{ provide: DATA_READER, useClass: HttpDataReader }]
~~~

Then request `DATA_READER` using `inject(DATA_READER)` or `@Inject(DATA_READER)`.

## 18. What happens when I inject an unprovided token?

For a required dependency, Angular throws a no-provider error, commonly NG0201. Check that the provider exists and is registered in the application, route, or component injector visible to the consumer. Also confirm that the provider and consumer use the same token object.

Do not automatically register every missing dependency at root. Choose the scope based on which consumers should share the value and how long it should live.

## 19. How do I make a dependency optional with `inject()`?

Pass the optional flag:

~~~ts
readonly analytics = inject(AnalyticsService, { optional: true });
~~~

The result can be `null`, so the TypeScript type and calling code must handle that:

~~~ts
this.analytics?.trackPurchase(orderId);
~~~

Use this only when the capability is truly optional. Making a required dependency optional can hide a provider setup bug.

## 20. How do I make a constructor dependency optional?

Add `@Optional()` to the constructor parameter and reflect possible absence in the type:

~~~ts
constructor(
  @Optional() private readonly analytics: AnalyticsService | null,
) {}
~~~

The same principle applies: optional DI means a missing provider is expected and handled, not that the provider is unnecessary for a required behavior.

## 21. What do `self`, `skipSelf`, and `host` do?

They change where a specific dependency request searches in Angular's injector hierarchy:

- `self` checks only the current injector.
- `skipSelf` starts at the parent injector instead of the current one.
- `host` limits the upward search at the host boundary for element injectors.
- `optional` changes what happens if resolution finds no provider, returning `null` instead of throwing.

With `inject()`, pass these as options. Constructor injection has corresponding parameter decorators such as `@Self()`, `@SkipSelf()`, `@Host()`, and `@Optional()`. They do not create providers. The following chapter covers the injector hierarchy and these boundaries in depth.

## 22. Can I use `@Inject()` without a constructor parameter?

`@Inject()` is a parameter decorator for constructor injection. To inject into a field, call `inject(TOKEN)` directly. Do not add `@Inject()` to a field as if it were a field-injection decorator.

## 23. Can a class mix `inject()` fields and constructor parameters?

Angular supports both forms, but mixing them without a design reason makes dependencies less predictable to readers. Prefer one style per class. A concrete case such as framework decorators, base-class constructor requirements, or gradual migration may justify a mix; document that reason and keep ownership clear.

## 24. Does `new MyService()` use Angular DI?

No. Calling `new` constructs the class directly. Angular does not resolve its dependencies, honor provider overrides, or manage the instance's injector scope. Let Angular create DI-managed services through their provider and inject them into consumers.

Manual construction can be appropriate for a plain data object with no Angular-managed dependencies, but it is not a substitute for injecting a configured service.

## 25. Can I use both injection styles in one application?

Yes. Different classes can use different styles. For example, an existing component may use constructor injection while a new root service uses `@Service()` and `inject()`. Keep each class internally consistent and respect the version and decorator requirements.

## 26. What does NG0203 mean?

NG0203 means `inject()` was called outside an injection context. Move the call to a class field initializer, a constructor body while Angular is constructing the class, or another documented context such as a provider factory or functional guard.

A typical fix is to store the dependency first:

~~~ts
private readonly search = inject(SearchService);

submit(query: string): void {
  this.search.run(query);
}
~~~

## 27. Can I inject a dependency inside `ngOnInit()`?

No, not with `inject()`. `ngOnInit()` runs after construction, so it is outside the constructor injection context. Declare the dependency as a field or constructor parameter, then use it in `ngOnInit()` if needed.

## 28. Does optional injection solve a no-provider error?

It can prevent a throw when a dependency is intentionally optional. It is not the right fix for a required dependency. If the consumer cannot work without the value, register the provider at the appropriate scope and keep injection required so configuration mistakes remain visible.

## 29. How can tests replace an injected dependency?

Configure the test injector with a provider for the same token, using a fake value or class. Components and services then receive the test implementation through normal DI. This keeps tests independent of real HTTP calls or external integrations.

The test provider must use the same class or `InjectionToken` object requested by the code under test. Matching token labels alone are not enough.

## 30. Which style should I use for a new class?

Use this quick decision:

1. If it is a current Angular v22 root service decorated with `@Service()`, use `inject()` fields.
2. If a service requires constructor injection, advanced provider configuration, or an earlier-version pattern, use `@Injectable()` as appropriate.
3. For components and directives, use either `inject()` fields or constructor parameters, based on clarity and project conventions.
4. Use `InjectionToken<T>` for values without a class token.
5. If a dependency can be absent, request it as optional and handle `null` explicitly.

## Official references

- [Dependency-injection overview](https://angular.dev/guide/di)
- [The `inject()` API](https://angular.dev/api/core/inject)
- [Injection context](https://angular.dev/guide/di/dependency-injection-context)
- [Defining dependency providers](https://angular.dev/guide/di/defining-dependency-providers)
- [Hierarchical injectors](https://angular.dev/guide/di/hierarchical-dependency-injection)

## Continue learning

- Lesson: [Inject dependencies with inject and constructor injection](../lessons/050-inject-dependencies-with-inject-and-constructor-injection.md)
- Example: [Inject dependencies with inject and constructor injection](../examples/050-inject-dependencies-with-inject-and-constructor-injection.md)
- Next lesson: [Hierarchical dependency injection](../lessons/051-hierarchical-dependency-injection.md)
