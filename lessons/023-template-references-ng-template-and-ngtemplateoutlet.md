# 23. Template references, ng-template, and ngTemplateOutlet

Angular template references let a template refer to an element, component, directive, or reusable template fragment. The `<ng-template>` element declares a fragment without rendering it immediately. `NgTemplateOutlet` can render that fragment where it is needed and provide local values to it.

~~~html
<ng-template #notice let-message>
  <p>{{ message }}</p>
</ng-template>

<ng-container
  [ngTemplateOutlet]="notice"
  [ngTemplateOutletContext]="{ $implicit: 'Your changes were saved.' }">
</ng-container>
~~~

The reference named `notice` identifies a `TemplateRef`. The context supplies the value exposed as `message`. The outlet renders the paragraph at its location.

## Learning goals

After this lesson, you should be able to:

- Explain what an Angular template reference variable points to.
- Describe why an `<ng-template>` fragment is not visible until Angular renders it.
- Reference a fragment with a template reference variable.
- Render a fragment with `NgTemplateOutlet`.
- Pass values through a context object and declare template-local variables with `let-`.
- Use `$implicit` for the default local value.
- Explain the role of `<ng-container>` around an outlet.
- Keep template fragment scope and component expression scope straight.
- Choose between a template fragment and a separate component.

## 1. Template reference variables

A template reference variable starts with `#` and names a value associated with an element in a template:

~~~html
<input #searchBox>
<app-search-panel #searchPanel>
<ng-template #emptyState>
  <p>No results.</p>
</ng-template>
~~~

Angular assigns the reference value according to where the variable is declared:

- On an ordinary HTML element, it refers to that DOM element.
- On an Angular component, it refers to the component instance.
- On a directive with an `exportAs` name, use that export name to refer to the directive instance, for example `#form="ngForm"`.
- On an `<ng-template>`, it refers to a `TemplateRef` representing that fragment.

A reference can be used in the template scope where it is declared, and in nested scopes that can see the declaration. It is not a global variable and cannot be accessed from unrelated component templates.

## 2. What an ng-template declares

An `<ng-template>` declares a template fragment: a portion of Angular template that can be rendered later. Angular does not render the fragment's contents just because the `<ng-template>` appears in the source.

~~~html
<p>This paragraph renders immediately.</p>

<ng-template>
  <p>This paragraph is only a declared fragment.</p>
</ng-template>
~~~

In this example, only the first paragraph appears. To display the fragment, Angular needs a reference to it and a rendering mechanism such as `NgTemplateOutlet` or `ViewContainerRef`.

A fragment is not a component and does not create a normal wrapper element. Its bindings remain part of the Angular view that declared it. Expressions inside the fragment can access that declaring component's members, even if the fragment is rendered at another outlet location.

## 3. Name a fragment with a template reference

Put a template reference variable directly on the `<ng-template>`:

~~~html
<ng-template #welcome>
  <p>Welcome back.</p>
</ng-template>
~~~

The name `welcome` now refers to the fragment's `TemplateRef`. This declaration alone still renders nothing. The reference can be passed to an outlet or queried from component code when programmatic handling is needed.

Use a descriptive name that communicates the fragment's purpose, such as `loadingState`, `emptyState`, or `userCard`.

## 4. Render a fragment with NgTemplateOutlet

`NgTemplateOutlet` renders a prepared template fragment. In a standalone component, import it from `@angular/common`:

~~~ts
import { Component } from '@angular/core';
import { NgTemplateOutlet } from '@angular/common';

@Component({
  selector: 'app-welcome-panel',
  standalone: true,
  imports: [NgTemplateOutlet],
  templateUrl: './welcome-panel.html',
})
export class WelcomePanel {}
~~~

Then bind the fragment reference to the outlet:

~~~html
<ng-template #welcome>
  <p>Welcome back.</p>
</ng-template>

<ng-container [ngTemplateOutlet]="welcome"></ng-container>
~~~

The fragment is rendered at the outlet's position. `NgTemplateOutlet` can also use Angular's structural-directive shorthand:

~~~html
<ng-container *ngTemplateOutlet="welcome"></ng-container>
~~~

Both forms render the referenced fragment. The property-binding form is useful when you also bind the context or injector explicitly. A standalone component only needs to import `NgTemplateOutlet`; a component using `CommonModule` already has access to it.

## 5. Why use ng-container?

`<ng-container>` is an Angular grouping point that does not add a corresponding element to the rendered DOM. It is a natural place to attach `NgTemplateOutlet` when an extra `<div>` or other wrapper would be unnecessary or would interfere with layout or semantics.

~~~html
<ng-container [ngTemplateOutlet]="welcome"></ng-container>
~~~

The rendered paragraph is inserted at this location without an extra container element. Use real semantic HTML such as `<section>`, `<ul>`, or `<article>` when the content needs that meaning; `ng-container` does not replace semantic elements.

## 6. Pass context values to a fragment

A template fragment can declare local values with `let-` attributes. `NgTemplateOutlet` supplies those values through its context object.

~~~html
<ng-template #message let-text>
  <p>{{ text }}</p>
