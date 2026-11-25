# Angular DevTools, Debugging, and Production Diagnostics — Examples

These examples provide a small Angular screen to inspect, then show how to trace a runtime problem or performance issue. Adapt imports and file paths to your workspace. The snippets assume a modern standalone Angular application.

## 1. Create a component with inspectable state

~~~ts
import { ChangeDetectionStrategy, Component, computed, signal } from '@angular/core';

type Task = {
  id: number;
  title: string;
  done: boolean;
};

@Component({
  selector: 'app-task-list',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: @@BT@@
    <section aria-labelledby="tasks-heading">
      <h1 id="tasks-heading">Tasks</h1>

      <label for="task-filter">Filter tasks</label>
      <input
        id="task-filter"
        type="search"
        [value]="query()"
        (input)="setQuery($event)"
      />

      <button type="button" (click)="addTask()">Add sample task</button>

      <p aria-live="polite">{{ visibleTasks().length }} tasks shown</p>

      @if (visibleTasks().length > 0) {
        <ul>
          @for (task of visibleTasks(); track task.id) {
            <li>
              <span>{{ task.title }}</span>
              <button
                type="button"
                [attr.aria-label]="'Mark ' + task.title + ' complete'"
                (click)="toggleTask(task.id)"
              >
                {{ task.done ? 'Completed' : 'Mark complete' }}
              </button>
            </li>
          }
        </ul>
      } @else {
        <p>No matching tasks.</p>
      }
    </section>
  @@BT@@,
})
export class TaskListComponent {
  readonly query = signal('');
  readonly tasks = signal<Task[]>([
    { id: 1, title: 'Review the profiler recording', done: false },
    { id: 2, title: 'Inspect the failed network request', done: true },
  ]);

  private nextId = 3;

  readonly visibleTasks = computed(() => {
    const normalizedQuery = this.query().trim().toLowerCase();

    return this.tasks().filter((task) =>
      task.title.toLowerCase().includes(normalizedQuery),
    );
  });

  setQuery(event: Event): void {
    const input = event.target as HTMLInputElement;
    this.query.set(input.value);
  }

  addTask(): void {
    const id = this.nextId++;
    this.tasks.update((tasks) => [
      ...tasks,
      { id, title: 'New task ' + id, done: false },
    ]);
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

This component is useful for inspection because it has explicit signal state, a computed value, user events, and a repeated list. In Angular DevTools, select TaskListComponent and inspect its state. Change the filter and toggle a task to see how the component updates. In the browser's Elements panel, verify the resulting DOM and accessible names.

## 2. Record one profiler interaction

1. Run the application with the Angular CLI development server.
2. Open the page in Chrome or Firefox and open developer tools.
3. Select the Angular DevTools Profiler and start a recording.
4. Type a short filter, add one task, and mark one task complete.
5. Stop recording and select the cycles around those interactions.
6. Inspect the component details and look for unexpected repeated work.
7. Repeat the same interaction after a code change.

The example is intentionally small, so the profiler may show no meaningful bottleneck. Use it to learn how to record and inspect a cycle rather than to infer production performance from a tiny sample.

## 3. Capture framework events in Chrome Performance

When the app needs Angular startup events in a Chrome Performance recording, enable profiling before bootstrapping:

~~~ts
import { enableProfiling } from '@angular/core';
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

enableProfiling();

bootstrapApplication(AppComponent, appConfig).catch((error: unknown) => {
  console.error('Application bootstrap failed', error);
});
~~~

Open Chrome DevTools, go to **Performance**, start recording, and perform a short interaction. Inspect the Angular track alongside browser tasks. Profiling integration is for development diagnostics. Avoid shipping profiling setup as a substitute for production monitoring.

## 4. Trace an event with a breakpoint

To diagnose an unexpected task update:

1. Set a breakpoint inside toggleTask.
2. Click **Mark complete**.
3. Inspect the id parameter and the current task array.
4. Step through the update callback and confirm the returned array.
5. Resume and inspect the rendered DOM.
6. Remove the breakpoint after confirming the state transition.

Prefer a breakpoint at the state owner over adding logs to every template expression. If you temporarily add a log, keep it narrow and remove it once the cause is understood.

## 5. Check a failed HTTP request

For an application that loads tasks from a server:

1. Open the browser's **Network** panel and enable **Preserve log**.
2. Trigger the load action.
3. Select the request and inspect its URL, method, status, timing, and response.
4. Compare the request with the server contract and application environment.
5. Check the Console for the first related error.

A 401/403 may indicate an authentication or authorization issue; a 404 may indicate a route or base URL mismatch; a 5xx response usually requires checking the server. The exact meaning depends on the API. Do not paste real tokens or personal request payloads into a public issue.

## 6. Add a narrow global reporting boundary

A custom ErrorHandler can report unexpected errors that Angular forwards to it:

~~~ts
import { ErrorHandler, Injectable } from '@angular/core';

@Injectable()
export class AppErrorHandler implements ErrorHandler {
  handleError(error: unknown): void {
    // Replace this with a privacy-conscious monitoring integration.
    // Do not include secrets, access tokens, or full user payloads.
    console.error('Unexpected Angular error', error);
  }
}
~~~

Register it in the application configuration:

~~~ts
import { ApplicationConfig, ErrorHandler } from '@angular/core';
import { AppErrorHandler } from './app-error-handler';

export const appConfig: ApplicationConfig = {
  providers: [
    { provide: ErrorHandler, useClass: AppErrorHandler },
  ],
};
~~~

This boundary does not remove the need to handle expected errors where they occur. A request handler, form submission, or user action usually has the context needed to show a useful recovery message.

## 7. Enable strict template checking and one diagnostic

A project can enable strict template checking and configure an individual extended diagnostic in its TypeScript configuration:

~~~json
{
  "angularCompilerOptions": {
    "strictTemplates": true,
    "extendedDiagnostics": {
      "checks": {
        "interpolatedSignalNotInvoked": "error"
      }
    }
  }
}
~~~

The exact available diagnostics depend on the Angular version. Review the current diagnostic name and default behavior before adding a check. Avoid setting every diagnostic to an error without considering how new checks can affect later framework upgrades.

## 8. Keep source maps private when appropriate

For a local diagnostic build, enable source maps in the build target of angular.json:

~~~json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "configurations": {
            "debug": {
              "sourceMap": true
            }
          }
        }
      }
    }
  }
}
~~~

The project and target names may differ. Use the configuration with the CLI, for example:

~~~sh
ng build --configuration debug
~~~

For production error monitoring, consider uploading source maps to a private service and excluding them from the public static files. Confirm the deployed JavaScript and uploaded map belong to the same build.

## 9. Use a concise debugging note

Record observations in a small, repeatable format:

~~~text
Issue:
Route and app version:
Browser and operating system:
Steps to reproduce:
Expected behavior:
Actual behavior:
First relevant console error:
Relevant request and status:
Angular component or service inspected:
Profile or breakpoint finding:
Fix and verification:
Sensitive data removed:
~~~

Keep the note focused on the evidence that helped isolate the issue. Redact user identifiers, credentials, and private URLs before sharing it.

## References

- [Angular DevTools](https://angular.dev/tools/devtools)
- [Angular DevTools Profiler](https://angular.dev/tools/devtools/profiler)
- [Angular signals in DevTools](https://angular.dev/tools/devtools/signals)
- [Profiling with Chrome DevTools](https://angular.dev/best-practices/profiling-with-chrome-devtools)
- [Angular error handling](https://angular.dev/best-practices/error-handling)
- [Extended diagnostics](https://angular.dev/extended-diagnostics)