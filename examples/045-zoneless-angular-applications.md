# 45. Example: zoneless Angular notifications

This standalone Angular application compares three update paths in a zoneless app:

1. A signal read in a template updates from a timer.
2. A plain field changes in a timer without notifying Angular, so the displayed value stays stale.
3. The same kind of plain-field timer calls markForCheck(), so Angular schedules the view update.

The two timer components are separate OnPush subtrees. Updating the signal counter therefore does not accidentally refresh the plain-field component and hide the missing notification.

## Requirements and setup

Use Angular v20.2 or later. Zoneless is the default in Angular v21+, and v20.2 can opt in using the stable provider.

Create and run a standalone app:

~~~sh
ng new zoneless-notifications-demo --standalone --style=css
cd zoneless-notifications-demo
ng serve
~~~

Keep the CLI-generated bootstrap files and replace or add the files below. The example explicitly configures the zoneless provider so its intent is visible.

## 1. Configure zoneless change detection

Update `src/app/app.config.ts`:

~~~ts
import {
  ApplicationConfig,
  provideZonelessChangeDetection,
} from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [provideZonelessChangeDetection()],
};
~~~

The generated `src/main.ts` should continue to bootstrap using the application config:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch((error) => console.error(error));
~~~

In Angular v21+, zoneless is already the default, so the provider is optional. Ensure that provideZoneChangeDetection() is not configured if the goal is zoneless scheduling.

## 2. Create the signal-driven counter

Create `src/app/signal-timer-counter.component.ts`:

~~~ts
import {
  ChangeDetectionStrategy,
  Component,
  DestroyRef,
  inject,
  signal,
} from '@angular/core';

@Component({
  selector: 'app-signal-timer-counter',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  templateUrl: './signal-timer-counter.component.html',
  styleUrl: './signal-timer-counter.component.css',
})
export class SignalTimerCounterComponent {
  readonly count = signal(0);
  private timerId: ReturnType<typeof setInterval> | undefined;

  constructor() {
    inject(DestroyRef).onDestroy(() => this.stop());
  }

  start(): void {
    this.stop();
    this.count.set(0);
    this.timerId = setInterval(() => {
      this.count.update((value) => value + 1);
    }, 1000);
  }

  stop(): void {
    if (this.timerId === undefined) {
      return;
    }

    clearInterval(this.timerId);
    this.timerId = undefined;
  }
}
~~~

Create `src/app/signal-timer-counter.component.html`:

~~~html
<section class="card">
  <h2>Signal state</h2>
  <p>Displayed count: <strong>{{ count() }}</strong></p>
  <p>The template reads a signal, so each timer update notifies Angular.</p>

  <div class="controls">
    <button type="button" (click)="start()">Start signal timer</button>
    <button type="button" (click)="stop()">Stop</button>
  </div>
</section>
~~~

Create `src/app/signal-timer-counter.component.css`:

~~~css
:host {
  display: block;
}

