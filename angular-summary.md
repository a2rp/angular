# Angular: Complete Summary

A course-wide guide to Angular's architecture, essential APIs, and the decisions involved in building a reliable web application. Use it to review and connect ideas; the [120-chapter roadmap](README.md#course-roadmap) contains the planned lessons, Q&A, and examples. For short definitions, see [Angular keywords and concepts](angular-keywords.md).

Angular APIs, CLI defaults, and the maturity of newer features can vary by release. The examples here show common modern patterns; check the official documentation and compatibility tables for the version used by a project.

## Contents

- [What Angular provides](#what-angular-provides)
- [The course map](#the-course-map)
- [Start a project](#start-a-project)
- [Application structure and bootstrap](#application-structure-and-bootstrap)
- [Components, templates, and UI](#components-templates-and-ui)
- [Component communication and reusable APIs](#component-communication-and-reusable-apis)
- [State, signals, and rendering](#state-signals-and-rendering)
- [Services and dependency injection](#services-and-dependency-injection)
- [RxJS and asynchronous work](#rxjs-and-asynchronous-work)
- [Forms and validation](#forms-and-validation)
- [Routing](#routing)
- [HTTP and backend communication](#http-and-backend-communication)
- [Architecture, accessibility, and localization](#architecture-accessibility-and-localization)
- [Testing and debugging](#testing-and-debugging)
- [Performance, SSR, and delivery](#performance-ssr-and-delivery)
- [Security](#security)
- [A practical development loop](#a-practical-development-loop)
- [Official references](#official-references)

## What Angular provides

Angular is a TypeScript-based framework for building interactive web applications. It includes a component model, template language, dependency injection, routing, HTTP support, forms, testing utilities, and build tooling through the Angular CLI.

A typical Angular application uses several layers:

- **Browser platform:** HTML, CSS, JavaScript, the DOM, browser events, storage, and network APIs.
- **Angular application:** components and templates describe the interface; services and state hold reusable behavior and data.
- **Router:** maps URL paths to views and navigation behavior in a single-page application.
- **Backend:** provides trusted data, authentication, authorization, and business rules through APIs.
- **Build and delivery:** the CLI compiles and bundles the application for development or production; hosting serves the files or renders routes on a server.

Angular is distinct from AngularJS 1.x. Their architectures and APIs differ; an AngularJS application is not simply a modern Angular application with an older version number.

The central idea is declarative UI. Components describe the view for the current inputs and state. Angular tracks relevant dependencies and updates rendered UI as those values change. Keep application state authoritative in one place and let templates reflect it instead of manually editing DOM nodes.

## The course map

The Angular roadmap has ten parts and 120 chapters:

| Chapters | Area |
| --- | --- |
| 001-012 | Angular and web foundations, TypeScript, Node/npm, CLI, project structure, bootstrap |
| 013-024 | Components, standalone imports, templates, bindings, events, built-in control flow, directives |
| 025-036 | Inputs, outputs, models, projection, queries, lifecycle, styles, pipes, custom directives |
| 037-048 | Signals, RxJS interop, change detection, zoneless apps, state ownership, DevTools |
| 049-060 | Services, dependency injection, providers, Observables, operators, cancellation, cleanup |
| 061-072 | Template-driven, reactive, typed, accessible, custom-control, and signal-based forms |
| 073-084 | Router configuration, links, route state, guards, resolvers, lazy loading, testing |
| 085-096 | HttpClient, API services, interceptors, errors, caching, resources, SSE, WebSockets |
| 097-108 | Application structure, reusable UI, accessibility, testing, diagnostics |
| 109-120 | Performance, deferred views, CSR/SSR/prerendering, hydration, security, deployment, upgrades, capstone |

Use the README roadmap for the individual chapter titles and reserved lesson/Q&A/example paths. A listed roadmap topic is a learning target; consult the repository files as they are filled in chapter by chapter.

## Start a project

Install the Angular CLI, create a practice app, and run its development server:

~~~sh
npm install -g @angular/cli
ng new task-tracker --routing --style=css
cd task-tracker
ng serve --open
~~~

Common project commands include:

~~~sh
ng test
ng build
ng update
~~~

The CLI creates a workspace with source files and configuration. Keep generated practice applications separate from this learning-notes repository. Before installing or upgrading, check the Angular version compatibility table for supported Node.js, TypeScript, and RxJS combinations.

A workspace can contain multiple applications and libraries. Important files commonly include:

- **angular.json:** workspace projects, builders, assets, styles, and CLI targets.
- **package.json:** dependencies and project scripts.
- **package-lock.json:** exact resolved npm dependency tree.
- **tsconfig.json:** TypeScript compiler options and project references.
- **src/**: application source, entry points, global styles, assets, and templates.

The CLI performs development serving, compilation, testing, and production builds. A builder implements a target. Schematics generate or modify project files. Version migrations can automate some changes during supported upgrades.

## Application structure and bootstrap

A modern standalone application usually starts from a root component and application-level configuration. The bootstrap connects that root to the browser and registers dependencies and framework features.

~~~ts
import { ApplicationConfig } from '@angular/core';
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter } from '@angular/router';
import { AppComponent } from './app/app.component';
import { routes } from './app/app.routes';

const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)],
};

bootstrapApplication(AppComponent, appConfig)
  .catch((error) => console.error(error));
~~~

A provider function such as provideRouter configures a framework feature for the application. ApplicationConfig is a convenient place to compose application-level providers. Feature boundaries can also have their own providers.

Standalone components import what their templates use. NgModules remain supported for existing code and libraries, but a new standalone app does not need an NgModule just to group components.

Separate configuration from secrets. Values compiled into browser JavaScript are visible to users; private credentials belong on a trusted server.

## Components, templates, and UI

A component owns a template and describes a reusable part of the interface. The selector places that component in another template. Its metadata can include a template, styles, imports, providers, and other options.

~~~ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  templateUrl: './counter.html',
})
export class CounterComponent {
  count = signal(0);

  increment(): void {
    this.count.update((value) => value + 1);
  }
}
~~~

~~~html
<p>Count: {{ count() }}</p>
<button type="button" (click)="increment()">Add one</button>
~~~

Angular templates are HTML with additional syntax:

- **Interpolation** displays text with double curly braces.
- **Property binding**, such as [disabled]="saving", sets a DOM property or component input.
- **Attribute binding**, such as [attr.aria-label]="label", targets an HTML attribute.
- **Class and style bindings** update an element's presentation from state.
- **Event binding**, such as (click)="save()", responds to browser or component events.
- **Two-way binding** combines reading a value and reporting its changes. [(ngModel)] belongs to template-driven forms.
- **Template references** name elements or template fragments for use within the template.
- **Pipes** format or transform values for display; AsyncPipe can consume an Observable or Promise with managed cleanup.

Prefer built-in template control flow for conditional and repeated content:

~~~html
@if (tasks().length === 0) {
  <p>No tasks yet.</p>
} @else {
  <ul>
    @for (task of tasks(); track task.id) {
      <li>{{ task.title }}</li>
    } @empty {
      <li>No matching tasks.</li>
    }
  </ul>
}
~~~

The track expression gives each item stable identity so Angular can reuse the corresponding view. Template expressions have restrictions; keep substantial business logic in TypeScript rather than making a template difficult to read.

Directives add behavior to elements or templates. Components are directives that own templates. Attribute directives enhance existing elements; structural directives create or remove views. Newer built-in control flow replaces many common uses of legacy NgIf and NgForOf. Custom pipes and directives should have a narrow, predictable responsibility.

Use semantic HTML and native browser controls first. A button should perform an action; a link should navigate. This preserves expected keyboard and assistive-technology behavior.

## Component communication and reusable APIs

A component API describes how other components provide data, receive events, and supply content.

- **Inputs** pass data from parent to child. Modern code can use signal-based input(); decorator-based @Input remains in existing code.
- **Required inputs** make callers provide values; transforms can normalize values at the boundary.
- **Outputs** report events from child to parent. They should communicate something that happened or a requested action.
- **Model inputs** pair a value and change event for reusable components that need two-way binding.
- **Content projection** places caller-provided child markup into ng-content slots.
- **View and content queries** locate child elements, components, directives, or projected content when a real component relationship requires it.
- **Host bindings and listeners** connect a directive or component to its host element; use clear template bindings when practical.
- **Host directives** compose reusable directive behavior into another component.

Prefer one-way data flow and a clear source of truth. Lift state to the nearest shared owner when siblings must coordinate. Do not copy input state into local state unless the component has a specific ownership rule for that copy. A ControlValueAccessor is the adapter for a custom control that participates in Angular forms.

Component CSS is scoped by the configured encapsulation strategy. The default emulated mode scopes component styles. Shadow DOM uses browser encapsulation; disabling encapsulation makes styles global. Put site-wide resets and design tokens in global styles, and component-specific rules with the component.

## State, signals, and rendering

State is data that can change and affect the UI. Place state with its owner and derive values instead of storing duplicate facts. For example, calculate a filtered list from the source list and filter rather than maintaining two independently updated lists.

Angular signals provide a synchronous reactive value model:

- **signal()** creates writable state. Read it by calling it; update it with set() or update().
- **computed()** derives a read-only value from signals it reads.
- **linkedSignal()** supports writable state whose value is related to changing source state.
- **effect()** runs when signals read by the effect change. Use it mainly to synchronize with an imperative API; use computed for derived values.
- **untracked()** reads a signal without adding that read as a dependency of the current reactive computation.
- **input()**, **output()**, and **model()** provide signal-oriented component APIs.
- **toSignal()** and **toObservable()** bridge signal values and RxJS streams.
- **resource()** and **httpResource()** represent asynchronous data in signal-oriented code where supported.

Signal reads in a template establish dependencies. When a dependency changes, Angular can mark the relevant view for checking. Signals use equality to decide whether a value changed; immutable object and array updates make state changes easier to detect and reason about.

Angular change detection checks views and updates the DOM. OnPush lets Angular skip subtrees when no relevant notification or input change requires checking. Zoneless change detection uses framework notifications rather than Zone.js to schedule updates. Choose one clear state model per use case; signals are useful for synchronous state, while RxJS is useful for streams and asynchronous composition.

Lifecycle hooks describe component and directive lifetime, including input changes, initialization, view/content readiness, and destruction. Avoid expensive recurring work in check hooks. Use DestroyRef or takeUntilDestroyed for context-bound cleanup. afterNextRender and related render callbacks are useful for carefully scheduled DOM integration; browser-only code must account for server rendering.

## Services and dependency injection

A service centralizes behavior that should be shared or separated from presentation, such as API access, business operations, or application state. Dependency injection makes these dependencies explicit and replaceable.

~~~ts
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';

interface Task { id: string; title: string; }

@Injectable({ providedIn: 'root' })
export class TaskApi {
  private readonly http = inject(HttpClient);

  list() {
    return this.http.get<Task[]>('/api/tasks');
  }
}
~~~

Angular can inject a dependency through a constructor or through inject() inside a valid injection context. An InjectionToken represents a runtime dependency that has no class token, such as configuration.

Providers describe how to resolve a token. Common strategies include useClass, useValue, useFactory, and useExisting. Provider location determines scope and instance lifetime:

- Root or environment providers serve the application or an environment subtree.
- Route providers are scoped to a route subtree.
- Component providers are scoped to a component and its descendants.

DI is hierarchical. Angular searches from a relevant child injector toward parent injectors, subject to modifiers such as optional, self, skipSelf, and host. Keep provider scope intentional; registering the same service in a component can create a separate instance instead of using the root instance.

## RxJS and asynchronous work

RxJS represents asynchronous or event-based sequences with Observables. An Observable can emit many values, fail, or complete. A Promise represents one eventual result. A signal represents a synchronous current value with tracked consumers. These models complement each other but are not interchangeable.

An Observable pipeline composes operators such as map, filter, catchError, and finalize. Flattening operators define what happens when one event starts asynchronous work:

| Operator | Typical behavior |
| --- | --- |
| switchMap | Cancel/unsubscribe the previous inner stream when a newer source value arrives; useful for latest-search requests. |
| concatMap | Queue inner work and run it in order. |
| mergeMap | Run inner work concurrently. |
| exhaustMap | Ignore new source values while current inner work is active. |

Choose based on user intent and server behavior. Unsubscribing can prevent stale results from reaching the UI, but cancellation of underlying work depends on the Observable source. Handle errors at a boundary where the application can show useful recovery. Release subscriptions with AsyncPipe, takeUntilDestroyed, or another explicit ownership strategy.

## Forms and validation

Angular supports multiple form approaches:

- **Template-driven forms** configure controls in HTML using forms directives such as ngForm and ngModel. They are convenient for smaller forms with simpler interaction.
- **Reactive forms** model FormControl, FormGroup, FormArray, or FormRecord instances in TypeScript. They make form structure and validation logic explicit.
- **Typed reactive forms** add TypeScript types to controls and form values to catch mismatches during development.
- **Signal Forms** provide signal-oriented models, field state, schemas, validation, and custom-control APIs. Check the official docs for the Angular version and feature maturity before adopting them.

A form control tracks value, status, validators, and interaction state. Common state includes valid/invalid, pending, disabled, touched/untouched, and dirty/pristine. Built-in validators cover rules such as required, length, numeric bounds, and pattern. Cross-field validation belongs at the group that owns the compared controls. Async validators can report pending state while they run.

~~~ts
import { FormControl, FormGroup, Validators } from '@angular/forms';

const form = new FormGroup({
  title: new FormControl('', {
    nonNullable: true,
    validators: [Validators.required, Validators.maxLength(80)],
  }),
});

function submit(): void {
  if (form.invalid) {
    form.markAllAsTouched();
    return;
  }

  const value = form.getRawValue();
  // Send validated data to an application service.
}
~~~

setValue expects the complete structure; patchValue updates supplied fields. reset changes values and interaction state. getRawValue includes disabled controls. Validate on both client and server: browser validation improves feedback but is not a security boundary.

Accessible forms have visible labels, clear instructions, keyboard-operable controls, and error text associated with fields. Explain what needs correction, not only that a value is invalid. Custom controls integrate through ControlValueAccessor and must preserve expected focus and keyboard behavior.

## Routing

The Angular Router maps URL paths to components and supports navigation without a full document reload. A route table describes paths and can attach components, loaders, providers, guards, resolvers, titles, and child routes.

~~~ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  { path: '', pathMatch: 'full', redirectTo: 'tasks' },
  {
    path: 'tasks',
    loadComponent: () =>
      import('./tasks/tasks-page').then((module) => module.TasksPage),
  },
  { path: '**', loadComponent: () => import('./not-found/not-found-page')
      .then((module) => module.NotFoundPage) },
];
~~~

Register routes with provideRouter(routes). RouterOutlet marks where the active routed component appears. RouterLink creates navigable links; RouterLinkActive can communicate the active route state.

ActivatedRoute exposes the current route, parameters, query parameters, data, and snapshots. A snapshot is a point-in-time value; use its observable or signal-based reactive alternative when the same component can remain active while route parameters change. Query parameters and fragments are useful for shareable filters, search, pagination, and in-page locations.

Use programmatic navigation when an action should change the route. Route guards can allow, redirect, or cancel navigation; they improve client flow but do not secure backend data. A resolver loads data before activation when that behavior is useful. Lazy loading can defer route code until needed. Nested routes render through child outlets; auxiliary routes use named outlets. Configure redirects, not-found behavior, route titles, preloading, scroll restoration, and navigation tests deliberately.

## HTTP and backend communication

HttpClient makes requests and returns Observables. Configure it in a standalone app with provideHttpClient(). Put endpoint operations and response mapping in an injectable API service rather than scattering request details throughout templates.

~~~ts
import { inject, Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

interface Task {
  id: string;
  title: string;
}

@Injectable({ providedIn: 'root' })
export class TaskApi {
  private readonly http = inject(HttpClient);

  list() {
    return this.http.get<Task[]>('/api/tasks');
  }

  create(title: string) {
    return this.http.post<Task>('/api/tasks', { title });
  }
}
~~~

The TypeScript type describes the expected response; it does not check that runtime JSON truly has that shape. Parse or validate external data at the boundary when correctness or security requires it.

Requests can configure headers, query parameters, credentials, response type, progress events, and response observation mode. Functional interceptors can add cross-cutting behavior such as request identifiers or authentication metadata. Keep interceptor ordering and retry behavior explicit. Do not blindly retry non-idempotent operations.

Model loading, success, empty, and error states in the UI. Cancel or supersede stale requests when user intent changes. Use HttpTestingController to test requests and responses. CORS is configured by the server, and XSRF defenses require correct server and cookie behavior. Authentication and authorization must be backed by trusted server-side checks.

Signals and resources offer alternate ways to represent asynchronous data in current Angular APIs. Choose based on the project's Angular version and the needs of the feature; no single abstraction replaces all RxJS use cases.

## Architecture, accessibility, and localization

Organize code around features and clear responsibilities. Keep reusable UI components focused, and avoid a giant shared folder that hides ownership. Libraries can create boundaries when multiple applications or teams need stable shared code. Dynamic component creation is appropriate when the component type is genuinely selected at runtime; static templates are simpler for known UI.

Accessibility is a design requirement, not a final automated test. Use semantic HTML, native controls, visible focus, logical keyboard order, accessible names, and correctly associated form errors. Add ARIA only when native HTML does not express the required role or state. Test real keyboard interactions and assistive-technology-relevant behavior.

Internationalization prepares content and UI for multiple languages and regions. Localization adapts translations, dates, numbers, currencies, plural rules, and conventions for a locale. Keep user-visible text translatable and avoid assembling sentences from fragments that cannot be translated naturally.

## Testing and debugging

Choose test scope according to the behavior being protected:

- **Unit tests** cover small functions, validators, pipes, or service behavior.
- **Component tests** verify rendered output, input changes, events, and user interactions.
- **Integration tests** exercise collaborating parts such as a form with a service or a routed page.
- **End-to-end tests** verify browser-level user journeys using a separately selected automation tool.

TestBed configures Angular dependencies and components for tests. ComponentFixture provides the instance and rendered host. Run change detection intentionally and query what a user can see or interact with. Use HTTP testing providers and HttpTestingController for requests. Configure router tests with the router testing utilities supported by the project.

A test double can replace a dependency with a fake, stub, or mock. Keep assertions about observable behavior rather than implementation details. Coverage shows which code executed; it does not prove that assertions are meaningful. Include regression tests for fixed defects and test failure, loading, empty, and success cases.

The Angular CLI test runner can change between releases or project configurations. Check generated project scripts and current CLI guidance instead of assuming every Angular workspace uses the same runner. Browser developer tools and Angular DevTools help inspect console errors, network requests, component structure, change detection, and performance.

## Performance, SSR, and delivery

Performance work begins with measurement. Check bundle transfer size, parsing and execution cost, route loading, rendering work, and Core Web Vitals. Use lazy routes and @defer where delaying code improves the experience, with a useful placeholder and error state. Stable tracking, simple templates, OnPush, and sensible state updates reduce unnecessary work. Profile before optimizing.

Angular can render on the client, on a server, or ahead of time. Hybrid rendering selects modes by route:

- **CSR:** browser JavaScript renders the view.
- **SSR:** server returns rendered HTML for a request.
- **Prerendering/SSG:** HTML for known routes is generated before a request.
- **Hybrid rendering:** an application combines modes based on route and content needs.

Hydration connects browser Angular to server-rendered HTML. Event replay can preserve eligible early interactions; incremental hydration can activate selected portions later. Server rendering can improve initial content delivery and search visibility, but it adds runtime, deployment, caching, and server-compatible-code requirements. Do not assume window, document, localStorage, or other browser globals exist on the server.

Keep server and browser state consistent to avoid hydration mismatches. Use transfer mechanisms for eligible data instead of issuing duplicate startup requests. Configure route-level rendering based on actual content and operational requirements.

A service worker can cache application resources and support offline use or update flows. Design for intermittent networks and communicate when a new app version is available. Deploy the correct base URL and configure hosting to serve client-side routes. SSR apps also need an appropriate server runtime and proxy configuration. Use the Angular update guide and migrations when upgrading, then review breaking changes, peer dependencies, tests, and production build behavior.

## Security

Angular sanitizes many template-bound values in context-specific ways, but a framework does not secure the whole product automatically.

- Treat URL, form, and API data as untrusted until validated.
- Prefer normal text interpolation and Angular bindings over direct DOM manipulation.
- Avoid trust-bypass APIs unless a reviewed, context-specific value is demonstrably safe.
- Use a Content Security Policy and consider Trusted Types as part of a wider defense.
- Keep secrets on a trusted server; browser bundles and public repositories are visible.
- Protect server endpoints with authentication and authorization. Hiding a route or button is not authorization.
- Use correct cookie settings and server-side XSRF protections.
- Keep dependencies updated and review security advisories.
- Avoid logging tokens, passwords, personal data, or sensitive request bodies.

## A practical development loop

1. Clarify the user-facing behavior and the data the feature owns.
2. Create or update a feature-focused component and its template.
3. Keep domain and API behavior in typed services; choose signals, RxJS, or forms according to the data flow.
4. Build semantic, keyboard-accessible UI with explicit loading, empty, success, and error states.
5. Add tests for component behavior, service boundaries, forms, routes, and failure cases.
6. Run the project's formatter, linting, tests, and production build when configured.
7. Inspect browser console and network behavior; measure before attempting performance changes.
8. Verify the exact Angular, Node.js, TypeScript, and RxJS versions and deploy using the chosen rendering mode.
