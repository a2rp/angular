# Q&A: Template references, ng-template, and ngTemplateOutlet

## Core concepts

### 1. What is an Angular template reference variable?

It is a named reference declared with `#name` on an element in an Angular template. Depending on where it is declared, it can refer to a DOM element, a component instance, an exported directive instance, or a `TemplateRef` for an `<ng-template>`.

### 2. What does a reference on an ordinary HTML element refer to?

It refers to that element's DOM object. For example, `<input #searchBox>` makes the input element available as `searchBox` within the template scope.

### 3. What does a reference on an Angular component refer to?

It refers to the component instance. To reference a directive instance, the directive must expose an `exportAs` name and the template reference uses that name, such as `#form="ngForm"`.

### 4. What does a reference on an ng-template refer to?

It refers to a `TemplateRef` object representing the declared template fragment. It does not refer to a rendered paragraph or wrapper element.

### 5. What is an Angular template fragment?

It is a portion of an Angular template declared with `<ng-template>`. The fragment can contain normal template markup and bindings, and Angular can render it later using an outlet or programmatic APIs.

### 6. Does Angular render the contents of ng-template immediately?

No. Declaring an `<ng-template>` defines the fragment but does not display its contents by itself. Angular must be instructed to instantiate the fragment, for example with `NgTemplateOutlet` or `ViewContainerRef`.

### 7. Is ng-template a component?

No. It does not create a component instance or a normal HTML wrapper. It is a declaration for template content that can be rendered as an embedded view.

### 8. What does TemplateRef represent?

`TemplateRef` is Angular's reference to a declared template fragment. It can be used by APIs such as `NgTemplateOutlet` or `ViewContainerRef` to create a view from that fragment.

## Rendering fragments

### 9. What does NgTemplateOutlet do?

The `NgTemplateOutlet` directive renders an embedded view from a prepared `TemplateRef` at the outlet's location. It can also receive a context object for the fragment's local variables.

### 10. Where should NgTemplateOutlet usually be placed?

Angular recommends using it on an `<ng-container>`. The fragment renders at that location, while `ng-container` does not add a corresponding wrapper element to the DOM.

### 11. What is the minimum syntax for rendering a fragment?

~~~html
<ng-template #greeting>
  <p>Hello.</p>
</ng-template>

<ng-container [ngTemplateOutlet]="greeting"></ng-container>
~~~

The outlet binds to the reference declared on the fragment.

### 12. What is the asterisk shorthand for NgTemplateOutlet?

You can write `<ng-container *ngTemplateOutlet="greeting">` instead of binding `[ngTemplateOutlet]` directly. Angular's structural-directive syntax expands the shorthand into an underlying template form.

### 13. Is the shorthand required?

No. It is a convenience. The explicit property-binding form is clear when binding multiple inputs, such as both the template and its context.

### 14. Which import does a standalone component need?

Import `NgTemplateOutlet` from `@angular/common` and add it to the standalone component's `imports` array. Importing `CommonModule` also makes it available.

### 15. Why might Angular report that ngTemplateOutlet is unknown?

The directive may not be in the template's compilation scope. In a standalone component, add `NgTemplateOutlet` or `CommonModule` to its imports. In an NgModule-based application, make sure the declaring module imports the module that exports the directive.

### 16. Does ng-container appear as an element in the rendered DOM?

No. It is an Angular grouping construct and does not add a normal DOM element. Use a semantic element when the content needs a real structure, accessible landmark, or element-specific styling.

### 17. Does the outlet render the fragment inside a wrapper element?

The outlet renders the fragment at the outlet location. When placed on `ng-container`, there is no additional `ng-container` DOM element around the rendered content.

### 18. Can one fragment be rendered more than once?

Yes. Multiple outlets can reference the same fragment. Each outlet can pass a different context so the shared markup receives different values.

## Context and local variables

### 19. What is the outlet context?

It is an object supplied to `NgTemplateOutlet` through `ngTemplateOutletContext`. Its properties are made available to the fragment through its `let-` declarations.

### 20. How do I pass a context object?

Bind the outlet context input:

~~~html
<ng-container
  [ngTemplateOutlet]="personCard"
  [ngTemplateOutletContext]="{ $implicit: person, heading: 'Team member' }">
</ng-container>
~~~

The fragment declares locals that map to those context entries.

