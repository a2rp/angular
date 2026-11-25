# Example: RxJS Observables, subscriptions, and stream basics

This Angular app lets you see the basic Observable contract in action:

- A cold Observable starts one producer execution for each subscription.
- **subscribe()** receives values and completion through an Observer object.
- A custom Observable returns teardown that clears its interval.
- You can stop a running stream before it completes.
- **AsyncPipe** subscribes to a finite Observable in the template and cleans it up.

The examples use RxJS creation functions and APIs available from **rxjs**. Start with a standalone Angular CLI app, then add or replace the files below.

## 1. Create a finite Observable with teardown

Create **src/app/counting-stream.ts**:

~~~ts
import { Observable } from 'rxjs';

export function createCountingStream(
  valueCount = 5,
  periodMs = 750,
): Observable<number> {
  return new Observable<number>((subscriber) => {
    if (valueCount <= 0) {
      subscriber.complete();
      return;
    }

    let value = 0;

    const timerId = setInterval(() => {
      value += 1;
      subscriber.next(value);

      if (value >= valueCount) {
        subscriber.complete();
      }
    }, periodMs);

    return () => {
      clearInterval(timerId);
      console.info('Counting stream teardown ran');
    };
  });
}
~~~

Each subscription to this cold source creates its own interval. When it emits the requested number of values, it completes and RxJS calls the returned teardown. If a subscriber unsubscribes earlier, the same teardown clears the interval.

## 2. Subscribe from an Angular component

Create **src/app/app.component.ts**:

~~~ts
import { AsyncPipe } from '@angular/common';
import {
  Component,
  DestroyRef,
  inject,
  signal,
} from '@angular/core';
import { Observable, of } from 'rxjs';
import type { Subscription } from 'rxjs';
import { createCountingStream } from './counting-stream';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [AsyncPipe],
  templateUrl: './app.component.html',
})
export class AppComponent {
  readonly greeting$ = of('AsyncPipe subscribed to this greeting Observable.');
  readonly values = signal<number[]>([]);
  readonly streamStatus = signal('Ready');
  readonly coldRuns = signal<number[]>([]);
  readonly synchronousTrace = signal<string[]>([]);

  private readonly destroyRef = inject(DestroyRef);

  private counterSubscription: Subscription | undefined;

  constructor() {
    this.destroyRef.onDestroy(() => {
      this.counterSubscription?.unsubscribe();
    });
  }

  startStream(): void {
    this.counterSubscription?.unsubscribe();
    this.values.set([]);
    this.streamStatus.set('Running');

    this.counterSubscription = createCountingStream(8, 750).subscribe({
      next: (value) => {
        this.values.update((current) => [...current, value]);
      },
      error: (reason: unknown) => {
        this.streamStatus.set('Failed');
        console.error('Counting stream failed:', reason);
      },
      complete: () => {
        this.streamStatus.set('Completed');
      },
    });
  }

  cancelStream(): void {
    if (!this.counterSubscription || this.counterSubscription.closed) {
      return;
    }

    this.counterSubscription.unsubscribe();
    this.streamStatus.set('Cancelled');
  }

  subscribeToColdSourceTwice(): void {
    this.coldRuns.set([]);

    const source$ = new Observable<number>((subscriber) => {
      const runNumber = this.coldRuns().length + 1;
      this.coldRuns.update((runs) => [...runs, runNumber]);
      subscriber.next(runNumber);
      subscriber.complete();
    });

    source$.subscribe({
      next: (value) => console.log('First subscription value:', value),
      complete: () => console.log('First subscription complete'),
    });

    source$.subscribe({
      next: (value) => console.log('Second subscription value:', value),
      complete: () => console.log('Second subscription complete'),
    });
  }

  showSynchronousDelivery(): void {
    this.synchronousTrace.set(['Before subscribe']);

    of('Value from of()').subscribe({
      next: (value) => {
        this.synchronousTrace.update((trace) => [...trace, value]);
      },
      complete: () => {
        this.synchronousTrace.update((trace) => [...trace, 'Complete']);
      },
    });

    this.synchronousTrace.update((trace) => [...trace, 'After subscribe']);
  }
}
~~~

The component saves the long-running **Subscription** so the cancel button can stop it. The **DestroyRef** callback is a final safety net: if the component is destroyed during the stream, it unsubscribes.

The **subscribe()** call uses an Observer object with named **next**, **error**, and **complete** handlers. The custom stream normally completes after eight values, but you can cancel it first.

Create **src/app/app.component.html**:

~~~html
<main>
  <h1>RxJS Observable basics</h1>

  <section>
    <h2>AsyncPipe</h2>
    <p>{{ greeting$ | async }}</p>
  </section>

  <section>
    <h2>Finite stream and cancellation</h2>
    <p>Status: {{ streamStatus() }}</p>
    <p>Values: {{ values().join(', ') || 'No values yet' }}</p>

    <button type="button" (click)="startStream()">Start stream</button>
    <button type="button" (click)="cancelStream()">Cancel stream</button>
  </section>

  <section>
    <h2>Cold Observable</h2>
    <p>
      Click the button to subscribe twice to one Observable instance. Each
      subscription starts the producer again.
    </p>
    <button type="button" (click)="subscribeToColdSourceTwice()">
      Subscribe twice
    </button>
    <p>Producer executions in the latest run: {{ coldRuns().join(', ') }}</p>
  </section>

  <section>
    <h2>Synchronous delivery</h2>
    <button type="button" (click)="showSynchronousDelivery()">
      Show notification order
    </button>
    <ol>
      @for (step of synchronousTrace(); track $index) {
        <li>{{ step }}</li>
      }
    </ol>
  </section>
