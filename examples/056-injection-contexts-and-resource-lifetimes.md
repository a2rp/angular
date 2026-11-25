# Example: Injection contexts and resource lifetimes

This standalone Angular example gives you visible ways to exercise the concepts in the lesson:

- A custom helper injects a token and asserts that it is called in an injection context.
- A button calls that helper later through **runInInjectionContext()**.
- A child component owns a component-scoped service and a **DestroyRef**.
- **takeUntilDestroyed()** and **toSignal()** stop their subscriptions when the child is removed.
- An effect cancels its timer before rerunning or being destroyed.
- A **resource()** loader receives an **AbortSignal** and cancels an outdated read.
- A root service stays app-scoped while the child component can be destroyed and recreated.

The **resource()** API is stable starting in Angular 22. If your project uses an earlier Angular version, check that version's resource support or skip the resource files and use an Observable with **takeUntilDestroyed()**.

Create a standalone Angular app with the Angular CLI, then add or replace the files below. The CLI project includes RxJS for the observable example.

## 1. Create a token

Create **src/app/demo-message.token.ts**:

~~~ts
import { InjectionToken } from '@angular/core';

export const DEMO_MESSAGE =
  new InjectionToken<string>('DEMO_MESSAGE');
~~~

Create **src/app/app.config.ts**:

~~~ts
import type { ApplicationConfig } from '@angular/core';
import { DEMO_MESSAGE } from './demo-message.token';

export const appConfig: ApplicationConfig = {
  providers: [
    {
      provide: DEMO_MESSAGE,
      useValue: 'Resolved from the application injector',
    },
  ],
};
~~~

## 2. Write a helper that requires an injection context

Create **src/app/inject-demo-message.ts**:

~~~ts
import {
  assertInInjectionContext,
  inject,
} from '@angular/core';
import { DEMO_MESSAGE } from './demo-message.token';

export function injectDemoMessage(): string {
  assertInInjectionContext(injectDemoMessage);
  return inject(DEMO_MESSAGE);
}
~~~

The helper checks its contract, then reads the token. It works during a component field initializer. Calling it from a later event handler directly would fail; the component will use **runInInjectionContext()** for that case.

## 3. Create a root service with an Observable

Create **src/app/tick-source.service.ts**:

~~~ts
import { DestroyRef, Injectable, inject } from '@angular/core';
import { interval } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class TickSource {
  private readonly destroyRef = inject(DestroyRef);

  readonly ticks$ = interval(1000);

  constructor() {
    this.destroyRef.onDestroy(() => {
      console.info('The root TickSource scope was destroyed');
    });
  }
}
~~~

**interval()** is cold: it does not start producing values until something subscribes. The root service can expose it without owning a running subscription. Each child component below creates its own subscriptions and ends them with its own lifetime.

The log registered here belongs to the root injector. Hiding the child component does not destroy the root service.

## 4. Create a component-scoped service

Create **src/app/feature-panel-state.service.ts**:

~~~ts
import { DestroyRef, Injectable, inject } from '@angular/core';

let nextPanelId = 0;

@Injectable()
export class FeaturePanelState {
  readonly id = ++nextPanelId;

  constructor() {
    inject(DestroyRef).onDestroy(() => {
      console.info('FeaturePanelState destroyed:', this.id);
    });
  }
}
~~~

This service does not use **providedIn: 'root'**. The child component registers it in its **providers** array, so each child instance gets its own state object and destruction scope.

## 5. Add an abortable demo loader

Create **src/app/demo-user.ts**:

~~~ts
export interface DemoUser {
  readonly id: string;
  readonly name: string;
}

export function loadDemoUser(
  id: string,
  abortSignal: AbortSignal,
): Promise<DemoUser> {
  return new Promise((resolve, reject) => {
    if (abortSignal.aborted) {
      reject(new DOMException('The request was cancelled', 'AbortError'));
      return;
    }

    const timeoutId = setTimeout(() => {
      abortSignal.removeEventListener('abort', cancel);
      resolve({
        id,
        name: 'Demo user ' + id,
      });
    }, 1200);

    const cancel = (): void => {
      clearTimeout(timeoutId);
      reject(new DOMException('The request was cancelled', 'AbortError'));
    };

    abortSignal.addEventListener('abort', cancel, { once: true });
  });
}
~~~

