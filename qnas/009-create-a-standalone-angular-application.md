# Q&A: Create a Standalone Angular Application

## Creating the project

### 1. What does ng new create?

It creates an Angular workspace and an initial application inside a new project directory. The workspace contains configuration, package metadata, and the source for the initial app. You can later add more applications or libraries to a workspace.

### 2. Where should I run ng new?

Run it from the parent directory where you want the new project folder to be created, not from inside the Angular workspace you are creating. The command generates the named directory.

### 3. What does the standalone option mean in ng new?

It asks the CLI to create an application based on Angular's standalone APIs instead of requiring an NgModule-based root bootstrap. Current CLI versions create standalone apps by default, but the explicit option can make the intention clear. Check the installed CLI help if the option differs.

### 4. What does routing do in the new command?

The routing option asks the CLI to include the initial routing setup and associated configuration. You can add routing later, but selecting it during creation gives the generated app a route configuration to start from.

### 5. What does the style option control?

It selects the stylesheet format for generated component styles, such as CSS, SCSS, or another supported choice. Check ng new --help for the formats supported by the installed CLI.

### 6. Why might I use skip-git?

Angular CLI initializes Git for a new project by default. skip-git tells it not to create that nested repository. This can be useful when generating a practice project inside a directory that is already a Git repository. Keeping the practice app in a separate directory is still the clearest setup.

### 7. Why pass a package manager to ng new?

It tells the CLI which package manager to use when installing the new project's dependencies. The project should use one manager and its corresponding lockfile consistently. If omitted, the CLI uses a default or asks according to the installed version and environment.

### 8. Are CLI prompts and defaults stable?

No. Prompts, options, and generated files can change between Angular CLI versions. Use the installed command's help and review the generated project rather than assuming a tutorial's answers always match.

### 9. Does ng new create production hosting?

No. It creates a project and installs dependencies. Serving, building, and deployment are distinct tasks. The generated app still needs an appropriate build and hosting workflow.

## Standalone applications and components

### 10. What is a standalone Angular application?

It is an Angular app whose root component is bootstrapped directly with the standalone APIs, rather than through a root NgModule. It can still use components, directives, pipes, services, routes, libraries, and application-level providers.

### 11. What does standalone mean for a component?

A standalone component makes its template dependencies available through its own component metadata, including an imports array. Current Angular components are standalone by default. Before Angular 19, the default was not standalone, so older code may explicitly use standalone: true or standalone: false.

### 12. Does standalone mean the application has only one component?

No. It describes how components declare dependencies and how the application root is bootstrapped. A standalone app can be split into many reusable components and services.

### 13. Is standalone the same as server-side rendering?

No. Standalone concerns Angular's component and bootstrap APIs. Server-side rendering determines where and how the application is initially rendered. A standalone Angular application can be configured for browser rendering, server rendering, or prerendering.

### 14. Are NgModules removed from Angular?

No. NgModule-based applications remain supported, and Angular provides compatibility APIs. Standalone APIs are the recommended approach for new development, while existing NgModule code can continue to work or be migrated gradually.

## How the application starts

### 15. What is the role of index.html?

It is the browser document that hosts the Angular application. It usually contains the root element whose tag matches the root component's selector, such as app-root.

### 16. What is the role of main.ts?

It is commonly the JavaScript or TypeScript entry point configured by the Angular build. It imports the root component and starts the Angular application, usually by calling bootstrapApplication.

### 17. What does bootstrapApplication do?

It bootstraps an Angular app and renders a standalone root component. It can also receive an application configuration that supplies providers. The API returns a promise that resolves after bootstrapping.

### 18. Does bootstrapApplication require an ApplicationConfig?

No. The root component is required; the configuration argument is optional. An app may pass an ApplicationConfig to register application-wide providers such as routing or HTTP support.

### 19. What connects the root component to the HTML document?

The component's selector must match the root host element in index.html. Angular bootstraps the component, locates the host element, and renders its template there. If the names do not match, the root component may not render as expected.

### 20. What does the root component contribute?

It is the top-level UI component for the application. Its metadata identifies its selector and template or template URL, and its class can hold the initial state and behavior. Other components can be composed beneath it.

### 21. What is app.config.ts for?

When generated, it exports application configuration passed to bootstrapApplication. It commonly registers root-level providers, such as router configuration. It is optional in minimal setups; keep the generated configuration that your app uses.

### 22. What is the difference between a component imports array and application providers?

A component imports array makes standalone components, directives, and pipes available to that component's template. Application providers register injectable dependencies in an application injector. They solve different dependency problems.

### 23. Does the root template need CommonModule for Angular control flow?

Built-in control-flow syntax such as @if and @for does not need CommonModule. Other directives and pipes have their own import requirements. Check which template features are used and import the corresponding dependencies.

## Running and building

### 24. What does ng serve do after creating the app?

It invokes the configured development-server target, compiles the project, and serves it locally. With the open option, it also asks the CLI to open a browser. It watches changes while the command remains running.

### 25. How do I stop the development server?

Press Ctrl+C in the terminal where ng serve is running. Keep that terminal open while the development server is active.

### 26. What does ng build verify?

It runs the configured build target and produces output files. This shows that the configured compilation and build completed; it does not deploy the result or prove that every user workflow behaves correctly.

### 27. Why can the generated files look different from a tutorial?

Angular CLI has changed file naming and project defaults over time. Current projects may use concise names such as app.ts and app.html, while older styles use app.component.ts and app.component.html. Follow the actual paths in the project.

### 28. Why is a project-local Angular CLI important?

The local CLI version is declared by the project's dependencies and lockfile. It aligns project commands with the toolchain expected by that workspace. A globally installed CLI can be a different version, so use the project's scripts for its serve and build tasks.

## Troubleshooting and version differences

### 29. What if ng new says an option is unknown?

Run ng version and ng new --help. The installed CLI may differ from the documentation or tutorial you are following. Use an option supported by that CLI or omit it and answer the prompt if available.

### 30. What if index.html has a root element but the app is blank?

Inspect the terminal and browser console. Check that the root selector matches the host element, that main.ts bootstraps the intended component, and that startup or template errors are not preventing rendering.

### 31. How do I handle older Angular projects that bootstrap an NgModule?

Do not assume they are broken. NgModule-based applications remain supported. Learn their existing bootstrap path before converting it. If migration is desired, use Angular's documented standalone migration in stages and verify the app between steps.

### 32. Why should I not copy the illustrative app.config.ts over the generated one?

The generated configuration may already register providers required by routing, HTTP, or other chosen features. The lesson's empty providers array only illustrates the type and placement. Preserve the project's actual configuration.

### 33. Where can I check the current standalone defaults?

Use the current Angular CLI ng new reference and Angular component guide. Defaults differ by version; the CLI reference lists the standalone option and its default for that release.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="008-angular-cli-installation-and-command-basics.md">008 - Angular CLI Installation and Command Basics</a></td><td><a href="../lessons/009-create-a-standalone-angular-application.md">Lesson</a></td><td><a href="../examples/009-create-a-standalone-angular-application.md">Example</a></td><td><a href="010-workspace-structure-source-files-and-configuration.md">010 - Workspace Structure, Source Files, and Configuration</a></td>
</tr>
  </tbody>
</table>
