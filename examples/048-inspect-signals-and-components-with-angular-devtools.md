# Inspect signals and components with Angular DevTools — examples

This small standalone application provides a repeatable interaction for exploring the Angular DevTools Components tab, signal graph, and Profiler. It renders a list of tasks, filters them, and lets a row request a state change from its parent.

## File layout

~~~text
src/
├── main.ts
└── app/
    ├── app.component.ts
    └── task-row.component.ts
~~~

## 1. Create a task row component

~~~ts
// src/app/task-row.component.ts
import {
  ChangeDetectionStrategy,
  Component,
  input,
  output,
} from '@angular/core';

export interface Task {
  readonly id: number;
  readonly title: string;
  readonly done: boolean;
}

@Component({
  selector: 'app-task-row',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <article>
      <span>{{ task().title }}</span>

      @if (task().done) {
        <span>Complete</span>
      } @else {
        <span>Open</span>
      }

      <button type="button" (click)="toggle.emit(task().id)">
        {{ task().done ? 'Reopen' : 'Complete' }}
      </button>
    </article>
  `,
})
export class TaskRowComponent {
  readonly task = input.required<Task>();
  readonly toggle = output<number>();
}
~~~

The row receives its task through an input and emits the task ID when the user clicks its button. The row does not own a second copy of the task.

## 2. Create the parent with signals and computed values

~~~ts
// src/app/app.component.ts
import { Component, computed, signal } from '@angular/core';
import { Task, TaskRowComponent } from './task-row.component';

type TaskFilter = 'all' | 'open' | 'done';

const initialTasks: readonly Task[] = Array.from(
  { length: 120 },
  (_, index) => ({
    id: index + 1,
    title: 'Task ' + (index + 1),
    done: index % 3 === 0,
  }),
);

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TaskRowComponent],
  template: `
    <main>
      <h1>Task dashboard</h1>

      <p>{{ completedCount() }} of {{ tasks().length }} tasks complete</p>

      <section aria-label="Task filters">
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
      </section>

      <section aria-label="Tasks">
        @for (task of visibleTasks(); track task.id) {
          <app-task-row
            [task]="task"
            (toggle)="toggleTask($event)"
          />
        } @empty {
          <p>No tasks match this filter.</p>
        }
      </section>
    </main>
  `,
})
export class AppComponent {
  readonly tasks = signal<readonly Task[]>(initialTasks);
  readonly filter = signal<TaskFilter>('all');

  readonly completedCount = computed(
    () => this.tasks().filter((task) => task.done).length,
  );

  readonly visibleTasks = computed(() => {
    const filter = this.filter();

    return this.tasks().filter((task) => {
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

  toggleTask(id: number): void {
    this.tasks.update((tasks) =>
      tasks.map((task) =>
        task.id === id ? { ...task, done: !task.done } : task,
      ),
    );
  }
}
~~~

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

Run the standalone app with the Angular CLI development server, then open its local page in a supported browser.

## 3. Inspect the component tree and properties

1. Open the browser's developer tools and select the Angular tab.
2. In Components, select the root app component.
3. Inspect its tasks, filter, completed count, and visible task values in the properties panel.
4. Expand the tree and select an app-task-row instance.
5. Inspect its task input and compare its ID with the corresponding item in the parent.
6. Use the inspect-element control to select a visible row from the page.
7. Use host-node navigation to connect the selected Angular row to its DOM element.

The 120 generated tasks make the repeated row instances easy to find. They are for exploration, not a performance target. The app's state still has one authoritative task array in the parent.

## 4. Trace the signal graph

Select the root app component and open its signal graph. Look for:

- The tasks signal as a source.
- The filter signal as a source for the visible list.
- The completedCount computation, which reads tasks.
- The visibleTasks computation, which reads tasks and filter.
- The component template as a consumer of the values it renders.

Click the Open filter. The visible list changes because filter changes, while the completed count still describes all tasks. Click Complete on a row. The parent maps the task array to a replacement array and replaces only the changed task object. The computations that depend on tasks become relevant to the new value.

Then select a row component and inspect its task input and signal graph if available. The row's displayed state comes from its input rather than from a separate local task copy.

A computed dependency is dynamic. For example, visibleTasks reads both filter and tasks, so both participate in its current dependency graph. Reproduce each state you want to inspect before drawing a conclusion about conditional dependencies.

## 5. Record a profile for a repeatable interaction

1. Open the Profiler and start recording.
2. Click Open or Done, then click Complete on a task.
3. Stop recording and select the cycles caused by those interactions.
4. Inspect the component chart or flame graph to see where Angular spent time.
5. Compare the cost of the root component and repeated task rows.
6. Save the profile as JSON if you need to revisit the same recording.

This modest example may complete quickly, so the recorded durations can be small. A tiny duration is not evidence of a performance problem. Use the profiler on a real interaction and realistic data before deciding that code needs optimization.

If you increase the generated task count for experimentation, do so only enough to make the interface manageable and the measurement repeatable. Avoid treating synthetic row counts as a substitute for profiling the real application.

## 6. Use the other runtime trees when investigating scope

The sample keeps its signals in the root component, so it is most useful for the Components, signal graph, and Profiler panels. In an application that uses services, select Injector Tree to inspect which injector provides a service and whether separate component providers created separate instances. Use Router Tree to inspect the configured route hierarchy.

## What to look for

- Does the component tree match the components you expect the page to render?
- Does the row input match the parent task with the same stable ID?
- Does changing the filter update the visible list without changing the total count?
- Does toggling a row update one task through an immutable replacement?
- Does the signal graph show the expected sources and consumers?
- Does the profiler identify a measurable slow component in a real interaction, or are the recorded costs negligible?

DevTools gives runtime evidence for these questions. The source code and the reproduced user interaction explain why the runtime state changed.
