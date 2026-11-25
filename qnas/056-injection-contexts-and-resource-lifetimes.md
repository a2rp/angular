# Q&A: Injection contexts and resource lifetimes

These answers explain when Angular can resolve **inject()**, how **runInInjectionContext()** changes the active injector for a synchronous callback, and how to tie work to the correct destruction scope. See the [lesson](../lessons/056-injection-contexts-and-resource-lifetimes.md) for the full guide and the [worked example](../examples/056-injection-contexts-and-resource-lifetimes.md) for a standalone cleanup demo.

## 1. What is an injection context?

An injection context is a synchronous execution frame where Angular has made an injector available to **inject()**. Angular uses it while creating DI-managed classes and while running supported factories or callbacks.

It answers which injector can resolve a token at that moment. It does not mean that every later method on the same object can call **inject()**.

## 2. Where can I call inject()?

Common valid locations include:

- A field initializer or constructor of a component, directive, or service created by Angular.
- A **useFactory** provider factory or an **InjectionToken** factory.
- A functional route guard or resolver invoked by Angular.
- A synchronous callback run with **runInInjectionContext()**.

Functional interceptors, initializers, and other Angular functional APIs can also run in an injection context when Angular invokes them.

## 3. Can I call inject() in ngOnInit()?

No. Angular has finished the constructor and field-initialization phase before calling **ngOnInit()**. Capture the service during construction, then use it in the hook:

~~~ts
export class ProfilePage {
  private readonly profiles = inject(ProfileService);

  ngOnInit(): void {
    this.profiles.load();
  }
}
~~~

## 4. Can an event handler call inject()?

Not by default. An event handler runs later, outside the construction context. Capture the service in a field initializer and call its methods from the handler.

If a reusable callback specifically needs to invoke **inject()**, call it synchronously through **runInInjectionContext()** with an injector.

## 5. Why does inject() stop working after await?

Angular's injection context is synchronous. An **await** resumes execution in a later microtask after the context that started the function has ended.

Inject before the asynchronous boundary and keep using the captured reference:

~~~ts
const api = inject(ProfileService);
const profile = await api.load();
~~~

## 6. Does runInInjectionContext() make an async function safe for injection?

Only during the synchronous part of its callback. This is valid because the dependency is retrieved before the Promise is returned:

~~~ts
const request = runInInjectionContext(injector, () => {
  const api = inject(ProfileService);
  return api.load();
});
~~~

Calling **inject()** after an **await** inside that callback is still invalid.

## 7. What does runInInjectionContext() take?

It takes an **Injector** and a synchronous function. While that function executes, calls to **inject()** resolve against the supplied injector. It returns the function's return value.

Use the injector whose provider scope should resolve the dependencies. Creating this temporary context does not extend that injector's lifetime.

## 8. Should I use runInInjectionContext() for every method?

No. For an Angular-managed class, capture dependencies in field initializers or the constructor. Use **runInInjectionContext()** when external or reusable code must call injection-aware APIs synchronously.

For a simple delayed lookup, **injector.get(Token)** may be easier. Passing a dependency as an ordinary function argument is often simplest for plain utility code.

## 9. What is the difference between Injector.get() and inject()?

**inject()** reads from the injector active in the current injection context. **injector.get(Token)** explicitly asks a saved injector for a value and does not require an active injection context.

Use the explicit injector when deferred lookup is necessary. Prefer capturing a dependency early when the owning class knows what it needs.

## 10. What does assertInInjectionContext() do?

It checks that a reusable helper is currently being called inside an injection context. If not, it throws a clear error that points to the helper.

~~~ts
export function injectCurrentUser(): UserService {
  assertInInjectionContext(injectCurrentUser);
  return inject(UserService);
}
~~~

The assertion does not create a context; the caller must still call this helper from a valid location.

## 11. What is NG0203?

