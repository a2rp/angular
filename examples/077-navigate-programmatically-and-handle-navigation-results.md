# Navigate programmatically and handle navigation results: example

This standalone example has an order editor, a confirmation page, and a Help page. It demonstrates two programmatic navigation styles:

- Build a dynamic target with `Router.navigate()` after an order is saved.
- Open a known absolute path with `Router.navigateByUrl()`.

Both methods return a promise. The editor handles a resolved `false` separately from a rejected promise.

## File layout

~~~text
src/
├── app/
│   ├── pages/
│   │   ├── help-page.ts
│   │   ├── order-confirmation-page.ts
│   │   └── order-editor-page.ts
│   ├── app.component.ts
│   ├── app.config.ts
│   └── app.routes.ts
└── main.ts
~~~

## 1. Define the routes

Create `src/app/app.routes.ts`:

~~~ts
import {Routes} from '@angular/router';

import {HelpPage} from './pages/help-page';
import {OrderConfirmationPage} from './pages/order-confirmation-page';
import {OrderEditorPage} from './pages/order-editor-page';

export const routes: Routes = [
  {
    path: 'orders/:orderId/edit',
    component: OrderEditorPage,
    title: 'Edit order',
  },
  {
    path: 'orders/:orderId/confirmation',
    component: OrderConfirmationPage,
    title: 'Order saved',
  },
  {
    path: 'help',
    component: HelpPage,
    title: 'Help',
  },
  {
    path: '',
    pathMatch: 'full',
    redirectTo: 'orders/42/edit',
  },
  {
    path: '**',
    redirectTo: 'help',
  },
];
~~~

## 2. Register the router and outlet

Create `src/app/app.config.ts`:

~~~ts
import {ApplicationConfig} from '@angular/core';
import {provideRouter} from '@angular/router';

import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)],
};
~~~

Create `src/app/app.component.ts`:

~~~ts
import {Component} from '@angular/core';
import {RouterLink, RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterLink, RouterOutlet],
  template: `
    <header>
      <a routerLink="/orders/42/edit">Order editor</a>
    </header>
    <main>
      <router-outlet />
    </main>
  `,
})
export class AppComponent {}
~~~

The component imports both directives used by its template: `RouterLink` for the anchor and `RouterOutlet` for the active route view.

The CLI-generated `src/main.ts` should bootstrap the root component with the configuration:

~~~ts
import {bootstrapApplication} from '@angular/platform-browser';

import {AppComponent} from './app/app.component';
import {appConfig} from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch((error: unknown) => console.error(error));
~~~

## 3. Save, then navigate with a command array

Create `src/app/pages/order-editor-page.ts`:

~~~ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute, Router, RouterLink} from '@angular/router';

@Component({
  selector: 'app-order-editor-page',
  standalone: true,
  imports: [RouterLink],
  template: `
    <h1>Edit order {{ orderId }}</h1>
    <p>Make the required changes, then save the order.</p>

    <button
      type="button"
      [disabled]="saving"
      (click)="saveAndContinue()"
    >
      {{ saving ? 'Saving...' : 'Save and continue' }}
    </button>

    <button type="button" (click)="openHelp()">
      Open help
    </button>

    @if (message) {
      <p role="status">{{ message }}</p>
    }

    <p>
      <a routerLink="/help">Help page (regular link)</a>
    </p>
  `,
})
export class OrderEditorPage {
  private readonly route = inject(ActivatedRoute);
  private readonly router = inject(Router);

  readonly orderId =
    this.route.snapshot.paramMap.get('orderId') ?? 'unknown';

  saving = false;
  message = '';

  async saveAndContinue(): Promise<void> {
    this.saving = true;
    this.message = '';

    let savedOrderId: string;

    try {
      savedOrderId = await this.persistOrder();
    } catch (error: unknown) {
      this.message = 'The order could not be saved.';
      console.error('Order save failed', error);
      this.saving = false;
      return;
    }

    try {
      const succeeded = await this.router.navigate([
        '/orders',
        savedOrderId,
        'confirmation',
      ]);

      if (!succeeded) {
        this.message = 'The order was saved, but the next page did not open.';
      }
      // On success, the confirmation route replaces this editor in the outlet.
    } catch (error: unknown) {
      this.message = 'The order was saved, but navigation encountered an error.';
      console.error('Navigation failed', error);
    } finally {
      this.saving = false;
    }
  }

  async openHelp(): Promise<void> {
    this.message = '';

    try {
      const succeeded = await this.router.navigateByUrl('/help');

      if (!succeeded) {
        this.message = 'The Help page did not open.';
      }
    } catch (error: unknown) {
      this.message = 'Opening Help encountered an unexpected error.';
      console.error('Help navigation failed', error);
    }
  }

  private async persistOrder(): Promise<string> {
    // Replace this demo with the real order-save request.
    await Promise.resolve();
    return this.orderId;
  }
}
~~~

