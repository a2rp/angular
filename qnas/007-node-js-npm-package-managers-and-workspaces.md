# Q&A: Node.js, npm, Package Managers, and Workspaces

## Node.js and Angular tooling

### 1. What is Node.js?

Node.js is a JavaScript runtime that runs outside a browser. It uses the V8 engine and provides APIs for tasks such as reading files and operating on processes. Angular CLI and related build tools use Node.js on the developer's machine.

### 2. Does an Angular browser application need a Node.js server in production?

Not necessarily. Node.js is commonly used to build and serve an Angular application during development, but the compiled browser application can be deployed to static hosting or another server. A project that uses server-side rendering or a separate backend has additional server requirements.

### 3. Are Node.js and a browser the same runtime?

No. Browser APIs such as window and document are not generally available in a Node process. Node APIs such as filesystem access are not available to ordinary browser code. Build tools may use Node APIs to generate browser assets, but that does not make those APIs available inside the resulting application.

### 4. Why does Angular care about the Node.js version?

Angular CLI and its build tools run on Node.js, and each Angular release supports specified Node.js, TypeScript, and RxJS ranges. A version outside that supported matrix may fail to install, compile, or run reliably. Check the matrix for the specific Angular version being used.

### 5. If npm install succeeds, does that prove the toolchain versions are compatible?

No. Package installation and framework compatibility are different checks. A successful resolution does not guarantee that the selected Node.js, Angular CLI, Angular packages, TypeScript, and RxJS combination is supported. Consult Angular's compatibility table and resolve peer dependency or build diagnostics carefully.

## Packages, manifests, and dependencies

### 6. What is a package?

A package is a named, versioned unit of JavaScript code and metadata that can be installed and reused. It might be a library, a development tool, or an application. Packages can come from a registry or other supported source.

### 7. What is npm?

npm is a command-line package manager and project tool. It can install packages, run scripts from package.json, and manage npm's dependency metadata. Node.js runs the JavaScript programs; npm manages packages and project commands.

### 8. What is package.json used for?

It describes a project or package. Common fields include the project name, scripts, dependency declarations, module-format metadata, and version requirements. Read the file before assuming which commands or tools a project provides.

### 9. What is the difference between dependencies and devDependencies?

dependencies are packages needed by the application or package in its normal runtime or bundled use. devDependencies are tools needed for development tasks, such as compilation, testing, or linting. Exact conventions can depend on the package type and build pipeline.

### 10. What is a peer dependency?

A peer dependency declares that a package expects a compatible host package to be provided alongside it. For example, a library may declare compatibility with certain Angular versions rather than bundling a separate Angular framework copy. Check the peer range when diagnosing an installation warning.

### 11. What does an optional dependency mean?

It declares a dependency that may be used when it can be installed, but its absence is intended not to prevent the whole install from working. Whether it is truly optional depends on the package's design and platform. Do not assume every optional dependency can be ignored by the application.

### 12. What does a version range mean?

A range describes which package versions are acceptable to the manifest. It does not necessarily pin one exact version. Semantic-versioning operators such as caret and tilde have specific rules about which updates they allow, so inspect the exact range rather than guessing.

### 13. What is a lockfile?

A lockfile records the concrete dependency resolution selected for a project, including transitive dependencies. npm uses package-lock.json. Other package managers have their own formats. The lockfile helps teammates and automation install the same dependency tree.

### 14. Should package-lock.json be committed?

For an application, normally yes, when npm is the chosen package manager. Commit the intended lockfile so collaborators and CI can reproduce the resolution. Follow project policy for published libraries, but do not delete the lockfile as a generic troubleshooting step.

### 15. Should node_modules be committed?

Usually no. It is generated from the manifest and lockfile and can be recreated with the package manager. Repositories normally exclude it through .gitignore.

## Installing dependencies and running commands

### 16. What is the difference between npm install and npm ci?

npm install installs project dependencies and may update the lockfile when the manifest changes. npm ci is designed for clean and reproducible installs: it requires an existing compatible lockfile, removes the existing node_modules directory, fails if manifest and lockfile do not agree, and does not rewrite them. Use npm ci in automation when the project is configured for it.

### 17. Can npm ci add one new dependency?

No. It installs the project as a whole from the lockfile. To add or remove a dependency, use the project's normal package-manager workflow, then review the manifest and lockfile changes.

### 18. Why can npm ci fail when npm install seemed to work?

The manifest and lockfile may be out of sync, or the lockfile may have been generated with install settings that are not present in the clean environment. Check the diagnostic, npm configuration, and the repository's documented install process. Avoid deleting the lockfile before understanding the mismatch.

### 19. What does npm run do?

It runs a script declared in package.json. Running npm run without a script name lists available scripts. For a command such as npm run build, npm exposes the project's local executable paths to that script, which helps the command use project-installed tooling.

### 20. Why should I inspect scripts before running commands?

Script names and meanings are project-specific. A script can run arbitrary commands, so review it before execution, especially in an unfamiliar repository. Do not assume every project has start, build, or test scripts with identical behavior.

