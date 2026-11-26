# Component-scoped and route-scoped providers

Application providers are useful when the whole application needs a dependency. Many dependencies belong to a smaller area: one component tree or one route feature. Angular lets you register providers at both levels so that state, configuration, and service instances stay close to the code that uses them.

This lesson compares component providers with route providers, explains how Angular resolves a dependency through its injector hierarchy, and shows how to choose a scope without accidentally creating a global service or an isolated copy.

## Learning outcomes

After this lesson, you should be able to:

- Explain the difference between an ElementInjector and an EnvironmentInjector.
- Register a service in a component or directive.
- Choose between **providers** and **viewProviders**.
- Register services and values on a route.
- Explain which descendants, guards, and resolvers can use a route provider.
- Describe how a nearer provider shadows a provider farther up the hierarchy.
- Choose a scope based on ownership and lifetime.

## Two hierarchies resolve dependencies

Angular has two injector hierarchies that matter in a standalone application:

- **ElementInjector hierarchy:** component and directive injectors associated with elements in the rendered component tree. A component's providers and viewProviders configure its ElementInjector.
- **EnvironmentInjector hierarchy:** application and feature environments. The root EnvironmentInjector is configured during bootstrap. Route configuration can create another EnvironmentInjector for a route and its children.

When a component or directive requests a dependency, Angular first checks its ElementInjector and walks through element ancestors. If it still has not found a provider, Angular continues through the EnvironmentInjector hierarchy. Resolution stops at the first matching provider.

That means a provider can be visible below its registration point, but a child provider is not visible upward to its parent or sideways to a sibling. Register a provider at the narrowest common ancestor that owns the state and whose consumers need to share it.

## Component-scoped providers

A component's providers array registers dependencies in that component instance's ElementInjector. The component and eligible descendants can inject them. A separate instance of the same component gets its own provider instance.

~~~ts
import { Component, Service, inject, signal } from '@angular/core';

@Service({ autoProvided: false })
export class EditorDraft {
  readonly text = signal('');

  update(value: string): void {
    this.text.set(value);
  }
}

@Component({
  selector: 'app-editor',
  standalone: true,
  providers: [EditorDraft],
  templateUrl: './editor.component.html',
})
export class EditorComponent {
  protected readonly draft = inject(EditorDraft);
}
~~~

~~~html
<!-- editor.component.html -->
<label>
  Draft
  <input
    [value]="draft.text()"
    (input)="draft.update($any($event.target).value)"
  />
</label>
<p>Current draft: {{ draft.text() || 'Empty' }}</p>
~~~

Each <app-editor> instance gets a separate EditorDraft. Editing one draft does not change the other. This is useful for reusable forms, dialogs, widgets, or editors that own independent state.

A component provider is also visible to components and directives below it, subject to Angular's content-projection visibility rules. The service instance is associated with that component instance and is destroyed when that component is destroyed. It is not available to the component's parent or to a sibling outside that subtree.

The same providers option is available on directives. Directives on the same element share that element's injector, so avoid relying on the order of competing providers for the same token on one element.

### providers and viewProviders

Both options register dependencies in a component's ElementInjector, but their visibility differs:

- **providers** can be used by the component, its view descendants, and projected content that resolves through the component's injector.
- **viewProviders** can be used by the component and its own view descendants. Projected content supplied by a parent cannot see these providers.

Use providers for a dependency that projected content may need. Use viewProviders when the dependency is an implementation detail of the component's own template.

~~~ts
@Component({
  selector: 'app-panel',
  standalone: true,
  viewProviders: [PanelInternalState],
  templateUrl: './panel.component.html',
})
export class PanelComponent {
  protected readonly state = inject(PanelInternalState);
}
~~~

Content projection changes where the projected component is declared in the logical injector tree. A child written by the parent and placed inside <ng-content> does not gain access to the receiving component's viewProviders.

## Route-scoped providers

A route's providers array registers dependencies for that route and its child routes. The Router creates an EnvironmentInjector for that route and uses it for the route and its children. Components, directives, guards, and resolvers associated with the route can resolve those providers.

~~~ts
import { InjectionToken } from '@angular/core';
import type { Routes } from '@angular/router';
import { AdminWorkspace } from './admin-workspace';
import { adminGuard } from './admin.guard';
import { AdminPage } from './admin-page';

export const ADMIN_AREA_NAME =
  new InjectionToken<string>('ADMIN_AREA_NAME');

export const routes: Routes = [
  {
    path: 'admin',
    providers: [
      AdminWorkspace,
      { provide: ADMIN_AREA_NAME, useValue: 'Admin workspace' },
    ],
    canActivate: [adminGuard],
    component: AdminPage,
    children: [
      // Child routes can inject AdminWorkspace and ADMIN_AREA_NAME too.
    ],
  },
];
~~~

A functional guard can use a dependency supplied by its route:

~~~ts
import { inject } from '@angular/core';
import type { CanActivateFn } from '@angular/router';
import { AdminWorkspace } from './admin-workspace';

