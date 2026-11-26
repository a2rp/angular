# Organize Projects, Libraries, and Shared Code — Examples

This walkthrough creates a multi-project Angular workspace with two applications and one small reusable library. It demonstrates workspace setup and a deliberate public API. It is an example layout; use a single application workspace if your project does not need multiple projects.

## 1. Create the workspace and projects

Use the installed Angular CLI. The workspace generator may ask about styling or other project choices.

~~~sh
ng new shop-workspace --no-create-application
cd shop-workspace

ng generate application storefront
ng generate application admin
ng generate library ui-kit
~~~

The workspace contains shared configuration and a projects directory. The exact generated files can vary by Angular CLI version.

~~~text
shop-workspace/
  angular.json
  package.json
  tsconfig.json
  projects/
    storefront/
    admin/
    ui-kit/
~~~

The CLI registers each project in the workspace configuration. Use the project names from your own angular.json when running commands.

## 2. Add a focused component to the library

Generate a component in the library:

~~~sh
ng generate component button --project ui-kit
~~~

Open the generated component source and give it a narrow presentation role:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'ui-button',
  template: '<button type="button"><ng-content /></button>'
})
export class UiButton {}
~~~

Use the generated file name and any required standalone imports from your installed Angular CLI version. This component intentionally leaves behavior to its consumer; a production design-system button would also define accessibility, disabled state, styles, variants, and documented inputs.

## 3. Export only the supported API

The generated library includes a public API file. Export the component from that entry point:

~~~ts
// projects/ui-kit/src/public-api.ts
export * from './lib/button/button';
~~~

The generated path may use a different file suffix. Point the export to the actual generated source file. Consumers should import UiButton from the library entry point and should not import from projects/ui-kit/src/lib/... directly.

The public API is a compatibility boundary. Do not export every implementation helper merely because it exists.

## 4. Consume the library from an application

Import the public symbol in an application component:

~~~ts
import { Component } from '@angular/core';
import { UiButton } from 'ui-kit';

@Component({
  selector: 'app-root',
  imports: [UiButton],
  template: '<h1>Storefront</h1><ui-button>Continue</ui-button>'
})
export class App {}
~~~

The CLI configures the workspace library relationship. Do not add an unrelated manual path alias if the generated workspace already provides the needed mapping. If the import cannot be resolved, inspect the generated library package name and workspace configuration.

## 5. Build the projects by name

~~~sh
ng build ui-kit
ng build storefront
ng build admin
~~~

An application build produces an application output intended for deployment. A library build produces a library package artifact for workspace use or later publishing. The exact output paths and targets are defined by generated configuration.

## 6. Decide what should be a library

Consider a library if the UI component is intentionally used by storefront and admin, has a stable contract, and has an owner who can review changes. Keep a component inside one app if it only serves that app and does not need an independent boundary.

Before publishing ui-kit outside this workspace, review its generated package configuration, peer dependencies, supported Angular versions, package format, and release process. A workspace build alone does not define how external consumers receive compatible versions.

## 7. Review the dependency graph

A sensible direction is:

~~~text
storefront ----\
                -> ui-kit
admin --------/
~~~

The applications depend on the UI library. The library should not import application pages or business workflows. If both apps need business rules, create a separate, deliberately named domain library only when its API and ownership are clear.

## 8. Verify the project layout

Use the CLI to inspect projects and targets:

~~~sh
ng config projects
ng build ui-kit
ng build storefront
~~~

For a real workspace, also run the team's normal tests and review that application imports use public entry points. Do not include generated build artifacts in version control unless the repository's release process explicitly requires them.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="097-feature-based-application-architecture.md">097 - Feature-Based Application Architecture</a></td><td><a href="../lessons/098-organize-projects-libraries-and-shared-code.md">Lesson</a></td><td><a href="../qnas/098-organize-projects-libraries-and-shared-code.md">Q&amp;A</a></td><td><a href="099-reusable-ui-components-and-design-system-boundaries.md">099 - Reusable UI Components and Design-System Boundaries</a></td>
</tr>
  </tbody>
</table>
