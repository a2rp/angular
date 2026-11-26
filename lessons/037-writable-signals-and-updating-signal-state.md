# 37. Writable signals and updating signal state

A writable signal stores a value and exposes a small API for reading and changing it. Read the current value by calling the signal, replace it with `set()`, or derive a replacement from the current value with `update()`. Angular tracks reads in templates and other reactive consumers so it can respond when the signal changes.

Writable signals are useful for local state such as a counter, selected option, form draft, or list of items. They are not a reason to wrap every value in a signal: use them when a value changes and the UI or another reactive consumer needs to follow that change.

## Learning goals

After this lesson, you should be able to:

- Create and read a writable signal with `signal()`.
- Choose between `set()` and `update()` for a state change.
- Update arrays and objects immutably so new references are produced.
- Avoid mutating a nested value while keeping the same signal value reference.
- Expose a readonly signal while keeping write operations inside a service or component.
- Recognize where signal writes belong and where they do not.

## Create and read a writable signal

Import `signal` from `@angular/core` and provide an initial value:

~~~ts
import { signal } from '@angular/core';

const count = signal(0);
const name = signal('Ada');
const selectedIds = signal<number[]>([]);
~~~

The `signal()` function returns a callable value. Call it to read the current state:

~~~ts
console.log(count()); // 0
console.log(name()); // 'Ada'
~~~

The initial value determines the signal's type. When a value can be absent, express that in the type:

~~~ts
const selectedUser = signal<string | undefined>(undefined);
~~~

In a component, signals commonly represent state owned by that component:

~~~ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <p>Count: {{ count() }}</p>
    <button type="button" (click)="increment()">Add one</button>
  `,
})
export class CounterComponent {
  readonly count = signal(0);

  increment(): void {
    this.count.update((current) => current + 1);
  }
}
~~~

The signal is called in the template: `{{ count() }}`. That read lets Angular know the template depends on the signal. When the value changes, Angular can refresh the relevant view. A signal read in an `OnPush` component template also marks that component for checking when the signal changes.

## Change a value with `set()`

Use `set()` when you already have the complete next value:

~~~ts
const count = signal(0);

count.set(5);
count.set(0);
~~~

In a component method, `set()` is a clear fit for reset or for applying a value that arrived from another operation:

~~~ts
reset(): void {
  this.count.set(0);
}
~~~

For example, after receiving a value from a request or parsing user input, set the signal when that operation completes:

~~~ts
this.settingsService.load().then((settings) => {
  this.settings.set(settings);
});
~~~

Do the asynchronous work outside the signal updater. `set()` accepts the next state value; it is not an async callback API.

## Derive a value with `update()`

Use `update()` when the next value depends on the current value. Angular passes the current value to the callback, and the callback returns the replacement:

~~~ts
const count = signal(0);

count.update((current) => current + 1);
count.update((current) => Math.max(0, current - 1));
~~~

This is convenient for increments, toggles, and other changes expressed from the current state:

~~~ts
readonly isOpen = signal(false);

 toggle(): void {
  this.isOpen.update((open) => !open);
}
~~~

The callback should calculate and return the next value. Keep it synchronous and free of side effects: do not start a request, change another signal, log for business behavior, or mutate the current object from inside the updater. Put such work in the event handler or service method that decides to update the signal.

## `set()` or `update()`?

Both methods write a new value. Choose based on where that value comes from:

| Situation | Use | Example |
| --- | --- | --- |
| The full next value is already known | `set()` | `this.count.set(0)` |
| The next value depends on the current one | `update()` | `this.count.update((n) => n + 1)` |
| Replace state with a server response | `set()` | `this.profile.set(response)` |
| Change one field while preserving the rest | `update()` | `this.profile.update((p) => ({ ...p, name }))` |

For dependent changes, `update()` keeps the read-and-replace logic together. Avoid first reading a value and later writing a derived value if another update could happen between those operations.

## Update objects without mutating them

Signals use equality to decide whether the new value should notify dependents. By default, Angular uses referential equality (`Object.is()`). If an object is changed in place, its reference remains the same:

~~~ts
interface Profile {
  name: string;
  city: string;
}

const profile = signal<Profile>({ name: 'Asha', city: 'Pune' });

// Avoid: this changes the existing object in place.
profile().city = 'Jaipur';
profile.set(profile());
~~~

That is not a reliable signal update. Create a new object and preserve the fields that did not change:

~~~ts
profile.update((current) => ({
  ...current,
  city: 'Jaipur',
}));
~~~

For nested state, copy each object along the path being changed:

~~~ts
interface Preferences {
  notifications: {
    email: boolean;
    productUpdates: boolean;
  };
}

const preferences = signal<Preferences>({
  notifications: { email: true, productUpdates: false },
});

preferences.update((current) => ({
  ...current,
  notifications: {
    ...current.notifications,
    productUpdates: true,
  },
}));
~~~

The spread is shallow at each level, so copy the nested object whose property changes. Do not assume that calling `asReadonly()` later will freeze nested objects; readonly signals prevent signal writes through that reference, not deep mutation of the value.

## Update arrays immutably

Use array methods that produce a new array:

~~~ts
const tasks = signal(['Read the guide']);

// Add an item.
tasks.update((current) => [...current, 'Build the example']);

// Remove matching items.
tasks.update((current) => current.filter((task) => task !== 'Read the guide'));
~~~

When the items are objects, copy the changed item and the containing array:

~~~ts
interface Task {
  id: number;
  title: string;
  done: boolean;
}

const taskList = signal<Task[]>([
  { id: 1, title: 'Read the guide', done: false },
]);

taskList.update((current) =>
  current.map((task) =>
    task.id === 1 ? { ...task, done: true } : task,
  ),
);
~~~

The original task and array remain unchanged. The replacement references make the state change visible to signal consumers and keep updates easier to reason about.

## Expose readonly state

A component or service can keep the writable signal private and expose its readonly view to consumers:

~~~ts
import { Injectable, signal } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class CounterState {
  private readonly _count = signal(0);
  readonly count = this._count.asReadonly();

  increment(): void {
    this._count.update((current) => current + 1);
  }

  reset(): void {
    this._count.set(0);
  }
}
~~~

Other code can call `state.count()` to read the value, but it cannot call `set()` or `update()` on `state.count`. It must ask the service to perform an allowed operation. This helps keep state transitions in one place and prevents unrelated consumers from writing arbitrary values.

`asReadonly()` is an API boundary, not a deep freeze. If the signal contains an object or array, a consumer could still mutate a nested property on the object it reads. Keep updates immutable and avoid exposing mutable objects that consumers are expected to edit directly.

## Writable signals and readonly signals

| Signal kind | Read | `set()` / `update()` | Typical use |
| --- | --- | --- | --- |
| Writable signal from `signal()` | Call it, such as `count()` | Yes | State owned by this component or service |
| Readonly view from `asReadonly()` | Call it | No | Expose state without exposing its write API |
| Computed signal from `computed()` | Call it | No | Derived state; covered in the next lesson |
| Input signal from `input()` | Call it | No | Value provided by a parent component |

A read-only signal can still change when its original writable signal changes. Read-only describes the operations available through that reference; it does not mean the value is constant.

## Where signal writes belong

Update state in response to an explicit action or completed operation, such as:

- A click or keyboard event handler.
- A component method called by the template or another component API.
- A service method that completes an application operation.
- A request callback after asynchronous work finishes.

Do not write a signal while Angular is evaluating a template or computing another signal. A derived calculation that depends on signals should return its value rather than write that result into another signal. The next lesson introduces `computed()` for that purpose.

A signal update is synchronous state change, not an event stream. If a value represents a stream of time-based events or needs RxJS operators, use the appropriate RxJS tools and interop rather than treating repeated `set()` calls as a replacement for an Observable.

## Common mistakes

- Reading `count` instead of calling `count()`.
- Calling `.set()` or `.update()` on a computed signal, input signal, or readonly signal.
- Mutating an object or array and writing the same reference back.
- Updating a nested object without copying the containing object path.
- Using `set()` after calculating a dependent value from stale state when `update()` would express the transition clearly.
- Putting network requests, logging, or other side effects inside an `update()` callback.
- Trying to `await` inside `update()` and accidentally storing a Promise instead of the intended value.
- Assuming `asReadonly()` freezes an object or array deeply.
- Turning a value into a signal even though it is constant or never read reactively.
- Writing to a signal from a template expression or a `computed()` derivation.

## Key takeaways

- Read a signal by calling it; change a writable signal with `set()` or `update()`.
- Use `set()` for a complete next value and `update()` when the next value depends on the current one.
- Return new object and array references instead of mutating signal values in place.
- Keep updater callbacks synchronous and free of side effects.
- Use `asReadonly()` to expose readable state while keeping write methods behind a service or component API.
- A readonly signal reference does not deep-freeze the objects it contains.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="036-custom-pipes-attribute-directives-and-reusable-behavior.md">036 - 36. Custom pipes, attribute directives, and reusable behavior</a></td><td><a href="../qnas/037-writable-signals-and-updating-signal-state.md">Q&amp;A</a></td><td><a href="../examples/037-writable-signals-and-updating-signal-state.md">Example</a></td><td><a href="038-computed-signals-and-derived-values.md">038 - 38. Computed signals and derived values</a></td>
</tr>
  </tbody>
</table>
