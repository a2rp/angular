# 22. Render collections with for, track, and empty

Angular's @for block repeats template content for each item in a collection:

~~~html
@for (task of tasks; track task.id) {
  <li>{{ task.title }}</li>
}
~~~

The track expression tells Angular how each item corresponds to its rendered view. For dynamic lists, a stable unique identifier lets Angular update only the views needed when items are added, removed, reordered, or replaced.

## Learning goals

After this lesson, you should be able to:

- Render an iterable with @for.
- Choose a stable tracking key for each item.
- Use $index only when collection positions are stable.
- Show an empty state with @empty.
- Use the contextual variables for index, count, and position.
- Alias contextual variables for readable and nested loops.
- Recognize duplicate-key and unstable-identity problems.
- Understand how Angular reuses views as a collection changes.

## 1. Basic @for syntax

A for block has an item variable, the collection expression, and a required track expression:

~~~html
@for (item of items; track item.id) {
  <p>{{ item.name }}</p>
}
~~~

The item variable refers to the current collection item inside that block. Choose a clear variable name that describes the item.

The collection can be a JavaScript iterable. Angular has additional performance optimizations for arrays, so arrays are a common choice for UI collections.

Built-in @for is Angular template syntax. It does not require importing NgFor or CommonModule.

## 2. Track each item with a stable key

The track expression gives Angular a key it can use to associate an item with its rendered view:

~~~html
@for (user of users; track user.id) {
  <app-user-row [user]="user" />
}
~~~

For a collection that can change, use a stable, unique property such as id or uuid. The key should identify the same real-world item over time and should not change just because that item was edited.

Good tracking keys let Angular reuse the corresponding view and perform the minimum DOM work needed for changes.

## 3. Why $index is usually wrong for a changing list

$index is the current zero-based position of an item. It is a useful key for a collection that never changes:

~~~html
@for (label of fixedLabels; track $index) {
  <span>{{ label }}</span>
}
~~~

For a list that can be inserted into, deleted from, or reordered, indices shift. If Angular tracks by index, it may associate an existing view with a different logical item. This can move local DOM state, such as an input's focus or selection, to the wrong row.

For dynamic data, prefer a stable identity such as item.id.

## 4. Tracking by object identity

You can track the item itself:

~~~html
@for (item of items; track item) {
  <p>{{ item.name }}</p>
}
~~~

This uses object identity. It can be less effective when code frequently replaces items with new object instances, because Angular sees them as different identities even if their fields are equal. A unique stable ID usually communicates identity more reliably.

Avoid tracking by a value that is not unique, such as a display name that can repeat. Duplicate keys make it ambiguous which item should keep which view and can cause correctness and performance problems.

## 5. Show an empty state with @empty

Put an optional @empty block immediately after the repeated block:

~~~html
<ul>
  @for (task of tasks; track task.id) {
    <li>{{ task.title }}</li>
  } @empty {
    <li>No tasks have been added.</li>
  }
</ul>
~~~

Angular renders @empty when the collection has no items. Use it for a helpful empty state rather than rendering an empty list with no explanation.

The collection should be an iterable. If data can be null or undefined before loading completes, handle that state deliberately, for example by providing an empty array or conditionally rendering the loop.

## 6. Contextual variables

Angular exposes these local values inside every @for block:

| Variable | Meaning |
| --- | --- |
| $index | Zero-based index of the current item |
| $count | Number of items in the collection |
| $first | Whether this is the first item |
| $last | Whether this is the last item |
| $even | Whether the index is even |
| $odd | Whether the index is odd |

Use them for presentation such as numbering, first/last markers, or alternating classes:

~~~html
@for (item of items; track item.id) {
  <p>
    {{ $index + 1 }} of {{ $count }}:
    {{ item.name }}
  </p>
}
~~~

These values describe the current iteration. They are not collection state to mutate.

## 7. Alias contextual variables

Use the let segment to give contextual variables shorter or more descriptive names:

~~~html
@for (
  item of items;
  track item.id;
  let index = $index, first = $first, last = $last
) {
  <p>
    {{ index + 1 }}. {{ item.name }}
    @if (first) { <span>First</span> }
    @if (last) { <span>Last</span> }
  </p>
}
~~~

Aliases are especially useful when loops are nested so the inner and outer indices are easy to distinguish. They are local to the loop block and its nested content.

## 8. Track-expression scope and methods

The track expression can refer to the current item, $index, and component members. It is not an arbitrary separate callback declaration.

When calling a component method as part of a track expression, call it with the item:

~~~html
@for (item of items; track trackItem(item)) {
  <p>{{ item.name }}</p>
}
~~~

Prefer the direct stable key expression such as track item.id when possible. A method call should be simple, deterministic, and return the same key for the same item identity.

## 9. How Angular reuses views

When a collection changes, Angular compares item keys and reuses, moves, creates, or removes views to match the new collection.

If an item's tracked key stays the same, Angular can preserve that item's view while updating its bound data. This is useful when replacing an array immutably:

~~~ts
this.items = this.items.map((item) =>
  item.id === changedId ? { ...item, done: true } : item
);
~~~

The item has a new object reference but keeps the same id. Angular can associate it with the same rendered view and update its bindings.

If the key changes, Angular treats it as a different identity. If the track expression returns duplicates, Angular cannot reliably map items to their views.

## 10. Limitations and practical choices

- @for requires a track expression.
- It supports JavaScript iterables, with performance optimizations for arrays.
- It does not support JavaScript-style break or continue statements.
- It does not iterate an object's properties automatically; prepare an iterable in component code when needed.
- Track keys should be unique within the collection and stable across updates.
- Use $index for static collections whose ordering and membership do not change.
- Use @empty for the zero-items case; handle loading and error states separately.

## Common mistakes

- Forgetting the required track expression.
- Tracking a dynamic list by $index.
- Tracking by a label or value that can repeat.
- Generating a new random key during every render.
- Replacing stable IDs when only the item's content changed.
- Expecting @empty to handle a null or non-iterable collection.
- Using continue or break inside @for.
- Expecting @for to loop over object properties directly.
- Using contextual variables outside their loop scope.
- Ignoring duplicate-key warnings.

## Practical checklist

1. Is the collection a valid iterable?
2. Can items be inserted, removed, or reordered?
3. What stable, unique key identifies each item?
4. Is $index only being used for a static collection?
5. Is there a useful empty state?
6. Would index, count, first, last, even, or odd improve the presentation?
7. Are aliased values scoped to the correct loop?
8. Do immutable updates keep each item's stable key?

## Further reading

- [Angular control flow](https://angular.dev/guide/templates/control-flow)
- [Angular @for API](https://angular.dev/api/core/%40for)
- [Angular expression syntax](https://angular.dev/guide/templates/expression-syntax)
- [Angular control-flow migration](https://angular.dev/reference/migrations/control-flow)
- [Angular error NG0955: duplicate tracking keys](https://angular.dev/errors/NG0955)
- [Angular error NG0956: tracking causes DOM recreation](https://angular.dev/errors/NG0956)