### 21. What does a plain let-person declaration read?

It reads the context object's `$implicit` property. For `let-person` to receive a value, the context should include `$implicit: someValue`.

### 22. How do I read a named context property?

Use the context key as the value of the local declaration. For example, `let-heading="heading"` reads the `heading` property from the context object.

### 23. What is the difference between the local name and context key?

In `let-heading="title"`, `heading` is the local name used inside the template, while `title` is the key read from the context object. They can have the same name, but they do not have to.

### 24. Can I use more than one local value?

Yes. Declare a `let-` variable for each needed context value. For example, `let-person` can read `$implicit` and `let-title="heading"` can read the named `heading` key.

### 25. What is the purpose of $implicit?

It is the conventional default entry in a template context. A local declaration with no explicit key, such as `let-item`, receives the value stored under `$implicit`.

### 26. Does the context object replace the component's fields?

No. It supplies local values to that fragment instance. It does not modify the declaring component's fields.

### 27. Can a fragment use both context locals and component properties?

Yes. A fragment can use the local values declared by `let-` and the component members in whose template the fragment was declared.

### 28. Which component owns expressions inside a fragment rendered elsewhere?

They remain associated with the component that declared the fragment. Rendering the fragment at another outlet does not transfer its template expression scope to the outlet's component.

### 29. Can two outlets pass different values to the same fragment?

Yes. Each outlet can supply its own context. For example, one outlet can pass a team lead and another a designer through `$implicit`, while both render the same card structure.

## Scope, APIs, and design choices

### 30. Where can a template reference variable be used?

Within the template scope where it is declared and nested scopes that can see it. It is not global and is not automatically available to another component's template.

### 31. Can a parent template access a reference declared inside a nested block?

No. A reference declared in a nested template scope, such as inside a control-flow block, is not available to its parent scope. A nested scope can generally access values from its enclosing scope.

### 32. How is a template reference different from a let variable?

A reference such as `#personCard` names the template fragment itself. A local variable such as `let-person` names a value supplied to an instance of that fragment. One identifies the fragment; the other provides data inside it.

### 33. When should I use NgTemplateOutlet instead of a component?

Use an outlet when you need to render reusable markup with supplied local values and do not need a separate component boundary. Use a component when the UI needs its own inputs, outputs, state, lifecycle, or independent behavior.

### 34. How does NgTemplateOutlet differ from ViewContainerRef?

`NgTemplateOutlet` renders a fragment declaratively in a template. `ViewContainerRef` gives TypeScript code programmatic control to create and manage embedded views. Use the outlet for ordinary template composition and the container API when programmatic view management is needed.

### 35. Can I get a TemplateRef without a template reference variable?

Yes. Angular queries can retrieve a fragment, and a directive applied directly to an `<ng-template>` can inject `TemplateRef`. A template reference variable is a convenient option when the fragment is used in the same template.

### 36. What does ngTemplateOutletInjector do?

It is an optional `NgTemplateOutlet` input that controls the injector used by the embedded view. Most simple examples do not need it. Use it only when the fragment needs a deliberate dependency-injection context, and consult the API documentation for the Angular version in use.

### 37. Does an outlet context create a new component instance?

No. The context supplies values to a fragment instance. The fragment is an embedded view, not a component with its own component class or component lifecycle.

### 38. What should I check when a local is undefined?

Check that the outlet binds the correct context object, that the object contains the expected key, and that the fragment's declaration maps to that key. A plain `let-value` expects `$implicit`; `let-value="name"` expects a `name` property.

### 39. Should every repeated block become a template fragment?

No. Use a fragment when reuse or choosing template content makes the markup clearer. Use a component when reuse needs an explicit interface, independent behavior, local component state, or a meaningful styling and accessibility boundary.

### 40. What is a useful checklist before shipping an outlet?

Confirm that the fragment is referenced by the intended outlet, the directive is imported, every `let-` declaration matches the context, `$implicit` is intentional, and the resulting structure uses appropriate semantic HTML.

## Official references

- [Create template fragments with ng-template](https://angular.dev/guide/templates/ng-template)
- [Variables in templates](https://angular.dev/guide/templates/variables)
- [NgTemplateOutlet API](https://angular.dev/api/common/NgTemplateOutlet)
- [ng-template API](https://angular.dev/api/core/ng-template)