The sample shows the save and navigation outcomes separately. A save failure does not start navigation. After a successful save, the editor awaits the router. If the router resolves `false` or rejects, the editor can show a message; if the navigation succeeds, the confirmation component appears at the outlet.

The `persistOrder()` method is an asynchronous stand-in for a real API request. In an application, call the data service and use the saved order ID it returns.

## 4. Add the destination components

Create `src/app/pages/order-confirmation-page.ts`:

~~~ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute, RouterLink} from '@angular/router';

@Component({
  selector: 'app-order-confirmation-page',
  standalone: true,
  imports: [RouterLink],
  template: `
    <h1>Order saved</h1>
    <p>Order ID: {{ orderId }}</p>
    <a [routerLink]="['/orders', orderId, 'edit']">Return to the editor</a>
  `,
})
export class OrderConfirmationPage {
  private readonly route = inject(ActivatedRoute);

  readonly orderId =
    this.route.snapshot.paramMap.get('orderId') ?? 'unknown';
}
~~~

Create `src/app/pages/help-page.ts`:

~~~ts
import {Component} from '@angular/core';
import {RouterLink} from '@angular/router';

@Component({
  selector: 'app-help-page',
  standalone: true,
  imports: [RouterLink],
  template: `
    <h1>Help</h1>
    <p>This page can be opened with a normal link or programmatically.</p>
    <a routerLink="/orders/42/edit">Return to the order editor</a>
  `,
})
export class HelpPage {}
~~~

## 5. Observe failures across the application

A local caller should handle the promise for the navigation it initiated. For diagnostics across multiple routes, a service can listen to router events. This example logs unexpected errors, cancellations, and skipped requests:

~~~ts
import {DestroyRef, inject, Injectable} from '@angular/core';
import {takeUntilDestroyed} from '@angular/core/rxjs-interop';
import {
  Event as RouterEvent,
  NavigationCancel,
  NavigationError,
  NavigationSkipped,
  Router,
} from '@angular/router';
import {filter} from 'rxjs';

type DiagnosticNavigationEvent =
  | NavigationCancel
  | NavigationSkipped
  | NavigationError;

function isDiagnosticNavigationEvent(
  event: RouterEvent,
): event is DiagnosticNavigationEvent {
  return event instanceof NavigationCancel ||
    event instanceof NavigationSkipped ||
    event instanceof NavigationError;
}

@Injectable({providedIn: 'root'})
export class RouterDiagnostics {
  private readonly router = inject(Router);
  private readonly destroyRef = inject(DestroyRef);

  constructor() {
    this.router.events
      .pipe(
        filter(isDiagnosticNavigationEvent),
        takeUntilDestroyed(this.destroyRef),
      )
      .subscribe((event) => {
        if (event instanceof NavigationError) {
          console.error('Router error', event.url, event.error);
        } else {
          console.info('Router navigation did not activate its requested URL', {
            url: event.url,
            code: event.code,
          });
        }
      });
  }
}
~~~

Inject `RouterDiagnostics` somewhere in the application if these logs are needed. A redirect may cancel its original navigation and then complete a new one, so not every cancellation should be presented as a visitor-facing error.

## 6. Try the example

Start the development server:

~~~sh
ng serve
~~~

Then check:

1. The root URL redirects to `/orders/42/edit`.
2. Save and continue calls the asynchronous save stub, then navigates to `/orders/42/confirmation`.
3. The confirmation page shows the order ID from the route.
4. Open Help calls `navigateByUrl('/help')` and activates the Help page.
5. The regular Help anchor works without a click handler.
6. If a guard is temporarily added to reject confirmation navigation, the editor stays visible and shows its resolved-false message.
7. If a navigation error is triggered during development, the catch branch logs it and keeps an error message in the editor.
8. Router diagnostics logs relevant events without treating every cancellation as an unexpected exception.

## Key points

- Use `RouterLink` for a regular destination and programmatic navigation for logic-driven transitions.
- `navigate()` constructs a URL from commands and extras.
- `navigateByUrl()` accepts a complete absolute URL path or `UrlTree`.
- Both return `Promise<boolean>`.
- Handle resolved `false` separately from rejected navigation errors.
- A cancellation event can be part of a redirect; check the final outcome.
- `void` alone does not catch promise rejections.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="076-query-parameters-fragments-and-navigation-state.md">076 - Query parameters, fragments, and navigation state</a></td><td><a href="../lessons/077-navigate-programmatically-and-handle-navigation-results.md">Lesson</a></td><td><a href="../qnas/077-navigate-programmatically-and-handle-navigation-results.md">Q&amp;A</a></td><td><a href="078-route-guards-and-access-control-boundaries.md">078 - Route guards and access control boundaries</a></td>
</tr>
  </tbody>
</table>
