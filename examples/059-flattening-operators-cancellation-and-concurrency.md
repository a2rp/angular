# Example: Flattening operators, cancellation, and concurrency

This standalone Angular app simulates slow and fast jobs with RxJS **timer()**. Start job A, then quickly start job B in each section to compare the four flattening policies:

- **switchMap()** cancels A and keeps B.
- **mergeMap()** runs A and B concurrently.
- **concatMap()** queues B until A completes.
- **exhaustMap()** ignores B while A is active.
- **finalize()** reports whether an inner timer completed or was cancelled.
- **takeUntilDestroyed()** owns the component's subscriptions and active inner work.

The timers make the behavior visible without a backend. Afterward, a real **HttpClient** search can use the same **switchMap()** policy. Create a standalone Angular app, then add or replace these files.

## 1. Create the component

Create **src/app/app.component.ts**:

~~~ts
import {
  Component,
  DestroyRef,
  inject,
  signal,
} from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import {
  concatMap,
  exhaustMap,
  finalize,
  map,
  mergeMap,
  Subject,
  switchMap,
  tap,
  timer,
} from 'rxjs';
import type { Observable } from 'rxjs';

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  readonly events = signal<string[]>([]);

  private readonly destroyRef = inject(DestroyRef);
  private readonly switchRequests$ = new Subject<string>();
  private readonly mergeRequests$ = new Subject<string>();
  private readonly concatRequests$ = new Subject<string>();
  private readonly exhaustRequests$ = new Subject<string>();

  constructor() {
    this.switchRequests$
      .pipe(
        switchMap((label) => this.runJob('switchMap', label)),
        takeUntilDestroyed(this.destroyRef),
      )
      .subscribe((message) => this.record(message));

    this.mergeRequests$
      .pipe(
        mergeMap((label) => this.runJob('mergeMap', label)),
        takeUntilDestroyed(this.destroyRef),
      )
      .subscribe((message) => this.record(message));

    this.concatRequests$
      .pipe(
        concatMap((label) => this.runJob('concatMap', label)),
        takeUntilDestroyed(this.destroyRef),
      )
      .subscribe((message) => this.record(message));

    this.exhaustRequests$
      .pipe(
        tap((label) => this.record('exhaustMap received ' + label)),
        exhaustMap((label) => this.runJob('exhaustMap', label)),
        takeUntilDestroyed(this.destroyRef),
      )
      .subscribe((message) => this.record(message));
  }

  runSwitchMap(label: string): void {
    this.switchRequests$.next(label);
  }

  runMergeMap(label: string): void {
    this.mergeRequests$.next(label);
  }

  runConcatMap(label: string): void {
    this.concatRequests$.next(label);
  }

  runExhaustMap(label: string): void {
    this.exhaustRequests$.next(label);
  }

  private runJob(operator: string, label: string): Observable<string> {
    const durationMs = label === 'A' ? 1800 : 700;
    let completed = false;

    this.record(operator + ' started ' + label);

    return timer(durationMs).pipe(
      map(() => {
        completed = true;
        return operator + ' result ' + label;
      }),
      finalize(() => {
        const outcome = completed ? 'completed ' : 'cancelled ';
        this.record(operator + ' inner ' + outcome + label);
      }),
    );
  }

  private record(message: string): void {
    this.events.update((current) => [...current.slice(-39), message]);
  }
}
~~~

Each A timer takes 1.8 seconds; each B timer takes 0.7 seconds. **finalize()** runs after normal completion and when an inner subscription is cancelled. The local **completed** flag lets the log distinguish those paths.

Each trigger stream is subscribed to in the component constructor. **takeUntilDestroyed()** completes those subscriptions when Angular destroys the component, which also tears down any active inner timer.

## 2. Create the template

Create **src/app/app.component.html**:

~~~html
<main>
  <h1>Flattening operators</h1>
  <p>
    In each section, start A and quickly start B. A is slow and B is fast.
    Compare which inner jobs start, finish, wait, or get cancelled.
  </p>

  <div class="demo-grid">
    <section aria-labelledby="switch-heading">
      <h2 id="switch-heading">switchMap()</h2>
      <p>A newer job cancels the previous inner subscription.</p>
      <button type="button" (click)="runSwitchMap('A')">
        Start slow A
      </button>
      <button type="button" (click)="runSwitchMap('B')">
        Start fast B
      </button>
    </section>

    <section aria-labelledby="merge-heading">
      <h2 id="merge-heading">mergeMap()</h2>
      <p>Both jobs run concurrently; the faster B can finish first.</p>
      <button type="button" (click)="runMergeMap('A')">
        Start slow A
      </button>
      <button type="button" (click)="runMergeMap('B')">
        Start fast B
      </button>
    </section>

    <section aria-labelledby="concat-heading">
      <h2 id="concat-heading">concatMap()</h2>
      <p>B waits in order until the active A job completes.</p>
      <button type="button" (click)="runConcatMap('A')">
        Start slow A
      </button>
      <button type="button" (click)="runConcatMap('B')">
        Queue fast B
      </button>
    </section>

    <section aria-labelledby="exhaust-heading">
      <h2 id="exhaust-heading">exhaustMap()</h2>
      <p>B is ignored while A's inner job is still active.</p>
      <button type="button" (click)="runExhaustMap('A')">
        Start slow A
      </button>
      <button type="button" (click)="runExhaustMap('B')">
        Try fast B
      </button>
    </section>
  </div>

  <section aria-labelledby="events-heading">
    <h2 id="events-heading">Event log</h2>
    @if (events().length > 0) {
      <ol>
        @for (event of events(); track $index) {
          <li>{{ event }}</li>
        }
      </ol>
    } @else {
      <p>Start a job above to see the inner-stream behavior.</p>
    }
  </section>