</main>
~~~

## 3. Bootstrap the standalone app

Create **src/main.ts**:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

## 4. Try the app

1. Click **Start stream**. Values arrive every 750 milliseconds. The stream completes after eight values.
2. Click **Cancel stream** while it is running. The status changes to **Cancelled**, and the console reports that the producer teardown ran.
3. Click **Subscribe twice**. The same cold Observable is subscribed twice, so its producer executes separately for each subscription.
4. Click **Show notification order**. The **of()** value and completion appear before **After subscribe**, demonstrating synchronous delivery.
5. Remove the root component or close the app while the stream is running. Its **DestroyRef** callback unsubscribes the active execution.

The greeting above is consumed with **AsyncPipe**. Angular owns that template subscription and releases it when the component is destroyed.

## 5. Compare common creation functions

These short snippets demonstrate common sources. Try them in a component method or service where you can observe their values:

~~~ts
import {
  from,
  fromEvent,
  interval,
  of,
  timer,
} from 'rxjs';

of('first', 'second').subscribe((value) => {
  console.log('of emits each argument:', value);
});

from(['first', 'second']).subscribe((value) => {
  console.log('from emits each array item:', value);
});

from(Promise.resolve('one resolved value')).subscribe({
  next: (value) => console.log(value),
  error: (reason: unknown) => console.error(reason),
});

timer(500).subscribe((value) => {
  console.log('One delayed value:', value);
});

const intervalSubscription = interval(1000).subscribe((value) => {
  console.log('Repeating value:', value);
});

// Stop an ongoing source when its owner no longer needs it.
intervalSubscription.unsubscribe();

// Use a real element reference in an Angular component.
const clickSubscription = fromEvent<MouseEvent>(
  buttonElement,
  'click',
).subscribe((event) => {
  console.log('Pointer position:', event.clientX, event.clientY);
});

clickSubscription.unsubscribe();
~~~

**of(array)** would emit the whole array as one value; **from(array)** emits its items separately. **interval()** continues until unsubscribed. A **fromEvent()** subscription attaches an event listener and removes it when unsubscribed.

For Angular components, use **takeUntilDestroyed()**, **AsyncPipe**, or **toSignal()** when they fit the ownership and display needs. The [resource-lifetime lesson](../lessons/056-injection-contexts-and-resource-lifetimes.md) explains those cleanup choices; the upcoming cleanup chapter covers them again alongside error handling.

## 6. Observe an error notification

A source reports a failure through **subscriber.error()**. The Observer's **error** callback handles it, and the execution then ends:

~~~ts
import { Observable } from 'rxjs';

const failing$ = new Observable<string>((subscriber) => {
  subscriber.error(new Error('The demo source failed'));
});

failing$.subscribe({
  next: (value) => console.log(value),
  error: (reason: unknown) => console.error('Handled error:', reason),
  complete: () => console.log('This does not run after an error'),
});
~~~

Keep error handling appropriate to the stream. This example shows the channel only; retries, recovery, and UI error states belong in the error-handling lesson.

## 7. Use an Observer with a finite source

For a small one-shot sequence, **of()** is easier to read than constructing an Observable manually:

~~~ts
import { of } from 'rxjs';

const statuses$ = of('queued', 'processing', 'complete');

statuses$.subscribe({
  next: (status) => console.log('Status:', status),
  complete: () => console.log('Status sequence finished'),
});
~~~

Create a custom Observable only when an existing creation function does not represent the producer you need. Custom producers must provide teardown for any work they start.

## 8. What to notice

| Action | Observable behavior |
| --- | --- |
| Create **greeting$** with **of()** | The source is described; the template subscribes when **AsyncPipe** uses it. |
| Click **Start stream** | **subscribe()** starts a new interval-backed producer. |
| Let it emit eight values | The producer sends **complete** and the teardown clears its timer. |
| Click **Cancel stream** early | **unsubscribe()** stops delivery and runs teardown; the **complete** callback is not called. |
| Subscribe twice to **source$** | A cold source creates two producer executions. |
| Read the synchronous trace | **of()** emits and completes during the **subscribe()** call. |

## Further reading

- [RxJS glossary and semantics](https://rxjs.dev/guide/glossary-and-semantics)
- [RxJS Subscription guide](https://rxjs.dev/guide/subscription)
- [Observer API](https://rxjs.dev/api/index/interface/Observer)
- [Subscription API](https://rxjs.dev/api/index/class/Subscription)
- [Angular AsyncPipe API](https://angular.dev/api/common/AsyncPipe)
- [Angular toSignal() API](https://angular.dev/api/core/rxjs-interop/toSignal)
- [Angular HttpClient Observables](https://angular.dev/guide/http/making-requests)

## Related files

- [Lesson: RxJS Observables, subscriptions, and stream basics](../lessons/057-rxjs-observables-subscriptions-and-stream-basics.md)
- [Q&A: RxJS Observables, subscriptions, and stream basics](../qnas/057-rxjs-observables-subscriptions-and-stream-basics.md)
- [Next lesson: RxJS operators for mapping and combining streams](../lessons/058-rxjs-operators-for-mapping-and-combining-streams.md)
