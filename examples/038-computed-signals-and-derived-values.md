# 38. Example: Computed signals and derived values

This standalone task-list example stores only the values users can change: the task list, search query, selected filter, and whether extra details are visible. Computed signals derive the visible tasks, remaining-task count, and summaries from that source state.

## What you will build

- A writable task list, text query, filter choice, and details toggle.
- A `visibleTasks` computed signal that combines the list, query, and filter.
- A `remainingCount` computed signal derived from the task list.
- A conditional summary whose dependencies change depending on whether details are shown.
- Immutable task updates that cause dependent computed values to refresh.

## Start a standalone Angular app

If you do not already have a standalone project, create and run one with the Angular CLI:

~~~sh
ng new computed-signals-demo --standalone --style=css
cd computed-signals-demo
ng serve
~~~

Use the generated root component as the starting point. The snippets use conventional `app.component.*` filenames; if your CLI uses shorter names such as `app.*`, use its generated names and keep the root component bootstrap consistent.

## File layout

~~~text
src/app/
  app.component.css
  app.component.html
  app.component.ts
~~~

## 1. Define source signals and computed values

Replace or adapt `src/app/app.component.ts`:

~~~ts
import { Component, computed, signal } from '@angular/core';

interface Task {
  id: number;
  title: string;
  done: boolean;
}

type TaskFilter = 'all' | 'open' | 'done';

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  readonly tasks = signal<Task[]>([
    { id: 1, title: 'Read the Signals guide', done: true },
    { id: 2, title: 'Build a small computed example', done: false },
    { id: 3, title: 'Review the example', done: false },
  ]);

  readonly query = signal('');
  readonly filter = signal<TaskFilter>('all');
  readonly showDetails = signal(false);

  readonly visibleTasks = computed(() => {
    const normalizedQuery = this.query().trim().toLowerCase();
    const selectedFilter = this.filter();

    return this.tasks().filter((task) => {
      const matchesQuery = task.title.toLowerCase().includes(normalizedQuery);
      const matchesFilter =
        selectedFilter === 'all' ||
        (selectedFilter === 'open' && !task.done) ||
        (selectedFilter === 'done' && task.done);

      return matchesQuery && matchesFilter;
    });
  });

  readonly remainingCount = computed(
    () => this.tasks().filter((task) => !task.done).length,
  );

  readonly resultSummary = computed(
    () => `${this.visibleTasks().length} of ${this.tasks().length} tasks shown`,
  );

  readonly extraSummary = computed(() => {
    if (!this.showDetails()) {
      return 'Extra summary is hidden.';
    }

    const total = this.tasks().length;
    const remaining = this.remainingCount();
    return `${remaining} of ${total} tasks are still open.`;
  });

  private nextTaskId = 4;

  updateQuery(value: string): void {
    this.query.set(value);
  }

  setFilter(filter: TaskFilter): void {
    this.filter.set(filter);
  }

  addTask(): void {
    const task: Task = {
      id: this.nextTaskId,
      title: `New task ${this.nextTaskId}`,
      done: false,
    };
    this.nextTaskId += 1;

    this.tasks.update((current) => [...current, task]);
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

  toggleDetails(): void {
    this.showDetails.update((visible) => !visible);
  }
}
~~~

The component keeps source state writable and derived values computed. The `visibleTasks` calculation reads `query`, `filter`, and `tasks`, so those signals are its dependencies. The `extraSummary` reads the task-derived values only when `showDetails()` is true; its dependencies change when that branch changes.

## 2. Display the derived values

Create or replace `src/app/app.component.html`:

