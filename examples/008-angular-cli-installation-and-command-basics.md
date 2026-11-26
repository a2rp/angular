# Examples: Angular CLI Installation and Command Basics

This hands-on walkthrough installs Angular CLI, creates a disposable workspace, and uses common commands. Run the commands in a terminal on your own computer. Keep the practice app separate from this learning-notes repository so generated project files do not get mixed into your notes.

## 1. Check Node.js and npm

Open a terminal and run:

~~~sh
node --version
npm --version
~~~

Check Angular's version compatibility table before choosing a Node.js version. If you already have an Angular project, first inspect its Angular and CLI versions and its package-manager lockfile.

## 2. Install and verify Angular CLI

For the npm workflow, install the CLI globally:

~~~sh
npm install -g @angular/cli
~~~

Then verify the command:

~~~sh
ng version
ng help
~~~

The setup guide also documents pnpm, Yarn, and Bun. Use a single package manager consistently for a given project. If the command is not found, read the full install output and check whether the npm global executable directory is available to the current terminal.

## 3. Create a separate practice workspace

Change to a scratch directory that is not inside the angular notes repository. Create a new practice app:

~~~sh
ng new cli-practice
~~~

Answer the prompts for the practice project. Prompt choices and defaults can change with CLI releases. If you are unsure about a choice, use the suggested default and inspect the generated configuration afterward.

The command creates a new cli-practice directory. Move into it:

~~~sh
cd cli-practice
~~~

Confirm that this directory contains the generated Angular workspace configuration and package.json before running project commands.

## 4. Inspect the generated project

Check which CLI version and workspace projects are present:

~~~sh
ng version
ng help
npm run
~~~

Open package.json and find the local @angular/cli dependency and available scripts. The generated script names can vary, so use the names listed by npm run and do not assume every target is configured.

Check the installed Node.js version against the Angular compatibility table if setup reported a warning or the initial install failed.

## 5. Generate a small component

From inside the new workspace, run:

~~~sh
ng generate component task-summary
~~~

The CLI applies the component schematic using the workspace's configuration. Inspect the new files and any project files the command modified. File naming, test-file generation, and component style choices depend on the CLI version and project settings.

If you want to inspect generator options before changing the project, ask the installed CLI:

~~~sh
ng generate component --help
~~~

## 6. Serve and stop the development server

From the workspace root, run:

~~~sh
ng serve --open
~~~

The CLI compiles the app, starts a local development server, and opens the browser when supported. Watch the terminal for the local address and for build errors. Edit a source file and observe the development rebuild.

When you are finished, return to the same terminal and press Ctrl+C to stop the server.

This server is for local development. It does not deploy the application or produce a production hosting setup.

## 7. Build the project

From the same workspace root, run:

~~~sh
ng build
~~~

Inspect the terminal result and the configured output directory, commonly dist. If the build fails, read the first relevant diagnostic and inspect the file it identifies. If it succeeds, remember that it confirms the configured build completed; it does not deploy the app.

If the project defines test or lint targets, inspect them and ask the CLI for their options:

~~~sh
ng test --help
ng lint --help
~~~

Only run a target that exists in the workspace.

## 8. Inspect project scripts and local tooling

Compare the global CLI command with the project-local setup:

1. Open package.json and find @angular/cli under the project's dependencies.
2. Find the scripts section and note which commands are available.
3. Run npm run to list the scripts.
4. Run the documented project scripts when present, for example npm start or npm run build.
5. Check package-lock.json or the lockfile for the package manager this repository uses.

Using a project script normally makes the project-local executable available on the script path. This avoids depending on an unrelated global CLI version for the project's build workflow.

## 9. Practice command help without changing files

Use help before trying unfamiliar flags:

~~~sh
ng new --help
ng serve --help
ng build --help
ng version --help
~~~

Write down one option for each command that is relevant to your practice project. Confirm its accepted value and default in the help output for the installed CLI version. Do not copy options from tutorials that target a different release without checking.

## 10. Diagnose common problems

### The terminal says ng is not found

Check whether the global installation completed successfully and whether the current shell can find npm's global executables. If this is an existing workspace, check its local CLI dependency and project scripts too.

### Node.js is outside the supported range

Check the Angular version and compare it with the official compatibility table. Switch to a supported Node.js version using the version-management method appropriate for your operating system, then reopen the terminal and run node --version again.

### Angular cannot find a workspace

Run the command from the intended project directory. Use ng new outside the project being created, and run commands such as ng generate, ng serve, and ng build from inside the workspace.

### The command rejects an option

Use ng version and request help for that exact command. The option may belong to a different CLI version or another command. Do not update dependencies just to make a copied example work.

### Installation fails on Windows

Read the complete error. Determine whether it concerns Node.js compatibility, network or registry access, permissions, or PowerShell command execution. Follow current platform-specific documentation for that cause; avoid broad machine-policy changes as a first response.

## 11. Clean up the practice project

Stop any running dev server with Ctrl+C. If you want to keep the project, leave its source and intended lockfile in place and exclude generated dependencies such as node_modules from Git. If you created it only for this exercise and no longer need it, remove only the scratch project directory after confirming its exact location.

## Try these variations

1. Run ng version inside the practice workspace and compare the reported CLI information with package.json.
2. Generate another component using a different name and inspect the changed files.
3. Run ng serve without --open and open the displayed local address yourself.
4. Compare the generated output after ng build with the files in src.
5. If a test target exists, run it and inspect its configured runner.
6. Find the serve and build targets in angular.json and relate them to the commands you ran.
7. Write down which commands need a workspace and which command creates one.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="007-node-js-npm-package-managers-and-workspaces.md">007 - Node.js, npm, Package Managers, and Workspaces</a></td><td><a href="../lessons/008-angular-cli-installation-and-command-basics.md">Lesson</a></td><td><a href="../qnas/008-angular-cli-installation-and-command-basics.md">Q&amp;A</a></td><td><a href="009-create-a-standalone-angular-application.md">009 - Create a Standalone Angular Application</a></td>
</tr>
  </tbody>
</table>
