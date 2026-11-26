# 42. Example: bridge Angular signals and RxJS

This standalone Angular page shows both directions of interop. A `BehaviorSubject` exposes its current theme through `toSignal(..., { requireSync: true })`. A Signal-backed search query passes through `toObservable()`, RxJS operators, and back into Angular as results with `toSignal()`.

## What you will build

- A theme value supplied by a `BehaviorSubject` and read as an Angular Signal.
- A query Signal exposed as an Observable.
- A debounced, duplicate-filtered search pipeline using `switchMap()`.
- A results Signal that the template reads synchronously.
- An optional error-handling pattern for a real API Observable.

## Requirements and setup

The project needs Angular 20 or later for stable `toSignal()` and `toObservable()` APIs, plus the RxJS version installed by the Angular CLI project. Create a practice app if needed:

~~~sh
ng new rxjs-signals-demo --standalone --style=css
cd rxjs-signals-demo
ng serve
~~~

Use the generated root component. The files below use conventional `app.component.*` names; adjust them if your CLI generated shorter names such as `app.*`.

## File layout

~~~text
src/app/
  app.component.ts
  app.component.html
  app.component.css
~~~

## 1. Create the interop pipeline

Replace or adapt `src/app/app.component.ts`:

~~~ts
import { Component, signal } from '@angular/core';
import { toObservable, toSignal } from '@angular/core/rxjs-interop';
import {
  BehaviorSubject,
  debounceTime,
  distinctUntilChanged,
  map,
  switchMap,
  timer,
} from 'rxjs';

interface LearningTopic {
  id: number;
  title: string;
  category: string;
}

const TOPICS: LearningTopic[] = [
  { id: 1, title: 'Signals basics', category: 'Angular state' },
  { id: 2, title: 'Computed values', category: 'Angular state' },
  { id: 3, title: 'RxJS interop', category: 'Angular and RxJS' },
  { id: 4, title: 'Observable operators', category: 'RxJS' },
  { id: 5, title: 'Component templates', category: 'Angular components' },
];

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  private readonly themeSubject = new BehaviorSubject<'light' | 'dark'>('light');

  // BehaviorSubject emits its current value synchronously on subscription.
  readonly theme = toSignal(this.themeSubject, { requireSync: true });

  readonly query = signal('');
  readonly query$ = toObservable(this.query);

  readonly results = toSignal(
    this.query$.pipe(
      map((query) => query.trim().toLowerCase()),
      debounceTime(250),
      distinctUntilChanged(),
      switchMap((query) =>
        // timer() simulates an asynchronous search request.
        timer(300).pipe(
          map(() =>
            TOPICS.filter((topic) =>
              `${topic.title} ${topic.category}`.toLowerCase().includes(query),
            ),
          ),
        ),
      ),
    ),
    { initialValue: TOPICS },
  );

  setQuery(value: string): void {
    this.query.set(value);
  }

  toggleTheme(): void {
    this.themeSubject.next(this.theme() === 'light' ? 'dark' : 'light');
  }
}
~~~

The theme Observable emits synchronously because a `BehaviorSubject` immediately sends its current value to each new subscriber. `requireSync: true` tells Angular to enforce that guarantee and gives `theme` a non-optional value type.

The search query starts as a Signal. `toObservable()` publishes its current state for the RxJS pipeline. The pipeline trims and lowercases text, waits for typing to pause, skips repeated normalized queries, and uses `switchMap()` to cancel a prior simulated search when a newer query arrives. `toSignal()` makes the latest results easy to read from the template. Its `initialValue` shows the full topic list until the first search completes.

Each bridge is created once as a component field and reused. This avoids accidental duplicate subscriptions.

## 2. Add the template

Create or replace `src/app/app.component.html`:

~~~html
<main class="page" [class.dark-theme]="theme() === 'dark'">
  <header class="page-header">
    <p class="eyebrow">Angular example 42</p>
    <h1>Signals meet RxJS</h1>
    <p>
      Keep current state in signals and cross into RxJS where stream operators
      are useful.
    </p>
    <button type="button" (click)="toggleTheme()">
      Switch to {{ theme() === 'light' ? 'dark' : 'light' }} theme
    </button>
  </header>

  <section class="card" aria-labelledby="search-heading">
    <h2 id="search-heading">Search learning topics</h2>
    <label for="topic-query">Search by topic or category</label>
    <input
      id="topic-query"
      type="search"
      [value]="query()"
      (input)="setQuery($any($event.target).value)"
      placeholder="For example, signals or RxJS"
    />
    <p class="hint">
      Results update after a short pause. If you keep typing, the earlier
      simulated search is replaced by the latest one.
    </p>

    <ul class="topic-list">
      @for (topic of results(); track topic.id) {
        <li>
          <strong>{{ topic.title }}</strong>
          <span>{{ topic.category }}</span>
        </li>
      } @empty {
        <li>No matching topics. Try another search.</li>
      }
    </ul>
  </section>

  <section class="card" aria-labelledby="flow-heading">
    <h2 id="flow-heading">Interop directions</h2>
    <ol>
      <li><code>BehaviorSubject → toSignal() → theme()</code></li>
      <li><code>query Signal → toObservable() → RxJS operators</code></li>
      <li><code>search Observable → toSignal() → results()</code></li>
    </ol>
    <p class="hint">
      Open the component code and identify where each subscription starts and
      how its lifetime is tied to the component.
    </p>
  </section>
