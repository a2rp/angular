# Services and the injectable pattern

An Angular service is a class that owns reusable application behavior or state. Components focus on presenting a view and responding to user interaction; services let multiple parts of the application share work such as loading data, applying business rules, tracking a session, or writing logs.

A class becomes available to Angular's dependency-injection (DI) system when it is registered with a provider. The decorator and provider configuration describe how Angular can create it. A service is not automatically a global singleton merely because it is a TypeScript class: its provider and the injector that owns that provider determine where an instance is available and how long it lives.

> **Version note:** Current Angular v22 documentation introduces `@Service()` as the concise default for a root-provided service that uses `inject()`. `@Injectable()` remains supported and is still the right choice for constructor injection, advanced provider configuration, and non-root `providedIn` scopes. On Angular versions before v22, use the established `@Injectable` patterns in this lesson.

## Learning outcomes

After this lesson, you should be able to:

- Explain the difference between a service class, a provider, an injector, and an injected instance.
- Choose a service for reusable behavior or state without moving view-specific logic out of a component unnecessarily.
- Use `@Service()` for a modern root-provided singleton in Angular v22+.
- Use `@Injectable({ providedIn: 'root' })` for compatible projects and cases that need its broader options.
- Inject dependencies with `inject()` and recognize the contexts where it is valid.
- Scope a service at the application, route, or component level and predict when separate instances are created.
- Diagnose common provider and injector-scope mistakes.

## Why use a service?

A component should coordinate its own view: it reads data, handles user events, and exposes values to its template. A service is useful when behavior or state needs a separate owner, when multiple components need the same capability, or when the behavior is easier to reason about outside a particular view.

Common service responsibilities include:

- Fetching and transforming data from an API.
- Applying business rules that are used by more than one component.
- Sharing state between parts of a feature or application.
- Managing authentication/session information or application configuration.
- Logging, analytics, and other integrations with external systems.

Do not make a service merely because a function exists. A tiny piece of view-only logic can stay in its component. Avoid a single service that becomes a home for unrelated data fetching, UI state, formatting, authentication, and every other concern. Prefer focused classes whose names describe their responsibility.

## Create a service

The Angular CLI can create the starting file:

~~~sh
ng generate service products/product-catalog
~~~

Review the generated code for your Angular CLI version. The command creates a TypeScript class; the decorator and provider configuration determine how Angular can instantiate and provide it.

A service that owns reusable data access can be as small as this:

~~~ts
import { Service, inject, signal } from '@angular/core';

export interface Product {
  id: number;
  name: string;
  price: number;
}

@Service()
export class ProductApi {
  private readonly products: Product[] = [
    { id: 1, name: 'Notebook', price: 4.5 },
    { id: 2, name: 'Pen', price: 1.25 },
  ];

  getProducts(): Product[] {
    return this.products.map((product) => ({ ...product }));
  }
}

@Service()
export class ProductCatalogService {
  private readonly api = inject(ProductApi);
  readonly products = signal<readonly Product[]>([]);

  load(): void {
    this.products.set(this.api.getProducts());
  }
}
~~~


`ProductApi` is a separate dependency, so the catalog service delegates data access instead of taking on every task itself. In a real application, the API layer might use `HttpClient` and return an `Observable` or a promise; the service boundary remains useful even when the implementation changes.

The example uses `@Service()` for current Angular v22+. A simple service can also keep ordinary private fields and methods; it does not need signals unless consumers need reactive state. Use a readonly signal or another deliberate public API when components need to observe changing service state.

## How dependency injection connects classes

Dependency injection separates the code that asks for a dependency from the code that constructs it. A class declares a dependency, Angular looks for a provider for that token in the current injector hierarchy, creates or retrieves the instance, then supplies it.

For class-based services, the class itself is commonly the token. Providers can also use `InjectionToken` values for configuration, functions, interfaces, or other non-class dependencies. The provider tells Angular what value or implementation to return for a token.

In Angular v22+, `@Service()` automatically makes a class available from the root injector by default. Use `inject()` in a field initializer to retrieve another dependency:

~~~ts
import { Service, inject, signal } from '@angular/core';

@Service()
export class ProductCatalogService {
  private readonly api = inject(ProductApi);
  private readonly logger = inject(AuditLogger);

  load(): Product[] {
    const products = this.api.getProducts();
    this.logger.info(`Loaded ${products.length} products`);
    return products;
  }
}
~~~

A component can request the same service:

~~~ts
import { Component, inject } from '@angular/core';

@Component({
  selector: 'app-product-page',
  template: '<p>Products: {{ catalog.products().length }}</p>',
})
export class ProductPageComponent {
  readonly catalog = inject(ProductCatalogService);

