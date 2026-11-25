# Angular Compared with AngularJS

Angular and AngularJS are related by name and history, but they are different generations of the framework. This chapter explains how to tell them apart, what ideas they share, and what changes when a legacy AngularJS application is moved to modern Angular.

## Learning objectives

After this lesson, you should be able to:

- Distinguish AngularJS 1.x from modern Angular.
- Recognize the main building blocks and syntax used by each framework.
- Explain why AngularJS concepts cannot be copied directly into an Angular application.
- Describe the difference between upgrading an AngularJS application and migrating it to Angular.
- Choose sensible first steps when assessing a legacy application.

## Names and versions

AngularJS usually means the original framework in the 1.x series. It is written primarily in JavaScript and uses concepts such as AngularJS modules, controllers, scopes, directives, and the digest cycle.

Angular usually means the modern framework that followed AngularJS. It is commonly written in TypeScript and organizes user interfaces around components. Its ecosystem includes the Angular CLI, templates, dependency injection, routing, forms, HTTP support, and build tools.

The names are easy to confuse. In this course, “AngularJS” means the 1.x framework, while “Angular” means the modern framework. The official AngularJS project ended long-term support in January 2022. That matters when maintaining an old application: its framework no longer receives official maintenance and security updates from the AngularJS team. See the [AngularJS migration guide](https://docs.angularjs.org/guide/migration) and the [Angular overview](https://angular.dev/overview).

## The short comparison

| Area | AngularJS 1.x | Modern Angular |
| --- | --- | --- |
| Typical language | JavaScript | TypeScript is commonly used |
| Main UI unit | Controller and template, or a directive/component | Component class and template |
| UI state | Often exposed through a scope or controller instance | Usually held by a component class, signals, or another state service |
| Template binding | Expressions such as interpolation, ng-model, and ng-click | Interpolation, property/event bindings, and directives such as @if and @for |
| Dependency injection | Available through AngularJS modules and services | Available through Angular's injector and providers |
| Module terminology | An AngularJS module registers application parts and dependencies | Modern Angular supports standalone APIs; NgModules remain available in existing and some library code |
| UI update model | Scope changes are observed by the digest cycle | Angular schedules and performs change detection; signals can make state dependencies explicit |
| Typical tooling | Script tags and older build setups are common in legacy apps | Angular CLI and a modern build pipeline are common |
| Current status | AngularJS official support ended in January 2022 | Angular is the actively developed framework |

This is a high-level comparison. Real projects vary: AngularJS apps can use components and TypeScript, and modern Angular applications can still use NgModules or RxJS. A project's actual code and Angular version matter more than assumptions based on its age.

## AngularJS building blocks

A typical AngularJS application may contain:

- A module created with the global AngularJS API. It groups application components and declares dependencies.
- A controller that prepares behavior and data for a view.
- A scope that connects controller data and template expressions.
- Directives such as ng-app, ng-controller, ng-model, and ng-click.
- Services for shared logic or browser and network tasks.
- The digest cycle, which checks watched values and updates bindings when AngularJS knows that application state may have changed.

For example, an AngularJS template might use the directive ng-controller with the controller alias CounterController as vm and display the expression {{ vm.count }}. The controller and template work together through AngularJS's binding system.

AngularJS supports more than one style. Older tutorials often show controllers injecting $scope; later codebases may use the controller-as pattern or AngularJS components. When reading a legacy application, follow the convention already used by that project.

## Modern Angular building blocks

A modern Angular interface is normally a tree of components. A component combines a TypeScript class with metadata and a template. The class holds behavior and state; the template describes the view and binds to that state.

A small component can look like this:

~~~ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: '<p>Count: {{ count() }}</p><button type="button" (click)="increment()">Add one</button>',
})
export class CounterComponent {
  readonly count = signal(0);

  increment(): void {
    this.count.update((currentCount) => currentCount + 1);
  }
}
~~~

The component's selector lets another template use it as an element. The signal stores a value that can change; reading it in the template makes that value available for rendering. Modern Angular components are standalone by default in Angular 19 and later. Earlier Angular versions commonly required an explicit standalone setting or registration in an NgModule, so check the version used by a project. Learn more in the official [component guide](https://angular.dev/guide/components) and [signals guide](https://angular.dev/guide/signals).

## Similar ideas, different APIs

Some concepts have familiar counterparts, but the APIs and lifecycles are different.

| Familiar idea | AngularJS form | Modern Angular form |
| --- | --- | --- |
| Reusable UI | Directive or AngularJS component | Component |
| Shared application behavior | Service registered on an AngularJS module | Injectable service provided through Angular's dependency injection |
| Template event | ng-click="vm.save()" | (click)="save()" |
| Template input | ng-model="vm.name" | A property binding, event binding, or a form control |
| Display a value | {{ vm.name }} | {{ name }} or a signal read such as {{ name() }} |
| Conditional or repeated UI | ng-if, ng-repeat | Built-in control flow such as @if and @for, or structural directives |
| Navigation | Often a separate AngularJS routing library | Angular Router |
| HTTP | Often $http | Angular's HttpClient |

