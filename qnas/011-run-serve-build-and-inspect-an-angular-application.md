# Q&A: Run, Serve, Build, and Inspect an Angular Application

## Workspace context and targets

### 1. From which directory should I run ng serve and ng build?

Run them from the Angular workspace where the intended project's angular.json is available. If the workspace contains several projects, specify the project when necessary.

### 2. What does ng serve do?

It runs the configured serve target, starts a local development server, compiles the application, and watches for source changes. The server is intended for development, not production hosting.

### 3. What does ng build do?

It invokes the configured build target and writes output files to the configured output path. It normally does not start a development server or deploy the result.

### 4. How are the serve and build targets related?

A serve target commonly invokes a development-server builder and points to a build target, often through buildTarget. This lets ng serve use build options and a named build configuration. Inspect angular.json to see the actual wiring for a project.

### 5. What is a builder?

A builder is a package tool that performs a configured task for Angular CLI. Build and serve targets identify builders and options in angular.json. Builder names and schemas depend on the Angular CLI version.

### 6. What is a target configuration?

A target has default options and may have named configurations such as development or production. A command can select one using the configuration option. A command-line option can also override a configured value for that invocation.

### 7. Does every project use exactly the same targets?

No. Targets depend on project type and generated or customized configuration. A workspace may not have test, lint, or other targets. Inspect angular.json and the installed CLI help.

## Development server

### 8. How do I start the development server?

From the workspace root, run:

~~~sh
ng serve
~~~

The CLI reports the local address after starting. Keep the process running while using the app.

### 9. What does the open option do?

It asks the development server command to open the local app in a browser. If the browser does not open, use the address printed in the terminal.

### 10. How do I stop ng serve?

Press Ctrl+C in the terminal where the server is running.

### 11. Does ng serve rebuild after source changes?

Normally yes. It watches files and rebuilds as you edit. The exact behavior and supported watch options depend on the configured builder.

### 12. Is ng serve suitable for production hosting?

No. It is a development server for local work. Use the configured build and a hosting or server deployment process that matches the app's rendering mode.

### 13. Why does ng serve report that the port is already in use?

Another process may be using that port. Read the CLI message, choose a supported alternate port or stop the other process, and retry.

### 14. What if ng serve uses the wrong app?

Check the default project in angular.json and pass the intended project name explicitly when the workspace contains several applications.

## Build configurations and output

### 15. Which configuration does ng build use by default?

Angular CLI's standard generated configuration commonly selects production for ng build. The actual default is controlled by the workspace target and can be customized. Inspect angular.json rather than assuming.

### 16. Which configuration does ng serve use by default?

Generated workspaces commonly configure the serve target to use the development build configuration. A customized project can behave differently. Follow the serve target and its default configuration.

### 17. How do I select a named build configuration?

Use the configuration option, for example:

~~~sh
ng build --configuration production
~~~

The name must exist in the project's angular.json. Check command help for supported option syntax.

### 18. What is different between development and production builds?

The named configurations can override options such as optimization, source maps, file replacements, and budgets. Production configuration commonly enables optimizations; development configuration commonly favors local iteration and debugging. Exact settings are workspace-specific.

### 19. Does ng build always output to dist/<project>?

No. That is a common default, but outputPath can be customized. Server-rendered or hybrid projects may have separate browser and server output. Read the terminal summary and outputPath configuration.

### 20. What does a successful build prove?

It proves that the configured build target completed without a build failure. It does not prove that every route, API, browser interaction, accessibility behavior, or deployment setup is correct.

### 21. Does ng build deploy the app?

No. It creates build artifacts. A separate deployment process must copy or publish the right output to the chosen hosting environment and configure routing or server rendering as needed.

### 22. How do I build a particular project in a multi-project workspace?

Pass the project name when supported, for example:

~~~sh
ng build task-tracker
~~~

Confirm the project name in angular.json. Do not rely on an unknown default when several apps or libraries exist.

### 23. Can a build command option override angular.json?

Yes. Supported command-line options can override configured values for that invocation. The exact option names and value types are defined by the installed CLI and builder help.

## Inspecting runtime behavior

### 24. What should I inspect if the browser displays a blank page?

Check the development server terminal, browser console, and Network panel. Confirm that the root component renders and look for runtime exceptions, missing assets, or failed requests.

### 25. What is the difference between a build error and a runtime error?

A build error prevents the configured compilation or bundling step from completing. A runtime error occurs when code executes in the browser or server. The app can compile successfully and still fail at runtime.

### 26. Which browser developer tools panels are useful?

The Console panel shows runtime errors and warnings. Elements shows the rendered DOM and applied styles. Network shows document, asset, script, and API requests and their responses.

### 27. Should a missing asset be fixed by changing the build output path?

Not automatically. Check where the asset is stored, how it is referenced, and which paths are configured in angular.json. Then check the request URL and browser response in the Network panel.

### 28. How do I investigate a TypeScript or template build error?

Start with the first diagnostic and the referenced file and line. Trace the type or template expression and fix the underlying mismatch. Avoid disabling strict checking just to make the error disappear.

### 29. How do I inspect the exact command options for this Angular CLI version?

Use command help:

~~~sh
ng serve --help
ng build --help
~~~

This shows options supported by the installed CLI. A tutorial may describe an older version.

### 30. Why can ng build succeed while ng serve fails?

The commands use different targets and can use different configurations. Check the serve target, its selected buildTarget, and the development configuration. A custom dev-server setting can fail even if the build target itself succeeds.

### 31. Why can ng serve work while the production build fails?

Serve commonly uses a development configuration, while ng build commonly selects production. Production-specific options, budgets, optimization, replacements, or rendering settings may expose errors that development did not.

### 32. What should I do when output is in an unexpected directory?

Inspect the selected project's build target and outputPath. Check whether the app uses server rendering or a custom builder, and read the build summary for the browser and server output locations.

### 33. What is ng run useful for?

It runs a named project target using a target string such as project:target or project:target:configuration. It is useful when invoking a configured target directly. Use the standard command when it already expresses the task clearly.

### 34. Where can I find the official behavior for build and serve?

Use the current Angular CLI command references and workspace configuration guide. These describe the builders, options, and named configurations supported by the installed Angular generation.

## Official references

- [Angular CLI ng serve](https://angular.dev/cli/serve)
- [Angular CLI ng build](https://angular.dev/cli/build)
- [Angular workspace configuration](https://angular.dev/reference/configs/workspace-config)
- [Angular CLI builders](https://angular.dev/tools/cli/cli-builder)
- [Angular deployment](https://angular.dev/tools/cli/deployment)
- [Angular build environments](https://angular.dev/tools/cli/environments)

---

[← Lesson 011: Run, Serve, Build, and Inspect an Angular Application](../lessons/011-run-serve-build-and-inspect-an-angular-application.md) | [Examples 011 →](../examples/011-run-serve-build-and-inspect-an-angular-application.md)
