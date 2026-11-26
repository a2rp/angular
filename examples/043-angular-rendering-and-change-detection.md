# 43. Example: follow an Angular rendering update

This standalone Angular page demonstrates the path from state to DOM. A template-read Signal changes after a button event, and an external timer updates plain component state then calls `markForCheck()` so Angular schedules a future check. The example makes the notification and rendering steps visible without forcing detection manually.

## What you will build

- A counter Signal read by the template.
- A computed value derived from that Signal.
- A bound event that updates state and schedules an Angular view refresh.
- A timer callback that uses `ChangeDetectorRef.markForCheck()` for a plain-field update.
- A simple experiment that shows why zoneless applications need Angular notifications.

## Requirements and setup

Create a standalone Angular app if needed:

~~~sh
ng new rendering-cycle-demo --standalone --style=css
cd rendering-cycle-demo
ng serve
~~~

Use the generated root component. The files below use conventional `app.component.*` names; adapt them if your CLI generated shorter names such as `app.*`.

## File layout

~~~text
src/app/
  app.component.ts
  app.component.html
  app.component.css
~~~

## 1. Create a component with observable update paths

Replace or adapt `src/app/app.component.ts`:

~~~ts
import {
  ChangeDetectionStrategy,
  ChangeDetectorRef,
  Component,
  computed,
  inject,
  signal,
} from '@angular/core';

@Component({
  selector: 'app-root',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  private readonly changeDetector = inject(ChangeDetectorRef);

  readonly count = signal(0);
  readonly doubledCount = computed(() => this.count() * 2);

  externalStatus = 'Waiting for an external update';
  private externalUpdateNumber = 0;

  increment(): void {
    this.count.update((value) => value + 1);
  }

  simulateExternalUpdate(): void {
    window.setTimeout(() => {
      this.externalUpdateNumber += 1;
      this.externalStatus = `Updated by timer #${this.externalUpdateNumber}`;

      // Marks the view for an upcoming scheduled check; it does not render here.
      this.changeDetector.markForCheck();
    }, 500);
  }
}
~~~

The counter is a Signal read in the template. Updating it in a bound click handler gives Angular a framework-aware notification, and the template dependency lets Angular refresh this component. The computed value derives from the same source Signal without a separate effect or copied state.

The timer updates a plain field from outside a template listener. Calling `markForCheck()` tells Angular that the view needs a future check. In a zoneless application, that Angular notification is important; ZoneJS scheduling in older/Zone-based projects can behave differently.

## 2. Add the template

Create or replace `src/app/app.component.html`:

~~~html
<main class="page">
  <header class="page-header">
    <p class="eyebrow">Angular example 43</p>
    <h1>From state update to DOM</h1>
    <p>
      Change state, notify Angular, and let its scheduled view check update the
      rendered values.
    </p>
  </header>

  <section class="card" aria-labelledby="counter-heading">
    <h2 id="counter-heading">Signal state and template rendering</h2>
    <p>Count: <strong>{{ count() }}</strong></p>
    <p>Doubled count: <strong>{{ doubledCount() }}</strong></p>
    <button type="button" (click)="increment()">Increment</button>
    <p class="hint">
      The click updates a template-read signal. Angular schedules a view check,
      reevaluates the bindings, and updates their DOM text.
    </p>
  </section>

  <section class="card" aria-labelledby="external-heading">
    <h2 id="external-heading">Plain state from an external callback</h2>
    <p role="status" aria-live="polite">{{ externalStatus }}</p>
    <button type="button" (click)="simulateExternalUpdate()">
      Update from a timer
    </button>
    <p class="hint">
      The timer changes a plain field, then calls markForCheck() so Angular
      schedules a future view check. The page does not call detectChanges().
    </p>
  </section>

  <section class="card" aria-labelledby="cycle-heading">
    <h2 id="cycle-heading">Update cycle</h2>
    <ol>
      <li>A click handler changes the count Signal.</li>
      <li>The Signal and event provide Angular-aware notifications.</li>
      <li>Angular schedules synchronization and checks eligible views.</li>
      <li>Changed bindings are applied to the DOM.</li>
    </ol>
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
  width: min(100% - 2rem, 44rem);
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

button {
  padding: 0.6rem 0.8rem;
  border: 0;
  border-radius: 0.4rem;
  color: #fff;
  background: #244b9a;
  font: inherit;
  cursor: pointer;
}

button:focus-visible {
  outline: 3px solid #f2a900;
  outline-offset: 2px;
}

.hint {
  color: #526078;
  line-height: 1.55;
}
~~~

## 4. Run and explore

1. Start the app with `ng serve` and open the local page.
2. Click **Increment**. The displayed count and doubled value should update after Angular processes the event and signal notification.
3. Click **Update from a timer**. After about half a second, the status should update. The timer callback changes a plain field and calls `markForCheck()`.
4. Temporarily remove the `markForCheck()` call and try the timer again in a zoneless Angular app. The field changes in memory, but that callback no longer notifies Angular to refresh this view. Restore the call afterward.
5. Notice that the app never calls `detectChanges()` for these normal updates. Angular handles the scheduled synchronization.

## 5. Optional: immediate local detection

`detectChanges()` checks the current view and its children immediately. It is intended for deliberate local rendering strategies, often when a view has been detached, rather than as the standard response to every state change:

~~~ts
this.changeDetector.detectChanges();
~~~

Prefer normal Angular notifications such as template-read signals and `markForCheck()` where they fit. A manual call can conceal a missing notification or create confusing update order.

## Concepts to notice

- State changes, view checks, and DOM writes are separate steps.
- The template reads `count()` and `doubledCount()`, so Angular can connect those values to the component view.
- A computed signal derives data without copying state through an effect.
- A bound listener is a notification; the external timer uses `markForCheck()` to notify Angular.
- `markForCheck()` schedules a future check; `detectChanges()` performs a local check immediately.
- The component declares `OnPush` explicitly so the example works consistently across Angular versions. Angular v22 uses it as the default strategy.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="042-rxjs-interop-with-tosignal-and-toobservable.md">042 - 42. RxJS interop with `toSignal()` and `toObservable()`</a></td><td><a href="../lessons/043-angular-rendering-and-change-detection.md">Lesson</a></td><td><a href="../qnas/043-angular-rendering-and-change-detection.md">Q&amp;A</a></td><td><a href="044-onpush-change-detection-and-immutable-updates.md">044 - 44. OnPush change detection and immutable updates</a></td>
</tr>
  </tbody>
</table>
