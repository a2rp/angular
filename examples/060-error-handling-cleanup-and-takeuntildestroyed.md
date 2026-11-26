# Example: Error handling, cleanup, and takeUntilDestroyed

This standalone Angular demo shows four outcomes without needing a backend:

- A successful request renders a success state.
- A permanent simulated failure becomes a safe error message.
- A transient failure is retried twice and then succeeds.
- Starting another action while a slow request is pending cancels the old inner stream.
- **finalize()** records whether each request stream completed or was cancelled.
- **takeUntilDestroyed()** binds the component's event subscription to its Angular lifetime.

The API service is a local simulation. The final section shows how the same pipeline can call **HttpClient**. Create a standalone Angular application and replace the following files.

~~~sh
ng new angular-error-demo --style=css
cd angular-error-demo
ng serve --open
~~~

## 1. Create the simulated API service

Create **src/app/demo-api.service.ts**:

~~~ts
import { Injectable } from '@angular/core';
import {
  defer,
  map,
  mergeMap,
  throwError,
  timer,
} from 'rxjs';
import type { Observable } from 'rxjs';

export type DemoMode = 'success' | 'slow' | 'fail' | 'transient';

@Injectable({ providedIn: 'root' })
export class DemoApiService {
  load(mode: DemoMode): Observable<string> {
    let attempt = 0;

    return defer(() => {
      attempt += 1;

      if (mode === 'fail') {
        return timer(600).pipe(
          mergeMap(() =>
            throwError(() => new Error('Simulated service unavailable')),
          ),
        );
      }

      if (mode === 'transient' && attempt < 3) {
        return throwError(
          () => new Error('Temporary simulated connection failure'),
        );
      }

      const delayMs = mode === 'slow' ? 3500 : 650;

      return timer(delayMs).pipe(
        map(() => {
          if (mode === 'transient') {
            return 'Recovered successfully on attempt ' + attempt + '.';
          }

          return 'The simulated request completed successfully.';
        }),
      );
    });
  }
}
~~~

**defer()** runs its factory each time the source is subscribed. The transient case fails on the first two subscriptions and succeeds on the third, making the retry policy visible. The permanent failure waits briefly so its error state can be observed.

## 2. Create the component

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
  catchError,
  finalize,
  map,
  of,
  retry,
  startWith,
  Subject,
  switchMap,
  tap,
  throwError,
  timer,
} from 'rxjs';
import { DemoApiService } from './demo-api.service';
import type { DemoMode } from './demo-api.service';

type LoadState =
  | { kind: 'idle' }
  | { kind: 'loading' }
  | { kind: 'success'; message: string }
  | { kind: 'error'; message: string };

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  private readonly api = inject(DemoApiService);
  private readonly destroyRef = inject(DestroyRef);
  private readonly actions$ = new Subject<DemoMode>();

  readonly state = signal<LoadState>({ kind: 'idle' });
  readonly events = signal<string[]>([]);

  constructor() {
    this.actions$
      .pipe(
        switchMap((mode) => {
          let completed = false;

          return this.api.load(mode).pipe(
            retry({
              count: mode === 'transient' ? 2 : 0,
              delay: (_error, retryCount) => {
                this.addEvent(
                  'Retry ' + retryCount + ' scheduled after a short delay.',
                );
                return timer(retryCount * 400);
              },
            }),
            map(
              (message): LoadState => ({
                kind: 'success',
                message,
              }),
            ),
            catchError((error: unknown) => {
              const detail =
                error instanceof Error ? error.message : 'Unknown failure';
              this.addEvent('Diagnostic: ' + detail);

              return of<LoadState>({
                kind: 'error',
                message: 'The request failed. You can try again.',
              });
            }),
            startWith<LoadState>({ kind: 'loading' }),
            tap({
              complete: () => {
                completed = true;
              },
            }),
            finalize(() => {
              const outcome = completed ? 'finished' : 'cancelled';
              this.addEvent(mode + ' request ' + outcome + '.');
            }),
          );
        }),
        takeUntilDestroyed(this.destroyRef),
      )
      .subscribe((state) => this.state.set(state));
  }

  run(mode: DemoMode): void {
    this.addEvent('Started ' + mode + ' request.');
    this.actions$.next(mode);
  }

  clearEvents(): void {
    this.events.set([]);
  }

  private addEvent(message: string): void {
    const timestamp = new Date().toLocaleTimeString();

    this.events.update((events) =>
      [timestamp + ' - ' + message, ...events].slice(0, 10),
    );
  }
}
~~~

