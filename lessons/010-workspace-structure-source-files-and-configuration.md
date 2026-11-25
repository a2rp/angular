# Workspace Structure, Source Files, and Configuration

An Angular workspace is more than the application source folder. It includes workspace-wide configuration, package metadata, TypeScript settings, application entry points, assets, and one or more Angular projects. This lesson maps the important files and explains which tool or part of the app uses each one.

Generated names and some files vary by Angular CLI version and the choices made with ng new. Treat the structure in your own workspace as authoritative, then use this guide to understand its roles.

## Learning objectives

After this lesson, you should be able to:

- Distinguish a workspace from an application project and its source directory.
- Locate Angular application entry points, root component files, and static assets.
- Explain the purposes of angular.json, package.json, lockfiles, and tsconfig files.
- Understand how Angular CLI project entries relate to files on disk.
- Find build, serve, and other configured targets.
- Recognize optional files generated for routing, tests, or server rendering.
- Make configuration changes in the file that owns the relevant setting.

## Workspace, project, and source tree

A workspace is the directory managed by Angular CLI. It contains shared configuration and one or more projects. A project is an Angular application or library with its own source files and build configuration.

A basic single-application workspace commonly resembles this:

~~~text
task-tracker/
  angular.json
  package.json
  package-lock.json
  tsconfig.json
  .editorconfig
  .gitignore
  README.md
  public/
  src/
    index.html
    main.ts
    styles.css
    app/
      app.ts
      app.html
      app.css
      app.config.ts
      app.routes.ts
      app.spec.ts
~~~

This is an orientation map, not a promise that every generated project has exactly these files. For example, routing and tests are optional choices, SSR adds files, and the CLI's file naming style may use app.component.ts instead of app.ts.

In the default single-app layout, application source is under the workspace's src directory. A multi-project workspace usually keeps additional applications and libraries under projects/<project-name>. Each project has a source directory of its own.

## Workspace-level files

### angular.json

This is the Angular CLI workspace configuration. It describes the projects in the workspace and the defaults used by Angular CLI commands. Project entries contain information such as project type, root paths, selector prefix, and CLI targets for tasks like building and serving.

The initial app created by ng new may live directly under src, even though its named project appears under the projects property in angular.json. In a multi-project workspace, additional application and library source trees normally appear under projects on disk. The JSON project map and directory tree describe related things, but their structures are not identical.

Targets specify how the CLI performs tasks. A build target identifies a builder and its options and configurations; a serve target can invoke a development server builder and refer to a build target. The exact builder package and option names can change between CLI releases. Inspect the generated configuration and the matching Angular CLI documentation before editing it.

Configuration values can exist at workspace, project, and named-configuration levels. More specific project settings can override workspace defaults, and command-line options can override configuration for one invocation. Configuration keys in the JSON use camelCase even when a command-line option uses dash-case.

### package.json and the lockfile

package.json declares the workspace's npm packages and scripts. Angular packages and tools are installed according to this manifest. package-lock.json records the selected dependency versions for npm.

Use the package manager and lockfile already chosen by the workspace. Add or update packages through that package manager, review manifest and lockfile changes together, and do not edit the lockfile by hand.

### tsconfig.json and child configurations

The root tsconfig.json contains TypeScript options shared by projects in the workspace. Application and test configurations can extend it and add settings specific to their files.

Common files include:

- tsconfig.app.json for application compilation.
- tsconfig.spec.json for unit test compilation.
- tsconfig.lib.json and tsconfig.spec.json for a generated library.
- Server-specific TypeScript configuration when SSR is enabled.

Angular-specific compiler settings can live in angularCompilerOptions. TypeScript compiler settings belong under compilerOptions. The TypeScript and Angular compiler configurations are related but have different options and consumers.

### .gitignore and editor configuration

.gitignore lists generated or local files Git should not track, such as node_modules and build caches. Keep it consistent with the project tools; do not commit installed dependencies just because they exist on disk.

.editorconfig gives supported editors shared formatting defaults such as indentation and line endings. It does not replace formatter or lint configuration.

### README.md

The generated application README describes that workspace. Update it with project-specific setup and commands if the app will be maintained by others. The learning-notes repository README serves a different purpose: it documents the course and links its chapters.

## Application source files

### src/index.html

This is the browser document served for the application. It contains the root host element whose tag matches the root component selector. The Angular build tool adds the generated scripts and styles; you generally do not manually insert build output paths into this HTML file.

### src/main.ts

This is the application entry point. A standalone app commonly imports the root component and calls bootstrapApplication. The entry point may also import an app configuration and handle startup errors.

The CLI's build configuration identifies this entry point. If the application fails before rendering, inspect the terminal and browser console and check that the entry point bootstraps the intended root component.

### src/app/

This directory holds application logic and UI files, commonly components, templates, and component styles. Current CLI naming may use concise filenames such as app.ts and app.html; other naming styles use app.component.ts and app.component.html.

