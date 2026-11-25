# 24. Built-in directives and structural directive syntax

Directives add behavior to elements and components. Angular has attribute directives that change an element's appearance or behavior, and structural directives that add or remove embedded views. Angular also has built-in template control-flow blocks such as `@if` and `@for`; those are part of template syntax, not directives.

This lesson focuses on built-in directives and the asterisk syntax used by structural directives. The newer control-flow blocks were introduced in the previous lessons and are the preferred syntax for new conditional and repeated template content.

## Learning goals

After this lesson, you should be able to:

- Distinguish components, attribute directives, structural directives, and control-flow blocks.
- Recognize common built-in directives such as `NgClass` and `NgStyle`.
- Explain the purpose of the structural-directive asterisk shorthand.
- Translate a simple structural shorthand expression into its `<ng-template>` form.
- Read `let` variables, key-expression pairs, and aliases in structural microsyntax.
- Explain why only one asterisk structural directive can appear on an element.
- Use `ng-container` to create a structural wrapper without adding a DOM element.
- Recognize legacy `*ngIf`, `*ngFor`, and `*ngSwitch` syntax and know the modern alternatives.
- Import directives into standalone components when they are used in a template.

## 1. What is a directive?

A directive is an Angular class that adds behavior to an existing element or component. A selector tells Angular where to apply it. Directives do not necessarily render their own component template.

Angular's main UI building blocks include:

| Kind | Main purpose | Example |
| --- | --- | --- |
| Component | Owns a template and defines a reusable UI boundary | `<app-user-card>` |
| Attribute directive | Changes the appearance or behavior of an existing host | `[ngClass]` |
| Structural directive | Creates or removes embedded template views | Legacy `*ngIf` |
| Control-flow block | Selects or repeats template content as built-in template syntax | `@if`, `@for`, `@switch` |

A component is also a directive in Angular's implementation model, but it is useful to distinguish components from attribute and structural directives when reasoning about what the template is doing.

## 2. Attribute directives

An attribute directive applies behavior to the element that carries it. It generally changes appearance or behavior without deciding whether the element exists.

### NgClass

`NgClass` adds and removes CSS classes based on an expression. It can receive a string, an array, a set, or an object whose keys are class names and whose values determine whether those classes are active:

~~~html
<div [ngClass]="{ selected: isSelected, muted: isArchived }">
  {{ itemName }}
</div>
~~~

In a standalone component, import `NgClass` from `@angular/common` and add it to the component's `imports` array. A component using `CommonModule` also has access to it.

For a simple case, a direct class binding is often clearer and does not require importing `NgClass`:

~~~html
<div [class.selected]="isSelected">
  {{ itemName }}
</div>
~~~

Use `NgClass` when its expression makes a group of dynamic classes easier to manage. Use a direct class binding for a single class or a simple conditional.

### NgStyle

`NgStyle` applies multiple dynamic inline styles from a key-value object:

~~~html
<div [ngStyle]="{ 'font-size.px': fontSize, color: textColor }">
  Resizable text
</div>
~~~

The optional unit suffix in a key, such as `.px`, lets the directive add a unit to the value. In a standalone component, import `NgStyle` from `@angular/common`. `CommonModule` also exports it.

For a small number of styles, direct style bindings are often easier to read:

~~~html
<div [style.font-size.px]="fontSize" [style.color]="textColor">
  Resizable text
</div>
~~~

Direct class and style bindings are built into Angular template syntax. Choose the form that makes the number of bindings and their conditions clearest.

### Other built-in directives

Angular also provides built-in directives for other tasks, including `NgTemplateOutlet` for rendering a template fragment and `NgOptimizedImage` for image-loading improvements. See the relevant lessons and API documentation before importing or using a directive. Each directive has a specific purpose; do not assume every directive belongs to the category that its name suggests.

## 3. Structural directives change rendered views

A structural directive controls whether or how many embedded views Angular creates from a template. The familiar asterisk form marks the element's content as the template controlled by that directive.

For example, legacy `*ngIf` conditionally creates its host content:

~~~html
<p *ngIf="isReady">The data is ready.</p>
~~~

