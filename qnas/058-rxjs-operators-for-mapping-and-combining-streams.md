# Q&A: RxJS operators for mapping and combining streams

These answers explain how to compose RxJS operators, transform or filter values, and choose a combination strategy. See the [lesson](../lessons/058-rxjs-operators-for-mapping-and-combining-streams.md) for the full walkthrough and the [example](../examples/058-rxjs-operators-for-mapping-and-combining-streams.md) for a runnable Angular demo.

## 1. What is an RxJS operator?

An operator is a function that takes an Observable and returns a new Observable. It can transform values, filter them, combine streams, or alter notification behavior.

Operators compose with **pipe()**.

## 2. What does pipe() do?

**pipe()** passes a source Observable through operator functions in the order written and returns the resulting Observable.

~~~ts
const labels$ = values$.pipe(
  map((value) => value * 2),
  filter((value) => value > 10),
);
~~~

The output of **map()** becomes the input to **filter()**.

## 3. Does pipe() subscribe to an Observable?

No. **pipe()** creates a derived Observable. A subscription is still needed to start observing a cold source.

Angular's **AsyncPipe** and **toSignal()** also subscribe when they consume an Observable.

## 4. Does pipe() change the source Observable?

No. The source remains as it was. **pipe()** returns a new Observable that describes the transformed sequence.

~~~ts
const source$ = of(1, 2);
const doubled$ = source$.pipe(map((value) => value * 2));
~~~

A subscriber to **source$** receives 1 and 2; a subscriber to **doubled$** receives 2 and 4.

## 5. What does map() do?

**map(project)** calls a projection function for each emitted value and emits the result.

~~~ts
const names$ = users$.pipe(map((user) => user.name));
~~~

If the source is **Observable<User>**, this projection produces an **Observable<string>**.

## 6. Does map() work like Array.prototype.map()?

The transformation idea is similar. Array **map()** transforms each item in an array immediately; RxJS **map()** transforms each value when the Observable emits it over time.

## 7. When should I use filter() instead of map()?

Use **filter(predicate)** to decide whether a value should pass. Use **map(project)** to change a value.

~~~ts
const passingScores$ = scores$.pipe(
  filter((score) => score >= 60),
);
~~~

This emits the original passing scores. Add **map()** if those scores should also be converted to another shape.

## 8. Does filter() change the value that passes?

No. It either forwards the original value or drops it. Use **map()** for transformation.

## 9. Does operator order matter?

Yes. Each operator receives values from the operator before it.

~~~ts
const result$ = numbers$.pipe(
  filter((number) => number > 0),
  map((number) => number * 2),
);
~~~

Here the predicate checks the original number and only positive numbers are doubled.

## 10. What does startWith() do?

**startWith(value)** emits the supplied value or values immediately when a subscription begins, then forwards values from the source.

It is useful when a stream such as a form control's **valueChanges** does not emit its current value automatically.

## 11. Why seed valueChanges before using combineLatest()?

**combineLatest()** waits for every input to emit at least once. Angular form **valueChanges** reports changes after they happen, not the current control value just because the stream was subscribed to.

~~~ts
const query$ = searchControl.valueChanges.pipe(
  startWith(searchControl.value),
);
~~~

The seed lets the combination calculate an initial result.

## 12. What does distinctUntilChanged() compare?

By default, it compares each value with the previously emitted value and suppresses a value considered equal to the previous one. For primitive values such as strings and numbers, the default comparison is usually what you expect.

## 13. Why does distinctUntilChanged() still emit similar objects?

Two separately created objects are different references, even if their fields contain the same values. Provide a comparator or compare a stable primitive field.

~~~ts
const usersById$ = users$.pipe(
  distinctUntilChanged((previous, current) => previous.id === current.id),
);
~~~

## 14. What happens if a map() projection throws?

The derived Observable sends an error notification and terminates unless the stream has an appropriate recovery strategy. Keep projection functions predictable and handle fallible work in the stream's error-handling design.

