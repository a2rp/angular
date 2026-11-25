# Angular: Practical Notes

These are my personal learning notes for Angular: a practical, beginner-friendly course and reference for building accessible, reliable web applications.

Angular is a TypeScript-based web framework for building applications from components, templates, services, and other framework APIs. This roadmap starts with web and TypeScript foundations, then moves through components, signals, forms, routing, HTTP, testing, rendering, security, and production delivery.

## Learning outcomes

By following the roadmap, you will learn to:

- Create and run a modern standalone Angular application with the Angular CLI.
- Build components and templates with accessible HTML, bindings, pipes, and directives.
- Design component APIs with inputs, outputs, model inputs, and content projection.
- Manage local and shared state with signals, RxJS, and dependency-injected services.
- Build forms with template-driven forms, typed reactive forms, and Signal Forms.
- Navigate with the Angular Router and load data from HTTP APIs.
- Test components, services, forms, routes, and user-facing behavior.
- Choose client rendering, server rendering, and prerendering for an application's needs.
- Improve accessibility, performance, security, maintainability, and deployment quality.

## Before you begin

You should be comfortable with JavaScript fundamentals, HTML, CSS, a terminal, and Git. Basic TypeScript helps, but the opening chapters introduce the types and language features used throughout the course. Some later chapters assume familiarity with HTTP, promises, and observables.

## Start an Angular practice app

Install the Angular CLI and create a practice app in its own directory. Keep the learning-notes repository separate from generated application files.

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

Angular CLI defaults and supported Node.js, TypeScript, and RxJS versions change over time. Check the official version compatibility and update guides when creating or upgrading a project.

## How to use this repository

1. Follow the roadmap in order; each chapter builds on earlier concepts.
2. Read the lesson and try the example in a separate Angular application.
3. Use the Q&A file to review concepts and check for gaps.
4. Look up version-specific behavior in the official Angular documentation.
5. Apply the accessibility, testing, and security practices throughout the course.

The course focuses on current standalone APIs and modern Angular patterns while explaining older patterns when they matter for existing applications. APIs can differ by Angular version; check the linked official references before using a version-specific feature. Signal Forms and other newer APIs are called out with their compatibility notes in the relevant chapters.

## Quick references

- [Angular keywords and concepts](angular-keywords.md)

## Course roadmap

The roadmap contains 120 chapters across ten parts. Every row reserves a lesson, Q&A, and practical example path. Files are developed chapter by chapter.

### 1. Angular foundations and setup

| No. | Lesson | Q&A | Example |
| --- | --- | --- | --- |
| 001 | [Angular and component-based web applications](lessons/001-angular-and-component-based-web-applications.md) | [Open Q&A](qnas/001-angular-and-component-based-web-applications.md) | [Open Example](examples/001-angular-and-component-based-web-applications.md) |
| 002 | [Angular compared with AngularJS](lessons/002-angular-compared-with-angularjs.md) | [Open Q&A](qnas/002-angular-compared-with-angularjs.md) | [Open Example](examples/002-angular-compared-with-angularjs.md) |
| 003 | [Browser, DOM, HTTP, and JavaScript foundations](lessons/003-browser-dom-http-and-javascript-foundations.md) | [Open Q&A](qnas/003-browser-dom-http-and-javascript-foundations.md) | [Open Example](examples/003-browser-dom-http-and-javascript-foundations.md) |
| 004 | [TypeScript essentials: values, types, and inference](lessons/004-typescript-essentials-values-types-and-inference.md) | [Open Q&A](qnas/004-typescript-essentials-values-types-and-inference.md) | [Open Example](examples/004-typescript-essentials-values-types-and-inference.md) |
| 005 | [TypeScript interfaces, unions, generics, and narrowing](lessons/005-typescript-interfaces-unions-generics-and-narrowing.md) | [Open Q&A](qnas/005-typescript-interfaces-unions-generics-and-narrowing.md) | [Open Example](examples/005-typescript-interfaces-unions-generics-and-narrowing.md) |
| 006 | [Classes, modules, decorators, and compiler options](lessons/006-classes-modules-decorators-and-compiler-options.md) | [Open Q&A](qnas/006-classes-modules-decorators-and-compiler-options.md) | [Open Example](examples/006-classes-modules-decorators-and-compiler-options.md) || 007 | [Node.js, npm, package managers, and workspaces](lessons/007-node-js-npm-package-managers-and-workspaces.md) | [Open Q&A](qnas/007-node-js-npm-package-managers-and-workspaces.md) | [Open Example](examples/007-node-js-npm-package-managers-and-workspaces.md) |
| 008 | [Angular CLI installation and command basics](lessons/008-angular-cli-installation-and-command-basics.md) | [Open Q&A](qnas/008-angular-cli-installation-and-command-basics.md) | [Open Example](examples/008-angular-cli-installation-and-command-basics.md) |
| 009 | [Create a standalone Angular application](lessons/009-create-a-standalone-angular-application.md) | [Open Q&A](qnas/009-create-a-standalone-angular-application.md) | [Open Example](examples/009-create-a-standalone-angular-application.md) |
| 010 | [Workspace structure, source files, and configuration](lessons/010-workspace-structure-source-files-and-configuration.md) | [Open Q&A](qnas/010-workspace-structure-source-files-and-configuration.md) | [Open Example](examples/010-workspace-structure-source-files-and-configuration.md) |
| 011 | [Run, serve, build, and inspect an Angular application](lessons/011-run-serve-build-and-inspect-an-angular-application.md) | [Open Q&A](qnas/011-run-serve-build-and-inspect-an-angular-application.md) | [Open Example](examples/011-run-serve-build-and-inspect-an-angular-application.md) |
| 012 | [Bootstrap, ApplicationConfig, and application providers](lessons/012-bootstrap-applicationconfig-and-application-providers.md) | [Open Q&A](qnas/012-bootstrap-applicationconfig-and-application-providers.md) | [Open Example](examples/012-bootstrap-applicationconfig-and-application-providers.md) |

