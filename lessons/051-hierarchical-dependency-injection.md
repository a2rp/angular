# Hierarchical dependency injection

Dependency injection (DI) in Angular is hierarchical. A provider belongs to an injector, and an injection request searches a defined path through injectors until it finds a provider for the requested token. Where you register a provider determines which parts of the application can use it, which instance they receive, and when that instance is destroyed.

This lesson focuses on injector scope and lookup. It builds on [injecting dependencies with inject() and constructor injection](050-inject-dependencies-with-inject-and-constructor-injection.md); the next lesson covers application and environment providers in more detail.

## Learning outcomes

After this lesson, you should be able to:

- Describe Angular’s EnvironmentInjector and ElementInjector hierarchies.
- Choose an application, route, component, or directive scope for a provider.
- Trace how Angular resolves a token from the requesting component upward.
- Explain how a nearer provider shadows a provider higher in the hierarchy.
- Predict when two components share a service and when they get separate instances.
- Choose between providers and viewProviders.
- Use optional, self, skipSelf, and host to adjust a lookup.
- Diagnose common provider-scope errors.

## An injector is a scoped registry

A provider connects a runtime token to the value or class Angular should return when code requests that token. Providers live in injectors. An application has multiple injectors arranged in parent–child hierarchies; a component’s request can see its own injector and the allowed ancestors, but it cannot search down into its children.

Think of a request as asking: “Starting here, which visible injector first provides this token?” If the same token appears more than once along the search path, the closest matching provider wins.

## Angular’s two main injector hierarchies

### EnvironmentInjector

The EnvironmentInjector hierarchy is for providers associated with application or feature environments, rather than one particular rendered element. The root EnvironmentInjector is created during application bootstrap. Root-provided services and application-level providers are available to the application’s component trees.

Current Angular supports @Service() for a root-provided service. An application can also register providers during bootstrap. Route configuration can provide services for a route and the components, guards, and resolvers in that route. These are environment-level scopes; chapter 052 explores their configuration and use.

~~~ts
import { Service } from '@angular/core';

@Service()
export class LoggingService {
  log(message: string): void {
    console.log(message);
  }
}
~~~

For an app-wide provider configured at bootstrap:

~~~ts
bootstrapApplication(AppComponent, {
  providers: [LoggingService],
});
~~~

In existing applications using @Injectable({ providedIn: 'root' }), the same root-scope idea applies. NgModule-based applications also have a ModuleInjector hierarchy. ModuleInjector is a legacy-architecture detail: imported module providers are flattened into that hierarchy, and lazy-loaded NgModules can introduce child ModuleInjectors.

### ElementInjector

Angular creates an ElementInjector for each element in the rendered component/directive tree. It is empty unless a component or directive registers providers or viewProviders. A provider declared on a component belongs to that component instance’s injector.

~~~ts
import { Component, Service } from '@angular/core';

@Service({ autoProvided: false })
export class EditorSession {
  private draftText = '';

  updateDraft(text: string): void {
    this.draftText = text;
  }

  readDraft(): string {
    return this.draftText;
  }
}

@Component({
  selector: 'app-editor',
  providers: [EditorSession],
  template: '<app-editor-toolbar />',
})
export class EditorComponent {}
~~~

Every EditorComponent instance gets its own EditorSession. Components and directives inside that editor can resolve the instance from the editor’s injector. Another editor instance gets another instance. When an editor is destroyed, its component-scoped service instance is destroyed with that scope.

This scope is useful for state that belongs to one view, such as a form session, a temporary wizard, an editor tab, or a modal. It also prevents unrelated parts of the application from accidentally sharing that local state.

## How Angular resolves a request

For a dependency requested by a component or directive, Angular:

1. Checks the requester’s ElementInjector.
2. Walks up the relevant ElementInjector ancestry until it finds a provider or reaches the top of that hierarchy.
3. If none was found, checks the EnvironmentInjector hierarchy for the request.
4. Throws a no-provider error if no visible injector provides the token.

A simplified picture:

~~~text
Application root EnvironmentInjector
└── Feature EnvironmentInjector (when configured)
    └── Component tree
        ├── ElementInjector: app-root
        │   └── ElementInjector: app-editor (EditorSession)
        │       ├── ElementInjector: app-toolbar
        │       └── ElementInjector: app-preview
        └── ElementInjector: another app-editor (different EditorSession)
~~~

The tree is logical, not necessarily a literal map of every browser DOM node. Component templates and projected content have Angular-specific visibility rules, so use Angular’s logical view model when reasoning about component providers.

### The closest provider wins

Suppose ApiClient is provided in the root EnvironmentInjector and a particular component also provides ApiClient. That component and its descendants resolve the component-scoped provider. Other components without a nearer provider resolve the root one.

Angular returns the first matching provider; it does not merge providers from different levels. If you need multiple values for one token, configure a multi provider as such. Avoid registering the same token on multiple directives attached to one element: Angular documents that which same-element directive provider wins is undefined.

### Parent components cannot inject from their children

Lookup travels from the requesting element toward ancestors. If ChildComponent provides a service, its parent cannot inject that child’s instance. Move the provider to a shared ancestor if both need one shared instance, or keep the provider local if the state belongs only to the child subtree.

## Choosing a provider scope

| Scope | Typical registration | Who can resolve it | Good fit |
| --- | --- | --- | --- |
| Application root | Root service or bootstrap providers | Application component trees, unless shadowed | Logging, shared API access, app-wide configuration |
| Route or feature environment | A route’s providers | Components and route guards/resolvers in that route | Feature-only state or route-specific configuration |
| Component | Component’s providers | That component and eligible descendants | Isolated form, editor, modal, or wizard state |
| Directive/element | Directive’s providers | The directive’s element and eligible descendants | Behavior or state owned by one host element |

