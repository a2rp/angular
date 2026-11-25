# Injection contexts and resource lifetimes

Angular's **inject()** function reads a dependency from the injector that is active at the current point in synchronous execution. That active point is an **injection context**. It exists only while Angular is constructing an object or running a supported factory or callback; it does not follow a component into every method or asynchronous callback.

A separate question is **how long the work should live**. Components, directives, routes, and services can have different destruction scopes. Use the right injector scope and tie subscriptions, timers, effects, requests, and external resources to that scope so they stop when their owner is destroyed.

This lesson explains where injection is allowed, how to deliberately create an injection context, how **DestroyRef** exposes the active lifetime, and how Angular cleans up common reactive work.

## Learning outcomes

After this lesson, you should be able to:

- Identify common places where Angular provides an injection context.
- Explain why calling **inject()** in a later method, lifecycle hook, or after **await** fails.
- Use **runInInjectionContext()** when a callback needs to call **inject()** synchronously.
- Use **assertInInjectionContext()** to make a custom injection helper fail clearly.
- Explain how provider scope determines the lifetime represented by **DestroyRef**.
- Register teardown work with **DestroyRef.onDestroy()**.
- Clean up RxJS subscriptions, signal effects, and asynchronous reads at the correct scope.
- Choose between an injector lookup, a captured dependency, and a new injection context.

## Injection context is a synchronous capability

An injector maps runtime tokens to values. **inject(Token)** uses the injector active in the current Angular execution context to look up a value. Angular creates that context for specific synchronous work.

Typical injection contexts include:

- A constructor or field initializer of a component, directive, or service created by Angular.
- A provider factory configured with **useFactory**.
- An **InjectionToken** factory.
- Functional APIs Angular invokes in an injection context, such as route guards and resolvers.
- A callback executed synchronously by **runInInjectionContext()**.

~~~ts
import { Component, inject } from '@angular/core';
import { Router } from '@angular/router';
import { AccountService } from './account.service';

@Component({
  selector: 'app-account-toolbar',
  standalone: true,
  template: '<button type="button">Open account</button>',
})
export class AccountToolbar {
  private readonly router = inject(Router);
  private readonly accounts = inject(AccountService);

  openAccount(id: string): void {
    this.accounts.recordNavigation(id);
    void this.router.navigate(['/accounts', id]);
  }
}
~~~

The calls to **inject()** happen in field initializers while Angular creates the component. The later **openAccount()** method uses the captured dependencies. It does not call **inject()** again.

### Places where inject() is not automatically available

The injection context does not remain active for the whole lifetime of an object. These are not normally injection contexts:

- An event handler called after the component was constructed.
- **ngOnInit()**, **ngOnDestroy()**, or another lifecycle hook.
- A timer, Promise callback, RxJS subscription callback, or ordinary function called later.
- Code after an **await** in an asynchronous function.
- A manually created class that Angular did not construct.

This fails because **ngOnInit()** runs after construction:

~~~ts
export class AccountPage {
  ngOnInit(): void {
    // NG0203: this lifecycle hook is not an injection context.
    const accounts = inject(AccountService);
  }
}
~~~

Capture the dependency during construction instead:

~~~ts
export class AccountPage {
  private readonly accounts = inject(AccountService);

  ngOnInit(): void {
    this.accounts.load();
  }
}
~~~

A common asynchronous mistake is to assume the context survives an **await**:

~~~ts
async function loadAccount(): Promise<void> {
  await Promise.resolve();

  // Not valid: execution has resumed after the synchronous context ended.
  const accounts = inject(AccountService);
}
~~~

Capture the dependency before asynchronous work, or pass it in as an ordinary argument:

~~~ts
async function loadAccount(accounts: AccountService): Promise<void> {
  await accounts.load();
}
~~~

## runInInjectionContext()

Use **runInInjectionContext(injector, callback)** when an existing callback must synchronously use **inject()**. The function temporarily makes the given injector active while it calls the callback.

~~~ts
import {
  Injector,
  inject,
  runInInjectionContext,
} from '@angular/core';
import { AccountService } from './account.service';

export class AccountPage {
  private readonly injector = inject(Injector);

  refreshFromCallback(): void {
    runInInjectionContext(this.injector, () => {
      const accounts = inject(AccountService);
      accounts.refresh();
    });
  }
}
~~~

The callback can return a value:

