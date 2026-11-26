# Rendering Performance, Change Detection, and Profiling - Q&A

## 1. What is Angular change detection?

Change detection synchronizes Angular views with application state. When Angular is notified that state may have changed, it checks the relevant views and updates bindings whose values differ. It does not necessarily recreate the entire DOM.

## 2. Does every change-detection pass update the DOM?

No. Angular evaluates applicable bindings and updates the DOM when a binding's value changes. A check can find no visible changes. Expensive expressions can still cost time even when the resulting DOM remains the same.

## 3. Should I optimize change detection before profiling?

No. Profile a reproducible user interaction first. A delay could come from a request, third-party JavaScript, layout, or paint rather than Angular checks. Optimize the measured bottleneck.

## 4. What does OnPush do?

OnPush lets Angular skip an unchanged component subtree until a relevant update notification occurs. This can limit work in a large component tree when component inputs and state updates are modeled clearly.

## 5. Is OnPush the default?

Current Angular documentation says OnPush is the default starting in Angular v22. Earlier versions generally used eager checking by default. Check the installed Angular version and the component's actual configuration instead of assuming one default for every project.

## 6. When can Angular check an OnPush component?

Common notifications include a changed value from a template-bound input, an event handled within the component subtree, a signal read by its template changing, an AsyncPipe emission, or ChangeDetectorRef.markForCheck. Exact scheduling details depend on the Angular version and application configuration.

## 7. Why can mutating an input object cause a stale child view?

OnPush input checks rely on the input value changing. If the parent mutates an object but passes the same object reference, Angular may not treat it as a new input. Prefer an immutable update that creates a new reference.

## 8. Why should I avoid methods in templates?

A method used in a template can be evaluated during applicable checks. If it performs filtering, sorting, formatting, or other expensive work, it may repeat unnecessarily. Use cheap expressions, derived state, computed signals, or a pure pipe when appropriate.

## 9. Are computed signals always fast?

No. Computed signals memoize results until their tracked dependencies change, which avoids recomputing when dependencies remain unchanged. The computation can still be expensive when it does run. Profile it and improve the underlying algorithm if needed.

## 10. What is the benefit of track in an @for loop?

A stable tracking key helps Angular associate each item with its existing view and DOM nodes. This can reduce unnecessary creation and removal when a list changes. Use a stable unique identifier when available.

## 11. Is track by index always safe?

No. If the list is reordered, inserted into, or filtered, an index can refer to a different logical item. Stable IDs are usually better. Index tracking is only appropriate for collections whose order and identity behavior make it correct.

## 12. Does OnPush eliminate the need for list virtualization?

No. OnPush can skip unchanged component subtrees, while virtualization limits how many list items are rendered at once. A very large visible list may still need pagination or virtualization if profiling shows rendering and DOM size are the bottleneck.

## 13. What is zoneless change detection?

Zoneless change detection schedules Angular view synchronization from framework notifications rather than relying on ZoneJS to observe asynchronous browser activity. This can reduce unnecessary checks and remove the ZoneJS dependency.

## 14. Which Angular versions enable zoneless by default?

Angular's current documentation says zoneless is stable from v20.2 and enabled by default in Angular v21 and later. Confirm the current workspace version and ensure the app has not explicitly configured ZoneJS-based change detection.

## 15. What needs attention when migrating to zoneless?

Ensure state changes notify Angular. Signals read in templates, template listeners, ComponentRef.setInput, AsyncPipe, and markForCheck are common notification paths. Check third-party callbacks and reactive forms flows, because a model update alone may not schedule rendering in every integration.

## 16. When should I use markForCheck?

Use markForCheck when an update occurs through a path Angular does not otherwise observe and the view should be included in a future change-detection pass. Prefer a reactive state or framework notification pattern where possible.

## 17. Should I use detectChanges to fix a stale UI?

Usually not as a general fix. detectChanges performs an immediate local check and can hide the fact that the update path does not notify Angular. Find the source of the state change and connect it to a normal notification mechanism first.

## 18. Should I detach a view for performance?

Only for a deliberate use case with a clear update strategy. A detached view is not checked automatically, so it can display stale data until explicitly refreshed. Detachment is an advanced control, not a default optimization.

## 19. What is zone pollution?

In a ZoneJS-based app, a third-party library may schedule timers or events that cause Angular to check views even when application state did not change. Use profiling to confirm this. If appropriate, run non-Angular work outside the Angular zone and notify Angular only when visible application state changes.

## 20. Why can DOM measurements be slow?

Reading layout properties can force synchronous style and layout calculation. Alternating layout reads and DOM writes repeatedly can cause layout thrashing. Batch reads and writes, prefer CSS layout, and confirm the cost in a browser performance trace.

## 21. Does OnPush improve performance automatically?

No. It creates opportunities to skip unchanged views, but a component can still have expensive template work, frequent input changes, large lists, or costly child rendering. Verify that the target subtree is actually skipped or faster in a profile.

## 22. Are development profiler timings production benchmarks?

No. Development instrumentation, runtime mode, machine load, and browser settings affect the result. Use traces to locate expensive work and use production-like builds and field data to assess user-facing performance.

## 23. How do I know whether a slow interaction is caused by Angular?

Compare Angular DevTools with Chrome Performance. Angular DevTools helps identify component checks and lifecycle work. Chrome Performance shows broader JavaScript, layout, paint, network, and third-party activity.

## 24. What is a reliable optimization process?

1. Reproduce and record the slow interaction.
2. Profile the same scenario on a representative build.
3. Identify a specific costly expression, component, list, or browser operation.
4. Make one focused change.
5. Repeat the same trace and verify the user-visible behavior.
6. Check for stale state or missed notifications.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="111-lazy-loading-and-deferrable-views-with-defer.md">111 - Lazy Loading and Deferrable Views with defer</a></td><td><a href="../lessons/112-rendering-performance-change-detection-and-profiling.md">Lesson</a></td><td><a href="../examples/112-rendering-performance-change-detection-and-profiling.md">Example</a></td><td><a href="113-client-rendering-server-rendering-and-prerendering.md">113 - Client Rendering, Server Rendering, and Prerendering</a></td>
</tr>
  </tbody>
</table>
