# State ownership and avoiding duplicated state

A component can read state owned by a parent, a service, a form, or the server. The key design question is not “where can this value be stored?” but “which part of the application is responsible for deciding that this value changes?”

When the same fact is stored in multiple places, those copies can disagree. A task may be renamed in the task list while a separately stored selected-task object still shows the old name. A filtered count may stop matching the filtered list. Clear ownership keeps one authoritative value and derives the views that depend on it.

## Learning outcomes

After this lesson, you should be able to:

- Choose an owner for local, shared, input, form, and server state.
- Distinguish canonical state from a derived view of that state.
- Use `computed()` for read-only derived values and `linkedSignal()` for dependent values that users can also change.
- Share state through a service with the correct Angular dependency-injection scope.
- Avoid effect-driven copies and stale object snapshots.
- Keep temporary UI and unsaved form drafts separate from saved application data.

## Start with the owner

For every state value, answer these questions:

1. Who is allowed to change it?
2. Which parts of the UI need to observe it?
3. How long should it live?
4. Is it an independent fact, or can it be calculated from another fact?
5. If its source changes, should the value reset, preserve a valid choice, or become empty?

The answers suggest a natural owner:

| State kind | Typical owner | Examples |
| --- | --- | --- |
| Temporary view state | The component using it | Open panel, current tab, local filter, hover state |
| Parent-controlled state | The parent component | Selected record passed to a child |
| Editable child value | The child API, intentionally connected to a parent | A reusable quantity control using a model input |
| Shared feature state | A feature or route-scoped service | Current checkout, editor document, feature filters |
| App-wide client state | A root-provided service | Current user session, global preferences |
| Server state | A data-access layer or feature service | Loaded records, request status, mutation results |
| Derived state | No separate writable owner | Counts, filtered lists, selected record, totals |

These are useful defaults, not rules that every application must follow. The right owner is the narrowest scope that contains every consumer that must share the same value.

## Keep one canonical value

Canonical state represents the facts the application needs to remember. A derived value is a view calculated from those facts.

Suppose a task board has an array of tasks. The task array is canonical state. The number of completed tasks, the visible tasks, and the selected task are derived views:

~~~ts
readonly tasks = signal<Task[]>(initialTasks);
readonly completedCount = computed(
  () => this.tasks().filter((task) => task.done).length,
);
~~~

Do not store `tasks`, `completedCount`, and another copy of the completed tasks as independently writable state unless they truly represent separate facts. If one can be calculated from the other, calculate it. This removes synchronization work and makes contradictory values harder to create.

The same principle applies outside signals. A server response may be the source of truth for a record, while a view model maps it into display-ready data. A form draft may intentionally differ from the last saved record; that difference is meaningful state, not an accidental duplicate.

## Store stable identity when an object can change

Storing an entire selected object beside the collection often creates a stale snapshot:

~~~ts
readonly tasks = signal<Task[]>(initialTasks);
readonly selectedTask = signal<Task | null>(initialTasks[0]);
~~~

If a task is updated in `tasks`, the separately stored object may still be the older object. Prefer storing its stable ID and deriving the current object:

~~~ts
readonly selectedTaskId = signal<number | null>(1);

readonly selectedTask = computed(() => {
  const id = this.selectedTaskId();
  return this.tasks().find((task) => task.id === id) ?? null;
});
~~~

Now an immutable task update creates a new task object in the collection, and `selectedTask()` finds that current object. A missing ID produces `null`, which the template can handle explicitly.

This pattern is especially useful for selected rows, routes to records, normalized entity collections, and references that must survive updates. It is not necessary for every small component: a local object can be perfectly reasonable when its lifetime and ownership are unambiguous.

## Choose `computed()` or `linkedSignal()`

Use `computed()` when a value is a read-only calculation. Its result follows its dependencies and should not be independently set:

~~~ts
readonly fullName = computed(
  () => this.firstName() + ' ' + this.lastName(),
);
~~~

Use `linkedSignal()` when a value is based on another value but must also be writable. For example, a list of shipping options can choose a default while still letting the user select a different option.

~~~ts
readonly selectedOptionId = linkedSignal(() =>
  this.options()[0]?.id ?? null,
);
~~~

When the options change, that computation supplies a new default. If you want to keep the user's current choice when it still exists, use the source-and-computation form and check the previous selection:

~~~ts
readonly selectedOptionId = linkedSignal<
  ShippingOption[],
  number | null