~~~html
<main class="page">
  <header class="page-header">
    <p class="eyebrow">Angular example 38</p>
    <h1>Computed task list</h1>
    <p>
      Search, choose a filter, and change tasks. The visible list and counts
      are derived from the current signals.
    </p>
  </header>

  <section class="panel" aria-labelledby="summary-title">
    <div class="summary-heading">
      <div>
        <h2 id="summary-title">Task summary</h2>
        <p>{{ remainingCount() }} tasks remain open.</p>
      </div>
      <button type="button" class="secondary" (click)="toggleDetails()">
        {{ showDetails() ? 'Hide details' : 'Show details' }}
      </button>
    </div>
    <p class="detail" aria-live="polite">{{ extraSummary() }}</p>
  </section>

  <section class="panel" aria-labelledby="tasks-title">
    <div class="tasks-heading">
      <div>
        <h2 id="tasks-title">Tasks</h2>
        <p aria-live="polite">{{ resultSummary() }}</p>
      </div>
      <button type="button" (click)="addTask()">Add task</button>
    </div>

    <label class="search-field">
      <span>Search task titles</span>
      <input
        type="search"
        [value]="query()"
        (input)="updateQuery($any($event.target).value)"
        placeholder="Try “review”"
      />
    </label>

    <div class="filter-row" aria-label="Filter tasks">
      <button
        type="button"
        [class.active]="filter() === 'all'"
        [attr.aria-pressed]="filter() === 'all'"
        (click)="setFilter('all')"
      >
        All
      </button>
      <button
        type="button"
        [class.active]="filter() === 'open'"
        [attr.aria-pressed]="filter() === 'open'"
        (click)="setFilter('open')"
      >
        Open
      </button>
      <button
        type="button"
        [class.active]="filter() === 'done'"
        [attr.aria-pressed]="filter() === 'done'"
        (click)="setFilter('done')"
      >
        Done
      </button>
    </div>

    @if (visibleTasks().length === 0) {
      <p class="empty-state">No tasks match this search and filter.</p>
    } @else {
      <ul class="task-list">
        @for (task of visibleTasks(); track task.id) {
          <li>
            <span class="task-title" [class.done]="task.done">
              {{ task.title }}
            </span>
            <div class="task-actions">
              <button
                type="button"
                class="secondary"
                (click)="toggleTask(task.id)"
              >
                {{ task.done ? 'Mark open' : 'Mark done' }}
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

The template calls computed signals just like writable signals. It does not manually rebuild the filtered list or update the remaining count from each click handler.

## 3. Style the page

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
  width: min(100% - 2rem, 56rem);
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

h2 {
  margin-block: 0 0.4rem;
}

.panel {
  margin-block: 1.25rem;
  padding: clamp(1rem, 4vw, 1.75rem);
  border: 1px solid #cbd5e1;
  border-radius: 0.9rem;
  background: #fff;
  box-shadow: 0 0.5rem 1.5rem rgb(15 23 42 / 6%);
}

.summary-heading,
.tasks-heading {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}

.detail {
  margin-bottom: 0;
  color: #43516a;
}

.search-field {
  display: grid;
  gap: 0.45rem;
  margin-block: 1.5rem 1rem;
  font-weight: 650;
}

input[type='search'] {
  width: 100%;
  min-height: 2.75rem;
  padding: 0.6rem 0.75rem;
  border: 1px solid #8b9ab1;
  border-radius: 0.5rem;
  color: #172033;
  background: #fff;
  font: inherit;
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

button.active {
  border-color: #173b72;
  color: #fff;
  background: #173b72;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 3px;
}

.filter-row,
.task-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.55rem;
}

.filter-row {
  margin-block: 1rem;
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
  padding: 0.85rem;
  border: 1px solid #d7dfeb;
  border-radius: 0.6rem;
}

.task-title.done {
  color: #536176;
  text-decoration: line-through;
}

.empty-state {
  padding: 1rem;
  border: 1px dashed #8b9ab1;
  border-radius: 0.6rem;
  color: #43516a;
}
~~~

## How the derivations behave

1. The user can change only `tasks`, `query`, `filter`, and `showDetails` directly through component methods.
2. `visibleTasks` derives a filtered array from the task list, query, and selected filter. Updating any of those source signals invalidates this result.
3. `remainingCount` derives its number from task completion state. Toggling or removing a task updates the task signal with a new array and the count follows automatically.
4. `resultSummary` composes other computed values and source state; it does not duplicate or separately store a result count.
5. `extraSummary` reads task values only when its details branch is enabled. While details are hidden, its current computation depends only on `showDetails`.
6. Computed values are lazy and cached. They are recalculated when read after a dependency changes; they are not a place to run logging or other required side effects.

## Try these variations

- Search for “signals”, then select **Open** or **Done**. The list and result summary reflect both source values.
- Mark a task done while the **Open** filter is selected. It disappears from the visible list, and the open count changes.
- Remove all visible tasks, then clear the search or switch the filter to see the empty state.
- Add a filter such as “priority” by adding a source signal and reading it inside `visibleTasks`.
- Add a separate computed value such as a completion percentage. Handle an empty task list deliberately to avoid dividing by zero.
- Inspect the Signal Graph in Angular DevTools to see the signals and computed dependencies for this component.

## Common problems

- **A computed value is blank or stale:** Confirm the template calls it with parentheses and the calculation reads the source signals it needs.
- **A change to a plain property does not update the list:** Convert changing inputs into signals and read them inside the computed callback.
- **Trying to assign to `visibleTasks`:** It is read-only. Change `tasks`, `query`, or `filter` instead.
- **The list does not react after mutating a task in place:** Return a new task object and a new array from the update.
- **A request runs fewer times than expected:** Computed callbacks are lazy and memoized. Start asynchronous work in an appropriate resource or service workflow, not inside `computed()`.
- **A signal is missing from the active dependency set:** Only signals read in the latest execution are tracked. A conditional branch can add or remove dependencies.

## Further reading

- [Lesson 38: Computed signals and derived values](../lessons/038-computed-signals-and-derived-values.md)
- [Q&A 38](../qnas/038-computed-signals-and-derived-values.md)
- [Angular Signals guide](https://angular.dev/guide/signals)
- [computed API](https://angular.dev/api/core/computed)
- [Angular DevTools signal graph](https://angular.dev/tools/devtools/signals)