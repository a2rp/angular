# Q&A: Angular CLI Installation and Command Basics

## Installation and prerequisites

### 1. What is the Angular CLI?

Angular CLI is Angular's command-line tool for creating workspaces, generating code, serving and building applications, and managing other Angular tasks. It is published as the @angular/cli npm package and provides the ng executable.

### 2. What does the CLI do that Angular itself does not?

The CLI coordinates project scaffolding, builders, schematics, and command workflows. Angular framework packages provide the framework APIs used by the application. The CLI helps create and build the application; it is not itself the application runtime.

### 3. Why is Node.js needed for Angular development?

The CLI and its build tools run as JavaScript programs on Node.js outside the browser. Node.js also supplies APIs used by development tooling. That does not mean a static Angular browser application must run on a Node.js server in production.

### 4. How do I choose a Node.js version for Angular?

Check the official Angular compatibility table for the specific Angular version. It lists supported Node.js, TypeScript, and RxJS ranges. Requirements change, so avoid copying a Node.js version from an old tutorial.

### 5. What is the basic npm command to install the global Angular CLI?

Use:

~~~sh
npm install -g @angular/cli
~~~

The global install provides ng on the system path when the npm global binary directory is configured for the shell. The Angular setup guide also documents other package managers.

### 6. Do I have to install the CLI globally?

No. A global install is convenient for creating a workspace and using ng from the terminal. A project can declare its own local CLI dependency. Use the project's local CLI and scripts when maintaining an existing workspace.

### 7. Is the global CLI version automatically the same as the workspace's local CLI?

No. They are separate installations and can have different versions. The workspace manifest and lockfile record project dependencies. Run project commands through its package scripts where possible so its local binaries are used.

### 8. What should I check after installing the CLI?

Check the runtime and tool versions and inspect the available commands:

~~~sh
node --version
npm --version
ng version
ng help
~~~

If a command is unavailable, first check installation and the shell path. If the project already exists, check its local CLI and scripts.

### 9. Why might ng not be recognized in PowerShell?

The global CLI might not be installed, or npm's global executable directory may not be on the shell path. Windows script policy may also affect command shims. Read the full error and follow the official platform-specific setup directions; avoid changing a broad system setting without understanding why the command failed.

## Command context and workspace basics

### 10. What is the general Angular CLI command pattern?

Most commands follow the shape:

~~~text
ng command-name [arguments] [options]
~~~

The exact arguments and options differ by command and installed CLI version. Ask that command for help rather than relying on a flag from a different version.

### 11. Where should I run ng new?

Run it outside the workspace that you are creating. It creates a new directory and generates an Angular workspace with an initial application. The CLI may ask setup questions, and prompt choices can vary by version.

### 12. Where should I run ng generate?

Run it inside the Angular workspace where the new code belongs. The command uses the workspace configuration to select a project and generate files.

### 13. What does ng serve do?

It invokes the configured serve target, starts a local development server, and rebuilds as source files change. It is for development and does not deploy an application or replace a production build.

### 14. What does the --open option do with ng serve?

It asks the development server command to open the application in a browser. The exact supported options can be checked with ng serve --help.

### 15. What does ng build do?

It invokes the configured build target and writes build output, commonly under dist. The output and behavior depend on the workspace's target and configuration. A successful build does not publish or deploy the application.

### 16. What does ng generate do?

It runs an Angular schematic to create or update code. For example, ng generate component task-card requests a component schematic. Review the changed files because project settings and CLI versions can affect the generated output.

### 17. What are CLI aliases?

Some commands have short aliases, such as generate and g, serve and s, build and b, or version and v. Full command names are clearer while learning and in documentation. Verify aliases with the installed CLI help.

### 18. How do I find options for one command?

Ask for that command's help, for example:

~~~sh
ng serve --help
ng generate component --help
ng new --help
~~~

This shows the supported usage for the installed version. An option from another release may be renamed, removed, or behave differently.

### 19. What is the current working directory, and why does it matter?

