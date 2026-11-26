# Q&A: Render Collections with for, track, and empty

## @for basics

### 1. What does @for do?

It repeats a template block for each item in an iterable collection.

### 2. What is the basic syntax?

~~~html
@for (item of items; track item.id) {
  <p>{{ item.name }}</p>
}
~~~

The loop variable refers to one item at a time.

### 3. Is track optional?

No. The @for block requires a track expression so Angular can associate collection items with their rendered views.

### 4. What does the track expression do?

It gives Angular a key for identifying each item across collection changes. Angular uses these keys to decide which views to keep, move, create, update, or remove.

### 5. What makes a good tracking key?

It should be unique within the collection and stable for the lifetime of the logical item. A database ID or generated UUID is often suitable.

### 6. Why should I not track a dynamic list by $index?

When items are inserted, removed, or moved, indexes shift. Angular can then associate an existing view with a different item, which can move local DOM state such as focus or input selection to the wrong row.

### 7. When is track $index appropriate?

It is appropriate for a collection whose membership and order never change, such as a fixed list of labels.

### 8. Can I track by an item's display name?

Only if the value is guaranteed unique and stable. Names and labels often repeat or change, so a dedicated identity field is safer.

### 9. What happens when tracking keys are duplicated?

Angular cannot unambiguously associate each item with a view. Duplicate keys can cause incorrect view reuse and slower tracking. Angular can report this as NG0955.

### 10. What happens if the key changes for one item?

Angular sees a new identity for that item and may remove the old view and create a new one. Keep the key stable when only the item's content changed.

## Identity and collection updates

### 11. Can I track the object itself with track item?

Yes. This uses object reference identity. It can work when the same object instances remain in the collection, but replacing items with fresh objects can make Angular treat them as entirely new items.

### 12. Why is tracking by object identity often weaker than tracking by id?

Data refreshes commonly create new object instances for the same logical records. A stable ID lets Angular recognize those records even when object references change.

### 13. Does a stable key preserve a view when an item object is replaced?

Yes. If the key remains the same, Angular can reuse the existing view and update its bindings to reflect the replacement item.

### 14. Should tracking keys be random values created in the template?

No. A key must remain stable between renders. A newly generated random key makes every item look new and can force unnecessary DOM recreation.

### 15. Can I use trackBy like *ngFor?

The syntax is different. @for uses a track expression directly, such as track item.id, rather than the *ngFor trackBy input.

### 16. Can the track expression call a component method?

Yes, but call the method with the item, for example track getItemKey(item). Keep the method deterministic and prefer a direct stable property expression when possible.

### 17. What can a track expression refer to?

It can refer to the current item, $index, and component members. It can also use an alias for $index introduced by the loop's let segment.

### 18. Do I have to replace an array immutably?

No. @for can respond to collection changes. Immutable replacements are common in modern state patterns, and stable keys let Angular reuse the matching item views. Make sure the application state mechanism notifies Angular about the change.

### 19. What if every item is replaced and every ID changes?

Angular considers them new identities and may recreate the views. If the records are logically the same, preserve their IDs across refreshes.

### 20. What is NG0956?

It is a diagnostic for a tracking expression that causes broad DOM recreation. Review whether the key stays stable for items that are logically unchanged.

## Empty collections and context

### 21. What does @empty do?

It renders an optional fallback block when the iterable has no items.

### 22. Where does @empty go?

Immediately after the repeated block's closing brace:

~~~html
@for (item of items; track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <li>No items yet.</li>
}
~~~

### 23. Does @empty handle null or undefined?

It is for an empty iterable. A null or non-iterable value should be handled before it reaches @for, for example by using a loading condition or supplying an empty array.

### 24. Does @for iterate object properties?

No. It iterates an iterable. Convert an object's entries to an array in component code if that is the desired data source.

### 25. Can @for iterate a string?

Strings are JavaScript iterables, but repeating individual characters is rarely the intended UI. Provide an array of domain items for normal lists.

### 26. What does $index mean?

It is the current item's zero-based position in the collection.

### 27. What does $count mean?

It is the number of items in the collection being iterated.

### 28. What do $first and $last tell me?

They are booleans indicating whether the current item is first or last in that iteration.

### 29. What do $even and $odd tell me?

They are booleans indicating whether the current zero-based index is even or odd. They can support alternating presentation.

### 30. Can I rename contextual variables?

Yes. Add aliases in the let segment, for example let index = $index, first = $first. This can improve readability or prevent name collisions in nested loops.

### 31. Can an alias be used after the @for block?

No. Contextual variables and their aliases are scoped to that loop block and its nested content.

### 32. Are @for contextual variables values to mutate?

No. They describe the current iteration. Use them for display and logic; update the collection in component state instead.

## Behavior and limitations

### 33. Does @for support break and continue?

No. It does not support flow-modifying statements like JavaScript break or continue. Filter or transform the collection in component code, or use a condition inside the row when appropriate.

### 34. Does @for require importing NgFor or CommonModule?

No. Built-in control flow is part of Angular template syntax. Other directives and pipes in the same template may still require their imports.

### 35. What happens to a row's child component when its key stays stable?

Angular can reuse the associated view and update its bindings rather than destroying and recreating the row solely because the object reference changed.

### 36. Does track affect visual ordering?

The collection's order determines the rendered order. The track key helps Angular associate each item with its view as that order changes.

### 37. Does @empty replace a loading or error state?

No. It represents the zero-items case. Loading, error, and empty are different application states and should be modeled and rendered distinctly.

### 38. What should I do if I see a duplicate-key warning?

Find the duplicate key values and change the track expression to a truly unique item identity. Do not hide the warning by appending $index to a key for a dynamic list.

### 39. What is the safest default tracking strategy for a mutable list?

Use a stable, unique identifier from each item, such as track item.id.

### 40. What is the most important rule to remember?

Track logical identity, not changing position or display text. Use @empty for the no-items case, and use contextual variables only for presentation and iteration-specific logic.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="021-built-in-control-flow-with-if-and-switch.md">021 - 21. Built-in control flow with if and switch</a></td><td><a href="../lessons/022-render-collections-with-for-track-and-empty.md">Lesson</a></td><td><a href="../examples/022-render-collections-with-for-track-and-empty.md">Example</a></td><td><a href="023-template-references-ng-template-and-ngtemplateoutlet.md">023 - 23. Template references, ng-template, and ngTemplateOutlet</a></td>
</tr>
  </tbody>
</table>
