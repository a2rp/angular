# Angular CLI Installation and Command Basics

The Angular CLI is the command-line tool for creating and maintaining Angular applications. It scaffolds projects, generates source files, invokes builders, serves applications during development, and helps update dependencies. This lesson covers installation and the everyday command patterns; the next lesson creates a standalone application and inspects its structure.

## Learning objectives

After this lesson, you should be able to:

- Explain what the Angular CLI does and what it does not do.
- Check Node.js and npm before installing Angular tooling.
- Install and verify the CLI.
- Distinguish global CLI commands from commands that operate on a project.
- Recognize the purpose of common Angular CLI commands.
- Use command help and options rather than guessing flags.
- Understand why ng serve is for development and ng build creates build output.
- Troubleshoot common command and compatibility errors.

## What the Angular CLI does

Angular CLI is a command-line interface published as the @angular/cli npm package. Its executable is named ng. It can create Angular workspaces, generate code, invoke build and serve targets, and manage Angular-related tasks from a terminal.

The CLI is a development tool. It does not replace TypeScript, the browser, or Angular itself. It coordinates tools and project configuration so common tasks use a consistent workflow.

Most Angular CLI commands have this general shape:

~~~text
ng command-name [arguments] [options]
~~~

The exact arguments and options depend on the command and CLI version. Use command help for the installed version instead of relying on remembered flags.

## Check prerequisites first

Angular CLI requires Node.js. Its package manager, such as npm, installs and runs JavaScript tooling. Angular also supports particular ranges of Node.js, TypeScript, and RxJS for each Angular version.

Check the current runtime:

~~~sh
node --version
npm --version
~~~

Before creating a project or upgrading dependencies, compare the installed Node.js version with the official Angular version compatibility table. The compatible version range changes as Angular and Node.js release new versions. Use a supported LTS or maintenance LTS Node.js line as directed by Angular's setup guide.

If an existing project already declares Angular and CLI dependencies, inspect its package.json and lockfile before installing a global tool. Existing projects should use their recorded dependency workflow.

## Install the Angular CLI

The Angular setup guide documents installation with npm, pnpm, Yarn, and Bun. For npm, the global installation command is:

~~~sh
npm install -g @angular/cli
~~~

A global CLI makes the ng command available from the terminal and is convenient for creating a new workspace. The installation location must be on the shell's executable path. On some Windows systems, PowerShell script policy or terminal configuration can affect npm command shims. Read the current Angular and npm instructions before changing a machine-level setting.

Verify that the command is available:

~~~sh
ng version
ng help
~~~

The version command reports Angular CLI version information. The help command lists available commands. Exact command availability can vary by installed version and workspace context.

A globally installed CLI and a project's local CLI are separate installations. The global CLI is useful when creating a project. Once inside an existing workspace, use that project's local CLI and package scripts where available. The local version is recorded in the project's dependency manifest and lockfile, helping the team use the same toolchain.

## Know where to run each command

The current directory matters:

- Run ng new outside an Angular workspace when creating a new workspace. It creates a new project directory.
- Run commands such as ng generate, ng serve, and ng build from inside a workspace because they use that workspace's configuration.
- Some commands, such as ng version and ng help, can be used for inspection without editing project files.

If a command reports that it cannot find angular.json or that no project is available, check the current directory before changing dependencies. You may be one directory too high or too deep.

## Create a workspace

The basic command is:

~~~sh
ng new task-tracker
~~~

The CLI creates a new directory named task-tracker, asks setup questions according to the installed version, and installs the project's dependencies using the selected package manager. Prompts and defaults may change between releases. Review the choices and the generated files instead of assuming an old tutorial's defaults.

After generation, move into the project directory:

~~~sh
cd task-tracker
~~~

The new Angular workspace includes configuration and a starter application. The CLI's generated package.json and lockfile describe the project's dependencies. Later lessons examine the workspace structure and application configuration in detail.

## Common commands inside a workspace

### Inspect versions and help

~~~sh
ng version
ng help
ng generate --help
~~~

Use help to see the supported arguments and options for the installed CLI. A command's short alias may also be available; for example, generate is commonly shortened to g, serve to s, build to b, and version to v. Prefer the full command when learning or writing documentation because it is easier to read.

### Generate project code

~~~sh
ng generate component task-card
~~~

The generate command applies an Angular schematic to create or update project files. A schematic is a rule-driven code generator. The exact file names and test files depend on project configuration and CLI version.

Use generate from inside the intended workspace. Before accepting changes, review which files were created or modified. Generated code is a starting point that you still need to understand, adapt, and maintain.

