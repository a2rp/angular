# RxJS Observables, subscriptions, and stream basics

An RxJS **Observable** describes a sequence of values that may arrive now, later, many times, or not at all. A consumer starts observing it by calling **subscribe()**. The returned **Subscription** represents that observation and can stop it with **unsubscribe()**.

Observables appear throughout Angular: **HttpClient** returns them, reactive forms expose them, and many router and framework APIs use them. This lesson covers the basic Observable contract, how an Observer receives notifications, how subscriptions start and stop work, and the difference between common Observable sources.

## Learning outcomes

After this lesson, you should be able to:

- Describe an Observable as a producer that sends zero or more values to a subscriber.
- Use **next**, **error**, and **complete** notifications correctly.
- Explain why **subscribe()** begins an Observable execution and returns a **Subscription**.
- Cancel ongoing work with **unsubscribe()** and explain producer teardown.
- Recognize that repeated subscriptions may start repeated executions.
- Distinguish cold producer behavior from shared or hot producer behavior.
- Create basic streams with **of**, **from**, **interval**, **timer**, and **fromEvent**.
- Use an Observable in an Angular template with **AsyncPipe** or a signal with **toSignal()**.
- Avoid starting subscriptions that have no clear owner or cleanup plan.

## What an Observable represents

An Observable describes how to connect a consumer to a producer. It is a reusable description of a possible execution; it is not necessarily a stored list of values, a running task, or a Promise.

~~~ts
import { Observable } from 'rxjs';

const message$: Observable<string> = new Observable((subscriber) => {
  subscriber.next('First value');
  subscriber.next('Second value');
  subscriber.complete();
});
~~~

Creating **message$** does not call the producer function. Subscribing asks the Observable to execute it:

~~~ts
message$.subscribe({
  next: (message) => console.log(message),
  complete: () => console.log('No more values'),
});
~~~

This particular producer is synchronous, so both values and the completion notification arrive before **subscribe()** returns. Other sources, such as **interval()**, produce values later.

The suffix **$** is a community naming convention that helps readers recognize an Observable. TypeScript does not give it special meaning.

## The Observable notification contract

An Observable can send three kinds of notifications:

- **next(value)** delivers a value.
- **error(reason)** reports a failure and ends the execution.
- **complete()** reports successful end of the sequence.

An execution may send zero or more **next** notifications and then at most one terminal notification: either **error** or **complete**. After a terminal notification, it sends no further notifications. A consumer can also unsubscribe before a terminal notification arrives.

~~~ts
const oneValue$ = new Observable<number>((subscriber) => {
  subscriber.next(42);
  subscriber.complete();
});
~~~

A subscription observer can be an object with named callbacks:

~~~ts
oneValue$.subscribe({
  next: (value) => console.log('Value:', value),
  error: (reason: unknown) => console.error('Failed:', reason),
  complete: () => console.log('Finished'),
});
~~~

For code that can fail, provide an **error** handler. If no handler is supplied, RxJS reports the error as unhandled. A **complete** callback is optional and runs only when the source sends **complete**.

### Completion is different from unsubscription

If the source calls **complete()**, the observer's **complete** callback runs. If the source calls **error(reason)**, the observer's **error** callback runs. If the consumer calls **unsubscribe()**, neither observer callback is called; the consumer is simply no longer interested.

Unsubscription can still run the producer's teardown. This distinction matters for cleanup logic. Do not use a **complete** callback as the only place to release work that might be cancelled.

## Subscribe to start an execution

Calling **subscribe()** connects an Observer to the producer. For a cold Observable, each subscription usually starts a new execution:

~~~ts
let executions = 0;

const source$ = new Observable<number>((subscriber) => {
  executions += 1;
  console.log('Producer execution:', executions);
  subscriber.next(executions);
  subscriber.complete();
});

source$.subscribe((value) => console.log('First subscriber:', value));
source$.subscribe((value) => console.log('Second subscriber:', value));
~~~

The producer runs twice and each subscription gets its own value. A second call to **subscribe()** is not automatically a request to replay or share the first result.

Some Observables represent a producer that already exists and is shared by several consumers, such as an event source or a **Subject**. In those cases, subscriptions can observe the same ongoing producer. Always check the specific source's behavior.

### Cold and hot describe producer behavior

A **cold** Observable creates a producer for each subscription. A **hot** Observable connects observers to a producer that exists outside an individual subscription. These words describe how a source behaves; they are not different TypeScript types.

Examples:

- **of('a', 'b')** creates a small sequence for each subscriber.
- **interval(1000)** starts a separate timer execution for each subscriber.
- **HttpClient.get()** sends a new request for each subscription.
- A DOM event Observable attaches a listener for each subscription.
- A **Subject** can multicast values that are pushed into it to its current subscribers.

Sharing and multicasting operators are covered later. Do not assume two subscriptions share work unless the source or an explicit sharing strategy does so.

## The Subscription object

**subscribe()** returns a **Subscription**:

~~~ts
import { interval } from 'rxjs';

const subscription = interval(1000).subscribe({
  next: (value) => console.log(value),
});

