# State ownership and avoiding duplicated state — examples

This example builds a small task board with two sibling components:

- The task list owns its temporary filter because only that panel uses it.
- A root-provided service owns the task collection and selected task ID because both panels need the same state.
- The selected task object and filtered list are derived values. Neither is stored as a second writable copy.

The code uses standalone components and Angular's built-in control flow.

## File layout

~~~text
src/
├── main.ts
└── app/
    ├── app.component.ts
    ├── task-board-state.ts
    ├── task-list.component.ts
    └── task-details.component.ts
~~~

## 1. Define the shared state owner

~~~ts
// src/app/task-board-state.ts
import { computed, Injectable, signal } from '@angular/core';

export interface Task {
  readonly id: number;
  readonly title: string;
  readonly done: boolean;
}

const initialTasks: Task[] = [
  { id: 1, title: 'Choose the state owner', done: true },
  { id: 2, title: 'Derive the selected task', done: false },
  { id: 3, title: 'Keep the filter local', done: false },
];

@Injectable({ providedIn: 'root' })
export class TaskBoardState {
  readonly #tasks = signal<Task[]>(initialTasks);

  // Consumers can read the collection but only change it through methods below.
  readonly tasks = this.#tasks.asReadonly();

  readonly #selectedTaskId = signal<number | null>(null);
  readonly selectedTaskId = this.#selectedTaskId.asReadonly();

  // Always look up the selected task from the latest collection.
  readonly selectedTask = computed(() => {
    const id = this.selectedTaskId();

    if (id === null) {
      return null;
    }

    return this.tasks().find((task) => task.id === id) ?? null;
  });

  readonly completedCount = computed(
    () => this.tasks().filter((task) => task.done).length,
  );

  selectTask(id: number): void {
    const exists = this.tasks().some((task) => task.id === id);

    if (exists) {
      this.#selectedTaskId.set(id);
    }
  }

  toggleTask(id: number): void {
    this.#tasks.update((tasks) =>
      tasks.map((task) =>
        task.id === id ? { ...task, done: !task.done } : task,
      ),
    );
  }

  removeTask(id: number): void {
    // Clear selection so it cannot point to a removed task.
    if (this.selectedTaskId() === id) {
      this.#selectedTaskId.set(null);
    }

    this.#tasks.update((tasks) => tasks.filter((task) => task.id !== id));
  }
}
~~~

The service keeps one collection and one selected ID. The selected task and completed count are read-only computations. Updating a task therefore cannot leave a stale selected object or count behind.

## 2. Keep the filter local to the list panel

~~~ts
// src/app/task-list.component.ts
import { Component, computed, inject, signal } from '@angular/core';
import { TaskBoardState } from './task-board-state';

type TaskFilter = 'all' | 'open' | 'done';

@Component({
  selector: 'app-task-list',
  standalone: true,
  template: `
    <section aria-labelledby="task-list-title">
      <h2 id="task-list-title">Tasks</h2>

      <p>{{ state.completedCount() }} completed</p>

      <div role="group" aria-label="Filter tasks">
        <button
          type="button"
          [attr.aria-pressed]="filter() === 'all'"
          (click)="setFilter('all')"
        >
          All
        </button>
        <button
          type="button"
          [attr.aria-pressed]="filter() === 'open'"
          (click)="setFilter('open')"
        >
          Open
        </button>
        <button
          type="button"
          [attr.aria-pressed]="filter() === 'done'"
          (click)="setFilter('done')"
        >
          Done
        </button>
      </div>

      <ul>
        @for (task of visibleTasks(); track task.id) {
          <li>
            <button
              type="button"
              [attr.aria-pressed]="task.id === state.selectedTaskId()"
              (click)="state.selectTask(task.id)"
            >
              {{ task.title }}
            </button>

            @if (task.done) {
              <span> — complete</span>
            } @else {
              <span> — open</span>
            }
          </li>
        } @empty {
          <li>No tasks match this filter.</li>
        }
      </ul>
    </section>
  `,
})
export class TaskListComponent {
  readonly state = inject(TaskBoardState);
  readonly filter = signal<TaskFilter>('all');

