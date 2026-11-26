# 31. Q&A: Lifecycle hooks and component initialization

## Lifecycle fundamentals

### 1. What is a component lifecycle?

A component lifecycle is the sequence of stages Angular takes a component through: creating it, checking it as application state changes, rendering its view, and eventually destroying it. Lifecycle hooks let a component run code at specific points in that sequence.

### 2. What is the lifecycle hook order for a component's first check?

The main order is:

1. `constructor`
2. `ngOnChanges` (when the component has input changes)
3. `ngOnInit`
4. `ngDoCheck`
5. `ngAfterContentInit`
6. `ngAfterContentChecked`
7. `ngAfterViewInit`
8. `ngAfterViewChecked`
9. Angular completes the render; registered render callbacks can run

On later checks, `ngOnChanges` runs when an input changes, then `ngDoCheck`, the content and view checked hooks, and render callbacks according to their registration. Destruction later calls `ngOnDestroy` and registered destroy callbacks.

### 3. What is the difference between a constructor and `ngOnInit`?

The constructor is JavaScript class construction. Angular dependency injection is available there, but Angular has not yet supplied the component's initial inputs. `ngOnInit` runs once after the initial inputs are set and before Angular initializes the component's own view. Use it when initialization depends on input values; keep constructors focused on dependency setup and simple field initialization.

### 4. Does `ngOnChanges` run before `ngOnInit`?

Yes. When a component has input changes, the initial `ngOnChanges` runs before `ngOnInit`. This lets the component respond to initial input values before one-time initialization.

### 5. When does `ngOnChanges` run?

Angular calls it when one or more inputs change. It runs once during the initial input setup and again on later input changes. It does not run just because an unrelated local field changed.

### 6. Can `ngOnChanges` observe signal inputs?

Yes. Angular calls `ngOnChanges` for changed inputs, including inputs declared with the `input()` API. Read a signal input by calling it, such as `this.title()`. The `SimpleChanges` record uses the input property's TypeScript name as its key, including when a decorator input has a public alias.

### 7. What information does `SimpleChanges` contain?

For each changed input, `SimpleChanges` provides a `SimpleChange` with `previousValue`, `currentValue`, and `firstChange`. Its `isFirstChange()` method reports whether the change is the initial input assignment.

```ts
ngOnChanges(changes: SimpleChanges<MyComponent>): void {
  const titleChange = changes.title;

  if (titleChange) {
    console.log(titleChange.previousValue);
    console.log(titleChange.currentValue);
    console.log(titleChange.firstChange);
  }
}
```

### 8. Does `ngOnInit` run again when an input changes?

No. `ngOnInit` runs once per component instance. Use `ngOnChanges` or a reactive computation for work that should respond to later input values.

### 9. What is `ngDoCheck` for?

`ngDoCheck` lets a component implement custom change-detection checks. Angular calls it frequently, so expensive work in this hook can slow down the whole application. Prefer signals, computed values, and normal input change handling when they express the need.

### 10. What does “content” mean in content lifecycle hooks?

Content is markup projected into a component through `<ng-content>`. `ngAfterContentInit` runs once after projected content is initialized; `ngAfterContentChecked` runs after Angular checks it.

### 11. What does “view” mean in view lifecycle hooks?

A component's view is the template it owns, including its child views. `ngAfterViewInit` runs once after that view is initialized; `ngAfterViewChecked` runs after Angular checks it. These hooks are often used when querying child view elements, though query signals are usually a more direct way to read queries.

### 12. Why should I avoid changing bound state in after-init and after-checked hooks?

Angular has already checked the relevant bindings at those points. Changing a value that affects them can make the current check inconsistent and produce `ExpressionChangedAfterItHasBeenCheckedError` in development mode. Set initial state earlier, or perform DOM work in an appropriate render callback.

### 13. What is the difference between `ngAfterContentInit` and `ngAfterViewInit`?

`ngAfterContentInit` is for projected content. `ngAfterViewInit` is for the component's own template and child views. The content initialization hook runs before the view initialization hook.

### 14. Are `ngAfterContentChecked` and `ngAfterViewChecked` one-time hooks?

No. They run whenever Angular checks the relevant content or view. Keep them very small and avoid using them as general-purpose update handlers.

## Render callbacks

### 15. What is `afterNextRender`?

`afterNextRender` registers a function to run once after Angular has rendered all components to the DOM. It is useful for browser DOM work that must wait until rendering completes. It is a function API, not a method named `ngAfterNextRender`.

### 16. How is `afterNextRender` different from `afterEveryRender`?

`afterNextRender` runs after the next application render only. `afterEveryRender` runs after every application render. Both are application-wide render callbacks rather than hooks tied to one component's instance-check sequence.

### 17. Where can render callbacks be registered?

They must be registered in an injection context, commonly in a component or service constructor. They can also be registered with an explicit injector through the API options. Do not register them from an ordinary event handler unless you deliberately establish an injection context.

### 18. Do render callbacks run during server-side rendering?

No. Angular documents render callbacks as browser-only; they do not run during server-side rendering or prerendering. Code that accesses `window`, `document`, or browser-only APIs must account for that. Also, a render callback alone does not guarantee that a component has completed hydration.

### 19. What are the render callback phases?

