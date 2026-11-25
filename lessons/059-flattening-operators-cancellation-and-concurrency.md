# Flattening operators, cancellation, and concurrency

An Observable can emit another Observable. That creates a higher-order stream: an outer stream of events, where each event starts an inner stream such as an HTTP request, timer, or save operation. A flattening operator chooses how those inner streams are subscribed to and how their values are forwarded.

The four common policies are **switchMap()**, **mergeMap()**, **concatMap()**, and **exhaustMap()**. Their key difference is what happens when a new outer value arrives while an inner operation is still running.

## Learning outcomes

After this lesson, you should be able to:

- Recognize an Observable that emits other Observables.
- Explain why **map()** alone does not flatten an inner Observable.
- Choose between switching, concurrent, sequential, and exhaust behavior.
- Describe when each operator unsubscribes from or ignores inner work.
- Use **switchMap()** for latest-query searches and understand HTTP cancellation.
- Limit **mergeMap()** concurrency when work should run in parallel.
- Preserve write order with **concatMap()** or block duplicate submits with **exhaustMap()**.
- Give component-owned subscriptions a lifecycle cleanup strategy.

## Higher-order Observables

Suppose an event stream contains user IDs, and each ID is used to request a user:

~~~ts
import { map, of } from 'rxjs';

const userIds$ = of(10, 20, 30);

const userRequests$ = userIds$.pipe(
  map((id) => this.http.get<User>('/api/users/' + id)),
);
~~~

The type of **userRequests$** is conceptually **Observable<Observable<User>>**: its values are request Observables, not User objects. A normal **map()** changes each outer value into an inner Observable, but it does not subscribe to or flatten those inner streams.

A flattening operator maps an outer value to an inner Observable and then applies a policy to those inner subscriptions:

~~~ts
const users$ = userIds$.pipe(
  switchMap((id) => this.http.get<User>('/api/users/' + id)),
);
~~~

Now **users$** emits User values from the inner request stream. The inner operation may be a request, timer, event stream, or any other Observable.

## Four policies for handling inner work

### switchMap(): keep only the newest inner stream

When a new outer value arrives, **switchMap()** unsubscribes from the previous inner Observable and subscribes to the new one. Only values from the newest inner stream are forwarded.

~~~ts
import { switchMap } from 'rxjs';

const results$ = searchTerms$.pipe(
  switchMap((term) => searchApi.search(term)),
);
~~~

This is a good fit when newer input makes older results irrelevant, such as type-ahead search, route-driven data, or a preview that should reflect the latest selection.

For Angular **HttpClient**, unsubscribing from an in-progress request aborts that client request. This helps prevent stale search responses from updating the view. Cancellation does not roll back server-side work that has already reached the server, so do not treat it as a transaction or use it casually for writes.

### mergeMap(): allow inner streams to run concurrently

**mergeMap()** subscribes to each projected inner Observable and forwards values as those inners emit. Inner work can overlap, and outputs can arrive in a different order from the outer inputs.

~~~ts
import { mergeMap } from 'rxjs';

const savedItems$ = itemsToSave$.pipe(
  mergeMap((item) => saveItem(item)),
);
~~~

By default, **mergeMap()** permits unlimited concurrency. A second argument limits the number of active inner subscriptions:

~~~ts
const savedItems$ = itemsToSave$.pipe(
  mergeMap((item) => saveItem(item), 3),
);
~~~

With concurrency set to 3, up to three saves run at once; additional values wait until a slot is available. Use **mergeMap()** when operations are independent and parallel completion is acceptable. Consider a concurrency limit when the source can produce work quickly.

### concatMap(): queue inner streams in order

**concatMap()** subscribes to one inner Observable at a time. It does not start the next inner until the current one completes, so inner work proceeds in source order.

~~~ts
import { concatMap } from 'rxjs';

const savedItems$ = itemsToSave$.pipe(
  concatMap((item) => saveItem(item)),
);
~~~

This suits ordered writes, autosave queues, or steps that depend on the previous operation finishing. If outer values arrive faster than inner work completes, **concatMap()** queues them. An unbounded or long-lived queue can use memory and create a growing delay.