</main>
~~~

## 3. Add simple styles

Create or replace `src/app/app.component.css`:

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
  width: min(100% - 2rem, 48rem);
  min-height: 100vh;
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header,
.card {
  margin-block-end: 1.25rem;
}

.eyebrow {
  color: #3156a6;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.card {
  padding: 1.25rem;
  border: 1px solid #d6deeb;
  border-radius: 0.75rem;
  background: #fff;
}

h1,
h2 {
  line-height: 1.2;
}

label,
input {
  display: block;
}

input {
  box-sizing: border-box;
  width: 100%;
  margin-block: 0.5rem;
  padding: 0.65rem;
  border: 1px solid #9aa8bd;
  border-radius: 0.4rem;
  font: inherit;
}

button {
  padding: 0.6rem 0.8rem;
  border: 0;
  border-radius: 0.4rem;
  color: #fff;
  background: #244b9a;
  font: inherit;
  cursor: pointer;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #f2a900;
  outline-offset: 2px;
}

.topic-list {
  display: grid;
  gap: 0.75rem;
  padding-inline-start: 1.25rem;
}

.topic-list li span {
  display: block;
  color: #526078;
}

.hint {
  color: #526078;
  line-height: 1.55;
}

.dark-theme {
  color: #eef3ff;
  background: #172033;
}

.dark-theme .card {
  border-color: #43516a;
  background: #222e44;
}

.dark-theme .hint,
.dark-theme .topic-list li span {
  color: #c0cbe0;
}
~~~

## 4. Run and explore

1. Start the app with `ng serve`.
2. The initial theme is available immediately because `BehaviorSubject` emits synchronously and `requireSync` verifies that guarantee.
3. The initial topic list is shown while the first search pipeline settles.
4. Type `signals`, then keep typing before the debounce finishes. The stream waits for a pause before starting the simulated search.
5. Type a different query while a search is pending. `switchMap()` unsubscribes from the earlier timer and the latest query wins.
6. Clear the input. The empty query matches every topic.
7. Switch the theme. The component publishes to the subject, and the `theme` Signal exposes its latest value to the template.

## 5. Optional: handle a real Observable error before `toSignal()`

A `toSignal()` created from an Observable that errors will throw when that Signal is read. Convert expected failures into a value or explicit state in the pipeline:

~~~ts
import { catchError, of } from 'rxjs';

interface Profile {
  name: string;
}

readonly profile = toSignal(
  this.profileService.loadProfile().pipe(
    catchError((error: unknown) => {
      console.error('Profile request failed', error);
      return of(null);
    }),
  ),
  { initialValue: null },
);
~~~

The resulting signal has a `Profile | null` value. For a production UI, model loading, success, and failure explicitly rather than relying on console output alone.

## 6. Compare `initialValue` and `requireSync`

Use `initialValue` when the source may emit later:

~~~ts
readonly temperature = toSignal(this.temperature$, {
  initialValue: 20,
});
~~~

Use `requireSync` only when the source guarantees a synchronous first emission:

~~~ts
private readonly statusSubject = new BehaviorSubject('Ready');
readonly status = toSignal(this.statusSubject, { requireSync: true });
~~~

If the guarantee is false, `requireSync` throws at runtime. Do not use it to remove a type that represents a real loading period.

## 7. Understand state versus events

Suppose a Signal changes several times synchronously:

~~~ts
const query = signal('');
const query$ = toObservable(query);

query.set('a');
query.set('an');
query.set('angular');
~~~

`toObservable()` can publish only the final stabilized value, `'angular'`. That is appropriate for current search state. If every action must be preserved as an event, publish events through a `Subject` or another Observable source instead.

## Concepts to notice

- `toSignal()` starts its subscription immediately and exposes the latest emission as current state.
- `BehaviorSubject` has a synchronous current value, making it suitable for the `requireSync` demonstration.
- `toObservable()` publishes Signal state asynchronously after the first available value; multiple fast writes can be coalesced.
- The RxJS pipeline is created once, and `toSignal()` ties its subscription to the component's injection context by default.
- `switchMap()` cancels the previous inner search when a newer query arrives.
- State bridges are not event logs; use a Subject or Observable when every event matters.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="041-signal-equality-dynamic-dependencies-and-untracked-reads.md">041 - 41. Signal equality, dynamic dependencies, and untracked reads</a></td><td><a href="../lessons/042-rxjs-interop-with-tosignal-and-toobservable.md">Lesson</a></td><td><a href="../qnas/042-rxjs-interop-with-tosignal-and-toobservable.md">Q&amp;A</a></td><td><a href="043-angular-rendering-and-change-detection.md">043 - 43. Angular rendering and change detection</a></td>
</tr>
  </tbody>
</table>
