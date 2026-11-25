# Create a Standalone Angular Application

The Angular CLI can create a working Angular application from a few terminal prompts. A modern standalone application starts from a root component and uses Angular's standalone bootstrap API rather than requiring a root NgModule. This lesson creates that app, follows how the browser reaches its root component, and verifies the development and build workflows.

## Learning objectives

After this lesson, you should be able to:

- Create a new Angular application in its own workspace.
- Choose a standalone application and understand the basic setup prompts.
- Describe how the browser document, main entry point, root component, and template connect.
- Explain the role of bootstrapApplication and application configuration.
- Start the development server and produce a build.
- Recognize that CLI prompts and generated filenames can vary by version and options.

## Before creating the app

You need a terminal, Node.js version supported by the Angular version you are creating, a package manager, and Angular CLI. Check the official Angular setup and compatibility documentation before choosing or changing a Node.js version.

Choose a practice location outside this learning-notes repository. The Angular CLI initializes Git for a new project by default, so keeping the practice app in its own directory also keeps its Git history separate.

## Create the workspace and application

From a terminal in the parent directory where you want the practice project, run:

~~~sh
ng new task-tracker --standalone --routing --style=css --package-manager=npm --skip-git
~~~

This asks the CLI to create a workspace and initial standalone application named task-tracker, configure routing, use CSS files, install dependencies with npm, and skip initializing a separate Git repository. CLI prompts and options can change by version; check ng new --help if an option is not recognized.

The CLI creates a task-tracker directory. Move into it:

~~~sh
cd task-tracker
~~~

Check the generated project and tool versions:

~~~sh
ng version
npm run
~~~

Review the generated package.json and lockfile. The CLI version and Angular packages in the project are recorded there. Generated file names and setup choices can vary. Current Angular CLI projects commonly use concise names such as app.ts and app.html; other projects may use names such as app.component.ts and app.component.html.

## Start the application

From the workspace root, run:

~~~sh
ng serve --open
~~~

Angular CLI compiles the application and starts a development server. The terminal reports the local address, often localhost:4200, and the browser opens when supported. Keep the terminal running while using the page. Stop the server with Ctrl+C.

The development server rebuilds when you change project source files. It is a local development workflow, not a production deployment.

## The browser-to-component path

A minimal standalone application has a path like this:

1. The browser loads the project's index.html document.
2. The document contains the root element matching the root component's selector, commonly app-root.
3. The browser loads the JavaScript entry point configured by the build, commonly src/main.ts.
4. main.ts calls Angular's bootstrapApplication with the root component.
5. Angular creates that component and renders its template into the matching host element.

A simplified document might contain:

~~~html
<body>
  <app-root></app-root>
</body>
~~~

A simplified root component might look like this:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class App {
  title = 'task-tracker';
}
~~~

Its template could render the component's state:

~~~html
<main>
  <h1>{{ title }}</h1>
  <p>Your standalone Angular application is running.</p>
</main>
~~~

A current Angular component is standalone by default. In Angular versions before standalone became the default, the component metadata needed to mark a standalone component explicitly. When following older project examples, check the Angular version and the component's standalone setting.

## How bootstrapApplication starts the root

The application entry point commonly imports the root component and bootstrap API:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { App } from './app/app';
import { appConfig } from './app/app.config';

bootstrapApplication(App, appConfig).catch((error) => console.error(error));
~~~

The exact import path depends on the generated filenames. Some minimal projects pass only the root component; others also pass an application configuration.

bootstrapApplication renders a standalone component as the application's root. It does not require a root NgModule. It returns a promise that resolves after the application has bootstrapped, which is why generated entry points may attach a catch handler for startup errors.

## Application configuration

When present, app.config.ts exports configuration that is passed during bootstrap. It can register application-wide providers, such as router or HTTP providers.

A small illustrative config is:

~~~ts
import { ApplicationConfig } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [],
};
~~~

The CLI may generate additional providers based on the chosen options. For example, enabling routing can configure router providers and route definitions. Keep generated configuration that the project uses; do not replace the file with this minimal illustration.

Standalone components declare template dependencies in their component metadata. A component that uses another component, directive, or pipe imports that dependency in its imports array. Application-wide providers belong in application configuration or another suitable provider scope, rather than in a component's template imports.

## Build the application

From the workspace root, run:

~~~sh
ng build
~~~

The CLI runs the configured build target and writes output, commonly under dist. The output is produced for a configured build; it is not uploaded or deployed automatically.

You can inspect available scripts and targets in package.json and angular.json. A generated project may also define test or other commands. Use the project's actual configuration rather than assuming every workspace has the same set.

## Standalone does not mean a single component

Standalone describes how Angular components, directives, and pipes make their dependencies available. It does not mean an application must contain only one component, cannot use services, or cannot use routing. A standalone root application can still contain many components, services, routes, and libraries.

Standalone APIs are the recommended direction for new Angular development. NgModule-based applications remain supported, and existing code can use compatibility APIs. The key distinction is that a standalone application can bootstrap its root component directly with bootstrapApplication instead of bootstrapping a root NgModule.

## Common creation issues

### The CLI rejects an option

Check the installed CLI with ng version and inspect ng new --help. Options and defaults can differ among Angular CLI versions. Remove or adjust only the option that the installed version does not support.

### Node.js is incompatible

Check the Angular version being installed and compare it with the official version compatibility table. Choose a supported Node.js version and retry in a new terminal.

### The browser shows a blank page

Check the terminal and browser console for the first error. Verify that the root selector in the component matches the root element in index.html, and that main.ts bootstraps the intended component.

### The browser cannot connect to the development server

Confirm that ng serve is still running and use the exact address printed by the terminal. Another process may already occupy the configured port; read the CLI message and follow its prompt or options.

### The generated files differ from a tutorial

Angular CLI changes defaults and file naming conventions over time. Use the current project files and the installed CLI help. The next lesson explains the generated workspace structure and configuration.

## Key takeaways

- ng new creates an Angular workspace and its initial application.
- Modern CLI-generated applications use the standalone API by default.
- main.ts starts the application by calling bootstrapApplication with the root component.
- The component selector connects Angular's root component to a host element in index.html.
- Application configuration can provide router and other application-wide services.
- ng serve is for local development; ng build creates configured build output.
- CLI prompts, generated filenames, and defaults depend on the installed version.

## Further reading

- [Angular local environment setup](https://angular.dev/tools/cli/setup-local)
- [Angular CLI ng new command](https://angular.dev/cli/new)
- [Angular bootstrapApplication API](https://angular.dev/api/platform-browser/bootstrapApplication)
- [Angular component guide](https://angular.dev/guide/components)
- [Angular Component API](https://angular.dev/api/core/Component)
- [Angular version compatibility](https://angular.dev/reference/versions)

---

[← Lesson 008: Angular CLI Installation and Command Basics](008-angular-cli-installation-and-command-basics.md) | [Q&A 009 →](../qnas/009-create-a-standalone-angular-application.md) | [Examples 009 →](../examples/009-create-a-standalone-angular-application.md)
