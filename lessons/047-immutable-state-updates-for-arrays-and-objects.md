# Immutable state updates for arrays and objects

JavaScript arrays and objects are mutable: a property can change while the value keeps the same reference. Angular signals and input bindings use identity to decide whether a value changed. If state is mutated in place and its reference is reused, Angular may not notify every consumer that depends on it.

An immutable update returns a new array or object for each changed level, while reusing unchanged values. This gives Angular a clear change signal and makes state transitions easier to inspect.

## Learning outcomes

After this lesson, you should be able to:

- Update an item in an array without mutating the original array.
- Add, remove, and reorder items with non-mutating patterns.
- Copy every level of a nested object path that changes.
- Use signal `set()` and `update()` without assuming `update()` makes mutation safe.
- Preserve unchanged item references and use stable keys in rendered lists.
- Understand the limits of TypeScript `readonly` and Angular read-only signals.
- Avoid unnecessary deep cloning and mutations hidden inside derivations.

## What an immutable update means

An immutable update does not modify the existing state value. It creates a replacement value representing the next state.

~~~ts
const nextUser = {
  ...user,
  displayName: 'Asha',
};
~~~

The old user remains unchanged. The new object has a different reference, and its unchanged properties are shared with the old object. This is a shallow copy.

Immutability is an update convention; JavaScript does not automatically make ordinary objects immutable. `const` prevents rebinding a variable, but it does not prevent changing the referenced array or object.

~~~ts
const names = ['Ada'];
names.push('Grace'); // The array contents change.
~~~

## Replace an item in an array

Use `map()` to create a new array and replace the matching item. Keep the original object reference for every item that did not change.

~~~ts
interface Task {
  readonly id: number;
  readonly title: string;
  readonly done: boolean;
}

function renameTask(
  tasks: readonly Task[],
  taskId: number,
  title: string,
): Task[] {
  return tasks.map((task) =>
    task.id === taskId ? { ...task, title } : task,
  );
}
~~~

The returned array is new. The renamed task is also new. Other tasks keep their references. If IDs are unique, exactly one item changes. If the ID is absent, this implementation still returns a new array; a state owner can detect that case and return the original array to avoid a needless notification.

Use a stable identity such as an ID to select an item. Avoid relying on an array index when items can be inserted, removed, or reordered.

## Common array transitions

| Intent | Mutating operation to avoid on stored state | Immutable transition |
| --- | --- | --- |
| Add to the end | `push(item)` | `[...items, item]` |
| Add to the start | `unshift(item)` | `[item, ...items]` |
| Remove matching items | `splice(index, 1)` | `items.filter(...)` |
| Replace matching item | `items[index] = nextItem` | `items.map(...)` |
| Sort for display | `items.sort(compare)` | `[...items].sort(compare)` |
| Reverse for display | `items.reverse()` | `[...items].reverse()` |

`sort()`, `reverse()`, `push()`, `splice()`, and similar methods mutate their receiver. Do not call them on an array held in state unless it is already a private temporary copy.

When adding an item, validate constraints such as unique IDs in the owner that accepts the transition. For removal, decide what should happen to selections or references to the removed item.

## Update a nested object

Object spread copies only one level. Copy the outer object and every nested object along the changed path:

~~~ts
interface Profile {
  readonly name: string;
  readonly contact: {
    readonly email: string;
    readonly city: string;
  };
}

function changeCity(profile: Profile, city: string): Profile {
  return {
    ...profile,
    contact: {
      ...profile.contact,
      city,
    },
  };
}
~~~

For a user stored in an array, copy the array, the matching user, and the changed nested objects:

~~~ts
const nextUsers = users.map((user) =>
  user.id === userId
    ? {
        ...user,
        profile: {
          ...user.profile,
          contact: {
            ...user.profile.contact,
            city: 'Pune',
          },
        },
      }
    : user,
);
~~~

A shallow copy of only the outer user would still share the old profile object. Mutating that nested object would also change the old state.

## Remove an object property

Destructure the property away to create an object that does not contain it:

~~~ts
const { temporaryToken, ...userWithoutToken } = user;
~~~

This is useful when “remove this optional field” is the intended transition. Setting a field to `undefined` is not identical in every context: the property still exists, and serialization or property-existence checks can distinguish the cases.

## Update Set and Map values

`Set` and `Map` instances are mutable too. Copy the collection before changing it:

~~~ts
this.#selectedIds.update((ids) => {
  const next = new Set(ids);
  next.add(id);
  return next;
});
~~~

~~~ts
this.#usersById.update((users) => {
  const next = new Map(users);
  next.set(user.id, user);
  return next;
});
~~~

The copy is shallow. If a value inside a map is an object and that object changes, replace that object as well. Consider whether the data structure and update cost fit the application.

## Use signals without mutating their values

Use `set()` when you already have a replacement value, and `update()` when the next value is calculated from the current one:

~~~ts
readonly #tasks = signal<readonly Task[]>([]);

