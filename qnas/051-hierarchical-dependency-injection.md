# Q&A: Hierarchical dependency injection

These questions explain how Angular chooses a provider when a component, directive, or service requests a token. For the full mental model and a runnable example, see the [lesson](../lessons/051-hierarchical-dependency-injection.md) and [example](../examples/051-hierarchical-dependency-injection.md).

## 1. What does hierarchical dependency injection mean?

Angular stores providers in injectors arranged in parent–child hierarchies. When code requests a token, Angular searches from the requesting context through the applicable ancestors. The location where a provider is registered determines its visibility and instance scope.

## 2. What is the difference between a provider and an injector?

An injector is a registry and resolver. A provider is one registration in that registry that tells Angular what value or class to use for a token. An injector can hold many providers, and a request asks that injector hierarchy to resolve one token.

## 3. What are Angular’s main injector hierarchies?

Modern standalone applications primarily use two: the EnvironmentInjector hierarchy for application or feature environments, and the ElementInjector hierarchy associated with rendered component/directive elements. NgModule-based applications also use a ModuleInjector hierarchy.

## 4. What is the root EnvironmentInjector?

It is the application-level environment created during bootstrap. Root-provided services and bootstrap providers can be resolved by the application’s component trees unless a nearer provider overrides the token.

## 5. When should I use a root provider?

Use root scope when independent parts of the application should normally share one service instance, such as a logging service or a shared API client. Avoid putting state in root when separate feature or component instances should stay isolated.

## 6. What is an ElementInjector?

Angular creates an ElementInjector for each element in its component/directive tree. It is empty unless a component or directive registers providers or viewProviders. A local provider is visible from its own eligible component/directive context and descendants, subject to view and content-projection rules.

## 7. What is a ModuleInjector?

It belongs to the NgModule-based DI architecture. NgModule providers and imported module providers are flattened into the module hierarchy; lazy-loaded NgModules can create child ModuleInjectors. Standalone applications use EnvironmentInjector instead of adding NgModules just to scope providers.

## 8. In what order does Angular resolve a dependency?

For a component or directive request, Angular first searches the requester’s ElementInjector ancestry. If it finds no provider there, it searches the applicable EnvironmentInjector ancestry. NgModule-based applications also involve the ModuleInjector hierarchy. If no visible injector provides the token, resolution fails with a no-provider error.

## 9. If the same token is provided at root and on a component, which one is used?

The nearest provider on the lookup path wins. The component and eligible descendants resolve the component-level value; consumers outside that component subtree can resolve the root value. Angular does not merge providers from different injector levels.

## 10. Can a parent component inject a service provided by a child?

No. Injection searches from the request upward through ancestors, not downward into children. Move the provider to a shared parent or another common ancestor if both need the same instance.

## 11. Do sibling components share a component-scoped provider?

Not when each sibling component declares its own provider. Each component instance has its own injector entry and usually its own service instance. To share one instance between siblings, provide it on a common ancestor or at a broader scope.

## 12. How many instances does a component provider create?

Angular creates an instance for each component instance that registers that provider, when the token is first needed. Its eligible descendants resolve that instance. If there are three independent editor component instances, each can own its own editor state.

## 13. When is a component-scoped service destroyed?

Its instance belongs to the component’s element scope, so it is destroyed when that component is destroyed. This makes component providers useful for temporary state owned by a form, modal, wizard, or editor.

## 14. Are route providers the same as component providers?

No. A route’s providers configure an environment-level scope associated with that route and its components, guards, and resolvers. Component providers configure an element-level scope for that component instance and eligible descendants. Route injector reuse and cleanup can depend on router configuration, so do not assume it behaves exactly like a component lifetime.

## 15. What is the difference between providers and viewProviders?

Both configure provider visibility for a component. providers can be visible to projected content; viewProviders are available inside the component’s own view but not to content projected through ng-content. Choose based on whether projected content should resolve that token.

## 16. Can projected content see a service in viewProviders?

No. Projected content originates outside the component’s own view and cannot see providers registered only in viewProviders. If the projected content is meant to use the provider, consider providers instead.

## 17. What does optional: true do?

It makes a dependency optional. If Angular cannot find the token, inject() returns null instead of throwing a missing-provider error. The code should handle null explicitly; optional does not register a provider.

~~~ts
const analytics = inject(AnalyticsService, { optional: true });

if (analytics) {
  analytics.track('page-opened');
}
~~~

## 18. What does self: true do?

It restricts the search to the current element injector. Angular will not continue to parent element injectors to find the token. Pair it with optional when the provider may be absent, and handle a possible null result.

## 19. What does skipSelf: true do?

It skips the current injector and starts searching at the parent. This is useful when a token is provided locally but a component intentionally needs the parent’s instance. If no parent provider exists, the request fails unless it is also optional.

## 20. What does host: true do?

It sets a boundary for the lookup at the host component view. Angular does not keep searching beyond that boundary. It is useful when a component or directive must limit which ancestor providers its request can see.

## 21. Which resolution modifiers can be combined?

Modifiers can be combined when their meanings are compatible. self cannot be combined with skipSelf or host. optional is often useful with self or skipSelf when the token may not exist in the limited search path.

## 22. Do modifiers create a provider or change the service scope?

No. They affect one injection request’s search behavior. Provider registration still determines where the value is available and which injector owns its instance.

## 23. Why can two consumers receive different instances of the same class?

They may resolve the token from different injectors. A nearer component or route provider can shadow a root provider; repeated component instances can also each own a local instance. Compare the providers along each consumer’s lookup path.

## 24. Why is InjectionToken identity important?

Angular uses the actual InjectionToken object as the runtime key, not just its description string or TypeScript type. Export one token object and import that same object wherever the provider and consumer are declared. Creating two tokens with the same description creates two different tokens.

## 25. Does changing from constructor injection to inject() change the provider hierarchy?

No. Both request a token from the same DI system. Changing syntax does not add a provider, alter its injector, or make a child provider visible to a parent.

## 26. How should I debug a missing or unexpected service?

Start with the exact token object, then find every place it is provided. Begin at the requesting component and walk upward through eligible element injectors; if no provider is found, inspect the environment or module configuration. Check for a nearer override, content projection, route scope, and resolution modifiers. Angular’s DI troubleshooting guide covers common symptoms.

## 27. Where should shared state between two child components be provided?

Provide it in the nearest common ancestor that owns the shared workflow. Both child subtrees can then resolve that ancestor’s instance. If only one child should own the state, provide it on that child instead.

## 28. Should I register every service in the root injector?

No. Choose scope based on the intended sharing boundary. Root scope is appropriate for app-wide shared behavior; route scope fits route-specific features; component scope fits state that should be isolated per component instance.

## Continue learning

- Lesson: [Hierarchical dependency injection](../lessons/051-hierarchical-dependency-injection.md)
- Example: [Hierarchical dependency injection](../examples/051-hierarchical-dependency-injection.md)
- Next lesson: [Application and environment providers](../lessons/052-application-and-environment-providers.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="050-inject-dependencies-with-inject-and-constructor-injection.md">050 - Inject dependencies with `inject()` and constructor injection</a></td><td><a href="../lessons/051-hierarchical-dependency-injection.md">Lesson</a></td><td><a href="../examples/051-hierarchical-dependency-injection.md">Example</a></td><td><a href="052-application-and-environment-providers.md">052 - Application and environment providers</a></td>
</tr>
  </tbody>
</table>
