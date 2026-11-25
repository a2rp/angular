# Examples: Render Collections with for, track, and empty

This standalone Angular component displays a task collection. You can mark tasks complete, remove them, add another task, reorder the list, and clear it to see @empty. The @for block tracks each task by a stable ID.

Adapt file names to the Angular workspace created in the setup lessons.

## 1. Create the component class

Create src/app/task-list.ts:

~~~ts
import { Component } from '@angular/core';

interface Task {
  id: number;
  title: string;
  done: boolean;
}

@Component({
  selector: 'app-task-list',
  standalone: true,
  templateUrl: './task-list.html',
  styleUrl: './task-list.css',
})
export class TaskList {
  tasks: Task[] = [
    { id: 1, title: 'Read the lesson', done: true },
    { id: 2, title: 'Try the example', done: false },
    { id: 3, title: 'Review the Q&A', done: false },
  ];

  private nextTaskId = 4;

  get completedCount(): number {
    return this.tasks.filter((task) => task.done).length;
  }

  addTask(): void {
    const task: Task = {
      id: this.nextTaskId++,
      title: 'New practice task',
      done: false,
    };

    this.tasks = [...this.tasks, task];
  }

  toggleComplete(taskId: number): void {
    this.tasks = this.tasks.map((task) =>
      task.id === taskId
        ? { ...task, done: !task.done }
        : task
    );
  }

  removeTask(taskId: number): void {
    this.tasks = this.tasks.filter((task) => task.id !== taskId);
  }

  moveFirstTaskToEnd(): void {
    const firstTask = this.tasks[0];

    if (this.tasks.length < 2 || !firstTask) {
      return;
    }

    this.tasks = [...this.tasks.slice(1), firstTask];
  }

  clearTasks(): void {
    this.tasks = [];
  }
}
~~~

Each task has a unique id. The update methods replace the array while preserving IDs for tasks that still represent the same items.

## 2. Add the template

Create src/app/task-list.html:

~~~html
<section class="task-panel" aria-labelledby="task-list-title">
  <h2 id="task-list-title">Practice tasks</h2>

  <p>Completed: {{ completedCount }} of {{ tasks.length }}</p>

  <div class="actions" role="group" aria-label="Task list actions">
    <button type="button" (click)="addTask()">Add task</button>

    <button
      type="button"
      (click)="moveFirstTaskToEnd()"
      [disabled]="tasks.length < 2"
    >
      Move first task to the end
    </button>

    <button
      type="button"
      (click)="clearTasks()"
      [disabled]="tasks.length === 0"
    >
      Clear all tasks
    </button>
  </div>

  <ul class="task-list">
    @for (
      task of tasks;
      track task.id;
      let index = $index, total = $count, first = $first, last = $last
    ) {
      <li [class.is-complete]="task.done">
        <div class="task-details">
          <label>
            <input
              type="checkbox"
              [checked]="task.done"
              (change)="toggleComplete(task.id)"
            >
            <span>{{ task.title }}</span>
          </label>

          <p>
            Task {{ index + 1 }} of {{ total }}.
            @if (first) {
              <span class="position-marker">First</span>
            }
            @if (last) {
              <span class="position-marker">Last</span>
            }
          </p>

          <p>{{ task.done ? 'Complete' : 'Not complete' }}</p>
        </div>

        <button
          type="button"
          class="remove-button"
          (click)="removeTask(task.id)"
        >
          Remove {{ task.title }}
        </button>
      </li>
    } @empty {
      <li class="empty-state">
        No tasks yet. Add a task to get started.
      </li>
    }
  </ul>
</section>
~~~

The collection is rendered in its current order. Each row uses task.id to keep its identity as the array changes. The let segment gives readable names to the index, count, first-item flag, and last-item flag.

When tasks becomes empty, Angular displays the @empty content. The checkboxes and buttons use native controls, while their event handlers update the component collection.

## 3. Add component styles

Create src/app/task-list.css:

~~~css
:host {
  display: block;
  max-width: 48rem;
  color: #1f2937;
}

.task-panel {
  padding: 1.25rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: #fff;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
  margin-block: 1rem;
}

button {
  padding: 0.55rem 0.8rem;
  font: inherit;
  cursor: pointer;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

.task-list {
  display: grid;
  gap: 0.65rem;
  padding: 0;
  list-style: none;
}

.task-list li {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 0.8rem;
  border: 1px solid #e2e8f0;
  border-radius: 0.5rem;
}

.task-details p {
  margin-block: 0.35rem 0;
  color: #475569;
}

.task-details label {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
}

.is-complete .task-details label span {
  text-decoration: line-through;
}

.position-marker {
  margin-inline-start: 0.35rem;
  font-weight: 600;
}

.remove-button {
  flex: none;
}

.empty-state {
  color: #475569;
  text-align: center;
}

@media (max-width: 32rem) {
  .task-list li {
    align-items: flex-start;
    flex-direction: column;
  }
}
~~~

## 4. Render the component

Import TaskList into the root component and place its selector in the root template. For example, if the root component is src/app/app.ts:

~~~ts
import { Component } from '@angular/core';
import { TaskList } from './task-list';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TaskList],
  template: '<app-task-list />',
})
export class App {}
~~~

If your root component already has imports, providers, or a separate template, preserve them and add TaskList and app-task-list.

Start the application:

~~~sh
ng serve
~~~

## 5. Try the collection updates

1. Toggle a task's checkbox. The task object and array are replaced, but its id stays the same.
2. Remove a task. Other rows keep their identities even though their indexes change.
3. Move the first task to the end. The visible order changes while each task keeps its ID.
4. Add a task. It receives a new unique ID.
5. Clear all tasks. The @empty block appears.
6. Add a task after clearing. The collection renders again and the empty block disappears.

## 6. Observe why tracking matters

The track task.id expression uses logical task identity. When a task's done property changes, its id stays stable, so Angular can reuse that task's view and update its bindings. When the list is reordered, Angular can move the matching views rather than treating every position as a new item.

For a fixed list that never changes, track $index can be appropriate. Avoid switching this example to index tracking: adding, removing, and moving tasks changes their positions.

## 7. Practice variations

1. Add a priority property and show it inside each row without changing the tracking key.
2. Add a filter in component code and display the filtered iterable with @for.
3. Add first and last styling using the existing contextual aliases.
4. Create two tasks with the same title but different IDs; both should render as separate items.
5. Temporarily use a non-unique title as the tracking key and inspect Angular's duplicate-key diagnostic. Restore task.id afterward.
6. Add a loading or error state outside the loop. Keep those states distinct from the empty-list state.
7. Build a nested collection and alias the outer and inner indexes with different names.

Do not generate IDs during template rendering. Create an item's stable identifier when the item is created, then keep it unchanged while that item remains in the collection.
