# Navigate programmatically and handle navigation results: Q&A

## 1. What is programmatic navigation?

Programmatic navigation is a route change initiated by application code through Angular's `Router` service. It is useful after an operation such as saving a form or when a workflow decides which view should come next.

For an ordinary user-selected destination, prefer an anchor with `RouterLink`.

## 2. How do I inject the router?

Use Angular dependency injection in a component or service:

~~~ts
private readonly router = inject(Router);
~~~

Import `Router` from `@angular/router`. The application must have configured the router, commonly with `provideRouter(routes)`.

## 3. What is the difference between `navigate()` and `navigateByUrl()`?

- `navigate(commands, extras)` builds a target from path segments and URL creation options.
- `navigateByUrl(url, extras)` navigates to a complete absolute URL path or a `UrlTree`.

Use `navigate()` when building a route from dynamic segments or when passing query parameters and fragments as options. Use `navigateByUrl()` when the complete URL is already known.

## 4. How do I navigate to a route with a dynamic ID?

Pass the route segments as an array:

~~~ts
await this.router.navigate(['/orders', orderId, 'confirmation']);
~~~

If `orderId` is `'42'`, Angular targets `/orders/42/confirmation`, assuming the route is configured.

## 5. How do relative navigation commands work?

Pass `relativeTo` with the activated route when a command should resolve from that route:

~~~ts
await this.router.navigate(['edit'], {
  relativeTo: this.route,
});
~~~

Without `relativeTo`, commands start at the application root. A leading slash makes a root-relative destination explicit.

## 6. Can `navigate()` also set query parameters or a fragment?

Yes. Pass URL creation options:

~~~ts
await this.router.navigate(['/search'], {
  queryParams: {term: 'angular'},
  fragment: 'results',
});
~~~

## 7. What does `navigateByUrl()` accept?

It accepts a complete absolute route path string or a `UrlTree`:

~~~ts
await this.router.navigateByUrl('/search?term=angular#results');
~~~

It does not apply a path delta to the current URL. When passing a `UrlTree`, create the full target tree first.

## 8. What does a successful navigation promise resolve to?

`Router.navigate()` and `Router.navigateByUrl()` return `Promise<boolean>`. The promise resolves to `true` when the requested navigation succeeds.

## 9. What does a resolved `false` mean?

It means the requested navigation did not complete successfully. Possible reasons include a guard cancellation, a navigation being superseded, or a same-URL request being skipped. The exact reason can be investigated through router events.

A redirect is a special case: the original navigation can be canceled as part of following a redirect, while the redirected navigation succeeds. Do not turn every cancellation event into an error message for the visitor.

## 10. Can the navigation promise reject?

Yes. By default, an unexpected navigation error rejects the promise. Handle it with `try/catch` when awaiting navigation:

~~~ts
try {
  const succeeded = await this.router.navigate(['/next']);

  if (!succeeded) {
    this.message = 'The page was not opened.';
  }
} catch (error: unknown) {
  this.message = 'Navigation encountered an unexpected error.';
  console.error(error);
}
~~~

A resolved `false` is not the same as a rejected promise.

## 11. When should I await the navigation promise?

Await it when subsequent code depends on the outcome, such as updating a status message, closing a workflow only after success, or deciding what to do when navigation did not complete.

If no caller needs the result, the promise still needs a decision about errors.

## 12. Does `void this.router.navigate(...)` catch errors?

No. `void` discards the TypeScript expression value; it does not handle a rejected promise. If deliberately not awaiting the boolean result, attach a rejection handler:

~~~ts
void this.router.navigate(['/help']).catch((error: unknown) => {
  console.error('Navigation failed', error);
});
~~~

When navigation affects the UI, awaiting it and handling both outcomes is usually clearer.

## 13. What is `NavigationCancel`?

It is a router event emitted when a navigation is canceled. A guard returning `false` can cancel a navigation; redirects also cancel the original navigation while the router proceeds to another URL. Use the event's stable code for diagnostics rather than relying on its human-readable reason string.

## 14. What is `NavigationSkipped`?

It is an event emitted when the router skips a navigation request, such as when the requested URL is already current and same-URL navigation is configured to be ignored. A same-URL skip is not necessarily an application error.

## 15. What is `NavigationError`?

It is an event emitted when a navigation fails because of an unexpected error. The navigation promise rejects by default in this case. The event includes the destination URL and error for diagnostics.

## 16. How can an application observe router navigation outcomes?

Subscribe to `Router.events` and filter for event classes such as `NavigationCancel`, `NavigationSkipped`, and `NavigationError`. Use class checks or a TypeScript type guard so the event type is narrowed before reading event-specific fields.

Use a lifecycle-aware subscription strategy such as `takeUntilDestroyed` in a service or component that owns the subscription.

## 17. Is event monitoring a replacement for awaiting the promise?

No. A caller that initiated a specific navigation should inspect its returned promise when it needs a local outcome. The router event stream is useful for broader logging and diagnostics across navigations, including browser Back and Forward.

## 18. What does `resolveNavigationPromiseOnError` change?

When enabled in router configuration, an unexpected navigation error resolves the navigation promise to `false` instead of rejecting it. Router events can still provide error details. Without this option, catch the rejected promise.

## 19. Should I configure `resolveNavigationPromiseOnError`?

Only when the application intentionally wants navigation failures to use a resolved-false flow. The default rejection behavior makes unexpected errors distinct from guard cancellations and skipped navigations. Either way, record or handle errors appropriately.

## 20. Should navigation happen from a template expression?

No. Trigger it from a user event or after an application operation completes. Navigating during rendering or from a computed value can cause repeated navigation and confusing state changes.

## 21. Is `Router.navigate()` a replacement for a link?

No. Use an anchor with `RouterLink` when the visitor is choosing a normal destination. Programmatic navigation is for application-driven transitions. A button is appropriate when the user performs an action that then causes the application to navigate, such as submitting a form.

## 22. What should happen after saving a form?

A common flow is:

1. Attempt to save.
2. If saving fails, show the save error and do not navigate.
3. If saving succeeds, request navigation to the next route.
4. Await the navigation result.
5. Show a fallback if it resolves to `false` and catch unexpected errors.

Do not treat a successful save as proof that the next route activated.

## 23. Where can I verify the behavior?

See the official Angular programmatic navigation guide, Router API, NavigationCancel API, NavigationSkipped API, and NavigationError API.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="076-query-parameters-fragments-and-navigation-state.md">076 - Query parameters, fragments, and navigation state</a></td><td><a href="../lessons/077-navigate-programmatically-and-handle-navigation-results.md">Lesson</a></td><td><a href="../examples/077-navigate-programmatically-and-handle-navigation-results.md">Example</a></td><td><a href="078-route-guards-and-access-control-boundaries.md">078 - Route guards and access control boundaries</a></td>
</tr>
  </tbody>
</table>