// Later, stop this execution:
subscription.unsubscribe();
~~~

A **Subscription** represents the resources held by an execution. It exposes **closed**, which becomes true after the subscription ends.

~~~ts
if (!subscription.closed) {
  subscription.unsubscribe();
}
~~~

Calling **unsubscribe()** more than once is safe. It will not cause the producer to restart or send a completion notification.

### Group related subscriptions

A parent **Subscription** can own child subscriptions or teardown callbacks. Unsubscribing the parent releases everything added to it:

~~~ts
import { interval, Subscription } from 'rxjs';

const group = new Subscription();

group.add(
  interval(1000).subscribe((value) => console.log('One:', value)),
);
group.add(
  interval(1500).subscribe((value) => console.log('Two:', value)),
);

// Stops both child executions.
group.unsubscribe();
~~~

Use a group when one owner intentionally starts several subscriptions. In Angular components, prefer lifecycle-aware cleanup such as **takeUntilDestroyed()**, **AsyncPipe**, or **toSignal()** when they fit. The later cleanup chapter covers those patterns in more depth.

## Teardown logic releases producer resources

An Observable producer can return a teardown function. RxJS runs it when the execution completes, errors, or is unsubscribed.

~~~ts
const heartbeat$ = new Observable<number>((subscriber) => {
  let beat = 0;
  const timerId = setInterval(() => subscriber.next(++beat), 1000);

  return () => {
    clearInterval(timerId);
    console.log('Heartbeat producer stopped');
  };
});
~~~

The teardown is tied to the subscription. If three consumers subscribe, each cold execution gets its own timer and its own teardown.

### A finite Observable with an explicit teardown

~~~ts
const countdown$ = new Observable<number>((subscriber) => {
  let count = 1;

  const timerId = setInterval(() => {
    subscriber.next(count);

    if (count === 5) {
      subscriber.complete();
    }

    count += 1;
  }, 1000);

  return () => clearInterval(timerId);
});
~~~

When the fifth value is emitted, **complete()** ends the execution and RxJS runs the returned teardown. If a consumer unsubscribes earlier, the same teardown clears the timer without calling the observer's **complete** callback.

Return teardown for timers, event listeners, sockets, and other resources started by a custom producer. Use RxJS creation functions such as **interval()** or **fromEvent()** instead of writing a custom Observable when one already expresses the source.

## Basic Observable creation functions

RxJS provides creation functions for common sources. Import them from **rxjs**.

### of(): emit provided values

**of()** emits each argument as a value, then completes:

~~~ts
import { of } from 'rxjs';

const colors$ = of('blue', 'green', 'red');

colors$.subscribe({
  next: (color) => console.log(color),
  complete: () => console.log('All colors emitted'),
});
~~~

Passing an array to **of()** emits that array as one value:

~~~ts
of(['blue', 'green', 'red']).subscribe((colors) => {
  console.log(colors); // One array value
});
~~~

### from(): convert an iterable or Promise

**from()** converts supported inputs, including arrays, iterables, and Promises:

~~~ts
import { from } from 'rxjs';

from(['a', 'b', 'c']).subscribe((letter) => console.log(letter));

from(Promise.resolve('Saved')).subscribe({
  next: (message) => console.log(message),
  error: (reason: unknown) => console.error(reason),
});
~~~

An array passed to **from()** emits its items one by one. A Promise emits its resolved value and then completes, or sends an error if the Promise rejects. Unsubscribing from an Observable created from a Promise stops delivery to that subscriber, but it cannot generally cancel the underlying Promise operation.

### interval(): repeat at a fixed period

**interval(periodMs)** emits an increasing counter beginning at zero. It does not complete on its own.

~~~ts
import { interval } from 'rxjs';

const seconds$ = interval(1000);

const subscription = seconds$.subscribe((seconds) => {
  console.log(seconds);
});

// Let a few values arrive, then stop the ongoing source.
setTimeout(() => subscription.unsubscribe(), 3500);
~~~

An ongoing source such as **interval()** needs an owner and a cancellation plan.

### timer(): wait once or start a periodic sequence

With one argument, **timer(delayMs)** waits and emits zero once, then completes:

~~~ts
import { timer } from 'rxjs';

timer(500).subscribe(() => console.log('Half a second passed'));
~~~

A second period argument makes **timer(delayMs, periodMs)** emit repeatedly, similar to an interval that starts after a delay. Repeated timer sources also need cleanup.

### fromEvent(): observe a browser event

**fromEvent(target, eventName)** creates an Observable from a compatible event target:

~~~ts
import { fromEvent } from 'rxjs';

const clicks$ = fromEvent<MouseEvent>(buttonElement, 'click');

const subscription = clicks$.subscribe((event) => {
  console.log('Clicked at:', event.clientX, event.clientY);
});

// Unsubscribe to remove the event listener.
subscription.unsubscribe();
~~~

The event listener is attached when a subscriber subscribes and is removed when that subscriber unsubscribes. In an Angular template, normal event binding is usually simpler for a button click. **fromEvent()** is useful when an event needs to participate in an RxJS stream.

