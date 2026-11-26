# Examples: Parent-child communication and one-way data flow

Build a task list with one parent and two child component types. The parent owns the task collection. Each task row receives one task through an input and emits toggle or delete requests. A sibling summary receives counts from the same parent.

~~~text
AppComponent owns tasks
  ├── TaskSummary receives total and completed counts
  └── TaskRow receives one task and emits an action request
             │
             └── AppComponent updates tasks and passes new values down
~~~

This example uses standalone components and signal-based inputs and outputs. Keep your Angular CLI-generated bootstrap and application configuration; connect the root component shown here using your existing setup.

## 1. Create a task row child

Create `src/app/task-row.ts`:

~~~ts
import { Component, input, output } from '@angular/core';

export interface Task {
  id: number;
  title: string;
  completed: boolean;
}

@Component({
  selector: 'app-task-row',
  standalone: true,
  templateUrl: './task-row.html',
  styleUrl: './task-row.css',
})
export class TaskRow {
  readonly task = input.required<Task>();
  readonly toggleRequested = output<number>();
  readonly deleteRequested = output<number>();

  requestToggle(): void {
    this.toggleRequested.emit(this.task().id);
  }

  requestDelete(): void {
    this.deleteRequested.emit(this.task().id);
  }
}
~~~

Create `src/app/task-row.html`:

~~~html
<article class="task-row">
  <div class="task-details">
    <h2 [class.completed]="task().completed">{{ task().title }}</h2>
    <p>{{ task().completed ? 'Completed' : 'Not completed' }}</p>
  </div>

  <div class="actions">
    <button
      type="button"
      [attr.aria-pressed]="task().completed"
      (click)="requestToggle()"
    >
      {{ task().completed ? 'Mark incomplete' : 'Mark complete' }}
    </button>
    <button class="delete" type="button" (click)="requestDelete()">
      Delete
    </button>
  </div>
</article>
~~~

Create `src/app/task-row.css`:

~~~css
.task-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 1rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.65rem;
  background: white;
}

.task-details h2 {
  margin: 0;
  font-size: 1rem;
}

.task-details h2.completed {
  color: #64748b;
  text-decoration: line-through;
}

.task-details p {
  margin: 0.35rem 0 0;
  color: #475569;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

button {
  padding: 0.5rem 0.7rem;
  border: 0;
  border-radius: 0.4rem;
  background: #1d4ed8;
  color: white;
  font: inherit;
  cursor: pointer;
}

button.delete {
  background: #b91c1c;
}

button:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 2px;
}

@media (max-width: 34rem) {
  .task-row {
    align-items: flex-start;
    flex-direction: column;
  }
}
~~~

The row reads `task()` but does not modify the input. Its outputs contain the task ID so the parent can identify which item the user acted on.

## 2. Create a sibling summary component

Create `src/app/task-summary.ts`:

~~~ts
import { Component, input } from '@angular/core';

@Component({
  selector: 'app-task-summary',
  standalone: true,
  templateUrl: './task-summary.html',
  styleUrl: './task-summary.css',
})
export class TaskSummary {
  readonly total = input.required<number>();
  readonly completed = input.required<number>();
}
~~~

Create `src/app/task-summary.html`:

~~~html
<section class="summary" aria-label="Task summary">
  <strong>{{ completed() }} of {{ total() }}</strong>
  <span>tasks completed</span>
</section>
~~~

Create `src/app/task-summary.css`:

~~~css
.summary {
  display: flex;
  flex-wrap: wrap;
  gap: 0.35rem;
  padding: 0.8rem 1rem;
  border-radius: 0.6rem;
  background: #e0f2fe;
  color: #0c4a6e;
}
~~~

This sibling does not read the task rows directly. The parent sends it the values it needs.

## 3. Let the parent own and update the list

Create or adapt the root component at `src/app/app.ts`:

