# Q&A: Services and the injectable pattern

These answers follow the services lesson and current Angular v22 documentation. Check the Angular version installed in your project before using the newer `@Service()` decorator; projects before v22 should use the `@Injectable()` patterns supported by their version.

## 1. What is an Angular service?

A service is a class that owns reusable behavior, data access, or state outside a particular component view. Examples include an API client, a shopping-cart state owner, an authentication/session service, or a logger. Angular's dependency-injection system can create the class and supply it to components and other services.

A service is still an ordinary TypeScript class. Angular DI behavior comes from its provider configuration and the injector that provides it.

## 2. What is the difference between a service, provider, and injector?

- A **service** is the class or value a consumer wants to use.
- A **provider** tells Angular what value or implementation to return for a token.
- An **injector** stores and resolves providers. Angular asks the current injector for a token and may search parent injectors when it is not found locally.
- An **injected instance** is the value the injector returns to a consumer.

For a class token, a simple provider can look like `providers: [ProductApi]`. A provider can also map a token to a class, value, alias, or factory.

## 3. Does every service have to use a decorator?

For a class that Angular must create with dependencies or register through the decorator's automatic provision, mark it with the decorator supported by the installed Angular version: `@Service()` in current Angular v22 for the common root-provided case, or `@Injectable()` for the broader and earlier-supported patterns.

A plain class can also be registered with an explicit provider, but it will not receive Angular DI metadata merely because its filename ends in `.service.ts`.

## 4. What does `@Service()` do?

In current Angular v22, `@Service()` marks a class as a service and makes it automatically available from the root injector by default. It is a concise option for a root-provided class whose dependencies are obtained with `inject()`.

`@Service()` supports `autoProvided: false` when the class should be manually registered, and it can accept a `factory` option for custom creation. It does not support constructor-based DI or all the advanced provider configuration options available with `@Injectable()`.

## 5. What does `@Injectable()` do?

`@Injectable()` marks a class so Angular can inspect and construct it for dependency injection. With `@Injectable({ providedIn: 'root' })`, it also registers a tree-shakeable provider in the root injector.

`@Injectable()` without `providedIn` does not automatically register the service in an injector. Add it to an appropriate `providers` array or configure another provider if you use that form.

## 6. Should I choose `@Service()` or `@Injectable()`?

For a new Angular v22 root singleton that uses `inject()`, `@Service()` is the concise default. Choose `@Injectable()` when constructor-based injection, provider strategies such as `useClass` or `useExisting`, or a non-root `providedIn` scope is needed.

For Angular versions before v22, use `@Injectable({ providedIn: 'root' })` for a typical application-wide service. Do not assume `@Service()` exists in a project whose installed Angular version predates it.

## 7. Is `@Service()` supported in every Angular project?

No. It is part of the current Angular v22 API. Earlier Angular releases use the established `@Injectable()` approach. Check the project's installed Angular package versions and their matching documentation before copying a snippet.

## 8. How does `inject()` work?

`inject(Token)` asks the active Angular injector for a value associated with that token. In a service or component, store the result in a field initializer so the dependency is retrieved when Angular creates the class:

~~~ts
export class ProductPageComponent {
  readonly catalog = inject(ProductCatalogService);
}
~~~

A service can use the same pattern to receive its own dependencies. This is the only dependency-injection style supported by `@Service()`.

## 9. Where is `inject()` allowed?

Angular requires an injection context. Common valid places are field initializers and constructors of classes created by Angular, provider and token factories, and framework callbacks documented to run in an injection context, such as functional route guards.

Calling `inject()` later from an ordinary event handler, ordinary method, or lifecycle hook usually fails with NG0203 because the construction-time injection context has ended. Inject the dependency into a field first, then call its methods from normal code.

## 10. Can I use constructor injection with `@Service()`?

No. Current `@Service()` is designed for the `inject()` function. If you want Angular to provide constructor parameters, use `@Injectable()`:

~~~ts
@Injectable({ providedIn: 'root' })
export class ProductCatalogService {
  constructor(private readonly api: ProductApi) {}
}
~~~

An `@Injectable()` class can also use `inject()` in field initializers. Pick the style that best fits the class and keep it consistent.

## 11. Is a root-provided service always one instance?

It is one instance per token in a given root injector. A separately bootstrapped Angular application has its own root injector. A child injector can also provide the same token again, in which case consumers below that child can receive a nearer, separate instance.

So “singleton” means shared within the injector scope that provides the token, not one JavaScript object across every Angular app, browser tab, or server process.

## 12. How do I make a service shared across the whole application?

In Angular v22, use `@Service()` for a typical root service using `inject()`. With `@Injectable()`, use `@Injectable({ providedIn: 'root' })`. Then inject the service where needed without adding it repeatedly to component providers.

For a service manually provided with `@Service({ autoProvided: false })` or plain `@Injectable()`, register it at the application level if the whole app should share it.

## 13. When should I use component-level providers?

Use a component provider when that component and its descendants should own an isolated instance. Each instance of the providing component receives its own service instance, and the instance is destroyed with that component.

Examples include a form's draft state, one editor's undo history, or a modal's local state. Do not add a shared root service to every component's providers array unless separate instances are intended; the local provider shadows the parent provider.

## 14. Can I manually scope a `@Service()` class?

Yes. Set `@Service({ autoProvided: false })`, then register the class in an application, route, or component provider list at the intended scope:

~~~ts
@Service({ autoProvided: false })
export class DraftStateService {}

@Component({
  selector: 'app-draft-editor',
  providers: [DraftStateService],
  template: '<ng-content />',
})
export class DraftEditorComponent {}
~~~

