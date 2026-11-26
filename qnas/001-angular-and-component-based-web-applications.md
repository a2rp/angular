# Q&A: Angular and Component-Based Web Applications

## Core concepts

### 1. What is Angular?

Angular is a TypeScript-based framework for building web applications. It provides a component model, template syntax, dependency injection, routing, forms, HTTP APIs, testing utilities, and CLI tooling. It uses the browser's web platform rather than replacing it.

### 2. Is Angular a programming language?

No. Angular is a framework. TypeScript and JavaScript are programming languages used in Angular projects. Angular's decorators and template syntax are framework constructs compiled and interpreted by Angular tooling.

### 3. What is the difference between a framework and a UI library?

A UI library generally provides reusable UI pieces that an application can choose to use. A framework provides broader application conventions and integrated APIs for structuring and running an application. Angular includes UI components and also supports routing, dependency injection, forms, HTTP, and build tooling.

### 4. Does Angular replace HTML, CSS, JavaScript, or the browser?

No. HTML describes content, CSS styles it, JavaScript runs application behavior, and the browser provides the DOM and web APIs. Angular adds a structured way to compose and update the application using those platform capabilities.

### 5. What is a component?

A component is a reusable Angular UI unit. It has a TypeScript class, metadata that describes its use, a template, and optionally component styles. Components can be nested and composed into an application.

### 6. What are the main parts of a component?

The **class** holds state and behavior. The **metadata**, commonly attached with @Component, tells Angular the selector, template, styles, imports, and other options. The **template** describes rendered HTML. Optional **styles** describe presentation.

### 7. What does the @Component decorator do?

It attaches Angular component metadata to a TypeScript class. Angular uses that metadata to compile the component and understand how it can be created and used in templates.

### 8. What is a component selector?

A selector is the pattern used to place a component in another template. If a component declares selector app-counter, a parent template can use a matching app-counter element, provided the parent can access that component.

### 9. What are the host element and the view?

The **host element** is the DOM element matching the component selector. The **view** is the DOM content produced by the component's template and rendered inside or associated with that host.

### 10. What is a component tree?

It is the hierarchy formed by components containing or rendering other components. For example, App can contain TaskList, which can contain TaskItem components. The tree helps describe UI composition and component ownership.

### 11. Is a component tree the same as a backend or database architecture?

No. It describes UI relationships. Backend services, database models, and data ownership are separate design concerns, even though a UI component may display data that came from them.

## Templates, rendering, and interaction

### 12. What is an Angular template?

A template is HTML enhanced with Angular syntax for displaying data, binding properties, listening for events, using pipes, declaring local references, and controlling which views appear. Angular compiles templates as part of the application.

### 13. How does a click update the screen in the counter example?

The (click) binding calls the component's increase method. The method updates the count signal. Angular knows the template reads that signal, so the rendered count is refreshed to reflect the new value.

### 14. What does declarative UI mean?

It means describing the UI that should appear for the current state rather than manually issuing DOM changes for every state update. The template declares the relationship between state and rendered content; Angular performs the framework-managed update.

### 15. Is Angular's template ordinary HTML?

It is HTML plus Angular-specific syntax. The browser does not interpret Angular bindings or control-flow blocks by itself. Angular compiles and processes the template before and while rendering the application.

### 16. What is the difference between application state and the DOM?

Application state is the data the application treats as authoritative. The DOM is the browser's rendered document structure. In a declarative Angular app, the template derives DOM output from state; avoid directly editing Angular-managed DOM to keep a separate copy in sync.

### 17. Why is the counter signal read as count()?

Angular signals are callable accessors. Calling count() reads the current value and lets Angular track where the value is used. The component changes it with operations such as set() or update().

### 18. Does Angular automatically choose what the application should do?

No. Angular connects templates, state, and events, but the developer defines application rules, state ownership, error behavior, data validation, and component responsibilities.

## Architecture and project setup

### 19. What are standalone components?

Standalone components can be used directly by other standalone components through their imports metadata. They do not need to be declared in an NgModule. Modern Angular CLI projects use the standalone approach by default; existing projects may still use NgModules.

### 20. Are NgModules removed?

No. NgModules remain supported and appear in existing Angular applications and libraries. They are not required to organize a new standalone application, but developers may need to understand them when maintaining older code.

### 21. What does the Angular CLI do?

The Angular CLI creates workspaces and applications, serves projects during development, builds deployable output, runs configured tests, and supports generators and migrations. Exact prompts and defaults can differ by CLI version.