Choose the narrowest scope that matches the intended sharing boundary. A global service is easy to share, but every consumer then shares its state. A local service makes isolation explicit, but it is not visible to the parent or unrelated siblings.

## providers and viewProviders

Both properties configure providers for a component’s element/view injector. Their difference matters when the component uses content projection.

- providers are available to the component and eligible descendants, including projected content.
- viewProviders are available inside the component’s own view, but not to content projected through ng-content.

Use providers when projected content is meant to participate in the component’s provider scope. Use viewProviders when the dependency is an implementation detail of the component’s own template and should not be visible to projected content.

Do not treat viewProviders as a general “more private service” mechanism. Its special behavior concerns visibility across the component view/content-projection boundary. Check the actual component tree when a projected child resolves a different instance than expected.

## Resolution modifiers

The inject() options can change where a request starts, where it stops, or what happens when no provider is found:

| Option | Effect |
| --- | --- |
| optional: true | Return null if the token cannot be resolved instead of throwing. Account for null in the type and code. |
| self: true | Check only the current element injector. Often paired with optional: true when the provider may be absent. |
| skipSelf: true | Start at the parent injector, ignoring a provider on the requesting element. |
| host: true | Stop the search at the host component boundary. This is useful for controlling visibility across component views. |

~~~ts
import { inject } from '@angular/core';

const optionalParent = inject(EditorSession, {
  optional: true,
  skipSelf: true,
});
~~~

The options can be combined where their meanings are compatible. Do not combine self with skipSelf, or self with host. The decorators @Optional(), @Self(), @SkipSelf(), and @Host() are the corresponding options used with constructor injection in projects that use that style.

These modifiers do not register providers. They only adjust a particular lookup. If a required service is missing, prefer fixing its provider scope over suppressing the error with optional.

## Provider scope determines instance identity

A “singleton” means one instance per injector scope, not necessarily one instance for all time or every context. A root provider normally gives consumers in that application the same root instance, unless a nearer provider overrides it. A component provider gives each component instance its own service instance. A route provider is associated with its route environment.

When debugging unexpected state:

1. Identify the exact token object being injected.
2. Find every provider registration for that token.
3. Start from the component requesting it and walk upward.
4. Find the first provider on that path.
5. Check whether the consumer is inside a component view or projected content.
6. Check whether a modifier changes the search path.
7. Check whether the component or route was recreated, which can create a new scoped instance.

A service being available in one subtree does not make it available to its parent, sibling subtree, or another application. Route-scoped providers can also have their own lifetime behavior; do not infer cleanup timing from component-scoped providers. Consult the router behavior configured by the application.

## Common mistakes

### Providing a local service and expecting the root instance

A component provider shadows a root provider for that subtree. Remove the local registration if all consumers should share root state, or keep it if the local override is intentional.

### Providing a service on a child and injecting it from the parent

Parents cannot search downward. Move the provider to the parent or another shared ancestor when both sides need the same instance.

### Registering the same stateful service at multiple levels

Each provider registration can produce a separate instance. A descendant may mutate a local copy while another part of the application reads the root copy. Decide whether the feature should share state or isolate it, and register the provider at that boundary.

### Using viewProviders when projected content needs the dependency

Projected content cannot see a provider declared only in viewProviders. Use providers if the projected content is meant to resolve that provider.

### Using optional just to hide a missing provider

optional makes absence part of the result and returns null. Handle that state deliberately. If the dependency is required, provide it at the correct level.

### Using a different InjectionToken object with the same description

InjectionToken identity is based on the token object, not its debug description. Export one token object and import that same object wherever it is provided or injected.

## Practical checklist

Before adding a provider, ask:

- Who should share this instance: the whole application, a route, one component tree, or one element?
- Does this state need to be isolated between repeated component instances?
- Can the requesting component actually see the provider by walking upward?
- Is there a nearer provider that shadows the one I expected?
- Is the request inside the component view or projected content?
- Would optional, self, skipSelf, or host clarify the intended lookup?
- Does the service’s lifetime match the scope where it is registered?

## Key takeaways

- Angular organizes DI through EnvironmentInjector and ElementInjector hierarchies; NgModule applications also have ModuleInjector.
- Component and directive providers configure an element-level scope. Each component instance can own a distinct service instance.
- Angular searches from the requesting element upward; the closest matching provider wins.
- If no ElementInjector provides the token, Angular checks the applicable EnvironmentInjector hierarchy.
- Component-scoped providers are destroyed with their component; route and environment scopes have their own lifecycle rules.
- providers and viewProviders differ at the content-projection boundary.
- Resolution modifiers adjust one lookup; they do not create providers or change the scope of other requests.

## Continue learning

- Previous: [Inject dependencies with inject() and constructor injection](050-inject-dependencies-with-inject-and-constructor-injection.md)
- Q&A: [Hierarchical dependency injection](../qnas/051-hierarchical-dependency-injection.md)
- Example: [Hierarchical dependency injection](../examples/051-hierarchical-dependency-injection.md)
- Next: [Application and environment providers](052-application-and-environment-providers.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="050-inject-dependencies-with-inject-and-constructor-injection.md">050 - Inject dependencies with `inject()` and constructor injection</a></td><td><a href="../qnas/051-hierarchical-dependency-injection.md">Q&amp;A</a></td><td><a href="../examples/051-hierarchical-dependency-injection.md">Example</a></td><td><a href="052-application-and-environment-providers.md">052 - Application and environment providers</a></td>
</tr>
  </tbody>
</table>
