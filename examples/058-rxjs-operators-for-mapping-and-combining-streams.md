# Example: RxJS operators for mapping and combining streams

This Angular app demonstrates how operators shape streams:

- **startWith()**, **map()**, **distinctUntilChanged()**, and **filter()** normalize and transform values.
- **combineLatest()** derives visible tasks from task data, a status filter, and a search query.
- **withLatestFrom()** captures the current visible tasks only when the refresh action occurs.
- **zip()**, **forkJoin()**, **merge()**, and **concat()** show different relationships between finite sources.
- **toSignal()** gives the component a signal view of the Observable results and owns cleanup with the component's injection context.

Start with a standalone Angular CLI app that has RxJS installed, then add or replace these files. The snippets use the modern **rxjs** import path.

## 1. Create the task-board service

Create **src/app/task-board.service.ts**:

~~~ts
import { Injectable } from '@angular/core';
import {
  BehaviorSubject,
  combineLatest,
  distinctUntilChanged,
  map,
  startWith,
  Subject,
  withLatestFrom,
} from 'rxjs';

export interface Task {
  id: number;
  title: string;
  done: boolean;
}

export type TaskStatus = 'all' | 'open' | 'done';

const INITIAL_TASKS: readonly Task[] = [
  { id: 1, title: 'Read the RxJS lesson', done: true },
  { id: 2, title: 'Build a task filter', done: false },
  { id: 3, title: 'Try combineLatest', done: false },
  { id: 4, title: 'Review operator choices', done: true },
];

@Injectable({ providedIn: 'root' })
export class TaskBoardService {
  private readonly tasksSubject =
    new BehaviorSubject<readonly Task[]>(INITIAL_TASKS);
  private readonly statusSubject = new BehaviorSubject<TaskStatus>('all');
  private readonly querySubject = new BehaviorSubject<string>('');
  private readonly refreshSubject = new Subject<void>();

  private readonly tasks$ = this.tasksSubject.asObservable();
  private readonly status$ = this.statusSubject.asObservable();

  private readonly query$ = this.querySubject.pipe(
    startWith(''),
    map((query) => query.trim().toLocaleLowerCase()),
    distinctUntilChanged(),
  );

  readonly visibleTasks$ = combineLatest({
    tasks: this.tasks$,
    status: this.status$,
    query: this.query$,
  }).pipe(
    map(({ tasks, status, query }) =>
      tasks.filter((task) => {
        const matchesStatus =
          status === 'all' ||
          (status === 'done' ? task.done : !task.done);
        const matchesQuery = task.title
          .toLocaleLowerCase()
          .includes(query);

        return matchesStatus && matchesQuery;
      }),
    ),
  );

  readonly refreshSnapshot$ = this.refreshSubject.pipe(
    withLatestFrom(this.visibleTasks$),
    map(([, tasks]) => ({
      count: tasks.length,
      titles: tasks.map((task) => task.title),
    })),
  );

  setStatus(status: TaskStatus): void {
    this.statusSubject.next(status);
  }

  setQuery(query: string): void {
    this.querySubject.next(query);
  }

  toggleTask(taskId: number): void {
    const nextTasks = this.tasksSubject.value.map((task) =>
      task.id === taskId ? { ...task, done: !task.done } : task,
    );

    this.tasksSubject.next(nextTasks);
  }

  refresh(): void {
    this.refreshSubject.next(undefined);
  }
}
~~~

The **BehaviorSubject** instances hold the current task list and filter values, so each supplies an initial value to **combineLatest()**. The query stream is seeded with **startWith('')**, normalized with **map()**, and skips repeated normalized strings with **distinctUntilChanged()**.

The view list is recomputed when the tasks, status, or query changes. The output of **refreshSnapshot$** changes only when **refreshSubject** emits; **withLatestFrom()** attaches the latest visible tasks to that action.

## 2. Create the standalone component

Create **src/app/app.component.ts**:

~~~ts
import { Component, inject, signal } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import {
  BehaviorSubject,
  combineLatest,
  concat,
  distinctUntilChanged,
  filter,
  forkJoin,
  map,
  merge,
  of,
  startWith,
  withLatestFrom,
  zip,
} from 'rxjs';
import { TaskBoardService } from './task-board.service';
import type { Task, TaskStatus } from './task-board.service';

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
})
export class AppComponent {
  private readonly taskBoard = inject(TaskBoardService);

