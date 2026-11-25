# 30. Q&A: View queries and content queries

## 1. What is an Angular query?

A query finds matching child components, directives, template references, or values associated with elements in a component tree. Signal-based query functions expose their results as signals that update as matching elements change.

## 2. What is a view query?

A view query searches the component's own template. Use `viewChild()` to get one match or `viewChildren()` to get all matches.

## 3. What is a content query?

A content query searches content supplied between the receiving component's tags, including projected content. Use `contentChild()` for one match and `contentChildren()` for multiple matches.

## 4. How do I query one child component in my view?

Pass the component class to `viewChild()`:

~~~ts
readonly header = viewChild(CardHeader);
~~~

Read the query signal with `this.header()`.

## 5. How do I query multiple child components in my view?

Pass the component or directive class to `viewChildren()`:

~~~ts
readonly actions = viewChildren(CardAction);
~~~

The signal contains an array of matching instances, or an empty array if there are no matches.

## 6. How do I query one projected component?

Use `contentChild()` with its component or directive class:

~~~ts
readonly projectedToggle = contentChild(ToggleControl);
~~~

The query searches consumer-provided content.

## 7. How do I query multiple projected components?

Use `contentChildren()`:

~~~ts
readonly menuItems = contentChildren(MenuItem);
~~~

By default it finds direct children in the content tree.

## 8. What do query signals contain when no result is found?

Single-result queries such as `viewChild()` and `contentChild()` can return `undefined`. Multiple-result queries return an array, which is empty if no matches are present.

## 9. Do signal query results update when control flow changes?

Yes. Angular keeps query signals up to date when matching elements appear or disappear, for example when an `@if` condition changes.

## 10. When should I use a required query?

Use `viewChild.required()` or `contentChild.required()` when the component contract guarantees that one matching child is present. Angular reports an error if it cannot find the result, and the type no longer includes `undefined`.

## 11. Can a required query be used for multiple results?

There is no required form of `viewChildren()` or `contentChildren()`. A multiple-result query returns an array, including an empty array when there are no matches.

## 12. How do I use a template reference variable as a query locator?

Declare a reference in the template and pass its name as a string:

~~~html
<input #searchInput />
~~~

~~~ts
readonly searchInput = viewChild<ElementRef<HTMLInputElement>>('searchInput');
~~~

## 13. Can I use a CSS selector as a query locator?

No. Angular query locators are not CSS selectors. Use a component or directive type, a template reference variable name, or a supported provider token.

## 14. What does the `read` option do?

It tells Angular to retrieve a different value from the matched element's injector than the locator would normally return. Common values include `ElementRef` and `TemplateRef`.

## 15. How can I query an element reference?

Use the template reference name and set `read` to `ElementRef`:

~~~ts
readonly searchInput = viewChild('searchInput', {
  read: ElementRef,
});
~~~

Add a type argument when you want the native element type represented in TypeScript.

## 16. Should I use `ElementRef` for normal rendering changes?

Usually no. Prefer Angular bindings and templates for rendering. `ElementRef` is appropriate for browser operations that need a DOM reference, such as focusing an input.

## 17. Can a query use an injection token as its locator?

Yes. A provider token can locate an element that provides that token. This is an advanced option when querying by an exposed provider contract is more appropriate than querying by a concrete component class.

## 18. What is the default descendant behavior of `contentChild()`?

`contentChild()` traverses descendants in the declaring template by default. It can find a matching directive nested under another element or component host in that same template, subject to component boundaries.

## 19. What is the default descendant behavior of `contentChildren()`?

`contentChildren()` finds direct children by default. It does not traverse descendants unless configured to do so.

## 20. How do I make `contentChildren()` include descendants?

Pass `{ descendants: true }`:

~~~ts
readonly allItems = contentChildren(MenuItem, {
  descendants: true,
});
~~~

It still searches only within the same declaring template and does not pierce component templates.

## 21. Do view queries search descendants?

Yes. View queries search through the component's own view descendants. They still stop at component boundaries and do not inspect elements inside another component's template.

## 22. Can a parent query a button inside a child component's template?

No. The parent can query the child component instance if it appears in the parent's own view, but it cannot query the child's internal button. That internal template belongs to the child component.

## 23. Can a content query find a component that is nested inside projected content?

`contentChild()` traverses descendants by default. `contentChildren()` only finds direct children unless `{ descendants: true }` is set. Neither query crosses into another component's private template.

## 24. Do query functions return signals?

Yes. `viewChild`, `viewChildren`, `contentChild`, and `contentChildren` return signals. Call them to read current results, and use them in reactive contexts such as `computed()` where useful.

## 25. What are the decorator-based query APIs?

Existing code may use `@ViewChild`, `@ViewChildren`, `@ContentChild`, and `@ContentChildren`. They remain supported. Angular recommends signal-based query functions for new code.

## 26. When are decorator-based view queries available?

A `@ViewChild` result is available in `ngAfterViewInit`. Before then it may be undefined. Signal queries expose reactive results directly and do not require a lifecycle hook just to access the query signal.

## 27. When are decorator-based content queries available?

A `@ContentChild` result is available in `ngAfterContentInit`. Multiple decorator-based queries use a `QueryList`, which can be observed through its `changes` property.

## 28. Is `QueryList` the result type of `viewChildren()`?

No. The signal-based `viewChildren()` and `contentChildren()` return signals containing arrays. Decorator-based plural queries return `QueryList` objects.

## 29. How are queries related to content projection?

Content projection places consumer-provided markup at `<ng-content>` placeholders. Content queries can find matching Angular components or directives in that supplied content. A query does not change who owns the projected markup.

## 30. Can I query arbitrary text inside a component?

Queries are intended to find Angular elements, directive/component instances, template references, or injector values. Do not use them to inspect arbitrary rendered text or traverse the DOM structure.

## 31. Should I use a query to communicate with a child?

Use a query when the parent genuinely needs a child reference or a public child API. For ordinary state flow, pass values through inputs and report events through outputs. Queries do not replace those contracts.

## 32. Should I use a query to coordinate siblings?

Usually no. Keep shared state in their common parent and use inputs and outputs. A query can reference elements in a parent's view, but it should not become a hidden sibling communication system.

## 33. What happens if a queried element is inside an `@if` block?

The query can be absent while the condition is false and become available when the element is rendered. Use an optional query and handle `undefined`, or use a required query only if the match is guaranteed whenever it is read.

## 34. Do query functions support selecting by class or attribute?

Not as CSS query locators. If you need to find a specific element, add a template reference variable. If you need an Angular behavior contract, query for the directive or component that represents it.

## 35. What is the main difference between a query and an input?

An input is an explicit data contract from parent to child. A query retrieves a reference to something in a view or projected content. Prefer inputs when the child only needs a value; use a query when it needs a component, directive, template, or element reference.

## 36. Where can I read Angular's official query documentation?

See [Referencing component children with queries](https://angular.dev/guide/components/queries), [`viewChild()`](https://angular.dev/api/core/viewChild), [`contentChild()`](https://angular.dev/api/core/contentChild), [`viewChildren()`](https://angular.dev/api/core/viewChildren), and [`contentChildren()`](https://angular.dev/api/core/contentChildren).
