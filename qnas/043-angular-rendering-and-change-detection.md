# 43. Angular rendering and change detection — Q&A

## Core model

### 1. What is change detection in Angular?

It is Angular's process for checking template bindings in views and deciding which rendered values need to be updated. It connects application state to the DOM.

### 2. Is change detection the same as rendering?

No. Change detection checks binding values and determines what changed. Rendering applies necessary updates to the DOM so the browser shows the current state.

### 3. Does Angular recreate the entire page after every update?

No. Angular traverses relevant views, checks bindings, and updates the DOM where values have changed. It does not need to rebuild every DOM node for each state update.

### 4. What is a view tree?

It is the hierarchy of Angular views created by components and embedded template views. Change detection traverses this structure and checks eligible views.

### 5. What is the high-level update cycle?

A state update or Angular notification schedules synchronization; Angular checks eligible views and their bindings; lifecycle work runs at documented points; then Angular applies changed values to the DOM. Render callbacks run after Angular finishes rendering.

### 6. Does Angular guarantee an exact number of checks for a state update?

Application code should not rely on an assumed number of checks or on a specific internal scheduler sequence. Keep template expressions pure and use documented notifications and lifecycle APIs.

## Notifications, ZoneJS, and zoneless behavior

### 7. What kinds of things can notify Angular that a view needs refreshing?

Examples include a bound template or host event listener, a component input set through Angular, a signal read by a template changing, or a view being marked for check. APIs such as `AsyncPipe` also mark a view when values arrive.

### 8. If I update a signal in TypeScript, will every component refresh?

No. If a component template reads that signal, Angular tracks the relationship and can refresh that component when the signal changes. A signal that is not read by a template does not by itself identify that template as a consumer.

### 9. What did ZoneJS do for change detection?

In Zone-based applications, ZoneJS tracked asynchronous browser activity and helped Angular schedule synchronization after work such as events and timers. That could schedule checks even when application code had not explicitly notified Angular that a particular view needed updating.

### 10. What does zoneless mean?

Angular does not depend on ZoneJS tracking asynchronous browser tasks to schedule synchronization. Angular APIs such as template-read signals, bound listeners, input updates, `AsyncPipe`, and `markForCheck()` provide notifications.

### 11. Since which Angular version is zoneless the default?

Zoneless is the default starting in Angular v21. Angular v20 can opt in with `provideZonelessChangeDetection()`. Existing applications and older versions can have different configuration, so check the project setup.

### 12. Why might a plain field update from an external callback not appear in a zoneless template?

A plain assignment by itself is not an Angular notification. Use a Signal read by the template, or mark the view for checking with `ChangeDetectorRef.markForCheck()` when integrating an external callback.

~~~ts
this.status = 'Connected';
this.changeDetector.markForCheck();
~~~

### 13. Does `NgZone.run()` need to be used to make every update render in a zoneless application?

No. Zoneless applications use Angular notification APIs rather than ZoneJS task tracking. Prefer framework-aware state such as template-read Signals or `markForCheck()` instead of using `NgZone.run()` as a generic rendering command.

## Lifecycle and stable views

### 14. How are lifecycle hooks related to change detection?

Many hooks run while Angular traverses views and checks bindings. Each hook has a documented lifecycle phase; use the hook that matches the work instead of depending on accidental observed ordering.

### 15. What is the initial ordering of `ngOnChanges` and `ngOnInit`?

The initial `ngOnChanges` runs after inputs are set and before `ngOnInit`. Both occur before that component's template is initialized.

### 16. Why should I avoid `ngDoCheck`, `ngAfterContentChecked`, and `ngAfterViewChecked` for routine logic?

They run frequently when their associated views are checked. Deriving state, subscribing, or doing expensive work in them can add repeated cost. Prefer signals, computed state, event handlers, or a focused lifecycle API.

### 17. What is `ExpressionChangedAfterItHasBeenCheckedError` (`NG0100`)?

In development mode, Angular performs an additional stability check after change detection. `NG0100` means a binding changed after it had already been checked, so the view was not stable during that cycle.

### 18. What commonly causes `NG0100`?

Examples include a child changing a parent value after it was checked, mutating a binding in a late lifecycle hook, or a side-effecting template method that returns a different value during the same cycle.

### 19. What is the right fix for `NG0100`?

Correct the state flow or move the work to the lifecycle phase that fits it. Do not add arbitrary timers or repeated `detectChanges()` calls to hide the unstable update.

### 20. Do template methods run only once?

No. Angular can evaluate a binding again whenever its view is checked. Keep methods/getters used in templates fast, deterministic, and free of side effects. For reactive derivations, consider a `computed()` signal.

### 21. When does `afterNextRender()` run?

It runs after Angular renders the page to the DOM, once for the next render. It must be registered in an injection context and does not run during server-side rendering or build-time prerendering.

### 22. When should I use `afterEveryRender()`?

Only when a callback really needs to run after every completed render. For one-time work, use `afterNextRender()`. For element size or intersection, a browser observer may be a better fit.

## ChangeDetectorRef and strategy defaults

### 23. What does `markForCheck()` do?

It marks a view as needing a future check. It does not synchronously update the DOM at the call site. Angular processes it during a scheduled synchronization.

### 24. What does `detectChanges()` do?

It immediately checks the current view and its children. It is mainly useful for deliberate local rendering strategies, often alongside a detached view, rather than as a routine update mechanism.

### 25. What does `detach()` do?

It removes a view from the normal change-detection tree. Angular will not check that view during normal runs even if it is marked dirty. `reattach()` returns it to the tree.

### 26. Since which version is `OnPush` the default component strategy?

It is the default starting in Angular v22. Earlier versions defaulted to the eager strategy, historically named `Default`. Check a project's version and explicit component configuration. The next chapter covers `OnPush` behavior in detail.

### 27. Does setting `ChangeDetectionStrategy.OnPush` mean Angular never checks the component?

No. Angular checks an `OnPush` subtree when its documented triggers occur, such as a changed template-bound input, an event in the subtree, an explicit mark, or a changed signal read by the template. See the next chapter for scenarios and subtree behavior.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="042-rxjs-interop-with-tosignal-and-toobservable.md">042 - 42. RxJS interop with `toSignal()` and `toObservable()`</a></td><td><a href="../lessons/043-angular-rendering-and-change-detection.md">Lesson</a></td><td><a href="../examples/043-angular-rendering-and-change-detection.md">Example</a></td><td><a href="044-onpush-change-detection-and-immutable-updates.md">044 - 44. OnPush change detection and immutable updates</a></td>
</tr>
  </tbody>
</table>