The error is handled inside the **switchMap()** projection. A failed request therefore becomes an error state without killing **actions$**; you can click another action afterward. The bounded retry applies only to the transient mode. Starting a new action unsubscribes from the previous inner request, and **finalize()** records that cancellation.

**completed** is set by the completion notification after the error has been mapped to an error state. If a new action cancels the inner stream first, the completion callback does not run and the finalizer records cancellation. This value is for the demonstration log; production state should follow the application's request model.

## 3. Create the template

Create **src/app/app.component.html**:

~~~html
<main class="page">
  <header>
    <p class="eyebrow">Angular and RxJS</p>
    <h1>Request states and cleanup</h1>
    <p>
      Try success, failure, retry, and cancellation. The service is simulated;
      no network request is sent.
    </p>
  </header>

  <section class="panel" aria-labelledby="request-heading">
    <h2 id="request-heading">Run a request</h2>

    <div class="actions">
      <button type="button" (click)="run('success')">
        Load success
      </button>
      <button type="button" (click)="run('fail')">
        Load failure
      </button>
      <button type="button" (click)="run('transient')">
        Retry transient failure
      </button>
      <button type="button" class="secondary" (click)="run('slow')">
        Start slow request
      </button>
    </div>

    <p class="hint">
      Start a slow request, then click another button before it finishes.
      switchMap() unsubscribes from the old request.
    </p>

    @let current = state();

    <div class="result" aria-live="polite" aria-atomic="true">
      @switch (current.kind) {
        @case ('idle') {
          <p>Choose an action to begin.</p>
        }
        @case ('loading') {
          <p>Request in progress…</p>
        }
        @case ('success') {
          <p class="success">{{ current.message }}</p>
        }
        @case ('error') {
          <p class="failure">{{ current.message }}</p>
        }
      }
    </div>
  </section>

  <section class="panel" aria-labelledby="activity-heading">
    <div class="section-heading">
      <div>
        <h2 id="activity-heading">Activity log</h2>
        <p>Retry attempts and finalization are recorded here.</p>
      </div>
      <button type="button" class="secondary" (click)="clearEvents()">
        Clear log
      </button>
    </div>

    @if (events().length > 0) {
      <ol class="events">
        @for (event of events(); track $index) {
          <li>{{ event }}</li>
        }
      </ol>
    } @else {
      <p class="hint">No activity yet.</p>
    }
  </section>
</main>
~~~

The live region announces status changes to assistive technology. The buttons have visible labels and use native button behavior.

## 4. Add basic styles

Create **src/app/app.component.css**:

~~~css
:host {
  display: block;
  min-height: 100vh;
  color: #172033;
  background: #f3f6fb;
  font-family: system-ui, sans-serif;
}

.page {
  box-sizing: border-box;
  width: min(100% - 2rem, 52rem);
  margin: 0 auto;
  padding: 3rem 0;
}

header {
  margin-bottom: 1.5rem;
}

.eyebrow {
  margin-bottom: 0.4rem;
  color: #405fbd;
  font-size: 0.85rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1,
h2,
p {
  margin-top: 0;
}

h1 {
  margin-bottom: 0.65rem;
  font-size: clamp(2rem, 5vw, 3rem);
}

header > p:last-child,
.section-heading p {
  color: #4d5a70;
  line-height: 1.6;
}

.panel {
  margin-top: 1rem;
  padding: 1.25rem;
  border: 1px solid #d6deeb;
  border-radius: 0.8rem;
  background: #fff;
  box-shadow: 0 0.4rem 1.4rem rgb(27 45 79 / 6%);
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.65rem;
}

button {
  min-height: 2.75rem;
  padding: 0.65rem 0.9rem;
  border: 1px solid #334fa3;
  border-radius: 0.45rem;
  color: #fff;
  background: #334fa3;
  font: inherit;
  font-weight: 650;
  cursor: pointer;
}

button:hover {
  background: #263d83;
}

button:focus-visible {
  outline: 3px solid #f1b642;
  outline-offset: 3px;
}

button.secondary {
  border-color: #aeb9ca;
  color: #263247;
  background: #fff;
}

button.secondary:hover {
  background: #edf1f7;
}

.hint {
  margin: 1rem 0;
  color: #59677d;
  line-height: 1.55;
}

.result {
  min-height: 3rem;
  padding: 0.8rem 1rem;
  border-radius: 0.5rem;
  background: #f3f6fb;
}

.result p {
  margin: 0;
}

.success {
  color: #17633c;
  font-weight: 650;
}

.failure {
  color: #9e2f2f;
  font-weight: 650;
}

.section-heading {
  display: flex;
  align-items: start;
  justify-content: space-between;
  gap: 1rem;
}

.section-heading p {
  margin-bottom: 0;
}

.events {
  display: grid;
  gap: 0.55rem;
  padding-left: 1.5rem;
  color: #344158;
  line-height: 1.5;
}

@media (max-width: 34rem) {
  .page {
    padding: 1.5rem 0;
  }

  .section-heading {
    flex-direction: column;
  }
}
~~~

## 5. Confirm the bootstrap file

A new standalone CLI app already has a bootstrap file. It should bootstrap **AppComponent** with **bootstrapApplication()**. If the generated file differs, use **src/main.ts** like this:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

## 6. Try each outcome

1. Click **Load success** and observe the loading state followed by success.
2. Click **Load failure** and observe the safe error message plus a diagnostic entry.
3. Click **Retry transient failure**. The first two attempts fail, the retry delays are logged, and the third attempt succeeds.
4. Click **Start slow request**, then quickly start another action. The slow inner stream is cancelled and its finalizer logs that outcome.
5. Click **Load failure** again. The outer action stream remains usable because recovery is inside **switchMap()**.
6. Destroy or navigate away from the component in a larger application. **takeUntilDestroyed()** tears down the outer subscription and its active inner request.

The buttons can be clicked repeatedly. In a real app, give users clear feedback about whether another action replaces or queues an operation.

## 7. Adapt the pipeline to HttpClient

Provide **HttpClient** in the application's providers, then replace the simulated service call with a real request. For example, in a standalone application configuration:

~~~ts
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient()],
};
~~~

