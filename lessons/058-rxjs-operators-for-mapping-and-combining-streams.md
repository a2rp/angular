# RxJS operators for mapping and combining streams

RxJS operators transform, filter, and combine values without changing the source Observable. They are functions composed with **pipe()** to create a new Observable. This lesson covers value-by-value transformations and the main ways to combine independent streams. Higher-order mapping, request cancellation, and concurrency are covered in the next lesson.

## Learning outcomes

After this lesson, you should be able to:

- Explain how **pipe()** composes operators and why it does not subscribe by itself.
- Transform values with **map()** and keep matching values with **filter()**.
- Seed a stream with **startWith()** and avoid repeated equivalent values with **distinctUntilChanged()**.
- Combine current state with **combineLatest()** and trigger-based state with **withLatestFrom()**.
- Choose between **zip()**, **forkJoin()**, **merge()**, and **concat()**.
- Build a derived Angular view stream from form or service Observables.
- Identify when a combination waits for an initial value or for source completion.

## What an operator does

An operator takes a source Observable and returns another Observable. Calling **pipe()** connects operator functions in order; it does not begin a subscription or mutate the original source.

~~~ts
import { map, of } from 'rxjs';

const prices$ = of(10, 20, 30);

const pricesWithTax$ = prices$.pipe(
  map((price) => price * 1.08),
);
~~~

Here, **prices$** still describes the original values. **pricesWithTax$** describes the transformed values. A subscriber to the derived stream receives 10.8, 21.6, and 32.4. A cold source may run separately for each subscription, as discussed in the previous lesson.

With RxJS 7.2 and later, import most creation functions and operators from **rxjs**:

~~~ts
import { combineLatest, filter, map, startWith } from 'rxjs';
~~~

The older **rxjs/operators** import path remains in existing code, but the RxJS guide recommends importing most operators from **rxjs**.

## Transform values with map()

**map(project)** calls a projection function once for each value and emits the returned value. It is similar to **Array.prototype.map()**, but values can arrive over time.

~~~ts
import { map, of } from 'rxjs';

interface Product {
  name: string;
  price: number;
}

const products$ = of<Product>(
  { name: 'Notebook', price: 4 },
  { name: 'Pen', price: 2 },
);

const productLabels$ = products$.pipe(
  map((product) => `${product.name}: $${product.price.toFixed(2)}`),
);

productLabels$.subscribe((label) => console.log(label));
~~~

A projection can change the Observable's value type. **Observable<Product>** becomes **Observable<string>** in this example. Keep projection functions focused and predictable; perform ordinary value conversion here, and use a side-effect operator such as **tap()** only when you specifically need to observe a value without changing it.

A projection can throw. An exception from **map()** becomes an error notification on the output stream, so fallible work still needs an error-handling strategy.

## Keep matching values with filter()

**filter(predicate)** emits only source values for which the predicate returns true. It does not transform the values that pass.

~~~ts
import { filter, map, of } from 'rxjs';

const scores$ = of(42, 68, 91, 57, 84);

const passingLabels$ = scores$.pipe(
  filter((score) => score >= 60),
  map((score) => `Passed with ${score}`),
);

passingLabels$.subscribe((label) => console.log(label));
~~~

The output is **Passed with 68**, **Passed with 91**, and **Passed with 84**. The order of operators matters: filtering before mapping tests the original number; mapping before filtering tests the projected value.

## Normalize and seed input streams

Input streams often produce values that are equivalent for the screen, such as a search query with extra spaces or different letter casing. Normalize first, then use **distinctUntilChanged()** to suppress consecutive duplicates.

~~~ts
import { distinctUntilChanged, map, startWith } from 'rxjs';

const searchText$ = searchControl.valueChanges.pipe(
  startWith(searchControl.value),
  map((text) => text.trim().toLocaleLowerCase()),
  distinctUntilChanged(),
);
~~~

**startWith(value)** emits a value immediately when a subscription begins, before forwarding values from its source. This is useful with Angular form **valueChanges**, which does not emit the control's existing value just because a consumer subscribes.

The default comparison in **distinctUntilChanged()** uses equality suited to primitive values. If the stream emits a newly created object each time, two objects with the same fields are still different references. For objects, compare a stable key or provide a comparator:

~~~ts
const query$ = searchControl.valueChanges.pipe(
  map((text) => text.trim()),
  distinctUntilChanged((previous, current) => previous === current),
);
~~~

Use only the comparison rule that matches the state you mean to treat as unchanged.

## Combine streams with combineLatest()

