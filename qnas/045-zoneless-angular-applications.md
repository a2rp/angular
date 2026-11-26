# 45. Zoneless Angular applications - Q&A

## Core concepts and versions

### 1. What does zoneless mean in Angular?

Angular does not use ZoneJS task activity as its change-detection scheduler. It schedules view synchronization from Angular-aware notifications such as template events, changed template-read signals, AsyncPipe emissions, and markForCheck().

### 2. Does zoneless remove Angular change detection?

No. Angular still checks eligible views and updates the DOM. Zoneless changes how Angular learns that work is needed and which views need refreshing.

### 3. Does zoneless mean that timers, promises, HTTP requests, or DOM events stop working?

No. Those APIs continue to work. The difference is that an arbitrary asynchronous callback changing a plain component field is not, by itself, a notification that asks Angular to refresh the view.

### 4. Since which version is zoneless the default?

Angular v21 and later use zoneless change detection by default. Check that the application does not provide provideZoneChangeDetection(), which explicitly requests Zone-based scheduling.

### 5. How do I enable zoneless on Angular v20?

Angular v20 can opt in by adding provideZonelessChangeDetection() to the application providers. The API is stable starting in v20.2. Check the exact framework version before using it.

### 6. Should an Angular v21+ application still add provideZonelessChangeDetection()?

It is normally unnecessary because zoneless is already the default. Verify that provideZoneChangeDetection() is not overriding that default.

### 7. Does providing the zoneless scheduler remove ZoneJS from the bundle?

No. The provider changes Angular's scheduling behavior. ZoneJS can remain loaded, for example because another application or library needs its browser API patches. To remove it from the bundle, update polyfills and dependencies separately after checking integrations.

### 8. Can ZoneJS stay on the page in a zoneless application?

Yes. Angular can use zoneless scheduling while ZoneJS remains for another application or library. A fully zoneless application can remove ZoneJS to reduce payload when nothing depends on its patches.

## Notifications and state updates

### 9. What notifies Angular that a zoneless view needs checking?

Common notifications include a bound host or template listener, a changed signal read in a template, ChangeDetectorRef.markForCheck(), an AsyncPipe emission, ComponentRef.setInput(), and attaching a view that was marked dirty.

### 10. What is the simplest way to update component state from an async callback?

Use a signal that the component template reads, then update that signal in the callback. The signal change notifies Angular even though the callback came from a timer, promise, or external API.

### 11. Can I update a plain field from a timer?

Yes, but that assignment alone does not schedule a zoneless view refresh. Either represent the template-facing value with a signal or call markForCheck() after updating the field.

### 12. Does a promise continuation trigger change detection in zoneless mode?

The continuation runs normally, but updating a plain field inside it is not an Angular notification. Set a template-read signal or call markForCheck() after the update.

### 13. Does an HTTP response automatically refresh a template?

An HTTP request and response still work. The view refresh depends on the notification path used to expose the response: for example, a template-read signal, AsyncPipe, or markForCheck() after a plain-field update.

### 14. Do bound Angular event listeners still trigger updates?

Yes. Angular's template and host event listeners are notification sources. State changes made by an Angular-handled click or keyboard event can be rendered.

### 15. What does AsyncPipe do for zoneless change detection?

It subscribes to an Observable or Promise, returns its latest value, and marks the component for checking when a new value arrives. It also handles subscription cleanup with the view lifecycle.

### 16. Should I call markForCheck() after every signal update?

Usually not. If the component template reads the signal, changing it already provides the notification. Add an explicit mark only when another integration or state shape requires one.

### 17. When should I use markForCheck()?

Use it when an imperative integration changes a plain field and Angular otherwise has no notification. It marks the view for a future synchronization; it does not synchronously write to the DOM.

### 18. Is detectChanges() the normal way to update a zoneless view?

No. It forces an immediate local check and can hide missing notifications. Prefer signals, AsyncPipe, bound events, or markForCheck() at the correct state boundary.

### 19. How should a programmatically created component receive an input?

Use ComponentRef.setInput(). Directly assigning an @Input property from TypeScript does not provide the same Angular notification contract.

### 20. Do signals need to be immutable in zoneless applications?

Signals notify when their value changes according to their equality function. With default equality, mutating an object and setting the same reference does not count as a change. Replace the object or array so the signal can notify consumers.

## ZoneJS and integrations

### 21. What did ZoneJS do for Angular?

ZoneJS patches many browser APIs and observes asynchronous activity. Historically Angular used that activity as an indication that application state might have changed and scheduled change detection accordingly.

