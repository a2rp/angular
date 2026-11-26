# 37. Example: Writable signals and updating signal state

This small standalone app demonstrates writable signals in two places: a service owns a counter and exposes it as readonly state, while the component owns a profile and task list. The UI reads each signal by calling it. Buttons update state with `set()` or `update()` and replace arrays and objects instead of mutating them.

## What you will build

- A counter service with a private writable signal and a public readonly signal.
- A component that increments, decrements, and resets the counter.
- A component-owned profile object updated immutably.
- A task array where add, toggle, and remove operations return new arrays.

## Start a standalone Angular app

If needed, create and run a project with the Angular CLI:

~~~sh
ng new writable-signals-demo --standalone --style=css
cd writable-signals-demo
ng serve
~~~

Use the generated root component as the starting point. This example uses conventional `app.component.*` names; if your CLI generated shorter names such as `app.*`, use those names and keep the bootstrap pointed at your root component.

## File layout

~~~text
src/app/
  app.component.css
  app.component.html
  app.component.ts
  counter-state.service.ts
~~~

## 1. Create a service with a readonly signal API

Create `src/app/counter-state.service.ts`:

~~~ts
import { Injectable, signal } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class CounterStateService {
  private readonly _count = signal(0);
  readonly count = this._count.asReadonly();

  increment(): void {
    this._count.update((current) => current + 1);
  }

  decrement(): void {
    this._count.update((current) => Math.max(0, current - 1));
  }

  reset(): void {
    this._count.set(0);
  }
}
~~~

The service keeps the writable signal private. Other code can call `count()` to read the value and request an allowed operation through a method. Consumers cannot accidentally call `set()` or `update()` on the readonly view.

## 2. Create component-owned signals

Replace or adapt `src/app/app.component.ts`:

~~~ts
import { Component, inject, signal } from '@angular/core';
import { CounterStateService } from './counter-state.service';

interface Profile {
  name: string;
  city: string;
}

interface Task {
  id: number;
  title: string;
  done: boolean;
}

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  readonly counter = inject(CounterStateService);

  readonly profile = signal<Profile>({
    name: 'Asha',
    city: 'Pune',
  });

  readonly tasks = signal<Task[]>([
    { id: 1, title: 'Read the signals guide', done: false },
    { id: 2, title: 'Try an immutable update', done: true },
  ]);

  private readonly nextTaskId = signal(3);

  setCity(city: string): void {
    this.profile.update((current) => ({
      ...current,
      city,
    }));
  }

  addTask(): void {
    const id = this.nextTaskId();

    this.tasks.update((current) => [
      ...current,
      { id, title: `Practice task ${id}`, done: false },
    ]);

    this.nextTaskId.update((current) => current + 1);
  }

  toggleTask(id: number): void {
    this.tasks.update((current) =>
      current.map((task) =>
        task.id === id ? { ...task, done: !task.done } : task,
      ),
    );
  }

  removeTask(id: number): void {
    this.tasks.update((current) =>
      current.filter((task) => task.id !== id),
    );
  }
}
~~~

The component uses `update()` when a change depends on current state. `setCity()` creates a new profile object. The task operations create new arrays, and `toggleTask()` also creates a new object for the changed item. The updater callbacks only calculate and return values; they do not perform other work.

## 3. Read and update the signals in a template

Create or replace `src/app/app.component.html`:

~~~html
<main class="page">
  <header class="page-header">
    <p class="eyebrow">Angular example 37</p>
    <h1>Writable signals</h1>
    <p>
      Read a signal by calling it. Use <code>set()</code> when the next value is
      known and <code>update()</code> when it depends on the current value.
    </p>
  </header>

  <section class="panel" aria-labelledby="counter-title">
    <h2 id="counter-title">Counter service</h2>
    <p class="count" aria-live="polite">{{ counter.count() }}</p>
    <div class="button-row">
      <button type="button" (click)="counter.decrement()">−1</button>
      <button type="button" (click)="counter.increment()">+1</button>
      <button type="button" class="secondary" (click)="counter.reset()">
        Reset with set()
      </button>
    </div>
    <p class="note">
      The component can read <code>counter.count()</code>, but only the service
      can write the private signal.
    </p>
  </section>

  <section class="panel" aria-labelledby="profile-title">
    <h2 id="profile-title">Immutable object update</h2>
    <p>
      {{ profile().name }} lives in {{ profile().city }}.
    </p>
    <div class="button-row">
      <button type="button" (click)="setCity('Jaipur')">Move to Jaipur</button>
      <button type="button" class="secondary" (click)="setCity('Pune')">
        Move to Pune
      </button>
    </div>
    <p class="note">
      Each click returns a new profile object and preserves the name.
    </p>
  </section>

  <section class="panel" aria-labelledby="tasks-title">
    <h2 id="tasks-title">Immutable array and item updates</h2>
    <button type="button" (click)="addTask()">Add a task</button>

    @if (tasks().length === 0) {
      <p>No tasks yet. Add one to get started.</p>
    } @else {
      <ul class="task-list">
        @for (task of tasks(); track task.id) {
          <li>
            <span [class.done]="task.done">{{ task.title }}</span>
            <div class="button-row">
              <button
                type="button"
                class="secondary"
                (click)="toggleTask(task.id)"
              >
                {{ task.done ? 'Mark incomplete' : 'Mark complete' }}
              </button>
              <button
                type="button"
                class="secondary"
                (click)="removeTask(task.id)"
                [attr.aria-label]="'Remove ' + task.title"
              >
                Remove
              </button>
            </div>
          </li>
        }
      </ul>
    }
  </section>
