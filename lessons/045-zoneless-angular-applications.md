# 45. Zoneless Angular applications

Angular's zoneless mode schedules view updates from Angular-aware notifications instead of relying on ZoneJS to notice browser events and asynchronous tasks. Zoneless does not remove change detection. It makes the signals that schedule updates explicit and more closely tied to application state.

Zoneless change detection is the default in Angular v21 and later. Earlier Angular v20 projects can opt in with the stable provideZonelessChangeDetection() provider available since v20.2. When upgrading or maintaining an application, verify that a Zone-based provider is not overriding the zoneless default.

## Learning goals

After this lesson, you should be able to:

- Explain what ZoneJS did and what zoneless mode changes.
- Identify the Angular notifications that schedule a view update in a zoneless app.
- Update template state safely from timers, promises, and external callbacks.
- Decide when to use signals, AsyncPipe, markForCheck(), or an Angular event.
- Enable zoneless change detection on Angular versions that require an explicit provider.
- Remove ZoneJS from a build carefully and check integrations first.
- Handle reactive forms, SSR stability, and tests in a zoneless-compatible way.
- Recognize obsolete NgZone stability patterns and choose appropriate replacements.

## What zoneless means

ZoneJS patches many browser APIs and observes asynchronous activity such as timers, network work, and event listeners. In a Zone-based application, Angular has historically used those signals as evidence that application state might have changed and that it should run synchronization.

That approach can schedule checks even when the application data did not change. ZoneJS also adds a dependency and can make debugging and third-party API integration more complex.

In a zoneless application, Angular does not use ZoneJS task activity as the change-detection scheduler. Instead, it uses explicit Angular notifications such as a template-read signal changing, a bound event running, or a view being marked for checking. Angular still checks templates and updates the DOM; the scheduler has better information about which views need work.

Zoneless therefore does not mean:

- Angular has no change detection.
- Timers, promises, HTTP requests, or browser events stop working.
- Every asynchronous callback automatically refreshes the DOM.
- Applications must manually call detectChanges() after every state change.
- ZoneJS must be absent from the page in every situation. It can remain for another application or library, though a fully zoneless build can remove it to reduce overhead when safe.

## Angular version behavior

| Angular version | Zoneless behavior |
| --- | --- |
| v21 and later | Zoneless is the default. Check that provideZoneChangeDetection() is not used to opt back into Zone-based scheduling. |
| v20.2 | Zoneless can be enabled with provideZonelessChangeDetection(); the API is stable starting in v20.2. |
| Earlier releases | Follow the version-specific Angular migration guide. Do not copy a current provider into an older project without checking API availability. |

Angular's default can still be overridden by application providers. Inspect the project configuration rather than inferring the runtime mode only from the package version.

## Notifications Angular uses

Angular uses core API notifications to decide when synchronization is needed and which views are affected. The main ones are:

| Notification | Typical use |
| --- | --- |
| A template or host event listener runs | A button click or a bound keyboard event |
| A signal read in a template changes | Component or shared state exposed as a signal |
| A view is marked with ChangeDetectorRef.markForCheck() | Plain-field update from an external callback |
| AsyncPipe receives a new value | Observable or Promise state rendered in a template |
| ComponentRef.setInput() sets a component input | Programmatically controlled components |
| A view marked dirty is attached | Dynamic views and container operations |

Use an Angular notification at the point where state changes. A plain property assignment inside an arbitrary timer, promise continuation, WebSocket callback, browser observer, or third-party callback does not by itself tell a zoneless application that a view should refresh.

## Prefer state the template can observe

### Signals for component state

A signal read in a template is a direct notification path:

~~~ts
import {
  ChangeDetectionStrategy,
  Component,
  signal,
} from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  templateUrl: './counter.component.html',
})
export class CounterComponent {
  readonly count = signal(0);

  increment(): void {
    this.count.update((value) => value + 1);
  }
}
~~~

The corresponding template can read the signal:

~~~html
<p>Count: {{ count() }}</p>
<button type="button" (click)="increment()">Add one</button>
~~~

When increment() updates the signal, Angular knows which view depends on it and schedules that view to refresh. A component event is also a notification, but signals continue to make state changes observable when updates come from an asynchronous callback that is not itself an Angular template event.

Signal values should still be updated with appropriate identity semantics. For object and array state, return a replacement value rather than mutating a value and setting the same reference.

### AsyncPipe for Observable state

AsyncPipe subscribes to an Observable or Promise, returns its latest value, and marks the component for checking when a new value arrives. It also handles subscription cleanup with the view lifecycle.

~~~html
<p>{{ status$ | async }}</p>
~~~

