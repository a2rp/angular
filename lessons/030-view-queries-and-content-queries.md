# 30. View queries and content queries

Angular queries let a component find children or values associated with elements in its component tree. Use a **view query** to find something declared in the component's own template. Use a **content query** to find something supplied between the component's tags by its consumer.

The signal-based query functions return signals that Angular keeps up to date as the matching elements appear or disappear:

~~~ts
import { Component, ElementRef, viewChild } from '@angular/core';

@Component({
  selector: 'app-search-panel',
  standalone: true,
  template: `
    <input #searchInput type="search" />
    <button type="button" (click)="focusSearch()">Focus search</button>
  `,
})
export class SearchPanel {
  readonly searchInput = viewChild<ElementRef<HTMLInputElement>>('searchInput');

  focusSearch(): void {
    this.searchInput()?.nativeElement.focus();
  }
}
~~~

The example query locates a template reference declared in the component's own view. A content query instead searches content supplied by the parent, such as projected child components or directives.

## Learning goals

After this lesson, you should be able to:

- Choose between a view query and a content query.
- Find one result with `viewChild()` or `contentChild()` and multiple results with `viewChildren()` or `contentChildren()`.
- Read query results as signals and handle optional or required results.
- Use component/directive types or template reference names as query locators.
- Configure what value is returned with the `read` option.
- Understand descendant defaults and component-boundary limits.
- Use queries for deliberate child references without replacing inputs and outputs as the normal communication pattern.

## View queries inspect the component's own template

The component's **view** is the markup in its own template. Use `viewChild()` to find one matching result:

~~~ts
readonly saveButton = viewChild(SaveButton);
~~~

Use `viewChildren()` when the component needs all matching results:

~~~ts
readonly actions = viewChildren(CardAction);
~~~

For example, a component can derive labels from queried child component instances:

~~~ts
readonly actionLabels = computed(() =>
  this.actions().map((action) => action.label()),
);
~~~

`viewChild()` returns a signal whose value may be `undefined` when nothing matches. This can happen when the target is behind an `@if` or has not been created. `viewChildren()` returns a signal containing an array of matches; the array is empty when there are none.

Angular updates query signals when application state changes the matching view. A query can therefore track elements added or removed by built-in control flow.

## Content queries inspect consumer-provided content

A component's **content** is the markup nested inside its element by the component that uses it. This includes projected content placed by `<ng-content>`.

Use `contentChild()` to find one matching result:

~~~ts
readonly projectedHeading = contentChild(ProjectedHeading);
~~~

Use `contentChildren()` to find multiple matching direct children:

~~~ts
readonly projectedActions = contentChildren(ProjectedAction);
~~~

Content queries are useful when a component's contract accepts specific child components or directives and needs to inspect or coordinate them. For example, a menu component may query its projected menu items.

`contentChild()` returns a signal whose value may be `undefined`. `contentChildren()` returns a signal containing an array, which is empty when there are no matches. Both are kept up to date as matching content changes.

## Required queries

If a single result must always exist, use a required query:

~~~ts
readonly header = viewChild.required(CardHeader);
readonly body = contentChild.required(CardBody);
~~~

A required query's type does not include `undefined`. Angular reports an error if it cannot find a match. Use required queries only when the component's template or usage contract guarantees the element is present. If it may be hidden by `@if` or omitted by the consumer, keep the query optional and handle the missing result.

There is no required form of `viewChildren()` or `contentChildren()`; a multiple-result query simply returns an empty array when it has no matches.

## Query locators

A query's first argument is its **locator**. Most often, use a component or directive class:

~~~ts
readonly panel = viewChild(DetailsPanel);
readonly projectedItems = contentChildren(MenuItem);
~~~

You can also use a string that matches a template reference variable:

~~~html
<input #searchInput type="search" />
~~~

~~~ts
readonly searchInput = viewChild<ElementRef<HTMLInputElement>>('searchInput');
~~~

If multiple elements use the same reference name, a single-result query gets the first matching result. Angular query locators are not CSS selectors; `viewChild('.save-button')` does not search by class. Use a component or directive type, a template reference variable, or a supported provider token.

## Read a different value from a match

By default, a query locates an element and reads the value associated with the locator. Use the `read` option when you want a different value from the matched element's injector:

~~~ts
readonly inputElement = viewChild('searchInput', {
  read: ElementRef,
});
~~~

