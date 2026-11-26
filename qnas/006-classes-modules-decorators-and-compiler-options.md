# Q&A: Classes, Modules, Decorators, and Compiler Options

## Classes and object contracts

### 1. What is a TypeScript class?

A class is a JavaScript runtime construct that defines how to create objects and which methods or fields they have. TypeScript adds compile-time checks to its members. After compilation, the class still exists as JavaScript.

### 2. How is a class different from an interface?

A class can create instances at runtime and can contain implementation. An interface describes a type contract for the checker and is normally erased from emitted JavaScript. Use a class when runtime construction or behavior is needed; use an interface when describing the shape an object must have.

### 3. Does an interface create an object?

No. It can help TypeScript check an object literal or class, but it does not create a value at runtime. Write an object literal, call a class constructor, or use another runtime factory to create the object.

### 4. What does structural typing mean?

TypeScript generally checks whether a value has the required members, rather than requiring it to declare a particular interface by name. A value with the compatible shape can satisfy an interface. This flexibility still has limits, including excess-property checks on some fresh object literals.

### 5. What is the purpose of a constructor?

A constructor runs when a class is instantiated with the new operator. It can receive dependencies or input values and initialize the instance so it is ready to use.

~~~ts
class Task {
  constructor(public readonly title: string) {}
}
~~~

This parameter property declares and initializes a member as well as declaring a constructor parameter.

### 6. What does this refer to in a class method?

In an instance method, this refers to the instance on which the method is called. If a method is detached from its instance and later called as a plain function, its this behavior may differ. Arrow-function fields capture this, but they also create a function per instance, so choose them when the binding behavior is useful.

### 7. What is the difference between an instance member and a static member?

An instance member belongs to each constructed object and is accessed through that object. A static member belongs to the class constructor itself and is accessed through the class name.

~~~ts
class Ids {
  static next = 1;
}

const id = Ids.next;
~~~

### 8. What do public, protected, and private mean?

They control which TypeScript code may access a member:

- public is the default and can be accessed wherever the object is visible.
- protected is available inside the class and its subclasses.
- TypeScript private is restricted by the type checker to the declaring class.

These modifiers do not all provide the same runtime privacy guarantees. JavaScript #private fields provide runtime-enforced private access.

### 9. Does TypeScript private protect a value at runtime?

No. TypeScript's private modifier is primarily a compile-time access restriction. JavaScript #field syntax is enforced at runtime. Do not put secrets in browser-side class fields; client code can be inspected regardless of a modifier.

### 10. What does readonly guarantee?

It prevents assignment to a property after its initialization according to TypeScript's checks. It is not a deep immutability guarantee: an object held by a readonly reference can still have mutable nested properties unless those are also modeled as readonly or otherwise protected.

### 11. What is strictPropertyInitialization?

It is a strict TypeScript option that checks whether instance properties are initialized by a field initializer or on every applicable constructor path. Prefer real initialization. A definite-assignment assertion such as name! only tells the checker initialization happens elsewhere; it does not initialize the value.

### 12. When should I use extends and when should I use implements?

A class uses extends to inherit implementation and members from a base class. It uses implements to promise that it satisfies a type contract; the class must provide the required members. An interface can extend other interfaces, while a class can extend only one class.

### 13. What does the override keyword do?

It marks a subclass member as intentionally replacing a base-class member. With noImplicitOverride enabled, TypeScript can require this marker and catch accidental mismatches when a base class changes. It does not create a new kind of runtime dispatch.

### 14. When should I prefer composition over inheritance?

Prefer composition when one object can use another object's behavior without being a specialized kind of that object. It often makes responsibilities easier to replace and test. Use inheritance when the subtype relationship is genuine and the subclass can safely honor the base class's contract.

## Modules and Angular organization

### 15. What is a TypeScript module?

A file with a top-level import or export is an ECMAScript module. It has its own scope and can expose selected values or types. Export makes a declaration available to other modules; import consumes it.

### 16. What is the difference between a named export and a default export?

A named export is imported by its exported name, optionally with an alias. A module can have multiple named exports. A default export is the module's designated default value and is imported without braces. Consistent named exports can make refactoring and symbol discovery straightforward.

### 17. What is import type for?

It marks an import used only in type positions, so it can be removed from emitted JavaScript. It makes the type-only intent explicit and can help avoid runtime dependency edges. Use a normal import when the imported value is needed at runtime.

~~~ts
import type { Task } from './task';
import { TaskCounter } from './task-counter';
~~~

### 18. How is an ECMAScript module different from an Angular NgModule?

An ECMAScript module is a JavaScript or TypeScript file with imports and exports. An Angular NgModule is a framework class decorated with @NgModule that groups Angular declarations, imports, providers, and bootstrap configuration in module-based applications. Standalone Angular components are also supported; a TypeScript module is not the same thing as an NgModule.

### 19. Why does a file without imports or exports sometimes behave differently?