### Serve the application during development

~~~sh
ng serve
~~~

The serve command invokes the workspace's serve target, starts a development server, watches source files, and rebuilds as files change. The CLI commonly serves at localhost on a development port. The exact host and port can be changed through supported options or project configuration.

You can ask it to open the browser:

~~~sh
ng serve --open
~~~

Stop the process with Ctrl+C in the terminal. The development server is for local development; it does not publish the application or replace a production build and hosting setup.

### Build the application

~~~sh
ng build
~~~

The build command invokes the configured build target and writes output to the project's output directory, commonly under dist. The target can have named configurations such as development or production. Check the workspace configuration and CLI help when you need a specific target or option.

A successful build means the configured build completed. It does not deploy the application, configure a web server, or by itself prove that the user experience is correct.

### Run other configured targets

A workspace may define test, lint, or other targets:

~~~sh
ng test
ng lint
~~~

These commands work only when the corresponding target and builder are configured. Some generated projects do not include every target. Inspect the workspace configuration and project scripts before assuming a command exists.

### Add and update packages

The CLI also includes commands for integrating libraries and updating Angular packages:

~~~sh
ng add package-name
ng update
~~~

ng add runs a package's installation schematic when available, which may install dependencies and edit workspace files. ng update reports or applies supported package updates according to its options and update path. These commands can change manifests, lockfiles, and source configuration; review their proposed changes and follow the official update guide before applying updates to an important project.

## Options and command help

Options customize a command. For example, the serve command supports an option to open a browser, and generate commands accept options that affect generated code.

Ask the installed CLI for command-specific options:

~~~sh
ng serve --help
ng generate component --help
ng new --help
~~~

Use the spelling and values shown by the installed command help. Options can change over time. A flag copied from a different Angular version may be removed, renamed, or behave differently.

Arguments typically identify the project or item the command operates on. Options modify behavior. Read the command syntax carefully, especially when a command accepts a project name, schematic name, or target.

## Common errors and how to investigate them

### ng is not recognized or not found

The global CLI may not be installed, or its executable directory may not be on the current shell's path. Check the install result, reopen the terminal if needed, and follow the current platform-specific setup instructions. If you are already inside a project, check its local CLI dependency and scripts before installing a global package.

### The CLI rejects the Node.js version

Compare the installed Node.js version with Angular's compatibility table for the installed Angular version. Use a compatible Node version manager or installer according to your environment, then open a new terminal and verify node --version again.

### A command says it is outside a workspace

Check the current directory. Commands that create a workspace, such as ng new, are run outside the workspace being created. Commands that modify or run an existing project generally need to be launched from a directory within that Angular workspace.

### A command or option is unknown

Check ng version, then ask for help for the command. The project's local CLI may differ from a globally installed CLI. Follow the project's package scripts and documentation rather than upgrading dependencies just to make a tutorial command work.

### Installation fails on Windows

Read the full error and identify whether it mentions network access, permissions, shell execution policy, or an incompatible Node version. Do not make broad security or system changes to work around an error until you know which requirement failed. Follow the official Angular and Node.js setup guidance for the specific message.

## Safe command habits

- Confirm the current directory before running a command that changes files.
- Use a project-specific practice directory for exercises.
- Read command help when an option is unfamiliar.
- Keep package manager use consistent with the project's lockfile.
- Review generated files and dependency changes.
- Do not use ng update as a routine substitute for reading the migration path.
- Treat ng serve as a development server, not a production deployment.

## Key takeaways

- Angular CLI is the ng command-line tool for scaffolding, building, serving, generating, and maintaining Angular projects.
- Check Node.js compatibility before installing or upgrading Angular.
- A global CLI helps create projects; use the local CLI and scripts for an existing workspace.
- ng new creates a workspace and should be run outside the workspace being created.
- ng generate, ng serve, and ng build operate on an existing workspace.
- ng serve is a development server; ng build creates configured output.
- Command help describes the options supported by the installed CLI.
- The CLI can change project files, so review generated code and dependency updates.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="007-node-js-npm-package-managers-and-workspaces.md">007 - Node.js, npm, Package Managers, and Workspaces</a></td><td><a href="../qnas/008-angular-cli-installation-and-command-basics.md">Q&amp;A</a></td><td><a href="../examples/008-angular-cli-installation-and-command-basics.md">Example</a></td><td><a href="009-create-a-standalone-angular-application.md">009 - Create a Standalone Angular Application</a></td>
</tr>
  </tbody>
</table>
