# Examples: Event Bindings, Event Objects, and Keyboard Input

This standalone Angular component demonstrates input, submit, click, and keyboard event bindings. It uses a native form and buttons so their normal keyboard behavior works without custom key handling.

Adapt the file names to the Angular workspace created in the setup lessons.

## 1. Create the component class

Create src/app/task-event-demo.ts:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-task-event-demo',
  standalone: true,
  templateUrl: './task-event-demo.html',
  styleUrl: './task-event-demo.css',
})
export class TaskEventDemo {
  draft = '';
  taskCount = 0;
  lastAddedTask = '';
  message = 'Enter a task to get started.';

  updateDraft(event: Event): void {
    const target = event.currentTarget;

    if (target instanceof HTMLInputElement) {
      this.draft = target.value;
      this.message = 'Draft updated.';
    }
  }

  addTask(event: SubmitEvent): void {
    event.preventDefault();

    const task = this.draft.trim();

    if (task.length === 0) {
      this.message = 'Enter a task before adding it.';
      return;
    }

    this.taskCount += 1;
    this.lastAddedTask = task;
    this.draft = '';
    this.message = 'Task added: ' + task;
  }

  clearDraft(): void {
    this.draft = '';
    this.message = 'Draft cleared.';
  }
}
~~~

updateDraft accepts a general Event and checks currentTarget before reading the input's value. addTask accepts the submit event and prevents the browser's default form navigation because this demo handles submission itself.

## 2. Add the template

Create src/app/task-event-demo.html:

~~~html
<section class="task-panel" aria-labelledby="task-panel-title">
  <h2 id="task-panel-title">Quick task entry</h2>

  <form (submit)="addTask($event)">
    <label for="task-draft">Task name</label>

    <input
      id="task-draft"
      type="text"
      [value]="draft"
      (input)="updateDraft($event)"
      (keydown.escape)="clearDraft()"
      aria-describedby="task-help"
    />

    <p id="task-help">
      Press Enter to add the task. Press Escape to clear the draft.
    </p>

    <div class="actions">
      <button type="submit" [disabled]="draft.length === 0">
        Add task
      </button>

      <button type="button" (click)="clearDraft()">
        Clear draft
      </button>
    </div>
  </form>

  <p role="status">{{ message }}</p>
  <p>Tasks added: {{ taskCount }}</p>
  <p>Most recently added: {{ lastAddedTask || 'None yet' }}</p>
</section>
~~~

The template demonstrates four listeners:

- (input) passes $event so the component can read the edited value.
- (keydown.escape) filters keyboard input to the Escape key.
- (submit) passes the native SubmitEvent to the handler.
- (click) calls the clear action from a native button.

The Add task button submits the form. Pressing Enter in the text input uses the browser's native form behavior, which also raises submit. The listener cancels the page navigation and processes the task.

The text input uses [value] and (input), so the value flows from the component to the DOM and user edits flow back through the event handler. This is a one-way property binding plus an event listener, not two-way binding.

## 3. Add component styles

Create src/app/task-event-demo.css:

~~~css
:host {
  display: block;
  max-width: 34rem;
}

.task-panel {
  padding: 1.25rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  color: #1e293b;
  background: #fff;
}

label {
  display: block;
  margin-block-end: 0.35rem;
  font-weight: 600;
}

input {
  box-sizing: border-box;
  width: 100%;
  padding: 0.65rem;
  font: inherit;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.6rem;
  margin-block: 0.75rem;
}

button {
  padding: 0.55rem 0.85rem;
  font: inherit;
  cursor: pointer;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

#task-help {
  margin-block: 0.4rem;
  color: #475569;
  font-size: 0.9rem;
}
~~~

## 4. Render the component

Import TaskEventDemo into the root component and add its selector to the root template. For example, if the root component is src/app/app.ts:

~~~ts
import { Component } from '@angular/core';
import { TaskEventDemo } from './task-event-demo';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TaskEventDemo],
  template: '<app-task-event-demo />',
})
export class App {}
~~~

If the root component already has imports or a separate template, keep those and add TaskEventDemo and app-task-event-demo.

Start the app with:

~~~sh
ng serve
~~~

## 5. Try the interactions

1. Type in the task field. The input event updates draft and the status message.
2. Press Escape while the field is focused. The draft clears.
3. Type a task and press Enter. The native form submits and addTask handles the SubmitEvent.
4. Type a task and activate Add task with the mouse, Enter, or Space. The native submit button submits the form.
5. Activate Clear draft with the mouse or keyboard. Its click handler clears the input.
6. Submit an empty or whitespace-only value to see the component's validation message.

The two buttons use native button semantics. No custom Enter or Space key listener is needed to make them keyboard-operable.

## 6. Inspect the event object

Set a breakpoint inside updateDraft and inspect its event parameter:

- currentTarget is the input that owns the input listener.
- target is the element where the event originated.
- currentTarget is checked as an HTMLInputElement before reading value.

Set a breakpoint in addTask and inspect the SubmitEvent. Calling preventDefault() avoids the browser's normal form navigation while the component processes the submission.

## 7. Variations to practice

1. Change the shortcut from Escape to another supported key and update the help text.
2. Add an explicit KeyboardEvent parameter to a key handler and inspect key, code, and repeat.
3. Use a control-plus-Enter modifier on a textarea to practice a key combination without changing the form's normal Enter behavior.
4. Add a parent click listener and inspect how target and currentTarget differ when a child is clicked.
5. Add stopPropagation() to a deliberate nested-event example and compare it with preventDefault(). Remove it afterward so normal bubbling remains clear.
6. Try the same task action with a div and compare the extra keyboard and accessibility work. Keep the native button in the finished example.

Do not store event objects for later use. Read what the handler needs while it is running, and keep default browser behavior unless the application is intentionally replacing it.
