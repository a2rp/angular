# 40. Effects, cleanup, and when not to use effects

An Angular effect runs imperative code in response to signal changes. Use it to synchronize signal state with something outside Angular's reactive state model, such as browser storage, analytics, timers, or a third-party library. An effect is not the default way to calculate another piece of state: use `computed()` for read-only derived values and `linkedSignal()` when derived state also needs to be writable.

`effect()` is stable in Angular 20 and later. Check the target application's Angular version before using it in older projects. The examples in this lesson use modern signal APIs.

## Learning goals

After this lesson, you should be able to:

- Explain what an effect tracks and when it runs.
- Create an effect in an injection context or provide an `Injector` explicitly.
- Choose an effect only for a real imperative side effect.
- Register cleanup for timers, subscriptions, and asynchronous work.
- Explain automatic destruction, `EffectRef`, and manual cleanup.
- Use `untracked()` to keep incidental signal reads out of an effect's dependencies.
- Choose `afterRenderEffect()` for DOM work that must happen after rendering.
- Avoid copying or propagating application state through effects.

## What an effect does

An effect is a reactive consumer. It runs at least once, tracks signal reads made synchronously during that run, and is scheduled again when a tracked dependency changes. Dependencies are dynamic: Angular uses the signals read during the most recent run, so a conditional branch can add or remove dependencies.

~~~ts
import { effect, signal } from '@angular/core';

const name = signal('Ada');
const greeting = effect(() => {
  console.log(`Hello, ${name()}`);
});

name.set('Grace'); // The effect is scheduled to run again.
~~~

`effect()` returns an `EffectRef`, not a value that represents the effect's result. Use effects for their interaction with an imperative API, not to calculate a value for a template.

### Scheduling depends on the effect context

Effects run as part of Angular's reactive scheduling, rather than immediately when a signal is written. Component effects run during Angular synchronization and can interact with component state. Root effects run as microtasks and are not tied to a component view's change-detection pass. Do not rely on an effect to run synchronously after `.set()`; if code needs the new value immediately, read the signal directly.

## Good reasons to use an effect

Effects are appropriate when signal state needs to cross into a non-reactive system, for example:

- Save a preference to `localStorage` or another imperative storage API.
- Send analytics or diagnostic output when a relevant value changes.
- Update a chart, canvas, or other third-party widget.
- Start an imperative timer, subscription, or request whose lifetime follows signal state.

~~~ts
import { effect, signal } from '@angular/core';

const selectedTheme = signal<'light' | 'dark'>('light');

effect(() => {
  // This is a browser API, so guard it when the application also renders on a server.
  if (typeof window !== 'undefined') {
    window.localStorage.setItem('theme', selectedTheme());
  }
});
~~~

In a real application, consider storage access failures such as browser privacy settings or quota limits. Keep browser-only work out of server rendering, and avoid making storage the only source of truth for important application data.

## Prefer derivation APIs for state

Do not use an effect to copy one signal into another just to derive a value:

~~~ts
// Avoid: creates a second source of truth and an extra scheduled update.
effect(() => {
  fullName.set(`${firstName()} ${lastName()}`);
});
~~~

Use a computed signal for read-only derived data:

~~~ts
const fullName = computed(() => `${firstName()} ${lastName()}`);
~~~

If the value follows source state but can also be changed by the user, use `linkedSignal()` where appropriate. These choices keep dependencies explicit and prevent stale state, circular updates, and unnecessary change-detection work.

## Create effects in an injection context

By default, `effect()` needs an injection context. A component, directive, or service constructor and a DI-created class's field initializer are common places to create one:

~~~ts
import { Component, effect, signal } from '@angular/core';

@Component({
  selector: 'app-status',
  template: `<p>{{ status() }}</p>`,
})
export class StatusComponent {
  readonly status = signal('Ready');

  constructor() {
    effect(() => {
      console.log('Status changed:', this.status());
    });
  }
}
~~~

When an effect needs to be created later from a method, capture an `Injector` in an injection context and pass it in the options:

~~~ts
import { Component, effect, inject, Injector, signal } from '@angular/core';

@Component({
  selector: 'app-audit-log',
  template: '',
})
export class AuditLogComponent {
  private readonly injector = inject(Injector);
  readonly message = signal('Ready');

  startLogging(): void {
    effect(
      () => console.log(this.message()),
      { injector: this.injector },
    );
  }
}
~~~

Calling `inject()` itself must happen in an injection context, such as a field initializer. Avoid starting the same effect repeatedly from a method unless multiple independent effects are truly intended; store and destroy an `EffectRef` when the effect needs a controlled lifetime.

## Clean up work started by an effect

An effect callback can receive `onCleanup`. Register cleanup immediately after starting work that must not outlive that run. Angular calls the cleanup before the effect runs again and when the effect is destroyed.

~~~ts
import { effect, signal } from '@angular/core';

const currentUser = signal('Ada');

effect((onCleanup) => {
  const user = currentUser();
  const timer = window.setTimeout(() => {
    console.log(`Still viewing ${user}`);
  }, 1000);

  onCleanup(() => window.clearTimeout(timer));
});
~~~

When `currentUser` changes before the second passes, Angular clears the first timer before scheduling the next effect run. Cleanup should undo the resources created by that specific run: cancel its timer, unsubscribe from its stream, remove its listener, or abort its request.

### Cancel stale asynchronous requests