A file without a top-level import or export may be treated as a script whose declarations share global scope. Adding export {} makes it a module, though the right fix depends on the project. Follow the repository's module configuration and avoid adding exports only to silence an unrelated design issue.

## Decorators and Angular

### 20. What is a decorator?

A decorator is syntax attached to a class or member that participates in decorator processing. In Angular, decorators such as @Component provide framework metadata that Angular uses to create and compile components. A decorator is not a TypeScript type annotation.

### 21. What does @Component describe?

It associates Angular metadata with a component class, including information such as its selector, template or template URL, styles, and other component options. Angular's compiler uses that metadata to understand how the component participates in an application.

### 22. Are TypeScript decorators and Angular decorators interchangeable with any JavaScript decorator?

No. Decorator behavior depends on the TypeScript and framework toolchain. Angular has compiler and runtime requirements. Follow the Angular version's generated configuration and current Angular documentation rather than assuming every decorator proposal or compiler mode is interchangeable.

### 23. What changed with standardized decorators in TypeScript 5?

TypeScript 5 added support for the newer standardized decorator model, whose type-checking and emitted behavior differ from TypeScript's older experimental decorator implementation. The newer model does not support every legacy feature, including some patterns based on parameter decorators and emitDecoratorMetadata. Angular applications have specific compiler requirements, so do not switch decorator modes or remove Angular-generated options casually.

### 24. What does experimentalDecorators mean in an Angular configuration?

It enables TypeScript's legacy experimental decorator implementation when the toolchain requires it. Angular CLI-generated configuration can include decorator-related options. Keep the settings generated for the installed Angular and TypeScript versions unless you have verified the migration path and compiler requirements.

### 25. Should I add decorators to ordinary helper classes?

Only when a framework feature needs them. A plain data model, utility, or algorithm usually does not need an Angular decorator. Keeping framework metadata at framework boundaries makes simple TypeScript code easier to reuse.

## Compiler options and troubleshooting

### 26. Where do compiler options live in an Angular project?

TypeScript options usually live under compilerOptions in a tsconfig file. Angular-specific compiler options can live under angularCompilerOptions. Build and serve targets are configured in the workspace configuration, commonly angular.json. The exact files and inheritance depend on the Angular CLI version and project structure.

### 27. What is the difference between strict and strictTemplates?

TypeScript's strict option enables a family of stronger checks for TypeScript source. Angular's strictTemplates enables more rigorous checking of Angular templates, including bindings and expressions. One does not replace the other; an Angular project may configure both in their respective sections.

### 28. What do target, module, and moduleResolution control?

target influences the JavaScript language level emitted by TypeScript. module controls the module format or module syntax expected by the toolchain. moduleResolution controls how TypeScript finds imported modules. Modern Angular builders coordinate these settings with bundling, so preserve the CLI's supported configuration instead of copying options from an unrelated project.

### 29. What does extends do in a tsconfig file?

It lets one TypeScript configuration inherit options from another. Angular CLI projects commonly have a root configuration and app or spec configurations that extend it. A child configuration can override inherited settings. When debugging, inspect the configuration actually used by the failing build target, not just the root file.

### 30. Why is changing compilerOptions by hand risky?

A setting can affect type checking, emitted code, module resolution, decorators, or compatibility with the Angular builder. Copying an option from an older tutorial may conflict with the installed versions. Identify the precise diagnostic, check the Angular and TypeScript versions, consult their matching documentation, then change only the needed setting.

### 31. How should I investigate a TypeScript compiler error?

Read the first relevant diagnostic and identify whether it is a type error, a missing module, a configuration issue, or an Angular template/compiler error. Check the source and the active tsconfig path. Make the smallest correction and use the project's normal CLI command to verify it.

### 32. How should I investigate an Angular template type error?

Check the component's public template-facing members, input/output types, nullability, and the template expression named in the diagnostic. With strict template checking, a template error can reveal a real mismatch that looser checking missed. Prefer fixing the component or template contract over disabling strict checking.

### 33. Does a TypeScript type validate JSON received at runtime?

No. Type annotations and interfaces are erased and cannot prove that untrusted runtime data has the expected shape. Parse input as unknown, then validate it with runtime checks or a schema validator before treating it as a particular type.

### 34. Do these notes replace the official documentation?

No. Use these notes to learn the concepts, then verify version-sensitive details in the official references below and in the documentation for the versions installed in the project.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="005-typescript-interfaces-unions-generics-and-narrowing.md">005 - TypeScript Interfaces, Unions, Generics, and Narrowing</a></td><td><a href="../lessons/006-classes-modules-decorators-and-compiler-options.md">Lesson</a></td><td><a href="../examples/006-classes-modules-decorators-and-compiler-options.md">Example</a></td><td><a href="007-node-js-npm-package-managers-and-workspaces.md">007 - Node.js, npm, Package Managers, and Workspaces</a></td>
</tr>
  </tbody>
</table>