</ng-template>

<ng-container
  [ngTemplateOutlet]="message"
  [ngTemplateOutletContext]="{ $implicit: statusMessage }">
</ng-container>
~~~

The `let-text` declaration reads the `$implicit` entry from the context. In the component, `statusMessage` might be:

~~~ts
statusMessage = 'The profile was updated.';
~~~

Named context keys work in the same way:

~~~html
<ng-template #personCard let-person let-heading="heading">
  <article>
    <h2>{{ heading }}</h2>
    <p>{{ person.name }} — {{ person.role }}</p>
  </article>
</ng-template>

<ng-container
  [ngTemplateOutlet]="personCard"
  [ngTemplateOutletContext]="{
    $implicit: selectedPerson,
    heading: 'Selected person'
  }">
</ng-container>
~~~

Here, `let-person` receives `$implicit`, while `let-heading="heading"` receives the context object's `heading` property. The name after `let-` is the local name used within the fragment; the value after `=` identifies the context key.

The context object provides local values to the fragment. It does not change the declaring component's fields. The fragment can also use component members directly, but explicit context values make the fragment's varying inputs easier to see.

## 7. Reuse one fragment with different contexts

A single fragment can be rendered more than once with different context values:

~~~html
<ng-template #personCard let-person let-heading="heading">
  <article>
    <h2>{{ heading }}</h2>
    <p>{{ person.name }} — {{ person.role }}</p>
  </article>
</ng-template>

<ng-container
  [ngTemplateOutlet]="personCard"
  [ngTemplateOutletContext]="{
    $implicit: teamLead,
    heading: 'Team lead'
  }">
</ng-container>

<ng-container
  [ngTemplateOutlet]="personCard"
  [ngTemplateOutletContext]="{
    $implicit: designer,
    heading: 'Designer'
  }">
</ng-container>
~~~

Both outlets use the same markup, but each receives a different person and heading. This is useful when one template structure should be repeated with different data. If each item needs its own component behavior, state, or substantial styling, a dedicated component may be a clearer choice.

## 8. Fragment scope and context are different

There are two sources of values inside a fragment:

1. **Declaring component scope:** normal template expressions resolve against the component that declared the `<ng-template>`.
2. **Fragment context:** local values declared with `let-` come from the context object passed to the outlet.

For example, a fragment declared by `TeamPage` can use a `teamName` member from `TeamPage`, even when rendered at a different outlet. A `let-person` local, however, comes from the outlet context. Moving the outlet does not change which component owns the fragment's expressions.

Treat the context as the fragment's supplied local data. Do not assume that rendering the fragment inside another component automatically changes its expression scope or component ownership.

## 9. Choose a rendering approach

Use `NgTemplateOutlet` when:

- The same section of template should appear in multiple places with different local data.
- A caller or component needs to choose among template fragments.
- You want reusable markup without adding a component boundary.
- An empty, loading, or alternate-state fragment is supplied to an outlet.

Use a separate component when the repeated UI has its own inputs, outputs, state, lifecycle, or independent behavior. Use `ViewContainerRef` when the fragment must be created, moved, or removed programmatically from TypeScript. Those approaches solve related but different problems.

## 10. Common mistakes

### Expecting ng-template to render by itself

The element declares a fragment; it is not the rendered content. Reference it from an outlet or render it programmatically.

### Forgetting to import NgTemplateOutlet

A standalone component must import `NgTemplateOutlet` from `@angular/common`, or import `CommonModule`. Without the directive in the component's template scope, Angular cannot resolve the outlet directive.

### Passing a context key that does not match

For `let-heading="heading"`, the context object must provide a `heading` property. For plain `let-person`, provide `$implicit`.

### Confusing a reference with a local value

`#personCard` refers to the template fragment. `let-person` declares a local value available inside that fragment. They have separate jobs.

### Expecting a wrapper from ng-container

`ng-container` does not add a normal element. Add an actual semantic element inside the fragment if the rendered content needs structure or styling.

### Reaching outside a variable's scope

A template reference variable is available only in its template scope and nested scopes. A reference declared inside a nested block is not available to its parent template.

### Using a fragment for a full component

A template fragment does not provide a new component instance or independent component API. Prefer a component when the UI needs its own behavior, state, lifecycle, or clear public inputs and outputs.

## Practical checklist

Before using a template fragment, check that:

- The fragment has a clear purpose and a descriptive reference name.
- You have chosen where and when it should render.
- The component imports `NgTemplateOutlet` or `CommonModule`.
- Every `let-` local matches a key in the outlet context.
- `$implicit` is used intentionally for the default local.
- `ng-container` is appropriate and semantic HTML is used for meaningful structure.
- A component would not provide a clearer boundary for this reusable UI.

## Further reading

- [Create template fragments with ng-template](https://angular.dev/guide/templates/ng-template)
- [Variables in templates](https://angular.dev/guide/templates/variables)
- [NgTemplateOutlet API](https://angular.dev/api/common/NgTemplateOutlet)
- [ng-template API](https://angular.dev/api/core/ng-template)
