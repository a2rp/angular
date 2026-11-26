# Q&A: Built-in directives and structural directive syntax

## Directive basics

### 1. What is an Angular directive?

A directive is an Angular class that adds behavior to an element or component. Its selector determines which template nodes receive that behavior.

### 2. What are the three common directive categories?

Components define UI with their own template. Attribute directives change the appearance or behavior of a host element. Structural directives control the creation or removal of embedded template views.

### 3. Is a component also a directive?

Yes. Components are built on Angular's directive model, but they add their own template and component boundary. In everyday discussion, Angular developers distinguish components from attribute and structural directives because their roles differ.

### 4. What is an attribute directive?

It applies behavior to the element that carries it. For example, `NgClass` changes an element's CSS classes and `NgStyle` updates its inline styles.

### 5. What is a structural directive?

It controls whether or how many embedded views Angular renders from a template. Structural directives commonly use the asterisk shorthand, such as legacy `*ngIf`.

### 6. Are @if, @for, and @switch directives?

No. They are built-in Angular control-flow blocks in template syntax. They do not need to be imported as directives.

### 7. Do attribute directives normally add or remove their host element?

No. They typically change the host element's appearance or behavior. Structural directives control embedded views and can make content appear or disappear.

## Built-in attribute directives

### 8. What does NgClass do?

`NgClass` adds or removes CSS classes according to its expression. It accepts supported forms such as a string, array, set, or object whose keys are class names and whose truthy values activate those classes.

### 9. When is a direct class binding preferable to NgClass?

For one class or a simple condition, a binding such as `[class.selected]="isSelected"` is often shorter and needs no directive import. Use `NgClass` when its richer expression improves readability.

### 10. What does NgStyle do?

`NgStyle` applies multiple inline style values from an object. A key can include a unit suffix, such as `font-size.px`, so Angular applies the value using that unit.

### 11. When is a direct style binding preferable to NgStyle?

For a small number of values, a binding such as `[style.font-size.px]="fontSize"` can be clearer. Use `NgStyle` when applying a collection of dynamic styles makes the template easier to understand.

### 12. Does NgClass or NgStyle create a wrapper element?

No. They act on the host element that carries the directive.

### 13. What is another built-in directive covered in the previous chapter?

`NgTemplateOutlet` renders a template fragment from a `TemplateRef`. It is a directive, unlike the built-in `@if` and `@for` control-flow blocks.

## Structural shorthand and microsyntax

### 14. What does the asterisk mean in *ngIf?

The asterisk invokes Angular's structural-directive shorthand. Angular creates an implicit `<ng-template>` and places the directive on that template, wrapping the original element and its descendants.

### 15. Does *ngIf simply set CSS display to none?

No. It controls whether the embedded view is instantiated. CSS hiding normally leaves the element in the DOM and changes only its presentation.

### 16. What is structural microsyntax?

It is the compact grammar inside an asterisk directive expression. It can describe directive inputs and local variables with parts such as `let item`, `of items`, `from: source`, and aliases.

### 17. How does this shorthand expand: *select="let data; from: source"?

Conceptually, Angular produces:

~~~html
<ng-template select let-data [selectFrom]="source">
  <!-- the original element and its children -->
</ng-template>
~~~

The directive selector is `select`; the key `from` maps to the input `selectFrom`.

### 18. What does let data mean in structural shorthand?

It declares a local template variable named `data`. With no explicit export after the equals sign, it reads the directive's `$implicit` context value.

### 19. How do I map a local variable to a named context value?

Use an explicit export, such as `let-data="result"`. This declares the local name `data` and reads the `result` value exported in the template context.

### 20. How are microsyntax keys mapped to directive inputs?

Angular PascalCases the key and prefixes it with the directive selector. For selector `select`, `from` maps to `selectFrom`. For legacy `ngFor`, the `of` key maps to `ngForOf`.

### 21. What does *select="source" mean for a directive with selector [select]?

A naked expression is bound to the input named after the directive prefix. Conceptually, `*select="source"` maps to an implicit template with `[select]="source"`.

### 22. What does an as alias do in structural microsyntax?

It gives a local template variable a convenient name for a value exported by the directive. For example, legacy `*ngIf="user as currentUser"` exposes the condition's value locally as `currentUser`.

### 23. Do all structural directives provide the same local variables?