**concatMap()** is equivalent to **mergeMap(project, 1)**: both allow only one active inner Observable, but **concatMap()** communicates the sequential policy more clearly.

### exhaustMap(): ignore new values while one inner is active

**exhaustMap()** starts an inner Observable for the first outer value. While that inner is active, it ignores additional outer values. After the inner completes, it can accept another value.

~~~ts
import { exhaustMap } from 'rxjs';

const loginResults$ = submitClicks$.pipe(
  exhaustMap(() => this.authApi.login(this.credentials)),
);
~~~

This can prevent repeated submit clicks from starting duplicate login requests. The ignored click is not queued. Once the request completes, a later click can start a new one.

## Compare the behavior

Assume event A starts a slow inner operation, then event B arrives before A finishes:

| Operator | What happens to A? | What happens to B? | Typical use |
| --- | --- | --- | --- |
| **switchMap()** | Unsubscribed when B arrives | Starts immediately and becomes current | Search, route changes, latest preview |
| **mergeMap()** | Continues | Starts concurrently | Independent uploads or parallel reads |
| **concatMap()** | Continues until complete | Waits in the queue | Ordered writes or sequential steps |
| **exhaustMap()** | Continues until complete | Ignored while A is active | Preventing duplicate submits |

The operator does not decide whether the work is important; your product requirements do. Choose based on whether old work should be cancelled, allowed to overlap, queued, or ignored.

## Use switchMap() for type-ahead search

A common Angular search stream starts from form input, trims and deduplicates the query, waits briefly for typing to pause, then switches to the newest HTTP request.

~~~ts
import { FormControl } from '@angular/forms';
import { HttpClient } from '@angular/common/http';
import {
  debounceTime,
  distinctUntilChanged,
  map,
  of,
  startWith,
  switchMap,
} from 'rxjs';

interface Product {
  id: number;
  name: string;
}

export class ProductSearchComponent {
  readonly queryControl = new FormControl('', { nonNullable: true });

