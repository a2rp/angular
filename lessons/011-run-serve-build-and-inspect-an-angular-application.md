# Run, Serve, Build, and Inspect an Angular Application

Once an Angular workspace exists, the usual workflow is to start the development server, inspect the application in a browser, make a source change, and build the configured project. These commands use targets in angular.json, so knowing the difference between the development server and a build helps explain what the CLI is doing.

## Learning objectives

After this lesson, you should be able to:

- Run Angular CLI commands from the correct workspace.
- Start and stop the development server.
- Explain how ng serve relates to the serve and build targets.
- Create a build and locate its configured output.
- Compare development and production configurations.
- Inspect runtime behavior with browser developer tools.
- Diagnose common serve and build failures from the first relevant error.

## Confirm the project and configuration

Open a terminal in the Angular workspace root, the directory containing angular.json. Check the tool versions and available scripts:

~~~sh
ng version
npm run
~~~

Open angular.json and find the intended application's project entry. Locate its serve and build targets, default configurations, and outputPath. A multi-project workspace may have several applications, each with different settings. Use the project name explicitly when you do not intend to use the workspace's default project.

CLI commands read this configuration to determine which builder and options to use. The exact builder and option schema can change across Angular releases. The generated angular.json and documentation for the installed CLI version are the references for a particular workspace.

## Start the development server

From the workspace root, run:

~~~sh
ng serve
~~~

The CLI invokes the application's configured serve target. The development server compiles the app, serves it locally, watches source files, and rebuilds when they change. When the build succeeds, the terminal prints a local address. Keep this process running while you use the browser.

To ask the CLI to open the address:

~~~sh
ng serve --open
~~~

If the workspace contains multiple applications, specify the project name:

~~~sh
ng serve task-tracker
~~~

The exact syntax and options are available from:

~~~sh
ng serve --help
~~~

To stop the server, press Ctrl+C in the terminal where it is running.

## Development serve uses a build configuration

A serve target commonly points to a build target through an option such as buildTarget. It may select a development configuration by default. The development configuration can favor fast rebuilds and more useful diagnostic messages.

The names and details are defined by the workspace; inspect the serve target and its referenced build configuration instead of assuming every project is identical. If a workspace defines a development serve configuration, it can be selected explicitly:

~~~sh
ng serve --configuration development
~~~

The development server is for local development. It is not a production web server and does not deploy the application.

## Inspect the running application

Use the browser and terminal together:

- Read the terminal for compilation status, warnings, and errors.
- Use the browser console for runtime exceptions and rejected requests.
- Use the Elements panel to inspect rendered HTML and styles.
- Use the Network panel to inspect document, script, style, and API requests.
- Change a template or component value and confirm the browser reflects the rebuild.
- If navigation is configured, enter a route directly in the address bar and observe how the dev server responds.

A successful compile does not guarantee that the page behaves correctly. Runtime errors, missing assets, failed network requests, and routing problems require separate inspection.

## Create a build

From the workspace root, run:

~~~sh
ng build
~~~

The CLI invokes the selected project's build target. Angular's standard build configuration uses production by default, unless the workspace or command specifies another configuration. Production settings commonly enable output optimizations. The build command writes output to the configured path, commonly under dist.

You can request a named configuration:

~~~sh
ng build --configuration development
ng build --configuration production
~~~

Configuration names and options are workspace-specific. Inspect angular.json to confirm they exist. CLI command options can override configured values for an individual build.

The application builder can produce different output depending on rendering mode. A browser-only app typically has browser assets; a server-rendered or hybrid app can also have server output. Do not assume that every project's deployable files are located at the same nested path. Read the terminal summary and the configured outputPath.

To build a particular project:

~~~sh
ng build task-tracker
~~~

To inspect command options:

~~~sh
ng build --help
~~~

## Development output and production output

Development and production are named configurations, not separate source-code copies. They can override build and serve options such as optimization, source maps, file replacements, budgets, and output settings.

A typical distinction is:

- Development builds support local iteration and debugging.
- Production builds apply optimizations intended for deployed output.

These are defaults, not promises about every custom workspace. Inspect the named configuration in angular.json. A production build is a build artifact; it is not automatically deployed or hosted.

## How ng serve and ng build relate

The commands are connected but serve different purposes:

- ng serve runs a local development server and watches for source changes. It uses the configured serve target, which normally uses a build target.
- ng build runs the build target and writes output files. It normally does not start a development server.
- Both commands use project configuration and can accept command-line options that override configuration.
- A build can succeed while the app still has runtime or user-experience problems.
- A working development server does not prove the production configuration or deployment is correct.

## Common failures and first checks

### The CLI cannot find angular.json

Check the current directory. Run workspace commands from the workspace root or a directory within the workspace where the CLI can identify the configuration.

### A command selects the wrong application

Check the workspace's default project and provide a project name explicitly when needed. In a multi-project workspace, do not assume the default project is the one you meant to serve or build.

### The server reports a port conflict

Read the CLI message. Another process may be using the selected port. Use a supported port option or stop the conflicting process, then retry.

### The build reports a TypeScript or template error

Start with the first diagnostic and the source file and line it identifies. Check the relevant type or template contract. Avoid disabling strict compiler options just to hide a real mismatch.

### The build warns about a budget or dependency

Find the corresponding build configuration in angular.json and understand the warning before changing a threshold or dependency. A warning can identify bundle size or compatibility concerns rather than a broken command.

### The browser shows an error after compilation succeeds

Open developer tools and inspect the console and Network panel. Distinguish compile-time diagnostics from runtime exceptions, failed HTTP requests, missing assets, and incorrect routes.

### The build output is not where expected

Check the build target's outputPath and any server-rendering configuration. The output layout depends on the builder and rendering mode; use the CLI summary and workspace configuration.

## A repeatable local workflow

1. Open the workspace containing the application.
2. Inspect the project and target in angular.json.
3. Start ng serve and keep its terminal visible.
4. Confirm the page renders and check browser diagnostics.
5. Make one small source change and observe rebuild behavior.
6. Stop the dev server with Ctrl+C.
7. Run ng build and read the complete summary.
8. Inspect the configured output directory.
9. If deployment is intended, follow the hosting and rendering-specific deployment steps separately.

## Key takeaways

- ng serve invokes the configured serve target and runs a local development server.
- ng build invokes the build target and writes configured output.
- A serve target usually selects a build target and configuration.
- ng build commonly uses production by default; confirm the project's configuration.
- Multi-project workspaces may require explicit project names.
- Use browser developer tools to inspect runtime behavior after compilation.
- A successful build does not deploy the app or prove the complete user experience.

## Further reading

- [Angular CLI ng serve](https://angular.dev/cli/serve)
- [Angular CLI ng build](https://angular.dev/cli/build)
- [Angular workspace configuration](https://angular.dev/reference/configs/workspace-config)
- [Angular CLI builders](https://angular.dev/tools/cli/cli-builder)
- [Angular deployment](https://angular.dev/tools/cli/deployment)
- [Angular build environments](https://angular.dev/tools/cli/environments)

---

[← Lesson 010: Workspace Structure, Source Files, and Configuration](010-workspace-structure-source-files-and-configuration.md) | [Q&A 011 →](../qnas/011-run-serve-build-and-inspect-an-angular-application.md) | [Examples 011 →](../examples/011-run-serve-build-and-inspect-an-angular-application.md)
