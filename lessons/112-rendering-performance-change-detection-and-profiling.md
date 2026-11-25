# Rendering Performance, Change Detection, and Profiling

Angular keeps component state and the rendered DOM in sync through change detection. The framework is optimized for common application patterns, so do not begin by adding manual change-detection controls. Measure a real slow interaction, identify the work taking time, and then address that cause.

This chapter focuses on runtime rendering. Bundle size and Web Vitals are covered in chapter 110; route and view code splitting are covered in chapter 111.

## Learning outcomes

After this lesson, you should be able to:

- Describe how Angular checks component views and updates the DOM.
- Use Angular DevTools and browser performance traces to find slow rendering.
- Apply OnPush boundaries and immutable input updates correctly.
- Use signals, computed values, and stable list tracking to avoid unnecessary work.
- Recognize expensive template expressions, lifecycle work, and layout reads.
- Understand zoneless change detection and version-dependent defaults.
- Validate a rendering optimization with a repeatable measurement.

## 1. What change detection does

A component template reads state and produces DOM. When Angular is notified that state may have changed, it synchronizes relevant views and updates bindings whose values differ. This is not the same as rebuilding the entire page for every update.

Potential sources of rendering cost include:

- A large component tree that is checked frequently.
- Expensive work in template expressions or lifecycle hooks.
- Large lists with unstable item identity.
- Unnecessary state changes or recreated objects.
- Repeated synchronous layout reads mixed with DOM writes.
- Third-party timers or event sources causing unnecessary checks in ZoneJS-based apps.

Most applications do not need manual change-detection APIs. Start with the trace and use the narrowest change that fixes the measured bottleneck.

## 2. Profile a user-visible interaction

Use a repeatable interaction such as opening a large menu, typing in a filter, changing a tab, or updating a long list.

1. Use a production-like build and a representative device profile.
2. Open Angular DevTools Profiler or Chrome DevTools Performance.
3. Record only the relevant interaction.
4. Identify the slow change-detection cycle or long task.
5. Inspect the component, template, lifecycle method, and browser work involved.
6. Repeat the same action after making one focused change.