## 15. What does combineLatest() do?

It remembers the latest value from each input. After every input has emitted once, any input emission produces a new combined result using all current values.

Use it when a result should refresh as any part of its input state changes.

## 16. Does combineLatest() emit immediately?

Only if each input can supply an initial value. If one input has not emitted, **combineLatest()** waits.

Seed sources that need an initial value with **startWith()**, or use sources such as **BehaviorSubject** that already emit their current value to a new subscriber.

## 17. What does combineLatest() emit with an object of sources?

It emits an object with the same keys, where each property holds the latest value from the matching input.

~~~ts
const viewModel$ = combineLatest({
  user: user$,
  permissions: permissions$,
});
~~~

This can be more readable than remembering tuple positions.

## 18. What does combineLatest() do when one input changes?

Once all inputs have emitted, it emits a new combination using the changed input's value and the latest values remembered from all other inputs.

## 19. When does combineLatest() complete?

After its inputs have emitted values, the output normally completes when all input Observables complete. An ongoing input can keep the combined stream active.

If an input completes without ever emitting, a complete combination can never be formed, so no combined value is emitted.

## 20. When should I use combineLatest() for UI state?

Use it for a derived view that depends on multiple current values, such as products plus category filter plus search text. Any of those inputs changing should recalculate the view.

## 21. What does withLatestFrom() do?

It combines each emission from a chosen source with the latest value from one or more companion streams. Only source emissions trigger output.

~~~ts
const saveRequest$ = saveClicks$.pipe(
  withLatestFrom(draft$),
);
~~~

Editing the draft alone does not emit a save request.

## 22. What is the main difference between combineLatest() and withLatestFrom()?

**combineLatest()** emits when any input changes after all inputs have emitted. **withLatestFrom()** emits only when its source emits, using the latest companion values.

Choose based on which stream should trigger the result.

## 23. Can withLatestFrom() ignore an early source event?

Yes. It cannot produce a combined tuple until each companion Observable has emitted once. A source event that arrives before that point does not produce an output.

Seed a companion stream if the action should be able to use a default value.

## 24. What does zip() do?

**zip()** pairs values by position: first with first, second with second, and so on. It waits for each source to have the next item in that positional group.

Use it for corresponding sequences, not for latest UI state.

## 25. What if zip() sources emit different numbers of values?

Only complete positional groups can be emitted. Extra values without a corresponding value from the other source remain unmatched. A completed source with no possible next pair means no further groups can be made.

## 26. What does forkJoin() do?

**forkJoin()** waits for all inputs to emit and complete. It then emits one array or object containing the last value from every source and completes.

~~~ts
const pageData$ = forkJoin({
  profile: profileRequest$,
  settings: settingsRequest$,
});
~~~

## 27. When is forkJoin() useful?

It is useful when several finite operations should finish and you only need their final values together, such as a set of one-response HTTP requests.

Use another strategy if the inputs are ongoing or intermediate changes matter.

## 28. Why does forkJoin() never emit with an interval?

An **interval()** does not complete by itself. **forkJoin()** waits for every source to complete before producing its one final result.

Use **combineLatest()** for live streams or explicitly bound the interval to a finite number of values if that matches the requirement.

## 29. What happens if an input to forkJoin() completes without a value?

There is no final value for that input, so **forkJoin()** cannot produce its complete result object or array. Make sure each input has the emission and completion behavior the operation requires.

## 30. What does merge() do?

**merge()** subscribes to several inputs and forwards all their values into one stream as they arrive. It does not pair values or wait to calculate a current-state object.

~~~ts
const allNotifications$ = merge(emailAlerts$, inAppAlerts$);
~~~

The output completes after its inputs complete.

## 31. Does merge() guarantee which source emits first?

No fixed interleaving is guaranteed for asynchronous sources. Values arrive according to each source's timing. Synchronous sources can emit in subscription order, but do not rely on that to coordinate asynchronous work.

## 32. What does concat() do?

