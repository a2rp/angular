# Examples: Node.js, npm, Package Managers, and Workspaces

These exercises inspect a real Angular CLI project and then build a tiny two-package npm workspace. The first part shows how Node.js and npm support Angular development. The second part demonstrates what npm workspaces add when a repository contains multiple packages.

## Part 1: Inspect an Angular project

### 1. Check the installed runtimes

Open a terminal in an Angular application directory and inspect the installed versions:

~~~sh
node --version
npm --version
~~~

Compare the Node.js version with the Angular compatibility table for the version listed in the project's dependencies. Do not install a different runtime just because a tutorial names one; check the compatibility requirements for this project.

### 2. Read the package manifest and lockfile

Inspect **package.json** and identify:

- The Angular framework packages and their version ranges.
- The local Angular CLI package in development dependencies.
- The scripts the project actually defines.
- Other development tools, such as the TypeScript compiler and test tooling.

Then identify the lockfile. For npm this is **package-lock.json**. The lockfile records selected versions, including transitive dependencies. It should match the package manager used by the repository.

Do not use several package managers on the same project during this exercise. If the project has **pnpm-lock.yaml** or a Yarn lockfile, use the manager indicated by that lockfile and the project instructions.

### 3. Install and inspect project commands

For an npm project that has a package-lock file, install from the project root:

~~~sh
npm install
npm run
~~~

The first command installs the project's dependencies. The second lists scripts defined by this project's manifest. The exact list depends on the Angular CLI version and project choices.

If the project defines the corresponding scripts, try:

~~~sh
npm start
npm run build
npm run test
~~~

Some Angular workspaces define a serve script instead of start. Use the names shown by npm run. These commands use the project's installed tooling through its scripts; they do not require you to guess which global Angular CLI version is active.

### 4. Practice a clean npm installation

If this is a disposable learning project with a committed and compatible package-lock file, run:

~~~sh
npm ci
~~~

npm ci removes the existing node_modules directory and installs the complete project from the lockfile. It should not rewrite package.json or package-lock.json. If it reports that the lockfile and manifest disagree, inspect the changes and project instructions instead of deleting the lockfile.

## Part 2: Create a two-package npm workspace

This small example uses only Node.js built-ins and local files, so it does not need a third-party package. Create the following structure in a new practice directory:

~~~text
workspace-demo/
  package.json
  apps/
    greeting-app/
      package.json
      index.js
  packages/
    greeting/
      package.json
      index.js
~~~

### 1. Add the root manifest

Create **workspace-demo/package.json**:

~~~json
{
  "name": "workspace-demo",
  "private": true,
  "type": "module",
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "scripts": {
    "start": "npm run start --workspace=@demo/greeting-app"
  }
}
~~~

The workspaces patterns tell npm which nested package directories belong to the root. The private field helps prevent accidentally publishing this demonstration root package. The type field tells Node to interpret ordinary JavaScript files in this package scope as ECMAScript modules.

### 2. Create the shared package

Create **packages/greeting/package.json**:

~~~json
{
  "name": "@demo/greeting",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "exports": "./index.js"
}
~~~

Create **packages/greeting/index.js**:

~~~js
export function greeting(name) {
  return 'Hello, ' + name + '!';
}
~~~

This package exposes one function through its package entry point.

### 3. Create the application package

Create **apps/greeting-app/package.json**:

~~~json
{
  "name": "@demo/greeting-app",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "dependencies": {
    "@demo/greeting": "1.0.0"
  },
  "scripts": {
    "start": "node index.js"
  }
}
~~~

Create **apps/greeting-app/index.js**:

~~~js
import { greeting } from '@demo/greeting';

console.log(greeting('Angular learner'));
~~~

The application declares the shared package as a dependency by its package name. This declaration makes the dependency explicit; the app should not rely on npm accidentally placing a package somewhere it can be found.

### 4. Install and run from the root

Open a terminal in **workspace-demo** and run:

~~~sh
npm install
npm run start
npm run start --workspace=@demo/greeting-app
~~~

The root install reads the workspace configuration and links the local greeting package into the npm installation. Both start commands run the application package's script: the root script forwards to it, and the second command selects it directly.

Expected output:

~~~text
Hello, Angular learner!
~~~

After installation, inspect the generated package-lock.json and node_modules directory. Commit the root package.json, each workspace package.json, source files, and the lockfile if this project is placed under version control. Do not commit node_modules.

### 5. Try workspace changes

1. Change the greeting text and run the app again.
2. Add a second exported function to the greeting package and import it from the app.
3. Add a script to the greeting package and run it with npm run followed by the workspace option.
4. Remove the local package dependency declaration temporarily and observe why explicit dependency declarations matter.
5. Inspect the root lockfile after installation. Do not edit it manually.

## Relating npm workspaces to Angular

An Angular CLI workspace is organized by Angular CLI configuration, commonly **angular.json**. An npm workspace is defined by the root package.json and coordinates multiple packages. A repository may use both, for example to organize several Angular apps and shared packages, but an ordinary single-app Angular project does not need an npm monorepo.

For a multi-package Angular repository, decide which layer owns each concern:

- Angular CLI configuration describes Angular applications and libraries and their build targets.
- npm workspace configuration identifies local package directories and coordinates package installs and scripts.
- Each package manifest declares that package's name, scripts, and dependencies.
- Lockfiles record the package manager's dependency resolution for reproducible installs.

Before adding workspace configuration to an existing project, check its Angular CLI structure and package-manager setup. Follow the current Angular and npm documentation for the versions in use.

## References

- [Lesson 007: Node.js, npm, Package Managers, and Workspaces](../lessons/007-node-js-npm-package-managers-and-workspaces.md)
- [Q&A 007](../qnas/007-node-js-npm-package-managers-and-workspaces.md)
- [Angular local environment setup](https://angular.dev/tools/cli/setup-local)
- [Angular version compatibility](https://angular.dev/reference/versions)
- [Node.js introduction](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs)
- [Node.js packages and module formats](https://nodejs.org/api/packages.html)
- [npm workspaces](https://docs.npmjs.com/cli/v11/using-npm/workspaces/)
- [npm install](https://docs.npmjs.com/cli/v12/commands/npm-install/)
- [npm ci](https://docs.npmjs.com/cli/v11/commands/npm-ci/)

---

[← Lesson 007](../lessons/007-node-js-npm-package-managers-and-workspaces.md) | [← Q&A 007](../qnas/007-node-js-npm-package-managers-and-workspaces.md)