This is a convenient notification bridge when the source of truth is already an Observable. Avoid manually subscribing only to copy emissions into a plain field unless an integration requires that shape.

### markForCheck() for plain-field integrations

When an external API or legacy service must update a plain field, call markForCheck() after changing the field:

~~~ts
import {
  ChangeDetectionStrategy,
  ChangeDetectorRef,
  Component,
  inject,
} from '@angular/core';

@Component({
  selector: 'app-connection-status',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: '<p>{{ status }}</p>',
})
export class ConnectionStatusComponent {
  private readonly changeDetector = inject(ChangeDetectorRef);
  status = 'Waiting';

  onExternalStatus(status: string): void {
    this.status = status;
    this.changeDetector.markForCheck();
  }
}
~~~

markForCheck() notifies Angular that the view should be checked in a future synchronization. It does not synchronously render the DOM. For new application state, a signal read by the template is usually simpler.

### ComponentRef.setInput() for dynamic components

When setting an input on a programmatically created component, use ComponentRef.setInput(). Angular recognizes it as an input update and can schedule the correct view work.

~~~ts
componentRef.setInput('record', nextRecord);
~~~

Directly assigning an @Input field through a component reference does not provide the same notification contract.

## Enable zoneless change detection

### Angular v21 and later

Zoneless is the default. Check that the application does not provide provideZoneChangeDetection(), which would explicitly request Zone-based scheduling. An Angular v21+ application can keep its usual bootstrap setup without adding another provider.

### Opt in on Angular v20.2+

For a standalone application using bootstrapApplication, provide provideZonelessChangeDetection() in the application providers:

~~~ts
import {
  ApplicationConfig,
  provideZonelessChangeDetection,
} from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [provideZonelessChangeDetection()],
};
~~~

The CLI-generated main.ts should continue bootstrapping the app with its appConfig:

~~~ts
bootstrapApplication(AppComponent, appConfig)
  .catch((error) => console.error(error));
~~~

For an NgModule-based application, add the provider to the module providers as supported by that Angular version:

~~~ts
@NgModule({
  providers: [provideZonelessChangeDetection()],
})
export class AppModule {}
~~~

The provider changes Angular's scheduling behavior. It does not by itself remove the ZoneJS package from the build.

## Removing ZoneJS from the build

A zoneless scheduler can operate even if ZoneJS remains on the page for another dependency. To remove ZoneJS entirely and reduce the build payload, first confirm that the application and its dependencies do not rely on ZoneJS patches.

Then inspect the actual project configuration:

1. Remove zone.js and zone.js/testing from the polyfills lists for the build and test targets in angular.json, if present.
2. If the project imports ZoneJS from a polyfills.ts file, remove those imports.
3. Remove the zone.js dependency from package.json with the package manager.
4. Run the project's normal build and tests and verify third-party integrations, event listeners, and browser APIs.

Do not remove ZoneJS blindly from a mixed application or a dependency that still needs its patches. A project can use zoneless Angular scheduling while ZoneJS remains available for unrelated code.

## Common async integrations

### Timers and promise callbacks

A timer or promise continuation is ordinary JavaScript asynchronous work. In zoneless mode, updating a plain field inside it is not a notification:

~~~ts
setTimeout(() => {
  this.status = 'Complete';
  // A plain assignment alone does not schedule a zoneless view refresh.
}, 1000);
~~~

Use a template-read signal:

~~~ts
readonly status = signal('Waiting');

completeLater(): void {
  setTimeout(() => {
    this.status.set('Complete');
  }, 1000);
}
~~~

Or, for a plain field that must remain, assign it and call markForCheck():

~~~ts
setTimeout(() => {
  this.status = 'Complete';
  this.changeDetector.markForCheck();
}, 1000);
~~~

Choose one clear source of truth. Do not maintain a signal and a duplicated plain field for the same rendered state without a specific reason.

### Third-party callbacks and browser APIs

A callback from a third-party widget, WebSocket, observer, or native browser API can update a signal consumed by the template or call markForCheck() after a plain-field update. Use the notification at the integration boundary.

NgZone.run() and NgZone.runOutsideAngular() remain compatible with zoneless applications. They are not required just to make zoneless change detection work. Keeping them may still help a shared library operate well in applications that use ZoneJS, or help isolate high-frequency tasks in Zone-based applications.

### Reactive forms

Reactive form APIs such as setValue(), patchValue(), and FormArray.push() update form state and emit form observables, but do not automatically schedule zoneless change detection for a template that reads form state.

For a template depending on reactive form state, connect the relevant form observable to markForCheck(), or expose the state through a signal consumed by the template. Choose a notification that follows the actual state changes; do not poll the form with frequent checks.

## APIs tied to Zone stability

