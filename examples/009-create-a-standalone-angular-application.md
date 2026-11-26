# Examples: Create a Standalone Angular Application

This guided exercise creates a standalone Angular task-tracker app, identifies how Angular starts, changes its root screen, then serves and builds it. Use a scratch directory separate from the angular notes repository.

## 1. Confirm the CLI environment

Before creating the app, check Node.js, npm, and the Angular CLI:

~~~sh
node --version
npm --version
ng version
~~~

Compare the Node.js version with Angular's compatibility table. If ng is not available, follow the previous lesson's CLI setup steps.

## 2. Generate a standalone app

From the parent directory where you want the practice project, run:

~~~sh
ng new task-tracker --standalone --routing --style=css --package-manager=npm --skip-git
~~~

This creates a new workspace and app, selects standalone APIs, enables routing, uses CSS, selects npm, and skips a nested Git repository. If the installed CLI rejects any option, check:

~~~sh
ng new --help
~~~

CLI prompts, options, and defaults depend on the installed version. Review the choices before continuing.

Enter the new project directory and inspect the available scripts:

~~~sh
cd task-tracker
ng version
npm run
~~~

## 3. Locate the root component and bootstrap files

Inspect the generated source directory. Depending on the CLI version and file naming style, the root component may be named app.ts or app.component.ts. Its template may be app.html or app.component.html. Use the paths that actually exist in your workspace.

Find the root component's selector. A simplified standalone root component can look like this:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  imports: [],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class App {
  title = 'Task tracker';
}
~~~

Replace the illustrative file paths with the generated paths in your app. In current Angular, components are standalone by default; older Angular versions may require standalone: true explicitly.

The external template can be:

~~~html
<main>
  <h1>{{ title }}</h1>
  <p>Your standalone Angular application is running.</p>
</main>
~~~

The browser document, usually src/index.html, needs the matching host element:

~~~html
<body>
  <app-root></app-root>
</body>
~~~

Do not replace the full generated index.html; verify that it already contains the matching root element and only adjust the relevant markup if needed.

## 4. Follow the application entry point

Open src/main.ts. A simplified generated entry point is:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { App } from './app/app';
import { appConfig } from './app/app.config';

bootstrapApplication(App, appConfig).catch((error) => console.error(error));
~~~

Your generated imports may use different file names or may omit appConfig. Follow the actual generated entry point rather than copying these paths blindly.

The flow is:

1. The browser loads index.html.
2. The build loads the JavaScript generated from main.ts.
3. main.ts calls bootstrapApplication with the root component.
4. Angular matches the component selector to the app-root host element.
5. Angular renders the component template.

## 5. Understand application configuration

Because this practice app uses the routing option, inspect the generated app.config.ts and app.routes.ts. A simplified router configuration looks like:

~~~ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)],
};
~~~

The generated configuration may include additional providers depending on CLI version and setup choices. Keep those settings if the project uses them. The configuration passed to bootstrapApplication is where application-level providers can be registered.

## 6. Run the application

From the workspace root, run:

~~~sh
ng serve --open
~~~

Wait for the terminal to report a successful build and the local address. Check that the browser displays the title from the root component. Keep the terminal running while editing files; the development server should rebuild changes. Press Ctrl+C to stop it.

If the project defines an npm start script, you can inspect and use it as an alternative:

~~~sh
npm run
npm start
~~~

Only use npm start if it appears in the project's scripts.

## 7. Create a build

Stop the dev server if it is still running, then run:

~~~sh
ng build
~~~

The CLI writes configured build output, commonly under dist. This creates deployable assets but does not upload or host them. The output directory and configuration can be inspected in the generated workspace settings.

## 8. Make a small change and trace it

1. Change the title property in the root component.
2. Confirm the template displays the updated title.
3. Find the root host element in index.html.
4. Find the selector in the root component metadata.
5. Find the bootstrapApplication call in main.ts.
6. Find the application providers in app.config.ts.
7. Check the terminal and browser console if the page stops rendering.

This traces the path from static HTML through Angular bootstrap to rendered template.

## 9. Troubleshoot common issues

- If the app is blank, compare the component selector with the root element and read the first browser or terminal error.
- If the browser cannot connect, confirm ng serve is still running and use its printed URL.
- If the CLI rejects an option, inspect ng new --help and use options supported by that version.
- If a file path does not exist, use the generated naming style in your project rather than assuming an older tutorial's names.
- If the app config contains providers you do not recognize, inspect the generated choices before removing them.

## Try these variations

1. Create a second practice app with routing disabled and compare the generated files.
2. Create another practice app with SCSS and inspect the stylesheet references.
3. Remove the appConfig argument from a disposable minimal app's bootstrap call, then identify which configured providers would no longer be registered. Restore the generated call afterward.
4. Change the root selector and matching host element together, then serve the app again.
5. Inspect how the generated CLI version and Angular package versions are recorded in package.json and the lockfile.
6. Compare ng serve with ng build: observe which one starts a local server and which one writes build output.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="008-angular-cli-installation-and-command-basics.md">008 - Angular CLI Installation and Command Basics</a></td><td><a href="../lessons/009-create-a-standalone-angular-application.md">Lesson</a></td><td><a href="../qnas/009-create-a-standalone-angular-application.md">Q&amp;A</a></td><td><a href="010-workspace-structure-source-files-and-configuration.md">010 - Workspace Structure, Source Files, and Configuration</a></td>
</tr>
  </tbody>
</table>