This syntax is not a normal HTML attribute. Angular interprets the asterisk syntax and creates an implicit `<ng-template>` around the element and its children. When the condition is false, the embedded view is absent; this differs from merely hiding an existing element with CSS.

In new templates, use the built-in control-flow syntax instead:

~~~html
@if (isReady) {
  <p>The data is ready.</p>
}
~~~

The `@if` block is built into Angular's template language and does not require importing `NgIf` or `CommonModule`.

## 4. Structural shorthand is microsyntax

The content after the asterisk follows Angular's structural-directive microsyntax. It allows the template to express directive inputs and local template variables compactly.

A custom structural directive could use this shorthand:

~~~html
<p *select="let data; from: source">
  {{ data }}
</p>
~~~

Angular expands it conceptually to:

~~~html
<ng-template select let-data [selectFrom]="source">
  <p>{{ data }}</p>
</ng-template>
~~~

The asterisk shorthand creates an implicit template, places the structural directive on it, and wraps the original element inside. This is the same template model described in the previous lesson on `ng-template`.

### Let declarations

In `let data`, the local variable `data` receives the template context's `$implicit` value. To map a local to a named context value, write an explicit export:

~~~html
<ng-template select let-data="result">
  <p>{{ data }}</p>
</ng-template>
~~~

The declaration means that `data` reads the `result` property exported by the structural directive's template context. A directive defines which context values it provides.

### Key-expression pairs

In `from: source`, `from` is a microsyntax key and `source` is a template expression. Angular converts the key to PascalCase and prefixes it with the directive selector. For `select`, the key `from` maps to the input `selectFrom`.

This naming convention explains why a custom structural directive often names its inputs with the directive's selector as a prefix. It also explains familiar legacy syntax such as `*ngFor="let user of users"`: the `of` key maps to the `ngForOf` input.

### Aliases

Microsyntax can assign a directive-exported value to a local alias. A familiar legacy example is:

~~~html
<div *ngIf="user as currentUser">
  {{ currentUser.name }}
</div>
~~~

This aliases the value exported by `ngIf`. New conditional code should use `@if` and its own supported alias syntax:

~~~html
@if (user; as currentUser) {
  <div>{{ currentUser.name }}</div>
}
~~~

The exact variables available depend on the directive or control-flow construct. Do not assume every structural directive exports the same values.

## 5. One asterisk structural directive per element

Only one asterisk structural directive can be attached to the same element:

~~~html
<!-- Invalid: two structural shorthand directives on one element -->
<li *ngFor="let user of users" *ngIf="user.active">
  {{ user.name }}
</li>
~~~

Each asterisk requires Angular to create an implicit `<ng-template>`. Two directives on one element would require Angular to guess which template should wrap the other.

Use built-in control flow for new templates:

~~~html
@for (user of users; track user.id) {
  @if (user.active) {
    <li>{{ user.name }}</li>
  }
}
~~~

If you are working with a structural directive in existing code, use explicit nesting to make the intended order clear:

~~~html
<ng-container *ngFor="let user of users">
  <li *ngIf="user.active">{{ user.name }}</li>
</ng-container>
~~~

This legacy example is for reading and maintaining existing templates. For new list rendering, use `@for` and `@if`.

## 6. Use ng-container for grouping

`<ng-container>` provides a template location or grouping point without adding a normal element to the rendered DOM. It can create a nesting layer when structural behavior must be separated from the actual HTML element:

~~~html
<ng-container *ngFor="let user of users">
  <li>{{ user.name }}</li>
</ng-container>
~~~

The `ng-container` itself does not render as a wrapper. The `li` elements do. For new loops, prefer the built-in `@for` block.

Do not replace semantic elements with `ng-container` when a real element is needed for meaning, accessibility, styling, or interaction.

## 7. Built-in structural directives and modern control flow

Older Angular templates often use structural directives from `@angular/common`:

- `*ngIf` conditionally renders a view.
- `*ngFor` repeats a view for collection items.
- `[ngSwitch]` together with `*ngSwitchCase` and `*ngSwitchDefault` chooses matching views.