  constructor() {
    this.catalog.load();
  }
}
~~~

For a standalone component, make sure any template dependencies are imported in its `imports` metadata. The short template above uses no other Angular directives.

You can inject dependencies through a constructor when using `@Injectable()`:

~~~ts
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class ProductCatalogService {
  constructor(private readonly api: ProductApi) {}
}
~~~

Constructor injection makes dependencies explicit in the constructor signature and remains a well-supported Angular pattern. `@Injectable` classes can also use `inject()` in a field initializer. Do not put both forms in a class without a clear reason; choose one style consistently for that class.

## `@Service()` and `@Injectable()`

Use this decision guide for current Angular:

| Situation | Suitable approach |
| --- | --- |
| New root-provided singleton using `inject()` on Angular v22+ | `@Service()` |
| Project uses Angular before v22 | `@Injectable({ providedIn: 'root' })` |
| Constructor-based dependency injection | `@Injectable(...)` |
| Need `useClass`, `useValue`, `useExisting`, or `useFactory` provider options | `@Injectable(...)` plus provider configuration |
| Need a non-root `providedIn` scope such as `platform` | `@Injectable({ providedIn: ... })` |
| Need `@Service()` but want explicit route or component registration | `@Service({ autoProvided: false })` plus a `providers` array |

The `@Injectable` decorator marks a class for Angular DI and can configure automatic provision with `providedIn`. For the common application-wide case, `@Injectable({ providedIn: 'root' })` registers a tree-shakeable root provider. `@Injectable()` without `providedIn` supplies injectable metadata but does not, by itself, register the class in an injector; add it to a provider configuration where it should be available.

`@Service()` is not a replacement for every capability of `@Injectable`. It uses `inject()` rather than constructor parameters and focuses on root provision. If the class needs constructor injection, a provider strategy such as `useExisting`, or a non-root `providedIn` scope, keep `@Injectable` and configure it for the use case.

## `inject()` and injection context

Angular's `inject()` function is available only while Angular is resolving or constructing a dependency in an injection context. Common valid contexts include:

- A field initializer on a DI-created service, component, or directive.
- The constructor of a class being created by Angular.
- A provider or `InjectionToken` factory.
- Framework callbacks that Angular documents as running in an injection context, such as functional route guards.

This is valid because Angular constructs the component and evaluates its field initializers:

~~~ts
export class ProductPageComponent {
  readonly catalog = inject(ProductCatalogService);
}
~~~

This is invalid when called later from an ordinary method, event handler, or lifecycle hook:

~~~ts
refreshFromClick(): void {
  const catalog = inject(ProductCatalogService); // No active injection context here.
  catalog.load();
}
~~~

Inject the dependency once in a field initializer or constructor, then call its methods from event handlers:

~~~ts
readonly catalog = inject(ProductCatalogService);

refreshFromClick(): void {
  this.catalog.load();
}
~~~

There are advanced APIs such as `runInInjectionContext` for code that genuinely needs to execute with a specific injector. Most component and service code should inject dependencies when Angular constructs the class and keep ordinary methods as ordinary methods.

## Provider scope and instance lifetime

A provider belongs to an injector. Angular searches the current injector and then its ancestors for a provider for the requested token. The closest matching provider wins.

### Root scope

A root provider is the usual choice for application-wide services such as an API client or authentication state. Each application root injector has one instance for a given token. Classes decorated with `@Service()` are root-provided by default in current Angular. The classic equivalent is:

~~~ts
@Injectable({ providedIn: 'root' })
export class SessionService {}
~~~

This gives broad availability and allows unused providers to be removed from the production bundle when the application is optimized.

### Component scope

A component can provide a service in its `providers` metadata. The component and its descendants resolve that component-scoped instance. Each instance of the component gets its own service instance; Angular destroys that instance with the owning component.

~~~ts
@Service({ autoProvided: false })
export class DraftStateService {
  // State belongs to one editor instance.
}

@Component({
  selector: 'app-draft-editor',
  providers: [DraftStateService],
  template: '<ng-content />',
})
export class DraftEditorComponent {}
~~~

Choose this scope for deliberately isolated state, such as a draft owned by one editor, a wizard session, or state that should be cleared when a panel closes. Do not provide a shared root state service again on each child component: the nearer provider shadows the root provider and creates separate state.

### Route scope

Routes can also declare providers. A route-scoped service is available to that route's components, guards, and resolvers. This is useful for feature-specific dependencies or feature state that should be separate from unrelated routes. Route scopes are a separate injector boundary; they are not equivalent to a component's local provider.

### Remember the singleton rule accurately