.card {
  padding: 1rem;
  border: 1px solid #86efac;
  border-radius: 0.75rem;
  background: #f0fdf4;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

button {
  padding: 0.6rem 0.8rem;
  border: 1px solid #a1a1aa;
  border-radius: 0.4rem;
  background: white;
  color: inherit;
  cursor: pointer;
}

button:focus-visible {
  outline: 3px solid #2563eb;
  outline-offset: 2px;
}
~~~

The signal is read by this component's template. Updating it from setInterval() is an Angular notification even though the timer callback is not an Angular template event.

## 3. Create the plain-field comparison

Create `src/app/plain-field-timer.component.ts`:

~~~ts
import {
  ChangeDetectionStrategy,
  ChangeDetectorRef,
  Component,
  DestroyRef,
  inject,
} from '@angular/core';

@Component({
  selector: 'app-plain-field-timer',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  templateUrl: './plain-field-timer.component.html',
  styleUrl: './plain-field-timer.component.css',
})
export class PlainFieldTimerComponent {
  count = 0;
  private timerId: ReturnType<typeof setInterval> | undefined;
  private readonly changeDetector = inject(ChangeDetectorRef);

  constructor() {
    inject(DestroyRef).onDestroy(() => this.stop());
  }

  startWithoutNotification(): void {
    this.startTimer(false);
  }

  startWithMarkForCheck(): void {
    this.startTimer(true);
  }

  stop(): void {
    if (this.timerId === undefined) {
      return;
    }

    clearInterval(this.timerId);
    this.timerId = undefined;
  }

  private startTimer(notifyAngular: boolean): void {
    this.stop();
    this.count = 0;

    // The button event checks this view and displays the reset value.
    this.timerId = setInterval(() => {
      this.count += 1;

      if (notifyAngular) {
        this.changeDetector.markForCheck();
      }
    }, 1000);
  }
}
~~~

Create `src/app/plain-field-timer.component.html`:

~~~html
<section class="card">
  <h2>Plain field</h2>
  <p>Displayed count: <strong>{{ count }}</strong></p>

  <div class="controls">
    <button type="button" (click)="startWithoutNotification()">
      Start without notification
    </button>
    <button type="button" (click)="startWithMarkForCheck()">
      Start with markForCheck()
    </button>
    <button type="button" (click)="stop()">Stop</button>
  </div>

  <p>
    The first timer changes the TypeScript field but does not notify Angular.
    The second timer marks this view after each change.
  </p>
</section>
~~~

Create `src/app/plain-field-timer.component.css`:

~~~css
:host {
  display: block;
}

.card {
  padding: 1rem;
  border: 1px solid #fbbf24;
  border-radius: 0.75rem;
  background: #fffbeb;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

button {
  padding: 0.6rem 0.8rem;
  border: 1px solid #a1a1aa;
  border-radius: 0.4rem;
  background: white;
  color: inherit;
  cursor: pointer;
}

button:focus-visible {
  outline: 3px solid #2563eb;
  outline-offset: 2px;
}
~~~

Both timer methods update the same plain field. The difference is the notification: startWithMarkForCheck() marks the view after every timer tick; startWithoutNotification() does not.

## 4. Add both components to the app

Update `src/app/app.component.ts`:

~~~ts
import {
  ChangeDetectionStrategy,
  Component,
} from '@angular/core';
import { PlainFieldTimerComponent } from './plain-field-timer.component';
import { SignalTimerCounterComponent } from './signal-timer-counter.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    PlainFieldTimerComponent,
    SignalTimerCounterComponent,
  ],
  changeDetection: ChangeDetectionStrategy.OnPush,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {}
~~~

Update `src/app/app.component.html`:

~~~html
<main class="page">
  <h1>Zoneless update notifications</h1>

  <p>
    Start each counter and compare what happens after its timer callbacks run.
  </p>

  <div class="examples">
    <app-signal-timer-counter />
    <app-plain-field-timer />
  </div>
</main>
~~~

Update `src/app/app.component.css`:

~~~css
:host {
  display: block;
  color: #18181b;
  font-family: system-ui, sans-serif;
}

.page {
  width: min(100% - 2rem, 52rem);
  margin: 3rem auto;
}

.examples {
  display: grid;
  gap: 1rem;
  margin-top: 1.5rem;
}


@media (min-width: 44rem) {
  .examples {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}
~~~

## 5. Try the notification paths

### Signal timer

Click **Start signal timer**. The count increases every second. The timer callback changes a signal read by the child template, so Angular schedules that OnPush child view.

### Plain field without a notification

Click **Start without notification**. The displayed count resets to zero from the Angular button event. Later timer callbacks increase the TypeScript field, but the displayed value remains stale because the callback does not notify Angular.

### Plain field with markForCheck()

Click **Start with markForCheck()**. The value updates each second because every timer callback marks the component view for a future check.

The buttons themselves are Angular listeners, so clicking them also checks the relevant component. That is why the reset value appears immediately even in the no-notification version. After the click handler finishes, the timer callback has no Angular listener to trigger a check.

## Expected behavior

| Update path | State update | Zoneless notification | Display |
| --- | --- | --- | --- |
| Signal timer | Changes a template-read signal | Signal update | Advances each second |
| Plain timer without mark | Changes a plain field | None from the timer | Stays at its last rendered value |
| Plain timer with mark | Changes a plain field | markForCheck() | Advances each second |
| Button click | Starts or stops a timer | Bound template listener | The view is checked for the event |

## Optional: remove ZoneJS from this demo

The explicit provider configures Angular's scheduler; ZoneJS may still be loaded by the generated project. To remove ZoneJS entirely, first confirm that no dependency needs its patches. Then inspect the generated configuration:

1. Remove zone.js and zone.js/testing from build and test polyfill entries in angular.json, if present.
2. Remove explicit ZoneJS imports from a polyfills file, if the project has one.
3. Remove the zone.js package with the project's package manager only if no integration requires it.
4. Run the project's normal build and tests, then check third-party event and browser API integrations.

The exact CLI configuration varies by Angular version. Do not copy a polyfills edit from another project without checking this app's generated angular.json.

## Variations

### Use a signal for an external callback

If a WebSocket or browser observer updates the page, update a signal read by the template:

~~~ts
this.connectionState.set('Connected');
~~~

If the external integration must update a plain field, call markForCheck() after the assignment instead.

### Use AsyncPipe with an Observable

When the source is already an Observable, render it with AsyncPipe. New emissions mark the view for checking, and the pipe manages its subscription with the view lifecycle.

~~~html
<p>{{ status$ | async }}</p>
~~~

### Use an Angular event

A bound template listener is already a zoneless notification. State changes made inside a click or keyboard handler can be reflected without calling markForCheck() solely because the handler ran through Angular.

## Common mistakes

- Expecting setInterval() or a Promise continuation to refresh a plain field automatically.
- Calling detectChanges() in every asynchronous callback instead of using a signal or markForCheck().
- Adding provideZonelessChangeDetection() to an Angular v21+ app without understanding that zoneless is already the default.
- Removing ZoneJS without checking libraries that depend on its patched APIs.
- Combining a signal-driven child with a plain-field example in the same view and accidentally letting the signal refresh both.
- Forgetting to clear intervals when their components are destroyed.
- Treating NgZone.isStable as a completion indicator in a zoneless app.

## Key takeaways

- Zoneless Angular still renders and checks views; Angular-aware notifications schedule that work.
- A signal read by a template can notify Angular when updated from a timer or external callback.
- A plain field changed outside an Angular event needs an explicit notification such as markForCheck().
- AsyncPipe, ComponentRef.setInput(), and bound listeners are other common notification paths.
- Keep unrelated update examples in separate component views when demonstrating which view each notification refreshes.
- Remove ZoneJS from the build only after checking the project's configuration and dependencies.

## Official references

- [Angular without ZoneJS (Zoneless)](https://angular.dev/guide/zoneless)
- [provideZonelessChangeDetection API](https://angular.dev/api/core/provideZonelessChangeDetection)
- [Angular Signals](https://angular.dev/guide/signals)
- [ChangeDetectorRef API](https://angular.dev/api/core/ChangeDetectorRef)
