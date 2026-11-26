# Q&A: Components, Metadata, and Component Boundaries

## Component fundamentals

### 1. What is an Angular component?

A component is a UI building block made from a TypeScript class and Angular component metadata, with a template that defines its view. It can also have styles associated with that view.

### 2. What parts are commonly found in a component?

A component commonly has a class, an @Component decorator, a selector, a template, and optional styles. A larger component may keep its template and styles in separate files.

### 3. What does the component class do?

The class contains the component's TypeScript state and behavior. Angular creates instances of that class when the component is instantiated in the application.

### 4. What is the @Component decorator for?

It marks a class as an Angular component and attaches metadata that tells Angular how to compile, instantiate, and use the component.

### 5. What is component metadata?

Metadata is the configuration object passed to @Component. It can describe the selector, view, styles, template dependencies, providers, and other component behavior.

### 6. Is a component just a TypeScript class?

No. The class supplies behavior and state, while Angular metadata connects the class to a template and tells Angular how to use it as a component.

### 7. Is every directive a component?

No. A component is a kind of directive that has a template. Other directives add behavior to or change existing elements or views without defining their own component template.

### 8. Can two components be instantiated on the same template element?

Angular allows only one component to be instantiated for a given element. Directives can also match the element, but a second component cannot share that same component host element.

## Metadata and files

### 9. What does selector do?

The selector identifies where a component can be used in a template. Angular selector naming and matching rules are covered in more detail in the dedicated selectors lesson.

### 10. What is the difference between template and templateUrl?

template contains an inline template string in the component metadata. templateUrl points to an external template file, usually relative to the component source file.

### 11. Can I specify both template and templateUrl?

No. Choose one way to define the template for a component.

### 12. What is the difference between styles and styleUrl?

styles contains inline CSS in component metadata. styleUrl points to one external stylesheet file. The styleUrls array can point to multiple stylesheet files.

### 13. Are external template and style files required?

No. Angular supports inline and external templates and styles. External files can make larger components easier to read, while inline files can be convenient for small examples.

### 14. Are templateUrl and styleUrl paths relative to the workspace root?

Normally they are resolved relative to the component's source file. Check the component file location when a path cannot be resolved.

### 15. Can metadata contain every component behavior?

Metadata configures the component, but it is not a replacement for class logic. State, methods, and other TypeScript behavior belong in the class or in appropriate services.

### 16. Do I need to specify every metadata option?

No. Specify the options needed by the component and use suitable framework defaults. Adding configuration without a need can make components harder to understand.

## Component tree and responsibility

### 17. What is the Angular component tree?

It is the hierarchy formed when component templates contain other components. The root component is at the top, and its nested components make up child branches of the rendered UI.

### 18. Does the component tree have to match the data model?

No. The component tree describes UI composition. The data model and services can have different structures.

### 19. What does a component boundary own?

A component boundary groups a view with the class behavior and styles needed for that view. It provides a place to understand one meaningful part of the UI.

### 20. How do I know when to create a child component?

Create one when a piece of UI has a clear reusable or independently understandable responsibility, or when dividing it makes a large parent easier to work with. Avoid splitting every small HTML fragment without a design reason.

### 21. Should a page component contain every UI behavior on the page?

Not necessarily. A page can coordinate a screen, while focused child components handle distinct pieces of presentation and interaction. Shared business operations may belong in services.

### 22. How do parent and child components communicate?

Common patterns include inputs for data flowing into a child and outputs for events flowing back to a parent. Shared state or operations may use services. These patterns are covered in later lessons.

### 23. Does a component boundary make its data private or secure?

No. It is an architectural and rendering boundary, not an authorization boundary. Security decisions must be enforced by trusted systems, especially the server.

## Standalone and version context

### 24. What does standalone mean for a component?

A standalone component can be used directly without declaring it in an NgModule. It manages its own template dependencies. The next lesson explores standalone imports.

### 25. Are components standalone by default?

In current Angular, yes. The default changed in Angular 19. Earlier projects may default to NgModule-based components and may explicitly set standalone: true or standalone: false.

### 26. Should I add standalone: true in a new component anyway?

It is optional in current Angular, but spelling it out can make a teaching example or a mixed-version project easier to read. Follow the workspace's Angular version and conventions.

### 27. Can an older NgModule-based project still use components?

Yes. Existing projects can declare components through NgModules. Angular supports maintaining that style; standalone components are the recommended default for new development.

## Styles and responsibility choices

### 28. What does component style encapsulation do?

It controls how component styles are applied. The default emulated mode scopes component styles so they normally target that component's view without using native Shadow DOM.

### 29. Does emulated encapsulation use native Shadow DOM?

No. Emulated encapsulation rewrites/scopes selectors to approximate style isolation. ShadowDom mode uses the browser's native Shadow DOM behavior.

### 30. Are component styles completely isolated from global CSS?

No. Global styles can still affect the page, and style encapsulation modes have different behavior. Do not treat the default as a perfect CSS sandbox.

### 31. Can style encapsulation protect sensitive UI or data?

No. CSS encapsulation is not a security feature. Browser-rendered content and client code are accessible to the user; enforce access control outside CSS.

### 32. Where should global design tokens or resets go?

Global resets, shared typography, and application-wide design tokens generally belong in global stylesheets. Styles tied to one component's view usually belong with that component.

### 33. When should behavior go in a service instead of a component?

Use a service when behavior or data should be shared across components or does not belong to a single view. Keep view-specific rendering state and interaction in the relevant component when that makes ownership clearer.

## Troubleshooting

### 34. Why does Angular say that a class is not a component?

Check that the class has a correctly imported @Component decorator and valid metadata, and confirm that the file is included in the expected build. Also check the actual compiler error for a more specific metadata issue.

### 35. Why is an external template or stylesheet not found?

Check the path relative to the component TypeScript file, confirm the filename and letter case, and ensure the file exists at that location.

### 36. Why does a component work in one template but not another?

The component must be in the scope of the template where it is used. Standalone imports and NgModule declarations establish that scope; the next lesson covers standalone template imports.

### 37. What should I inspect when a component becomes difficult to maintain?

Review whether its class, template, and styles still describe one cohesive UI responsibility. Check for unrelated state or operations that belong in child components or services, and split only where a meaningful boundary improves clarity.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="012-bootstrap-applicationconfig-and-application-providers.md">012 - 12. Bootstrap, ApplicationConfig, and application providers</a></td><td><a href="../lessons/013-components-metadata-and-component-boundaries.md">Lesson</a></td><td><a href="../examples/013-components-metadata-and-component-boundaries.md">Example</a></td><td><a href="014-standalone-components-and-template-imports.md">014 - 14. Standalone components and template imports</a></td>
</tr>
  </tbody>
</table>