### 22. Why use zoneless scheduling?

It can avoid checks triggered by async work that did not change application state, reduce the ZoneJS dependency overhead, simplify debugging, and avoid some compatibility issues with patched browser APIs.

### 23. Must I remove every NgZone.run() call?

No. NgZone.run() and runOutsideAngular() remain compatible with zoneless applications. They are not required just to enable zoneless scheduling, but shared libraries may retain them to work well in Zone-based applications too.

### 24. Do NgZone.onStable and onMicrotaskEmpty emit in zoneless mode?

No. onMicrotaskEmpty, onUnstable, and onStable do not emit in a zoneless application. NgZone.isStable is always true, so it should not be used as a completion signal.

### 25. What should replace onStable when I need to wait for an Angular render?

Use afterNextRender() for one render or afterEveryRender() when work must run after every render. If the real requirement is to observe a specific DOM mutation, a browser API such as MutationObserver may be more direct.

### 26. Should I remove NgZone.runOutsideAngular() from a reusable library?

Not automatically. It remains compatible and can prevent unnecessary work in Zone-based applications. Evaluate the library's supported environments and the task it is isolating.

### 27. How do I handle a third-party widget callback?

At the integration boundary, update a signal read by the template or update a plain field and call markForCheck(). If the widget creates high-frequency work, profile and manage that work deliberately.

## Forms and SSR

### 28. Do reactive form APIs automatically schedule zoneless change detection?

Not necessarily. Methods such as setValue(), patchValue(), and FormArray.push() update form state and emit form observables, but they do not automatically schedule component change detection for a template that reads that state.

### 29. How can a template react to reactive form state?

Connect the relevant form observable to markForCheck(), or reflect the state through a signal consumed by the template. Choose a notification that corresponds to the state changes instead of polling.

### 30. Why does zoneless SSR need PendingTasks?

Server rendering needs to know when asynchronous work that affects the HTML is still pending. In zoneless SSR, register that work with PendingTasks so Angular waits for it before serializing the response.

### 31. Should every background task be registered with PendingTasks?

No. Register work that must finish before the server renders the response. Unrelated background work should not delay serialization.

### 32. What does PendingTasks.run() do?

It marks an asynchronous operation as pending while its callback runs, then clears that pending state when the operation completes. SSR can use that state to avoid serializing before required data is ready.

### 33. Is PendingTasks only for SSR?

Its main zoneless guidance use is informing SSR about work that affects serialization. Use it when that lifecycle requirement applies; ordinary client-side template updates should use normal signals or notifications.

## Tests and migration

### 34. Does TestBed use zoneless change detection?

Current Angular documentation says TestBed uses zoneless change detection by default, even when ZoneJS is present in the test polyfills. Add provideZoneChangeDetection() in TestBed only when a test specifically needs Zone-based scheduling.

### 35. Why can fixture.detectChanges() hide a zoneless bug?

It forces a check regardless of whether production code generated an Angular notification. A test that passes only after forced detection may hide a plain-field update that would remain stale in the application.

### 36. What should a zoneless-compatible test do instead?

Exercise the production notification, such as clicking a bound control, updating a template-read signal, setting an input through Angular, or emitting through AsyncPipe. Then wait for Angular's scheduled work to stabilize when appropriate.

### 37. Do I have to rewrite every existing test that calls fixture.detectChanges()?

No. Existing tests may use it extensively. Migrate deliberately, prioritizing tests where forced checks conceal a missing production notification.

### 38. What should I check before removing ZoneJS?

Check angular.json build and test polyfills, explicit polyfills.ts imports, package dependencies, and third-party integrations that rely on patched browser APIs. Then run the project's normal build and test workflows.

### 39. What is the most common migration bug?

A plain field gets updated from a timer, promise, external event, or library callback and the code assumes the async task itself will refresh the view. Use a template-read signal or call markForCheck() at that boundary.

### 40. Is OnPush required for zoneless compatibility?

No. Angular recommends OnPush for application components because it encourages clear notification paths, but it is not required. A component using another strategy can still be compatible if it notifies Angular when its rendered state changes.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="044-onpush-change-detection-and-immutable-updates.md">044 - 44. OnPush change detection and immutable updates</a></td><td><a href="../lessons/045-zoneless-angular-applications.md">Lesson</a></td><td><a href="../examples/045-zoneless-angular-applications.md">Example</a></td><td><a href="046-state-ownership-and-avoiding-duplicated-state.md">046 - State ownership and avoiding duplicated state</a></td>
</tr>
  </tbody>
</table>