**concat()** subscribes to the first input and forwards its values. After it completes, it subscribes to the next input, continuing in order.

It is suited to finite streams that must run sequentially.

## 33. Why can concat() leave a later source waiting forever?

The next source is not subscribed until the current source completes. If an earlier source is ongoing and never completes, later sources never start.

## 34. How do I choose among zip(), combineLatest(), and forkJoin()?

- Choose **zip()** to pair values by index.
- Choose **combineLatest()** to recompute with latest values as inputs change.
- Choose **forkJoin()** to emit each finite input's final value once all have completed.

## 35. How do I choose between merge() and concat()?

Choose **merge()** to forward emissions from subscribed sources as they arrive. Choose **concat()** when a later source must wait until the prior source completes.

## 36. Are combineLatest() and combineLatestWith() the same?

They express related combination behavior in different forms. **combineLatest()** is a creation function that takes independent Observables. **combineLatestWith()** is a pipeable operator that combines the source with the listed additional Observables.

Use the form that makes the data flow clearest. Avoid the deprecated pipeable **combineLatest** import from the old **rxjs/operators** path.

## 37. Should I import operators from rxjs/operators?

In modern RxJS, most operator functions are exported from **rxjs**, which is the preferred import path. Existing code may still use **rxjs/operators**, but newer code can use:

~~~ts
import { filter, map, startWith } from 'rxjs';
~~~

## 38. How can I combine form state into one view model?

Seed each form stream with the control's current value, combine those streams with data, then map the result into the shape the view needs.

~~~ts
const filters$ = combineLatest({
  query: queryControl.valueChanges.pipe(startWith(queryControl.value)),
  category: categoryControl.valueChanges.pipe(
    startWith(categoryControl.value),
  ),
  products: products$,
});
~~~

A following **map()** can filter or project that object into a view model.

## 39. Should I manually subscribe to a combined stream in a component?

Not only to display its value. Expose the stream to **AsyncPipe**, or create one **toSignal()** during setup. If imperative subscription work is necessary, give it an owner and lifecycle cleanup.

## 40. Are combineLatest() and switchMap() interchangeable?

No. **combineLatest()** combines the latest values from independent inputs. **switchMap()** maps each source value to another Observable and switches to the newest inner stream. Higher-order mapping and cancellation are covered in the next chapter.

## 41. Can a combination stream fail?

Yes. Any input can send an error, and a projection used by **map()** can throw. Design an error path that matches the UI or service behavior; combinations do not remove the need for error handling.

## 42. What is a reliable way to choose an operator?

Ask what relationship the output should have to its inputs:

1. Transform each value? Use **map()**.
2. Keep only matching values? Use **filter()**.
3. Recalculate from latest values whenever any changes? Use **combineLatest()**.
4. Run only when an action emits, attaching current state? Use **withLatestFrom()**.
5. Pair streams by their positions? Use **zip()**.
6. Wait for final results from finite sources? Use **forkJoin()**.
7. Forward independent events as they arrive? Use **merge()**.
8. Run finite sources one after another? Use **concat()**.

## Continue learning

- Lesson: [RxJS operators for mapping and combining streams](../lessons/058-rxjs-operators-for-mapping-and-combining-streams.md)
- Example: [RxJS operators for mapping and combining streams](../examples/058-rxjs-operators-for-mapping-and-combining-streams.md)
- Next lesson: [Flattening operators, cancellation, and concurrency](../lessons/059-flattening-operators-cancellation-and-concurrency.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="057-rxjs-observables-subscriptions-and-stream-basics.md">057 - RxJS Observables, subscriptions, and stream basics</a></td><td><a href="../lessons/058-rxjs-operators-for-mapping-and-combining-streams.md">Lesson</a></td><td><a href="../examples/058-rxjs-operators-for-mapping-and-combining-streams.md">Example</a></td><td><a href="059-flattening-operators-cancellation-and-concurrency.md">059 - Flattening operators, cancellation, and concurrency</a></td>
</tr>
  </tbody>
</table>
