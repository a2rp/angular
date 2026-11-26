# Examples: Classes, Modules, Decorators, and Compiler Options

This walkthrough builds a small task counter and task list with plain TypeScript modules, then connects them to an Angular standalone component. It demonstrates where JavaScript classes, TypeScript-only contracts, Angular decorators, and compiler settings each belong.

## 1. Create a task contract

Create **src/app/models/task.ts**:

~~~ts
export type TaskStatus = 'open' | 'done';

export interface Task {
  readonly id: number;
  title: string;
  status: TaskStatus;
}
~~~

The interface and union type help TypeScript check callers. They do not create runtime objects or validate data received from a server.

## 2. Add a runtime class

Create **src/app/models/task-counter.ts**:

~~~ts
export class TaskCounter {
  private total = 0;

  get count(): number {
    return this.total;
  }

  add(): void {
    this.total += 1;
  }

  reset(): void {
    this.total = 0;
  }
}
~~~

TaskCounter exists in the emitted JavaScript. Its private modifier prevents ordinary TypeScript callers from accessing total, while its getter exposes the count. This TypeScript private field is not a security boundary.

## 3. Keep state behavior in a small service

Create **src/app/services/task-store.ts**:

~~~ts
import { Injectable } from '@angular/core';
import type { Task } from '../models/task';

@Injectable({ providedIn: 'root' })
export class TaskStore {
  private tasks: Task[] = [
    { id: 1, title: 'Read the lesson', status: 'open' },
    { id: 2, title: 'Try the example', status: 'done' },
  ];

  getAll(): readonly Task[] {
    return this.tasks;
  }

  add(title: string): void {
    const nextId = this.tasks.reduce(
      (largest, task) => Math.max(largest, task.id),
      0,
    ) + 1;

    this.tasks = [...this.tasks, { id: nextId, title, status: 'open' }];
  }

  complete(id: number): void {
    this.tasks = this.tasks.map((task) =>
      task.id === id ? { ...task, status: 'done' } : task,
    );
  }
}
~~~

The class contains runtime behavior and is decorated with Angular's Injectable metadata so Angular's dependency injection system can provide it. The Task import is type-only because it is used only for checking. The returned readonly array prevents callers from using that reference to push or remove entries; the task objects themselves are not deeply immutable.

## 4. Use a component decorator

Create **src/app/task-board.component.ts**:

~~~ts
import { Component, inject } from '@angular/core';
import { TaskCounter } from './models/task-counter';
import { TaskStore } from './services/task-store';

@Component({
  selector: 'app-task-board',
  standalone: true,
  templateUrl: './task-board.component.html',
})
export class TaskBoardComponent {
  readonly counter = new TaskCounter();
  readonly taskStore = inject(TaskStore);

  addExampleTask(): void {
    this.taskStore.add('Practice classes and decorators');
  }
}
~~~

Create the external template at **src/app/task-board.component.html**:

~~~html
<section aria-labelledby="task-heading">
  <h2 id="task-heading">Task board</h2>

  <p>Counter: {{ counter.count }}</p>
  <button type="button" (click)="counter.add()">Add to counter</button>
  <button type="button" (click)="counter.reset()">Reset counter</button>

  <ul>
    @for (task of taskStore.getAll(); track task.id) {
      <li>
        {{ task.title }} — {{ task.status }}
        @if (task.status === 'open') {
          <button type="button" (click)="taskStore.complete(task.id)">
            Mark done
          </button>
        }
      </li>
    }
  </ul>

  <button type="button" (click)="addExampleTask()">Add example task</button>
</section>
~~~

The @Component decorator associates Angular metadata with the class. The component is standalone, so it does not need to be declared in an NgModule. Its template uses Angular control-flow syntax supported by current Angular versions. If an older workspace is being used, check that version's template syntax and adapt the loop accordingly.

To render this component, import it into the standalone root component or route and place the app-task-board element in the parent template. If the application was generated with a different structure, follow its existing bootstrap and routing setup.

## 5. Compare the pieces

| Piece | Exists at runtime? | What it does here |
| --- | --- | --- |
| Task interface | No | Describes the object shape checked by TypeScript |
| TaskStatus type | No | Limits valid status values during type checking |
| TaskCounter class | Yes | Holds a small counter and exposes methods |
| TaskStore class | Yes | Owns task data and operations |
| @Injectable | Angular metadata | Lets Angular provide the service |
| @Component | Angular metadata | Connects the component class to its template |
| ECMAScript import/export | Yes, as module structure | Shares code between source files |

TypeScript types catch many mistakes while editing and building, but they do not validate arbitrary JSON at runtime. Validate external data before treating it as a Task.

## 6. Review compiler configuration

Angular CLI workspaces contain generated TypeScript and Angular compiler configuration. Read the existing files before editing them. A simplified illustration of the relevant option sections is:

~~~json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitOverride": true
  },
  "angularCompilerOptions": {
    "strictTemplates": true
  }
}
~~~

This is an illustration of option placement, not a replacement tsconfig for an Angular workspace. Real CLI projects may extend shared configurations and require additional options for decorators, module resolution, builds, or tests. Keep the configuration generated for the project's Angular and TypeScript versions. In particular, do not toggle decorator settings based on a generic TypeScript-only example.

## 7. Try small modifications

1. Add a description property to Task and initialize it in both initial tasks and newly added tasks. Observe how the type checker identifies incomplete objects.
2. Add a remove(id: number) method to TaskStore using filter, and add a button that calls it.
3. Make the title read-only and update the completion method so it still changes only the status.
4. Rename a component method, then update the template call. Strict template checking helps catch a stale method name.
5. Temporarily pass a string where a number is required. Read the diagnostic and correct the call without disabling strict mode.
6. Add a subclass of TaskCounter that overrides a method. Add the override keyword and inspect how the compiler checks the relationship.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="005-typescript-interfaces-unions-generics-and-narrowing.md">005 - TypeScript Interfaces, Unions, Generics, and Narrowing</a></td><td><a href="../lessons/006-classes-modules-decorators-and-compiler-options.md">Lesson</a></td><td><a href="../qnas/006-classes-modules-decorators-and-compiler-options.md">Q&amp;A</a></td><td><a href="007-node-js-npm-package-managers-and-workspaces.md">007 - Node.js, npm, Package Managers, and Workspaces</a></td>
</tr>
  </tbody>
</table>
