# Immutable state updates for arrays and objects — Q&A

## 1. What is an immutable update?

It creates a new value for the part of state that changed instead of modifying the existing value in place. Unchanged values can keep their references.

## 2. Does JavaScript make objects immutable when I declare them with `const`?

No. `const` prevents assigning a different reference to the variable. The array or object referenced by that variable can still be mutated.

## 3. Why do immutable updates matter in Angular?

Angular signals use equality to decide whether a value changed, and input bindings compare values by identity. Replacing changed values makes updates observable to signal consumers and component inputs.

## 4. Does `signal.update()` automatically make an update immutable?

No. `update()` calls a function with the current value and stores the function's result. If the function mutates the current value and returns the same reference, the stored reference did not change.

## 5. What is the difference between `set()` and `update()`?

Use `set(nextValue)` when you already have the complete replacement. Use `update(current => nextValue)` when constructing the replacement from the current value. Both are valid when the value you store follows the state owner's update rules.

## 6. How do I add an item to an array immutably?

Use a new array, for example `[...items, newItem]`. For a signal whose next value depends on its current value, `itemsSignal.update(items => [...items, newItem])` expresses the transition.

## 7. How do I remove an item immutably?

Use `filter()` to produce a new array containing the items that remain. If removal affects selection or related state, let the state owner handle that rule too.

## 8. How do I replace one item in an array?

Use `map()`. Return a replacement object for the item with the matching stable ID and return each other item unchanged.

## 9. Why prefer IDs over array indexes for updates?

An index refers to a position, not an entity. Inserting, removing, sorting, or filtering can change positions. A stable ID continues to identify the intended item.

## 10. How do I update a nested object immutably?

Copy the outer object and every nested object along the path that changes. For example, updating a user's city requires a new users array, a new target user, a new profile, and a new address object.

## 11. Is object spread a deep copy?

No. Spread copies only the top level. Nested objects remain shared references unless they are explicitly copied.

## 12. Is it enough to copy an outer object and mutate a nested field?

No. The nested object is still shared with the previous value. Mutating it also changes the old state. Copy the nested object before changing its property.

## 13. Should I deep-clone the whole state for every edit?

Usually not. It creates new references even for unchanged data, can be expensive, and can discard useful identity. Copy only the collection and objects along the changed path.

## 14. What is structural sharing?

Structural sharing means retaining references to the parts of state that did not change while replacing only the changed collection and path. It keeps updates immutable without recreating the entire state tree.

## 15. Does returning a new array mean every child row must be recreated?

No. The array reference changes, but unchanged item objects can keep their references. A stable track key also helps Angular associate each rendered row with the same logical item.

## 16. Do immutable references and track item.id do the same job?

No. A stable tracking key identifies which rendered row corresponds to an item. A replacement object reference indicates that the item's data changed. They complement each other.

## 17. Why is array.sort() risky on state?

sort() mutates the array it receives. Sorting an array held in state changes that state in place. For a display-only order, sort a copy, such as [...items].sort(compare).

## 18. Does reverse() mutate its array?

Yes. Like sort(), reverse() mutates the receiver. Copy the array first if the reversal is only for presentation.

## 19. Is filter() safe for an immutable removal?

filter() returns a new array and does not remove items from the original array. It is a suitable operation for immutable removal.

## 20. What about Set and Map?

They are mutable objects. Create a new Set or Map from the current one before adding, removing, or replacing entries. If a stored map value is an object that changes, replace that object too.

## 21. Can I expose a read-only signal and still have mutable state?

Yes. A read-only signal prevents consumers from calling set() and update() on the signal. It does not deeply freeze an array or object returned by the signal. Keep writable references private and avoid mutating exposed values.

## 22. Does TypeScript readonly freeze a value at runtime?

No. It restricts writes during type-checking. It does not add runtime freezing, and a readonly property containing a mutable nested object does not automatically make that nested object readonly.

## 23. How can I express readonly state in types?

Use readonly on object properties and readonly T[] or ReadonlyArray<T> for arrays. Declare nested properties readonly too when callers should not change them. This helps at compile time but does not replace immutable update logic.

## 24. What does Object.freeze() do?

It prevents certain modifications to an object at runtime, but it is shallow by default. Freezing the outer object does not recursively freeze nested objects. It can be a development aid, but it is not a substitute for clear state ownership.

## 25. What does Angular's default signal equality compare?

Angular's default signal equality uses Object.is(). A replacement object or array normally has a different identity and notifies consumers. Setting the same reference again normally does not.

## 26. Should I use a deep equality function for signals instead of immutable updates?

Usually not as a way to compensate for mutation. Deep comparisons add work, can hide unclear state transitions, and do not prevent shared references from being changed. Use equality functions only when the value's semantics and performance needs justify them.

## 27. Can a mutation appear to work in a component?

Yes. An event handled by Angular can cause the event's component subtree to be checked, so the mutated value may appear in that component. A sibling OnPush child or a signal consumer might still not be notified. Apparent success in one view does not make in-place mutation a reliable state strategy.

## 28. Can I mutate a newly created temporary array?

Yes, if it is an independent temporary value that does not alias stored state and you do not expose it as an old state's value. For example, sorting [...items] in place is safe because the array passed to sort() is a new copy.

## 29. Can I update state with Object.assign()?

Object.assign(target, source) mutates its target. It is safe only when the target is a fresh object that does not alias stored state. Object spread is often clearer for creating a replacement.

## 30. Should update callbacks perform network requests?

No. Keep signal update functions focused on calculating the next state. Perform asynchronous work in an explicit method or data-access layer, then apply the resulting state transition.

## 31. What if the value did not actually change?

A state owner can return the original array or object when an update is a no-op. With default referential equality, this avoids notifying consumers for an unchanged value. Do this when it improves correctness or avoids meaningful unnecessary work; do not add complex checks without a need.

## 32. What happens when an item ID is missing during an update?

Usually return the current state unchanged. Define this behavior consistently, and consider whether a missing ID is expected, an error, or a stale event that can safely be ignored.

## 33. How should adding an item enforce unique IDs?

The state owner should validate or generate the ID before adding the item. A UI component should not be the only place enforcing an invariant that other callers can bypass.

## 34. Is it okay to change the array order in place if the user clicked “move up”?

Represent the reorder as an explicit state transition that returns a replacement array. In this case the order is canonical state, not just a presentation view. Keep the item objects unchanged if their own fields did not change.

## 35. How do immutable updates help debugging?

A transition produces a clear before-and-after value. Logs, devtools, tests, and memoized computations can distinguish which references changed, and the owner is a natural place to inspect the rule.

## 36. What previous Angular lesson is related to this one?

Chapter 044 explains how immutable references relate to OnPush input checks and rendering. This chapter focuses on the mechanics of replacing array and nested object state correctly, including signal behavior and reference preservation.

## 37. Which official references explain these details?

- [Angular signals](https://angular.dev/guide/signals)
- [WritableSignal API](https://angular.dev/api/core/WritableSignal)
- [TypeScript object types and readonly properties](https://www.typescriptlang.org/docs/handbook/2/objects.html)
- [TypeScript spread syntax](https://www.typescriptlang.org/docs/handbook/variable-declarations.html#spread)
