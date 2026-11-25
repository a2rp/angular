# Route guards and access control boundaries

Angular route guards let the router ask application code whether a navigation should continue, redirect, or fall through to another route definition. They are useful for navigation policy and user experience: send a signed-out visitor to a sign-in screen, keep a user from accidentally leaving an unsaved editor, or select a route variant when a feature is enabled.

A guard runs in the browser. It is not a security boundary for private data. Users can change client-side JavaScript and call an API without using the Angular interface. The server must authenticate requests and authorize access to protected resources independently. See [Authentication, authorization, secrets, and browser storage](118-authentication-authorization-secrets-and-browser-storage.md) for the broader security model.

## Learning outcomes

After this lesson, you should be able to:

- Choose `CanActivateFn`, `CanActivateChildFn`, `CanDeactivateFn`, or `CanMatchFn` for a navigation decision.
- Return `true`, `false`, a `UrlTree`, a `RedirectCommand`, a promise, or an observable correctly.
- Redirect from a guard by returning a router result instead of starting a second navigation.
- Explain why `CanMatch` returning `false` tries another matching route.
- Protect unsaved work without treating a browser prompt as a security mechanism.
- Keep authorization checks on the server even when client routes are guarded.

## What a guard controls

A route guard participates in router navigation. Its result can influence whether Angular activates a route, leaves a route, or considers a route configuration to be a match. It controls what the application router displays; it does not control whether a person can reach a URL directly, inspect downloaded JavaScript, or call a backend endpoint.

Prefer a guard for a navigation rule that belongs at the route boundary. Keep ordinary rendering decisions in components and templates. A guard should make a clear decision from current application state, route information, or a service result. Avoid using it to perform unrelated writes, show repeated side effects, or hide sensitive data that an API still returns.

Angular recommends functional guards. The examples below use `inject()` to access services within the guard function.

## The four guard types

### `CanActivateFn`: may this route become active?

Use `canActivate` when the target route itself needs a check, commonly for a signed-in session or a role-based section.

~~~ts
import {inject} from '@angular/core';
import {CanActivateFn, Router} from '@angular/router';

export const signedInGuard: CanActivateFn = (_route, state) => {
  const session = inject(SessionService);
  const router = inject(Router);

  if (session.isSignedIn()) {
    return true;
  }

  return router.createUrlTree(['/login'], {
    queryParams: {returnUrl: state.url},
  });
};
~~~

The router supplies the target route snapshot and target router state. `state.url` is useful when the sign-in page should return to the requested page. Treat a return URL as untrusted input when consuming it; allow only intended in-app destinations.

### `CanActivateChildFn`: may a child route become active?

Place `canActivateChild` on a parent route to apply a policy to its descendant routes. This fits a section with several nested pages, such as administration screens. It is a check on child activation: add `canActivate` too if the parent route’s own component also needs protection.

~~~ts
import {CanActivateChildFn, Router} from '@angular/router';

export const adminChildGuard: CanActivateChildFn = () => {
  const session = inject(SessionService);
  const router = inject(Router);

  return session.hasRole('admin')
    ? true
    : router.createUrlTree(['/forbidden']);
};
~~~

When the policy applies to a whole child tree, keeping it on the parent avoids copying the same guard to every child. A child may still have additional checks.

### `CanDeactivateFn`: may the current route be left?

Use `canDeactivate` when leaving a component could discard user work. The guard receives the component instance, so the component can expose a focused method such as `hasUnsavedChanges()`.

~~~ts
import {CanDeactivateFn} from '@angular/router';

export interface UnsavedWork {
  hasUnsavedChanges(): boolean;
}

export const confirmDiscardGuard: CanDeactivateFn<UnsavedWork> = component => {
  if (!component.hasUnsavedChanges()) {
    return true;
  }

  return window.confirm('Discard your unsaved changes?');
};
~~~

This native dialog is a small browser-only teaching example. A production application may instead inject an accessible dialog service and return its `Promise<boolean>` or `Observable<boolean>`. Keep the guard focused on the leave decision, and make the save state reliable so the prompt does not appear after a successful save.

### `CanMatchFn`: should this route configuration match?

Use `canMatch` when route selection depends on a condition such as a feature flag. If it returns `false`, Angular skips that route configuration and tries later matching configurations. This makes it possible to configure a fallback at the same path.

~~~ts
import {CanMatchFn} from '@angular/router';

export const reportsFeatureGuard: CanMatchFn = () =>
  inject(FeatureFlagsService).isEnabled('reports');