export const adminGuard: CanActivateFn = () =>
  inject(AdminWorkspace).canOpen();
~~~

Routes outside the admin route subtree cannot resolve AdminWorkspace from this provider. If the same service is also provided in the root environment, an outside route can resolve the root instance, while the admin subtree resolves its closer route instance.

A route provider is **route-feature scoped**, not a promise of a fresh service for every URL visit or every parameter value. Router route reuse and the route configuration determine how activated routes and their injectors are reused. Do not use route providers as a substitute for explicitly managing a per-navigation resource lifetime.

Route providers are especially useful for feature configuration, feature-specific caches or state, and services shared by a route's components and guards. Put a provider on a parent route when multiple child routes need the same feature dependencies. Put it on a narrower child route when only that section needs them.

## How provider overrides work

The closest matching provider wins. A component can override a route provider for its own subtree; a route provider can override an application provider for that route subtree.

~~~ts
// Application configuration
providers: [
  { provide: THEME_NAME, useValue: 'default' },
]

// A route overrides the application value for the feature
{
  path: 'admin',
  providers: [
    { provide: THEME_NAME, useValue: 'admin' },
  ],
}
~~~

Code below the admin route receives admin; other routes receive default. If no closer provider exists, Angular continues upward.

Resolution modifiers such as **Self**, **SkipSelf**, **Host**, and **Optional** can constrain or soften parts of a lookup. Use them when the dependency contract specifically requires that behavior; they do not change where a provider is registered.

## Choose a provider scope

| Put the provider in | Use it when | Typical examples |
| --- | --- | --- |
| Root ApplicationConfig.providers | Many unrelated areas need one app-wide dependency | Router, HTTP configuration, global logger |
| A parent route's providers | One feature and its child routes share dependencies | Admin workspace, checkout feature config |
| A child route's providers | Only one route section needs the dependency | Report builder state |
| A component's providers | Each component instance owns separate state | Editor draft, dialog state, form-specific state |
| A component's viewProviders | A dependency is only for the component's own view | Internal presentation state hidden from projected content |

Prefer explicit scoping when a service owns state. Keep truly shared cross-feature services at application scope. Avoid registering the same stateful service at root and locally unless the override is intentional and understood.

## Common mistakes

- **Providing on a child and injecting from its parent:** Angular searches upward, not downward. Move the provider to a shared ancestor or inject it only from the child subtree.
- **Putting all services at the root:** This can make unrelated features share state and complicates isolation. Scope feature-specific state more narrowly.
- **Expecting component siblings to share a component provider:** Siblings have separate component injectors. Move the provider to their parent if they should share one instance.
- **Confusing providers with viewProviders:** Projected content can resolve providers but not viewProviders from the receiving component.
- **Providing a service both locally and globally without realizing it:** The closest registration shadows the outer provider; those consumers may receive different instances.
- **Assuming route scope means one instance per navigation:** Route injectors are managed by the Router and route reuse. Pick the scope based on the route feature, and explicitly manage resources that need a shorter lifetime.
- **Using a component provider for app-wide coordination:** Component state is intentionally isolated. Use an appropriate shared ancestor or environment provider when unrelated trees must coordinate.

For manual local or route provisioning in Angular v22.1 and newer, define a service with **Service({ autoProvided: false })** and add it to the relevant providers array. In projects using **Injectable**, **Injectable()** without providedIn is the corresponding manually provided pattern. Existing **Injectable({ providedIn: 'root' })** services can still be overridden locally, but a separate non-auto-provided service makes the intended scope clearer.

## Debugging scope problems

When a dependency is unexpectedly missing or shared:

1. Identify the exact token being injected.
2. Find every place that provides that token, including service metadata and provider arrays.
3. Draw the relevant element and environment ancestry from the consumer to the root.
4. Check whether a nearer registration is shadowing another one.
5. Check whether projected content is crossing a viewProviders boundary.
6. Use Angular DevTools' Injector Tree to inspect the element and environment injector hierarchies.

A NullInjectorError usually means Angular found no matching provider in the consumer's lookup path. Unexpected shared state often means the provider is registered higher than intended; unexpectedly separate state often means it is registered independently in multiple component trees.

## Continue learning

- Previous: [Application and environment providers](052-application-and-environment-providers.md)
- Q&A: [Component-scoped and route-scoped providers](../qnas/053-component-scoped-and-route-scoped-providers.md)
- Example: [Component-scoped and route-scoped providers](../examples/053-component-scoped-and-route-scoped-providers.md)
- Next: [InjectionToken and typed application configuration](054-injectiontoken-and-typed-application-configuration.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="052-application-and-environment-providers.md">052 - Application and environment providers</a></td><td><a href="../qnas/053-component-scoped-and-route-scoped-providers.md">Q&amp;A</a></td><td><a href="../examples/053-component-scoped-and-route-scoped-providers.md">Example</a></td><td><a href="054-injectiontoken-and-typed-application-configuration.md">054 - InjectionToken and typed application configuration</a></td>
</tr>
  </tbody>
</table>
