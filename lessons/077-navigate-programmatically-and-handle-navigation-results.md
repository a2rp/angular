# Navigate programmatically and handle navigation results

Use the Angular Router service when navigation should happen because of application logic, such as after a form is saved or when a workflow chooses the next page. For ordinary user-facing destinations, prefer an anchor with `RouterLink`. A semantic link already expresses navigation and provides familiar browser behavior.

Programmatic navigation is an asynchronous operation. Its result tells the caller whether the requested navigation completed successfully. Handle that result when later UI work depends on the destination being active, and handle rejected promises when an unexpected navigation error occurs.

## Learning outcomes

After this lesson, you should be able to:

- Inject `Router` and navigate with `navigate()` or `navigateByUrl()`.
- Choose a command array for building a URL or a full URL/`UrlTree` for an absolute target.
- Build dynamic and relative targets.
- Await the returned `Promise<boolean>` and handle success, a non-success result, and errors.
- Distinguish navigation completion from cancellation, skipping, and unexpected errors.
- Use router events for application-level diagnostics.

## Use declarative links for ordinary navigation

For a normal destination that a visitor can activate, use an anchor with `RouterLink`:

~~~html
<a routerLink="/help">Help</a>
~~~

Use programmatic navigation for logic-driven transitions, for example after saving a record, when a user completes a multi-step task, or after application code decides where to continue. Do not replace ordinary links with click handlers just to call the router.

## Inject the router

Inject `Router` into a component or service that owns the navigation decision:

~~~ts
import {Component, inject} from '@angular/core';
import {Router} from '@angular/router';

@Component({
  selector: 'app-order-editor',
  template: '<button type="button" (click)="openHelp()">Open help</button>',
})
export class OrderEditorPage {
  private readonly router = inject(Router);

  async openHelp(): Promise<void> {
    const succeeded = await this.router.navigateByUrl('/help');

    if (!succeeded) {
      // The requested navigation did not complete successfully.
      return;
    }

    // The Help route is now active.
  }
}
~~~

`Router` is provided by the router configuration, normally through `provideRouter(routes)`.

## Build a target with `navigate()`

`navigate()` takes an array of commands and optional navigation extras. It is useful when assembling a destination from path segments, parameters, or relative route context:

~~~ts
await this.router.navigate(['/orders', orderId, 'confirmation']);
~~~

With `orderId` equal to `'42'`, this targets `/orders/42/confirmation`.

Use `relativeTo` when the commands should be interpreted from a specific activated route:

~~~ts
await this.router.navigate(['edit'], {
  relativeTo: this.route,
});
~~~

Without `relativeTo`, a leading slash makes a target root-relative. A command without a leading slash is relative to the current route context. Relative navigation is useful in nested route trees; use an explicit root-relative path when the target should always start at the application root.

`navigate()` also accepts URL creation options, such as query parameters and fragments:

~~~ts
await this.router.navigate(['/search'], {
  queryParams: {term: 'angular', page: 1},
  fragment: 'results',
});
~~~

## Navigate to a complete target with `navigateByUrl()`

`navigateByUrl()` takes an absolute route path or a `UrlTree`. Use it when the complete target URL is already known:

~~~ts
await this.router.navigateByUrl('/search?term=angular&page=1#results');
~~~

Unlike `navigate()`, `navigateByUrl()` does not build a delta from the current route. Its extras control navigation behavior; pass query and fragment values in the URL itself or construct a `UrlTree` when needed.

## Understand the navigation result

Both methods return a `Promise<boolean>`:

- `true` means the requested navigation succeeded.
- `false` means it did not complete successfully, for example because it was canceled or skipped.
- By default, an unexpected navigation error rejects the promise.

Await or return the promise when code needs to react to the result. Use `try/catch` for rejected errors:

~~~ts
async continueWorkflow(): Promise<void> {
  try {
    const succeeded = await this.router.navigate(['/next-step']);

    if (!succeeded) {
      this.message = 'The next page could not be opened.';
      return;
    }

    this.message = 'Workflow continued.';
  } catch (error: unknown) {
    this.message = 'Navigation encountered an unexpected error.';
    console.error('Navigation failed', error);
  }
}
~~~

A resolved `false` and a rejected promise are different outcomes. Do not treat `false` as a thrown exception, and do not leave a rejected promise unhandled.