>({
  source: this.options,
  computation: (options, previous) =>
    options.some((option) => option.id === previous.value)
      ? previous.value
      : options[0]?.id ?? null,
});
~~~

The behavior is a product decision: reset to a default, preserve a still-valid choice, or clear the value. Make that rule explicit. Do not use an effect to copy a source signal into another signal to simulate this behavior.

## Keep effects at the boundary

Effects are for work that must happen outside Angular's signal graph, such as logging or synchronizing a value to an imperative browser or third-party API. They are usually not the right way to keep two pieces of application state aligned.

Avoid this pattern:

~~~ts
effect(() => {
  this.selectedTask.set(
    this.tasks().find((task) => task.id === this.selectedTaskId()) ?? null,
  );
});
~~~

It introduces a second writable copy and an asynchronous synchronization step. The values can briefly disagree, the effect can accidentally create a cycle, and the relationship is less visible than a direct derived signal. Model the relationship with `computed()` or `linkedSignal()` instead.

## Scope shared state with Angular DI

A service is a useful state owner when multiple components need the same state and transitions. A service provided with `providedIn: 'root'` is available through the root injector, so consumers normally share that instance.

~~~ts
@Injectable({ providedIn: 'root' })
export class PreferencesState {
  readonly compactMode = signal(false);
}
~~~

A service listed in a component's `providers` array gets an instance in that component's injector. Descendants can share that instance, but another component with its own provider gets another instance. That can be useful for independent editors or dialogs; it is a bug if sibling screens were supposed to share one store.

Use the narrowest scope that satisfies the feature:

- Use component signals for state that only matters to that component.
- Use a route or feature provider for state shared inside one feature instance.
- Use a root provider for state intentionally shared across the application.
- Avoid making every service root-scoped by habit; app-wide lifetime can retain state longer than intended.

## Inputs, outputs, models, and ownership

A parent usually owns a value and passes it to a child through an input. The child can request a change through an output, and the parent decides whether and how to update the source value.

A `model()` input is appropriate when a reusable child control is designed to edit a value and expose that edit through a two-way component API. It is not a reason to create a second copy of every input. If a child needs an independent draft—for example, an editor with Save and Cancel—make that draft explicit and define when it is initialized and committed.

## Treat drafts as real, temporary state

An unsaved form draft is not necessarily a harmful duplicate. It has a different meaning from the saved record:

- The record is the last committed value.
- The draft is the user's in-progress edit.
- Save validates and sends the draft, then updates the saved state after success.
- Cancel discards the draft and returns to the saved value.

Make these boundaries clear. Do not silently overwrite an active draft every time a background refresh arrives. Decide whether to preserve it, merge it, or ask the user to resolve a conflict.

## Update state through its owner

Consumers should ask the owner to perform a meaningful transition. A service can expose methods such as `selectTask`, `toggleTask`, or `removeTask` instead of giving every component unrestricted write access to internal state.

This makes it easier to validate IDs, preserve invariants, clear a selection after deletion, and later add persistence or server calls without duplicating rules in multiple components.

## A practical review checklist

Before adding a new signal, field, or store entry, ask:

- Is this an independent fact or a calculation?
- Is the same fact already stored under a different shape?
- Which component or service is responsible for changing it?
- Do all consumers need the same instance?
- Is this state temporary, feature-scoped, or app-wide?
- When its source changes, should this value reset, remain if valid, or clear?
- Is this a meaningful unsaved draft, or just a copy that could become stale?
- Can a child request a change from the owner instead of keeping its own competing copy?

## Key takeaways

- Give each independent fact one clear owner.
- Derive counts, filtered collections, and selected objects instead of storing synchronized copies.
- Store stable identifiers when the referenced object can be replaced or updated.
- Use `computed()` for read-only derivation and `linkedSignal()` for writable dependent state.
- Use effects for appropriate external side effects, not for copying application state.
- Choose service scope deliberately; component-level providers create separate instances.
- Keep form drafts when their separate, temporary meaning is intentional.

## Further reading

- [Signals](https://angular.dev/guide/signals)
- [Dependent state with `linkedSignal`](https://angular.dev/guide/signals/linked-signal)
- [Effects](https://angular.dev/guide/signals/effect)
- [Defining dependency providers](https://angular.dev/guide/di/dependency-injection-providers)
- [Hierarchical dependency injection](https://angular.dev/guide/di/hierarchical-dependency-injection)
