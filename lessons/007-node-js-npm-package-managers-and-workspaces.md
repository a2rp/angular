# Node.js, npm, Package Managers, and Workspaces

Angular development uses more than the browser. Node.js runs JavaScript tooling on your computer, a package manager installs project dependencies, and workspace configuration organizes projects and commands. Knowing which tool owns each job makes setup and dependency problems easier to diagnose.

## Learning objectives

After this lesson, you should be able to:

- Explain what Node.js and npm do in an Angular development workflow.
- Distinguish a runtime, package manager, package, and project workspace.
- Read the important parts of package.json and understand a lockfile.
- Choose between installing a dependency, running a project script, and using a CLI.
- Explain Angular version compatibility and why the project-local CLI matters.
- Distinguish an Angular CLI workspace from an npm workspace.
- Use an npm workspace for multiple related packages.
- Recognize CommonJS and ECMAScript module markers in Node projects.

## Node.js runs JavaScript tools outside the browser

Node.js is a JavaScript runtime built on the V8 engine. It lets JavaScript run outside a web page and includes APIs for tasks such as working with files, processes, and network connections.

An Angular browser application eventually runs in a browser. During development, however, Angular CLI and build tools run on Node.js. They read project files, compile TypeScript and templates, bundle code, run development servers, and execute other project scripts. Node.js is therefore part of the development toolchain even when an application does not use a Node.js server in production.

Node.js and a browser are different runtimes. Browser APIs such as window and document are not automatically available in a Node process. Likewise, Node APIs such as filesystem access are not browser APIs. Code intended to run in the browser should use browser APIs; build tooling and server-side programs can use Node APIs.

Check the required Node version for the Angular version in the project. Angular publishes a compatibility table for Node.js, TypeScript, and RxJS. Requirements change over time, so use the table for the version you are installing or maintaining rather than copying a version number from an old tutorial.

## npm is a package manager and command-line tool

Node.js distributions commonly include npm. npm can install packages, run commands declared by a project, and manage dependency metadata. The npm registry is one source of published packages; projects can also use other registries or package sources.

A package is code plus metadata that can be installed or shared. An Angular project depends on packages such as Angular framework packages and development tools. A package manager resolves those dependencies and places installed files in a local installation directory, commonly node_modules.

A package manager does not replace Node.js. Node runs JavaScript programs; npm manages packages and provides commands for working with a project.

## The project manifest and dependency categories

A package.json file describes a JavaScript project or package. It can contain a package name, scripts, dependency declarations, supported runtime information, and other metadata.

A typical Angular application's manifest includes packages in these broad categories:

- dependencies are packages needed by the application at runtime or during its normal bundled execution.
- devDependencies are packages used for development tasks such as compilation, testing, linting, or building.
- peerDependencies describe compatibility expectations for a package intended to work alongside a host package. Libraries often use them to declare which host versions they support.
- optionalDependencies are dependencies that may be installed when available but are not required in every environment.

The correct category depends on how a package is used and on the package or build tool's conventions. Do not move dependencies between categories just to make the manifest look smaller.

A version declaration usually describes a range, not necessarily one exact version. Semantic versioning uses major, minor, and patch numbers. Range operators such as caret and tilde allow some updates within specified boundaries. The exact interpretation depends on the range and starting version; inspect the declaration and package manager documentation before assuming what can change.

## Lockfiles make dependency resolution repeatable

A manifest says which versions are acceptable. A lockfile records the dependency resolution selected for this project, including transitive dependencies. npm uses package-lock.json; other package managers have their own lockfile formats.

For a given package manager:

- Commit the intended lockfile so collaborators and automated builds can use the same resolved dependency tree.
- Do not edit a lockfile by hand.
- Avoid committing multiple competing lockfiles for one project unless the team intentionally maintains that setup.
- Review manifest and lockfile changes together when dependencies are added, removed, or updated.
- Keep node_modules out of source control; it is generated by installation.

With npm, npm install is the normal command for installing project dependencies and updating the lockfile when the manifest changes. npm ci is intended for a clean, reproducible install in automation or when you need to verify that the manifest and lockfile agree. npm ci requires a compatible lockfile, removes an existing node_modules directory, and does not update the manifest or lockfile. It is not the command for adding one package.

## Use the project-local Angular CLI

Angular CLI is published as the @angular/cli npm package and provides the ng command. An Angular project typically declares the CLI version it uses in its own development dependencies. This local version is the one the project can install and record with its lockfile.

Installing the CLI globally can provide a convenient ng command for creating or opening projects. A global installation is separate from the versions declared by a particular project. When working inside an existing repository, use that repository's local CLI through its package scripts or the documented local-execution command. This keeps project commands aligned with the repository's Angular version.

Common ways to inspect or run an Angular project include:

~~~sh
node --version
npm --version
npm install
npm run
npm start
npm run build
npm run test
~~~

The available script names vary. Read the scripts section of package.json before assuming that start, build, or test exists. npm run with no script name lists the available scripts. Commands such as ng new create a workspace; commands such as ng generate are run from an Angular workspace.

