# Inject dependencies with `inject()` and constructor injection

Dependency injection lets a class request the values it needs while Angular's injector supplies them. Angular components, directives, and DI-created services can receive dependencies in two common class-based styles: call `inject()` in an injection context, or declare constructor parameters.

Both styles use the same provider system and injector hierarchy. The difference is where the dependency request appears in your class. `inject()` is the modern field-based style used by `@Service()` classes; constructor parameters remain supported and useful, especially with `@Injectable()` services. Choose one style deliberately and understand the injection-context rules before using either.

> **Version note:** Current Angular v22 supports `@Service()` for root-provided services that use `inject()`. It does not support constructor-based dependency injection. Use `@Injectable()` when a service needs constructor parameters. Components can use either style. Projects on earlier Angular versions should follow the APIs supported by their installed version.

## Learning outcomes

After this lesson, you should be able to:

- Explain how DI supplies dependencies instead of a class constructing them itself.
- Inject a class token with `inject()` in a field initializer or constructor body.
- Receive dependencies through constructor parameters.
- Use `InjectionToken` for runtime values and TypeScript interfaces that cannot serve as DI tokens.
- Inject a token with `inject()` or with `@Inject()` on a constructor parameter.
- Handle an optional dependency without incorrectly treating it as guaranteed.
- Recognize valid injection contexts and avoid NG0203.
- Choose between the styles based on readability, version, and provider requirements.

## What happens during injection?

A consumer asks for a token. Angular searches the applicable injector hierarchy for a provider registered for that token. If it finds a provider, Angular returns the value or creates the instance according to that provider. A class token is common, but tokens can represent configuration objects, functions, or other runtime values too.

For example, a component does not need to construct an API client itself:

~~~ts
@Component({
  selector: 'app-catalog-page',
  template: '<h1>Catalog</h1>',
})
export class CatalogPageComponent {
  private readonly catalog = inject(ProductCatalogService);
}
~~~

The `ProductCatalogService` token must have a provider in an injector visible to this component. The component class also needs to import `Component` and `inject` from `@angular/core`, and the service must be imported from its source file.

The injector manages creation and reuse. A consumer should not call `new ProductCatalogService()` when it expects Angular-managed dependencies, provider overrides, or lifecycle scope.

## Style 1: use `inject()` in a field initializer

A field initializer is a concise place to request dependencies from Angular:

~~~ts
import { Component, inject } from '@angular/core';
import { ProductCatalogService } from './product-catalog.service';
import { Router } from '@angular/router';

@Component({
  selector: 'app-catalog-page',
  template: '<h1>Catalog</h1>',
})
export class CatalogPageComponent {
  private readonly catalog = inject(ProductCatalogService);
  private readonly router = inject(Router);

  openProduct(id: number): void {
    this.router.navigate(['/products', id]);
    this.catalog.recordVisit(id);
  }
}
~~~

The dependencies are stored as fields, so ordinary methods can use them later. Do not call `inject()` again inside `openProduct`; the event handler runs after Angular has created the component.

Field injection is also useful in services. Current Angular v22's `@Service()` pattern expects dependencies to be obtained using `inject()`:

~~~ts
import { inject, Service } from '@angular/core';

@Service()
export class ProductCatalogService {
  private readonly api = inject(ProductApiService);
  private readonly logger = inject(AuditLoggerService);

  recordVisit(id: number): void {
    this.logger.info('Visited product ' + id);
    this.api.recordVisit(id);
  }
}
~~~

The `@Service()` decorator provides a root service in current Angular. Use `@Injectable()` instead when the service needs constructor injection or another capability covered by the previous lesson.

## Style 2: use constructor parameters

Constructor injection lists dependencies in the constructor signature. It works in components, directives, and services with the appropriate Angular decorator and provider:

~~~ts
import { Component } from '@angular/core';
import { ProductCatalogService } from './product-catalog.service';
import { Router } from '@angular/router';

@Component({
  selector: 'app-catalog-page',
  template: '<h1>Catalog</h1>',
})
export class CatalogPageComponent {
  constructor(
    private readonly catalog: ProductCatalogService,
    private readonly router: Router,
  ) {}

  openProduct(id: number): void {
    this.router.navigate(['/products', id]);
    this.catalog.recordVisit(id);
  }
}
~~~

TypeScript parameter properties declare and initialize the fields for you. The equivalent longer form declares fields and assigns the constructor arguments manually:

~~~ts
export class CatalogPageComponent {
  private readonly catalog: ProductCatalogService;
  private readonly router: Router;

  constructor(catalog: ProductCatalogService, router: Router) {
    this.catalog = catalog;
    this.router = router;
  }
}
~~~

