# 44. Example: OnPush change detection and immutable updates

This standalone Angular example shows why an OnPush child can display stale data when the parent mutates an object and keeps the same reference. It then updates the same task immutably so the parent signal and child input both receive new references.

## What you will build

- An OnPush parent that stores a task list in a signal.
- OnPush task-row components that receive one task as an input signal.
- A button that mutates the first task and writes the same array reference back to the signal.
- A button that publishes the same task change with a new array and task object.
- A computed count that demonstrates how a same-reference signal write does not invalidate its dependents.

## Requirements and setup

Use a recent Angular CLI with standalone component support. Create and run a project:

~~~sh
ng new onpush-identity-demo --standalone --style=css
cd onpush-identity-demo
ng serve
~~~

Replace or add the files below. The generated application already has a root bootstrap configuration; keep the CLI-generated configuration and update the source files.

## 1. Define the task model

Create `src/app/task.model.ts`:

~~~ts
export interface Task {
  id: number;
  title: string;
  done: boolean;
}
~~~

## 2. Create an OnPush task row

Create `src/app/task-row.component.ts`:

~~~ts
import {
  ChangeDetectionStrategy,
  Component,
  input,
} from '@angular/core';
import type { Task } from './task.model';

@Component({
  selector: 'app-task-row',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  templateUrl: './task-row.component.html',
  styleUrl: './task-row.component.css',
})
export class TaskRowComponent {
  readonly task = input.required<Task>();
}
~~~

Create `src/app/task-row.component.html`:

~~~html
<span>{{ task().title }}</span>
<strong [class.done]="task().done">
  {{ task().done ? 'Done' : 'Pending' }}
</strong>
~~~

Create `src/app/task-row.component.css`:

~~~css
:host {
  display: flex;
  flex: 1;
  justify-content: space-between;
  gap: 1rem;
}

.done {
  color: #15803d;
}
~~~

The child reads its input signal in its template. It uses OnPush explicitly so the example behaves consistently in earlier Angular versions as well as Angular v22+, where OnPush is the default.

## 3. Create the parent component

Create `src/app/app.component.ts`:

~~~ts
import {
  ChangeDetectionStrategy,
  Component,
  computed,
  signal,
} from '@angular/core';
import { TaskRowComponent } from './task-row.component';
import type { Task } from './task.model';

function createInitialTasks(): Task[] {
  return [
    { id: 1, title: 'Read the OnPush lesson', done: false },
    { id: 2, title: 'Practice immutable updates', done: false },
  ];
}

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TaskRowComponent],
  changeDetection: ChangeDetectionStrategy.OnPush,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  readonly tasks = signal<Task[]>(createInitialTasks());

  readonly completedCount = computed(
    () => this.tasks().filter((task) => task.done).length,
  );

  mutateFirstInPlace(): void {
    const currentTasks = this.tasks();
    const firstTask = currentTasks[0];

    if (!firstTask) {
      return;
    }

    // This changes the object but preserves both object and array identity.
    firstTask.done = true;
    this.tasks.set(currentTasks);
  }

  markFirstDoneImmutably(): void {
    this.tasks.update((tasks) =>
      tasks.map((task, index) =>
        index === 0 ? { ...task, done: true } : task,
      ),
    );
  }

  reset(): void {
    this.tasks.set(createInitialTasks());
  }
}
~~~

Create `src/app/app.component.html`:

~~~html
<main class="page">
  <h1>OnPush and immutable updates</h1>

  <p class="count">
    Completed tasks: <strong>{{ completedCount() }}</strong>
  </p>

  <div class="controls">
    <button type="button" (click)="mutateFirstInPlace()">
      1. Mutate the first task in place
    </button>

    <button type="button" (click)="markFirstDoneImmutably()">
      2. Publish an immutable update
    </button>

    <button type="button" (click)="reset()">
      Reset
    </button>
  </div>

  <p class="hint">
    Click the first button, observe the stale row, then click the second button.
  </p>

  <ul class="task-list">
    @for (task of tasks(); track task.id) {
      <li class="task-row">
        <app-task-row [task]="task" />
      </li>
    }
  </ul>

  <p class="explanation" aria-live="polite">
    The parent signal and computed count only observe replacement values.
    The child row needs a changed task input reference to refresh.
  </p>
</main>
~~~

Create `src/app/app.component.css`:

~~~css
:host {
  display: block;
  color: #18181b;
  font-family: system-ui, sans-serif;
}

.page {
  width: min(100% - 2rem, 44rem);
  margin: 3rem auto;
}

.count {
  font-size: 1.2rem;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  margin-block: 1.5rem;
}