~~~ts
const accountCount = runInInjectionContext(this.injector, () => {
  return inject(AccountService).count();
});
~~~

The context is synchronous. If a callback returns a Promise, **inject()** is only valid before the callback reaches an asynchronous boundary. Capture dependencies inside the callback and let those objects perform later asynchronous work.

~~~ts
const request = runInInjectionContext(this.injector, () => {
  const accounts = inject(AccountService);
  return accounts.load();
});

await request;
~~~

Do not put **inject()** after an **await**, even if the async function was started inside **runInInjectionContext()**.

### Choose the smallest tool that fits

- **Capture a dependency** in a field or constructor when the class owns the work. This is usually clearest.
- Use **injector.get(Token)** when code needs a deferred lookup from a saved injector and does not need to call other injection-aware APIs.
- Use **runInInjectionContext()** when an external function must call **inject()** synchronously or needs a context-sensitive API.
- Pass a dependency as an ordinary parameter when that makes the function simpler to understand and test.

Creating a context does not extend the lifetime of the injector or keep a component alive. It only selects an injector for synchronous lookups.

## Assert custom helpers are called in context

A reusable helper that calls **inject()** can use **assertInInjectionContext()**. Passing the helper's own function reference gives a more useful error if a caller invokes it from an event handler or another invalid location.

~~~ts
import {
  ElementRef,
  assertInInjectionContext,
  inject,
} from '@angular/core';

export function injectHostElement<T extends Element>(): T {
  assertInInjectionContext(injectHostElement);
  return inject(ElementRef).nativeElement as T;
}
~~~

Use the helper during construction:

~~~ts
export class FocusRing {
  private readonly host = injectHostElement<HTMLElement>();

  focus(): void {
    this.host.focus();
  }
}
~~~

A call from **focus()** would be too late if the helper itself tried to inject. Here, the method uses the element captured during construction.

Assertions clarify the helper's contract; they do not create a context or make a later call valid.

## Injection context and resource lifetime are different ideas

An **injection context** answers: “Which injector can this synchronous code use right now?”

A **resource lifetime** answers: “When should this object, subscription, callback, effect, or request stop?”

Angular often creates the two together, but they are not interchangeable:

- A field initializer has an injection context while the class is being created.
- A component's **DestroyRef** represents that component's destruction scope.
- A root service's **DestroyRef** generally represents the root injector's lifetime, which lasts for the application.
- A service provided by a component or route is tied to that provider scope and can be destroyed with it.
- A callback can temporarily run in an injection context without changing the owner or extending its lifetime.

Choose provider scope deliberately. A component-scoped service is useful for state or work that should be isolated to one component subtree. A root-provided service is appropriate for shared application services, but work started by it should not accidentally be expected to stop when one page closes.

## DestroyRef

**DestroyRef** lets code register cleanup for the lifetime represented by the injector where it was obtained. Inject it in a component or directive to register work for that instance's destruction.

~~~ts
import { DestroyRef, inject } from '@angular/core';

export class ResizeAwarePanel {
  private readonly destroyRef = inject(DestroyRef);

  constructor() {
    const observer = new ResizeObserver((entries) => {
      console.log('Panel resize entries:', entries.length);
    });

    observer.observe(document.documentElement);

    this.destroyRef.onDestroy(() => {
      observer.disconnect();
    });
  }
}
~~~

**onDestroy(callback)** returns a function that unregisters the callback. Use that return value if you cancel the cleanup registration before destruction:

~~~ts
const unregister = this.destroyRef.onDestroy(() => {
  console.log('Panel destroyed');
});

// Later, if cleanup is no longer needed:
unregister();
~~~

The **destroyed** property can guard delayed work that cannot be cancelled:

~~~ts
setTimeout(() => {
  if (this.destroyRef.destroyed) {
    return;
  }

  this.status.set('Ready');
}, 500);
~~~

Prefer cancelling work when possible. A guard is useful when a callback may still arrive after its owner has gone away.

### DestroyRef scope follows where it is injected

The same API can represent different scopes:

~~~ts
@Injectable({ providedIn: 'root' })
export class AppCache {
  private readonly destroyRef = inject(DestroyRef);

  constructor() {
    this.destroyRef.onDestroy(() => {
      console.info('The root injector is being destroyed');
    });
  }
}
~~~

The callback above runs when the root injector is destroyed, typically when the application is torn down. It does not run when an individual page component disappears.