### 2. Components, templates, and bindings

| No. | Lesson | Q&A | Example |
| --- | --- | --- | --- |
| 013 | [Components, metadata, and component boundaries](lessons/013-components-metadata-and-component-boundaries.md) | [Open Q&A](qnas/013-components-metadata-and-component-boundaries.md) | [Open Example](examples/013-components-metadata-and-component-boundaries.md) |
| 014 | [Standalone components and template imports](lessons/014-standalone-components-and-template-imports.md) | [Open Q&A](qnas/014-standalone-components-and-template-imports.md) | [Open Example](examples/014-standalone-components-and-template-imports.md) |
| 015 | [Selectors and the component host element](lessons/015-selectors-and-the-component-host-element.md) | [Open Q&A](qnas/015-selectors-and-the-component-host-element.md) | [Open Example](examples/015-selectors-and-the-component-host-element.md) |
| 016 | [Angular templates and HTML structure](lessons/016-angular-templates-and-html-structure.md) | [Open Q&A](qnas/016-angular-templates-and-html-structure.md) | [Open Example](examples/016-angular-templates-and-html-structure.md) |
| 017 | [Interpolation and template expressions](lessons/017-interpolation-and-template-expressions.md) | [Open Q&A](qnas/017-interpolation-and-template-expressions.md) | [Open Example](examples/017-interpolation-and-template-expressions.md) |
| 018 | [Property, attribute, class, and style bindings](lessons/018-property-attribute-class-and-style-bindings.md) | [Open Q&A](qnas/018-property-attribute-class-and-style-bindings.md) | [Open Example](examples/018-property-attribute-class-and-style-bindings.md) |
| 019 | [Event bindings, event objects, and keyboard input](lessons/019-event-bindings-event-objects-and-keyboard-input.md) | [Open Q&A](qnas/019-event-bindings-event-objects-and-keyboard-input.md) | [Open Example](examples/019-event-bindings-event-objects-and-keyboard-input.md) |
| 020 | [Two-way binding and ngModel](lessons/020-two-way-binding-and-ngmodel.md) | [Open Q&A](qnas/020-two-way-binding-and-ngmodel.md) | [Open Example](examples/020-two-way-binding-and-ngmodel.md) |
| 021 | [Built-in control flow with if and switch](lessons/021-built-in-control-flow-with-if-and-switch.md) | [Open Q&A](qnas/021-built-in-control-flow-with-if-and-switch.md) | [Open Example](examples/021-built-in-control-flow-with-if-and-switch.md) |
| 022 | [Render collections with for, track, and empty](lessons/022-render-collections-with-for-track-and-empty.md) | [Open Q&A](qnas/022-render-collections-with-for-track-and-empty.md) | [Open Example](examples/022-render-collections-with-for-track-and-empty.md) |
| 023 | [Template references, ng-template, and ngTemplateOutlet](lessons/023-template-references-ng-template-and-ngtemplateoutlet.md) | [Open Q&A](qnas/023-template-references-ng-template-and-ngtemplateoutlet.md) | [Open Example](examples/023-template-references-ng-template-and-ngtemplateoutlet.md) |
| 024 | [Built-in directives and structural directive syntax](lessons/024-built-in-directives-and-structural-directive-syntax.md) | [Open Q&A](qnas/024-built-in-directives-and-structural-directive-syntax.md) | [Open Example](examples/024-built-in-directives-and-structural-directive-syntax.md) |

### 3. Reusable components and template APIs