**NG0203** means **inject() must be called from an injection context**. Frequent causes are calling it in a lifecycle hook, an event handler, a timer or Promise callback, a manually created class, or after an **await**.

Capture dependencies during construction, use an explicit **Injector** for deferred lookups, or synchronously use **runInInjectionContext()** when the callback must call **inject()**.

## 12. What is a resource lifetime?

A resource lifetime is the period during which a component, service, subscription, timer, effect, observer, or request should remain active.

The owner might be a component instance, a route injector, or the application root. Pick that owner before choosing cleanup code.

## 13. Is injection context the same as lifetime?

No. Injection context controls which injector can be read synchronously. Lifetime controls how long the injector-owned object or work should remain active.

**runInInjectionContext()** creates a temporary lookup context; it does not keep a component or injector alive.

## 14. What is DestroyRef?

**DestroyRef** is Angular's handle for registering destruction callbacks in the current scope. It can also report whether that scope has already been destroyed.

~~~ts
private readonly destroyRef = inject(DestroyRef);

constructor() {
  this.destroyRef.onDestroy(() => {
    this.observer.disconnect();
  });
}
~~~

The cleanup should release the resource started by that same owner.

## 15. What does DestroyRef.onDestroy() return?

It returns a function that unregisters the callback. Call that function only if the cleanup should no longer be registered before destruction.

~~~ts
const unregister = this.destroyRef.onDestroy(cleanup);

// If the resource was released early:
unregister();
~~~

## 16. Does every DestroyRef represent the same lifetime?

No. Its scope follows the injector where it is injected.

A component or directive's **DestroyRef** runs when that instance is destroyed. A root service's **DestroyRef** normally runs when the root injector is destroyed, typically when the application is torn down.

## 17. When is a component-scoped service destroyed?

A service provided in a component's **providers** array belongs to that component's injector scope. Angular creates a scoped instance for that component subtree and destroys it with that scope.

A service provided in root is shared by the application and is not destroyed just because one page component is removed.

## 18. Does navigating away always destroy a route-provided service?

Do not assume every route injector is destroyed on each navigation. Route providers belong to the route's environment injector, whose lifetime follows Angular's router and route configuration behavior. Check your route setup and test the lifecycle you rely on.

If work must stop with a particular component, own that work in the component or pass its **DestroyRef** explicitly.

## 19. How do I stop an RxJS subscription when a component is destroyed?

Use **takeUntilDestroyed()** from **@angular/core/rxjs-interop**:

~~~ts
source$
  .pipe(takeUntilDestroyed(this.destroyRef))
  .subscribe((value) => this.handle(value));
~~~

The operator completes the subscription when the supplied **DestroyRef** is destroyed.

## 20. Can takeUntilDestroyed() be used without an argument?

Yes, when called in an injection context, such as during construction. It injects the current **DestroyRef**.

From a method or click handler that runs later, pass a **DestroyRef** captured during construction. Otherwise the operator's internal injection lookup has no context.

## 21. What does toSignal() clean up?

**toSignal()** subscribes to an Observable and exposes its latest value as a signal. By default, it ties the subscription to the injection context or explicit injector used to create it, and unsubscribes when that scope is destroyed.

Create the signal once and reuse it; each call to **toSignal()** creates a subscription.

## 22. When should I pass an Injector to toSignal()?

Pass an **Injector** when **toSignal()** is created outside a normal injection context and should still use injector-scoped cleanup. Alternatively, use the Observable's own completion semantics with **manualCleanup** only when automatic teardown is not the intended behavior.

## 23. How does effect() cleanup work?

An effect can accept an **onCleanup** callback. Angular runs cleanup before the effect runs again and when its associated scope is destroyed.

~~~ts
effect((onCleanup) => {
  const timer = setTimeout(() => this.refresh(), 500);

  onCleanup(() => clearTimeout(timer));
});
~~~

Use this for timers, observers, subscriptions, or external work created by that effect run.

## 24. Is an effect automatically destroyed?