Angular DevTools can show which components and directives consumed time in a change-detection cycle. Chrome Performance can reveal whether the delay is Angular work, another script, layout, paint, or a combination. See [Angular slow computations](https://angular.dev/best-practices/slow-computations) and [profiling with Chrome DevTools](https://angular.dev/best-practices/profiling-with-chrome-devtools).

## 3. Keep template expressions inexpensive

Angular evaluates template expressions during the checks that apply to a view. A costly method call can therefore repeat more often than expected:

~~~ts
get visibleProducts(): Product[] {
  return this.products.filter((product) =>
    product.name.toLowerCase().includes(this.query.toLowerCase()),
  );
}
~~~

Prefer state that is derived when its inputs change. Signals can express this relationship with computed:

~~~ts
readonly query = signal('');
readonly products = signal<Product[]>([]);
readonly visibleProducts = computed(() => {
  const normalizedQuery = this.query().trim().toLowerCase();

  return this.products().filter((product) =>
    product.name.toLowerCase().includes(normalizedQuery),
  );
});
~~~

A computed signal is lazy and memoized until a tracked dependency changes. Still, its calculation must be efficient when it does run. If profiling shows the filtering algorithm itself dominates, improve the algorithm, reduce the searched data, or move expensive work to an appropriate asynchronous boundary.

Pure pipes are another option for values derived from stable inputs. Memoization can help for repeated expensive calculations but has memory and invalidation trade-offs. Choose based on evidence.

## 4. Track repeated items by stable identity

For a rendered collection, stable tracking lets Angular associate an existing DOM view with the corresponding data item:

~~~html
@for (product of products(); track product.id) {
  <app-product-row [product]="product" />
}
~~~

Use an identifier that remains stable for the item. Tracking by object identity can cause unnecessary DOM removal and recreation when a server response creates new object instances for the same records. Tracking by index can be incorrect when items are inserted, removed, or reordered.

Stable tracking helps Angular reuse views; it does not make an expensive row template or huge list free. Consider pagination or virtual scrolling only when the measured data volume and user experience warrant it.

## 5. Use OnPush boundaries deliberately

OnPush lets Angular skip an unchanged component subtree until it receives a relevant notification. Current Angular documentation states that OnPush is the default change-detection strategy starting in Angular v22. Earlier versions commonly default to eager checking, so verify the application's Angular version and component behavior.

A component can declare the strategy explicitly:

~~~ts
import { ChangeDetectionStrategy, Component } from '@angular/core';

@Component({
  selector: 'app-product-card',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: '<h2>{{ product().name }}</h2>',
})
export class ProductCardComponent {
  readonly product = input.required<Product>();
}
~~~

Angular can check an OnPush subtree when its input binding receives a changed value, an event occurs within the subtree, a template-read signal changes, or the view is marked for check. The input comparison uses object identity. Mutating an object in place while keeping the same reference may not notify the child as intended.

Prefer immutable updates:

~~~ts
this.products.update((products) =>
  products.map((product) =>
    product.id === changed.id ? { ...product, ...changed } : product,
  ),
);
~~~

OnPush is not a substitute for profiling. It is a useful boundary when component inputs and update notifications are well-defined.

## 6. Understand Angular's update notifications

For zoneless change detection, Angular schedules synchronization from explicit framework notifications, including:

- Updating a signal read by a template.
- A bound host or template listener being triggered.
- Setting a component input through the framework.
- Calling ChangeDetectorRef.markForCheck.
- An AsyncPipe notification that marks a view for checking.

If a view changes data through an external callback that Angular does not observe, connect the update to a supported notification mechanism. Avoid relying on arbitrary asynchronous activity to refresh the UI.

## 7. Zoneless change detection

Zoneless change detection removes ZoneJS as Angular's scheduling signal and uses Angular notifications to decide when to synchronize views. It became stable in Angular v20.2 and is the default beginning in Angular v21. Check the installed version and workspace configuration before adding providers: adding a zone-based provider can override the default.

Existing applications need to ensure that state changes notify Angular, particularly when using third-party callbacks or reactive forms. Angular signals, template listeners, AsyncPipe, and markForCheck are common ways to provide the notification. A migration should be validated against real application flows and tests.

See the current [zoneless guide](https://angular.dev/guide/zoneless).

## 8. Avoid unnecessary layout work

Reading layout properties such as offsetWidth or getBoundingClientRect can force the browser to calculate layout. Repeating layout reads and DOM writes in a loop can cause layout thrashing.

When custom DOM measurement is necessary:

- Batch reads together and writes together.
- Prefer CSS layout and responsive techniques over manual measurements.
- Use browser performance traces to confirm a layout bottleneck.
- Avoid reading layout on every change-detection pass or for every list item.

Direct DOM access should be limited to cases that cannot be handled cleanly by Angular templates and CSS.

## 9. Zone pollution in ZoneJS applications

Some third-party libraries schedule timers or listen to frequent browser events even when their work does not change Angular state. In a ZoneJS application, those tasks can trigger checks unnecessarily.

Profile first. If the trace shows repeated checks from such a library, initialize its non-Angular work outside Angular's zone and re-enter Angular only when application state must update. Do not use this pattern in place of changing the source of needless work, and verify compatibility with zoneless applications.

## 10. Common anti-patterns

Avoid these patterns unless a measured and documented case requires them:

- Calling expensive methods from a frequently checked template.
- Mutating child input objects without changing their reference.
- Creating a new object or array during every template evaluation.
- Using ChangeDetectorRef.detectChanges as a general fix for stale UI.
- Detaching large views without a clear refresh strategy.
- Optimizing a component based only on a development-mode timing.
- Adding memoization without considering cache growth and invalidation.
- Deferring essential content in a way that creates layout shifts.

## 11. Verify the result

A complete optimization compares the same user journey before and after:

- Confirm the target interaction became faster in a representative profile.
- Check that the UI still updates for signal, event, async, and input changes.
- Inspect adjacent components for stale values or unnecessary rerenders.
- Compare browser traces, not just Angular profiler timings.
- Check field data when available.

Keep the trace, route, build, device, and reproduction steps with the change so that later measurements can be compared.

## Key takeaways

- Angular change detection is optimized; measure before introducing manual controls.
- The profiler identifies expensive cycles; the browser trace provides broader context.
- Keep template calculations cheap and list identity stable.
- OnPush boundaries work best with clear inputs and explicit update notifications.
- Angular v21+ uses zoneless change detection by default; OnPush becomes default in v22 according to current Angular docs.
- Re-measure the same interaction and confirm the UI still responds correctly.

## Further reading

- [Angular performance best practices](https://angular.dev/best-practices/performance)
- [Slow computations](https://angular.dev/best-practices/slow-computations)
- [Skipping component subtrees](https://angular.dev/best-practices/skipping-subtrees)
- [Angular zoneless change detection](https://angular.dev/guide/zoneless)
- [Angular DevTools Profiler](https://angular.dev/tools/devtools/profiler)