These rows are conceptual comparisons, not automatic replacements. For example, modern Angular has dependency injection too, but an AngularJS service cannot simply be passed into a modern Angular constructor without a migration design. Likewise, ng-model and modern Angular forms are not interchangeable spellings of the same API.

## “Module” means different things

The word module is especially easy to misread.

In AngularJS, an application commonly starts by registering an AngularJS module with a name and a list of dependencies. Controllers, services, and directives are registered with that module.

Modern Angular also has NgModules, but current Angular supports standalone components and other standalone APIs. A modern Angular app does not necessarily have an NgModule for every feature. File-system modules such as JavaScript's ES modules are another separate idea: they control imports and exports in source code.

When examining a codebase, identify which kind of module a file or tutorial means before applying advice from it.

## Understanding the update model

AngularJS commonly uses scopes, watchers, and a digest cycle. AngularJS checks watched expressions during a digest and updates the view when bound values change. Code that changes state outside AngularJS's normal event handling may need to re-enter its update mechanism.

Modern Angular also updates views when application state changes, but its component and change-detection model is different. Signals provide explicit reactive state: a template that reads a signal can be refreshed when that signal changes. RxJS streams and other state approaches can also be used. Do not translate every AngularJS watcher into a signal mechanically; first identify what state is shared, who owns it, and when it changes.

## Upgrade versus migration

These terms describe different amounts of change:

- An AngularJS version upgrade moves an application between AngularJS releases, such as updating an older 1.x application to a later compatible 1.x release. It remains an AngularJS application.
- An Angular migration moves an application from AngularJS to modern Angular. This changes the framework and usually requires changes to templates, components, dependency injection, build tooling, tests, and libraries.
- An Angular version upgrade moves an application from one modern Angular version to another. Follow the instructions for the project's current and target versions.

A migration is not just a package-version edit. AngularJS and modern Angular have different APIs, runtime assumptions, and ecosystems. Some organizations migrate gradually with a hybrid period; others replace the application feature by feature or rebuild it. The right approach depends on application size, test coverage, business risk, team capacity, and the health of its dependencies. Read the [official AngularJS migration guide](https://docs.angularjs.org/guide/migration) before planning a legacy migration.

## A practical legacy-app assessment

Before changing an AngularJS application:

1. **Confirm what is running.** Find the AngularJS version, bootstrap entry point, routing library, build commands, and deployment process.
2. **Map the boundaries.** Identify major screens, shared services, data flows, and integrations with non-AngularJS code.
3. **Check dependencies.** Record libraries that are unmaintained, unsupported, difficult to replace, or coupled to AngularJS internals.
4. **Establish confidence.** Run the project as it exists, document reproducible setup steps, and identify existing tests. Add or improve tests as part of a migration plan.
5. **Stabilize behavior.** Fix known defects and clarify the expected behavior of important user journeys before replacing their implementation.
6. **Choose a migration shape.** Compare continued containment, gradual migration, and a feature-level rewrite. Estimate the risks and operational cost of each.
7. **Migrate in verifiable slices.** Define boundaries, move a small feature, check its behavior, and repeat. Keep rollback and deployment plans explicit.

Do not begin by renaming files or changing syntax everywhere. A migration succeeds when the replacement preserves required behavior and can be maintained, not when old keywords have disappeared.

## Common misunderstandings

- **“AngularJS is just an old version of Angular.”** They are distinct frameworks with different programming models and APIs.
- **“They are entirely unrelated.”** They share history and some broad ideas, including templates, directives, and dependency injection, but details differ.
- **“AngularJS modules and Angular NgModules are the same.”** They are different registration and organization systems.
- **“A controller becomes a component by renaming it.”** A component also has a template contract, inputs, outputs, lifecycle, and framework metadata.
- **“Signals are just AngularJS watchers with new syntax.”** They have a different API and reactive model; first understand the dependency and ownership of state.
- **“Migration means upgrading one dependency.”** It is a planned application change across code, tests, dependencies, and delivery.
- **“AngularJS is safe because it still runs.”** A running application can still carry maintenance, security, browser-compatibility, and dependency risks.

## Knowledge check

1. Which framework uses ng-controller and $scope?
2. What is the usual primary UI building block in modern Angular?
3. Why is an AngularJS 1.x upgrade different from a migration to Angular?
4. What does the term module mean in AngularJS compared with modern Angular?
5. Name three facts to establish before choosing a migration approach.

Answers are in [Chapter 2 Q&A](../qnas/002-angular-compared-with-angularjs.md). To see both styles side by side, work through [Chapter 2 examples](../examples/002-angular-compared-with-angularjs.md).

## Further reading

- [Angular overview](https://angular.dev/overview)
- [Angular components](https://angular.dev/guide/components)
- [Angular signals](https://angular.dev/guide/signals)
- [AngularJS migration guide](https://docs.angularjs.org/guide/migration)
- [AngularJS scope guide](https://docs.angularjs.org/guide/scope)