Constructor injection is useful when you want the dependency list visible at the class boundary, when the class uses a constructor to initialize other state, or when a service uses `@Injectable()` and constructor-based DI. The Angular CLI's compiler metadata allows Angular to resolve constructor parameter types for normal class tokens.

Do not add a constructor just to call `inject()` unless there is a reason. A field initializer is shorter when it is only receiving dependencies. Also avoid re-declaring a constructor if a base class already needs constructor arguments without carefully forwarding them with `super(...)`.

## `inject()` from a constructor body

`inject()` is valid during construction of a class created by Angular. It can therefore be called in a field initializer or inside the constructor body:

~~~ts
@Component({ selector: 'app-example', template: '' })
export class ExampleComponent {
  private readonly router: Router;

  constructor() {
    this.router = inject(Router);
  }
}
~~~

This is legal but often more verbose than a field initializer or constructor parameter. Use it when constructor logic truly needs the resolved value or when you have a specific design reason. Keep the call synchronous and during construction.

## Injecting values that are not classes

TypeScript interfaces and type aliases do not exist at runtime, so Angular cannot use them as provider tokens. Use `InjectionToken<T>` to create a unique runtime token while retaining the TypeScript type:

~~~ts
// src/app/api-config.token.ts
import { InjectionToken } from '@angular/core';

export interface ApiConfig {
  readonly baseUrl: string;
  readonly timeoutMs: number;
}

export const API_CONFIG = new InjectionToken<ApiConfig>('api.config');
~~~

Register a value for that token at an injector boundary, for example in application configuration:

~~~ts
// src/app/app.config.ts
import type { ApplicationConfig } from '@angular/core';
import { API_CONFIG } from './api-config.token';

export const appConfig: ApplicationConfig = {
  providers: [
    {
      provide: API_CONFIG,
      useValue: {
        baseUrl: 'https://api.example.test',
        timeoutMs: 5000,
      },
    },
  ],
};
~~~

Pass this configuration to the standalone app bootstrap so the token has a provider:

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig);
~~~

With field injection, request the token directly:

~~~ts
import { inject, Service } from '@angular/core';
import { API_CONFIG } from './api-config.token';

@Service()
export class ProductApiService {
  private readonly config = inject(API_CONFIG);

  productsUrl(): string {
    return this.config.baseUrl + '/products';
  }
}
~~~

With constructor injection, use `@Inject()` to associate a parameter with the runtime token:

~~~ts
import { Inject, Injectable } from '@angular/core';
import { API_CONFIG } from './api-config.token';
import type { ApiConfig } from './api-config.token';

@Injectable({ providedIn: 'root' })
export class ProductApiService {
  constructor(@Inject(API_CONFIG) private readonly config: ApiConfig) {}

  productsUrl(): string {
    return this.config.baseUrl + '/products';
  }
}
~~~

The string passed to `new InjectionToken()` is a debugging label, not the identity of the token. Export one token object and import that same object wherever it is provided or injected. Creating a second token with the same label makes a different token that Angular cannot resolve from the first provider.

## Optional dependencies

If a dependency is genuinely optional, request it as optional and account for the missing value. With `inject()`, the result can be `null`:

~~~ts
readonly analytics = inject(AnalyticsService, { optional: true });

trackPurchase(orderId: string): void {
  this.analytics?.trackPurchase(orderId);
}
~~~

With constructor injection, use `@Optional()` and declare the possibility of `null` in the type:

~~~ts
import { Component, Optional } from '@angular/core';

@Component({ selector: 'app-checkout', template: '' })
export class CheckoutComponent {
  constructor(
    @Optional() private readonly analytics: AnalyticsService | null,
  ) {}
}
~~~

Optional injection is useful for a capability that can be absent by design. It should not hide a required service that was accidentally left unprovided. A missing required token should be fixed at the provider configuration, not silently treated as optional.

## Injection context: where `inject()` works

Angular needs an active injector to resolve `inject()`. The common contexts are:

- A field initializer for a component, directive, or DI-created service.
- The constructor body of a class being created by Angular.
- A provider factory or an `InjectionToken` factory.
- A framework callback that Angular runs in an injection context, such as a functional route guard.

This works:

~~~ts
export class SearchComponent {
  private readonly search = inject(SearchService);

  submit(query: string): void {
    this.search.run(query);
  }
}
~~~

This usually fails with NG0203 because the event handler runs after construction:

~~~ts
submit(query: string): void {
  const search = inject(SearchService); // No active injection context here.
  search.run(query);
}
~~~

The same restriction applies to lifecycle hooks such as `ngOnInit`. Inject dependencies before those methods run, then call the stored dependency from the hook or event handler. Advanced code can use `runInInjectionContext()` when it has an `EnvironmentInjector` and a real need to execute a callback with that injector; ordinary components rarely need this.

## Choose one style for a class