When a method only starts navigation and does not need to inspect its result, it can intentionally ignore the returned promise with `void`. Do this only when errors are handled elsewhere or cannot affect the caller's UI:

~~~ts
void this.router.navigate(['/help']);
~~~

## Cancellation, skipping, and errors

A navigation can fail to activate its requested route without an unexpected exception. For example, a guard may reject navigation, a newer navigation may supersede an earlier one, or the router may skip a request to the current URL. Such outcomes are represented by router navigation events and normally produce a non-success result.

An unexpected failure emits `NavigationError`. The promise returned by a navigation call rejects by default when such an error occurs.

A redirect can emit `NavigationCancel` for the original URL because that navigation is being replaced. The router then performs navigation to the redirect destination. Do not assume every cancellation event is a user-visible failure; inspect the event code and the final navigation outcome.

The optional `resolveNavigationPromiseOnError` router setting changes unexpected errors to a resolved `false` rather than a rejected promise:

~~~ts
import {ApplicationConfig} from '@angular/core';
import {provideRouter, withRouterConfig} from '@angular/router';

import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withRouterConfig({
        resolveNavigationPromiseOnError: true,
      }),
    ),
  ],
};
~~~

Use this setting only when the application intentionally wants promise-based error handling to use the same false-result path. Router events still provide details about the error. Otherwise, keep the default rejection behavior and catch errors where navigation is awaited.

## Monitor router events

The router exposes an event stream for application-wide diagnostics. A service can watch for cancellation, skipped navigation, and unexpected errors. Filter events by their classes or stable codes; the human-readable `reason` text is intended for debugging and may change.

~~~ts
import {DestroyRef, inject, Injectable} from '@angular/core';
import {takeUntilDestroyed} from '@angular/core/rxjs-interop';
import {
  NavigationCancel,
  NavigationError,
  NavigationSkipped,
  Router,
} from '@angular/router';
import {filter} from 'rxjs';

@Injectable({providedIn: 'root'})
export class RouterDiagnostics {
  private readonly router = inject(Router);
  private readonly destroyRef = inject(DestroyRef);

  constructor() {
    this.router.events
      .pipe(
        filter((event) =>
          event instanceof NavigationCancel ||
          event instanceof NavigationSkipped ||
          event instanceof NavigationError
        ),
        takeUntilDestroyed(this.destroyRef),
      )
      .subscribe((event) => {
        if (event instanceof NavigationError) {
          console.error('Navigation error', event.url, event.error);
        } else {
          console.info('Navigation did not activate its requested URL', {
            url: event.url,
            code: event.code,
          });
        }
      });
  }
}
~~~

Register or inject this diagnostics service if the application needs these logs. Avoid turning expected guard cancellations or same-URL skips into alarming end-user errors.

## Common mistakes

- **Using code-driven navigation for every link:** use `RouterLink` for normal user-selected destinations.
- **Not awaiting a result that affects later work:** later UI code can run before navigation finishes.
- **Ignoring `false`:** a resolved false result means the destination did not activate as requested.
- **Ignoring rejected promises:** catch unexpected errors or make an intentional decision to handle them through router events.
- **Assuming every cancellation means failure:** redirects cancel an original navigation and can still lead to a successful destination.
- **Confusing the APIs:** use `navigate()` to build a target from commands and creation options; use `navigateByUrl()` for a complete absolute path or `UrlTree`.
- **Using an ambiguous relative command:** choose a leading slash or an explicit `relativeTo` when the target must be clear.
- **Navigating during rendering:** do not trigger navigation from a template expression or a computed value; navigate in response to an event or a completed operation.

## A reliable workflow

1. Complete the application action that should happen before navigation, such as saving a form.
2. Build the target with `navigate()` or `navigateByUrl()`.
3. Await its result if the caller needs to know whether navigation succeeded.
4. Handle `false` separately from unexpected errors.
5. Use router events for cross-application diagnostics, not as a replacement for a local result that the caller already receives.

## Official references

- [Navigate to routes](https://angular.dev/guide/routing/navigate-to-routes)
- [Router API](https://angular.dev/api/router/Router)
- [NavigationCancel API](https://angular.dev/api/router/NavigationCancel)
- [NavigationSkipped API](https://angular.dev/api/router/NavigationSkipped)
- [NavigationError API](https://angular.dev/api/router/NavigationError)
- [RouterConfigOptions API](https://angular.dev/api/router/RouterConfigOptions)
