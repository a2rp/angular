# Angular Compared with AngularJS: Q&A

This Q&A accompanies [Chapter 2: Angular Compared with AngularJS](../lessons/002-angular-compared-with-angularjs.md).

## Framework identity

### 1. What does AngularJS mean?

AngularJS usually refers to the original 1.x framework. Its common concepts include modules, controllers, scopes, directives, services, and the digest cycle.

### 2. What does Angular mean in this course?

Angular means the modern framework that followed AngularJS. It is commonly written in TypeScript and builds user interfaces from components, templates, services, and other framework APIs.

### 3. Is Angular simply a new major version of AngularJS?

No. The names share history, but Angular is a distinct framework generation with different application architecture, APIs, tooling, and template syntax. AngularJS application code does not become modern Angular merely by updating a package version.

### 4. Are AngularJS and Angular completely unrelated?

No. They share history and broad ideas such as templates, directives, dependency injection, and reusable services. The details and APIs differ, so those similarities are useful for understanding concepts but do not guarantee that code can be reused unchanged.

### 5. Is AngularJS still officially supported?

The AngularJS project ended official long-term support in January 2022. Existing applications can continue running, but their owners must account for framework maintenance, security, browser compatibility, and dependency risks. See the official AngularJS migration guide.

## Architecture and terminology

### 6. What is the primary UI building block in AngularJS?

AngularJS applications often use controllers with templates, directives, or AngularJS components. The exact style depends on the age and conventions of the application.

### 7. What is the primary UI building block in modern Angular?

A component is the usual UI building block. It has a class, metadata such as a selector, and a template. Components are composed into a tree that represents the application's interface.

### 8. What role did a scope play in AngularJS?

A scope exposed data and behavior to AngularJS template expressions. Scopes could form parent-child relationships, and AngularJS used watchers and digest processing to notice changes and refresh bindings.

### 9. Does modern Angular use AngularJS scopes?

No. Modern Angular components own their class state and communicate with other parts of an application through component APIs, services, signals, RxJS, and other supported patterns. There is no direct modern Angular equivalent of the AngularJS scope object.

### 10. Is a controller just a component with a different name?

No. A component has a defined template boundary and framework metadata, and it participates in component lifecycle and composition. A controller may provide behavior to a view, but renaming it does not supply the component contract or convert its template and lifecycle.

### 11. What does “module” mean in AngularJS?

An AngularJS module is commonly an application registration unit. Code can register controllers, services, directives, and dependencies with it.

### 12. What is an NgModule?

An NgModule is a modern Angular organization and configuration mechanism. It is not the same API as an AngularJS module. Modern Angular also supports standalone components and standalone APIs, so a current application may not use NgModules for every feature.

### 13. Are NgModules obsolete?

No. Standalone APIs are the default direction for new modern Angular code, but NgModules remain supported and are present in many applications and libraries. When working in a repository, follow its Angular version and established structure.

### 14. Are AngularJS directives and Angular directives identical?

No. Both frameworks use the word directive for ways to extend or control template behavior, but the APIs and lifecycle differ. Learn the directive system for the framework version actually used by the project.

## Templates and state

### 15. What does AngularJS interpolation look like?

AngularJS commonly displays values with double curly braces, for example {{ vm.name }}. The expression is evaluated against the scope or controller context connected to the template.

### 16. Does modern Angular use interpolation too?

Yes. Modern Angular templates also use double curly braces for displaying values. The expression is evaluated against the component context. Although the punctuation looks familiar, surrounding APIs and state ownership may differ.

### 17. How do template events differ?

AngularJS commonly uses directives such as ng-click. Modern Angular commonly uses event binding syntax such as (click)="save()". This is a useful syntax comparison, not a universal one-to-one migration rule.

### 18. Is AngularJS ng-model the same as modern Angular forms?

No. Both can connect UI input and application data, but modern Angular offers different forms APIs, including template-driven forms and reactive forms. Choose the suitable modern form approach and preserve validation, accessibility, and submission behavior explicitly.

### 19. What is the AngularJS digest cycle?

It is AngularJS's mechanism for checking watched expressions and updating template bindings when values change. AngularJS runs digest processing in response to framework events and may require explicit integration when state changes outside its normal event flow.

### 20. How does modern Angular update a view?

Modern Angular has its own change-detection model. Signals provide reactive state whose reads can be tracked, and applications can also use RxJS or other state approaches. Do not assume that every AngularJS watcher has a direct modern replacement.