  readonly products$ = this.queryControl.valueChanges.pipe(
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

  constructor(private readonly http: HttpClient) {}
}
~~~

The **debounceTime()** step reduces requests while someone is typing. **switchMap()** ensures that a newer query replaces the previous request subscription. Display **products$** with **AsyncPipe** or adapt it once with **toSignal()** so Angular owns the subscription lifetime.

Do not rely on **switchMap()** to undo a request that has already been processed by the server. It can stop the client subscription and abort an in-flight HttpClient request, but server-side effects may already have happened.

## Use concatMap() when writes must stay ordered

For independent read requests, parallel completion may be fine. For writes where order matters, serialize the operations:

~~~ts
const writeResults$ = edits$.pipe(
  concatMap((edit) => this.http.put<Document>(
    '/api/documents/' + edit.id,
    edit,
  )),
);
~~~

Each write starts after the previous request Observable completes. This preserves the order in which the edits reach the mapping, but it does not guarantee that a server applies data transactionally. The server API and its own concurrency rules still matter.

## Use mergeMap() for bounded parallel work

A batch can process several items at once without starting every request simultaneously:

~~~ts
const uploadResults$ = files$.pipe(
  mergeMap((file) => this.upload(file), 2),
);
~~~

At most two upload Observables are active at one time. Further files wait until one active upload completes. The result order follows inner emission timing, not necessarily the original file order. If output order matters, include an index or sort the completed results afterward.

## Use exhaustMap() for duplicate-submit protection

A click stream can be mapped to an HTTP submit operation:

~~~ts
const submitResults$ = submitClicks$.pipe(
  exhaustMap(() => this.http.post<Receipt>('/api/orders', this.order)),
);
~~~

While the request is active, further clicks are ignored rather than queued or run concurrently. The button should still show a loading state and communicate whether the action is in progress. UI disabling can improve usability, while **exhaustMap()** enforces the stream policy.

## Error and completion behavior

A flattening operator forwards inner values to the output. An inner error normally errors the flattened output and ends that subscription unless an error recovery operator handles it. If a long-lived outer stream should continue after one request fails, the recovery is often placed inside the projection so it handles that individual inner operation. The next chapter covers error handling in detail.

Completion depends on both levels. When the outer source completes, a flattening operator generally waits for its active inner work to finish before completing the result. **switchMap()** has already unsubscribed from earlier inners when newer values arrived; **concatMap()** may still have queued work to process; **mergeMap()** may still have several active inners; and **exhaustMap()** may still have its accepted inner in progress.

A never-ending inner stream can keep the flattened output alive. For example, mapping a click to **interval()** with **concatMap()** means the next click's interval remains queued forever unless each inner is bounded or cancelled intentionally.

## Manage Angular subscription lifetimes

The flattening policy controls inner operations; it does not automatically make every outer subscription safe for a component's lifetime. For a template-bound stream, prefer **AsyncPipe** or **toSignal()**. For imperative side effects in a component or directive, **takeUntilDestroyed()** completes the subscription when its Angular context is destroyed.

~~~ts
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

this.route.paramMap.pipe(
  switchMap((params) => this.pageApi.load(params.get('id'))),
  takeUntilDestroyed(),
).subscribe((page) => {
  this.pageState.set(page);
});
~~~

When called inside an injection context such as a constructor, **takeUntilDestroyed()** can obtain the current destruction context. Outside that context, pass a **DestroyRef** explicitly. Do not use a component's lifecycle as a substitute for choosing the right inner-stream policy.

## Common mistakes

- **Using map() and expecting a flattened result:** **map()** returns an Observable of Observables when its projection returns an Observable.
- **Choosing switchMap() for every request:** it cancels the previous subscription when a new source value arrives; that may discard work you must preserve.
- **Assuming cancellation undoes server work:** cancellation stops client observation and may abort transport, but it is not a server transaction rollback.
- **Using mergeMap() when order matters:** inner values can arrive in a different order from source values.
- **Leaving mergeMap() concurrency unlimited for an unbounded source:** too many operations may run at once; set a limit if needed.
- **Using concatMap() for a rapid, endless event source:** queued work can accumulate faster than it is processed.
- **Expecting concatMap() to run inner work in parallel:** it deliberately allows only one active inner stream.
- **Expecting exhaustMap() to queue clicks:** values arriving while the current inner is active are discarded.
- **Using exhaustMap() without clear feedback:** users should know the operation is in progress and when another attempt is accepted.
- **Assuming HTTP cancellation proves a write did not happen:** the server may already have processed the request.
- **Manually subscribing without a lifecycle owner:** use **AsyncPipe**, **toSignal()**, or **takeUntilDestroyed()** as appropriate.
- **Forgetting inner errors:** an error can terminate the flattened output; choose an error-handling strategy for the stream's intended lifetime.

## Further reading

- [RxJS switchMap() API](https://rxjs.dev/api/operators/switchMap)
- [RxJS mergeMap() API](https://rxjs.dev/api/operators/mergeMap)
- [RxJS concatMap() API](https://rxjs.dev/api/operators/concatMap)
- [RxJS exhaustMap() API](https://rxjs.dev/api/operators/exhaustMap)
- [RxJS mergeMap() concurrency guide](https://rxjs.dev/api/operators/mergeMap)
- [Angular HttpClient Observables and cancellation](https://angular.dev/guide/http/making-requests)
- [Angular takeUntilDestroyed() API](https://angular.dev/api/core/rxjs-interop/takeUntilDestroyed)
- [Angular RxJS cleanup guide](https://angular.dev/ecosystem/rxjs-interop/take-until-destroyed)

## Continue learning

- Previous: [RxJS operators for mapping and combining streams](058-rxjs-operators-for-mapping-and-combining-streams.md)
- Q&A: [Flattening operators, cancellation, and concurrency](../qnas/059-flattening-operators-cancellation-and-concurrency.md)
- Example: [Flattening operators, cancellation, and concurrency](../examples/059-flattening-operators-cancellation-and-concurrency.md)
- Next: [Error handling, cleanup, and takeUntilDestroyed](060-error-handling-cleanup-and-takeuntildestroyed.md)
