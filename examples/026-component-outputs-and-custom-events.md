# Examples: Component outputs and custom events

Build a small task editor that emits two events to its parent. Saving emits a typed task object; cancelling emits no payload. The parent listens to both outputs, assigns the saved task an ID, and owns the displayed list.

This example uses standalone components and Angular's built-in control-flow syntax. Keep your Angular CLI-generated bootstrap and application configuration; connect the root component shown here using the setup already in your project.

## 1. Create the child task editor

Create `src/app/task-editor.ts`:

~~~ts
import { Component, output } from '@angular/core';

export type TaskPriority = 'normal' | 'high';

export interface TaskDraft {
  title: string;
  priority: TaskPriority;
}

@Component({
  selector: 'app-task-editor',
  standalone: true,
  templateUrl: './task-editor.html',
  styleUrl: './task-editor.css',
})
export class TaskEditor {
  readonly taskSaved = output<TaskDraft>();
  readonly cancelled = output<void>();
  readonly draftTitle = 'Review component outputs';

  saveTask(priority: TaskPriority): void {
    this.taskSaved.emit({
      title: this.draftTitle,
      priority,
    });
  }

  cancel(): void {
    this.cancelled.emit();
  }
}
~~~

Create `src/app/task-editor.html`:

~~~html
<section class="editor" aria-labelledby="editor-title">
  <h2 id="editor-title">Task editor</h2>
  <p>Draft: <strong>{{ draftTitle }}</strong></p>
  <p>Choose a priority to emit a task object to the parent.</p>

  <div class="actions">
    <button type="button" (click)="saveTask('normal')">
      Save as normal priority
    </button>
    <button type="button" (click)="saveTask('high')">
      Save as high priority
    </button>
    <button class="secondary" type="button" (click)="cancel()">
      Cancel
    </button>
  </div>
</section>
~~~

Create `src/app/task-editor.css`:

~~~css
.editor {
  padding: 1rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: #f8fafc;
}

.editor h2 {
  margin-top: 0;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

button {
  padding: 0.55rem 0.8rem;
  border: 0;
  border-radius: 0.4rem;
  background: #1d4ed8;
  color: white;
  font: inherit;
  cursor: pointer;
}

button.secondary {
  background: #475569;
}

button:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 2px;
}
~~~

`taskSaved` carries a `TaskDraft`, so each emitted value includes a title and a priority. `cancelled` is `output<void>()`, so its `.emit()` call carries no data.

## 2. Listen to the outputs in the parent

Create or adapt your root component at `src/app/app.ts`:

~~~ts
import { Component } from '@angular/core';
import { TaskEditor, type TaskDraft } from './task-editor';

interface SavedTask extends TaskDraft {
  id: number;
}

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TaskEditor],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class AppComponent {
  savedTasks: SavedTask[] = [];
  notice = '';
  private nextId = 1;

  addTask(draft: TaskDraft): void {
    const task: SavedTask = { ...draft, id: this.nextId++ };
    this.savedTasks = [...this.savedTasks, task];
    this.notice = 'Saved task: ' + task.title;
  }

  handleCancel(): void {
    this.notice = 'The draft was cancelled.';
  }
}
~~~

Create `src/app/app.html`:

~~~html
<main class="page">
  <h1>Component output demo</h1>
  <p>
    The editor emits events. This parent decides how to update its own state.
  </p>

  <app-task-editor
    (taskSaved)="addTask($event)"
    (cancelled)="handleCancel()"
  />

  <p class="notice" role="status" aria-live="polite">{{ notice }}</p>

  <section aria-labelledby="saved-heading">
    <h2 id="saved-heading">Saved tasks</h2>

    @if (savedTasks.length === 0) {
      <p>No tasks saved yet. Use one of the save buttons above.</p>
    } @else {
      <ul>
        @for (task of savedTasks; track task.id) {
          <li>
            <strong>{{ task.title }}</strong>
            <span> — {{ task.priority }} priority</span>
          </li>
        }
      </ul>
    }
  </section>
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

.notice {
  min-height: 1.5em;
  color: #1d4ed8;
}

li + li {
  margin-top: 0.5rem;
}
~~~

If your existing root component uses a different class or file name, keep that setup and apply the same imports, properties, handlers, and template bindings there. The important parent bindings are `(taskSaved)="addTask($event)"` and `(cancelled)="handleCancel()"`.

## 3. Run the app and observe the events

Start the Angular development server using the script configured by your project, commonly:

~~~sh
ng serve
~~~

Open the local address printed by the CLI. Choose either save button: the child emits a `TaskDraft`, Angular passes it to the parent as `$event`, and the parent adds a task to its list. Choose Cancel: the child emits a payload-free event and the parent updates its status message.

The child does not modify the parent's list directly. It reports what happened, and the parent owns the response. The outputs are bound on the `<app-task-editor>` component instance; they are Angular component events and do not bubble through the DOM.

## 4. Try these variations

- Change `draftTitle` and save again to see the new payload in the parent list.
- Add a field such as `dueDate` to `TaskDraft` and include it in the emitted object.
- Rename `taskSaved` to `taskCreated` in both the child property and the parent's template binding.
- Temporarily add `{ alias: 'taskCreated' }` to `output<TaskDraft>()`; keep the TypeScript property name and bind to the alias in the template.
- Add a nested wrapper component and explicitly listen to and re-emit the child's output if the wrapper's parent needs to receive it.
- Inspect the emitted event in the parent handler or a debugger, rather than reaching into the child's internal state.

## Expected behavior

- Saving emits one typed task object with a title and priority.
- The parent assigns an ID and renders the saved task.
- Cancelling emits no payload and updates the parent status message.
- The child remains reusable because it announces events without owning the parent's list.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="025-signal-inputs-required-inputs-and-input-transforms.md">025 - 25. Signal inputs, required inputs, and input transforms</a></td><td><a href="../lessons/026-component-outputs-and-custom-events.md">Lesson</a></td><td><a href="../qnas/026-component-outputs-and-custom-events.md">Q&amp;A</a></td><td><a href="027-model-inputs-and-component-level-two-way-binding.md">027 - 27. Model inputs and component-level two-way binding</a></td>
</tr>
  </tbody>
</table>
