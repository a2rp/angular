# Classes, Modules, Decorators, and Compiler Options

TypeScript uses JavaScript classes and modules, while Angular adds decorators such as @Component to describe framework metadata. The compiler configuration determines how TypeScript and Angular analyze and build the project. Understanding these pieces helps you read an Angular workspace without confusing language features, framework features, and build-tool settings.

## Learning objectives

After this lesson, you should be able to:

- Read a TypeScript class with fields, a constructor, methods, and access modifiers.
- Explain the difference between extends and implements.
- Describe what a TypeScript source module is and how imports and exports work.
- Distinguish ECMAScript modules from Angular NgModules.
- Explain what an Angular decorator such as @Component communicates.
- Recognize that TypeScript has legacy and standardized decorator modes.
- Identify common TypeScript and Angular compiler configuration sections.
- Avoid changing compiler options without understanding the project's toolchain.

## Classes are runtime JavaScript structures

A class defines a constructor and behavior for creating instances. Unlike interfaces and most type annotations, a class is emitted as JavaScript and exists at runtime.

~~~ts
class TaskCounter {
  private total: number;

  constructor(initialTotal: number = 0) {
    this.total = initialTotal;
  }

  add(amount: number = 1): number {
    this.total += amount;
    return this.total;
  }

  get count(): number {
    return this.total;
  }
}

const counter = new TaskCounter(2);
counter.add();
console.log(counter.count);
~~~

The constructor initializes instance state. Methods and getters use this to refer to the current instance. A getter is accessed like a property, even though its implementation is a method.

A class is useful when an application needs instances with associated state and behavior. Not every piece of application code needs to be a class; plain objects and functions are often simpler for data and stateless operations.

## Access modifiers and initialization

Class members are public by default. TypeScript also supports private and protected members:

- **public** members can be accessed from outside the class. Writing public is optional because it is the default.
- **private** members are checked as accessible only inside the declaring class by TypeScript.
- **protected** members are available inside the declaring class and its subclasses.
- **readonly** members can be assigned during initialization, such as in a field initializer or constructor, but cannot be reassigned later through that type.

TypeScript's private modifier is enforced by the type checker and is not the same as JavaScript's runtime private field syntax beginning with #. Use runtime private fields when runtime encapsulation is required; use TypeScript access modifiers when expressing and checking source-level access.

With strict property initialization enabled, a declared field must be initialized by a field initializer or constructor, or its type must account for an initially absent value. Avoid using the definite-assignment assertion merely to silence an error unless another mechanism truly guarantees initialization.

## Inheritance and interfaces

A class can extend a base class to inherit and specialize its implementation:

~~~ts
class BaseStore {
  describe(): string {
    return 'Base store';
  }
}

class TaskStore extends BaseStore {
  override describe(): string {
    return 'Task store';
  }
}
~~~

When a subclass replaces a base method, override makes that intention explicit. Projects that enable noImplicitOverride can require this keyword for overrides.

A class can also implement an interface:

~~~ts
interface Identifiable {
  id: number;
}

class TaskRecord implements Identifiable {
  constructor(public id: number, public title: string) {}
}
~~~

The implements clause checks that the class has the required shape. It does not create a missing property or method, and it does not change how the class body is inferred. An interface is still type information; TaskRecord is a runtime class.

Use inheritance when there is a real subtype relationship and shared behavior is useful. For unrelated objects that happen to have the same shape, structural typing often removes the need for a class hierarchy.

## ECMAScript modules: import and export

A TypeScript source file can export values and types for other files to import:

~~~ts
// task.ts
export interface Task {
  id: number;
  title: string;
}

export function taskLabel(task: Task): string {
  return task.id + ': ' + task.title;
}
~~~

~~~ts
// task-page.ts
import { taskLabel, type Task } from './task';

const task: Task = { id: 1, title: 'Read module basics' };
console.log(taskLabel(task));
~~~

An import or export makes a source file an ECMAScript module. Named exports make the imported name explicit. Type-only imports communicate that an imported symbol is used only for type checking and can be removed from runtime output.

A module is not the same as a class. A module is a source-file boundary for imports and exports; a class defines a runtime constructor and its instances.

## Do not confuse source modules with Angular NgModules

JavaScript and TypeScript modules use import and export statements between files. Angular NgModules are a separate Angular framework mechanism for grouping declarations and providers in applications that use that architecture.

Modern standalone Angular components can import other components, directives, and pipes directly in component metadata. This does not remove ECMAScript import statements: the component class itself still comes from a source file.

When a guide says “module,” check whether it means an ES module, an Angular NgModule, an npm package, or another unit.

## What decorators do in Angular

A decorator is written with an at sign before a class or member. Angular uses decorators to attach framework metadata:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-task-counter',
  standalone: true,
  template: '<button type="button">Tasks</button>',
})
export class TaskCounterComponent {}
~~~