  readonly visibleTasks = computed(() => {
    const filter = this.filter();

    return this.state.tasks().filter((task) => {
      if (filter === 'open') {
        return !task.done;
      }

      if (filter === 'done') {
        return task.done;
      }

      return true;
    });
  });

  setFilter(filter: TaskFilter): void {
    this.filter.set(filter);
  }
}
~~~

The filter exists only in TaskListComponent because no other part of this example needs to own it. The visible list is derived from the service's task collection and the local filter.

## 3. Read and update the shared selection in the details panel

~~~ts
// src/app/task-details.component.ts
import { Component, inject } from '@angular/core';
import { TaskBoardState } from './task-board-state';

@Component({
  selector: 'app-task-details',
  standalone: true,
  template: `
    <section aria-labelledby="task-details-title">
      <h2 id="task-details-title">Selected task</h2>

      @if (state.selectedTask(); as task) {
        <h3>{{ task.title }}</h3>
        <p>Status: {{ task.done ? 'Complete' : 'Open' }}</p>

        <button type="button" (click)="state.toggleTask(task.id)">
          {{ task.done ? 'Reopen task' : 'Complete task' }}
        </button>

        <button type="button" (click)="state.removeTask(task.id)">
          Remove task
        </button>
      } @else {
        <p>Select a task to see its details.</p>
      }
    </section>
  `,
})
export class TaskDetailsComponent {
  readonly state = inject(TaskBoardState);
}
~~~

The details panel reads the same service instance as the list. When it changes a task, the collection updates and the derived selection resolves the latest task object.

## 4. Put both panels under one application component

~~~ts
// src/app/app.component.ts
import { Component } from '@angular/core';
import { TaskDetailsComponent } from './task-details.component';
import { TaskListComponent } from './task-list.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TaskListComponent, TaskDetailsComponent],
  template: `
    <main>
      <h1>Task board</h1>

      <app-task-list />
      <app-task-details />
    </main>
  `,
})
export class AppComponent {}
~~~

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

Because both panels inject the same root-provided TaskBoardState, a selection or task update in one panel is visible in the other. Do not add TaskBoardState to each panel's providers array; that would create separate component-scoped instances and split the state.

## 5. Avoid an effect-driven mirror

An effect that copies a selected ID into a second selected-task signal stores two writable representations of one relationship:

~~~ts
readonly selectedTaskId = signal<number | null>(null);
readonly selectedTask = signal<Task | null>(null);

constructor() {
  effect(() => {
    const id = this.selectedTaskId();
    this.selectedTask.set(
      this.tasks().find((task) => task.id === id) ?? null,
    );
  });
}
~~~

The selected object can lag behind the collection, and the update path is harder to reason about. Store the ID and derive the object instead:

~~~ts
readonly selectedTask = computed(() => {
  const id = this.selectedTaskId();
  return this.tasks().find((task) => task.id === id) ?? null;
});
~~~

Use an effect for a genuine boundary side effect, such as synchronizing a signal to browser storage or an imperative chart. For dependent application state, use a computation or an explicit state transition.

## 6. A writable dependent choice

A shipping selector may need a default option while still allowing the user to choose another one. If options change, this rule preserves the selection when it remains valid and otherwise chooses the first option:

~~~ts
import { linkedSignal, signal } from '@angular/core';

interface ShippingOption {
  readonly id: string;
  readonly label: string;
}

readonly options = signal<ShippingOption[]>([
  { id: 'standard', label: 'Standard' },
  { id: 'express', label: 'Express' },
]);

readonly selectedOptionId = linkedSignal<
  ShippingOption[],
  string | null
>({
  source: this.options,
  computation: (options, previous) =>
    options.some((option) => option.id === previous.value)
      ? previous.value
      : options[0]?.id ?? null,
});
~~~

This is different from a read-only derived value: the user can set selectedOptionId, and the signal has an explicit rule for what happens when its source list changes.

## What this example demonstrates

- Each independent value has a clear owner.
- The service owns shared task data and selection; the list component owns its filter.
- The selected object, task count, and visible tasks are derived instead of mirrored.
- Components request changes through the service's transition methods.
- Root-level sharing is intentional. A route or common-ancestor provider would fit a state instance isolated to one feature.
- A linked signal is useful when a dependent choice must also remain writable.