A class is not automatically a process-wide singleton. Angular caches an instance per token in the injector that provides it. If a token is provided in the root and again in a child injector, a request below that child can resolve a different instance. Consider the provider location whenever two components unexpectedly see different state.

## Services can depend on other services

Services can be composed into layers. A component might depend on a feature service; that service might depend on an API client, a logger, and a configuration token. Angular resolves each dependency through DI, so callers do not need to construct the entire dependency graph themselves.

Keep dependencies pointed in a sensible direction. For example, a component can call a catalog service, and the catalog service can call an API client. The API client generally should not reach upward and manipulate a particular page component. This keeps the service usable from other components and easier to replace in tests.

Be alert for circular dependencies: if `A` depends on `B` and `B` depends on `A`, neither has a clean construction order. Move shared behavior into a third class or change the responsibilities so dependencies point in one direction.

## Testing and replacement

A service's provider boundary gives tests a place to supply a fake implementation. A component can then be tested against the behavior it needs without making real network requests or relying on unrelated application systems. For example, a test can provide a fake `ProductApi` that returns a known product list, then check how the catalog service handles it.

Prefer testing observable behavior and public methods over reaching into private implementation details. Keep a test double aligned with the token and methods the consumer actually uses.

## Common mistakes and how to fix them

### `NG0201: No provider found`

Angular could not find a provider for the requested token. Confirm that the service has automatic provision configured, or register it in the correct application, route, or component `providers` array. Check that the class imported by the consumer is the same token that was provided.

### `@Injectable()` was added, but injection still fails

`@Injectable()` without a `providedIn` value does not automatically put the class in an injector. Add a provider at the scope where it belongs, or use `@Injectable({ providedIn: 'root' })` for a root service. On Angular v22+, a root singleton using `inject()` can use `@Service()`.

### A service unexpectedly has duplicate state

Look for the same token in a component or route `providers` array in addition to a root provider. The nearer provider creates a local instance. Remove the extra provider if the feature is supposed to share the root instance; keep it if isolation is deliberate.

### `inject()` throws outside an injection context

Move the `inject()` call to a field initializer or constructor, or use a documented injection-context API if the architecture truly requires a supplied injector. Do not call `inject()` each time a normal method runs.

### Constructor parameters on `@Service()` do not behave as expected

Current `@Service()` is designed for `inject()` dependencies, not constructor-based DI. Change the dependencies to field initializers with `inject()`, or use `@Injectable()` when constructor injection is preferred or required.

### The `@Service` import is missing in an older project

`@Service()` is part of the current Angular v22 API. For a project using an earlier Angular release, use `@Injectable({ providedIn: 'root' })` and the DI features supported by that installed version. Do not copy a snippet from the current docs into an older app without checking the installed Angular version.

## Practical checklist

Before adding a service, ask:

- Does this behavior need to be shared, independently tested, or separated from view code?
- Is the class focused on one responsibility?
- Which Angular version is this project using, and should the class use `@Service` or `@Injectable`?
- Does the service need root, route, or component scope?
- Could a child provider accidentally shadow the intended shared instance?
- Are `inject()` calls limited to an injection context?
- Can a consumer receive a fake provider in a test?

## Key takeaways

- Services are classes for reusable behavior or state; DI providers tell Angular how to supply them.
- In current Angular v22+, use `@Service()` for a simple root-provided singleton that uses `inject()`.
- Use `@Injectable()` when constructor injection, advanced provider keys, or non-root `providedIn` scopes are needed; it is also the compatible pattern for earlier Angular releases.
- `inject()` only works in an injection context, not in an ordinary method called later.
- Provider location controls visibility, instance reuse, and lifetime; a nearer provider can shadow an ancestor.
- Keep service responsibilities focused and choose scope deliberately.

## Continue learning

- Previous: [Inspect signals and components with Angular DevTools](048-inspect-signals-and-components-with-angular-devtools.md)
- Q&A: [Services and the injectable pattern](../qnas/049-services-and-the-injectable-pattern.md)
- Example: [Services and the injectable pattern](../examples/049-services-and-the-injectable-pattern.md)
- Next: [Inject dependencies with inject and constructor injection](050-inject-dependencies-with-inject-and-constructor-injection.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="048-inspect-signals-and-components-with-angular-devtools.md">048 - Inspect signals and components with Angular DevTools</a></td><td><a href="../qnas/049-services-and-the-injectable-pattern.md">Q&amp;A</a></td><td><a href="../examples/049-services-and-the-injectable-pattern.md">Example</a></td><td><a href="050-inject-dependencies-with-inject-and-constructor-injection.md">050 - Inject dependencies with `inject()` and constructor injection</a></td>
</tr>
  </tbody>
</table>
