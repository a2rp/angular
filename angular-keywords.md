# Angular Keywords and Concepts

A practical glossary for the Angular learning roadmap. It explains Angular APIs, template syntax, TypeScript terms used in Angular code, and the ideas that connect them.

Angular does not add a separate set of reserved programming-language words. This reference distinguishes framework APIs and template syntax from JavaScript and TypeScript keywords. API availability can depend on the Angular version and project setup; use the linked official documentation when a feature is new or version-sensitive.

This file is a companion reference, not a replacement for the lessons. Follow the [Angular course roadmap](README.md#course-roadmap).

## Contents

- [Core architecture](#core-architecture)
- [Workspace, CLI, and startup](#workspace-cli-and-startup)
- [Components and component APIs](#components-and-component-apis)
- [Template syntax and bindings](#template-syntax-and-bindings)
- [Lifecycle and change detection](#lifecycle-and-change-detection)
- [Signals and reactive state](#signals-and-reactive-state)
- [Dependency injection and providers](#dependency-injection-and-providers)
- [RxJS and asynchronous work](#rxjs-and-asynchronous-work)
- [Forms and validation](#forms-and-validation)
- [Router and navigation](#router-and-navigation)
- [HTTP and API communication](#http-and-api-communication)
- [Architecture, accessibility, and localization](#architecture-accessibility-and-localization)
- [Testing and debugging](#testing-and-debugging)
- [Performance, rendering, and delivery](#performance-rendering-and-delivery)
- [Security and browser concerns](#security-and-browser-concerns)
- [TypeScript and JavaScript words](#typescript-and-javascript-words)
- [Official references](#official-references)

## Core architecture

| Term | Meaning |
| --- | --- |
| Angular | A TypeScript-based framework for building web applications from components, templates, services, and other framework APIs. |
| AngularJS | The older 1.x framework. It is separate from modern Angular; AngularJS applications are not modern Angular applications. |
| Component-based application | An application composed from UI units with explicit inputs, outputs, templates, and responsibilities. |
| Component tree | The nested parent-and-child structure of components that make up rendered UI. |
| Single-page application (SPA) | An application that can change views and URL state in the browser without requesting a complete new document for each navigation. |
| Declarative UI | Describing what the UI should look like for current data and state, while Angular updates the rendered view. |
| Template | HTML enhanced with Angular bindings, expressions, control flow, variables, and other template syntax. |
| View | Angular's representation of rendered template content. Components own views; embedded templates can create additional views. |
| Directive | A class or built-in construct that adds behavior to an element or template. Components are directives with their own templates. |
| Service | A class or value that provides reusable application behavior or data, commonly supplied through dependency injection. |
| Dependency injection (DI) | A mechanism for providing code with dependencies instead of constructing them internally. |
| Provider | A recipe that tells Angular how to create or obtain a dependency. |
| Standalone application | An application built with standalone components and providers without requiring an NgModule for application structure. |
| NgModule | A legacy and still supported Angular grouping mechanism. Modern standalone applications generally do not need NgModules for new code. |
| Zone.js | A library historically used to observe asynchronous work and schedule change detection. Applications can also use zoneless change detection. |
| Reactive state | State whose consumers are notified or re-evaluated when dependencies change. Signals and RxJS streams are distinct reactive tools. |
| Source of truth | The authoritative owner of a value. Keeping one owner prevents conflicting copies. |
| One-way data flow | Data usually moves from parent to child through inputs; child components report events or requested changes through outputs. |
| Framework API | An Angular-provided function, class, decorator, token, or template construct. It is distinct from a language keyword. |

## Workspace, CLI, and startup

| Term | Meaning |
| --- | --- |
| Angular CLI | Command-line tool for creating, serving, testing, building, and updating Angular projects. |
| Workspace | Top-level project area described by Angular workspace configuration; it can contain applications and libraries. |
| Application project | A configured Angular application within a workspace. |
| Library project | Reusable code that can be consumed by applications or published as a package. |
| Builder | CLI integration that performs a task such as building, serving, testing, or extracting translations. |
| Target | A named CLI operation for a project, such as build, serve, or test. |
| angular.json | Workspace configuration for projects, builders, assets, styles, and CLI targets. |
| package.json | npm manifest containing project metadata, scripts, and dependency declarations. |
| package-lock.json | npm lockfile recording the resolved dependency tree for reproducible installs. |
| tsconfig.json | TypeScript compiler configuration. A workspace can have root and application-specific configurations. |
| Source root | Directory containing an application's source files, commonly src. |
| ng new | CLI command that creates a new Angular workspace and application. |
| ng serve | CLI command that builds and serves an application for local development. |
| ng build | CLI command that builds application output for a selected configuration. |
| ng test | CLI command that runs configured unit tests. Runner defaults depend on the project's CLI version and setup. |
| ng update | CLI command used to update Angular packages and apply available migrations. |
| Schematic | Rule-based generator used by the CLI and libraries to create or modify project files. |
| Migration | Automated source or configuration transformation that helps move a project between versions or APIs. |
| Development build | Build intended for local work, generally with diagnostics and fewer production optimizations. |
| Production build | Optimized build intended for deployment. |
| Tree shaking | Removing unused code from the final bundle when the build can prove that code is unreachable. |
| Source map | Mapping from bundled output back to original source files for debugging. |
| Strict template checking | Compiler checks that catch many template type errors before the application runs. |

## Components and component APIs

| Term | Meaning |
| --- | --- |
| @Component | TypeScript decorator that marks a class as a component and supplies its metadata. |
| Component metadata | Configuration such as selector, template, styles, imports, providers, and change-detection options. |
| Selector | Element or attribute pattern used to place a component or directive in a template. |
| Host element | DOM element represented by a component's selector. |
| template / templateUrl | Inline HTML in metadata or the path to an external component template. |
| styles / styleUrl / styleUrls | Inline styles or paths to component stylesheet files. Supported metadata spellings can vary by Angular version. |
| imports | Standalone component metadata listing the components, directives, pipes, or NgModules its template uses. |
| input() | Signal-based API declaring a component or directive input; reading the returned signal gives the current value. |
| input.required() | Signal input declaration requiring a caller to supply a value. |
| @Input | Decorator-based input API that remains common in existing code. |
| output() | API declaring a component or directive event output. |
| @Output and EventEmitter | Decorator-based output pattern common in existing Angular code. |
| model() | API for a model input that supports a value and corresponding change output for component-level two-way binding. |
| Input transform | Function applied to an input value before the component reads it, useful for normalization or coercion. |
| Required input | Input that a caller must provide; template compilation can report a missing value. |
| Component API | Public inputs, outputs, models, projected slots, and documented behavior exposed to callers. |
| Content projection | Inserting caller-provided child content into a component template with ng-content. |
| Projection slot | A selected location that receives matching projected content. |
| ng-content | Template element marking where projected child content should render. |
| ng-template | Template fragment declared but not rendered until Angular instantiates it. |
| ng-container | Logical grouping element that does not add a wrapper to the rendered DOM. |
| Template reference variable | Template-local name declared with # to refer to an element, directive, or template reference. |
| View query | Query for elements or directives in a component's own view, commonly using viewChild or viewChildren. |
| Content query | Query for projected content, commonly using contentChild or contentChildren. |
| Host binding | Binding that sets an attribute, class, style, or property on a directive or component host. |
| Host listener | Listener for an event on a host element. Prefer template event bindings when they fit. |
| hostDirectives | Metadata that composes directive behavior onto a component or directive host. |
| Encapsulation | How component styles are scoped or applied. Angular's default emulated mode scopes styles through generated attributes. |
| ShadowDom encapsulation | Style encapsulation using browser Shadow DOM; it affects style boundaries and projected content. |
| None encapsulation | Component styles become global when encapsulation is disabled. |
| Component composition | Building larger UI by combining components and passing content or data through clear APIs. |
| ControlValueAccessor | Adapter interface connecting a custom UI control to Angular's forms APIs. |

## Template syntax and bindings

| Term | Meaning |
| --- | --- |
| Interpolation | Double-curly syntax such as {{ value }} that displays an expression as text. |
| Template expression | Restricted expression evaluated in a template context; it is not unrestricted JavaScript. |
| Template statement | An action expression used in response to an event. |
| Property binding | Bracket syntax such as [disabled]="isSaving" that sets a DOM property or directive/component input. |
| Attribute binding | Syntax such as [attr.aria-label]="label" that sets an HTML attribute rather than a DOM property. |
| Class binding | Syntax such as [class.active]="isActive" that conditionally applies a CSS class. |
| Style binding | Syntax such as [style.width.px]="width" that sets an inline style property. |
| Event binding | Parentheses syntax such as (click)="save()" that listens for an event. |
| Two-way binding | Syntax that sends a value to a target and propagates changes back, conventionally written with brackets and parentheses together. |
| [(ngModel)] | Two-way form-value binding through template-driven forms. It requires the relevant forms support to be imported. |
| Event object | Browser event value available to a template event expression, often referenced as $event. |
| Safe navigation | The ?. operator for reading a property only if the preceding value is not nullish. |
| Non-null assertion in templates | The ! marker used in supported template expressions to assert non-nullability to the type checker; it does not validate at runtime. |
| Built-in control flow | Angular template syntax such as @if, @for, and @switch for conditional and repeated content. |
| @if / @else if / @else | Conditional blocks for selecting which template branch to render. |
| @switch / @case / @default | Switch-style control flow for selecting a template branch from a value. |
| @for | Built-in loop block for rendering each item in a collection. |
| track expression | Stable identity expression in @for that lets Angular match collection items to rendered views efficiently. |
| $index / $count | @for context values for an item's position and the collection size. |
| $first / $last / $even / $odd | @for context values identifying item positions in a collection. |
| @empty | Optional @for block rendered when a collection has no items. |
| @let | Template declaration for a local value derived from an expression. |
| @defer | Deferrable view block that delays eligible dependencies until a trigger occurs. |
| @placeholder / @loading / @error | Optional template blocks for before loading, while loading, and when deferred loading fails. |
| Defer trigger | Condition such as viewport visibility, interaction, idle time, or a timer that starts a deferred block. |
| Template fragment | Reusable or conditionally instantiated section of a template, often declared with ng-template. |
| ngTemplateOutlet | Directive that renders an ng-template fragment and can pass it a context object. |
| Pipe | Declarative template transform written after the pipe character, such as date or currency formatting. |
| Built-in pipe | Angular-provided transform such as DatePipe, CurrencyPipe, DecimalPipe, PercentPipe, JsonPipe, or AsyncPipe. |
| AsyncPipe | Subscribes to an Observable or Promise in a template, exposes its latest value, and handles subscription cleanup. |
| Custom pipe | Developer-defined transform declared with @Pipe. Keep frequently evaluated template work inexpensive. |
| Structural directive | Directive that adds, removes, or repeats template content. The asterisk shorthand expands into an ng-template form. |
| Attribute directive | Directive that changes behavior or appearance of an existing element without owning a separate template. |
| Built-in directive | Angular directive such as NgClass, NgStyle, NgIf, or NgForOf. Newer control-flow blocks replace many common structural uses. |
| Expression limitations | Template expressions exclude some JavaScript syntax and have their own supported operators and globals. Keep complex logic in TypeScript. |
## Lifecycle and change detection

| Term | Meaning |
| --- | --- |
| Lifecycle hook | Method or render callback Angular invokes at a particular stage of a component or directive's lifetime. |
| ngOnChanges | Hook called when data-bound inputs change; its first invocation occurs before ngOnInit. |
| SimpleChanges | Object passed to ngOnChanges describing previous and current input values and whether each is initializing. |
| ngOnInit | Hook called once after Angular initializes a component's inputs. |
| ngDoCheck | Hook for custom change checking. It runs frequently and should be used only when necessary. |
| ngAfterContentInit / ngAfterContentChecked | Hooks called after projected content initializes and after subsequent checks. |
| ngAfterViewInit / ngAfterViewChecked | Hooks called after the component view initializes and after subsequent checks. |
| ngOnDestroy | Hook called before Angular destroys a component, directive, or service instance. |
| DestroyRef | API for registering cleanup tied to the lifetime of an injection context. |
| afterNextRender | Callback for work after Angular next renders to the DOM. Browser-only integrations must account for SSR. |
| afterEveryRender | Callback invoked after each render, with read/write phases available in current APIs. Use sparingly. |
| Change detection | Angular's process for checking state and updating the rendered view. |
| OnPush | Change-detection strategy that lets Angular skip subtrees when inputs and tracked state have not changed. Signals read in templates can mark a component when they change. |
| ChangeDetectorRef | API for interacting with a view's change detection, including marking it for checking or detecting changes. |
| markForCheck() | Marks a view for a future change-detection pass. |
| detectChanges() | Immediately checks a view and its children. Manual calls should be uncommon in normal component code. |
| Zoneless change detection | Running Angular without Zone.js scheduling; framework notifications such as signal changes and events drive updates. |
| Immutable update | Creating a new object or array for a state change instead of mutating the existing value in place. |
| Angular DevTools | Browser extension for inspecting Angular component trees, change detection, and supported profiling information. |

## Signals and reactive state

| Term | Meaning |
| --- | --- |
| Signal | Callable reactive value that tracks where it is read and notifies consumers when its value changes. |
| signal() | Creates a writable signal with an initial value. |
| Writable signal | Signal that can be changed through APIs such as set() or update(). |
| Read-only signal | Signal that exposes reads but not writable operations to its consumer. |
| set(value) / update(fn) | Writable-signal methods to replace the value or calculate a new value from the current one. |
| computed() | Creates a read-only signal derived from other signals. Dependencies are collected when its derivation runs. |
| linkedSignal() | Creates writable state dependent on source signals while allowing local edits. |
| effect() | Runs code in response to signals it reads. Prefer computed values for derived state; effects are mainly for synchronizing with imperative, non-reactive APIs. |
| Effect cleanup | Cleanup registered by an effect before its next execution or when its owner is destroyed. |
| Reactive context | Synchronous execution context where signal reads are tracked as dependencies. |
| Producer / consumer | A producer is a signal being read; a consumer is a reactive computation or template that depends on it. |
| Dynamic dependency | Dependency collected from signal reads made during the latest execution. Conditional reads can change the dependency set. |
| untracked() | Reads a signal or executes a callback without adding its reads as dependencies of the current reactive computation. |
| Equality function | Comparison used to decide whether a proposed signal value counts as a change. The default uses referential equality. |
| Signal graph | Dependency relationships between signals and the computations or templates that consume them. |
| input() signal | Read-only signal whose value is supplied by a component or directive caller. |
| output() | Component or directive event API for notifying a parent or consumer. It is event communication, not shared state. |
| model() signal | Writable component input API paired with a change event. |
| toSignal() | RxJS interop API that subscribes to an Observable and exposes its latest value as a signal. |
| toObservable() | RxJS interop API that exposes signal changes as an Observable. |
| resource() | Signal-oriented API for representing asynchronous data and loading, value, and error state. Check version and usage constraints. |
| httpResource() | Signal-oriented HTTP API for reactive requests and response state. Confirm availability for the project's Angular version. |
| RxJS interop | APIs for using signals and RxJS Observables together without treating them as the same abstraction. |
| Derived state | Value calculated from other state. Prefer computed() over a duplicate that can become stale. |
| State ownership | Decision about which component, service, or store is responsible for changing and exposing a value. |
| State colocation | Keeping state near the components that use it to reduce unnecessary sharing and coupling. |
| Signal Forms | Signal-oriented forms with a model, field state, and validation. Availability and maturity depend on Angular version. |

## Dependency injection and providers

| Term | Meaning |
| --- | --- |
| inject() | Function for obtaining a dependency in a valid injection context. |
| Constructor injection | Pattern where Angular supplies dependencies as constructor parameters. It remains supported and widely used. |
| Injection context | Synchronous context in which APIs such as inject() can resolve dependencies. |
| runInInjectionContext() | Runs a callback with a supplied injector as the active injection context. |
| @Injectable | Decorator marking a class as injectable; metadata can declare provider registration such as providedIn. |
| providedIn: 'root' | Registers a service with the application root injector for tree-shakable availability. |
| Injector | Object that resolves dependency tokens to configured values. |
| EnvironmentInjector | Injector for application or environment-level providers, separate from an element injector. |
| ElementInjector | Injector associated with an element and its component/directive hierarchy. |
| Hierarchical DI | Parent and child injector structure controlling visibility and which dependency instance resolves. |
| Provider configuration | Object or function describing how Angular should provide a token. |
| EnvironmentProviders | Provider values intended for environment-level configuration functions. |
| ApplicationConfig | Application-level configuration object commonly used to register providers during bootstrap. |
| provide* function | Functional setup API such as provideRouter() or provideHttpClient() that returns providers for a feature. |
| InjectionToken | Runtime token for injecting a value without a class token, often configuration or an interface-backed value. |
| useClass / useValue / useFactory / useExisting | Provider strategies for constructing a class, supplying a value, calling a factory, or aliasing another token. |
| Multi-provider | Configuration that collects several values for one token instead of replacing one value with another. |
| Optional injection | DI behavior allowing a missing dependency to resolve as null rather than throwing. |
| Self / SkipSelf / Host | DI resolution modifiers constraining where Angular searches for a dependency. |
| Provider scope | Injector location where a provider is registered; it determines visibility and instance lifetime. |
| Route-scoped provider | Dependency configuration attached to a route and available in its injector subtree. |
| Component-scoped provider | Dependency configuration attached to a component and available in that component's subtree. |
| Service lifetime | Period an instance remains available, determined by its owning injector and destruction lifecycle. |

## RxJS and asynchronous work

| Term | Meaning |
| --- | --- |
| RxJS | Library for composing asynchronous and event-based work with Observables and operators. |
| Observable | Lazy or event-driven stream that can emit values, errors, and completion notifications to subscribers. |
| Observer | The next, error, and complete callbacks that consume an Observable. |
| Subscription | Handle returned by subscribing that can stop an active Observable execution. |
| Subject | Observable and observer combined, often used to multicast events. Do not use it in place of clear state ownership without reason. |
| Operator | Function that transforms or combines an Observable pipeline. |
| map / filter | Operators that respectively transform values or forward only values satisfying a predicate. |
| switchMap | Switches to a new inner Observable and unsubscribes from the previous one; useful when stale search results should be discarded. |
| concatMap | Queues inner work and processes it sequentially. |
| mergeMap | Runs multiple inner operations concurrently. |
| exhaustMap | Ignores new source values while the current inner operation is active. |
| catchError | Operator for recovering from or rethrowing an Observable error. |
| finalize | Operator that runs cleanup when a stream terminates or is unsubscribed. |
| takeUntilDestroyed | Angular RxJS interop operator that ends a subscription when its associated context is destroyed. |
| Cold Observable | Observable whose work typically starts separately for each subscription. |
| Hot Observable | Observable that emits independently of a particular subscriber and may share events. |
| Completion | Notification that an Observable will emit no more values. |
| Cancellation | Stopping or unsubscribing from asynchronous work. Whether the underlying operation stops depends on its source. |
| Async boundary | Point such as await where synchronous execution pauses. Signal dependency tracking does not automatically continue across arbitrary async boundaries. |
| Race condition | Timing bug where overlapping results arrive in an order that conflicts with the latest user intent. |
| Stale response | Response belonging to an older request or state that should no longer update the UI. |

## Forms and validation

| Term | Meaning |
| --- | --- |
| Template-driven forms | Forms primarily configured through template directives such as ngForm and ngModel. |
| Reactive forms | Forms modeled explicitly in TypeScript with controls, groups, arrays, and observable state. |
| Typed reactive forms | Reactive forms whose control values and structure have TypeScript types. |
| Signal Forms | Signal-oriented forms with models, field state, validation, and custom controls; check version-specific support. |
| FormsModule | Angular module providing template-driven form directives such as ngModel. |
| ReactiveFormsModule | Angular module providing reactive form directives and bindings. |
| ngForm | Directive tracking a template-driven form's controls, value, and status. |
| ngModel | Directive binding a form control to a component value in template-driven forms. |
| FormControl | Reactive form object for one field's value, validation, and interaction state. |
| FormGroup | Reactive form object grouping named child controls. |
| FormArray | Reactive form object holding an ordered collection of controls, often with a variable number of entries. |
| FormRecord | Reactive form object holding controls keyed by dynamic string names. |
| FormBuilder / NonNullableFormBuilder | Helpers for creating controls and groups; the non-nullable builder creates non-nullable controls by default. |
| Control state | Properties such as valid, invalid, pending, disabled, touched, untouched, dirty, and pristine. |
| pristine / dirty | pristine means the value has not changed through interaction; dirty means it has. |
| touched / untouched | touched means the control has been blurred or explicitly marked; untouched means it has not. |
| pending | A control or form has asynchronous validation still in progress. |
| ValidatorFn / AsyncValidatorFn | Synchronous or asynchronous function type returning validation results. |
| Built-in validators | Reusable checks such as required, min, max, email, minLength, maxLength, pattern, and requiredTrue. |
| Cross-field validation | Validation comparing multiple controls, commonly implemented at a parent FormGroup level. |
| setValue() | Updates a control or group with a value matching its full structure. |
| patchValue() | Updates supplied values and leaves other controls unchanged. |
| reset() | Resets a control or form and can restore pristine and untouched state. |
| getRawValue() | Reads group values including disabled controls, unlike normal aggregate value behavior for disabled children. |
| Form status | VALID, INVALID, PENDING, or DISABLED. |
| Accessible error message | Text associated with a field that explains a validation issue and can be reached by assistive technology. |
| Signal form model / field state | Signal-backed data and field metadata such as value, validity, touched state, and errors. |
| Schema validation | Rules declared for Signal Forms fields or models according to the current API. |
## Router and navigation

| Term | Meaning |
| --- | --- |
| Angular Router | Angular's routing library for mapping URLs to views and navigation behavior. |
| Routes / Route | Array of route configurations; each Route can define a path, component or loader, providers, guards, and resolvers. |
| provideRouter() | Provider function that registers router configuration in a standalone application. |
| RouterOutlet | Placeholder where the router renders the component for the active route. |
| RouterLink | Directive that navigates when a link is activated. |
| RouterLinkActive | Directive that applies classes or state when a link matches the active route. |
| Router | Service for programmatic navigation and router state access. |
| ActivatedRoute | Service representing the route active at a component's position in the route tree. |
| Route parameter | Dynamic path value such as an item identifier within a route pattern. |
| ParamMap | Read-only map of path or query parameter values. |
| Snapshot | One-time view of route state. It does not update when the route changes. |
| Observable route state | Reactive route values such as params or queryParams that emit when navigation changes them. |
| Query parameter | Key-value data in the URL after ?, often used for filters, search, or pagination. |
| URL fragment | Portion after #, often used to identify a page section. |
| Navigation extras | Options passed to router navigation, including query handling and navigation state. |
| navigate() / navigateByUrl() | Router methods for navigating with command segments or a URL/UrlTree. |
| UrlTree | Structured representation of a URL used by the router. |
| Route guard | Function that allows, cancels, or redirects navigation. A client guard is not server authorization. |
| CanActivateFn / CanActivateChildFn | Functional guards deciding whether a route or its child route can activate. |
| CanDeactivateFn / CanMatchFn | Functional guards deciding whether navigation can leave a route or whether a route should match. |
| ResolveFn | Functional route resolver obtaining data before activation. |
| Route data | Static or resolved data attached to a route. |
| Lazy loading | Loading route code only when it is needed, commonly with loadComponent or loadChildren. |
| loadComponent / loadChildren | Route functions that lazily load a standalone component or child routes. |
| Nested route | Route configured under a parent and displayed through a child outlet. |
| Auxiliary route | Named-outlet route that can render alongside the primary outlet. |
| Redirect / wildcard route | Route that redirects elsewhere or catches otherwise unmatched paths, commonly for not-found UI. |
| Preloading | Loading lazy route code in the background according to a router strategy. |
| Scroll restoration | Router behavior for restoring or controlling scroll position after navigation. |
| Navigation event | Router event emitted as navigation proceeds, succeeds, redirects, or fails. |
| Route title | Static or resolved document title configured for a route. |

## HTTP and API communication

| Term | Meaning |
| --- | --- |
| HttpClient | Angular service for making HTTP requests and receiving typed Observable responses. |
| provideHttpClient() | Provider function configuring HttpClient and optional features. |
| HttpClientModule | NgModule-based HTTP setup found in existing applications; standalone projects commonly use providers. |
| HTTP request / response | Request sends a method, URL, headers, and optional body; response returns status, headers, and body. |
| HttpHeaders / HttpParams | Immutable collections for request headers and URL query parameters. |
| HttpContext | Typed request metadata read by interceptors without sending it as a header. |
| observe option | Selects body-only response, full response, or an event stream. |
| responseType | Selects response representation such as JSON, text, or binary data. |
| HttpResponse | Full successful response including body, headers, and status. |
| HttpErrorResponse | Error wrapper for failed HTTP requests or response parsing errors. |
| HttpInterceptorFn | Functional interceptor type for inspecting or transforming requests and responses. |
| withInterceptors() | HttpClient feature for registering functional interceptors. |
| HttpHandlerFn | Function through which an interceptor passes a request to the next handler. |
| Interceptor chain | Ordered sequence of request/response middleware functions. |
| Request context token | Typed key for interceptor-specific metadata stored in HttpContext. |
| Typed response | TypeScript annotation for an expected response shape; it does not validate server JSON at runtime. |
| Progress event | Event describing upload or download progress when configured and supported by the backend. |
| Retry policy / timeout | Deliberate rules for whether to retry a failed request and how long an operation may run. |
| XSRF / CSRF | Cross-site request forgery risk. Client token support does not replace server validation and correct cookie configuration. |
| CORS | Browser policy controlling whether a page can read a cross-origin response; the server configures allowed origins and headers. |
| Transfer cache | Reuse of eligible server-rendered HTTP response data during client startup to avoid duplicate requests. |
| Server-Sent Events (SSE) | One-way stream of server updates over an HTTP connection. |
| WebSocket | Persistent bidirectional connection for live messages. |
| API service | Injectable abstraction grouping requests for a backend resource or API boundary. |

## Architecture, accessibility, and localization

| Term | Meaning |
| --- | --- |
| Feature-based architecture | Organizing code around user-facing features or business capabilities instead of only technical file types. |
| Shared code | Reusable functionality used across features. Keep shared APIs focused to avoid a dumping ground. |
| Design system | Reusable visual rules, tokens, components, and interaction patterns used across an application. |
| Dynamic component | Component selected or created at runtime instead of declared statically in a template. |
| ViewContainerRef | API representing a view container where embedded or component views can be created. |
| createComponent() | API for programmatically creating a component in a supported rendering context. |
| ErrorHandler | Application-level error-handling abstraction. Logging and reporting still need application-specific design. |
| Internationalization (i18n) | Designing an application so content and behavior can be adapted to different languages and regions. |
| Localization (l10n) | Adapting text, dates, numbers, currencies, and conventions for a particular locale. |
| Locale / LOCALE_ID | Regional language rules and Angular injection token for locale-dependent formatting configuration. |
| Semantic HTML | Native HTML elements whose meaning and built-in behavior communicate structure and interaction. |
| Accessible name | Name exposed to assistive technology, commonly provided through visible text or an associated label. |
| Keyboard accessibility | Ability to use interactions by keyboard, with visible focus and logical focus order. |
| Focus management | Deliberate movement and restoration of keyboard focus when UI changes or navigation occurs. |
| ARIA | Attributes for communicating roles, states, and relationships when native HTML alone is insufficient. |
| Core Web Vitals | User-experience metrics for loading, interactivity, and visual stability. Measure real-user experience as well as lab results. |

## Testing and debugging

| Term | Meaning |
| --- | --- |
| Unit test | Test of a small unit, such as a function, pipe, service, or component behavior, with controlled dependencies. |
| Component test | Test exercising a component with its template, inputs, outputs, and interactions. |
| Integration test | Test checking collaborating parts together, such as a form with a service or a route with a component. |
| End-to-end (E2E) test | Browser-level test of a user flow. The automation tool is selected separately from Angular itself. |
| Test runner | Tool that discovers and executes tests. Defaults depend on CLI version and project configuration. |
| TestBed | Angular testing utility for configuring a test injector and creating or compiling Angular classes and components. |
| ComponentFixture | Test handle for a component instance, rendered element, and change-detection operations. |
| DebugElement | Test abstraction for inspecting an element and Angular properties or listeners. |
| fixture.detectChanges() | Test operation that runs change detection for a component fixture. |
| waitForAsync | Test helper for waiting for asynchronous work tracked by the Angular testing environment. |
| fakeAsync / tick | Helpers for controlling certain asynchronous tasks and timers in a compatible test environment. |
| By.css | Predicate helper for finding elements by CSS selector in Angular tests. |
| HttpTestingController | Utility for asserting and responding to HttpClient requests in tests. |
| provideHttpClientTesting() | Provider function configuring HTTP testing support for the test injector. |
| RouterTestingHarness | Helper for exercising routing and routed components in supported project setups. |
| Test double | Replacement dependency used in tests, such as a stub, fake, or mock. |
| Coverage | Measurement of code or branches exercised by tests. High coverage alone does not prove correctness. |
| Regression test | Test that protects against a previously fixed bug returning. |
| Browser developer tools | Browser utilities for inspecting DOM, styles, network requests, errors, and runtime performance. |

## Performance, rendering, and delivery

| Term | Meaning |
| --- | --- |
| Bundle / bundle size | Generated JavaScript or CSS output and the amount of code delivered to the browser. Consider transfer, parsing, and execution cost. |
| Lazy loading | Deferring code loading until a route or feature is needed. |
| Deferrable view | Template section managed by @defer and loaded when a trigger is satisfied. |
| Client-side rendering (CSR) | Rendering the application in the browser after JavaScript loads. |
| Server-side rendering (SSR) | Rendering application HTML on a server for a request, then optionally continuing in the browser. |
| Prerendering / static site generation (SSG) | Generating HTML ahead of a request for known routes or content. |
| Hybrid rendering | Using different rendering modes for different routes, such as server rendering, prerendering, and client rendering. |
| RenderMode.Client / Server / Prerender | Route rendering choices in supported Angular hybrid-rendering configuration. Check version-specific APIs. |
| Hydration | Connecting client-side Angular behavior to server-rendered HTML instead of replacing the whole page. |
| Event replay | Capturing eligible events before hydration completes and replaying them afterward. |
| Incremental hydration | Hydrating selected portions of server-rendered UI at different times, often coordinated with defer triggers. |
| TransferState | Mechanism for transferring selected state from server rendering to the browser at startup. |
| SSR-safe code | Code that does not assume browser globals such as window or localStorage exist during server rendering. |
| DOCUMENT | Injection token for the current document abstraction; an SSR document need not behave like a browser document. |
| isPlatformBrowser | Helper for checking whether code is executing in a browser platform context. |
| Search engine optimization (SEO) | Improving discovery and representation of content. Rendering mode is only one part of SEO. |
| Meta service | Angular API for setting document title and metadata; values still need to reflect the page's content. |
| Service worker | Browser worker that can cache assets and support offline/update behavior when configured. |
| Offline behavior | Application behavior when a network is unavailable or intermittent. |
| Deployment | Publishing build output to hosting or a server that supports the app's routing and rendering mode. |
| Base href | URL prefix used by the browser to resolve application-relative links and assets. |
| Upgrade guide | Version-specific instructions and migrations for moving a project to a newer Angular release. |
| Semantic versioning | Release numbering convention for compatibility expectations; check actual project dependencies and Angular support policy. |

## Security and browser concerns

| Term | Meaning |
| --- | --- |
| Cross-site scripting (XSS) | Injection of attacker-controlled script or markup. Avoid unsafe DOM writes and treat external data as untrusted. |
| Sanitization | Removing or neutralizing unsafe content for a security context. Angular sanitizes many template-bound values automatically. |
| Security context | Interpretation of a bound value such as HTML, style, URL, or resource URL; each context has different risks. |
| DomSanitizer | Service for sanitizing values or marking a value trusted for a context. Trust-bypass APIs are security-sensitive. |
| SafeValue | Angular wrapper for a value explicitly marked trusted in a security context; it does not prove the value is safe. |
| Trusted Types | Browser policy mechanism that can restrict injection sinks and help enforce safer DOM operations. |
| Content Security Policy (CSP) | HTTP response policy restricting content sources and execution behavior. |
| Authentication / authorization | Establishing identity and deciding permitted actions. Trusted server endpoints must enforce authorization. |
| Route guard security limit | A guard improves client navigation but does not protect data or replace backend authorization. |
| Browser storage | localStorage, sessionStorage, cookies, or IndexedDB. Choose based on threat model and data sensitivity. |
| HttpOnly cookie | Cookie inaccessible to JavaScript. It still requires correct server and CSRF-aware configuration. |
| Secret | Credential that must remain private. Anything delivered to a browser bundle or public repository is not a secret. |
| Service worker update | New cached application version becoming available. Use a clear update flow and avoid mixing incompatible cached assets. |
| Browser global | Runtime API such as window, document, navigator, or localStorage that may not exist during SSR. |

## TypeScript and JavaScript words

Angular uses TypeScript, which builds on JavaScript and adds type syntax. These are language constructs, not Angular APIs.

| Word or syntax | Meaning |
| --- | --- |
| const / let | Block-scoped bindings; const prevents reassignment, while let permits it. A const object can still be mutated. |
| class | Declares a class, commonly used for Angular components, directives, pipes, and services. |
| interface | TypeScript-only shape contract describing object structure; erased from emitted JavaScript. |
| type | TypeScript alias or type expression; erased from emitted JavaScript. |
| extends | Class inheritance and TypeScript interface/type composition syntax. |
| implements | Compile-time check that a class satisfies an interface contract. |
| import / export | Module syntax for using and exposing values or types. |
| Decorator | TypeScript syntax such as @Component or @Injectable that attaches framework metadata or behavior. |
| Generic | Type parameter that preserves relationships between types, such as Observable<User>. |
| Union type | Type formed by alternatives, such as string or number. |
| Narrowing | Refining a broad type after checks such as typeof, discriminant comparison, or an in-operator. |
| Optional property | Property marked with ?, indicating that it may be absent. |
| readonly | TypeScript modifier preventing reassignment through a typed reference; it does not freeze runtime objects. |
| async / await | JavaScript syntax for working with Promises; it does not make a function run in the background by itself. |
| Promise | JavaScript object representing one future result or failure. An Observable can represent multiple values. |
| null / undefined | JavaScript values representing absence or missing data; handle deliberately. |
| Type assertion | TypeScript instruction to treat an expression as a type. It does not validate or convert runtime values. |
| as | TypeScript assertion or import/export alias syntax depending on its position. |
| satisfies | TypeScript operator that checks compatibility while preserving a specific inferred type. |
| enum | TypeScript enum construct. String unions or constant objects are alternatives used in many codebases. |
| Access modifier | TypeScript visibility syntax such as public, protected, and private. It does not replace runtime authorization. |
