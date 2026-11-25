# Examples: Run, Serve, Build, and Inspect an Angular Application

This walkthrough runs an existing Angular workspace, observes a development rebuild, checks browser diagnostics, and compares configured build outputs. Use the practice app from the previous chapter or another Angular workspace.

## 1. Open the correct workspace

In a terminal, change to the Angular workspace root, the directory containing angular.json. Then inspect the project names and versions:

~~~sh
ng version
npm run
~~~

Open angular.json and locate the application's project entry. Write down the project name, serve target, build target, default configurations, and outputPath. If the workspace contains multiple projects, use the intended project name in the commands below.

## 2. Start the development server

Run the application and ask the CLI to open a browser:

~~~sh
ng serve --open
~~~

If the CLI does not open the browser, use the local address printed in the terminal. Keep the terminal open and note:

- Whether the initial compilation succeeds.
- Which configuration the serve target selects.
- Which local URL the server reports.
- Whether any warnings or errors appear.

Stop the server at the end of the exercise with Ctrl+C.

## 3. Inspect the rendered app

In the browser:

1. Open Developer Tools.
2. Look at Console for runtime errors and warnings.
3. Look at Elements to find the Angular root host element and rendered page structure.
4. Look at Network to inspect the document, generated scripts and styles, assets, and any API requests.
5. If routing is configured, navigate to a route and note the address.

The terminal reports compilation problems. Browser tools report what happens after code reaches the browser. Keep those diagnostic sources distinct.

## 4. Observe a development rebuild

With ng serve still running, open the root component template and make a small visible text change. Save the file.

Observe the terminal for a rebuild and the browser for an updated page. Revert the text change or keep it as a deliberate app edit. If the browser does not update, check the terminal first, then reload and inspect the Console.

## 5. Build using the configured default

Stop the development server, then run:

~~~sh
ng build
~~~

Read the full build summary. Note the selected project, active configuration, warnings, and output path. A production configuration is commonly used by default, but confirm this in the project's angular.json.

If the workspace has several projects, name the intended one:

~~~sh
ng build task-tracker
~~~

Replace task-tracker with the actual project name.

## 6. Compare named configurations

If the project's build target defines development and production configurations, build each explicitly:

~~~sh
ng build --configuration development
ng build --configuration production
~~~

Compare the CLI summaries and generated output. Look for configuration changes such as optimization, source maps, file replacements, or budgets. Do not infer that every project has identical settings.

If the configurations have the same output path, a later build may replace files written by an earlier build. Compare each summary as you run it, and use separate output paths only if the workspace is configured to do so.

## 7. Inspect the build output

Find the outputPath in angular.json and inspect that directory. For a common dist output, PowerShell can list generated files with:

~~~powershell
Get-ChildItem .dist -Recurse -File
~~~

The output may instead use a customized path or have browser and server subdirectories for server rendering. Follow the build summary and workspace configuration.

Do not open an output index.html directly with a file URL as a substitute for hosting. Angular routing and asset paths expect the build to be served in the appropriate web context. Deployment instructions depend on whether the app is browser-only, static-rendered, or server-rendered.

## 8. Diagnose an intentional problem

In the practice app, temporarily change a template binding to reference a property that does not exist. Save the file and observe the diagnostic in the development server terminal. Restore the correct binding and confirm the app rebuilds.

Next, create a harmless runtime issue only in a disposable practice app, such as calling a missing method from a click handler. Open the page, trigger the action, and inspect the browser Console. Restore the handler afterward.

This demonstrates the difference:

- A template or TypeScript error appears during compilation.
- A runtime error occurs while the compiled app is executing.
- A network error appears in the browser Network panel and may not be a compiler problem.

## 9. Compare serve and build behavior

Complete this table from the output you observed:

| Question | ng serve | ng build |
| --- | --- | --- |
| Starts a development server? |  |  |
| Watches and rebuilds source changes? |  |  |
| Writes configured build artifacts? |  |  |
| Uses a target in angular.json? |  |  |
| Automatically deploys the app? |  |  |

Expected idea: serve is the local development workflow; build creates configured artifacts. Neither command automatically deploys the application.

## 10. Troubleshooting checklist

- If angular.json is not found, check the current directory.
- If the wrong project runs, specify the intended project.
- If a port is in use, follow the CLI message or choose a supported alternate port.
- If a build fails, read the first diagnostic and inspect the referenced file.
- If compilation succeeds but the screen fails, inspect Console and Network.
- If the output path is unexpected, read the selected project's outputPath and build summary.
- If a named configuration is unknown, check that it exists in angular.json.

## Try these variations

1. Run ng serve without --open and enter the printed address manually.
2. Compare serving the default project with serving a named project.
3. Find the serve target's buildTarget and determine which build configuration it selects.
4. Change the outputPath only in a disposable practice project, then confirm the next build follows it.
5. Inspect whether the production configuration uses budgets and what the build reports.
6. If your project uses server rendering, identify browser and server outputs from the build summary.

## References

- [Lesson 011: Run, Serve, Build, and Inspect an Angular Application](../lessons/011-run-serve-build-and-inspect-an-angular-application.md)
- [Q&A 011](../qnas/011-run-serve-build-and-inspect-an-angular-application.md)
- [Angular CLI ng serve](https://angular.dev/cli/serve)
- [Angular CLI ng build](https://angular.dev/cli/build)
- [Angular workspace configuration](https://angular.dev/reference/configs/workspace-config)
- [Angular CLI builders](https://angular.dev/tools/cli/cli-builder)
- [Angular deployment](https://angular.dev/tools/cli/deployment)

---

[← Lesson 011](../lessons/011-run-serve-build-and-inspect-an-angular-application.md) | [← Q&A 011](../qnas/011-run-serve-build-and-inspect-an-angular-application.md)