Angular compatibility is a framework constraint as well as a package-manager concern. A successful package install does not prove that arbitrary Node, TypeScript, Angular CLI, Angular framework, and RxJS versions are compatible. Check the official Angular compatibility table when changing one of these versions.

## Package managers: choose one installation workflow

npm, pnpm, Yarn, and Bun can install JavaScript dependencies and run project commands. They share many concepts but have different commands, lockfiles, and installation behavior.

A repository's lockfile and setup instructions indicate its intended package manager. Use that manager consistently:

- npm uses package-lock.json.
- pnpm uses pnpm-lock.yaml.
- Yarn uses a Yarn lockfile.
- Bun uses bun.lock or bun.lockb depending on the version and configuration.

Do not convert or delete a lockfile just to try another package manager. A conversion can change the dependency graph, install behavior, or metadata. If a project is intentionally migrated, make that a deliberate change and review the resulting lockfile and build.

Angular's setup guide documents supported installation choices for the Angular CLI. The package manager used to install the global CLI does not automatically determine which package manager an existing project uses. For the project itself, follow its lockfile and team instructions.

## Angular CLI workspace and npm workspace are different

An Angular CLI workspace is the project structure and configuration managed by Angular CLI. It commonly has an angular.json file and may contain one or more Angular applications and libraries.

An npm workspace is a package-manager feature configured in the root package.json. It groups multiple local packages so npm can install and run scripts across them. One repository can use both concepts: Angular CLI can organize Angular applications and libraries, while npm workspaces can organize local Node packages. They are related only when the repository configures them that way.

A small npm workspace can look like this:

~~~text
team-dashboard/
  package.json
  apps/
    admin/
      package.json
  packages/
    shared-ui/
      package.json
~~~

The root package.json can declare workspace folders:

~~~json
{
  "name": "team-dashboard",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "scripts": {
    "build:admin": "npm run build --workspace=@team/admin"
  }
}
~~~

Each matched folder must itself have a package.json with a unique package name. For example, the shared package might declare:

~~~json
{
  "name": "@team/shared-ui",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "build": "tsc -p tsconfig.json"
  }
}
~~~

After workspace configuration exists, npm install at the root installs dependencies for the root and configured workspaces. npm links local workspace packages into the installation so sibling packages can consume them by package name. A workspace package must still declare the dependencies it uses; relying on an accidentally hoisted package can make a package fail when installed on its own.

Useful npm workspace command forms include:

~~~sh
npm install
npm run build --workspace=@team/admin
npm run test --workspaces
npm install some-package --workspace=@team/admin
~~~

A workspace command may run in the selected package context. Check each package's scripts and dependency declarations. For a simple Angular learning application, a monorepo is not required; use workspaces when multiple packages genuinely need to be developed and versioned together.

## Node.js module formats

Node.js supports CommonJS and ECMAScript modules. CommonJS commonly uses require and module.exports. ECMAScript modules use import and export. Node determines how JavaScript files are interpreted from markers such as .mjs or .cjs file extensions and the nearest package.json type field.

Angular source code normally uses TypeScript imports and exports and is processed by the Angular build toolchain. Do not add a Node module type setting to an Angular project merely because a Node tutorial uses it. If a project contains Node scripts, libraries, or server code, follow the module format configured for that part of the project.

## A reliable setup workflow

When setting up or repairing an Angular project:

1. Read the README and identify the package manager from its lockfile and instructions.
2. Check the required Node.js version against Angular's compatibility table.
3. Check the installed versions declared in package.json and recorded in the lockfile.
4. Use the selected package manager to install or clean-install dependencies.
5. Run the scripts already declared by the project.
6. If installation or a build fails, inspect the first relevant error and the package versions before changing configuration.
7. Review any manifest and lockfile changes before committing them.

Do not install a global package to compensate for a missing local dependency without first checking the project's manifest. Do not delete a lockfile as a general repair step. These actions can hide the cause or make the project use a different tool version.

## Key takeaways

- Node.js runs JavaScript tooling outside the browser; npm manages packages and project commands.
- Angular development needs a Node version supported by the Angular version in use.
- package.json declares dependencies and scripts; the lockfile records a concrete dependency resolution.
- Use the package manager indicated by the repository and keep its lockfile under version control.
- Prefer the Angular CLI version installed by the project when working in that project.
- An Angular CLI workspace and an npm workspace solve different organizational problems.
- Workspaces help manage multiple local packages, but a single Angular app does not need them.
- Node's CommonJS and ECMAScript module markers matter for Node-executed code; Angular bundling has its own project configuration.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="006-classes-modules-decorators-and-compiler-options.md">006 - Classes, Modules, Decorators, and Compiler Options</a></td><td><a href="../qnas/007-node-js-npm-package-managers-and-workspaces.md">Q&amp;A</a></td><td><a href="../examples/007-node-js-npm-package-managers-and-workspaces.md">Example</a></td><td><a href="008-angular-cli-installation-and-command-basics.md">008 - Angular CLI Installation and Command Basics</a></td>
</tr>
  </tbody>
</table>