### 22. What does ng new create?

It creates a workspace with an initial Angular application and project configuration. Depending on options and CLI defaults, it may also configure routing, styles, testing, and other features. Check the generated files for the selected version.

### 23. Why should Node.js, TypeScript, and RxJS compatibility be checked?

Angular releases support defined ranges of these tools. An arbitrary combination may not install or compile correctly. Use Angular's version compatibility documentation for the Angular release the project uses.

### 24. Should the notes repository be used as the generated practice app?

No. Keep the notes repository focused on documentation. Create a separate Angular workspace for running lesson examples so generated application files and dependencies do not mix with learning notes.

### 25. What is the relationship between Angular and its backend?

Angular usually runs in the browser and calls backend APIs for data and operations. The backend must validate requests and enforce trusted rules such as authentication and authorization. Angular can improve the user experience, but it cannot make client-side data or checks trustworthy.

## Angular versus AngularJS

### 26. What is AngularJS?

AngularJS usually means the original 1.x framework. Modern Angular is a substantially different framework with different architecture, tooling, and APIs.

### 27. Can an AngularJS tutorial be followed as a modern Angular tutorial?

Not directly. Concepts can overlap at a high level, but examples using AngularJS controllers, scopes, or $http do not describe modern Angular components, signals, or HttpClient. Check that documentation explicitly targets modern Angular.

### 28. Is upgrading AngularJS to Angular the same as updating an Angular package?

No. Migrating from AngularJS to modern Angular is an application migration between different frameworks and often requires architectural changes. Updating a modern Angular project means moving between supported Angular releases using their update guidance.

## Design decisions and understanding

### 29. Why split an application into components?

Components give UI regions clear responsibilities and boundaries. Small, focused components are easier to understand, reuse, test, and change. Do not split every line into a component; choose boundaries that represent meaningful UI or behavior.

### 30. Should all application state live in the root component?

No. Keep state with the component or service that owns it. If multiple components need the same state, move it to their nearest shared owner or a suitable service. A single global owner for every value can create unnecessary coupling.

### 31. Does a TypeScript interface validate a server response?

No. TypeScript types help during development and are removed from emitted JavaScript. An interface annotation does not verify that network JSON has the expected shape. Validate external data at runtime when the application needs that guarantee.

### 32. Does a route guard protect a backend resource?

No. A guard can control browser navigation, but users can still call an API outside the UI. The server must check authentication and authorization for protected operations.

### 33. Is Angular appropriate for every website?

No. A static content page may not need a client application framework. Angular is useful when the application benefits from its structure and APIs. Consider the complexity, team, performance, accessibility, and deployment needs before choosing.

### 34. Does using Angular guarantee accessible UI?

No. Angular supports accessible development, but developers must choose semantic elements, labels, keyboard behavior, focus handling, and meaningful error messages. Accessibility needs to be built into and checked as part of the product.

## Review scenario

### 35. A component displays a value and a button. Clicking changes the class property, but the template does not update. What should you inspect?

Check that the template reads the same state that the handler updates, and confirm the correct event binding is connected to the method. In modern Angular, a signal read in a template is a straightforward way to make that dependency explicit. Also check for console or compilation errors.

### 36. A page has one component containing navigation, a form, a list, HTTP calls, and all validation logic. What is a reasonable next step?

Identify responsibilities and meaningful UI boundaries. Extract focused child components for reusable or distinct UI, and move shared or API behavior into a service where appropriate. Preserve one clear source of truth instead of duplicating state.

### 37. A team finds an online tutorial that uses $http and $scope. What should they verify?

Check whether the tutorial targets AngularJS 1.x. If the application uses modern Angular, find documentation for components, HttpClient, and modern Angular APIs instead.

## Quick review

- Angular is a framework; TypeScript and JavaScript are languages.
- A component has a class, metadata, and template, plus optional styles.
- Components compose into a tree that describes the UI.
- Templates derive rendered output from component state.
- Events call component behavior; state changes drive the rendered update.
- Standalone components are the normal starting point for modern Angular; NgModules remain relevant to existing code.
- AngularJS and modern Angular are different frameworks.
- Client-side code improves the experience but does not replace server validation or authorization.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td>Start of course</td><td><a href="../lessons/001-angular-and-component-based-web-applications.md">Lesson</a></td><td><a href="../examples/001-angular-and-component-based-web-applications.md">Example</a></td><td><a href="002-angular-compared-with-angularjs.md">002 - Angular Compared with AngularJS</a></td>
</tr>
  </tbody>
</table>
