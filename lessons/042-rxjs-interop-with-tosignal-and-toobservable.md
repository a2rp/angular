# 42. RxJS interop with `toSignal()` and `toObservable()`

Angular applications may use signals for local and template state while existing services, HTTP APIs, or event streams use RxJS Observables. The `@angular/core/rxjs-interop` package provides two bridges:

- `toSignal()` subscribes to an Observable and exposes its latest value as a Signal.
- `toObservable()` exposes a Signal's changing state as an Observable for RxJS operators and APIs.

These bridges make it possible to choose the reactive model that fits each part of an application without replacing every Observable or converting every signal.

## Learning goals

After this lesson, you should be able to:

- Convert an Observable to a Signal with `toSignal()`.
- Choose between `initialValue`, `undefined`, and `requireSync`.
- Understand `toSignal()` subscription timing, teardown, errors, completion, and equality.
- Convert a Signal to an Observable with `toObservable()`.
- Account for asynchronous updates and stabilization when bridging to RxJS.
- Use RxJS operators such as `debounceTime`, `distinctUntilChanged`, and `switchMap` with signal state.
- Avoid duplicate subscriptions, lossy event conversion, and unnecessary round trips.

## Imports and package

Both functions are exported from `@angular/core/rxjs-interop`. RxJS operators and classes come from `rxjs` or `rxjs/operators` depending on the operator and installed RxJS version. Current RxJS versions export commonly used operators such as `map`, `switchMap`, and `debounceTime` from `rxjs`.

~~~ts
import { toObservable, toSignal } from '@angular/core/rxjs-interop';
import { BehaviorSubject, debounceTime, distinctUntilChanged, map, switchMap } from 'rxjs';
~~~

`toSignal()` and `toObservable()` are stable starting in Angular 20. Check the Angular/RxJS versions installed in the project when following examples in older codebases.

## Convert an Observable to a Signal with `toSignal()`

An Observable can emit zero, one, or many values over time. A Signal always has a current value whenever it is read. `toSignal()` subscribes to the Observable and exposes the most recently emitted value synchronously:

~~~ts
import { toSignal } from '@angular/core/rxjs-interop';
import { interval } from 'rxjs';

readonly seconds = toSignal(interval(1000), { initialValue: 0 });
~~~

The Observable's first emission updates `seconds`; later emissions replace its current value. Read the result like any other signal: `seconds()`.

### Choose an initial state

If the source does not emit synchronously, decide what the Signal should return before its first emission:

~~~ts
// The signal has type Signal<number | undefined> until the stream emits.
readonly value = toSignal(number$);

// Or choose an explicit initial state and type.
readonly count = toSignal(number$, { initialValue: 0 });
~~~

Choose a meaningful initial value when the UI can represent that state. If `undefined` has a different meaning from a real value, keep it in the type and handle it in the template or derived state. Do not use a fake default that the UI could mistake for loaded data.

### Use `requireSync` only for a synchronous source

Some sources guarantee that they emit during subscription, such as an RxJS `BehaviorSubject`:

~~~ts
import { BehaviorSubject } from 'rxjs';
import { toSignal } from '@angular/core/rxjs-interop';

private readonly modeSubject = new BehaviorSubject<'light' | 'dark'>('light');
readonly mode = toSignal(this.modeSubject, { requireSync: true });
~~~

`requireSync: true` asserts that the source emits immediately. It removes the initial `undefined` from the return type and requires no `initialValue`, but it throws at runtime if the guarantee is false. Do not use it for HTTP requests, timers, or an Observable pipeline that may defer its first emission.

## Subscription lifetime and injection context

`toSignal()` subscribes immediately when it is created. It normally needs an injection context, such as a component or service field initializer or constructor. By default, Angular ties the subscription cleanup to that context's `DestroyRef`; for a component, the subscription ends when the component is destroyed.

If you create the bridge later from a method, pass an `Injector` captured in an injection context:

~~~ts
import { inject, Injector } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';

private readonly injector = inject(Injector);

createValueSignal() {
  return toSignal(this.value$, {
    initialValue: 'Waiting',
    injector: this.injector,
  });
}
~~~

Avoid calling `toSignal()` repeatedly for the same Observable. Every call creates another subscription, which can duplicate HTTP requests, timers, or other side effects. Create it once and reuse the returned Signal.

### `manualCleanup`

`{ manualCleanup: true }` disables automatic teardown through `DestroyRef`; the subscription remains until the Observable completes. It is appropriate mainly for a stream that completes on its own. It does not give you a separate subscription handle to unsubscribe manually, so do not use it for a long-lived stream that never completes unless its lifetime is deliberately managed another way.

## Equality for Observable emissions

Use the `equal` option if an Observable emits values that should be treated as equivalent even when their identities differ:

~~~ts
interface Temperature {
  celsius: number;
  sampledAt: number;
}

readonly temperature = toSignal(this.temperature$, {
  initialValue: { celsius: 20, sampledAt: 0 },
  equal: (previous, next) => previous.celsius === next.celsius,
});
~~~

With this comparator, a new sample with the same temperature does not update the Signal. That is correct only if consumers do not need to react to `sampledAt`. As with other signal equality functions, compare every part of the value that downstream consumers care about.

