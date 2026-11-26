# 40. Effects, cleanup, and when not to use effects — Q&A

## Core concepts

### 1. What is an Angular effect?

An effect is a reactive operation that runs imperative code when signals it reads change. It is intended to synchronize Angular signal state with non-reactive systems such as browser storage, logging, timers, or third-party libraries.

### 2. Does an effect run once or only after a signal changes?

It runs at least once when Angular schedules it. After that, it is scheduled again when a signal read during its previous synchronous execution changes.

### 3. Does `effect()` return a calculated value?

No. It returns an `EffectRef`, which controls the effect's lifetime. If a template needs a value derived from other state, use a `computed()` signal instead.

### 4. Does an effect run immediately after `signal.set()`?

No. Effects are scheduled as part of Angular's reactive execution. Code that needs the value immediately can read the signal after calling `set()`; it should not wait for the effect to run.

### 5. Which signal reads become dependencies?

Synchronous reads made while the effect callback runs are tracked. Angular updates the dependency set after each run, so conditional branches can add or remove dependencies.

### 6. Are signal reads after `await` tracked?

No. Reactive context is synchronous. Capture the signal value before `await` or before starting a promise callback if that value should be a dependency.

~~~ts
effect(async () => {
  const selectedId = selectedUserId(); // Tracked.
  const user = await loadUser(selectedId);
  console.log(user.name, theme()); // theme() is read too late to be tracked.
});
~~~

A clearer version reads all dependencies before the asynchronous boundary:

~~~ts
effect(async () => {
  const selectedId = selectedUserId();
  const currentTheme = theme();
  const user = await loadUser(selectedId);
  console.log(user.name, currentTheme);
});
~~~

### 7. What kinds of effects does Angular have?

Angular distinguishes component effects and root effects. Component effects are created in component/directive context (including services tied to their injector) and run during Angular synchronization. Root effects are created outside a component tree, such as in a root-provided service; they run as microtasks and are not tied to a component view's change-detection pass. Their scheduling differs, so application code should not rely on effects as synchronous callbacks.

### 8. When is an effect a good choice?

Use one when a signal change must invoke an imperative API: persist a display preference, report analytics, start or cancel an imperative operation, or update a third-party chart or canvas. Prefer Angular templates and reactive APIs when they already express the behavior.

### 9. Why should an effect not copy one signal into another?

That creates redundant state and an extra scheduled update. The two values can become inconsistent, and circular updates can cause repeated work or change-detection errors. Use `computed()` for read-only derived state or `linkedSignal()` for derived state that can also be changed.

### 10. When should I use `computed()` versus `linkedSignal()`?

Use `computed()` when a value is entirely derived and should be read-only. Use `linkedSignal()` when a value follows source state but should remain writable between source changes, such as a selection that resets when its options list changes. Do not add an effect just to synchronize either value.

## Creating and managing effects

### 11. Where can I normally create an effect?

Create it in an injection context, commonly in a DI-created class constructor or field initializer. A component, directive, or service instantiated by Angular provides such a context.

### 12. How can I create an effect later from a method?

Capture an `Injector` while in an injection context, then pass it to the `effect()` options:

~~~ts
private readonly injector = inject(Injector);

start(): void {
  this.ref = effect(() => this.log(this.status()), {
    injector: this.injector,
  });
}
~~~

Also decide when the effect should stop. A method invoked repeatedly can otherwise create multiple independent effects.

### 13. What is the `onCleanup` callback for?

It registers work that undoes resources started by one effect run. Angular invokes the cleanup before the next run and when the effect is destroyed. Typical uses include clearing a timer, unsubscribing, removing an event listener, or aborting a request.

### 14. When should I register cleanup?

Register it as part of the same effect run that creates the resource, and do so before returning or crossing an asynchronous boundary. Cleanup should refer to that run's resource so it cannot accidentally dispose of a newer one.

### 15. How do I prevent an old request from overwriting a newer result?

Create an `AbortController` for the current run and register `controller.abort()` with `onCleanup`. When the query changes, Angular cleans up the prior run and its request can be cancelled. Still handle request errors, including the abort case.

### 16. Does Angular destroy effects automatically?