An effect created in an injection context is automatically associated with that context's cleanup scope. Component-context effects end with the component; effects created by a root service generally last until the root injector is destroyed.

An effect configured for manual cleanup must have its **EffectRef** destroyed explicitly.

## 25. How does resource() cancel a read?

A **resource()** loader receives an **AbortSignal**. Angular aborts an in-progress load when the resource's parameters change or when its owner is destroyed. Pass that signal to an API such as **fetch()** so the underlying request can respond to cancellation.

## 26. Does destroying a component cancel any fetch it started?

Not automatically for a plain **fetch()** call. Pass an **AbortSignal** to the request or register an **AbortController** with **DestroyRef**. A resource already supplies a signal to its loader.

Cancellation only reaches an operation that actually observes the signal.

## 27. Should I use resource() for mutations?

Usually no. Resources represent asynchronous reads and can cancel a load when parameters change or the owner is destroyed. A mutation such as saving a record or submitting a payment should use an explicit operation flow with semantics appropriate to that write.

## 28. Which Angular version includes stable resource()?

Angular documents **resource()** as stable starting in Angular 22. If supporting an earlier Angular version, check that version's API and use an appropriate alternative, such as **HttpClient** with **takeUntilDestroyed()**.

## 29. What is DestroyRef.destroyed useful for?

It lets delayed code check whether its owner has already been destroyed before updating state or touching a view.

Prefer cancelling the operation when possible. Use the flag when a callback may still arrive and cannot be cancelled reliably.

## 30. What if a service starts a subscription that should end with a component?

Make the owner explicit. Prefer returning an Observable and letting the component subscribe with **takeUntilDestroyed(componentDestroyRef)**. If a service function starts the work directly, accept a **DestroyRef** argument or return a cleanup handle.

Do not accidentally tie page-specific work to a root service's longer lifetime.

## 31. Does a context created for a callback keep the injector from being destroyed?

No. **runInInjectionContext()** temporarily selects an injector for synchronous lookups. It is not a retention or ownership mechanism and does not delay destruction.

Do not schedule asynchronous work that assumes the injector will still exist later. Capture the required dependency and give the work an explicit cancellation or cleanup owner.

## 32. What should I do when I get NG0203?

Check the exact call stack and ask:

1. Was **inject()** called in a field initializer, constructor, factory, or supported functional callback?
2. Did a lifecycle method, event handler, timer, subscription, or Promise call an injection-only helper?
3. Did the code cross an **await**?
4. Can the dependency be captured once and reused?
5. If a callback genuinely needs injection, can it run synchronously inside **runInInjectionContext()**?
6. Should a custom helper use **assertInInjectionContext()**?

Then check cleanup separately. Fixing NG0203 does not automatically fix a leaked subscription or a request with the wrong lifetime.

## Further reading

- [Injection context guide](https://angular.dev/guide/di/dependency-injection-context)
- [inject() API](https://angular.dev/api/core/inject)
- [runInInjectionContext() API](https://angular.dev/api/core/runInInjectionContext)
- [assertInInjectionContext() API](https://angular.dev/api/core/assertInInjectionContext)
- [DestroyRef API](https://angular.dev/api/core/DestroyRef)
- [takeUntilDestroyed() API](https://angular.dev/api/core/rxjs-interop/takeUntilDestroyed)
- [toSignal() API](https://angular.dev/api/core/rxjs-interop/toSignal)
- [Effect cleanup](https://angular.dev/guide/signals/effect)
- [Resource API](https://angular.dev/api/core/resource)

## Continue learning

- Lesson: [Injection contexts and resource lifetimes](../lessons/056-injection-contexts-and-resource-lifetimes.md)
- Example: [Injection contexts and resource lifetimes](../examples/056-injection-contexts-and-resource-lifetimes.md)
- Next lesson: [RxJS Observables, subscriptions, and stream basics](../lessons/057-rxjs-observables-subscriptions-and-stream-basics.md)