**combineLatest(inputs)** waits for every input Observable to emit at least once. After that, it emits the latest value from each input whenever any one of them emits.

This suits derived screen state: a result list can be recomputed when either the data or a filter changes.

~~~ts
import {
  combineLatest,
  distinctUntilChanged,
  map,
  startWith,
} from 'rxjs';

interface Task {
  title: string;
  done: boolean;
}

type TaskFilter = 'all' | 'open' | 'done';

declare const tasks$: Observable<readonly Task[]>;
declare const filter$: Observable<TaskFilter>;
declare const query$: Observable<string>;

const visibleTasks$ = combineLatest({
  tasks: tasks$,
  filter: filter$.pipe(startWith('all' as TaskFilter)),
  query: query$.pipe(
    startWith(''),
    map((query) => query.trim().toLocaleLowerCase()),
    distinctUntilChanged(),
  ),
}).pipe(
  map(({ tasks, filter, query }) =>
    tasks.filter((task) => {
      const matchesFilter =
        filter === 'all' ||
        (filter === 'done' ? task.done : !task.done);
      const matchesQuery = task.title.toLocaleLowerCase().includes(query);

      return matchesFilter && matchesQuery;
    }),
  ),
);
~~~

The object form keeps each input's role visible in the emitted value. The array form is useful when positional values are clear:

~~~ts
const total$ = combineLatest([subtotal$, taxRate$]).pipe(
  map(([subtotal, taxRate]) => subtotal * (1 + taxRate)),
);
~~~

### The initial-value rule

Until each input has emitted once, **combineLatest()** has no complete set of latest values and emits nothing. A form control's **valueChanges** is a common source that needs an initial value. **startWith(control.value)** seeds it so the combined view can calculate immediately.

For finite sources, **combineLatest()** completes after all inputs complete. If an input completes without emitting, the combination cannot produce a full tuple and completes without a value. A non-completing input is fine for ongoing UI state, but it means the combined stream normally remains active.

## Use withLatestFrom() for source-driven actions

**withLatestFrom(other)** also combines values, but only when its source Observable emits. The other streams provide their most recent values; they do not trigger an output by themselves.

~~~ts
import { Subject, withLatestFrom } from 'rxjs';

declare const currentDraft$: Observable<string>;

const saveClicks$ = new Subject<void>();

const saveRequests$ = saveClicks$.pipe(
  withLatestFrom(currentDraft$),
);
~~~

A click emits a pair containing the click and the latest draft text. A draft edit alone emits nothing from **saveRequests$**. The companion stream must have emitted at least once before a source click can produce output.

Use **combineLatest()** for “recalculate whenever any state changes.” Use **withLatestFrom()** for “when this action occurs, attach the current state.”

## Combine values by index with zip()

**zip(inputs)** pairs the first value from every source, then the second from every source, and so on. It waits until it can form a complete group at the same index.

~~~ts
import { of, zip } from 'rxjs';

const names$ = of('Ada', 'Grace');
const ids$ = of(10, 20, 30);

zip(names$, ids$).subscribe(([name, id]) => {
  console.log(name, id);
});
~~~

The output pairs **Ada** with 10 and **Grace** with 20. The unmatched 30 has no name to pair with. **zip()** is useful when streams represent corresponding positions; it is usually not the right choice for combining changing UI state.

## Wait for final values with forkJoin()

**forkJoin(inputs)** waits until every source has emitted and completed, then emits one array or object containing each source's last value. It is commonly useful when a group of finite requests should be presented together.

~~~ts
import { forkJoin } from 'rxjs';

declare const profile$: Observable<Profile>;
declare const settings$: Observable<Settings>;

const initialPageData$ = forkJoin({
  profile: profile$,
  settings: settings$,
});
~~~

The output shape is an object with **profile** and **settings** properties. If either input never completes, **forkJoin()** never emits. If an input completes without emitting, there is no complete set of final values to output. For streams that keep updating, choose **combineLatest()** instead.

## Forward values with merge() or concat()

These creation functions both combine multiple sources into one stream, but their subscription order differs.

### merge(): allow sources to emit as they arrive

**merge(a$, b$)** subscribes to its inputs and forwards values from whichever source emits. It does not pair values or keep a latest-value object.

~~~ts
import { merge } from 'rxjs';

const userMessages$ = merge(localMessages$, serverMessages$);
~~~

The result completes after all input streams complete. With asynchronous sources, values interleave according to their arrival times. Use **merge()** when events from multiple sources belong in one event stream.

### concat(): wait for each source to complete in order