It is the directory the terminal is currently operating in. Angular commands that act on a workspace use its configuration, commonly angular.json. If a command cannot find a project or workspace, check the current directory before reinstalling tools.

### 20. What is the difference between an Angular workspace and an application?

The workspace is the project container and configuration managed by Angular CLI. It can contain one or more Angular applications and libraries. An application is a runnable Angular project within that workspace.

## Project and maintenance commands

### 21. How do I run a development server from the workspace?

After changing into the workspace directory, run:

~~~sh
ng serve
~~~

Use the terminal output to find the local URL. Press Ctrl+C in the same terminal to stop the running development server.

### 22. What is the difference between ng serve and ng build?

ng serve runs a development server that rebuilds during local changes. ng build invokes the configured build target and writes output files. Neither command automatically deploys the result.

### 23. Does every Angular project support ng test or ng lint?

No. These commands depend on the targets and builders configured in the workspace. Inspect angular.json, package.json scripts, and command help before assuming a target exists.

### 24. What does ng add do?

It adds support for an external library or integration by using that package's Angular schematic when one is provided. It can install packages and edit workspace files. Review those changes and the package's documentation.

### 25. What does ng update do?

It checks or applies supported package updates for Angular workspaces, according to its options and the project's update path. Updates may modify dependency versions, lockfiles, and source files. Review the official update guide and generated changes before accepting them.

### 26. Is ng update the same as npm update?

No. ng update understands Angular packages and Angular-specific update workflows. npm update operates on npm dependency ranges and the package tree. Use the command that matches the change you intend, and follow Angular's migration instructions for framework updates.

### 27. What does ng config change?

It reads or writes Angular workspace configuration values in angular.json. Since that file controls project targets and settings, inspect the value before and after changing it.

### 28. How do npm scripts help use a local CLI?

A package script such as start or build can invoke a locally installed command. npm adds local executable directories to the script's path. Read package.json to see which scripts exist, then use the project's documented script rather than guessing which global executable will run.

### 29. How can I list project scripts?

From the package directory, run:

~~~sh
npm run
~~~

It lists scripts declared in that package.json. Script names and behavior are project-specific.

### 30. Should I upgrade the CLI whenever a tutorial uses a newer command?

No. First inspect the project's Angular and CLI versions, then check the installed command's help and the relevant Angular documentation. Updating a project changes dependencies and may require migrations.

## Troubleshooting

### 31. A CLI command says no workspace is available. What should I do?

Check whether the command is supposed to run inside a workspace and whether the current directory is within the one you intend to modify. ng new is an exception: it creates a new workspace from outside that workspace.

### 32. A tutorial's flag is unknown. Should I change package versions?

Not immediately. Run ng version and request command-specific help. The tutorial may target a different CLI release. Use a supported option for the installed version, or follow a matching migration path if an upgrade is genuinely required.

### 33. The command uses an unexpected CLI version. What should I inspect?

Compare the globally available CLI with the project's local @angular/cli dependency. Check the package manager and lockfile, and inspect package.json scripts. Prefer project scripts for build and serve workflows so they use the project's local tooling.

### 34. What should I do if ng serve works but deployment fails?

ng serve only verifies a local development workflow. Inspect the production build configuration, hosting requirements, output directory, and deployment process separately. Serving locally is not the same as publishing the app.

### 35. How should I safely practice CLI commands?

Use a disposable practice directory, confirm the current path, read help for unfamiliar options, and review files after generation or update commands. Do not run dependency updates or generators against an important project just to experiment.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="007-node-js-npm-package-managers-and-workspaces.md">007 - Node.js, npm, Package Managers, and Workspaces</a></td><td><a href="../lessons/008-angular-cli-installation-and-command-basics.md">Lesson</a></td><td><a href="../examples/008-angular-cli-installation-and-command-basics.md">Example</a></td><td><a href="009-create-a-standalone-angular-application.md">009 - Create a Standalone Angular Application</a></td>
</tr>
  </tbody>
</table>