| No. | Lesson | Q&A | Example || --- | --- | --- | --- |
| 025 | [Signal inputs, required inputs, and input transforms](lessons/025-signal-inputs-required-inputs-and-input-transforms.md) | [Open Q&A](qnas/025-signal-inputs-required-inputs-and-input-transforms.md) | [Open Example](examples/025-signal-inputs-required-inputs-and-input-transforms.md) |
| 026 | [Component outputs and custom events](lessons/026-component-outputs-and-custom-events.md) | [Open Q&A](qnas/026-component-outputs-and-custom-events.md) | [Open Example](examples/026-component-outputs-and-custom-events.md) |
| 027 | [Model inputs and component-level two-way binding](lessons/027-model-inputs-and-component-level-two-way-binding.md) | [Open Q&A](qnas/027-model-inputs-and-component-level-two-way-binding.md) | [Open Example](examples/027-model-inputs-and-component-level-two-way-binding.md) |
| 028 | [Parent-child communication and one-way data flow](lessons/028-parent-child-communication-and-one-way-data-flow.md) | [Open Q&A](qnas/028-parent-child-communication-and-one-way-data-flow.md) | [Open Example](examples/028-parent-child-communication-and-one-way-data-flow.md) |
| 029 | [Content projection and multiple slots](lessons/029-content-projection-and-multiple-slots.md) | [Open Q&A](qnas/029-content-projection-and-multiple-slots.md) | [Open Example](examples/029-content-projection-and-multiple-slots.md) |
| 030 | [View queries and content queries](lessons/030-view-queries-and-content-queries.md) | [Open Q&A](qnas/030-view-queries-and-content-queries.md) | [Open Example](examples/030-view-queries-and-content-queries.md) |
| 031 | [Lifecycle hooks and component initialization](lessons/031-lifecycle-hooks-and-component-initialization.md) | [Open Q&A](qnas/031-lifecycle-hooks-and-component-initialization.md) | [Open Example](examples/031-lifecycle-hooks-and-component-initialization.md) |
| 032 | [Host bindings, host listeners, and host directives](lessons/032-host-bindings-host-listeners-and-host-directives.md) | [Open Q&A](qnas/032-host-bindings-host-listeners-and-host-directives.md) | [Open Example](examples/032-host-bindings-host-listeners-and-host-directives.md) |
| 033 | [Component styles and view encapsulation](lessons/033-component-styles-and-view-encapsulation.md) | [Open Q&A](qnas/033-component-styles-and-view-encapsulation.md) | [Open Example](examples/033-component-styles-and-view-encapsulation.md) |
| 034 | [Global styles, CSS variables, and theming](lessons/034-global-styles-css-variables-and-theming.md) | [Open Q&A](qnas/034-global-styles-css-variables-and-theming.md) | [Open Example](examples/034-global-styles-css-variables-and-theming.md) |
| 035 | [Built-in pipes and the async pipe](lessons/035-built-in-pipes-and-the-async-pipe.md) | [Open Q&A](qnas/035-built-in-pipes-and-the-async-pipe.md) | [Open Example](examples/035-built-in-pipes-and-the-async-pipe.md) |
| 036 | [Custom pipes, attribute directives, and reusable behavior](lessons/036-custom-pipes-attribute-directives-and-reusable-behavior.md) | [Open Q&A](qnas/036-custom-pipes-attribute-directives-and-reusable-behavior.md) | [Open Example](examples/036-custom-pipes-attribute-directives-and-reusable-behavior.md) |

### 4. Signals, state, and change detection

| No. | Lesson | Q&A | Example |
| --- | --- | --- | --- |
| 037 | [Writable signals and updating signal state](lessons/037-writable-signals-and-updating-signal-state.md) | [Open Q&A](qnas/037-writable-signals-and-updating-signal-state.md) | [Open Example](examples/037-writable-signals-and-updating-signal-state.md) |
| 038 | [Computed signals and derived values](lessons/038-computed-signals-and-derived-values.md) | [Open Q&A](qnas/038-computed-signals-and-derived-values.md) | [Open Example](examples/038-computed-signals-and-derived-values.md) |
| 039 | [linkedSignal and dependent writable state](lessons/039-linkedsignal-and-dependent-writable-state.md) | [Open Q&A](qnas/039-linkedsignal-and-dependent-writable-state.md) | [Open Example](examples/039-linkedsignal-and-dependent-writable-state.md) |
| 040 | [Effects, cleanup, and when not to use effects](lessons/040-effects-cleanup-and-when-not-to-use-effects.md) | [Open Q&A](qnas/040-effects-cleanup-and-when-not-to-use-effects.md) | [Open Example](examples/040-effects-cleanup-and-when-not-to-use-effects.md) |
| 041 | [Signal equality, dynamic dependencies, and untracked reads](lessons/041-signal-equality-dynamic-dependencies-and-untracked-reads.md) | [Open Q&A](qnas/041-signal-equality-dynamic-dependencies-and-untracked-reads.md) | [Open Example](examples/041-signal-equality-dynamic-dependencies-and-untracked-reads.md) |
| 042 | [RxJS interop with toSignal and toObservable](lessons/042-rxjs-interop-with-tosignal-and-toobservable.md) | [Open Q&A](qnas/042-rxjs-interop-with-tosignal-and-toobservable.md) | [Open Example](examples/042-rxjs-interop-with-tosignal-and-toobservable.md) || 043 | [Angular rendering and change detection](lessons/043-angular-rendering-and-change-detection.md) | [Open Q&A](qnas/043-angular-rendering-and-change-detection.md) | [Open Example](examples/043-angular-rendering-and-change-detection.md) |
| 044 | [OnPush change detection and immutable updates](lessons/044-onpush-change-detection-and-immutable-updates.md) | [Open Q&A](qnas/044-onpush-change-detection-and-immutable-updates.md) | [Open Example](examples/044-onpush-change-detection-and-immutable-updates.md) |
| 045 | [Zoneless Angular applications](lessons/045-zoneless-angular-applications.md) | [Open Q&A](qnas/045-zoneless-angular-applications.md) | [Open Example](examples/045-zoneless-angular-applications.md) |
| 046 | [State ownership and avoiding duplicated state](lessons/046-state-ownership-and-avoiding-duplicated-state.md) | [Open Q&A](qnas/046-state-ownership-and-avoiding-duplicated-state.md) | [Open Example](examples/046-state-ownership-and-avoiding-duplicated-state.md) |
| 047 | [Immutable state updates for arrays and objects](lessons/047-immutable-state-updates-for-arrays-and-objects.md) | [Open Q&A](qnas/047-immutable-state-updates-for-arrays-and-objects.md) | [Open Example](examples/047-immutable-state-updates-for-arrays-and-objects.md) |
| 048 | [Inspect signals and components with Angular DevTools](lessons/048-inspect-signals-and-components-with-angular-devtools.md) | [Open Q&A](qnas/048-inspect-signals-and-components-with-angular-devtools.md) | [Open Example](examples/048-inspect-signals-and-components-with-angular-devtools.md) |