**concat(a$, b$)** subscribes to the next source only after the current source completes.

~~~ts
import { concat, of } from 'rxjs';

concat(
  of('Load cached data'),
  of('Load fresh data'),
).subscribe((message) => console.log(message));
~~~

The values appear in source order. If the first source never completes, **concat()** never subscribes to the second source. Use it for finite sequences that must be played one after another; do not use it to combine ongoing UI state.

## Quick comparison

| Function | When does it emit? | Useful for |
| --- | --- | --- |
| **combineLatest()** | After all sources emit once; then whenever any source emits | Recomputing derived state from current inputs |
| **withLatestFrom()** | Only when the source Observable emits, after companions have values | Attaching current state to a click, submit, or other action |
| **zip()** | When each source has a value at the same index | Pairing corresponding sequences |
| **forkJoin()** | Once, after all sources emit and complete | Waiting for final results from finite sources |
| **merge()** | Whenever any source emits | Forwarding events from concurrent sources |
| **concat()** | From each source in order, after the previous completes | Sequencing finite streams |

Do not select an operator only because its name sounds like “combine.” Decide whether you need latest state, an action trigger, index pairing, final values, concurrent events, or strict sequence.

## Use a derived Observable in an Angular view

Expose a derived stream from a component or service and let Angular own its subscription. **AsyncPipe** can render an Observable directly:

~~~ts
readonly visibleTasks$ = this.taskService.visibleTasks$;
~~~

~~~html
@if (visibleTasks$ | async; as tasks) {
  @for (task of tasks; track task.title) {
    <p>{{ task.title }}</p>
  }
}
~~~

Alternatively, create a signal once with **toSignal()** when the component needs synchronous reads:

~~~ts
readonly visibleTasks = toSignal(this.taskService.visibleTasks$, {
  initialValue: [],
});
~~~

The example that follows builds this pattern with filter and search streams. Prefer a derived Observable when the inputs are already Observables; use Angular signals where they fit the component's state model. Avoid manually subscribing just to copy a stream's latest value into another property.

## Common mistakes

- **Expecting pipe() to start work:** it creates a derived Observable; subscription is still required.
- **Mutating the source in map():** return a projected value and avoid changing objects owned by other consumers.
- **Using map() to remove values:** use **filter()** to decide which values pass.
- **Combining valueChanges without a starting value:** seed a control's current value before **combineLatest()**.
- **Using combineLatest() for a button action:** companion changes would also trigger output; use **withLatestFrom()** on the action stream.
- **Using zip() for live form state:** it pairs by index and can buffer unmatched values; use **combineLatest()** for latest state.
- **Using forkJoin() with an ongoing stream:** it waits for completion; choose an operator meant for ongoing inputs.
- **Using concat() with a never-ending source:** later sources cannot start until earlier sources complete.
- **Assuming merge() preserves a fixed interleaving:** asynchronous emissions arrive according to their timing.
- **Expecting distinctUntilChanged() to compare object fields:** provide a comparator or compare a stable property.
- **Forgetting errors can still occur:** a projection or source can fail and send an error through the resulting Observable.
- **Mixing up combination and flattening:** **switchMap()**, **mergeMap()**, **concatMap()**, and **exhaustMap()** handle Observables emitted by another Observable; they are introduced in the next lesson.

## Further reading

- [RxJS operators and pipe semantics](https://rxjs.dev/guide/glossary-and-semantics)
- [RxJS importing guide](https://rxjs.dev/guide/importing)
- [map() API](https://rxjs.dev/api/operators/map)
- [combineLatest() API](https://rxjs.dev/api/index/function/combineLatest)
- [withLatestFrom() API](https://rxjs.dev/api/operators/withLatestFrom)
- [forkJoin() API](https://rxjs.dev/api/index/function/forkJoin)
- [zip() API](https://rxjs.dev/api/index/function/zip)
- [Angular AsyncPipe API](https://angular.dev/api/common/AsyncPipe)
- [Angular toSignal() API](https://angular.dev/api/core/rxjs-interop/toSignal)

## Continue learning

- Previous: [RxJS Observables, subscriptions, and stream basics](057-rxjs-observables-subscriptions-and-stream-basics.md)
- Q&A: [RxJS operators for mapping and combining streams](../qnas/058-rxjs-operators-for-mapping-and-combining-streams.md)
- Example: [RxJS operators for mapping and combining streams](../examples/058-rxjs-operators-for-mapping-and-combining-streams.md)
- Next: [Flattening operators, cancellation, and concurrency](059-flattening-operators-cancellation-and-concurrency.md)
