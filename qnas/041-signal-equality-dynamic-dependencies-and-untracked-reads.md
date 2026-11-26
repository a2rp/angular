# 41. Signal equality, dynamic dependencies, and untracked reads - Q&A

## Equality and updates

### 1. What does Angular use to decide whether a signal value changed?

By default, signals use `Object.is(previous, next)`. If it returns `true`, Angular considers the proposed value equal and does not notify downstream consumers.

### 2. How does `Object.is()` compare primitives and objects?

Primitive values are compared according to `Object.is()` semantics. Objects and arrays are compared by identity: two separately created objects with identical fields are still different references.

~~~ts
Object.is(3, 3); // true
Object.is({ id: 1 }, { id: 1 }); // false: distinct objects
~~~

### 3. What happens when I set a signal to its current primitive value?

The default equality check says it is unchanged, so consumers do not need to be notified.

### 4. Why can mutating an object and setting it back fail to update the UI?

The signal still holds the same object reference. `Object.is(oldObject, sameObject)` is true, so the update is treated as unchanged. Create and set a new object or array instead.

~~~ts
settings.update((old) => ({ ...old, theme: 'dark' }));
~~~

### 5. Does `asReadonly()` make an object inside a signal deeply immutable?

No. It removes the writable signal methods from that reference, but callers can still mutate an object returned by the signal. Use immutable values and keep mutation ownership controlled.

### 6. What is a custom equality function?

It is a comparator passed through an `equal` option. It receives the previous and proposed values and returns `true` when they should be treated as equivalent. That can prevent a signal update from notifying dependents.

### 7. Can writable and computed signals both use custom equality?

Yes. Pass `{ equal: comparator }` to `signal(initialValue, options)` or `computed(derive, options)`. Other signal APIs that accept creation options may also support equality; check the API for the Angular version in use.

### 8. When might custom equality be useful?

When a new value has a different identity but is semantically equivalent for every consumer, and avoiding downstream work is worth the comparator cost. For example, a small normalized value may be recalculated to the same string after a formatting-only input change.

### 9. What should a custom comparator include?

Every field that any downstream consumer cares about. If it compares only an object's ID but consumers display its name, a name change may be incorrectly suppressed. A comparator is a semantic contract for all dependents, not just an optimization local to one component.

### 10. Should custom equality use deep comparison by default?

No. Deep comparisons can cost more than the updates they avoid, can become fragile as a model grows, and can hide meaningful differences if implemented incompletely. Start with `Object.is()` and add custom equality only for a clear, measured need.

### 11. Should an equality function have side effects or read signals?

No. Keep it deterministic, pure, and inexpensive. It should compare the two arguments, not perform application work or establish reactive dependencies.

### 12. What does equality mean for a computed signal?

After a dependency invalidates a computed, Angular may reevaluate it when it is next read. The computed's equality function compares its prior result with the new result. If they are equivalent, downstream consumers need not be notified about a changed result.

### 13. What can go wrong if an equality function ignores a changed field?

Angular may treat the proposed value as equivalent and suppress propagation, so consumers do not react to that field's change. Define equality around all observable meaning, not merely object identity or a convenient subset of properties.

## Dynamic dependencies

### 14. What is a dynamic dependency?

It is a signal that becomes a dependency only because the current evaluation actually read it. Dependencies are collected from the latest execution of a reactive consumer, including `computed()` derivations and `effect()` callbacks.

### 15. What happens to a signal read only in a conditional branch?

It is tracked only while the evaluation takes that branch. If the branch is not taken on a later run, the signal is removed from the consumer's dependency set.

### 16. If a signal is not currently a dependency, will changing it rerun the computed?

No. A consumer can only be invalidated by a dependency it currently tracks. In the hidden-count example, changing `count` does not invalidate `label` while `showCount` is false.

### 17. How can an inactive signal become a dependency again?

Change a currently tracked condition so that a later evaluation takes the branch that reads the signal. The new evaluation updates the dependency set to include it.

### 18. Are computed signals eager?

No. Computed signals are lazy and memoized. Their derivation runs when read and when the cached value has been invalidated. An invalidated computed does not necessarily recompute immediately; it recalculates when something next reads it.

### 19. Can dependencies be added and removed in one signal graph over time?

Yes. Each evaluation replaces the consumer's old dependency set with the signals read during its latest run. Conditional code can change the graph as application state changes.

### 20. If a computation contains `if (condition()) return a(); return b();`, what does it track?

It always tracks `condition()`, and it tracks either `a()` or `b()` depending on the branch evaluated most recently. The inactive branch signal is not a dependency.

## `untracked()`

### 21. What does `untracked()` do?

It runs a callback in a non-tracking context and returns its result. Signals read inside that callback still provide their current values, but those reads are not registered as dependencies of the surrounding reactive computation.

### 22. Does `untracked()` cache, freeze, or change the signal's value?

No. It only suppresses dependency tracking for reads inside its callback. It returns the value observed during that call. A later invocation can read a newer value.

### 23. Why would I call `untracked()` from an effect?

To prevent incidental signal reads from triggering the effect. For example, the effect may need to run when `currentUser` changes, while a logging helper happens to read a diagnostic counter that should not cause a rerun.

### 24. Can `untracked()` be used inside `computed()`?

Yes, but be careful. A signal read only through `untracked()` will not invalidate the computed. Use it only when that value is incidental and the computed is allowed to remain unchanged when it changes.

### 25. What should I do if a value should trigger reevaluation?

Read it normally in the current synchronous reactive evaluation. Do not wrap that read in `untracked()`.

### 26. What if a helper or third-party function reads signals internally?

If those reads should not become dependencies of the caller's effect or computed, call the helper inside `untracked()`. If they should cause reevaluation, let them happen in the tracked context or pass required values in explicitly.

### 27. Why might an effect stop responding after I added `untracked()`?

A value that used to be tracked may now be read only inside the untracked callback. Changes to it no longer invalidate the effect. Move the intended dependency read outside `untracked()`.

## Debugging decisions

### 28. A signal changed, but my computed or effect did not run. What should I inspect?

Check both parts of reactivity:

1. **Equality:** did the signal consider the proposed value changed?
2. **Dependencies:** did the consumer read that signal during its most recent synchronous evaluation?

A custom comparator can suppress an update, while a branch or `untracked()` can leave the signal outside the current dependency set.

### 29. My list displays stale data after updating one item. What is a likely cause?

The original array or item may have been mutated in place and then set back with the same identity. Create a new array and new changed object, for example:

~~~ts
items.update((current) =>
  current.map((item) =>
    item.id === updated.id ? { ...item, ...updated } : item,
  ),
);
~~~

### 30. What is a practical rule for using equality and `untracked()`?

Use the default equality and normal tracked reads unless you can explain precisely why a difference or dependency should be ignored. Equality decides whether a proposed value counts as changed; `untracked()` decides whether a read can invalidate the current consumer.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="040-effects-cleanup-and-when-not-to-use-effects.md">040 - 40. Effects, cleanup, and when not to use effects</a></td><td><a href="../lessons/041-signal-equality-dynamic-dependencies-and-untracked-reads.md">Lesson</a></td><td><a href="../examples/041-signal-equality-dynamic-dependencies-and-untracked-reads.md">Example</a></td><td><a href="042-rxjs-interop-with-tosignal-and-toobservable.md">042 - 42. RxJS interop with `toSignal()` and `toObservable()`</a></td>
</tr>
  </tbody>
</table>