@Component tells Angular's compiler and runtime how the class participates as a component. The object passed to the decorator is Angular metadata, including information such as selector, template, imports, and styles. Angular's [Component API reference](https://angular.dev/api/core/Component) describes this framework decorator.

A decorator is not itself a TypeScript type annotation. TypeScript type checking does not infer that a class has the shape of a component just because it is decorated. Angular's compiler interprets Angular metadata and checks Angular-specific concerns such as templates.

## Decorator modes and version awareness

TypeScript historically provided an experimental decorator implementation controlled by the experimentalDecorators compiler option. TypeScript 5.0 added support for the newer standardized decorator semantics. These modes differ in type checking and emitted JavaScript; decorators written for one mode are not necessarily compatible with the other.

The newer standardized decorators do not support parameter decorators and are not compatible with emitDecoratorMetadata. Existing Angular projects use the decorator setup expected by their Angular and TypeScript versions. Do not toggle experimentalDecorators or related metadata settings simply because a blog post shows a different configuration.

For an Angular application, preserve the configuration generated for its Angular version unless the official Angular or TypeScript migration instructions say otherwise. The [TypeScript 5.0 release notes](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html) explain the difference between standard and legacy decorators.

## The TypeScript configuration file

A tsconfig.json file tells the TypeScript compiler how to analyze and emit the project's source. Angular workspaces can also include Angular-specific compiler options:

~~~json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitOverride": true,
    "experimentalDecorators": true
  },
  "angularCompilerOptions": {
    "strictTemplates": true,
    "strictInjectionParameters": true
  }
}
~~~

This is an illustrative fragment, not a complete replacement for an Angular CLI-generated configuration. Real workspaces may split settings across tsconfig.json, tsconfig.app.json, and other files that extend a shared configuration.

The top-level compilerOptions object contains TypeScript compiler settings. The angularCompilerOptions object contains Angular compiler settings. Angular CLI build and serve targets are configured separately, commonly in angular.json.

## Common compiler options

| Option | What it influences |
| --- | --- |
| strict | Enables a group of stricter TypeScript checks |
| noImplicitOverride | Requires explicit override on overriding class members |
| target | Controls the JavaScript language level emitted by TypeScript and affects the default library assumptions |
| module | Controls the module format used or preserved by compilation |
| moduleResolution | Controls how TypeScript resolves imports and packages for the chosen toolchain |
| experimentalDecorators | Enables TypeScript's older experimental decorator mode |
| strictTemplates | Enables stricter type checking of Angular templates |
| strictInjectionParameters | Checks that Angular can determine injection types for constructor parameters |

The appropriate target, module, and moduleResolution depend on the Angular CLI builder, bundler, runtime, Angular version, and TypeScript version. Do not copy values from a Node.js tutorial into a browser application or vice versa without checking the toolchain.

The TypeScript [TSConfig reference](https://www.typescriptlang.org/tsconfig/) documents TypeScript compiler options. Angular's [compiler options guide](https://angular.dev/reference/configs/angular-compiler-options) documents angularCompilerOptions and configuration inheritance.

## Configuration inheritance

A configuration can extend another configuration. The child configuration inherits the base options and can override or add settings.

~~~json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "./out-tsc/app"
  },
  "angularCompilerOptions": {
    "strictTemplates": true
  }
}
~~~

Before changing a setting, find which configuration file is used by the relevant CLI target and inspect its extends chain. Editing a base setting may affect application code, tests, and other build targets.

## Practical reading order for a compiler error

When a TypeScript or Angular build reports an error:

1. Read the exact file and line reported.
2. Identify whether it is a TypeScript, Angular template, dependency-resolution, or build-tool error.
3. Check the relevant source value and inferred type before changing configuration.
4. Find which tsconfig file the build target uses.
5. Check the TypeScript and Angular versions supported by the project.
6. Change the smallest relevant source or configuration setting.
7. Re-run the command that originally reported the problem.

Configuration changes can make an error disappear by disabling checks without fixing the cause. Prefer correcting the source when the error identifies a real type or template mismatch.

## Common misunderstandings

- **“A class and an interface are both compile-time only.”** Classes produce runtime JavaScript constructors; interfaces are erased type information.
- **“implements creates the required members.”** It checks the class against the contract but does not generate missing behavior.
- **“private means a field is inaccessible at runtime.”** TypeScript's private is a source-level check; JavaScript # fields provide runtime privacy.
- **“An ES module is an Angular NgModule.”** They are different mechanisms with different roles.
- **“Decorators are type annotations.”** They attach or describe metadata and are interpreted according to the decorator system and framework.
- **“All TypeScript decorators behave the same.”** Standard and legacy decorator modes have different semantics and compiler support.
- **“experimentalDecorators should always be removed in TypeScript 5.”** Existing frameworks and projects may rely on the legacy mode; use the configuration supported by that project.
- **“strictTemplates is a TypeScript compiler option.”** It belongs in Angular compiler options, not the standard TypeScript compilerOptions object.
- **“Changing target fixes every compatibility problem.”** Target affects emitted JavaScript and library assumptions, but browser support, bundling, dependencies, and polyfills are separate concerns.

## Knowledge check

1. What exists at runtime after a TypeScript class is compiled?
2. What does implements check, and what does it not do?
3. How does an ECMAScript module differ from an Angular NgModule?
4. What information does Angular's @Component decorator provide?
5. How do TypeScript's newer and legacy decorator modes differ?
6. Which configuration section contains strictTemplates?
7. What do module and moduleResolution control at a high level?
8. Why should you inspect configuration inheritance before changing a setting?

Answers are in [Chapter 6 Q&A](../qnas/006-classes-modules-decorators-and-compiler-options.md). The [Chapter 6 examples](../examples/006-classes-modules-decorators-and-compiler-options.md) show these ideas in a small Angular workspace.

## Further reading

- [TypeScript classes](https://www.typescriptlang.org/docs/handbook/2/classes.html)
- [TypeScript modules introduction](https://www.typescriptlang.org/docs/handbook/modules/introduction.html)
- [TypeScript 5.0 decorators](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html)
- [TypeScript TSConfig reference](https://www.typescriptlang.org/tsconfig/)
- [Angular Component API](https://angular.dev/api/core/Component)
- [Angular compiler options](https://angular.dev/reference/configs/angular-compiler-options)