~~~

`CanMatch` is not interchangeable with `CanActivate`. A `false` from `CanActivate` cancels activation. A `false` from `CanMatch` means “try another route configuration that matches.” If there is no fallback, the router continues matching other routes or reaches the not-found route.

## Guard results and redirects

A functional guard can return:

- `true` to allow the relevant navigation step.
- `false` to cancel it, except that `CanMatch` skips that route configuration and tries another match.
- A `UrlTree` or `RedirectCommand` to redirect.
- A `Promise` or `Observable` of one of these values.

For a redirect, create and return the destination:

~~~ts
return router.createUrlTree(['/login']);
~~~

Do not return `false` and then call `router.navigate()` as a second action. Returning the target lets the router cancel the current attempt and handle the redirect as part of its navigation flow.

For an observable result, the router uses its first emitted value and unsubscribes. The observable must emit a result; an observable that completes without a value, such as `EMPTY`, cannot make the guard decision and causes navigation to fail. Handle errors intentionally in the service or map them to a deliberate route result.

`inject()` must be called while the guard function is executing in Angular’s injection context. In an asynchronous guard, obtain injected services before awaiting:

~~~ts
export const remoteAccessGuard: CanActivateFn = async () => {
  const access = inject(AccessService);
  const router = inject(Router);

  const allowed = await access.canOpenWorkspace();

  return allowed ? true : router.createUrlTree(['/forbidden']);
};
~~~

Calling `inject()` after an `await` is outside the synchronous injection context. Capture dependencies at the top of the function.

## Applying multiple guards

A route can list several guards:

~~~ts
{
  path: 'admin',
  component: AdminPage,
  canActivate: [signedInGuard, adminGuard],
}
~~~

All checks must allow navigation for it to continue. Do not make an asynchronous guard depend on work that another guard in the array is expected to finish first: the router can start asynchronous guards concurrently. Put shared loading or decision logic in a service, or combine dependent checks into one guard. The order in the array determines which non-allowing result takes priority; it is not a promise that one asynchronous guard will finish before the next starts.

Use the narrowest placement that expresses the policy. A parent `canActivateChild` is helpful for a protected subtree; a route-level `canActivate` is better for one destination. Avoid attaching a guard everywhere if that duplicates policy and makes navigation harder to reason about.

## Route guard versus real authorization

A client guard improves navigation behavior, but it cannot grant or deny access to server data securely:

1. The browser receives the application code and can be modified by its user.
2. A visitor can bypass a hidden link or invoke a backend request directly.
3. The API must validate the session or credential and check the requested operation against the user’s permissions.
4. The API should return an appropriate denial when authorization fails. The Angular app can then show a useful error or sign-in route.

Do not place passwords, API secrets, or authoritative permission decisions in a guard or bundle. A client may keep display-oriented session state to choose a route, but the server remains authoritative. Review [Angular’s security guide](https://angular.dev/best-practices/security) and the [route guards guide](https://angular.dev/guide/routing/route-guards) alongside this lesson.

## Practical checklist

Before adding a guard, ask:

- Is this a navigation or unsaved-work decision, rather than an API security rule?
- Is the guard type appropriate for activation, child activation, deactivation, or route matching?
- Does a redirect return a `UrlTree` or `RedirectCommand`?
- If it is asynchronous, does it emit or resolve exactly a deliberate guard result, and are errors handled?
- Does `CanMatch` have the intended fallback behavior?
- Are injected dependencies obtained before any `await`?
- Does the backend independently enforce authentication and authorization?
- Can keyboard and screen-reader users complete the redirect, sign-in, or confirmation flow?

## Related material

- [Route parameters, snapshots, and reactive parameters](075-route-parameters-snapshots-and-reactive-parameters.md)
- [Navigate programmatically and handle navigation results](077-navigate-programmatically-and-handle-navigation-results.md)
- [Resolvers, route data, and loading a page before activation](079-resolvers-route-data-and-loading-a-page-before-activation.md)
- [Authentication, authorization, secrets, and browser storage](118-authentication-authorization-secrets-and-browser-storage.md)
- [Official Angular route guards guide](https://angular.dev/guide/routing/route-guards)
- [CanActivateFn API](https://angular.dev/api/router/CanActivateFn)
- [CanMatchFn API](https://angular.dev/api/router/CanMatchFn)
- [RedirectCommand API](https://angular.dev/api/router/RedirectCommand)