The Angular API reference currently marks `NgIf`, `NgFor`, and `NgSwitch` as deprecated since v20 and recommends `@if`, `@for`, and `@switch` for new templates. This is important when following older tutorials: the old syntax still appears in existing code, but new course examples should use the current built-in blocks.

These are the main modern forms:

~~~html
@if (isReady) {
  <p>Ready</p>
} @else {
  <p>Loading</p>
}

@for (user of users; track user.id) {
  <p>{{ user.name }}</p>
} @empty {
  <p>No users yet.</p>
}

@switch (status) {
  @case ('active') {
    <p>Active</p>
  }
  @default {
    <p>Inactive or unknown</p>
  }
}
~~~

Built-in control-flow blocks do not need to be imported. They are not the same API as the legacy directives with similar purposes.

Other built-in directives, including `NgClass`, `NgStyle`, and `NgTemplateOutlet`, remain directives and must be in the template's scope when used.

## 8. Importing directives in standalone components

A standalone component declares its template dependencies in the component metadata:

~~~ts
import { Component } from '@angular/core';
import { NgClass, NgStyle } from '@angular/common';

@Component({
  selector: 'app-status-label',
  standalone: true,
  imports: [NgClass, NgStyle],
  template: '<span [ngClass]="classes" [ngStyle]="styles">{{ label }}</span>',
})
export class StatusLabel {
  classes = { active: true };
  styles = { 'border-width.px': 2 };
  label = 'Available';
}
~~~

You can import `CommonModule` instead when a component needs several common directives and pipes. Importing individual dependencies makes a small standalone component's needs explicit. The built-in blocks `@if`, `@for`, and `@switch` do not require either import.

## 9. Common mistakes

### Treating the asterisk as decoration

The asterisk activates structural-directive shorthand. Angular creates an implicit template and gives the directive control of its embedded view.

### Putting two asterisk directives on one element

Only one implicit template can be created for that element. Nest the behavior using `ng-container`, or express the logic with built-in control flow.

### Confusing hidden content with absent content

A structural directive creates or removes a view. CSS visibility leaves the element in the DOM and only affects presentation. Choose based on whether the content should exist in the rendered view.

### Forgetting the standalone import

Directives such as `NgClass` and `NgStyle` must be imported into a standalone component's template scope. By contrast, Angular's built-in `@if` and `@for` blocks need no import.

### Assuming microsyntax keys map directly to input names

The key is combined with the directive selector. For `*select="from: source"`, the input is `selectFrom`, not just `from`.

### Treating a control-flow block as a directive

`@if` and `@for` are built-in template syntax. They are not `NgIf` and `NgFor`, and they do not need to be imported.

### Copying old syntax into new code without checking deprecations

Older tutorials may use `*ngIf`, `*ngFor`, and `ngSwitch`. Check Angular's current API reference and prefer the recommended built-in control-flow blocks in new work.

## Practical checklist

When reading or writing directive syntax, ask:

- Does this behavior belong to an attribute directive, a structural directive, or a built-in control-flow block?
- If the template uses `NgClass` or `NgStyle`, is the directive imported?
- If the syntax starts with `*`, what implicit `<ng-template>` does Angular create?
- Does each microsyntax key map to the input name you expect?
- Is there more than one structural shorthand directive on the same element?
- Would `ng-container` clarify the nesting without creating an unnecessary DOM wrapper?
- For new conditional and repeated content, should this use `@if`, `@for`, or `@switch` instead?

## Further reading

- [Directives overview](https://angular.dev/guide/directives)
- [Structural directives and microsyntax](https://angular.dev/guide/directives/structural-directives)
- [Attribute directives](https://angular.dev/guide/directives/attribute-directives)
- [Control flow blocks](https://angular.dev/guide/templates/control-flow)
- [NgIf API and deprecation status](https://angular.dev/api/common/NgIf)
- [NgFor API and deprecation status](https://angular.dev/api/common/NgFor)
- [NgClass API](https://angular.dev/api/common/NgClass)
- [NgStyle API](https://angular.dev/api/common/NgStyle)