~~~ts
import { Component, computed, signal } from '@angular/core';
import { TaskRow, type Task } from './task-row';
import { TaskSummary } from './task-summary';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TaskRow, TaskSummary],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class AppComponent {
  readonly tasks = signal<Task[]>([
    { id: 1, title: 'Read the Angular lesson', completed: true },
    { id: 2, title: 'Practice input bindings', completed: false },
    { id: 3, title: 'Build a task row', completed: false },
  ]);

  readonly completedCount = computed(
    () => this.tasks().filter((task) => task.completed).length,
  );

  private nextId = 4;

  addTask(): void {
    const id = this.nextId++;
    const task: Task = {
      id,
      title: 'New task ' + id,
      completed: false,
    };

    this.tasks.update((tasks) => [...tasks, task]);
  }

  toggleTask(id: number): void {
    this.tasks.update((tasks) =>
      tasks.map((task) =>
        task.id === id ? { ...task, completed: !task.completed } : task,
      ),
    );
  }

  deleteTask(id: number): void {
    this.tasks.update((tasks) => tasks.filter((task) => task.id !== id));
  }
}
~~~

Create `src/app/app.html`:

~~~html
<main class="page">
  <header class="page-header">
    <div>
      <h1>Tasks</h1>
      <p>The parent owns the list; each row reports user actions.</p>
    </div>
    <button class="add" type="button" (click)="addTask()">Add task</button>
  </header>

  <app-task-summary
    [total]="tasks().length"
    [completed]="completedCount()"
  />

  @if (tasks().length === 0) {
    <p class="empty">No tasks yet. Add one to get started.</p>
  } @else {
    <ul class="task-list">
      @for (task of tasks(); track task.id) {
        <li>
          <app-task-row
            [task]="task"
            (toggleRequested)="toggleTask($event)"
            (deleteRequested)="deleteTask($event)"
          />
        </li>
      }
    </ul>
  }
</main>
~~~

Create `src/app/app.css`:

~~~css
:host {
  display: block;
  color: #172033;
  font-family: system-ui, sans-serif;
}

.page {
  width: min(100% - 2rem, 48rem);
  margin: 2rem auto;
}

.page-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1rem;
}

.page-header h1,
.page-header p {
  margin: 0;
}

.page-header p {
  margin-top: 0.35rem;
  color: #475569;
}

button.add {
  padding: 0.6rem 0.85rem;
  border: 0;
  border-radius: 0.4rem;
  background: #15803d;
  color: white;
  font: inherit;
  cursor: pointer;
}

button.add:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 2px;
}

.task-list {
  display: grid;
  gap: 0.75rem;
  margin: 1rem 0 0;
  padding: 0;
  list-style: none;
}

.empty {
  padding: 1rem;
  color: #475569;
}

@media (max-width: 34rem) {
  .page-header {
    align-items: flex-start;
    flex-direction: column;
  }
}
~~~

If your existing root component uses different filenames or a different class name, keep that setup and add the same imports, parent-owned state, handlers, and template bindings there. Keep the Angular CLI-generated `main.ts` and application configuration wired to your root component.

## 4. Run the app and trace the flow

Start the Angular development server with the script configured by your project, commonly:

~~~sh
ng serve
~~~

Open the local address printed by the CLI.

1. The parent creates the task list and passes each `Task` to a row through `[task]`.
2. Click **Mark complete** in a row. That child emits its task ID through `toggleRequested`.
3. The parent's `toggleTask()` creates an updated array. It does not ask the child to change its input.
4. Angular passes the updated task back to the row. The sibling summary also receives the new completed count.
5. Click **Delete** and observe the parent remove the task and update the summary. Click **Add task** to see the parent create and distribute a new task.

The event bindings are on the components that declare the outputs. Each row reports intent; the parent owns the state transition and decides how the rest of the view changes.

## Expected behavior

- The summary and task rows reflect the same parent-owned task collection.
- A row can request a toggle or deletion, but does not mutate its input.
- The parent updates the collection immutably and distributes the result through inputs.
- Adding, toggling, and deleting tasks updates both the list and the sibling summary.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="027-model-inputs-and-component-level-two-way-binding.md">027 - 27. Model inputs and component-level two-way binding</a></td><td><a href="../lessons/028-parent-child-communication-and-one-way-data-flow.md">Lesson</a></td><td><a href="../qnas/028-parent-child-communication-and-one-way-data-flow.md">Q&amp;A</a></td><td><a href="029-content-projection-and-multiple-slots.md">029 - 29. Content projection and multiple slots</a></td>
</tr>
  </tbody>
</table>