If the service is instead provided in a component's **providers** array, its instance and destruction callback belong to that component's injector scope. Providing a service in a component creates a separate instance for that component subtree; it is not the same lifetime as a root singleton.

## Clean up RxJS subscriptions

A manual subscription should end when its owner is destroyed. Angular's **takeUntilDestroyed()** operator completes the stream when the associated **DestroyRef** is destroyed.

~~~ts
import { DestroyRef, inject } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { interval } from 'rxjs';

export class PollingPanel {
  private readonly destroyRef = inject(DestroyRef);

  constructor() {
    interval(1000)
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe((seconds) => {
        console.log('Elapsed seconds:', seconds);
      });
  }
}
~~~

Passing **DestroyRef** explicitly is useful when the subscription is created from a method that runs later:

~~~ts
startWatching(): void {
  this.notifications.events$
    .pipe(takeUntilDestroyed(this.destroyRef))
    .subscribe((event) => this.show(event));
}
~~~

Without an explicit argument, **takeUntilDestroyed()** obtains the current **DestroyRef** through **inject()**. That form is convenient in a constructor or another injection context. Do not call it with no arguments from a click handler or an arbitrary callback.

Avoid hand-written **Subject<void>** plus **takeUntil()** patterns for ordinary component teardown when **takeUntilDestroyed()** fits. Manual patterns require more state and can be easy to forget to complete.

### toSignal() manages its subscription

**toSignal(observable, options)** subscribes to an Observable and exposes its latest value as a signal. By default, Angular unsubscribes when the injection context used to create the signal is destroyed.

~~~ts
import { inject } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';

export class SearchStatus {
  private readonly search = inject(SearchService);

  readonly results = toSignal(this.search.results$, {
    initialValue: [],
  });
}
~~~

Create the signal once during construction and reuse it. Calling **toSignal()** repeatedly creates repeated subscriptions. If it is created without an injection context, provide an **Injector** in the options. **manualCleanup** opts out of automatic destruction cleanup; use it only when the Observable's own completion is the intended stopping condition.

## Effect cleanup

An **effect()** created in an Angular injection context is automatically destroyed with that context's **DestroyRef**. Its **onCleanup** callback runs before the effect reruns and when the effect is destroyed.

Use cleanup to cancel work started by an effect:

~~~ts
import { effect, signal } from '@angular/core';

export class DelayedPreview {
  readonly query = signal('');

  constructor() {
    effect((onCleanup) => {
      const query = this.query();
      const timer = setTimeout(() => {
        console.log('Preview query:', query);
      }, 300);

      onCleanup(() => {
        clearTimeout(timer);
      });
    });
  }
}
~~~

When **query** changes before the timer fires, Angular runs the cleanup before scheduling the next effect run. When the owner is destroyed, Angular also runs cleanup.

Effects are not a replacement for **computed()** when deriving values. Use an effect for an external side effect, and register cleanup for each operation it starts. An effect created with **manualCleanup** must be destroyed manually through its **EffectRef**.

## Resource cleanup for asynchronous reads

Angular's **resource()** API models asynchronous reads as a value with reactive status. It accepts a reactive **params** function and a **loader**. Angular supplies an **AbortSignal** to the loader and aborts an in-progress load when its parameters change or the resource is destroyed.

~~~ts
import { resource, signal } from '@angular/core';

interface User {
  id: string;
  name: string;
}

export class UserPanel {
  readonly userId = signal('user-1');

  readonly user = resource({
    params: () => ({ id: this.userId() }),
    loader: async ({ params, abortSignal }): Promise<User> => {
      const response = await fetch(
        '/api/users/' + encodeURIComponent(params.id),
        { signal: abortSignal },
      );

      if (!response.ok) {
        throw new Error('Could not load user');
      }

      return response.json() as Promise<User>;
    },
  });
}
~~~

The fetch request receives the signal, so the browser can cancel it when the resource no longer needs the result. Merely creating an **AbortController** or destroying a component does not cancel an unrelated request; the operation must use the signal.

Resources are intended for reads. Do not use a cancellable resource loader for a mutation such as submitting a payment or saving a record: changing a parameter or destroying the resource could abort the operation.

The **resource()** API is stable starting in Angular 22. For an application on an earlier Angular version, check that version's documentation and use an available pattern such as **HttpClient** with **takeUntilDestroyed()** if resources are not available.