The provider location determines which consumers share that instance.

## 15. What is route-level service scope for?

A route can declare providers that are available to components, guards, and resolvers in that route. This is useful for state or dependencies owned by one feature area, such as an admin section or a checkout flow.

Route-level provisioning is different from component-level provisioning: the route injector serves the route's feature tree, while a component provider creates a scope at that component and its descendants.

## 16. What happens if a token is provided at both root and component level?

The nearest provider wins. A component below a local provider receives the local instance, even if the same token also has a root provider. This can be useful for overriding behavior or isolating state, but it can also explain why two components unexpectedly disagree about the current data.

Inspect the provider arrays from the consumer upward when debugging duplicate state.

## 17. Why does `@Injectable()` compile but Angular report no provider?

The decorator adds injectable metadata, but `@Injectable()` by itself does not choose an injector. If there is no `providedIn` option and no provider registration, Angular has no recipe for resolving the token and can report NG0201.

For a root service, use `@Injectable({ providedIn: 'root' })`. Otherwise register it at the desired application, route, or component scope.

## 18. What does NG0201 mean for services?

NG0201 means Angular could not find a provider for the requested token in the applicable injector hierarchy. Check that:

1. The class or token has a provider.
2. The provider is registered in an injector visible to the requesting component or service.
3. The consumer imports and requests the same token that was registered.
4. A conditional or lazy feature has actually been configured before the request occurs.

Do not “fix” every NG0201 by adding the service to the root. The correct scope depends on who should share it and for how long.

## 19. Why does `inject()` throw NG0203?

NG0203 indicates that `inject()` was called without an active injection context. Move the call to a field initializer or constructor of an Angular-created class, or into a supported provider/factory context. A normal method should use a dependency already stored on the class rather than asking the injector again.

## 20. Can one service inject another service?

Yes. For example, a catalog service can inject an API client and a logger. Angular resolves each dependency from the provider graph when it creates the catalog service.

Keep the dependency direction understandable. A component can depend on a feature service, which depends on lower-level API and logging services. Avoid circular dependencies where two services require each other to be constructed.

## 21. How are services different from components?

A component owns a view: its template, view-specific state, inputs, outputs, and user interactions. A service owns reusable behavior or state that should not belong to one specific view.

A component can still contain small local logic. Extract behavior when there is a clear reusable responsibility or a useful boundary for testing and maintenance; extraction is not a goal by itself.

## 22. Should every service be a singleton?

No. Application-wide clients and shared session state often belong at root. A form draft or modal state may need an isolated component-level instance. A feature workflow might be scoped to a route.

Choose the scope by the required sharing and lifetime, then put the provider at that injector boundary. A class name or decorator alone does not answer whether two consumers share an instance if providers exist at multiple levels.

## 23. Can I replace a service with a fake in tests?

Yes. Angular's provider system lets a test supply a fake implementation for a token, so a component or service can be checked without using the real network or integration. Keep the fake's public methods aligned with what the consumer calls.

For instance, provide a test `ProductApi` with a known product list and verify the catalog's public output. The exact TestBed setup depends on whether the test is for a service, component, or application configuration.

## 24. What are `useClass`, `useValue`, `useExisting`, and `useFactory`?

They are advanced provider strategies supported through Angular's provider configuration:

- `useClass` maps a token to a class implementation.
- `useValue` maps a token to a fixed object or value.
- `useExisting` maps a token to another already-registered token.
- `useFactory` calls a factory to produce the provided value.

Use `@Injectable()` plus provider configuration when you need these patterns. The `factory` option on `@Service()` is for choosing the service instance through a factory; it is not a general replacement for all provider keys.

## 25. Does the Angular CLI always generate the same decorator code?

No. Generated source can change between Angular CLI versions and project options. The command `ng generate service products/product-catalog` creates a service file, but inspect its decorator and provider configuration in the generated result instead of assuming every CLI version writes identical code.

## 26. Should a service hold signals?

It can, when the service owns reactive shared state that consumers need to observe. Expose a deliberate public API and avoid allowing every consumer to mutate internal state arbitrarily. A service with no reactive state does not need a signal just because Angular supports signals.

## 27. Should API calls live in components or services?

If the API behavior is reused or the component should not own transport details, put data access behind a service. A small, single-use interaction can remain in a component when that is simpler. The goal is a clear boundary, not moving every line into a service.

For a larger application, it can be useful to separate an API client from a feature service: the API client handles HTTP transport, and the feature service coordinates domain rules or state. For a small application, a single focused service may be enough.

## 28. What is the safest way to start a service?

1. State the responsibility in one sentence.
2. Decide who needs it and how long its state should live.
3. Check the installed Angular version.
4. Choose `@Service()` for the common v22 root case, or `@Injectable()` for earlier versions and advanced needs.
5. Register it at the intended injector scope.
6. Inject it where needed and keep `inject()` calls in valid contexts.
7. Verify that consumers share or isolate state as intended, and provide a fake in tests where useful.

## Continue learning

- Lesson: [Services and the injectable pattern](../lessons/049-services-and-the-injectable-pattern.md)
- Example: [Services and the injectable pattern](../examples/049-services-and-the-injectable-pattern.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="048-inspect-signals-and-components-with-angular-devtools.md">048 - Inspect signals and components with Angular DevTools</a></td><td><a href="../lessons/049-services-and-the-injectable-pattern.md">Lesson</a></td><td><a href="../examples/049-services-and-the-injectable-pattern.md">Example</a></td><td><a href="050-inject-dependencies-with-inject-and-constructor-injection.md">050 - Inject dependencies with `inject()` and constructor injection</a></td>
</tr>
  </tbody>
</table>