In zoneless mode, NgZone.onMicrotaskEmpty, onUnstable, and onStable do not emit. NgZone.isStable is always true, so it should not be used as a signal that application work has completed.

If code waits for a render:

- Use afterNextRender() when it needs to run after one render.
- Use afterEveryRender() only when it needs to run after each render.
- Use a direct browser observer such as MutationObserver when the real requirement is to observe a DOM mutation.

These APIs answer different questions. A render hook waits for Angular's render phase; a browser observer watches a particular DOM condition.

## Zoneless and server-side rendering

Server-side rendering must know whether asynchronous work that affects the rendered HTML is still pending. Zone-based applications have historically used ZoneJS stability to help answer this. In a zoneless SSR application, register relevant work with Angular's PendingTasks service so serialization waits for it to finish.

~~~ts
import { Injectable, PendingTasks, inject, signal } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class PageState {
  private readonly pendingTasks = inject(PendingTasks);
  readonly result = signal<string | null>(null);

  loadForServerRender(): void {
    this.pendingTasks.run(async () => {
      this.result.set(await loadDataNeededForRendering());
    });
  }
}
~~~

Use PendingTasks for work that must complete before the server serializes the page. Do not register background work that is irrelevant to the rendered response as a blocking task.

## Testing zoneless-compatible code

Angular's TestBed uses zoneless change detection by default in current Angular documentation. This helps expose components that update plain fields without an Angular notification.

Tests should exercise the same notifications that production uses, such as clicking a bound control, updating a template-read signal, setting a component input through Angular, or emitting through AsyncPipe. Prefer waiting for Angular's scheduled work to stabilize when appropriate.

Calling fixture.detectChanges() manually forces a check. It can hide missing notifications, so avoid using it as the only way to make a test pass when the production UI is expected to update on its own. Existing suites may use it extensively; migrate deliberately rather than changing every test mechanically.

## Migration checklist

- Confirm the Angular version and whether a provider explicitly enables Zone-based scheduling.
- Find plain component fields changed by timers, promises, external listeners, or library callbacks.
- Replace template-facing state with signals where practical, or call markForCheck() at imperative update boundaries.
- Use AsyncPipe for Observable state rendered by templates.
- Use ComponentRef.setInput() when setting dynamic component inputs.
- Review reactive forms, third-party widgets, and custom event integrations.
- Remove ZoneJS from the build only after dependencies and configuration have been checked.
- For zoneless SSR, register work that must finish before serialization with PendingTasks.
- Let tests exercise Angular notifications instead of relying exclusively on forced detection.

## Common mistakes

- **Assuming zoneless means no change detection:** Angular still checks views and updates the DOM.
- **Assuming any async callback refreshes plain fields:** an arbitrary timer or promise continuation is not a zoneless notification.
- **Calling detectChanges() after every update:** use a signal, AsyncPipe, a bound event, or markForCheck() at the correct boundary.
- **Removing ZoneJS before checking libraries:** a dependency may rely on patched APIs even if Angular itself is zoneless.
- **Keeping provideZoneChangeDetection() in a v21+ app unintentionally:** it overrides the zoneless default.
- **Using NgZone stability observables to wait for rendering:** they do not emit in zoneless mode.
- **Assuming reactive-form model updates schedule the view:** form state updates need to be connected to an Angular notification.
- **Forgetting PendingTasks for SSR-required async work:** the server may serialize before that work finishes.
- **Relying on fixture.detectChanges() to prove notifications work:** forced checks can hide a missing production notification.

## Key takeaways

- Zoneless is the default in Angular v21 and later; Angular v20.2 can opt in with provideZonelessChangeDetection().
- Angular still performs change detection, but schedules it from framework-aware notifications instead of ZoneJS async-task tracking.
- Template-read signals, bound listeners, AsyncPipe, markForCheck(), and ComponentRef.setInput() are common notification paths.
- Plain fields changed by arbitrary asynchronous callbacks need a signal or an explicit mark.
- ZoneJS can be removed from the bundle after checking configuration and dependency requirements.
- Reactive forms, SSR stability, and tests each have specific zoneless considerations.
- NgZone run methods remain compatible, while NgZone stability observables are not useful in zoneless applications.

## Official references

- [Angular without ZoneJS (Zoneless)](https://angular.dev/guide/zoneless)
- [provideZonelessChangeDetection API](https://angular.dev/api/core/provideZonelessChangeDetection)
- [Angular Signals](https://angular.dev/guide/signals)
- [AsyncPipe API](https://angular.dev/api/common/AsyncPipe)
- [Resolving Zone pollution](https://angular.dev/best-practices/zone-pollution)
- [Server and hybrid rendering](https://angular.dev/guide/ssr)