Common `read` values include `ElementRef` for a DOM element and `TemplateRef` for a template fragment. Queries can also use provider tokens as locators when a directive or component provides a value you want to find.

Use DOM references only when an imperative browser operation is needed, such as moving keyboard focus. Prefer a component's public API for behavior and ordinary bindings for state. Avoid using `ElementRef` to make routine content or state changes that Angular bindings can express.

## Descendants and component boundaries

View queries search through the component's own view descendants. They do not cross into another component's template. A parent can query the child component instance placed in its own template, but it cannot query elements inside that child's private template.

Content query defaults differ by query type:

| Query | Default search |
| --- | --- |
| `contentChild()` | Traverses descendants in the declaring template |
| `contentChildren()` | Matches direct children only |

Set `{ descendants: true }` on `contentChildren()` when it should include nested descendants in the same declaring template:

~~~ts
readonly allNestedItems = contentChildren(MenuItem, {
  descendants: true,
});
~~~

Even with `descendants: true`, a content query does not pierce through another component's template. It searches the consumer-provided template tree, not private implementation details of nested components.

## Queries and projected content

A content query complements content projection. `<ng-content>` determines where supplied markup renders; a content query finds matching component or directive instances in that supplied content. The query does not change who owns that markup: projected content remains part of the consumer's view.

Do not use a query to discover arbitrary text or inspect the rendered DOM structure. Queries are designed to find Angular elements and values associated with their injectors, such as component instances, directives, template references, or element references.

## Signal queries and decorator-based queries

Angular recommends signal-based query functions for new code:

- `viewChild()` and `viewChildren()` inspect the component's view.
- `contentChild()` and `contentChildren()` inspect consumer-provided content.

Existing code may use the supported decorators `@ViewChild`, `@ViewChildren`, `@ContentChild`, and `@ContentChildren`. Decorator-based queries remain supported. Their results are generally accessed in the corresponding lifecycle hooks: view query results in `ngAfterViewInit`, and content query results in `ngAfterContentInit`. Multiple decorator-based queries expose a `QueryList`.

Do not mix query APIs just for stylistic reasons. Follow the surrounding code when maintaining an existing component, and use signal queries when writing new components unless project constraints call for the decorator API.

## Queries do not replace component communication

Use inputs to provide data and outputs to report events. A query gives a component a reference to a matching child or projected item; it does not automatically establish a state flow or make sibling components communicate.

Reach for a query when a component genuinely needs to coordinate a child API, measure or focus an element, or inspect specific projected children. If the goal is simply to pass a value or respond to an action, regular bindings are clearer.

## Common mistakes

- **Using a view query for projected content:** use `contentChild()` or `contentChildren()` for consumer-supplied content.
- **Using a content query for a component's own template element:** use `viewChild()` or `viewChildren()` for the component's view.
- **Expecting a query to cross component boundaries:** queries do not inspect another component's internal template.
- **Assuming `contentChildren()` includes nested descendants:** it finds direct children by default; opt in with `{ descendants: true }` when needed.
- **Using CSS selectors as query locators:** query by type, template reference name, or provider token.
- **Treating an optional result as always present:** account for `undefined`, especially when a target is conditional.
- **Marking a query required when projected content is optional:** use a required query only when a match is guaranteed.
- **Using queries instead of inputs and outputs:** use queries for references and coordination, not as a hidden data communication channel.
- **Manipulating the DOM for ordinary rendering:** prefer Angular templates and bindings; use `ElementRef` only for appropriate imperative operations.

## Practical checklist

Before adding a query, ask:

- Is the target in my own template or supplied as content by my consumer?
- Do I need one match or all matches?
- Can the result be absent due to conditional rendering or optional content?
- Is a component/directive type or template reference a better locator than a provider token?
- Do I need the matched instance or another value via `read`?
- Should `contentChildren()` search direct children only or descendants too?
- Am I staying within the component boundary?
- Would an input, output, or ordinary template binding communicate the need more clearly?

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="029-content-projection-and-multiple-slots.md">029 - 29. Content projection and multiple slots</a></td><td><a href="../qnas/030-view-queries-and-content-queries.md">Q&amp;A</a></td><td><a href="../examples/030-view-queries-and-content-queries.md">Example</a></td><td><a href="031-lifecycle-hooks-and-component-initialization.md">031 - 31. Lifecycle hooks and component initialization</a></td>
</tr>
  </tbody>
</table>