## Observable versus Promise

| Observable | Promise |
| --- | --- |
| Can send zero, one, or many values over time | Settles once with a value or rejection |
| Consumer starts observing with **subscribe()** | Work typically starts when the Promise is created |
| Subscription can be cancelled | A Promise has no built-in general cancellation method |
| May be synchronous or asynchronous | Resolution is observed through Promise microtasks |
| Each subscription may create a separate producer | A Promise represents one settlement shared by its consumers |

These are useful general differences, not guarantees about every source. An Observable can wrap a hot producer or a Promise; inspect the source's behavior before assuming when work starts or whether it can be cancelled.

## Observables in Angular

### HttpClient returns Observables

Angular's **HttpClient** methods return Observables. For a typical HTTP request, no request is dispatched until someone subscribes. Subscribing a second time to the same request Observable normally sends another request.

~~~ts
readonly config$ = this.http.get<AppConfig>('/api/config');
~~~

The request can be consumed by a template or by code. Avoid subscribing once only to log or set a value and then accidentally subscribing again to fetch the same data. HTTP behavior and operators are covered in later chapters.

### AsyncPipe subscribes for the template

The **AsyncPipe** subscribes to an Observable in the template, exposes its latest value, and unsubscribes when the component is destroyed or the bound Observable reference changes.

~~~ts
readonly greeting$ = of('Hello from RxJS');
~~~

~~~html
<p>{{ greeting$ | async }}</p>
~~~

The pipe may produce **null** before an asynchronous source emits. Handle that initial state when the template requires a non-null value. Keep the Observable as a component property; avoid calling a method that creates a new Observable every change-detection pass.

### toSignal() provides a signal view

Angular's **toSignal()** subscribes to an Observable and exposes its latest value as a signal. It automatically cleans up the subscription with its injection scope by default.

~~~ts
readonly status = toSignal(this.status$, {
  initialValue: 'Starting',
});
~~~

Use **AsyncPipe** when a template needs the stream directly, and **toSignal()** when component logic or templates benefit from signal reads. Create the signal once and reuse it; do not call **toSignal()** repeatedly for the same Observable.

## A safe subscription checklist

Before calling **subscribe()**, answer these questions:

1. Who owns this subscription?
2. Does the source complete by itself, or can it continue indefinitely?
3. If the component or service is destroyed, what stops the execution?
4. Does every subscriber start a new producer or request?
5. Can the result be consumed with **AsyncPipe** or **toSignal()** instead?
6. If the source can fail, where is its **error** notification handled?

For a component-owned manual subscription, use an Angular-aware cleanup approach such as **takeUntilDestroyed()** or register it with the component's **DestroyRef**. Avoid unmanaged subscriptions in long-lived services.

## Common mistakes

- **Expecting an Observable to run when it is created:** a cold Observable normally starts on subscription.
- **Assuming every Observable is cold:** a hot or shared producer may already be running.
- **Subscribing multiple times without noticing:** each subscription can start a new execution or HTTP request.
- **Treating unsubscribe() as complete():** unsubscription does not call the observer's **complete** handler.
- **Forgetting teardown in a custom producer:** clear timers and detach listeners when the execution ends.
- **Leaving interval() running:** ongoing streams need an owner and cleanup.
- **Handling only next for a fallible source:** provide an error path or an appropriate recovery strategy.
- **Using a manual subscription only to display template data:** consider **AsyncPipe** or **toSignal()**.
- **Calling toSignal() in a template method:** create it once during setup and reuse it.
- **Assuming a Promise can be cancelled by unsubscribe():** the subscriber can stop receiving values, but the Promise's underlying work may continue.
- **Passing an array to of() expecting separate emissions:** **of(array)** emits one array; **from(array)** emits the array's items.

## Further reading

- [RxJS glossary and semantics](https://rxjs.dev/guide/glossary-and-semantics)
- [RxJS Subscription guide](https://rxjs.dev/guide/subscription)
- [Observer API](https://rxjs.dev/api/index/interface/Observer)
- [Subscription API](https://rxjs.dev/api/index/class/Subscription)
- [RxJS subscribe argument guidance](https://rxjs.dev/deprecations/subscribe-arguments)
- [Angular AsyncPipe API](https://angular.dev/api/common/AsyncPipe)
- [Angular toSignal() API](https://angular.dev/api/core/rxjs-interop/toSignal)
- [Making HTTP requests with Angular](https://angular.dev/guide/http/making-requests)

## Continue learning

- Previous: [Injection contexts and resource lifetimes](056-injection-contexts-and-resource-lifetimes.md)
- Q&A: [RxJS Observables, subscriptions, and stream basics](../qnas/057-rxjs-observables-subscriptions-and-stream-basics.md)
- Example: [RxJS Observables, subscriptions, and stream basics](../examples/057-rxjs-observables-subscriptions-and-stream-basics.md)
- Next: [RxJS operators for mapping and combining streams](058-rxjs-operators-for-mapping-and-combining-streams.md)
