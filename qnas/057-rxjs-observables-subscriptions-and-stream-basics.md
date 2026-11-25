# Q&A: RxJS Observables, subscriptions, and stream basics

These answers cover the Observable contract, how subscribing starts observation, how cancellation and teardown work, and how common RxJS sources behave in Angular. See the [lesson](../lessons/057-rxjs-observables-subscriptions-and-stream-basics.md) for the full explanation and the [example](../examples/057-rxjs-observables-subscriptions-and-stream-basics.md) for a runnable Angular demo.

## 1. What is an Observable?

An Observable is a description of a producer that can send values to a consumer over time. It can emit no values, one value, or many values. A consumer starts observing it by calling **subscribe()**.

An Observable is not necessarily a stored collection or a running task. The source determines when its producer executes and whether work is shared.

## 2. What is an Observer?

An Observer is the consumer's set of notification handlers. It can handle:

- **next(value)** when a value arrives.
- **error(reason)** when the execution fails.
- **complete()** when the source finishes successfully.

A subscription can use an object with these named callbacks:

~~~ts
source$.subscribe({
  next: (value) => console.log(value),
  error: (reason: unknown) => console.error(reason),
  complete: () => console.log('Finished'),
});
~~~

## 3. What is the Observable notification contract?

An execution may send zero or more **next** values, followed by at most one terminal notification: either **error** or **complete**. It cannot continue sending values after either terminal notification.

A consumer can unsubscribe before the source terminates.

## 4. What does subscribe() do?

It connects an Observer to the source and starts or joins an execution, depending on that source's behavior. It returns a **Subscription** that represents the observation.

For many cold Observables, every call to **subscribe()** starts a separate producer execution.

## 5. Does creating an Observable start it?

Usually not for a cold Observable. The Observable describes how to start the producer, and subscribing starts it.

Hot sources can have a producer that is already running before a consumer subscribes, so always consider the actual source.

## 6. Is every Observable asynchronous?

No. **of(1, 2, 3)** emits synchronously during subscription, while **interval(1000)** emits later. An Observable describes a notification sequence; it does not promise a particular scheduler or asynchronous timing.

Do not rely on values being asynchronous unless the source or scheduler guarantees it.

## 7. What does the dollar sign in names such as users$ mean?

It is a naming convention used by many RxJS developers to mark a variable as an Observable. It has no special meaning to TypeScript or JavaScript.

## 8. What is a Subscription?

A **Subscription** is a disposable handle for an Observable execution. Call **unsubscribe()** to stop receiving notifications and release the resources owned by that execution.

~~~ts
const subscription = interval(1000).subscribe((value) => {
  console.log(value);
});

subscription.unsubscribe();
~~~

## 9. What is the difference between completion and unsubscription?

**complete()** is a notification from the producer that it finished successfully. **unsubscribe()** is a request from the consumer to stop observing.

Unsubscription does not call the Observer's **complete** callback. Both completion and unsubscription can cause producer teardown to run.

## 10. What happens when I unsubscribe?

RxJS stops delivering values to that subscriber and runs the source's teardown for that execution. If the source can cancel work, the teardown should release it, such as clearing a timer or removing an event listener.

Whether the underlying work can actually be cancelled depends on the source.

## 11. Does unsubscribe() cancel a Promise?

Not generally. If **from(promise)** is unsubscribed, that subscriber stops receiving notifications, but the Promise's underlying operation usually continues because native Promises have no built-in general cancellation method.

Use an API with an explicit cancellation signal when the work itself must stop.

## 12. What is teardown logic?

Teardown is cleanup returned by an Observable's producer. RxJS runs it when the execution completes, errors, or is unsubscribed.

~~~ts
const clock$ = new Observable<number>((subscriber) => {
  const timerId = setInterval(() => subscriber.next(Date.now()), 1000);

  return () => clearInterval(timerId);
});
~~~

A custom producer should release every resource it starts.

## 13. Does the complete callback run if I unsubscribe?

No. The producer did not send **complete**; the consumer stopped the subscription. The teardown runs, but the Observer's **complete** callback does not.

Use teardown or a finalization mechanism for cleanup that must occur on completion, error, and cancellation.

## 14. What is the difference between error and complete?

**error(reason)** reports failure and ends the execution. **complete()** ends it successfully. Only one terminal notification can occur.

An error is not followed by completion for that execution.

## 15. Should I always provide an error handler?

Provide an error path for sources that can fail. Without one, RxJS reports the error as unhandled. A one-value local source that cannot fail may use a short next-only callback, but APIs such as HTTP should be treated as fallible.

~~~ts
request$.subscribe({
  next: (response) => this.show(response),
  error: (reason: unknown) => this.showError(reason),
});
~~~

Retry and recovery strategies are covered in the later error-handling lesson.

## 16. What is a cold Observable?

A cold Observable creates a producer for each subscription. Each subscriber generally gets an independent execution.

For example, **interval(1000)** creates a separate interval timer for each subscription.

## 17. What is a hot Observable?

A hot Observable connects observers to a producer that exists independently of any one subscription. Subscribers may observe the same ongoing source and can miss values emitted before they subscribe.

A DOM event source or a **Subject** can have hot/multicast behavior.

## 18. Are cold and hot separate Observable types?

No. They are terms for producer behavior, not different TypeScript classes or generic types. A source's behavior may change when it is shared or connected using RxJS mechanisms.

## 19. Does subscribing twice share one HTTP request?