### 21. What is the difference between a global Angular CLI and a local Angular CLI?

A global installation provides a convenient command for creating or opening projects. An Angular workspace may also install a particular CLI version in its own dependencies and lockfile. Inside an existing project, use the project's local version through its scripts or documented invocation so the tool matches the project.

### 22. Should I install a global CLI to fix a missing project command?

Not automatically. First check package.json, the lockfile, and project instructions to see whether the needed CLI is already a local development dependency. A global tool can be a different version and can hide a missing or incomplete project setup.

## Package-manager choices and workspaces

### 23. What is the difference between npm, pnpm, Yarn, and Bun?

They are different tools for installing JavaScript packages and running project commands. Their commands overlap, but their lockfiles, configuration, and installation layouts differ. Use the manager identified by the repository rather than mixing them casually.

### 24. Can one project keep package-lock.json and pnpm-lock.yaml as interchangeable backups?

They are not interchangeable. Each describes a resolution for its manager, and maintaining multiple active lockfiles can lead to different dependency trees. Use the lockfile and manager chosen by the project; migrate only deliberately.

### 25. What is an Angular CLI workspace?

It is an Angular project organization managed by Angular CLI, usually with angular.json configuration. It can contain Angular applications and libraries. The CLI uses this structure for commands such as serving, building, and generating Angular code.

### 26. What is an npm workspace?

It is an npm feature that groups multiple local packages under one root package.json. npm can install their dependencies and run scripts in selected workspaces. Workspace packages have their own package.json files and declared names.

### 27. Are an Angular CLI workspace and an npm workspace the same thing?

No. They solve different organization problems. An Angular CLI workspace organizes Angular projects and CLI configuration. An npm workspace groups local packages for package-manager operations. A repository can use both, but only if it configures both.

### 28. Does every Angular app need npm workspaces?

No. A single application can use a normal Angular CLI workspace and one package.json. npm workspaces are useful when multiple packages need to be developed together, such as related applications and shared libraries in one repository.

### 29. How does npm connect a local workspace package to another workspace?

The root package.json lists workspace folder patterns. During installation, npm recognizes matching package folders and links local workspace packages into the installation. A consuming package should still declare the dependency by package name and must not rely on undeclared, accidentally hoisted packages.

### 30. How can I run a script in one npm workspace?

Use npm's workspace option with the package name or path, for example:

~~~sh
npm run build --workspace=@team/admin
~~~

The package must exist in the root workspace configuration and define the requested script.

### 31. What is a monorepo?

A monorepo is one repository containing multiple related projects or packages. Workspaces can help manage local dependencies and scripts in a monorepo, but a monorepo does not require npm workspaces, and using workspaces does not automatically make a project a good monorepo.

## Node.js module formats

### 32. What is CommonJS?

CommonJS is a Node.js module format commonly associated with require and module.exports. Node also supports ECMAScript modules. Which format applies depends on file extensions and package configuration.

### 33. How does Node know whether a JavaScript file is an ECMAScript module?

Node can use explicit markers such as the .mjs or .cjs extension and the nearest package.json type field. A type value of module marks .js files in that package scope as ECMAScript modules; commonjs marks them as CommonJS. Follow the project's intended module setup.

### 34. Should I add a type field to an Angular project because a Node tutorial does?

No. Angular source is handled by the project's TypeScript and Angular build tools. A Node module setting matters for JavaScript that Node itself executes. Change project module configuration only when you understand which tool consumes it and how that setting affects the build.

## Troubleshooting and safe habits

### 35. What should I check when dependency installation fails?

Start with the first useful error. Check the Node and package-manager versions, project instructions, manifest, lockfile, registry configuration, and Angular compatibility requirements. Determine whether it is a network, authentication, peer dependency, platform, or version-range issue before changing files.

### 36. What should I check when a package command says the executable is missing?

Check whether the package is declared and installed in this project, whether the correct package manager was used, and whether the relevant script or local binary exists. Do not assume a global installation is the correct fix.

### 37. Is installing a package risk-free?

No. A dependency can run install-time scripts or add code that later runs in the application or build. Prefer packages from trusted maintainers, review proposed manifest and lockfile changes, and follow the project's security policy.

### 38. Where can I find the authoritative version requirements?

Use Angular's version compatibility guide for the Angular, Node.js, TypeScript, and RxJS combination. Use the official Node.js and npm documentation for runtime, module, package, and command behavior. Version-specific advice from an old tutorial may no longer apply.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="006-classes-modules-decorators-and-compiler-options.md">006 - Classes, Modules, Decorators, and Compiler Options</a></td><td><a href="../lessons/007-node-js-npm-package-managers-and-workspaces.md">Lesson</a></td><td><a href="../examples/007-node-js-npm-package-managers-and-workspaces.md">Example</a></td><td><a href="008-angular-cli-installation-and-command-basics.md">008 - Angular CLI Installation and Command Basics</a></td>
</tr>
  </tbody>
</table>