Angular provides `earlyRead`, `write`, `mixedReadWrite`, and `read` phases. Use `write` for DOM writes and `read` for layout reads; use `earlyRead` only when a read must precede writes. Avoid `mixedReadWrite` when separate phases can prevent layout thrashing.

### 20. When should I use a render callback instead of a view hook?

Use a render callback for DOM operations that must happen after Angular has rendered the application. A view hook runs during the component lifecycle check, before the application-wide render-complete point. Prefer Angular bindings and directives for DOM updates that Angular can perform declaratively.

## Cleanup and destruction

### 21. When does `ngOnDestroy` run?

Angular calls `ngOnDestroy` once just before it destroys a component, directive, pipe, or service instance. Use it to clean up resources that the instance owns, such as a manually created timer or third-party subscription.

### 22. What is `DestroyRef`?

`DestroyRef` lets code register cleanup with the lifecycle scope where it was injected. Its `onDestroy()` method accepts a callback and returns a function that can unregister that callback. Its `destroyed` property tells whether the scope has already been destroyed.

### 23. What does `takeUntilDestroyed` do?

`takeUntilDestroyed` is an RxJS operator that completes an observable subscription when its associated Angular context is destroyed. It can use the current injection context or receive a `DestroyRef` explicitly. This avoids maintaining a separate “destroy subject” for many subscriptions.

### 24. Should I use both `ngOnDestroy` and `DestroyRef`?

Usually one cleanup approach is enough for a given resource. Use `ngOnDestroy` when the cleanup naturally belongs in a component method. Use `DestroyRef` when cleanup is registered near the resource setup or inside a helper function. `takeUntilDestroyed` is convenient for RxJS subscriptions.

### 25. Do I have to implement Angular lifecycle interfaces?

No. Angular calls a lifecycle method by its documented name and signature; implementing the matching TypeScript interface is optional. The interfaces are useful because they give the compiler a check for misspelled method names and incorrect signatures.

### 26. Does a component have to implement every lifecycle hook?

No. Implement only the hooks needed by that component. Unused hooks add noise and can encourage unnecessary work during frequent checks.

## Common design and debugging questions

### 27. Is the ordering between a component and directives on the same host element guaranteed?

No. Angular does not guarantee lifecycle-hook ordering between a component and directives attached to the same element. Avoid designs that depend on one of those hooks running first.

### 28. What should I use for a value derived from other state?

Use a `computed()` signal when the value is derived from signals. Do not copy a derived value into a field from `ngDoCheck` or a checked hook; that adds repeated work and can create timing problems.

### 29. How can I tell whether a component is being recreated or merely checked?

Log its constructor and `ngOnDestroy` alongside check hooks. A new constructor call indicates a new instance; repeated check hooks on the same instance are normal change detection. Stable tracking and conditional rendering determine whether list items or conditional components are retained or recreated.

### 30. Why does an `ngDoCheck` log appear many times after clicking a button?

Angular may perform multiple checks as application state and events are processed. `ngDoCheck` is intentionally a frequent hook, not a one-time “something changed” notification. Remove temporary logging once you understand the sequence, and do not put expensive work there.

### 31. Can lifecycle hooks replace event handlers?

No. Hooks describe Angular-managed component lifecycle stages. Use event handlers for user actions, input handling or reactive state for state changes, and lifecycle hooks for setup, view/content readiness, and cleanup.

### 32. What is a safe way to perform layout measurement?

Wait until the relevant render, then use a render callback's read phase for layout measurement. Keep DOM writes in the write phase and reads in the read phase to reduce forced synchronous layout. Guard browser-specific APIs because render callbacks do not execute on the server.

### 33. What is the safest place to initialize state that affects the template?

Initialize default values as fields or signals, and use `ngOnInit` when one-time initialization needs initial inputs. For values derived from other reactive state, use `computed()`. This keeps state ready before later content and view checked hooks.

### 34. What should I check when a lifecycle hook seems not to run?

Confirm the component is actually instantiated, the hook name and signature are correct, and the specific condition for that hook occurred. For example, `ngOnChanges` requires an input change, `ngAfterContentInit` requires content initialization, and `ngOnDestroy` requires that Angular destroy the instance.

## Related material

- [Lesson 31: Lifecycle hooks and component initialization](../lessons/031-lifecycle-hooks-and-component-initialization.md)
- [Example 31: Lifecycle hooks and component initialization](../examples/031-lifecycle-hooks-and-component-initialization.md)
- Angular component lifecycle guide
- [Angular `afterNextRender` API](`https://angular.dev/api/core/afterNextRender`)
- [Angular `DestroyRef` API](`https://angular.dev/api/core/DestroyRef`)
- [Angular `takeUntilDestroyed` API](`https://angular.dev/api/core/rxjs-interop/takeUntilDestroyed`)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="030-view-queries-and-content-queries.md">030 - 30. View queries and content queries</a></td><td><a href="../lessons/031-lifecycle-hooks-and-component-initialization.md">Lesson</a></td><td><a href="../examples/031-lifecycle-hooks-and-component-initialization.md">Example</a></td><td><a href="032-host-bindings-host-listeners-and-host-directives.md">032 - 32. Host bindings, host listeners, and host directives</a></td>
</tr>
  </tbody>
</table>