A standalone component declares dependencies needed by its own template. Organize additional components and features according to the project conventions. Do not assume that every component must live directly beside the root component as the application grows.

### app.config.ts and app.routes.ts

app.config.ts, when generated, exports the application's provider configuration passed to bootstrapApplication. It can register routing, HTTP support, or other application-wide providers. Preserve generated providers required by the app.

app.routes.ts, when routing is enabled, exports route definitions. The router provider is generally included by the application configuration. Both files may be named differently or combined depending on CLI version and project options.

### src/styles.css

This is a global stylesheet included by the application build. It is suitable for page-wide rules, reset styles, and shared design tokens. Component styles are usually scoped to the component's view by Angular's default view encapsulation.

The workspace build configuration decides which global stylesheets are included. A stylesheet existing on disk does not automatically mean it is included in the application.

### public/ and assets

The public directory is intended for static files such as images and icons that should be served or copied as files. The workspace build configuration determines which asset paths are included. Some existing or customized workspaces instead use src/assets or another configured asset directory.

Use the file location and asset rules defined by the workspace. For a public file at public/logo.svg, a browser reference is commonly /logo.svg, while a source file placed under a configured assets directory may use a different URL. Check the configured assets entry instead of assuming every Angular version uses the same layout.

### Tests and optional server-rendering files

Test files commonly use a .spec.ts suffix. The test configuration and runner depend on the project setup. Files for server rendering, prerendering, or server-specific providers appear only when those features are configured. Their exact names vary by Angular version.

Do not delete a file simply because a minimal tutorial does not show it. First check whether a build, test, route, or server target references it.

## Read angular.json without editing it

When inspecting a workspace, find the named project under projects and identify:

- Its projectType: application or library.
- Its root and sourceRoot paths.
- Its build target and builder.
- Its serve target and which build configuration it uses.
- Its test target, if one is configured.
- The styles, assets, and entry points supplied to the builder.
- Named configurations such as development and production.

The target schema and builders are version-sensitive. Treat angular.json as tool configuration, not as a file to copy wholesale from a different Angular release.

## Choose the right configuration file

| If you need to change... | Inspect... |
| --- | --- |
| Angular project, build, serve, test, styles, or assets settings | angular.json |
| Installed package versions and package scripts | package.json and the selected lockfile |
| Shared TypeScript checks | tsconfig.json |
| App-specific TypeScript checks | tsconfig.app.json |
| Angular template compiler options | angularCompilerOptions in the relevant tsconfig |
| Root component metadata and template dependencies | The root component file |
| Application-wide providers | app.config.ts, when used |
| Route definitions | app.routes.ts, when used |
| Global CSS | src/styles.css or the path configured for global styles |
| Static image or icon | public/ or the asset path configured in angular.json |

Before changing a setting, search for which file defines it and check which target consumes that configuration.

## Common structure mistakes

### Editing the wrong angular.json

A monorepo can contain more than one workspace. Confirm the current directory and edit the angular.json at the root of the workspace you intend to change.

### Treating projects in angular.json as folders with the same layout

The first application can have source at the workspace root's src directory while its named project is still under the projects map. Read root and sourceRoot values to see the actual paths.

### Assuming a file is active because it exists

Stylesheets, assets, or TypeScript files may need to be referenced by configuration or imported by code. Trace how the build uses them.

### Replacing generated configuration with a tutorial snippet

A generated file may contain providers, build options, or version-specific settings required by the workspace. Make a small change and preserve unrelated generated configuration.

### Confusing public assets with source code

Static files and application source have different roles. Put source code under the application's source tree, and put static assets in the directory configured for static assets.

## Key takeaways

- A workspace can contain one or more Angular application and library projects.
- angular.json describes workspace projects and CLI targets; it does not map one-to-one to folders.
- package.json and its lockfile describe package dependencies and scripts.
- tsconfig files inherit TypeScript settings; Angular compiler options have their own section.
- main.ts starts the app, index.html hosts the root selector, and src/app contains application code.
- Global styles, component styles, and static assets are configured and served differently.
- File names and optional files vary with Angular CLI version and project choices.
- Inspect the active configuration before making changes.

## Further reading

- [Angular workspace and project file structure](https://angular.dev/reference/configs/file-structure)
- [Angular workspace configuration](https://angular.dev/reference/configs/workspace-config)
- [Angular CLI ng new command](https://angular.dev/cli/new)
- [Angular CLI ng config command](https://angular.dev/cli/config)
- [Angular application npm packages](https://angular.dev/reference/configs/npm-packages)
- [Angular Component API](https://angular.dev/api/core/Component)

---

[← Lesson 009: Create a Standalone Angular Application](009-create-a-standalone-angular-application.md) | [Q&A 010 →](../qnas/010-workspace-structure-source-files-and-configuration.md) | [Examples 010 →](../examples/010-workspace-structure-source-files-and-configuration.md)