</main>
~~~

The template calls each signal to read its current value. The counter buttons call service methods. The task buttons call component methods that return new arrays rather than mutating the existing task list.

## 4. Add the styles

Create or replace `src/app/app.component.css`:

~~~css
:host {
  display: block;
  min-height: 100vh;
  color: #172033;
  background: #f3f6fb;
  font-family: system-ui, sans-serif;
}

.page {
  width: min(100% - 2rem, 54rem);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  margin-bottom: 2rem;
}

.eyebrow {
  color: #425b84;
  font-size: 0.875rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

h1,
h2 {
  line-height: 1.2;
}

h1 {
  margin-block: 0.5rem 1rem;
  font-size: clamp(2rem, 6vw, 3rem);
}

.panel {
  margin-block: 1.25rem;
  padding: clamp(1rem, 4vw, 1.75rem);
  border: 1px solid #cbd5e1;
  border-radius: 0.9rem;
  background: #fff;
  box-shadow: 0 0.5rem 1.5rem rgb(15 23 42 / 6%);
}

.count {
  margin-block: 0.5rem 1rem;
  color: #173b72;
  font-size: 3rem;
  font-variant-numeric: tabular-nums;
  font-weight: 750;
}

.button-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
}

button {
  min-height: 2.65rem;
  padding: 0.55rem 0.9rem;
  border: 1px solid #1d4ed8;
  border-radius: 0.5rem;
  color: #fff;
  background: #1d4ed8;
  font: inherit;
  font-weight: 650;
  cursor: pointer;
}

button.secondary {
  border-color: #8b9ab1;
  color: #172033;
  background: #fff;
}

button:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 3px;
}

.note {
  margin-top: 1rem;
  color: #43516a;
  font-size: 0.95rem;
}

.task-list {
  display: grid;
  gap: 0.75rem;
  padding: 0;
  list-style: none;
}

.task-list li {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
  padding: 0.8rem;
  border: 1px solid #d7dfeb;
  border-radius: 0.6rem;
}

.done {
  color: #536176;
  text-decoration: line-through;
}

code {
  padding-inline: 0.2em;
  border-radius: 0.2em;
  background: #eef2f7;
}
~~~

## How to explore the state changes

1. Click **+1** several times. The component reads the service's readonly counter signal; the service changes its private writable signal with `update()`.
2. Click **Reset with set()**. The service already knows the complete next value, so it uses `set(0)`.
3. Change the profile city. `setCity()` uses `update()` and returns a new object that keeps the existing name.
4. Add, complete, and remove tasks. These methods use array spread, `map()`, and `filter()` to return new arrays.
5. Inspect `counter.count` in the component. It is a readonly signal view, so there is no `.set()` or `.update()` method available on it.

## Common problems

- **The template displays a function-like value:** Call the signal, for example `count()`, in the template.
- **The template does not refresh after an object edit:** Do not mutate the object and write the same reference back. Return a new object from `update()`.
- **A value has no `set()` method:** It may be a readonly signal such as `asReadonly()`, an input signal, or a computed signal.
- **The updater returns the wrong type:** The callback must return the complete next value, not just perform a mutation.
- **An async operation stores the wrong thing:** Await or subscribe outside `update()`, then write the completed result with `set()`.
- **A readonly API still exposes mutable nested data:** `asReadonly()` does not deep-freeze values. Avoid direct mutation and keep state updates immutable.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="036-custom-pipes-attribute-directives-and-reusable-behavior.md">036 - 36. Custom pipes, attribute directives, and reusable behavior</a></td><td><a href="../lessons/037-writable-signals-and-updating-signal-state.md">Lesson</a></td><td><a href="../qnas/037-writable-signals-and-updating-signal-state.md">Q&amp;A</a></td><td><a href="038-computed-signals-and-derived-values.md">038 - 38. Computed signals and derived values</a></td>
</tr>
  </tbody>
</table>