## Common lifetime patterns

| Work | Typical owner | Cleanup approach |
| --- | --- | --- |
| DOM event listener or observer | Component or directive | Remove listener or disconnect in **DestroyRef.onDestroy()** |
| RxJS subscription started by a component | Component or directive | **takeUntilDestroyed(destroyRef)** |
| Observable exposed as a signal | Component or service that calls **toSignal()** | Default injector-scoped cleanup |
| Timer started by an effect | Effect execution | Clear it in the effect's **onCleanup** callback |
| Reactive asynchronous read | **resource()** instance | Use the provided **AbortSignal** |
| Shared application cache | Root service | Root injector lifetime, or explicit eviction policy |
| Feature-local state | Route or component provider | Route/component injector lifetime |

Before adding cleanup, identify who owns the work. A root service should not silently create a subscription that is expected to stop when a child component disappears. Instead, expose an Observable and let the component own its subscription, or pass the intended **DestroyRef** to the helper that starts the work.

## Troubleshooting NG0203

**NG0203: inject() must be called from an injection context** usually means an injection-only function ran too late or outside Angular's synchronous setup code.

Check:

1. Is **inject()** in a field initializer, constructor, provider factory, token factory, or supported functional callback?
2. Is a lifecycle hook or event handler calling a helper that uses **inject()**?
3. Did the code cross an **await**, timer, Promise callback, or subscription boundary?
4. Could the dependency be captured during construction and used later?
5. If external code must call **inject()**, can it be invoked synchronously inside **runInInjectionContext()**?
6. Does a custom **injectX()** helper call **assertInInjectionContext()** to explain its requirement?

**runInInjectionContext()** is not a general fix for lifetime bugs. It does not cancel work, keep an injector alive, or make an asynchronous continuation an injection context.

## Common mistakes

- **Calling inject() in ngOnInit or an event handler:** capture the dependency during construction.
- **Calling inject() after await:** inject synchronously first, then use the captured object asynchronously.
- **Treating injection context as a lifetime:** context controls lookup; injector scope controls ownership and destruction.
- **Using runInInjectionContext() around an async callback and injecting later:** the context ends when the synchronous callback returns.
- **Expecting a root DestroyRef to run when a page closes:** root services are tied to the root injector.
- **Starting a subscription without assigning an owner:** use **takeUntilDestroyed()** or another explicit teardown.
- **Calling toSignal() repeatedly for one Observable:** create one signal and reuse it.
- **Ignoring effect cleanup:** cancel timers, observers, requests, or other work before reruns and destruction.
- **Assuming component destruction automatically aborts fetch:** connect the operation to an **AbortSignal** or cancel it explicitly.
- **Using resource() for a write:** resources can cancel reads; model mutations with an operation flow that matches their semantics.
- **Keeping a long-lived manual effect without destroying its EffectRef:** make manual ownership explicit.

## Further reading

- [Injection context guide](https://angular.dev/guide/di/dependency-injection-context)
- [inject() API](https://angular.dev/api/core/inject)
- [runInInjectionContext() API](https://angular.dev/api/core/runInInjectionContext)
- [assertInInjectionContext() API](https://angular.dev/api/core/assertInInjectionContext)
- [DestroyRef API](https://angular.dev/api/core/DestroyRef)
- [Component lifecycle and DestroyRef](https://angular.dev/guide/components/lifecycle)
- [takeUntilDestroyed() API](https://angular.dev/api/core/rxjs-interop/takeUntilDestroyed)
- [toSignal() API](https://angular.dev/api/core/rxjs-interop/toSignal)
- [Effect cleanup](https://angular.dev/guide/signals/effect)
- [Resource API](https://angular.dev/api/core/resource)
- [RxJS resource lifetime guide](https://angular.dev/ecosystem/rxjs-interop/take-until-destroyed)

## Continue learning

- Previous: [Provider recipes, multi providers, and provider functions](055-provider-recipes-multi-providers-and-provider-functions.md)
- Q&A: [Injection contexts and resource lifetimes](../qnas/056-injection-contexts-and-resource-lifetimes.md)
- Example: [Injection contexts and resource lifetimes](../examples/056-injection-contexts-and-resource-lifetimes.md)
- Next: [RxJS Observables, subscriptions, and stream basics](057-rxjs-observables-subscriptions-and-stream-basics.md)