This fake loader waits briefly so you can see loading and cancellation. It listens to the resource's signal and clears its timer when Angular aborts the load. A real read API should pass the same signal into a cancellable operation, such as **fetch()**.

## 6. Build the child component

Create **src/app/lifetime-panel.component.ts**:

~~~ts
import {
  Component,
  DestroyRef,
  Injector,
  effect,
  inject,
  resource,
  runInInjectionContext,
  signal,
} from '@angular/core';
import {
  takeUntilDestroyed,
  toSignal,
} from '@angular/core/rxjs-interop';
import { FeaturePanelState } from './feature-panel-state.service';
import { injectDemoMessage } from './inject-demo-message';
import { TickSource } from './tick-source.service';
import { loadDemoUser } from './demo-user';

@Component({
  selector: 'app-lifetime-panel',
  standalone: true,
  templateUrl: './lifetime-panel.component.html',
  providers: [FeaturePanelState],
})
export class LifetimePanelComponent {
  private readonly injector = inject(Injector);
  private readonly destroyRef = inject(DestroyRef);
  private readonly tickSource = inject(TickSource);
  private readonly panelState = inject(FeaturePanelState);

  readonly message = injectDemoMessage();
  readonly panelId = this.panelState.id;
  readonly tickSignal = toSignal(this.tickSource.ticks$, {
    initialValue: 0,
  });
  readonly manualTicks = signal(0);
  readonly contextResult = signal('Not run yet');
  readonly effectEnabled = signal(true);
  readonly effectStatus = signal('The effect has not run yet');
  readonly userId = signal('user-1');

  readonly user = resource({
    params: () => {
      const id = this.userId().trim();
      return id ? { id } : undefined;
    },
    loader: ({ params, abortSignal }) =>
      loadDemoUser(params.id, abortSignal),
  });

  constructor() {
    this.tickSource.ticks$
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe((tick) => this.manualTicks.set(tick));

    const timerId = setTimeout(() => {
      this.effectStatus.set('The component-owned timeout completed');
    }, 15000);

    this.destroyRef.onDestroy(() => {
      clearTimeout(timerId);
      console.info('LifetimePanelComponent destroyed:', this.panelId);
    });

    effect((onCleanup) => {
      if (!this.effectEnabled()) {
        this.effectStatus.set('Effect paused');
        return;
      }

      this.effectStatus.set('Effect timer is running');

      const effectTimerId = setTimeout(() => {
        this.effectStatus.set('Effect timer completed');
      }, 5000);

      onCleanup(() => clearTimeout(effectTimerId));
    });
  }

  runHelperLater(): void {
    const message = runInInjectionContext(
      this.injector,
      () => injectDemoMessage(),
    );

    this.contextResult.set(message);
  }

  updateUserId(event: Event): void {
    const input = event.target as HTMLInputElement;
    this.userId.set(input.value);
  }
}
~~~

Three different lifetime patterns are shown:

1. The **tickSignal** is created during field initialization. **toSignal()** ties its subscription to this component's injector.
2. The manual subscription passes this component's **DestroyRef** to **takeUntilDestroyed()**.
3. The effect and **resource()** are created in the component's injection context and are cleaned up with that context.

The 15-second timer is separately cleared in **DestroyRef.onDestroy()**. It exists to make direct cleanup registration easy to spot.

Create **src/app/lifetime-panel.component.html**:

~~~html
<section>
  <h2>Panel instance {{ panelId }}</h2>
  <p>{{ message }}</p>

  <h3>Injection context</h3>
  <p>
    The helper captured during construction:
    <strong>{{ message }}</strong>
  </p>
  <button type="button" (click)="runHelperLater()">
    Run the helper later inside an injection context
  </button>
  <p>Callback result: {{ contextResult() }}</p>

  <h3>Observable subscriptions</h3>
  <p>toSignal value: {{ tickSignal() }}</p>
  <p>takeUntilDestroyed value: {{ manualTicks() }}</p>

  <h3>Effect cleanup</h3>
  <label>
    <input
      type="checkbox"
      [checked]="effectEnabled()"
      (change)="effectEnabled.set($any($event.target).checked)"
    />
    Keep the effect timer enabled
  </label>
  <p>{{ effectStatus() }}</p>

  <h3>Abortable resource read</h3>
  <label for="user-id">Demo user ID</label>
  <input
    id="user-id"
    type="text"
    [value]="userId()"
    (input)="updateUserId($event)"
  />

  <p>Resource status: {{ user.status() }}</p>
  @if (user.isLoading()) {
    <p>Loading the current user…</p>
  }
  @if (user.hasValue()) {
    <p>Loaded: {{ user.value().name }} ({{ user.value().id }})</p>
  }
  @if (user.error(); as loadError) {
    <p>Load error: {{ loadError.message }}</p>
  }

  <p>
    Change the ID before the delay finishes to cancel the old read and start
    a new one.
  </p>