Normally, yes. An effect associated with a component or directive is destroyed with that owner. A root effect normally lives until application destruction. An `EffectRef` can be destroyed earlier with `.destroy()`.

### 17. What does `manualCleanup: true` change?

It opts out of automatic destruction registration with the current owner. The application must keep the `EffectRef` and call `.destroy()` when finished. If the reference is lost, the effect can live longer than intended; use this option only when a separately managed lifetime is necessary.

### 18. Can I use `untracked()` inside an effect?

Yes. It prevents signal reads inside a narrowly scoped function from becoming dependencies of the current reactive execution. Use it for incidental reads, such as a logging service reading signals internally. Do not wrap a value that should trigger the effect.

~~~ts
effect(() => {
  const user = currentUser();
  untracked(() => analytics.logUser(user));
});
~~~

### 19. Can cleanup itself read signals to control the next effect run?

No. Cleanup is for disposing of work created by the prior run. Read the dependencies in the effect callback and capture the values needed by cleanup. Do not rely on cleanup-time reads to establish effect dependencies.

## DOM, SSR, and troubleshooting

### 20. Should I use a regular effect to read an element after a conditional template renders?

Usually not. A regular effect can run before Angular updates the DOM for that synchronization. First consider template bindings, view queries, or component lifecycle APIs. If imperative work genuinely needs the committed DOM, use `afterRenderEffect()`.

### 21. When is `afterRenderEffect()` appropriate?

Use it for an imperative DOM integration that must run after Angular has rendered, such as updating a chart from a signal. It runs on the browser only, and a component may not yet be hydrated when it executes, so direct DOM access needs care.

### 22. What are `afterRenderEffect()` phases?

The phases are `earlyRead`, `write`, `mixedReadWrite`, and `read`. Prefer `write` for DOM writes and `read` for DOM reads, separating them where possible to reduce layout thrashing. `mixedReadWrite` combines them and should be avoided when separate phases can express the work.

### 23. Is `afterRenderEffect()` always the right API for observing DOM changes?

No. For element size, visibility, or DOM mutations, browser APIs such as `ResizeObserver`, `IntersectionObserver`, and `MutationObserver` may be a better fit than rerunning an Angular effect.

### 24. Can browser storage be accessed from an effect during server rendering?

Not safely without a browser guard: `window` does not exist on the server. Keep browser-specific work behind an environment check or another platform-aware design, and account for storage access failures in the browser as well.

### 25. Why does my effect run more often than expected?

Inspect all synchronous signal reads, including reads in helpers called by the effect. A helper may read additional signals and silently create dependencies. Use `untracked()` only for incidental reads that must not trigger reruns; otherwise, make dependencies explicit.

### 26. Why does an effect appear not to react to a value read in a callback?

The value may only be read later, after the effect has returned—for example, inside a timer, promise continuation, or event callback. Such a read is outside the effect's reactive context. Read and capture the signal synchronously in the effect if changes to it should rerun the effect.

### 27. What should I check if a method creates duplicate side effects?

Check whether the method creates a new effect each time it is called. Create the effect once when its owner is initialized, or retain its `EffectRef` and destroy the old instance before replacing it. Also make sure cleanup removes resources from earlier runs.

### 28. What is a good decision checklist before adding an effect?

Ask:

1. Is this code synchronizing with an imperative, non-signal API?
2. Could a template, `computed()`, or `linkedSignal()` express it directly?
3. Which signal reads should cause reruns?
4. Does each run create a timer, listener, subscription, or request that needs cleanup?
5. Does this effect need a browser-only or post-render guard?
6. Who owns the effect, and when should it be destroyed?

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="039-linkedsignal-and-dependent-writable-state.md">039 - 39. `linkedSignal` and dependent writable state</a></td><td><a href="../lessons/040-effects-cleanup-and-when-not-to-use-effects.md">Lesson</a></td><td><a href="../examples/040-effects-cleanup-and-when-not-to-use-effects.md">Example</a></td><td><a href="041-signal-equality-dynamic-dependencies-and-untracked-reads.md">041 - 41. Signal equality, dynamic dependencies, and untracked reads</a></td>
</tr>
  </tbody>
</table>
