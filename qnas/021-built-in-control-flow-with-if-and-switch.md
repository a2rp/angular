# Q&A: Built-in Control Flow with if and switch

## @if and its branches

### 1. What does @if do?

It conditionally renders its block when the Angular template expression is truthy.

### 2. What happens when an @if condition is false?

Angular does not render that block's embedded view. If there is no matching alternative branch, the conditional renders no content.

### 3. How do I provide a fallback branch?

Use @else:

~~~html
@if (isReady) {
  <p>Ready</p>
} @else {
  <p>Not ready</p>
}
~~~

### 4. Can I test several conditions?

Yes. Add any number of @else if branches before the optional final @else. Angular checks them in order and renders the first matching branch.

### 5. Does @if only accept boolean values?

No. It follows JavaScript-like truthiness. Values such as false, null, undefined, zero, an empty string, and NaN are falsy.

### 6. Why can @if (count) hide a zero count?

Because zero is falsy. If zero should be displayed, test the intended condition explicitly, for example @if (count >= 0).

### 7. How should I check that a value is present?

Use a condition that reflects the value's type and meaning. For a nullable object, an object truthiness check may work; for numeric or string data, compare explicitly when a falsy value is valid.

### 8. Can I use a ternary expression instead of @if?

A ternary can select an inline value, such as a short label. Use @if when the choice controls a block of markup or several elements.

### 9. Can I reuse the result of an @if condition?

Yes. Add an as alias:

~~~html
@if (account.profile; as profile) {
  <p>{{ profile.name }}</p>
}
~~~

The alias is available inside the block and its nested content.

### 10. Is the as alias available after the block?

No. It is scoped to the block and its descendants. A template variable declared in a nested view cannot be read from its parent view.

### 11. Does @if only hide an element with CSS?

No. Angular creates and removes the conditional embedded view. The content is not merely left in the DOM with display: none.

### 12. Is state inside a removed @if view preserved?

Do not rely on its local DOM or child component instance state remaining. When the branch is removed, its view and child components are destroyed; when it is shown again, Angular renders the branch again. Keep durable state in the component or an appropriate service.

### 13. Can I nest @if blocks?

Yes. Nested blocks create nested template scopes. Keep nesting readable and move complicated decisions into component code when that improves clarity.

### 14. Does @if require importing NgIf?

No. The built-in block syntax is part of Angular templates and does not require NgIf or CommonModule. Existing *ngIf directive templates can still be used when their directive is available.

## @switch and case matching

### 15. When should I use @switch?

Use it when one value chooses among several alternatives, such as a loading state, user role, or page mode. Use @if for ranges and unrelated boolean conditions.

### 16. How do I write a switch block?

~~~html
@switch (status) {
  @case ('ready') {
    <p>Ready</p>
  }
  @default {
    <p>Waiting</p>
  }
}
~~~

### 17. Does @switch compare values loosely?

No. Angular compares the switch expression and each case using strict equality, equivalent to ===.

### 18. Will numeric 1 match string '1'?

No. They have different types and are not strictly equal. Make sure the model value and case expressions use the same type.

### 19. Does @switch fall through to the next case?

No. It has no fall-through. A matching case renders its block without needing break.

### 20. Can multiple cases share one block?

Yes. Put consecutive @case labels before the shared block:

~~~html
@case ('editor')
@case ('reviewer') {
  <p>Content access</p>
}
~~~

### 21. Is @default required?

No. It is optional. If no case matches and there is no default block, Angular renders nothing for the switch.

### 22. What should @default show?

Show a truthful fallback for unrecognized or remaining states. Do not show a success view for an unknown status.

### 23. Can I use a function call as the switch expression?

Yes, but exhaustive type checking depends on TypeScript narrowing and works best with a typed variable. A function call or signal expression may prevent Angular from proving that every union case is handled.

### 24. What is @default never;?

It is a current Angular control-flow form that asserts the preceding cases cover all possible values. The block is written without a body:

~~~html
@default never;
~~~

If a typed union later gains an unhandled member, Angular's template type checker can report a missing case.

### 25. When does exhaustive switch checking work?

It relies on TypeScript type narrowing. The switched value should have a finite union type and be a variable the template checker can narrow. Check the project's Angular compiler support.

### 26. How can I use exhaustive checking with a signal?

Read the signal into a template local and switch on that local where supported:

~~~html
@let currentState = state();
@switch (currentState) {
  @case ('ready') {
    <p>Ready</p>
  }
  @default never;
}
~~~

The local still needs a type that describes the full union, and every possible member needs a case.

## Choosing and migrating syntax

### 27. Is @switch a replacement for every @if chain?

No. Use the syntax that best represents the logic. If branches test different predicates or ranges, @if and @else if are usually clearer.

### 28. Can @if and @switch be used in the same template?

Yes. Each block handles the conditional decision that fits it. They can also be nested when that remains understandable.

### 29. Do built-in blocks need CommonModule?

No. Built-in control flow is Angular template syntax. CommonModule may still be required for other directives or pipes used in the template.

### 30. Can old *ngIf and *ngSwitch templates remain?

Yes. Existing structural directive syntax can continue to work with the required imports. New built-in blocks do not force an immediate rewrite.

### 31. Is the built-in syntax exactly identical to the old directives?

No. The concepts are similar, but the block syntax has different syntax and template scoping. Verify branch behavior and local-variable scope when migrating.

### 32. How do I migrate structural control flow?

Use Angular's control-flow migration schematic or update templates carefully, then review the result. Preserve the current imports if other directives still need them.

### 33. Can I place declarations inside an @if block and use them outside?

No. Locals declared within a block belong to its nested view and are not available to its parent or sibling views.

### 34. Do the blocks render their contents as HTML strings?

No. Their contents remain Angular template markup and are compiled as part of the component template.

### 35. Does Angular evaluate every branch in an @else if chain?

Angular checks the conditions in order and renders the first matching branch. Put specific conditions before broader conditions so the intended branch wins.

### 36. What is a common bug with a broad @else?

A broad fallback can hide an unexpected or newly added state. For finite union types, exhaustive checking can catch missing cases; otherwise make the fallback's meaning clear.

### 37. Can @switch use object cases?

It compares by strict equality. Two separately created object literals are not equal just because their fields match. Prefer primitive discriminants such as a status string or use @if for property-based tests.

### 38. What should I check if a branch never renders?

Check the runtime value and its type, whether the expression is truthy, whether an earlier branch matches, whether a switch case has the same type, and whether the relevant Angular version supports the syntax.

### 39. What is the main distinction between rendering and visibility here?

A false conditional removes its embedded view from the rendered template. CSS visibility keeps an element rendered and only changes presentation. Choose based on whether the content should exist while inactive.

### 40. What is the most important rule to remember?

Use @if for conditions and @switch for one value with named alternatives. Account for truthiness, use explicit comparisons for meaningful falsy values, and make unknown states intentional.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="020-two-way-binding-and-ngmodel.md">020 - 20. Two-way binding and ngModel</a></td><td><a href="../lessons/021-built-in-control-flow-with-if-and-switch.md">Lesson</a></td><td><a href="../examples/021-built-in-control-flow-with-if-and-switch.md">Example</a></td><td><a href="022-render-collections-with-for-track-and-empty.md">022 - 22. Render collections with for, track, and empty</a></td>
</tr>
  </tbody>
</table>