  readonly visibleTasks = toSignal(this.taskBoard.visibleTasks$, {
    initialValue: [] as readonly Task[],
  });

  readonly latestSnapshot = toSignal(this.taskBoard.refreshSnapshot$, {
    initialValue: null,
  });

  readonly statuses: readonly TaskStatus[] = ['all', 'open', 'done'];
  readonly operatorResults = signal<string[]>([]);
  readonly combinationResults = signal<string[]>([]);

  setStatus(event: Event): void {
    const select = event.target as HTMLSelectElement;
    this.taskBoard.setStatus(select.value as TaskStatus);
  }

  setQuery(event: Event): void {
    const input = event.target as HTMLInputElement;
    this.taskBoard.setQuery(input.value);
  }

  toggleTask(task: Task): void {
    this.taskBoard.toggleTask(task.id);
  }

  refresh(): void {
    this.taskBoard.refresh();
  }

  runTransformationDemo(): void {
    const output: string[] = [];

    of(' RxJS ', 'Angular', 'angular', '   ')
      .pipe(
        startWith(' Streams '),
        map((value) => value.trim().toLocaleLowerCase()),
        distinctUntilChanged(),
        filter((value) => value.length > 0),
        map((value) => `Value: ${value}`),
      )
      .subscribe((value) => output.push(value));

    this.operatorResults.set(output);
  }

  runCombinationDemo(): void {
    const output: string[] = [];

    combineLatest({
      framework: of('Angular'),
      library: of('RxJS'),
    }).subscribe(({ framework, library }) => {
      output.push(`combineLatest: ${framework} + ${library}`);
    });

    const draft$ = new BehaviorSubject('Draft: operator notes');
    of('withLatestFrom trigger').pipe(
      withLatestFrom(draft$),
    ).subscribe(([trigger, draft]) => {
      output.push(`${trigger} -> ${draft}`);
    });

    zip(
      of('Ada', 'Grace'),
      of('Architect', 'Compiler pioneer'),
    ).subscribe(([name, role]) => {
      output.push(`zip: ${name} - ${role}`);
    });

    forkJoin({
      profile: of('loaded'),
      preferences: of('loaded'),
    }).subscribe(({ profile, preferences }) => {
      output.push(`forkJoin: profile ${profile}, preferences ${preferences}`);
    });

    merge(
      of('cache event'),
      of('server event'),
    ).subscribe((event) => {
      output.push(`merge: ${event}`);
    });

    concat(
      of('first finite source'),
      of('second finite source'),
    ).subscribe((event) => {
      output.push(`concat: ${event}`);
    });

    draft$.complete();
    this.combinationResults.set(output);
  }
}
~~~

The transformation demo uses **startWith()**, **map()**, **distinctUntilChanged()**, and the RxJS **filter()** operator. The combination demo uses finite, synchronous sources so the results appear immediately. The **merge()** ordering in this small demo follows those synchronous sources; separate asynchronous sources can interleave according to when they emit.

## 3. Create the template

Create **src/app/app.component.html**:

~~~html
<main>
  <h1>RxJS mapping and combination operators</h1>

  <section aria-labelledby="task-board-heading">
    <h2 id="task-board-heading">Live task board</h2>
    <p>
      The visible list is derived from three streams: tasks, status, and search.
      Changing any input recalculates the list with combineLatest().
    </p>

    <label for="status-filter">Task status</label>
    <select id="status-filter" (change)="setStatus($event)">
      @for (status of statuses; track status) {
        <option [value]="status">{{ status }}</option>
      }
    </select>

    <label for="task-search">Search tasks</label>
    <input
      id="task-search"
      type="search"
      placeholder="Try “rxjs”"
      (input)="setQuery($event)"
    />

    <ul>
      @for (task of visibleTasks(); track task.id) {
        <li>
          <button type="button" (click)="toggleTask(task)">
            Mark as {{ task.done ? 'open' : 'done' }}
          </button>
          <span>{{ task.title }} - {{ task.done ? 'Done' : 'Open' }}</span>
        </li>
      } @empty {
        <li>No tasks match these filters.</li>
      }
    </ul>

    <button type="button" (click)="refresh()">
      Capture current result
    </button>

    @if (latestSnapshot(); as snapshot) {
      <p>
        Refresh action captured {{ snapshot.count }} visible task(s):
        {{ snapshot.titles.join(', ') || 'none' }}
      </p>
    } @else {
      <p>No refresh action has been captured yet.</p>
    }
  </section>

  <section aria-labelledby="transform-heading">
    <h2 id="transform-heading">Transform and filter values</h2>
    <p>
      Run the stream through startWith(), map(), distinctUntilChanged(),
      filter(), and another map().
    </p>
    <button type="button" (click)="runTransformationDemo()">
      Run transformation demo
    </button>
    <ol>
      @for (result of operatorResults(); track $index) {
        <li>{{ result }}</li>
      } @empty {
        <li>Choose the button to see the transformed values.</li>
      }
    </ol>
  </section>

  <section aria-labelledby="combination-heading">
    <h2 id="combination-heading">Compare combination functions</h2>
    <p>
      Each line shows what the function contributes: latest values, action
      trigger, positional pairing, final results, concurrent forwarding, or
      sequential forwarding.
    </p>
    <button type="button" (click)="runCombinationDemo()">
      Run combination demo
    </button>
    <ol>
      @for (result of combinationResults(); track $index) {
        <li>{{ result }}</li>
      } @empty {
        <li>Choose the button to see the combined results.</li>
      }
    </ol>
  </section>
</main>
~~~

The task-board status and search controls update their current-value streams. Clicking a task publishes a new task array, which also causes **combineLatest()** to recalculate. The refresh button is the source for **withLatestFrom()**; changing search or status alone does not create a new snapshot.

## 4. Bootstrap the application

Create **src/main.ts**:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

Use this entry point if the starter app does not already bootstrap **AppComponent**.

## 5. Run and explore

Start the app with the Angular CLI command used by your workspace, usually **ng serve**.

1. Change the status filter and search text. The task list updates whenever any input changes.
2. Type a query with spaces or different letter casing. The stream normalizes it, and equivalent consecutive queries are suppressed.
3. Mark a task done or open. The service emits a new array, and the derived list is recalculated.
4. Click **Capture current result**. **withLatestFrom()** snapshots the current visible list only when this action stream emits.
5. Run the transformation demo. The initial value is seeded, strings are normalized, consecutive duplicates and empty results are removed, and remaining values are mapped to labels.
6. Run the combination demo. **combineLatest()** produces one object from the latest values; **withLatestFrom()** attaches a companion value to its source trigger; **zip()** pairs values by position; **forkJoin()** waits for finite sources and emits their final values; **merge()** forwards both event sources; and **concat()** emits the second finite source only after the first completes.

## 6. What to notice

| Interaction | Operator behavior |
| --- | --- |
| Change search or status | **combineLatest()** recomputes from the latest value of all three state streams. |
| Toggle a task | A new array reaches the combination, so the visible list is recalculated. |
| Click the refresh button | **withLatestFrom()** attaches the latest visible list to the action event. |
| Run the transform demo | **startWith()**, **map()**, **distinctUntilChanged()**, and **filter()** each perform a separate step. |
| Read the zip output | Names are paired with roles by their position. |
| Read the forkJoin output | Both finite inputs complete before their final values are emitted together. |
| Compare merge and concat | **merge()** forwards input events; **concat()** waits for each finite source to complete in order. |

## Related files

- [Lesson: RxJS operators for mapping and combining streams](../lessons/058-rxjs-operators-for-mapping-and-combining-streams.md)
- [Q&A: RxJS operators for mapping and combining streams](../qnas/058-rxjs-operators-for-mapping-and-combining-streams.md)
- [Next lesson: Flattening operators, cancellation, and concurrency](../lessons/059-flattening-operators-cancellation-and-concurrency.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="057-rxjs-observables-subscriptions-and-stream-basics.md">057 - RxJS Observables, subscriptions, and stream basics</a></td><td><a href="../lessons/058-rxjs-operators-for-mapping-and-combining-streams.md">Lesson</a></td><td><a href="../qnas/058-rxjs-operators-for-mapping-and-combining-streams.md">Q&amp;A</a></td><td><a href="059-flattening-operators-cancellation-and-concurrency.md">059 - Flattening operators, cancellation, and concurrency</a></td>
</tr>
  </tbody>
</table>