No, not for a normal Angular **HttpClient** request Observable. It is cold, so each subscription sends a new backend request.

If several consumers need to share one request or cached result, add an intentional sharing or caching strategy rather than expecting subscriptions to share automatically.

## 20. What does of() emit?

**of(a, b, c)** emits each argument as a separate value and then completes.

~~~ts
of('red', 'green').subscribe((color) => console.log(color));
~~~

This emits **red**, then **green**, then completion.

## 21. What happens with of(array) versus from(array)?

**of(array)** emits the entire array as one value. **from(array)** emits each array item as a separate value.

~~~ts
of([1, 2, 3]);   // One value: the array
from([1, 2, 3]);  // Three values: 1, then 2, then 3
~~~

## 22. What can from() convert?

**from()** accepts supported inputs such as arrays, other iterables, and Promises. It emits their values through an Observable and completes when that input finishes.

For a Promise, it emits the resolved value once or sends its rejection through the error notification.

## 23. What does interval() emit?

**interval(periodMs)** emits a counter beginning at zero, with values separated by the requested period. It does not complete by itself, so keep a subscription handle or use an Angular-aware lifecycle cleanup approach.

## 24. What does timer() emit?

**timer(delayMs)** waits, emits zero once, and completes. With a second period argument, it begins a repeating sequence after the initial delay.

A repeating timer needs the same cleanup planning as **interval()**.

## 25. What does fromEvent() do?

It adapts a compatible event target into an Observable. Subscribing attaches an event listener, and unsubscribing removes that subscriber's listener.

For ordinary Angular template interactions, use a template event binding unless the event needs to participate in a stream.

## 26. What does AsyncPipe do?

The Angular **AsyncPipe** subscribes to an Observable or Promise in a template and exposes its latest value. It marks the view for checking when new values arrive and cleans up its subscription when the component is destroyed or the bound reference changes.

Its result may be **null** before an asynchronous source emits.

## 27. What does toSignal() do with an Observable?

**toSignal()** subscribes to an Observable and exposes the latest value through a signal. By default, the subscription is cleaned up with the injection scope used to create it.

Create it once during component or service setup, then reuse the returned signal.

## 28. Should I manually subscribe to display a value in a template?

Usually not. Prefer **AsyncPipe** for a template-bound Observable or **toSignal()** when signal reads fit the component. Manual subscription is useful when imperative side effects or custom processing are needed, but it requires a clear cleanup owner.

## 29. Can two subscriptions to a cold source have different teardown?

Yes. Each subscription starts its own producer and gets its own teardown. Unsubscribing one subscription releases only that execution; other subscriptions remain active.

## 30. Can subscriptions be grouped?

Yes. Add child subscriptions or teardown callbacks to a parent **Subscription**. Unsubscribing the parent unsubscribes the children and runs their teardown.

In Angular component code, lifecycle-aware cleanup such as **takeUntilDestroyed()** is often clearer than manually maintaining a subscription group.

## 31. What happens if I subscribe without saving the returned Subscription?

The source still runs. You simply lose that handle for direct cancellation. If the source is ongoing, this can leave work active longer than intended.

Use **AsyncPipe**, **toSignal()**, **takeUntilDestroyed()**, or keep and dispose the handle according to the owner's lifetime.

## 32. How should I write subscribe() callbacks?

Use the observer object form when handling multiple notification types. It names each callback and avoids positional ambiguity.

~~~ts
source$.subscribe({
  next: handleValue,
  error: handleError,
  complete: handleCompletion,
});
~~~

A next-only callback remains convenient for a simple source that cannot fail or complete with any work requiring observation.

## 33. What is a Subject?

A **Subject** is both an Observable that can be subscribed to and an Observer that can receive values through **next()**, **error()**, and **complete()**. It can multicast values to current subscribers.

Do not introduce a Subject automatically just to store values. Use a plain Observable for read-only streams and choose a Subject when code genuinely needs an imperative producer or multicast behavior.

## 34. How do I know whether a stream needs cleanup?

Ask whether the source completes by itself. Finite sources such as **of()** and a typical HTTP response complete; ongoing sources such as **interval()**, event listeners, and WebSockets may not.

For component-owned work, prefer **AsyncPipe**, **toSignal()**, or **takeUntilDestroyed()**. For custom sources, return teardown that releases their resources.

## Further reading

- [RxJS glossary and semantics](https://rxjs.dev/guide/glossary-and-semantics)
- [RxJS Subscription guide](https://rxjs.dev/guide/subscription)
- [Observer API](https://rxjs.dev/api/index/interface/Observer)
- [Subscription API](https://rxjs.dev/api/index/class/Subscription)
- [RxJS subscribe argument guidance](https://rxjs.dev/deprecations/subscribe-arguments)
- [Angular AsyncPipe API](https://angular.dev/api/common/AsyncPipe)
- [Angular toSignal() API](https://angular.dev/api/core/rxjs-interop/toSignal)
- [Angular HttpClient Observables](https://angular.dev/guide/http/making-requests)

## Continue learning

- Lesson: [RxJS Observables, subscriptions, and stream basics](../lessons/057-rxjs-observables-subscriptions-and-stream-basics.md)
- Example: [RxJS Observables, subscriptions, and stream basics](../examples/057-rxjs-observables-subscriptions-and-stream-basics.md)
- Next lesson: [RxJS operators for mapping and combining streams](../lessons/058-rxjs-operators-for-mapping-and-combining-streams.md)
