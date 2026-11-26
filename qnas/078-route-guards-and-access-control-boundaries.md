# Route guards and access control boundaries: Q&A

## 1. What does an Angular route guard do?

A route guard gives the Angular Router a decision during navigation. Depending on the guard type, it can allow or cancel activation, decide whether a child can activate, decide whether the current component may be left, redirect, or skip a route configuration while matching.

## 2. Do route guards secure protected data?

No. A route guard runs in the user's browser, and browser code can be changed or bypassed. Enforce authentication and authorization in the server or API that owns the data. Use a guard to improve navigation and presentation, not as the only access control.

## 3. Which guard types should I know?

Angular provides four common functional guard types:

- `CanActivateFn` for activating a route.
- `CanActivateChildFn` for activating a child under a parent route.
- `CanDeactivateFn<T>` for leaving a route whose component has type `T`.
- `CanMatchFn` for deciding whether a route configuration is considered during matching.

## 4. When should I use `CanActivateFn`?

Use it when the destination route itself needs a navigation check, such as directing a signed-out visitor to sign in or routing a user without a required role to an explanatory page.

## 5. Does `CanActivateChildFn` protect the parent component too?

It protects the activation of child routes beneath the route where it is configured. If the parent component itself must be protected when activated, also add an appropriate `canActivate` guard to the parent route.

## 6. What is a good use for `CanDeactivateFn`?

Prevent accidental loss of work when leaving an editor or form with unsaved changes. The guard receives the component instance, so it can ask the component whether changes remain and return a boolean or an asynchronous confirmation result.

## 7. Is native `confirm()` appropriate for every application?

No. It is a convenient browser-only demonstration. A production interface may use an accessible application dialog whose promise or observable result is returned by the guard. Consider server rendering and non-browser environments before depending on browser globals such as `window`.

## 8. What does `CanMatchFn` decide?

It decides whether a route configuration can be used during route matching. It is useful for feature flags, conditional route loading, or selecting among routes that share a path.

## 9. What happens when `CanMatchFn` returns `false`?

Angular skips that route configuration and tries another matching route. It does not by itself cancel navigation in the same way as a `CanActivateFn` returning `false`. Add a fallback route at the same path when that is the intended behavior.

## 10. What may a guard return?

A guard may return `true`, `false`, a `UrlTree`, or a `RedirectCommand`. It may also return a `Promise` or `Observable` that resolves or emits one of those values.

## 11. How should a guard redirect?

Return a router destination, usually a `UrlTree`:

~~~ts
return inject(Router).createUrlTree(['/login']);
~~~

The Router cancels the attempted navigation and performs the redirect. Do not return `false` and separately call `router.navigate()`.

## 12. What does the Router do with an observable guard result?

It uses the first emitted value and unsubscribes. The observable must emit a guard result. If it completes without emitting, for example by returning `EMPTY`, the Router cannot decide and navigation fails.

## 13. What should an asynchronous guard do if its request fails?

Handle failure deliberately. Decide whether an unavailable authorization service should cancel, redirect to an error page, or produce another defined result. Avoid silently treating network failures as permission grants. The API still has to enforce access when a request reaches it.

## 14. Can I call `inject()` after an `await` inside an async guard?

No. `inject()` needs an Angular injection context, which is available while the guard function is running synchronously. Capture dependencies before awaiting:

~~~ts
export const accessGuard: CanActivateFn = async () => {
  const access = inject(AccessService);
  const router = inject(Router);

  const allowed = await access.check();
  return allowed ? true : router.createUrlTree(['/forbidden']);
};
~~~

## 15. If multiple guards are listed, do they run one at a time?

Do not assume that asynchronous guards wait for earlier asynchronous guards to finish. They can run concurrently. Their array order determines the priority of non-allowing results, but it does not make one guard a reliable setup step for another. Share data through a service or combine dependent checks.

## 16. Can a guard read route and navigation information?

Yes. For example, a `CanActivateFn` receives an `ActivatedRouteSnapshot` and a `RouterStateSnapshot`. The state contains the target URL, which can be useful as a return destination after sign-in. Validate that destination as an intended in-app route before navigating to it.

## 17. Should every route use a guard?

No. Add guards where a navigation decision is needed. Use a parent `canActivateChild` when a coherent set of nested pages shares a policy. Avoid duplicating the same check on every route unless each route has a distinct reason.

## 18. Can a hidden link replace a guard?

No. Hiding a link is only a presentation choice. A visitor may enter the URL directly or call the API. The router should make navigation understandable, and the server should authorize each protected operation.

## 19. Are guards the right place to fetch all page data?

Usually not. Use a resolver for data that should be ready before route activation, or load data in the page according to the app's loading experience. Keep guards focused on whether navigation may proceed. See [Resolvers, route data, and loading a page before activation](../lessons/079-resolvers-route-data-and-loading-a-page-before-activation.md).

## 20. What should I test around a guard?

Check its decision for allowed and denied states, its redirect destination, asynchronous success and failure, and any important route fallback. Also verify that protected API calls are denied server-side; a client-only route test cannot prove server authorization.

## 21. Does a guard stop someone from downloading the app bundle?

No. Client code is delivered to the browser. Do not put secrets or authoritative access decisions in a guard or assume that hiding a route keeps its code private.

## 22. Where can I read the official API guidance?

Start with the Angular route guards guide, then see the APIs for CanActivateFn, CanActivateChildFn, CanDeactivateFn, CanMatchFn, and RedirectCommand.

## Related material

- [Lesson: Route guards and access control boundaries](../lessons/078-route-guards-and-access-control-boundaries.md)
- [Example: Route guards and access control boundaries](../examples/078-route-guards-and-access-control-boundaries.md)
- [Authentication, authorization, secrets, and browser storage](../lessons/118-authentication-authorization-secrets-and-browser-storage.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="077-navigate-programmatically-and-handle-navigation-results.md">077 - Navigate programmatically and handle navigation results</a></td><td><a href="../lessons/078-route-guards-and-access-control-boundaries.md">Lesson</a></td><td><a href="../examples/078-route-guards-and-access-control-boundaries.md">Example</a></td><td><a href="079-resolvers-route-data-and-loading-a-page-before-activation.md">079 - Resolvers, route data, and loading a page before activation</a></td>
</tr>
  </tbody>
</table>