Bootstrap with that configuration:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig).catch((error: unknown) => {
  console.error(error);
});
~~~

A service can return a typed request Observable:

~~~ts
import { HttpClient } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { Observable } from 'rxjs';

interface Report {
  title: string;
}

@Injectable({ providedIn: 'root' })
export class ReportApi {
  private readonly http = inject(HttpClient);

  load(): Observable<Report> {
    return this.http.get<Report>('/api/report');
  }
}
~~~

Keep the bounded retry before **catchError()**, and use the error to produce a safe UI state:

~~~ts
import { HttpErrorResponse } from '@angular/common/http';
import { catchError, map, of, retry, throwError, timer } from 'rxjs';

const reportState$ = this.reportApi.load().pipe(
  retry({
    count: 2,
    delay: (error: unknown, retryCount) => {
      if (
        error instanceof HttpErrorResponse &&
        (error.status === 0 || error.status >= 500)
      ) {
        return timer(retryCount * 500);
      }

      return throwError(() => error);
    },
  }),
  map((report): LoadState => ({ kind: 'success', message: report.title })),
  catchError((_error: unknown) =>
    of<LoadState>({
      kind: 'error',
      message: 'The report could not be loaded. Try again.',
    }),
  ),
  startWith<LoadState>({ kind: 'loading' }),
);
~~~

The retry condition is only an example policy: adapt it to the API, authentication rules, rate limits, and idempotency guarantees. Do not automatically retry writes merely because the client observed an error.

## What this example demonstrates

- **catchError()** converts a failed request into an explicit error state.
- Catching inside the **switchMap()** projection keeps the outer action stream alive.
- **retry()** resubscribes only for the transient demonstration and has a fixed maximum.
- **finalize()** runs on normal completion, handled failure, and cancellation.
- **takeUntilDestroyed()** owns the component's imperative subscription.
- **switchMap()** cancellation is client-side subscription cleanup, not a server rollback.

## Continue learning

- Previous: [Flattening operators, cancellation, and concurrency](../lessons/059-flattening-operators-cancellation-and-concurrency.md)
- Lesson: [Error handling, cleanup, and takeUntilDestroyed](../lessons/060-error-handling-cleanup-and-takeuntildestroyed.md)
- Q&A: [Error handling, cleanup, and takeUntilDestroyed](../qnas/060-error-handling-cleanup-and-takeuntildestroyed.md)
- Next: [Choose template-driven, reactive, or signal forms](../lessons/061-choose-template-driven-reactive-or-signal-forms.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="059-flattening-operators-cancellation-and-concurrency.md">059 - Flattening operators, cancellation, and concurrency</a></td><td><a href="../lessons/060-error-handling-cleanup-and-takeuntildestroyed.md">Lesson</a></td><td><a href="../qnas/060-error-handling-cleanup-and-takeuntildestroyed.md">Q&amp;A</a></td><td><a href="061-choose-template-driven-reactive-or-signal-forms.md">061 - Choose template-driven, reactive, or signal forms</a></td>
</tr>
  </tbody>
</table>