Use an `AbortController` so that a request started for an old query can be cancelled when the query changes or its owner is destroyed:

~~~ts
import { effect, signal } from '@angular/core';

const query = signal('angular');

effect((onCleanup) => {
  const controller = new AbortController();
  const currentQuery = query();

  onCleanup(() => controller.abort());

  void fetch(`/api/search?q=${encodeURIComponent(currentQuery)}`, {
    signal: controller.signal,
  })
    .then((response) => {
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return response.json();
    })
    .then((results) => console.log(results))
    .catch((error: unknown) => {
      if (error instanceof DOMException && error.name === 'AbortError') return;
      console.error('Search failed', error);
    });
});
~~~

The query signal is read before starting the asynchronous work, so it is tracked. Reactive context only exists while the effect callback is executing synchronously. A signal read after an `await` or inside a later promise callback does not become a dependency of that effect. Capture the required signal values before the asynchronous boundary.

## Lifetime and manual destruction

Angular normally ties an effect's lifetime to its owner: a component- or directive-scoped effect is destroyed with that owner; a root effect lasts for the application lifetime. The returned `EffectRef` has a `destroy()` method for an explicit earlier end:

~~~ts
const loggingEffect = effect(() => {
  console.log('Count:', count());
});

// Call when this independently managed activity is no longer needed.
loggingEffect.destroy();
~~~

With `{ manualCleanup: true }`, the effect is not automatically registered for destruction with the current owner. You must retain its `EffectRef` and call `.destroy()` yourself. Prefer automatic cleanup unless the longer lifetime is intentional and you can guarantee disposal.

## Control incidental reads with `untracked()`

Every signal read made synchronously by an effect is normally tracked. If a helper called by the effect reads other signals only incidentally, those reads can cause unwanted reruns. Wrap that helper call in `untracked()` when those reads should not be dependencies:

~~~ts
import { effect, untracked } from '@angular/core';

effect(() => {
  const user = currentUser(); // The intended dependency.

  untracked(() => {
    analytics.logUser(user); // Signal reads inside this call are not tracked here.
  });
});
~~~

Use `untracked()` narrowly. If a value should cause the effect to run, read it normally and make the dependency explicit.

## DOM work belongs after rendering

A regular `effect()` can run before Angular has updated the DOM for that synchronization. Do not use it to query an element that the current render is about to create or to measure freshly rendered layout. Prefer template bindings and Angular view queries first. For imperative DOM integrations that really must run after Angular commits a render, use `afterRenderEffect()`:

~~~ts
import { afterRenderEffect, Component, viewChild, ElementRef } from '@angular/core';

@Component({
  selector: 'app-chart',
  template: `<canvas #canvas></canvas>`,
})
export class ChartComponent {
  readonly canvas = viewChild.required<ElementRef<HTMLCanvasElement>>('canvas');

  constructor() {
    afterRenderEffect({
      write: () => {
        // Update a third-party canvas/chart library here after Angular renders.
        const canvasElement = this.canvas().nativeElement;
        console.log('Canvas is available:', canvasElement);
      },
    });
  }
}
~~~

`afterRenderEffect()` is browser-only, and a component may not yet be hydrated when it runs. Use caution with direct DOM access. Its phases are `earlyRead`, `write`, `mixedReadWrite`, and `read`; prefer separating reads and writes into their named phases, because mixed layout reads and writes can trigger extra reflows. For resize, intersection, or mutation observation, browser observer APIs may be a better fit than repeatedly checking the DOM.

## Common mistakes

- **Deriving template state in an effect:** use `computed()` or `linkedSignal()` instead.
- **Expecting immediate execution after `.set()`:** effects are scheduled; read the signal directly when synchronous access is needed.
- **Forgetting cleanup:** timers, subscriptions, listeners, and requests can overlap or outlive their owner.
- **Reading signals only after an asynchronous boundary:** those reads are not tracked by the effect.
- **Creating an effect from an ordinary method without an injector:** pass an injector explicitly or create it in an injection context.
- **Creating repeated effects accidentally:** keep an `EffectRef` when you need to control the effect and destroy it.
- **Using a regular effect for freshly rendered DOM:** use Angular's template/query APIs or `afterRenderEffect()` for the imperative cases that need post-render timing.
- **Using `untracked()` to hide real dependencies:** only exclude incidental reads.

## Key takeaways

- An effect is for imperative side effects that synchronize signals with non-reactive APIs.
- Angular tracks synchronous signal reads dynamically and schedules reruns; effects are not synchronous derivations.
- Use `computed()` and `linkedSignal()` to model state relationships.
- Register `onCleanup` for resources created by a run, and rely on automatic destruction unless you deliberately manage an `EffectRef`.
- Use `untracked()` only for reads that should not trigger reruns.
- For direct DOM work, prefer normal Angular APIs and use `afterRenderEffect()` only when post-render timing is required.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="039-linkedsignal-and-dependent-writable-state.md">039 - 39. `linkedSignal` and dependent writable state</a></td><td><a href="../qnas/040-effects-cleanup-and-when-not-to-use-effects.md">Q&amp;A</a></td><td><a href="../examples/040-effects-cleanup-and-when-not-to-use-effects.md">Example</a></td><td><a href="041-signal-equality-dynamic-dependencies-and-untracked-reads.md">041 - 41. Signal equality, dynamic dependencies, and untracked reads</a></td>
</tr>
  </tbody>
</table>
