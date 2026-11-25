# Q&A: Standalone Components and Template Imports

## Standalone components

### 1. What is a standalone component?

A standalone component does not need to be declared in an NgModule. It can be imported directly by another standalone component, and it lists the dependencies used by its own template.

### 2. Are components standalone by default?

In current Angular, yes. The default changed in Angular 19. Projects using older Angular versions may require standalone: true explicitly or may be based on NgModules.

### 3. Is standalone: true required in current Angular?

No. It can be omitted in current Angular, although including it in a teaching example can make the intent obvious. Check the actual Angular version used by the workspace.

### 4. What does standalone: false mean?

It opts a component, directive, or pipe out of standalone behavior so it can be declared in an NgModule. Older Angular projects commonly use this style.

### 5. Do standalone components replace every NgModule immediately?

No. Standalone APIs are the recommended default for new development, while NgModules remain supported for existing applications and libraries. Teams can migrate incrementally.

## The imports array

### 6. What is the @Component imports array for?

It defines template dependencies available to that standalone component's template. Entries can include standalone components, directives, pipes, and NgModules.

### 7. Which dependencies may appear there?

Any standalone component, directive, or pipe that the template uses can be imported directly. An NgModule can also be imported when a library or older application exposes dependencies through a module.

### 8. What is the difference between a TypeScript import and @Component imports?

A TypeScript import makes a symbol available in the TypeScript source file. The @Component imports array makes eligible Angular template dependencies available in that component's template. A standalone dependency commonly needs both.

### 9. Why does Angular have two imports lists?

They serve different systems. The top-level TypeScript import resolves JavaScript/TypeScript symbols. The metadata array establishes Angular's template compilation scope.

### 10. Does importing a component in TypeScript alone let me use its selector in a template?

No. The symbol must also be in the consuming standalone component's @Component imports array, or be available through that component's NgModule scope in a module-based app.

### 11. Does listing a class in @Component imports load its source file?

No. The class must already be available to the TypeScript file, normally through a TypeScript import. The metadata entry then registers it as a template dependency.

### 12. Does every component need to import its own dependencies?

Yes. Each standalone component declares what its own template uses. A dependency in one component's imports array does not become a global app-wide template dependency.

### 13. Do a child component's imports become available in its parent template?

No. A child's imports apply to the child's template. The parent separately imports the child component if its own template uses that child.

### 14. If a page imports a child component, must the page also import the child's pipes?

No, unless the page's own template uses those pipes. The child component is responsible for its own template dependencies.

### 15. Can I put a service in @Component imports?

No. The imports array is for template dependencies, not injectable service configuration. Services are made available through dependency injection providers, often with providedIn: 'root' or application configuration.

### 16. Can I put provideRouter(routes) in @Component imports?

No. provideRouter configures application-level router providers. A component imports RouterOutlet or other template directives only when its own template uses them.

## Built-in template features and common imports

### 17. Does @if need an import?

No. Angular's built-in control-flow syntax such as @if and @for is part of template syntax and does not require NgIf or NgFor imports.

### 18. Does *ngIf need an import?

Yes. A template using the legacy *ngIf directive needs NgIf or a module that exports it, such as CommonModule, in that template's scope. Consider built-in @if syntax in newer Angular code.

### 19. Does every standalone component need CommonModule?

No. Import CommonModule only when the template needs exports it provides and the project uses those APIs. A component that uses no such directives or pipes does not need it.

### 20. Can I import a pipe directly instead of importing CommonModule?

Yes, when the pipe is standalone and directly exported for import. For example, a component that uses the date pipe can import DatePipe from @angular/common and list DatePipe in its metadata imports.

### 21. What happens if the template uses a pipe that is not in scope?

Angular reports a template compilation error or warning depending on the exact issue and compiler configuration. Add the pipe or its exporting module to the imports of the component whose template uses it.

### 22. Is importing RouterOutlet the same as configuring routes?

No. Import RouterOutlet to use its element in a component template. Configure the router and route definitions separately with application providers such as provideRouter(routes).

## NgModule interoperability

### 23. Can a standalone component import an NgModule?

Yes. This allows the standalone component's template to use the declarations and exports made available by that module.

### 24. Can a standalone component be placed in an NgModule declarations array?

No. Import the standalone component in the NgModule's imports array instead of declaring it.

### 25. Can an NgModule import a standalone component?

Yes. An NgModule can import standalone components, directives, and pipes and can export dependencies as needed.

### 26. Can a non-standalone component be imported directly by a standalone component?

No. A non-standalone component belongs to an NgModule. Import the NgModule that declares and exports it, or migrate the component if appropriate.

### 27. What does an NgModule export do?

It makes selected declarations or imported dependencies available to other NgModules or standalone components that import that NgModule.

### 28. Does importing an NgModule only affect template scope?

Not always. An NgModule may also configure providers. Check that module's documentation and provider behavior when importing it into a standalone component.

## Scope and troubleshooting

### 29. Where should I add a missing dependency?

Add it to the standalone component whose template uses it. Do not add every dependency to the root component unless the root template actually uses them.

### 30. Why does a selector render as an unknown element?

Check that the component is imported into the template scope of the component containing that element, or that the correct NgModule scope is established. Also verify the selector spelling.

### 31. Why does Angular say imports is unavailable or invalid?

Check whether the component is standalone. A component declared in an NgModule cannot also use the standalone component imports metadata.

### 32. What if I import the wrong symbol?

Check the import path and exported class name, then verify the imported value is an Angular standalone component, directive, pipe, or an NgModule. A similarly named service or plain TypeScript utility is not a template dependency.

### 33. Can I import the same dependency in multiple components?

Yes. Each template declares its own dependencies. This is normal when several components use the same shared pipe or UI component.

### 34. Should I create a shared module just to avoid repeating standalone imports?

Usually not for a new standalone app. Direct imports make dependencies explicit. A shared NgModule can still be useful for a legacy integration boundary or a library that intentionally exports a set of related dependencies.

### 35. What should I check when migrating a component to standalone?

Make it standalone, identify its current NgModule template dependencies, and list those dependencies in its own imports array. Then update consumers to import the component directly and check the build for dependencies that were previously inherited from module scope.

## Further reading

- [Using components and template imports](https://angular.dev/guide/components/importing)
- [Component decorator API](https://angular.dev/api/core/Component)
- [NgModules overview](https://angular.dev/guide/ngmodules/overview)
- [Standalone migration guide](https://angular.dev/reference/migrations/standalone)