</main>
~~~

The **exhaustMap()** input stream logs every outer event before the operator. When B arrives while A is active, the log shows that B reached the pipeline but no inner job starts for it.

## 3. Add simple styles

Create **src/app/app.component.css**:

~~~css
:host {
  display: block;
  color: #172033;
  font-family: system-ui, sans-serif;
}

main {
  box-sizing: border-box;
  max-width: 72rem;
  margin: 0 auto;
  padding: 2rem 1rem;
}

.demo-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 17rem), 1fr));
  gap: 1rem;
  margin-block: 1.5rem;
}

section {
  padding: 1rem;
  border: 1px solid #ccd3df;
  border-radius: 0.75rem;
  background: #f8faff;
}

h1,
h2 {
  line-height: 1.2;
}

button {
  margin: 0.25rem 0.35rem 0.25rem 0;
  padding: 0.55rem 0.8rem;
  border: 1px solid #455a7a;
  border-radius: 0.4rem;
  background: #fff;
  color: inherit;
  font: inherit;
  cursor: pointer;
}

button:hover {
  background: #eaf0fb;
}

button:focus-visible {
  outline: 3px solid #2367d1;
  outline-offset: 2px;
}

li {
  margin-block: 0.35rem;
}
~~~

## 4. Bootstrap the application

Create **src/main.ts** if the starter project does not already bootstrap **AppComponent**:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

The **styleUrl** component option and built-in **@if** and **@for** template blocks are available in modern standalone Angular projects.

## 5. Run the demo

Start the app with the Angular CLI command used by your workspace, usually **ng serve**.

For each operator, click **Start slow A**, then quickly click the adjacent B button:

1. With **switchMap()**, the log reports that A was cancelled. B starts immediately and produces the only result.
2. With **mergeMap()**, A and B both start. B finishes first because its timer is shorter.
3. With **concatMap()**, A starts first and B does not start until A completes.
4. With **exhaustMap()**, the log records B as received, but no B inner job starts while A is active. Try B again after A completes; it is then accepted.

The event log mixes all four demos. Read each line's operator name to follow one pipeline.

## 6. Connect the same policy to HTTP search

In an Angular component with an injected **HttpClient** and a non-nullable query control, the same latest-wins policy can be used for a real search:

~~~ts
readonly queryControl = new FormControl('', { nonNullable: true });

readonly results$ = this.queryControl.valueChanges.pipe(
  startWith(this.queryControl.value),
  map((query) => query.trim()),
  debounceTime(250),
  distinctUntilChanged(),
  switchMap((query) => {
    if (!query) {
      return of<readonly Product[]>([]);
    }

    return this.http.get<readonly Product[]>('/api/products', {
      params: { q: query },
    });
  }),
);
~~~

Import **FormControl** from **@angular/forms** and **startWith**, **map**, **debounceTime**, **distinctUntilChanged**, **switchMap**, and **of** from **rxjs**. The service must return an **Observable**. Display **results$** with **AsyncPipe** or adapt it once with **toSignal()**.

When a newer debounced query reaches **switchMap()**, RxJS unsubscribes from the previous request Observable. Angular **HttpClient** aborts an in-progress client request when it is unsubscribed. That does not undo work the server already performed, so this pattern is appropriate for stale reads, not a guarantee for cancelling writes.

## 7. What to notice

| Operator | Observable behavior in this app |
| --- | --- |
| **switchMap()** | Cancels A when B arrives and forwards B's result. |
| **mergeMap()** | Keeps both inner timers active; B finishes before A. |
| **concatMap()** | Queues B and starts it after A completes. |
| **exhaustMap()** | Receives the B event but does not create a B inner timer while A is active. |
| **finalize()** | Runs for completed timers and for the inner timer cancelled by **switchMap()**. |
| **takeUntilDestroyed()** | Unsubscribes from the component's operator pipelines when the component is destroyed. |

## Related files

- [Lesson: Flattening operators, cancellation, and concurrency](../lessons/059-flattening-operators-cancellation-and-concurrency.md)
- [Q&A: Flattening operators, cancellation, and concurrency](../qnas/059-flattening-operators-cancellation-and-concurrency.md)
- [Next lesson: Error handling, cleanup, and takeUntilDestroyed](../lessons/060-error-handling-cleanup-and-takeuntildestroyed.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="058-rxjs-operators-for-mapping-and-combining-streams.md">058 - RxJS operators for mapping and combining streams</a></td><td><a href="../lessons/059-flattening-operators-cancellation-and-concurrency.md">Lesson</a></td><td><a href="../qnas/059-flattening-operators-cancellation-and-concurrency.md">Q&amp;A</a></td><td><a href="060-error-handling-cleanup-and-takeuntildestroyed.md">060 - Error handling, cleanup, and takeUntilDestroyed</a></td>
</tr>
  </tbody>
</table>