### 5. Services, dependency injection, and RxJS

| No. | Lesson | Q&A | Example |
| --- | --- | --- | --- |
| 049 | [Services and the injectable pattern](lessons/049-services-and-the-injectable-pattern.md) | [Open Q&A](qnas/049-services-and-the-injectable-pattern.md) | [Open Example](examples/049-services-and-the-injectable-pattern.md) |
| 050 | [Inject dependencies with inject and constructor injection](lessons/050-inject-dependencies-with-inject-and-constructor-injection.md) | [Open Q&A](qnas/050-inject-dependencies-with-inject-and-constructor-injection.md) | [Open Example](examples/050-inject-dependencies-with-inject-and-constructor-injection.md) |
| 051 | [Hierarchical dependency injection](lessons/051-hierarchical-dependency-injection.md) | [Open Q&A](qnas/051-hierarchical-dependency-injection.md) | [Open Example](examples/051-hierarchical-dependency-injection.md) |
| 052 | [Application and environment providers](lessons/052-application-and-environment-providers.md) | [Open Q&A](qnas/052-application-and-environment-providers.md) | [Open Example](examples/052-application-and-environment-providers.md) |
| 053 | [Component-scoped and route-scoped providers](lessons/053-component-scoped-and-route-scoped-providers.md) | [Open Q&A](qnas/053-component-scoped-and-route-scoped-providers.md) | [Open Example](examples/053-component-scoped-and-route-scoped-providers.md) |
| 054 | [InjectionToken and typed application configuration](lessons/054-injectiontoken-and-typed-application-configuration.md) | [Open Q&A](qnas/054-injectiontoken-and-typed-application-configuration.md) | [Open Example](examples/054-injectiontoken-and-typed-application-configuration.md) |
| 055 | [Provider recipes, multi-providers, and provider functions](lessons/055-provider-recipes-multi-providers-and-provider-functions.md) | [Open Q&A](qnas/055-provider-recipes-multi-providers-and-provider-functions.md) | [Open Example](examples/055-provider-recipes-multi-providers-and-provider-functions.md) |
| 056 | [Injection contexts and resource lifetimes](lessons/056-injection-contexts-and-resource-lifetimes.md) | [Open Q&A](qnas/056-injection-contexts-and-resource-lifetimes.md) | [Open Example](examples/056-injection-contexts-and-resource-lifetimes.md) |
| 057 | [RxJS Observables, subscriptions, and stream basics](lessons/057-rxjs-observables-subscriptions-and-stream-basics.md) | [Open Q&A](qnas/057-rxjs-observables-subscriptions-and-stream-basics.md) | [Open Example](examples/057-rxjs-observables-subscriptions-and-stream-basics.md) |
| 058 | [RxJS operators for mapping and combining streams](lessons/058-rxjs-operators-for-mapping-and-combining-streams.md) | [Open Q&A](qnas/058-rxjs-operators-for-mapping-and-combining-streams.md) | [Open Example](examples/058-rxjs-operators-for-mapping-and-combining-streams.md) |
| 059 | [Flattening operators, cancellation, and concurrency](lessons/059-flattening-operators-cancellation-and-concurrency.md) | [Open Q&A](qnas/059-flattening-operators-cancellation-and-concurrency.md) | [Open Example](examples/059-flattening-operators-cancellation-and-concurrency.md) |
| 060 | [Error handling, cleanup, and takeUntilDestroyed](lessons/060-error-handling-cleanup-and-takeuntildestroyed.md) | [Open Q&A](qnas/060-error-handling-cleanup-and-takeuntildestroyed.md) | [Open Example](examples/060-error-handling-cleanup-and-takeuntildestroyed.md) |