addTask(task: Task): void {
  this.#tasks.update((tasks) => [...tasks, task]);
}

renameTask(taskId: number, title: string): void {
  this.#tasks.update((tasks) =>
    tasks.map((task) =>
      task.id === taskId ? { ...task, title } : task,
    ),
  );
}
~~~

`update()` is not a mutation helper. This still mutates the existing array and returns the same reference:

~~~ts
this.#tasks.update((tasks) => {
  (tasks as Task[]).push(newTask);
  return tasks;
});
~~~

Signals use referential equality by default. Returning the same reference means Angular sees no new array value. A custom equality function can change notification behavior, but it does not make mutation safe or keep other references consistent.

## Keep changes pure and centralized

An update function should calculate and return the next value. Avoid changing unrelated variables, editing another object in place, or performing network requests inside the updater.

~~~ts
this.#tasks.update((tasks) =>
  tasks.map((task) =>
    task.id === taskId ? { ...task, done: true } : task,
  ),
);
~~~

Put validation and related state changes in the state owner. For example, the owner can reject a duplicate item ID or clear the selected ID when removing the selected item. Components should request a transition instead of each implementing its own copy of the rules.

## Preserve references for unchanged data

Immutable does not mean “deep-copy everything.” Structural sharing keeps the identity of values that did not change:

- Replace the collection when its membership or an item changes.
- Replace the target item when one of its fields changes.
- Replace nested objects only along the path that changed.
- Reuse other items and nested objects.

This makes change boundaries precise and avoids unnecessary object creation. Deep-cloning an entire state tree for one field edit is usually wasteful and can discard useful identity. It can also fail or behave differently for values that do not fit the chosen cloning method.

## Derive views instead of mutating state for presentation

Do not sort the stored array in place just to display it. Create a sorted view:

~~~ts
readonly alphabetizedTasks = computed(() =>
  [...this.tasks()].sort((a, b) => a.title.localeCompare(b.title)),
);
~~~

Likewise, derive filtered lists and counts from their source state. Presentation should not change the canonical order or content unless reordering is itself a user action. If the user intentionally changes order, represent that as a state transition and return a replacement array.

## Track rendered items by stable identity

When rendering a collection with `@for`, use a stable unique key:

~~~html
@for (task of tasks(); track task.id) {
  <app-task-row [task]="task" />
}
~~~

Stable tracking tells Angular which rendered item corresponds to which data item when the collection changes. Immutable references and tracking keys serve related but distinct purposes: the key preserves the identity of the rendered row, while a new object reference communicates that the row's input data changed.

## TypeScript readonly is not runtime freezing

Marking a property or array as `readonly` prevents certain writes during TypeScript type-checking. It is not a runtime freeze, and it is shallow unless nested types also declare their properties as readonly.

An Angular read-only signal prevents a consumer from calling `set()` or `update()` on that signal. It does not deeply freeze an object or array returned by the signal. Keep writable references private, expose read-only types where useful, and follow immutable update conventions at the state boundary.

`Object.freeze()` is also shallow by default. Freezing data can help catch some accidental writes in development, but it does not replace well-defined state transitions.

## Common mistakes

- Calling `push()` or `splice()` on an array stored in a signal.
- Mutating a nested property after copying only the outer object.
- Treating `const`, `readonly`, or `asReadonly()` as deep immutability.
- Using `sort()` directly on the source array while building a display view.
- Deep-cloning everything for a one-field change and losing stable references.
- Updating a list by array index when items have stable IDs.
- Assuming `signal.update()` makes a mutating callback safe.
- Putting network or other side effects inside a state updater.
- Replacing every item when only one item changed.

## Practical checklist

Before committing a state transition, check:

1. Does the stored collection get a new reference when its content changes?
2. Does each changed object along the nested path get replaced?
3. Do unchanged items keep their existing references?
4. Is the update selected by stable identity?
5. Are related invariants, such as selection after deletion, handled by the owner?
6. Is any mutating operation applied only to a temporary copy?
7. Is the update function free of unrelated side effects?

## Key takeaways

- Make a replacement array or object instead of changing the stored value in place.
- Copy every level along a changed path; object spread is shallow.
- Use `map()` for replacement, `filter()` for removal, and spread for additions.
- Clone before using mutating sort or reverse operations for a view.
- Angular's signal update API notifies based on the value it receives; it does not enforce immutability.
- Preserve references for unchanged items instead of deep-cloning the entire state.
- TypeScript `readonly` and Angular read-only signals are useful boundaries, not runtime deep freezes.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="046-state-ownership-and-avoiding-duplicated-state.md">046 - State ownership and avoiding duplicated state</a></td><td><a href="../qnas/047-immutable-state-updates-for-arrays-and-objects.md">Q&amp;A</a></td><td><a href="../examples/047-immutable-state-updates-for-arrays-and-objects.md">Example</a></td><td><a href="048-inspect-signals-and-components-with-angular-devtools.md">048 - Inspect signals and components with Angular DevTools</a></td>
</tr>
  </tbody>
</table>