## Observable errors and completion

If the Observable errors, the error is thrown when the resulting Signal is read. Handle recoverable errors in the Observable pipeline before converting it:

~~~ts
import { catchError, of } from 'rxjs';

readonly profile = toSignal(
  this.profile$().pipe(
    catchError((error: unknown) => {
      console.error('Could not load profile', error);
      return of(null);
    }),
  ),
  { initialValue: null },
);
~~~

If the Observable completes after emitting, the Signal continues to return the most recent emitted value. Completion does not erase the Signal's value.

## Convert a Signal to an Observable with `toObservable()`

`toObservable()` lets RxJS code respond to Signal state:

~~~ts
import { toObservable } from '@angular/core/rxjs-interop';
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs';

readonly query = signal('');
readonly query$ = toObservable(this.query);
readonly results$ = this.query$.pipe(
  debounceTime(250),
  distinctUntilChanged(),
  switchMap((query) => this.searchService.search(query)),
);
~~~

This is useful when an Observable-based service or operator pipeline is already the right fit. The resulting Observable reflects the current state of the Signal; it is not an event log of every `.set()` call.

`toObservable()` normally needs an injection context. As with `toSignal()`, pass an `Injector` in its options if it must be created later outside the current context.

### Understand its timing and stabilization

`toObservable()` uses an Angular effect and a `ReplaySubject` to publish the Signal's value. The first value may be delivered synchronously to a subscriber if it is available. Subsequent updates are asynchronous. If the Signal is set several times before Angular runs the effect, the Observable emits the latest stabilized value rather than every intermediate value:

~~~ts
query.set('a');
query.set('an');
query.set('angular');
~~~

The Observable may emit only `'angular'` for those writes. This coalescing is useful for state such as a current search query. If every occurrence matters—such as clicks, keystrokes as events, or audit records—use an Observable or `Subject` as the event source instead of converting a Signal state value.

## Combine the bridges with RxJS

A common direction is Signal input → Observable operators → Signal output:

~~~ts
readonly query = signal('');
readonly query$ = toObservable(this.query);

readonly results = toSignal(
  this.query$.pipe(
    map((query) => query.trim()),
    debounceTime(250),
    distinctUntilChanged(),
    switchMap((query) => this.searchService.search(query)),
    catchError(() => of([])),
  ),
  { initialValue: [] },
);
~~~

Here:

1. The template writes the input into the `query` Signal.
2. `toObservable()` exposes stabilized query state.
3. RxJS operators trim and debounce queries, skip duplicates, and switch to the latest request.
4. `toSignal()` exposes the most recent results to the template.

The bridge does not replace the need to model loading and error states. Add an explicit state model or use Angular's resource APIs where those states and cancellation semantics are a better fit for the application.

## Choose the direction that matches the source of truth

- Use **`toSignal(observable$)`** when an Observable is the source and Angular code needs its latest value synchronously.
- Use **`toObservable(signal)`** when a Signal is the source and an RxJS pipeline or Observable-based API should react to its state.
- Keep an **Observable or `Subject`** as the source when every emitted event and its timing matter.
- Keep a **Signal** as the source for current state used by templates and synchronous derivations.
- Avoid converting Signal → Observable → Signal without a real RxJS boundary; each conversion adds scheduling and lifetime considerations.

## Common mistakes

- **Creating `toSignal()` in a template or repeatedly in a method:** each call subscribes; create one bridge and reuse it.
- **Using `requireSync` for an asynchronous stream:** it can throw if no value is emitted during subscription.
- **Ignoring the initial type:** without an initial value, the Signal can be `undefined` before the first emission.
- **Assuming `toObservable()` emits every `.set()`:** rapid updates can be coalesced to the final stabilized value.
- **Using a Signal as an event log:** represent events with an Observable or `Subject` when every event matters.
- **Forgetting that Observable errors are thrown when the Signal is read:** handle errors in the Observable pipeline or intentionally model an error state.
- **Using `manualCleanup` for an endless stream:** the subscription will persist because the stream never completes.
- **Creating both bridges without need:** choose one source of truth and convert only at the integration boundary.

## Key takeaways

- `toSignal()` subscribes immediately and exposes the latest Observable value as a Signal.
- Supply `initialValue` when the source is asynchronous, or use `requireSync` only when synchronous emission is guaranteed.
- Subscription cleanup is tied to an injection context by default; each bridge should be created once and reused.
- Observable errors are thrown when the Signal is read, while completion leaves the latest value available.
- `toObservable()` exposes Signal state through an effect-backed Observable; subsequent updates are asynchronous and may be coalesced.
- Use Observable/Subject sources for event streams that must preserve every event.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="041-signal-equality-dynamic-dependencies-and-untracked-reads.md">041 - 41. Signal equality, dynamic dependencies, and untracked reads</a></td><td><a href="../qnas/042-rxjs-interop-with-tosignal-and-toobservable.md">Q&amp;A</a></td><td><a href="../examples/042-rxjs-interop-with-tosignal-and-toobservable.md">Example</a></td><td><a href="043-angular-rendering-and-change-detection.md">043 - 43. Angular rendering and change detection</a></td>
</tr>
  </tbody>
</table>
