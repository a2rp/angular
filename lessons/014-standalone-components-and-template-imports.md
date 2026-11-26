# 14. Standalone components and template imports

A standalone component manages the dependencies used by its own template. When its template uses another component, directive, or pipe, the standalone component lists that dependency in its @Component imports array. This creates a clear, local description of what the template can use.

This chapter focuses on standalone components and template scope. The following lessons cover selector matching and template syntax in more detail.

## Learning goals

After this lesson, you should be able to:

- Explain what standalone means and how the default changed in Angular 19.
- Add standalone components, directives, and pipes to a component's template scope.
- Distinguish TypeScript module imports from @Component imports.
- Explain why one component's template imports do not automatically become another component's imports.
- Use an NgModule-based library from a standalone component when needed.
- Recognize which application setup belongs in bootstrap configuration instead.

## 1. What standalone means

A standalone component does not need to be declared in an NgModule. It can be imported directly by another standalone component and can list its own template dependencies.

In current Angular, components, directives, and pipes are standalone by default. This default changed in Angular 19. Older projects may have components that require NgModule declarations, or may explicitly use standalone: false.

A simple standalone component looks like this:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-welcome-panel',
  standalone: true,
  template: '<h2>Welcome</h2>',
})
export class WelcomePanel {}
~~~

The explicit standalone: true setting is optional in current Angular, but can make examples clear. Keep compatibility with the Angular version and conventions used by the workspace.

## 2. A standalone component's template scope

A template can use a component, directive, or pipe only when that dependency is in the template's compilation scope. For a standalone component, declare that scope in the imports property of its @Component metadata.

~~~ts
import { Component } from '@angular/core';
import { WelcomePanel } from './welcome-panel';

@Component({
  selector: 'app-dashboard',
  standalone: true,
  imports: [WelcomePanel],
  template: '<h1>Dashboard</h1><app-welcome-panel />',
})
export class Dashboard {}
~~~

WelcomePanel appears in the Dashboard template, so Dashboard imports it. Importing the TypeScript symbol at the top of the file makes the symbol available to the source file; listing it in the component metadata makes it available to that component's template.

These are related but different steps:

1. A TypeScript import loads a symbol into the TypeScript file.
2. The @Component imports array adds a standalone template dependency to that component's template scope.

Having only one of these imports does not do both jobs.

## 3. Import directives and pipes

Standalone directives and pipes can also be imported directly. For example, a template using the date pipe can import DatePipe from Angular's common package:

~~~ts
import { DatePipe } from '@angular/common';
import { Component } from '@angular/core';

@Component({
  selector: 'app-event-summary',
  standalone: true,
  imports: [DatePipe],
  template: '<p>Starts: {{ startsAt | date: "mediumDate" }}</p>',
})
export class EventSummary {
  readonly startsAt = new Date(2026, 0, 15);
}
~~~

Only the component that uses the pipe needs to import it. If a page component imports EventSummary, the page does not also need to import DatePipe unless its own template uses the pipe.

Angular's built-in template control-flow syntax, such as @if and @for, does not require importing NgIf or NgFor. Existing projects may still use structural directives such as *ngIf and *ngFor; those dependencies must be included in the relevant template scope, often through the direct directive imports or CommonModule.

## 4. Imports belong to each template

A child component's imports are not a shared list for the whole application. They describe the child component's own template scope.

For example:

- Dashboard imports WelcomePanel because Dashboard's template uses it.
- WelcomePanel imports DatePipe if WelcomePanel's template uses the date pipe.
- Dashboard does not gain DatePipe just because WelcomePanel imports it.

This local scope helps reveal dependencies while reading a component. It also means that if a template reports an unknown component, directive, or pipe, check that specific component's imports array first.

## 5. Import an NgModule when maintaining older code

A standalone component may import an NgModule. The component can then use the components, directives, and pipes that the module exports.

~~~ts
import { CommonModule } from '@angular/common';
import { Component } from '@angular/core';

@Component({
  selector: 'app-legacy-view',
  standalone: true,
  imports: [CommonModule],
  template: '<p *ngIf="visible">Visible</p>',
})
export class LegacyView {
  readonly visible = true;
}
~~~

This is useful when a library or existing project still exposes template dependencies through an NgModule. Prefer direct standalone imports or modern built-in syntax for new code when appropriate. Do not import a broad module automatically if the component needs only one direct dependency and the project supports that style.

An NgModule-based library can also provide dependencies through its exported module. Check its documentation to see what it exports and whether importing it has other setup requirements.

## 6. Standalone components in an NgModule application

Standalone components can be imported by an NgModule. They are not added to that module's declarations array. Conversely, a non-standalone component must be declared in exactly one NgModule and is generally made available through the module's exports.

Older components may be marked with standalone: false. Those components cannot be imported directly into a standalone component's imports array; import the NgModule that declares and exports them instead, or migrate the component if that is part of the project plan.

New applications should generally use standalone components. NgModules remain relevant when working with existing applications and libraries.

## 7. Template imports are not application providers

The @Component imports array is for template dependencies: components, directives, pipes, and compatible NgModules.

It is not the place to configure application-wide services or startup features. Those providers belong in the application configuration passed to bootstrapApplication, as shown in [Bootstrap, ApplicationConfig, and application providers](012-bootstrap-applicationconfig-and-application-providers.md).

For example:

- Import RouterOutlet in a component if its template contains router-outlet.
- Configure routes with provideRouter(routes) in application providers.
- Import DatePipe in a component if its template uses the date pipe.
- Configure application services through providers or providedIn as appropriate.

## 8. Keep imports focused

Each component should list what its own template uses. Prefer importing a specific standalone dependency when it is available and that makes the template scope clear. Import a module when the module is the supported integration boundary or when several of its exports are needed.

Avoid adding unrelated dependencies to a shared parent component just in case. A smaller scope makes missing imports easier to diagnose and reduces accidental coupling.

## Common mistakes

- Importing a symbol in TypeScript but forgetting to add it to @Component imports.
- Adding a dependency to the root component even though a child template is the one that uses it.
- Expecting a child's imports to become available automatically in its parent template.
- Putting services or provideRouter results in the template imports array.
- Declaring a standalone component in an NgModule instead of importing it.
- Importing a non-standalone component directly rather than importing its declaring NgModule.
- Treating CommonModule as mandatory in every standalone component.
- Adding NgIf or NgFor imports for templates that use built-in @if or @for control flow.
- Assuming the same standalone default applies to projects created before Angular 19.

## Practical checklist

- [ ] Identify the component whose template uses the dependency.
- [ ] Add a TypeScript import for the dependency's symbol.
- [ ] Add that symbol to the same component's @Component imports array.
- [ ] Check whether the dependency is standalone or provided through an NgModule.
- [ ] Keep application providers in application configuration, not template imports.
- [ ] Follow the installed Angular version and the project's existing conventions.

## Next lesson

Continue with [Selectors and the component host element](015-selectors-and-the-component-host-element.md).

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="013-components-metadata-and-component-boundaries.md">013 - 13. Components, metadata, and component boundaries</a></td><td><a href="../qnas/014-standalone-components-and-template-imports.md">Q&amp;A</a></td><td><a href="../examples/014-standalone-components-and-template-imports.md">Example</a></td><td><a href="015-selectors-and-the-component-host-element.md">015 - 15. Selectors and the component host element</a></td>
</tr>
  </tbody>
</table>
