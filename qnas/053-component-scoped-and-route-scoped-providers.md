# Q&A: Component-scoped and route-scoped providers

These answers explain how component and route providers control dependency visibility and service instances. See the [lesson](../lessons/053-component-scoped-and-route-scoped-providers.md) for the full guide and the [example](../examples/053-component-scoped-and-route-scoped-providers.md) for a complete setup.

## 1. What does provider scope mean?

Provider scope is the part of the application injector hierarchy where a dependency is registered and can be resolved. A component provider belongs to an element injector; a route provider belongs to an environment injector created for that route and its children.

## 2. What are Angular's two injector hierarchies?

The ElementInjector hierarchy follows components and directives in the rendered tree. The EnvironmentInjector hierarchy contains application and route environments. Angular checks element injectors first, then environment injectors, and uses the first provider it finds.

## 3. What does Component.providers configure?

It configures the ElementInjector for that component instance. The component and eligible descendants can inject those dependencies. The provider is not available upward to the component's parent or sideways to its siblings.

~~~ts
@Component({
  providers: [EditorDraft],
  // ...
})
export class EditorComponent {}
~~~

## 4. Does each instance of a component get a separate service?

Yes, when the component's providers array registers the service. Each component instance has its own provider instance, so two editor components can hold separate drafts.

## 5. When is a component-provided instance destroyed?

The instance belongs to the component injector. Angular destroys it when that component instance is destroyed. If the component stays mounted or is reused, its instance stays with it.

## 6. Can the parent inject a service provided by a child component?

No. Angular resolves dependencies upward through the injector hierarchy. It does not search down from a parent into a child injector. Move the provider to a common ancestor if the parent and child must share it.

## 7. Can sibling components share a provider declared on one sibling?

No. A sibling's injector is not in the other sibling's ancestor path. Put the provider on their parent, an appropriate route, or the application environment depending on who owns the shared state.

## 8. Can descendants of a component use its providers?

Yes, eligible descendants can resolve providers from ancestor component injectors. Angular finds the nearest matching provider while walking upward.

## 9. What is the difference between providers and viewProviders?

Both configure a component's ElementInjector. The component and its view descendants can use either. Projected content can see providers from the receiving component but cannot see its viewProviders.

Use viewProviders for dependencies that belong only to the component's own template implementation.

## 10. Why can projected content not see viewProviders?

Projected content is declared by its parent and keeps its logical declaration context when rendered inside ng-content. viewProviders are restricted to the receiving component's own view.

## 11. Can a directive have providers?

Yes. A directive can declare providers. Directives and components on the same element share that element's injector. Avoid putting different competing providers for the same token on directives attached to one element.

## 12. What does Route.providers do?

It registers dependencies in an EnvironmentInjector associated with that route. Angular uses that injector for the route and its child routes.

~~~ts
{
  path: 'admin',
  providers: [AdminWorkspace],
  component: AdminPage,
}
~~~

## 13. Can child routes inject a parent route's providers?

Yes. The route EnvironmentInjector is used for the route and its children. This makes a parent route a useful place for dependencies shared by a whole feature section.

## 14. Can an unrelated sibling route inject a route-scoped service?

No, it cannot resolve the service from another sibling route's injector. It can still resolve a separate provider for the same token from one of its own ancestors, such as the application environment.

## 15. Can a route guard or resolver inject a route-scoped provider?

Yes. Route providers are available to the route's components, guards, and resolvers. A functional guard can call inject() for a service registered on that route.

~~~ts
export const adminGuard: CanActivateFn = () =>
  inject(AdminWorkspace).canOpen();
~~~

A guard still needs to return a valid guard result such as a boolean, UrlTree, or RedirectCommand, as appropriate for the guard type.

## 16. Does a route provider guarantee a new service instance on every navigation?

No. Route providers scope dependencies to a route feature. The Router manages route injectors with its route configuration and reuse behavior. Do not treat a route provider as a guarantee of a fresh per-navigation or per-parameter instance.

## 17. What happens if a token is provided at several levels?

The closest provider in the consumer's lookup path wins. A local component provider can shadow a route provider, and a route provider can shadow an application provider for that route subtree.

Different consumers may therefore get different instances of the same service token.

## 18. Does a component provider replace an @Service() or @Injectable root provider?

For consumers below that component, the local provider is closer and wins. Consumers outside that component subtree continue resolving the application provider, if one exists.

Avoid duplicate registrations unless a local override is intentional.

## 19. How should I mark a service that will only be manually provided?

In current Angular, use @Service({ autoProvided: false }) and add the class to the appropriate component or route providers array. In projects using @Injectable, use @Injectable() without providedIn and register the class manually.

~~~ts
@Service({ autoProvided: false })
export class EditorDraft {}
~~~

## 20. When should state use component scope?

Use component scope when each rendered instance owns independent state, such as a form draft, dialog state, or reusable editor state. Place the provider on the component that owns that state.

## 21. When should I choose route scope?

Use route scope when a feature and its child routes need shared configuration or state, including route components, guards, and resolvers. Put the provider on the narrowest parent route shared by those consumers.

## 22. When should I choose application scope instead?

Use application scope when unrelated application areas need the same app-wide dependency or framework configuration, such as router setup, HTTP configuration, or a global logger. See the previous [application and environment providers lesson](../lessons/052-application-and-environment-providers.md).

## 23. Is a route provider a security boundary?

No. It controls dependency visibility inside the running application. It does not secure backend data or replace server-side authorization. A client-side route guard can improve navigation behavior, but the server must enforce access to protected resources.

## 24. How do Self, SkipSelf, Host, and Optional affect scope?

They are resolution modifiers. They constrain where Angular looks or whether a missing dependency should produce an error. They do not register a provider and do not make a child provider visible to its parent.

Use them only when the dependency contract needs those lookup rules.

## 25. How can I debug an unexpected instance or missing provider?

Check the token, every place it is registered, and the consumer's path through both injector hierarchies. Then check for a nearer override or a viewProviders projection boundary. Angular DevTools' Injector Tree can display element and environment injector hierarchies.

A NullInjectorError means Angular could not find the requested token in the consumer's lookup path.

## 26. How can a test provide a scoped dependency?

Configure the same scope that production uses. For a component, use TestBed's component provider configuration or override the component metadata. For a routed feature, configure the route with its providers and use the Router in the test. If testing only the consuming class, TestBed providers can supply a focused substitute.

~~~ts
TestBed.configureTestingModule({
  providers: [
    { provide: AdminWorkspace, useValue: fakeWorkspace },
  ],
});
~~~

A root test provider is a convenient substitute for an isolated unit test, but it does not prove that the production component or route has the correct provider scope.

## Further reading

- [The lesson](../lessons/053-component-scoped-and-route-scoped-providers.md)
- [The example](../examples/053-component-scoped-and-route-scoped-providers.md)
- [Hierarchical dependency injection](https://angular.dev/guide/di/hierarchical-dependency-injection)
- [Defining dependency providers](https://angular.dev/guide/di/defining-dependency-providers)
- [Route-level providers](https://angular.dev/guide/routing/define-routes#route-level-providers-for-dependency-injection)
- [Route API](https://angular.dev/api/router/Route)
- [Debugging and troubleshooting DI](https://angular.dev/guide/di/debugging-and-troubleshooting-di)