### 6. Forms and user input
| No. | Lesson | Q&A | Example |
| --- | --- | --- | --- |
| 061 | [Choose template-driven, reactive, or signal forms](lessons/061-choose-template-driven-reactive-or-signal-forms.md) | [Open Q&A](qnas/061-choose-template-driven-reactive-or-signal-forms.md) | [Open Example](examples/061-choose-template-driven-reactive-or-signal-forms.md) |
| 062 | [Template-driven forms and ngForm](lessons/062-template-driven-forms-and-ngform.md) | [Open Q&A](qnas/062-template-driven-forms-and-ngform.md) | [Open Example](examples/062-template-driven-forms-and-ngform.md) |
| 063 | [Reactive forms with FormControl and FormGroup](lessons/063-reactive-forms-with-formcontrol-and-formgroup.md) | [Open Q&A](qnas/063-reactive-forms-with-formcontrol-and-formgroup.md) | [Open Example](examples/063-reactive-forms-with-formcontrol-and-formgroup.md) |
| 064 | [Typed reactive forms and non-nullable controls](lessons/064-typed-reactive-forms-and-non-nullable-controls.md) | [Open Q&A](qnas/064-typed-reactive-forms-and-non-nullable-controls.md) | [Open Example](examples/064-typed-reactive-forms-and-non-nullable-controls.md) |
| 065 | [FormArray and dynamic form structures](lessons/065-formarray-and-dynamic-form-structures.md) | [Open Q&A](qnas/065-formarray-and-dynamic-form-structures.md) | [Open Example](examples/065-formarray-and-dynamic-form-structures.md) |
| 066 | [Synchronous validators and validation state](lessons/066-synchronous-validators-and-validation-state.md) | [Open Q&A](qnas/066-synchronous-validators-and-validation-state.md) | [Open Example](examples/066-synchronous-validators-and-validation-state.md) |
| 067 | [Cross-field and asynchronous validation](lessons/067-cross-field-and-asynchronous-validation.md) | [Open Q&A](qnas/067-cross-field-and-asynchronous-validation.md) | [Open Example](examples/067-cross-field-and-asynchronous-validation.md) |
| 068 | [Submission flows, pending state, and reset behavior](lessons/068-submission-flows-pending-state-and-reset-behavior.md) | [Open Q&A](qnas/068-submission-flows-pending-state-and-reset-behavior.md) | [Open Example](examples/068-submission-flows-pending-state-and-reset-behavior.md) |
| 069 | [Accessible labels, instructions, and validation messages](lessons/069-accessible-labels-instructions-and-validation-messages.md) | [Open Q&A](qnas/069-accessible-labels-instructions-and-validation-messages.md) | [Open Example](examples/069-accessible-labels-instructions-and-validation-messages.md) |
| 070 | [Custom controls with ControlValueAccessor](lessons/070-custom-controls-with-controlvalueaccessor.md) | [Open Q&A](qnas/070-custom-controls-with-controlvalueaccessor.md) | [Open Example](examples/070-custom-controls-with-controlvalueaccessor.md) |
| 071 | [Signal Forms: models, fields, and form setup](lessons/071-signal-forms-models-fields-and-form-setup.md) | [Open Q&A](qnas/071-signal-forms-models-fields-and-form-setup.md) | [Open Example](examples/071-signal-forms-models-fields-and-form-setup.md) |
| 072 | [Signal Forms: schemas, validation, and custom controls](lessons/072-signal-forms-schemas-validation-and-custom-controls.md) | [Open Q&A](qnas/072-signal-forms-schemas-validation-and-custom-controls.md) | [Open Example](examples/072-signal-forms-schemas-validation-and-custom-controls.md) |

### 7. Routing and navigation