</section>
~~~

The checkbox uses **$any()** to keep the template short while reading the native checkbox value. In a larger app, use a typed event handler instead.

## 7. Add a parent that destroys and recreates the panel

Create **src/app/app.component.ts**:

~~~ts
import { Component, signal } from '@angular/core';
import { LifetimePanelComponent } from './lifetime-panel.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [LifetimePanelComponent],
  templateUrl: './app.component.html',
})
export class AppComponent {
  readonly showPanel = signal(true);

  togglePanel(): void {
    this.showPanel.update((visible) => !visible);
  }
}
~~~

Create **src/app/app.component.html**:

~~~html
<main>
  <h1>Angular injection contexts and lifetimes</h1>
  <p>
    Hide the panel to destroy its injector scope, then show it again to create
    a fresh component-scoped service.
  </p>

  <button type="button" (click)="togglePanel()">
    {{ showPanel() ? 'Destroy panel' : 'Create panel' }}
  </button>

  @if (showPanel()) {
    <app-lifetime-panel />
  }
</main>
~~~

Create **src/main.ts**:

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig).catch((error: unknown) => {
  console.error(error);
});
~~~

## 8. Try the example

Run the Angular CLI development server and try these actions:

1. Click **Run the helper later inside an injection context**. The callback result shows the token resolved from the component's injector.
2. Watch both tick values update. They come from two separate subscriptions to the same cold Observable, and each subscription has teardown tied to this component.
3. Uncheck the effect option. The effect cleanup cancels its pending timer before the effect reruns.
4. Change the demo user ID quickly. A new parameter cancels the previous resource load; the latest ID is shown when its delay completes.
5. Click **Destroy panel**. The component and **FeaturePanelState** destruction messages appear in the console. The component-owned subscription, signal subscription, effect, resource read, and timer are cleaned up.
6. Click **Create panel**. A new panel and component-scoped state instance appear. The root **TickSource** remains the same application-level service.

If you wait for the 15-second timer, it updates the panel while it is present. If you destroy the panel first, the registered cleanup clears that timer.

## 9. Compare the two scopes

| Item | Scope in this example | What ends it |
| --- | --- | --- |
| **TickSource** service | Root injector | Application/root injector destruction |
| **FeaturePanelState** service | Child component injector | Hiding and destroying the child panel |
| **tickSignal** subscription | Child component's injection context | Child panel destruction |
| Manual RxJS subscription | Explicit child **DestroyRef** | Child panel destruction |
| Effect timer | Effect execution | Effect rerun or child panel destruction |
| User resource request | Resource owned by the child | Parameter change or child panel destruction |

A service can live longer than a component that consumes it. A subscription started by a component can still be tied to the shorter component lifetime by passing that component's **DestroyRef**.

## Related files

- [Lesson: Injection contexts and resource lifetimes](../lessons/056-injection-contexts-and-resource-lifetimes.md)
- [Q&A: Injection contexts and resource lifetimes](../qnas/056-injection-contexts-and-resource-lifetimes.md)
- [Next lesson: RxJS Observables, subscriptions, and stream basics](../lessons/057-rxjs-observables-subscriptions-and-stream-basics.md)

## Further reading

- [Injection context guide](https://angular.dev/guide/di/dependency-injection-context)
- [runInInjectionContext() API](https://angular.dev/api/core/runInInjectionContext)
- [DestroyRef API](https://angular.dev/api/core/DestroyRef)
- [takeUntilDestroyed() API](https://angular.dev/api/core/rxjs-interop/takeUntilDestroyed)
- [toSignal() API](https://angular.dev/api/core/rxjs-interop/toSignal)
- [Effect cleanup](https://angular.dev/guide/signals/effect)
- [Resource API](https://angular.dev/api/core/resource)
