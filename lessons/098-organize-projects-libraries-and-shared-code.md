# Organize Projects, Libraries, and Shared Code

An Angular workspace can contain one application or several applications and libraries. Choosing a workspace layout affects shared configuration, dependencies, build commands, release boundaries, and how teams collaborate. Start with the simplest setup that fits the repository's real ownership and reuse needs.

## Learning outcomes

After this lesson, you should be able to:

- Explain the difference between an Angular workspace and an Angular project.
- Choose between separate repositories and a multi-project workspace.
- Decide when repeated code deserves a library.
- Design a library's public API and dependency direction.
- Use Angular CLI project generation rather than hand-editing workspace structure.
- Keep project configuration, source layout, and release strategy aligned.

## 1. Workspace, application, and library

A **workspace** is the repository-level Angular CLI environment. It has shared configuration such as angular.json, package.json, and TypeScript configuration.

A **project** is an application or library that the workspace knows how to build, test, or run. An application can run as a browser application. A library is built for use by an application or another package; it is not launched as an application by itself.

A small repository normally has one workspace and one application. More applications or libraries can be added when they have a reason to share tools, dependencies, configuration, or release coordination.

The angular.json projects map describes project configuration. It does not have to mirror every directory exactly. Generated multi-project workspaces commonly place additional projects under projects/.

## 2. Separate repositories or one multi-project workspace?

Use separate workspaces when applications have independent ownership, release schedules, dependency policies, or toolchains. This limits coordination and makes repository boundaries visible.

A multi-project workspace can fit when applications and libraries are developed together, changes often span projects, and the team wants shared tooling and dependency management. It can simplify coordinated changes, but also increases the scope of workspace configuration and repository-wide builds.

Choose based on actual collaboration and release needs, not a belief that every app should be in one monorepo. A monorepo requires clear ownership, dependency boundaries, and build practices; a workspace alone does not provide those policies.

## 3. When code should become a library

A library is a good candidate when a capability has a stable API and is intentionally reused by multiple applications or teams. Examples include a design system, domain client, or reusable Angular integration.

Before extracting a library, ask:

- Is the code actually reused, or is reuse only expected someday?
- Can its consumers agree on a stable public API?
- Who owns breaking changes and releases?
- Which Angular and third-party package versions does it support?
- Does it need to be published outside this workspace?
- Would a simple shared feature folder solve the problem with less machinery?

Sharing code in the same workspace and publishing a reusable package are related but different goals. A package intended for publication needs packaging, peer dependency, compatibility, and release decisions.

## 4. Create projects with the Angular CLI

The CLI updates workspace configuration and creates the expected project files. For example, a workspace can start without a root application and then add explicit applications and libraries:

~~~sh
ng new acme-workspace --no-create-application
cd acme-workspace
ng generate application admin
ng generate application storefront
ng generate library ui-kit
~~~

In an existing workspace, the CLI can create a library:

~~~sh
ng generate library data-access
~~~

Use the CLI options shown by the installed version's help when naming a project or selecting its root. Avoid copying commands from a different Angular release without checking the current CLI. The CLI's library generator creates workspace configuration and library files; do not manually add a TypeScript path mapping unless your setup explicitly calls for it.

See the [Angular workspace and file structure guide](https://angular.dev/reference/configs/file-structure), [workspace configuration reference](https://angular.dev/reference/configs/workspace-config), and [library generator reference](https://angular.dev/cli/generate/library).

## 5. Give each library a deliberate public API

A library should expose what consumers are allowed to depend on. Its public API file is the boundary; consumers should import from the package entry point instead of reaching into private source files.

~~~ts
// projects/ui-kit/src/public-api.ts
export * from './lib/button/button';
export * from './lib/card/card';
~~~

Keep implementation details private. If consumers import internal file paths, it becomes difficult to reorganize code without breaking them.

The public API should be small enough to understand. Exporting every helper may turn internal details into a long-term compatibility obligation. Document behavior, inputs, outputs, accessibility expectations, and supported Angular versions for public UI components.

## 6. Manage dependencies and direction

A healthy dependency direction might look like:

~~~text
Application -> feature libraries -> small shared libraries
~~~

Avoid cycles between libraries. A foundational UI library should not depend on a business workflow application. If two libraries need the same contract, place a small stable contract at an appropriate lower-level boundary or redesign ownership.

Dependencies should reflect runtime needs. Avoid using a library only to create a folder boundary if it adds a package build or dependency burden without reuse or release value.

For publishable libraries, declare framework packages as peer dependencies where appropriate so consumers supply compatible Angular packages. Follow Angular's current library guidance and package builder requirements rather than treating a workspace import as proof the package will work when published.

## 7. Project names and commands

Use project names that communicate purpose, such as admin, storefront, ui-kit, or data-access. The project name is used by CLI commands and appears in workspace configuration.

~~~sh
ng build admin
ng build ui-kit
ng test admin
~~~

Exact target names depend on generated project configuration and the installed Angular version. Inspect angular.json or use CLI help to confirm available targets. A library's build output and an application's deployment output have different purposes.

## 8. Shared source versus a library

Use a feature-local folder when code belongs to one application feature. Use an application shared folder when a small implementation is reused within that app. Use a workspace library when there is a meaningful project boundary, package boundary, or stable reuse contract.

Do not extract every shared function into a library. Each project adds API, build, dependency, testing, and versioning decisions. Prefer the lightest boundary that protects ownership and makes changes safe.

## 9. Plan the boundary before extraction

When extracting existing code:

1. Identify consumers and actual reuse.
2. Define which files are public and which remain implementation details.
3. Move dependencies in one direction and remove cycles.
4. Generate the project with the CLI and preserve workspace conventions.
5. Update consumers to import only through public entry points.
6. Decide how the project is built and tested in this workspace.
7. If publishing, define compatibility, packaging, and versioning rules separately.
8. Document how consumers adopt changes.

A library is complete when its consumers can understand its public contract and the owning team can evolve it safely, not merely when a folder appears under projects/.