| No. | Lesson | Q&A | Example |
| --- | --- | --- | --- |
| 073 | [Configure the Angular Router in a standalone app](lessons/073-configure-the-angular-router-in-a-standalone-app.md) | [Open Q&A](qnas/073-configure-the-angular-router-in-a-standalone-app.md) | [Open Example](examples/073-configure-the-angular-router-in-a-standalone-app.md) |
| 074 | [RouterOutlet, RouterLink, and active route links](lessons/074-routeroutlet-routerlink-and-active-route-links.md) | [Open Q&A](qnas/074-routeroutlet-routerlink-and-active-route-links.md) | [Open Example](examples/074-routeroutlet-routerlink-and-active-route-links.md) |
| 075 | [Route parameters, snapshots, and reactive parameters](lessons/075-route-parameters-snapshots-and-reactive-parameters.md) | [Open Q&A](qnas/075-route-parameters-snapshots-and-reactive-parameters.md) | [Open Example](examples/075-route-parameters-snapshots-and-reactive-parameters.md) |
| 076 | [Query parameters, fragments, and navigation state](lessons/076-query-parameters-fragments-and-navigation-state.md) | [Open Q&A](qnas/076-query-parameters-fragments-and-navigation-state.md) | [Open Example](examples/076-query-parameters-fragments-and-navigation-state.md) |
| 077 | [Navigate programmatically and handle navigation results](lessons/077-navigate-programmatically-and-handle-navigation-results.md) | [Open Q&A](qnas/077-navigate-programmatically-and-handle-navigation-results.md) | [Open Example](examples/077-navigate-programmatically-and-handle-navigation-results.md) || 078 | [Route guards and access control boundaries](lessons/078-route-guards-and-access-control-boundaries.md) | [Open Q&A](qnas/078-route-guards-and-access-control-boundaries.md) | [Open Example](examples/078-route-guards-and-access-control-boundaries.md) |
| 079 | [Resolvers, route data, and loading a page before activation](lessons/079-resolvers-route-data-and-loading-a-page-before-activation.md) | [Open Q&A](qnas/079-resolvers-route-data-and-loading-a-page-before-activation.md) | [Open Example](examples/079-resolvers-route-data-and-loading-a-page-before-activation.md) |
| 080 | [Lazy-load routes and standalone components](lessons/080-lazy-load-routes-and-standalone-components.md) | [Open Q&A](qnas/080-lazy-load-routes-and-standalone-components.md) | [Open Example](examples/080-lazy-load-routes-and-standalone-components.md) |
| 081 | [Nested routes, child outlets, and auxiliary routes](lessons/081-nested-routes-child-outlets-and-auxiliary-routes.md) | [Open Q&A](qnas/081-nested-routes-child-outlets-and-auxiliary-routes.md) | [Open Example](examples/081-nested-routes-child-outlets-and-auxiliary-routes.md) |
| 082 | [Route-level providers and route configuration](lessons/082-route-level-providers-and-route-configuration.md) | [Open Q&A](qnas/082-route-level-providers-and-route-configuration.md) | [Open Example](examples/082-route-level-providers-and-route-configuration.md) |
| 083 | [Page titles, redirects, wildcard routes, and not-found pages](lessons/083-page-titles-redirects-wildcard-routes-and-not-found-pages.md) | [Open Q&A](qnas/083-page-titles-redirects-wildcard-routes-and-not-found-pages.md) | [Open Example](examples/083-page-titles-redirects-wildcard-routes-and-not-found-pages.md) |
| 084 | [Preloading, scroll restoration, and router testing](lessons/084-preloading-scroll-restoration-and-router-testing.md) | [Open Q&A](qnas/084-preloading-scroll-restoration-and-router-testing.md) | [Open Example](examples/084-preloading-scroll-restoration-and-router-testing.md) |

### 8. HTTP, APIs, and asynchronous data

| No. | Lesson | Q&A | Example |
| --- | --- | --- | --- |
| 085 | [Configure HttpClient with provideHttpClient](lessons/085-configure-httpclient-with-providehttpclient.md) | [Open Q&A](qnas/085-configure-httpclient-with-providehttpclient.md) | [Open Example](examples/085-configure-httpclient-with-providehttpclient.md) |
| 086 | [Build typed HTTP requests and API services](lessons/086-build-typed-http-requests-and-api-services.md) | [Open Q&A](qnas/086-build-typed-http-requests-and-api-services.md) | [Open Example](examples/086-build-typed-http-requests-and-api-services.md) |
| 087 | [URL parameters, headers, response types, and credentials](lessons/087-url-parameters-headers-response-types-and-credentials.md) | [Open Q&A](qnas/087-url-parameters-headers-response-types-and-credentials.md) | [Open Example](examples/087-url-parameters-headers-response-types-and-credentials.md) |
| 088 | [Upload and download progress events](lessons/088-upload-and-download-progress-events.md) | [Open Q&A](qnas/088-upload-and-download-progress-events.md) | [Open Example](examples/088-upload-and-download-progress-events.md) |
| 089 | [Functional interceptors and request context](lessons/089-functional-interceptors-and-request-context.md) | [Open Q&A](qnas/089-functional-interceptors-and-request-context.md) | [Open Example](examples/089-functional-interceptors-and-request-context.md) |
| 090 | [HTTP errors, retry policy, and timeout handling](lessons/090-http-errors-retry-policy-and-timeout-handling.md) | [Open Q&A](qnas/090-http-errors-retry-policy-and-timeout-handling.md) | [Open Example](examples/090-http-errors-retry-policy-and-timeout-handling.md) |
| 091 | [Cancel requests and prevent stale response races](lessons/091-cancel-requests-and-prevent-stale-response-races.md) | [Open Q&A](qnas/091-cancel-requests-and-prevent-stale-response-races.md) | [Open Example](examples/091-cancel-requests-and-prevent-stale-response-races.md) |
| 092 | [Authentication, cookies, and XSRF protection](lessons/092-authentication-cookies-and-xsrf-protection.md) | [Open Q&A](qnas/092-authentication-cookies-and-xsrf-protection.md) | [Open Example](examples/092-authentication-cookies-and-xsrf-protection.md) |
| 093 | [Caching, revalidation, and transfer cache](lessons/093-caching-revalidation-and-transfer-cache.md) | [Open Q&A](qnas/093-caching-revalidation-and-transfer-cache.md) | [Open Example](examples/093-caching-revalidation-and-transfer-cache.md) |
| 094 | [Loading, empty, success, and error UI states](lessons/094-loading-empty-success-and-error-ui-states.md) | [Open Q&A](qnas/094-loading-empty-success-and-error-ui-states.md) | [Open Example](examples/094-loading-empty-success-and-error-ui-states.md) || 095 | [Angular resources and signal-based asynchronous data](lessons/095-angular-resources-and-signal-based-asynchronous-data.md) | [Open Q&A](qnas/095-angular-resources-and-signal-based-asynchronous-data.md) | [Open Example](examples/095-angular-resources-and-signal-based-asynchronous-data.md) |
| 096 | [Server-sent events, WebSockets, and live updates](lessons/096-server-sent-events-websockets-and-live-updates.md) | [Open Q&A](qnas/096-server-sent-events-websockets-and-live-updates.md) | [Open Example](examples/096-server-sent-events-websockets-and-live-updates.md) |