Both styles are supported and they use the same provider resolution. Use the one that makes the class easiest to understand:

| Consideration | `inject()` field | Constructor parameter |
| --- | --- | --- |
| Dependencies visible in one place | As fields near the class logic | In the constructor signature |
| Boilerplate | Low | Low with parameter properties; more if assigning manually |
| Works with `@Service()` | Yes, this is the supported style | No |
| Works with `@Injectable()` | Yes | Yes |
| Non-class token | `inject(TOKEN)` | `@Inject(TOKEN)` on the parameter |
| Optional dependency | `{ optional: true }`, result may be `null` | `@Optional()`, parameter may be `null` |

Do not mix constructor parameters and `inject()` fields casually in the same class. There are valid advanced cases, but an inconsistent mixture makes dependencies harder to scan. Keep a codebase's conventions consistent and follow the requirements of the decorator in use.

## Resolution modifiers in brief

When the same token can be available at different injector levels, Angular can constrain where a particular request searches:

- `optional` returns `null` when no provider is found.
- `self` checks only the current injector.
- `skipSelf` begins searching at the parent injector.
- `host` limits the search to the host boundary for element injectors.

With `inject()`, these are options on the call, such as `inject(FeatureState, { optional: true, skipSelf: true })`. Constructor injection uses parameter decorators such as `@Optional()` and `@SkipSelf()`. These options alter resolution for that request; they do not register providers or change the provider's scope. The next lesson explores injector hierarchy and resolution in more depth.

## Common mistakes

### Calling `inject()` from a method or lifecycle hook

Move the dependency request to a field initializer or constructor, then use the stored value from methods. An ordinary method does not retain the construction-time injection context.

### Using an interface as the token

An interface is erased when TypeScript emits JavaScript. Create and export an `InjectionToken<YourInterface>` and register/inject that same token object.

### Using `@Inject()` as if it registered a provider

`@Inject(TOKEN)` tells constructor injection which token a parameter requests. A provider for that token must still exist in an injector visible to the consumer.

### Forgetting `@Inject()` for a non-class token in a constructor

The parameter's TypeScript type may be an interface or primitive that does not identify a runtime token. Add `@Inject(TOKEN)` to connect it to an `InjectionToken` or another explicit token.

### Treating an optional dependency as non-null

With optional injection, handle the `null` result in both the TypeScript type and runtime logic. If the dependency is actually required, register it instead of making it optional to suppress a provider error.

### Creating dependencies with `new`

Manual construction bypasses the injector for that instance. Angular will not provide its dependencies, apply the provider selected for that injector, or manage its provider scope. Let Angular create DI-managed services and components.

### Mixing two styles without purpose

Using constructor parameters for some dependencies and `inject()` fields for others can obscure the class boundary. Pick one style for the class unless a concrete framework or inheritance requirement calls for a mix.

## Practical checklist

Before injecting a dependency, check:

- Is there a provider for this token in a visible injector?
- Is this a class token, or do I need an `InjectionToken`?
- Does the decorator support the style I chose? `@Service()` requires `inject()`; `@Injectable()` supports both.
- If using `inject()`, is the call in a valid injection context?
- If the dependency is optional, does the type and code handle `null`?
- Would constructor parameters or field injection make this class clearer and more consistent?
- Am I asking for a separate service instance because of an injector override?

## Key takeaways

- DI supplies dependencies from providers; a consumer requests a token rather than constructing the dependency itself.
- Use `inject()` in a field initializer or constructor context, then use the stored field in normal methods.
- Constructor injection remains supported for components and `@Injectable()` services. `@Service()` services use `inject()`.
- TypeScript interfaces are not runtime tokens; use `InjectionToken<T>` and share the same token object between provider and consumer.
- Optional injection can return `null`, so handle it explicitly.
- Both styles use the same injector hierarchy; changing injection syntax does not change provider scope.

## Continue learning

- Previous: [Services and the injectable pattern](049-services-and-the-injectable-pattern.md)
- Q&A: [Inject dependencies with inject and constructor injection](../qnas/050-inject-dependencies-with-inject-and-constructor-injection.md)
- Example: [Inject dependencies with inject and constructor injection](../examples/050-inject-dependencies-with-inject-and-constructor-injection.md)
- Next: [Hierarchical dependency injection](051-hierarchical-dependency-injection.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="049-services-and-the-injectable-pattern.md">049 - Services and the injectable pattern</a></td><td><a href="../qnas/050-inject-dependencies-with-inject-and-constructor-injection.md">Q&amp;A</a></td><td><a href="../examples/050-inject-dependencies-with-inject-and-constructor-injection.md">Example</a></td><td><a href="051-hierarchical-dependency-injection.md">051 - Hierarchical dependency injection</a></td>
</tr>
  </tbody>
</table>