No. A directive defines its own template context and which values it exports. Check that directive's API or documentation before relying on names such as `index`, `first`, or `$implicit`.

### 24. What happens to ordinary attributes on the element with an asterisk directive?

Angular moves the structural directive and its bindings to the implicit template. Other attributes and bindings remain associated with the original element inside that template.

### 25. Can I put two asterisk structural directives on one element?

No. Each shorthand requires an implicit `<ng-template>`, and Angular cannot infer which one should wrap the other. Nest the directives on separate layers or use built-in control flow.

### 26. How can ng-container help when I need nested structural behavior?

It provides a wrapper layer without adding a normal DOM element. You can put one structural directive on the `ng-container` and another on an element inside it.

## Modern control flow and compatibility

### 27. Which syntax should new Angular templates use for conditional rendering?

Use `@if` with optional `@else if` and `@else` blocks. It is built into the template language and does not require importing `NgIf`.

### 28. Which syntax should new Angular templates use for list rendering?

Use `@for` and provide a stable `track` expression, for example `@for (user of users; track user.id)`. Use `@empty` for a fallback when the collection is empty.

### 29. Which syntax should new templates use for multi-branch selection?

Use `@switch` with `@case` blocks and, when needed, an `@default` block.

### 30. Are legacy NgIf, NgFor, and NgSwitch syntax still useful to learn?

Yes, to read and maintain existing Angular applications and older examples. Angular's current API reference marks these directives deprecated since v20 and recommends `@if`, `@for`, and `@switch` for new templates.

### 31. Do the built-in control-flow blocks require CommonModule?

No. `@if`, `@for`, and `@switch` are template syntax and are available without importing `CommonModule` or the legacy directives.

### 32. Does *ngIf need to be imported into a standalone component?

Yes. Legacy directives such as `NgIf` must be in the component's template scope. Import the specific directive from `@angular/common` or use `CommonModule`. The same applies to other common directives such as `NgClass` and `NgStyle`.

### 33. Why do old tutorials import CommonModule for *ngIf and *ngFor?

Those are directives exported from `CommonModule`. Older examples often import the module to make common directives and pipes available. New control-flow blocks do not need that import, though other directives or pipes may.

### 34. What should I do when migrating old structural syntax?

Use Angular's current migration guide or schematic for the target project version, review the resulting changes, and verify behavior. Do not assume a mechanical conversion captures every custom structural directive or template-context use case.

## Practical use and troubleshooting

### 35. What is the difference between a directive selector and its input name?

The selector determines where Angular applies the directive. A directive can also expose inputs. For structural shorthand key expressions, Angular combines the selector prefix with a PascalCased key to find an input, such as `select` plus `from` becoming `selectFrom`.

### 36. Why might Angular say that ngClass is unknown?

The directive may not be in the component's template scope. In a standalone component, import `NgClass` or `CommonModule`. In a module-based component, ensure the module that exports the directive is imported by the declaring module.

### 37. Does ng-container create an accessible landmark?

No. It does not create a normal DOM element, so it cannot serve as a landmark or receive element-specific semantics. Use meaningful HTML such as `<section>` or `<ul>` when that structure is needed.

### 38. What is the difference between ng-container and ng-template?

`ng-container` is a grouping or insertion point that does not add a normal element. `ng-template` declares a fragment that is not rendered unless Angular instantiates it. They serve related but different template roles.

### 39. How should I decide whether to use a directive or a component?

Use a directive to apply reusable behavior to an existing host. Use a component when the feature owns markup, UI state, or a distinct public interface. Use structural behavior when the central job is to create or remove template views.

### 40. What quick checks help debug structural shorthand?

Confirm the directive is imported, inspect the directive selector and input names, check how each microsyntax key maps to an input, verify the exported context locals, and make sure there is only one asterisk structural directive on each element.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="023-template-references-ng-template-and-ngtemplateoutlet.md">023 - 23. Template references, ng-template, and ngTemplateOutlet</a></td><td><a href="../lessons/024-built-in-directives-and-structural-directive-syntax.md">Lesson</a></td><td><a href="../examples/024-built-in-directives-and-structural-directive-syntax.md">Example</a></td><td><a href="025-signal-inputs-required-inputs-and-input-transforms.md">025 - 25. Signal inputs, required inputs, and input transforms</a></td>
</tr>
  </tbody>
</table>