### 9. Architecture, accessibility, and testing

| No. | Lesson | Q&A | Example |
| --- | --- | --- | --- |
| 097 | [Feature-based application architecture](lessons/097-feature-based-application-architecture.md) | [Open Q&A](qnas/097-feature-based-application-architecture.md) | [Open Example](examples/097-feature-based-application-architecture.md) |
| 098 | [Organize projects, libraries, and shared code](lessons/098-organize-projects-libraries-and-shared-code.md) | [Open Q&A](qnas/098-organize-projects-libraries-and-shared-code.md) | [Open Example](examples/098-organize-projects-libraries-and-shared-code.md) |
| 099 | [Reusable UI components and design-system boundaries](lessons/099-reusable-ui-components-and-design-system-boundaries.md) | [Open Q&A](qnas/099-reusable-ui-components-and-design-system-boundaries.md) | [Open Example](examples/099-reusable-ui-components-and-design-system-boundaries.md) |
| 100 | [Dynamic components and programmatic rendering](lessons/100-dynamic-components-and-programmatic-rendering.md) | [Open Q&A](qnas/100-dynamic-components-and-programmatic-rendering.md) | [Open Example](examples/100-dynamic-components-and-programmatic-rendering.md) |
| 101 | [Global error handling, logging, and diagnostics](lessons/101-global-error-handling-logging-and-diagnostics.md) | [Open Q&A](qnas/101-global-error-handling-logging-and-diagnostics.md) | [Open Example](examples/101-global-error-handling-logging-and-diagnostics.md) |
| 102 | [Internationalization, localization, and formatting](lessons/102-internationalization-localization-and-formatting.md) | [Open Q&A](qnas/102-internationalization-localization-and-formatting.md) | [Open Example](examples/102-internationalization-localization-and-formatting.md) |
| 103 | [Semantic HTML, keyboard access, and accessible Angular UI](lessons/103-semantic-html-keyboard-access-and-accessible-angular-ui.md) | [Open Q&A](qnas/103-semantic-html-keyboard-access-and-accessible-angular-ui.md) | [Open Example](examples/103-semantic-html-keyboard-access-and-accessible-angular-ui.md) |
| 104 | [Angular testing fundamentals and the CLI test runner](lessons/104-angular-testing-fundamentals-and-the-cli-test-runner.md) | [Open Q&A](qnas/104-angular-testing-fundamentals-and-the-cli-test-runner.md) | [Open Example](examples/104-angular-testing-fundamentals-and-the-cli-test-runner.md) |
| 105 | [Component tests, fixtures, and template behavior](lessons/105-component-tests-fixtures-and-template-behavior.md) | [Open Q&A](qnas/105-component-tests-fixtures-and-template-behavior.md) | [Open Example](examples/105-component-tests-fixtures-and-template-behavior.md) |
| 106 | [Test services, dependency injection, and HTTP requests](lessons/106-test-services-dependency-injection-and-http-requests.md) | [Open Q&A](qnas/106-test-services-dependency-injection-and-http-requests.md) | [Open Example](examples/106-test-services-dependency-injection-and-http-requests.md) |
| 107 | [Test routing, forms, accessibility, and user flows](lessons/107-test-routing-forms-accessibility-and-user-flows.md) | [Open Q&A](qnas/107-test-routing-forms-accessibility-and-user-flows.md) | [Open Example](examples/107-test-routing-forms-accessibility-and-user-flows.md) |
| 108 | [End-to-end testing, coverage, and test strategy](lessons/108-end-to-end-testing-coverage-and-test-strategy.md) | [Open Q&A](qnas/108-end-to-end-testing-coverage-and-test-strategy.md) | [Open Example](examples/108-end-to-end-testing-coverage-and-test-strategy.md) |

### 10. Performance, rendering, security, and delivery