button {
  padding: 0.65rem 0.9rem;
  border: 1px solid #a1a1aa;
  border-radius: 0.4rem;
  background: white;
  color: inherit;
  cursor: pointer;
}

button:focus-visible {
  outline: 3px solid #2563eb;
  outline-offset: 2px;
}

.hint,
.explanation {
  color: #52525b;
}

.task-row {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  padding: 0.75rem 1rem;
  border: 1px solid #d4d4d8;
  border-radius: 0.5rem;
}

.task-list {
  display: grid;
  gap: 0.75rem;
  padding: 0;
  list-style: none;
}
~~~

## 4. Try the broken update

Start from the reset state, where both tasks are pending and the completed count is zero.

Click **1. Mutate the first task in place**. The click is handled by the parent, so Angular checks the parent's view. The handler changes the first task's done property and calls set() with the same array reference.

The parent view may be checked because it handled the click, but the signal sees the same array identity under its default equality. It therefore sends no signal-change notification. The computed completedCount is not invalidated, so the displayed count remains zero. The task-row child also receives the same task object reference, so its OnPush subtree can be skipped and it continues to display Pending.

This intentionally demonstrates why the parent event can make a mutation look partly visible while a child remains stale. The button event is on the parent, outside the task-row child subtree.

## 5. Apply the immutable update

Now click **2. Publish an immutable update**. The method maps to a new array and creates a new object for the first task. The task signal receives a new array reference and notifies its template consumer. The computed count is invalidated and recalculated. The first task row receives a new input reference and updates to Done.

The second task object was not changed, so it keeps its reference. Angular can skip that OnPush row when its input and other notifications have not changed.

Click **Reset** to restore fresh task objects and repeat the comparison.

## Expected behavior

| Action | Parent signal value | Computed count | First task row |
| --- | --- | --- | --- |
| Initial state | Initial array | 0 | Pending |
| Mutate in place | Same array and task references | Remains 0 because no signal notification occurred | Remains Pending because the child input reference is unchanged |
| Immutable update | New array and new first-task object | 1 | Done |
| Reset | New initial array and task objects | 0 | Pending |

## Why track task.id?

The @for expression uses a stable task.id key so Angular can preserve each rendered row's DOM identity as the list changes. Tracking keys help Angular match rendered items; immutable updates separately provide new object identities for changed inputs. Neither one replaces the purpose of the other.

## Variations

### Use a plain object input

The child can use a conventional input property instead of an input signal. Replace the child field and change the template reads from task() to task:

~~~ts
import { Input } from '@angular/core';
import type { Task } from './task.model';

export class TaskRowComponent {
  @Input({ required: true }) task!: Task;
}
~~~

Update the child HTML to read the ordinary property:

~~~html
<span>{{ task.title }}</span>
<strong>{{ task.done ? 'Done' : 'Pending' }}</strong>
~~~

The identity rule is the same: a parent binding that supplies the same object reference does not count as a changed input.

### Update a nested property

For an object such as `{ id, profile: { name } }`, replace every object along the changed path:

~~~ts
this.users.update((users) =>
  users.map((user) =>
    user.id === userId
      ? {
          ...user,
          profile: {
            ...user.profile,
            name: nextName,
          },
        }
      : user,
  ),
);
~~~

This creates a new users array, a new changed user, and a new profile object. Unchanged users keep their references.

### If an imperative integration owns the update

Prefer updating the state that feeds the template binding. For a programmatically controlled component, use Angular's ComponentRef.setInput() API when setting an input. If an external callback updates a plain field, mark the view for a future check with ChangeDetectorRef.markForCheck().

## Common mistakes

- Mutating the task but keeping the same object reference.
- Calling signal.set() with the same mutated array and expecting a signal notification.
- Copying the user object but mutating its nested profile in place.
- Assuming that checking the parent forces each OnPush child to refresh.
- Confusing @for tracking identity with input reference changes.
- Using detectChanges() in every update handler instead of fixing the state and notification flow.

## Key takeaways

- OnPush can skip an unchanged child subtree even when its parent is checked.
- A signal with default equality does not notify consumers when set() receives the same reference.
- Immutable updates create new references for the collection and the changed object path.
- Unchanged child inputs can keep their references so Angular can skip those rows.
- Stable @for keys preserve DOM identity; immutable references communicate state changes.

## Official references

- [Skipping component subtrees](https://angular.dev/best-practices/skipping-subtrees)
- [Advanced component configuration](https://angular.dev/guide/components/advanced-configuration)
- [Angular Signals](https://angular.dev/guide/signals)
- [Angular input() API](https://angular.dev/api/core/input)
