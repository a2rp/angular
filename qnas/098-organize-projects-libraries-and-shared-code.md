# Organize Projects, Libraries, and Shared Code — Q&A

## 1. What is the difference between an Angular workspace and an Angular project?

A workspace is the Angular CLI environment that contains shared configuration and one or more projects. A project is an application or library the CLI can build, test, or otherwise process. The workspace's projects configuration maps project names to settings; it is not required to match the directory tree one-to-one.

## 2. Can a workspace contain more than one application?

Yes. Angular CLI supports multi-project workspaces. Additional applications and libraries are commonly placed under projects/, while sharing workspace-level configuration and package dependencies. Whether this is useful depends on whether those projects benefit from coordinated development and shared tooling.

## 3. When should applications use separate repositories?

Separate repositories can fit applications with independent ownership, deployment schedules, dependency policies, or toolchains. They make repository ownership clear and reduce the coordination surface, though code and configuration may be duplicated or shared through packages.

## 4. When is a multi-project workspace useful?

It can be useful when applications and libraries are developed together, changes frequently span projects, and shared dependencies or configuration are valuable. It also requires clear ownership, dependency direction, and build practices. A monorepo does not provide those policies automatically.

## 5. When should repeated code become a library?

Consider a library when code is truly reused and has a stable public API, clear ownership, and an intentional release or project boundary. A small helper shared within one application may not need a library. Avoid extracting code solely because reuse might happen later.

## 6. What is the difference between shared source code and a library?

A shared folder is usually an organizational boundary inside an application. A library is a separate Angular workspace project with its own configuration, public entry point, and build target. A library can be used across applications and may be packaged for distribution, but it brings additional API and compatibility responsibilities.

## 7. Why use the Angular CLI to generate a library?

The CLI creates the expected library structure and updates workspace project configuration. Hand-creating only a source folder does not register a buildable Angular library. Use the installed CLI's help to confirm version-specific options.

## 8. What is a library public API?

It is the supported import surface consumers can rely on. The public API file exports intended components, services, or types. Keep implementation details private, and avoid consumers importing internal source paths because those paths are harder to change safely.

## 9. Should every helper be exported from a library?

No. Every export can become a compatibility commitment. Export only what consumers need and what the owning team is prepared to support. A small, documented public API is easier to evolve than an accidental export of every implementation detail.

## 10. How should dependencies flow between projects?

A useful direction is from applications to feature or domain libraries and then to small shared foundations. Avoid cycles and avoid making a low-level UI library depend on a business application. If two projects need a common contract, define a small lower-level API or revisit ownership.

## 11. How do peer dependencies matter for a published Angular library?

A published library generally should not bundle another copy of Angular. Framework packages are commonly declared as peer dependencies so the consuming application provides a compatible version. Check Angular's current library packaging and compatibility guidance before publishing.

## 12. How do I build one project in a multi-project workspace?

Use the CLI target with the configured project name, for example ng build admin or ng build ui-kit. Exact project names and available targets depend on workspace configuration. Inspect angular.json or CLI help rather than assuming names.

## 13. Is every feature folder a library?

No. A feature folder groups code by capability inside an application. A library is a separate workspace project. Start with feature-local code and introduce a library when stable reuse, independent ownership, or a project boundary justifies it.

## 14. What should be planned before extracting a library?

Identify its consumers, define the supported API, choose ownership and dependency direction, generate the project with the CLI, update imports to use the public entry point, and decide how builds and tests work. If the library will be published, also plan package compatibility and versioning.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="097-feature-based-application-architecture.md">097 - Feature-Based Application Architecture</a></td><td><a href="../lessons/098-organize-projects-libraries-and-shared-code.md">Lesson</a></td><td><a href="../examples/098-organize-projects-libraries-and-shared-code.md">Example</a></td><td><a href="099-reusable-ui-components-and-design-system-boundaries.md">099 - Reusable UI Components and Design-System Boundaries</a></td>
</tr>
  </tbody>
</table>