| No. | Lesson | Q&A | Example |
| --- | --- | --- | --- |
| 109 | [Angular DevTools, debugging, and production diagnostics](lessons/109-angular-devtools-debugging-and-production-diagnostics.md) | [Open Q&A](qnas/109-angular-devtools-debugging-and-production-diagnostics.md) | [Open Example](examples/109-angular-devtools-debugging-and-production-diagnostics.md) |
| 110 | [Measure performance, bundle size, and Core Web Vitals](lessons/110-measure-performance-bundle-size-and-core-web-vitals.md) | [Open Q&A](qnas/110-measure-performance-bundle-size-and-core-web-vitals.md) | [Open Example](examples/110-measure-performance-bundle-size-and-core-web-vitals.md) || 111 | [Lazy loading and deferrable views with defer](lessons/111-lazy-loading-and-deferrable-views-with-defer.md) | [Open Q&A](qnas/111-lazy-loading-and-deferrable-views-with-defer.md) | [Open Example](examples/111-lazy-loading-and-deferrable-views-with-defer.md) |
| 112 | [Rendering performance, change detection, and profiling](lessons/112-rendering-performance-change-detection-and-profiling.md) | [Open Q&A](qnas/112-rendering-performance-change-detection-and-profiling.md) | [Open Example](examples/112-rendering-performance-change-detection-and-profiling.md) |
| 113 | [Client rendering, server rendering, and prerendering](lessons/113-client-rendering-server-rendering-and-prerendering.md) | [Open Q&A](qnas/113-client-rendering-server-rendering-and-prerendering.md) | [Open Example](examples/113-client-rendering-server-rendering-and-prerendering.md) |
| 114 | [Set up server-side rendering and hybrid route rendering](lessons/114-set-up-server-side-rendering-and-hybrid-route-rendering.md) | [Open Q&A](qnas/114-set-up-server-side-rendering-and-hybrid-route-rendering.md) | [Open Example](examples/114-set-up-server-side-rendering-and-hybrid-route-rendering.md) |
| 115 | [Hydration, event replay, and incremental hydration](lessons/115-hydration-event-replay-and-incremental-hydration.md) | [Open Q&A](qnas/115-hydration-event-replay-and-incremental-hydration.md) | [Open Example](examples/115-hydration-event-replay-and-incremental-hydration.md) |
| 116 | [Server-compatible code, metadata, and search visibility](lessons/116-server-compatible-code-metadata-and-search-visibility.md) | [Open Q&A](qnas/116-server-compatible-code-metadata-and-search-visibility.md) | [Open Example](examples/116-server-compatible-code-metadata-and-search-visibility.md) |
| 117 | [Security: XSS, sanitization, CSP, and trusted content](lessons/117-security-xss-sanitization-csp-and-trusted-content.md) | [Open Q&A](qnas/117-security-xss-sanitization-csp-and-trusted-content.md) | [Open Example](examples/117-security-xss-sanitization-csp-and-trusted-content.md) |
| 118 | [Authentication, authorization, secrets, and browser storage](lessons/118-authentication-authorization-secrets-and-browser-storage.md) | [Open Q&A](qnas/118-authentication-authorization-secrets-and-browser-storage.md) | [Open Example](examples/118-authentication-authorization-secrets-and-browser-storage.md) |
| 119 | [Service workers, offline behavior, deployment, and updates](lessons/119-service-workers-offline-behavior-deployment-and-updates.md) | [Open Q&A](qnas/119-service-workers-offline-behavior-deployment-and-updates.md) | [Open Example](examples/119-service-workers-offline-behavior-deployment-and-updates.md) |
| 120 | [Upgrade Angular safely and build an accessible capstone app](lessons/120-upgrade-angular-safely-and-build-an-accessible-capstone-app.md) | [Open Q&A](qnas/120-upgrade-angular-safely-and-build-an-accessible-capstone-app.md) | [Open Example](examples/120-upgrade-angular-safely-and-build-an-accessible-capstone-app.md) |

## Official references

- [Angular documentation](https://angular.dev/overview)
- [Angular installation](https://angular.dev/installation)
- [Angular CLI](https://angular.dev/tools/cli)
- [Angular version compatibility](https://angular.dev/reference/versions)
- [Angular release and support policy](https://angular.dev/reference/releases)
- [Angular templates and control flow](https://angular.dev/guide/templates)
- [Angular signals](https://angular.dev/guide/signals)
- [Angular forms overview](https://angular.dev/guide/forms)
- [Signal Forms](https://angular.dev/guide/forms/signals/overview)
- [Angular routing](https://angular.dev/guide/routing)
- [Angular HTTP](https://angular.dev/guide/http)
- [Angular testing](https://angular.dev/guide/testing)
- [Server-side and hybrid rendering](https://angular.dev/guide/ssr)
- [Hydration](https://angular.dev/guide/hydration)
- [Angular update guide](https://angular.dev/update)

## License

This repository is licensed under the MIT License. See [LICENSE](LICENSE).