### 21. Are Angular signals just watchers with different syntax?

No. A signal is a modern Angular reactive primitive with its own read and update API. A watcher is part of AngularJS's scope and digest model. During migration, first identify what the value represents, who owns it, and which parts of the UI depend on it.

## Dependencies, tooling, and support

### 22. Do both frameworks have dependency injection?

Yes. Both provide dependency-injection systems. The registration rules, providers, tokens, and application structure are different, so an AngularJS service does not automatically work as a modern Angular injectable service.

### 23. What does the Angular CLI contribute?

The Angular CLI provides commands and build tooling for common application tasks, such as creating projects, generating code, serving local builds, running configured checks, and producing production builds. AngularJS applications may use older script-based or custom build setups.

### 24. Does modern Angular require TypeScript?

No. Angular can be used with JavaScript, but TypeScript is the common choice and is used in most modern Angular examples and projects. Existing AngularJS applications are commonly written in JavaScript, though some used TypeScript.

### 25. Can AngularJS and modern Angular code run together during a migration?

Some migration plans use a temporary hybrid arrangement and official interoperability tooling. This is an architectural transition with constraints, testing needs, and lifecycle considerations; it is not a permanent shortcut that makes all APIs interchangeable. Consult the migration documentation for the Angular and AngularJS versions involved.

## Upgrades and migration

### 26. What is an AngularJS version upgrade?

It changes an AngularJS application from one AngularJS release to another while the application remains on AngularJS. It can still require compatibility checks and dependency work.

### 27. What is an Angular migration?

It moves an application from AngularJS to modern Angular. It typically involves changes to UI boundaries, templates, state, dependency injection, routing, HTTP, tests, libraries, and build and deployment tooling.

### 28. Is migrating to Angular just a package update?

No. It is an application-level change. AngularJS and modern Angular have different APIs and runtime assumptions. A package change alone will not transform controllers, scopes, directives, templates, tests, or integrations.

### 29. Should every AngularJS application be rewritten from scratch?

Not necessarily. Options include containing and maintaining a legacy system for a limited period, migrating gradually across feature boundaries, or rebuilding selected areas. The choice depends on business requirements, system complexity, test coverage, dependency health, risk, and available capacity.

### 30. What should be checked before choosing a migration strategy?

Confirm the framework and dependency versions, application entry points, routing, build and deployment steps, major feature boundaries, external integrations, test coverage, and business-critical user journeys. Also identify unsupported libraries and behaviors that currently lack clear specifications.

### 31. Why should behavior be documented before migration?

A replacement can be syntactically modern but still change important behavior. Documented examples and automated tests make it possible to compare the old and new implementations and catch regressions.

### 32. What is a sensible first migration slice?

Choose a bounded feature with clear inputs, outputs, and user-visible behavior. It should be representative enough to reveal integration issues, but small enough to test, review, and roll back safely.

### 33. Should AngularJS syntax be replaced everywhere at once?

Usually not. A broad mechanical rewrite can hide behavior changes and produce hard-to-review changes. Establish boundaries, migrate in verifiable slices, and validate each slice against expected behavior.

## Check your understanding

### 34. A project registers code with angular.module and uses ng-controller. Which framework family is it likely using?

It is likely an AngularJS 1.x application. Confirm the actual dependency version and bootstrap code rather than relying on one identifier alone.

### 35. A tutorial describes a standalone component with a TypeScript class and Angular template. Is it teaching AngularJS?

No. It is teaching modern Angular.

### 36. Why can a developer familiar with AngularJS still benefit from learning Angular from the beginning?

Shared ideas can help with orientation, but framework APIs, component architecture, template syntax, state patterns, tooling, and runtime behavior differ. Learning modern Angular directly avoids carrying over assumptions that no longer apply.

### 37. Where should you look for authoritative guidance on a specific project's migration?

Start with the official AngularJS migration guide and the official Angular documentation. Then check the specific Angular and AngularJS versions, dependencies, and project constraints before deciding how to apply that guidance.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="001-angular-and-component-based-web-applications.md">001 - Angular and Component-Based Web Applications</a></td><td><a href="../lessons/002-angular-compared-with-angularjs.md">Lesson</a></td><td><a href="../examples/002-angular-compared-with-angularjs.md">Example</a></td><td><a href="003-browser-dom-http-and-javascript-foundations.md">003 - Browser, DOM, HTTP, and JavaScript Foundations</a></td>
</tr>
  </tbody>
</table>
