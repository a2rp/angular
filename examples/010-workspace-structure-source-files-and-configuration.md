# Examples: Workspace Structure, Source Files, and Configuration

This walkthrough creates a disposable Angular workspace and traces how the workspace configuration, app source, component, and a static asset fit together. Use a practice directory separate from the angular notes repository.

## 1. Create a workspace to inspect

Create a standalone app with routing and CSS:

~~~sh
ng new workspace-tour --standalone --routing --style=css --package-manager=npm --skip-git
cd workspace-tour
~~~

If an option is not recognized, use ng new --help and adapt the command to the installed Angular CLI version. Generated names can differ across versions and file naming styles.

## 2. Map the workspace root

At the top level, find these files and directories if they are present:

~~~text
workspace-tour/
  angular.json
  package.json
  package-lock.json
  tsconfig.json
  .editorconfig
  .gitignore
  README.md
  public/
  src/
~~~

Make a note beside each file:

- angular.json: Angular CLI workspace and project configuration.
- package.json: package declarations and scripts.
- package-lock.json: npm's selected dependency resolution.
- tsconfig.json: shared TypeScript compiler configuration.
- .gitignore: files Git should not track.
- .editorconfig: editor formatting defaults.
- public/: static files copied or served according to the build configuration.
- src/: source files for the default application.

The exact root files depend on setup choices. Do not create or remove files only to make the tree match this illustration.

## 3. Inspect the Angular project entry

Open angular.json and find the projects object. Locate the workspace-tour project and identify:

1. projectType.
2. root and sourceRoot.
3. The build target and its builder.
4. The serve target and the build configuration it uses.
5. The configured styles and assets.
6. Any named development or production configurations.

The default app's sourceRoot is commonly src even though the project entry is nested under projects in the JSON. Read the configured paths; do not infer the folder layout from the project name.

The target schema and builder names depend on the CLI version. Treat the generated angular.json as the source for this workspace and consult the matching Angular docs before editing.

## 4. Follow the root application files

Under src, locate the browser document, entry point, and global stylesheet. Then open the app directory.

The app may look approximately like this:

~~~text
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

Your project may use longer names, may omit optional files, or may have extra files. Map the actual names in your editor.

Trace the startup path:

1. In index.html, find the root host element, commonly app-root.
2. In the root component file, find the selector that matches it.
3. In main.ts, find bootstrapApplication and the root component import.
4. If main.ts passes appConfig, open app.config.ts and find the registered providers.
5. If routing is enabled, open app.routes.ts and follow how the routes are provided.

This path connects the document loaded by the browser to the component Angular renders.

## 5. Trace TypeScript configuration inheritance

Open tsconfig.json and locate its compilerOptions and angularCompilerOptions. Then inspect tsconfig.app.json and tsconfig.spec.json if they exist.

For each child config, note:

- Which base configuration it extends.
- Which source files it includes or excludes.
- Which options it adds or overrides.
- Whether Angular template checking options appear under angularCompilerOptions.

Do not move options between compilerOptions and angularCompilerOptions. They are read by different parts of the toolchain.

## 6. Add and reference a static asset

Create **public/notes-mark.svg**:

~~~svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 160 48" role="img" aria-labelledby="title">
  <title id="title">Workspace tour</title>
  <rect width="160" height="48" rx="8" fill="#17324d"/>
  <circle cx="24" cy="24" r="12" fill="#79c0ff"/>
  <text x="46" y="30" fill="#ffffff" font-family="sans-serif" font-size="16">Angular</text>
</svg>
~~~

In the root component template, add:

~~~html
<img src="/notes-mark.svg" alt="Angular workspace tour">
~~~

The public directory is configured for static assets in current CLI workspaces. Verify the workspace's assets configuration if the image does not load. Do not add the SVG to angular.json unless the existing configuration requires it.

## 7. Find global styles and component styles

Open src/styles.css and identify rules that apply globally. Then inspect the stylesheet referenced by the root component metadata. A file that exists but is not referenced is not automatically part of the rendered app.

Add a simple global rule to src/styles.css:

~~~css
body {
  margin: 0;
  font-family: system-ui, sans-serif;
}
~~~

Add a component-specific rule to the stylesheet referenced by the root component:

~~~css
img {
  display: block;
  max-width: 10rem;
  height: auto;
}
~~~

Use the file path actually referenced by your component or global styles configuration.

## 8. Use a configuration ownership checklist

| Change | First file to inspect |
| --- | --- |
| Add or change an npm package | package.json and the selected lockfile |
| Change which source files compile | tsconfig.app.json or the relevant project config |
| Change a build target, style list, or asset list | angular.json |
| Change the root component's template dependencies | Root component metadata |
| Add a route | app.routes.ts, when present |
| Add an application-wide provider | app.config.ts, when present |
| Change global page styles | src/styles.css or the global styles list |
| Change component-only presentation | Component stylesheet |
| Add a static image or icon | public/ or the configured asset directory |

Before changing a value, confirm which target consumes it. If you are unsure, search the workspace for the setting or file name.

## 9. Compare a single-app and multi-project workspace

In a single-app workspace, the initial app's source commonly lives directly in src. A workspace created with no initial app can instead put generated applications and libraries under projects/<project-name>.

The angular.json file records each project's root and sourceRoot paths. Compare those values with folders on disk. Do not move directories manually to imitate a different workspace style; use Angular CLI generation commands and inspect the configuration they update.

## 10. Build and inspect

From the workspace root, run:

~~~sh
ng build
~~~

If the build completes, inspect the configured output directory, commonly dist, and locate the copied SVG. The build target and output layout depend on the generated configuration. A build does not deploy the app.

If the asset or stylesheet is missing, check its source path and the configured assets or styles list in angular.json.

## Try these variations

1. Open .gitignore and find which generated folders are excluded.
2. Change the project selector prefix in angular.json only after checking what generated code uses it.
3. Generate a second Angular project in the workspace and compare its sourceRoot to the initial app.
4. Turn off routing in a separate practice workspace and compare the optional files.
5. Find the default and named build configurations and note which one ng build uses.
6. Find a test file and identify which TypeScript configuration includes it.
7. Remove the SVG reference from the template while keeping the file; observe that a static file does not render until something references it.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="009-create-a-standalone-angular-application.md">009 - Create a Standalone Angular Application</a></td><td><a href="../lessons/010-workspace-structure-source-files-and-configuration.md">Lesson</a></td><td><a href="../qnas/010-workspace-structure-source-files-and-configuration.md">Q&amp;A</a></td><td><a href="011-run-serve-build-and-inspect-an-angular-application.md">011 - Run, Serve, Build, and Inspect an Angular Application</a></td>
</tr>
  </tbody>
</table>
