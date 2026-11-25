# Global Error Handling, Logging, and Diagnostics — Examples

This example separates an expected page-load failure from an unexpected application error. The feature displays a useful message for its own request; the global handler reports unexpected errors with a small, privacy-aware event.

## 1. Define a small diagnostic event

Do not send raw request data, tokens, full URLs, or arbitrary error messages. Start with a limited event shape:

~~~ts
export interface ClientDiagnostic {
  event: 'unexpected-client-error';
  category: string;
  release: string;
  occurredAt: string;
}
~~~

The release value should come from the application's build or deployment configuration. The example uses a fixed value only to show where that setting belongs.

## 2. Create a reporter that cannot throw into the app

~~~ts
import { Injectable } from '@angular/core';
import { ClientDiagnostic } from './client-diagnostic';

@Injectable({ providedIn: 'root' })
export class DiagnosticsService {
  reportUnexpected(error: unknown): void {
    const event: ClientDiagnostic = {
      event: 'unexpected-client-error',
      category: this.categoryOf(error),
      release: 'web-2026.11',
      occurredAt: new Date().toISOString()
    };

    try {
      if (typeof navigator !== 'undefined' && navigator.sendBeacon) {
        const body = new Blob([JSON.stringify(event)], {
          type: 'application/json'
        });
        navigator.sendBeacon('/api/client-diagnostics', body);
      }
    } catch {
      // Telemetry failure must not create another uncaught application error.
    }
  }

  private categoryOf(error: unknown): string {
    if (error instanceof TypeError) {
      return 'type-error';
    }

    if (error instanceof Error) {
      return 'application-error';
    }

    return 'unknown-error';
  }
}
~~~

This reporter intentionally omits error.message and stack because those values can contain sensitive information. In a real application, use an approved telemetry SDK and define an explicit redaction policy. A sendBeacon result is only a best-effort delivery signal; it is not a guaranteed durable report.

## 3. Forward unexpected Angular errors

~~~ts
import { ErrorHandler, Injectable, inject } from '@angular/core';
import { DiagnosticsService } from './diagnostics.service';

@Injectable()
export class AppErrorHandler implements ErrorHandler {
  private readonly diagnostics = inject(DiagnosticsService);

  handleError(error: unknown): void {
    this.diagnostics.reportUnexpected(error);

    if (isDevMode()) {
      console.error('Unexpected application error', error);
    }
  }
}
~~~

Add the missing Angular import at the top:

~~~ts
import { isDevMode } from '@angular/core';
~~~

In a production application, send only the minimized diagnostic event. Use the console branch for local development. Avoid recursively calling ErrorHandler if the diagnostic request fails.

## 4. Register the handler and browser error forwarding

~~~ts
// app.config.ts
import { ApplicationConfig, ErrorHandler } from '@angular/core';
import { provideBrowserGlobalErrorListeners } from '@angular/core';
import { AppErrorHandler } from './diagnostics/app-error-handler';

export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    { provide: ErrorHandler, useClass: AppErrorHandler }
  ]
};
~~~

Recent Angular CLI projects may already include provideBrowserGlobalErrorListeners. Inspect the generated configuration before adding it so the app does not register duplicate listeners. If custom window listeners are used instead, define one owner and avoid double forwarding.

## 5. Handle an expected request failure where it occurs

~~~ts
import { Component, inject, signal } from '@angular/core';
import { catchError, of } from 'rxjs';
import { toSignal } from '@angular/core/rxjs-interop';
import { ProfileApi } from './profile-api.service';

@Component({
  selector: 'app-profile-page',
  templateUrl: './profile-page.html'
})
export class ProfilePage {
  private readonly api = inject(ProfileApi);
  readonly loadMessage = signal<string | null>(null);

  readonly profile = toSignal(
    this.api.loadProfile().pipe(
      catchError(() => {
        this.loadMessage.set(
          'Your profile could not be loaded. Check your connection and try again.'
        );
        return of(null);
      })
    ),
    { initialValue: null }
  );
}
~~~

~~~html
@if (loadMessage(); as message) {
  <p role="alert">{{ message }}</p>
} @else if (profile(); as value) {
  <h1>{{ value.displayName }}</h1>
} @else {
  <p role="status">Loading profile…</p>
}
~~~

In a real page, add a retry action that invokes the request again. This error is handled by the feature because the page knows how to explain it. It should not be reported as an uncaught global crash unless the product intentionally records that operational failure as a separate event.

## 6. Optional: isolate a rendering subtree

Where supported by the Angular version in use, an error boundary can show fallback UI around a region that fails while rendering:

~~~html
@boundary {
  <app-analytics-dashboard />
} @error {
  <section role="alert">
    <h2>Dashboard unavailable</h2>
    <p>Try again later or open another section.</p>
  </section>
}
~~~

Use a boundary for rendering failures. It does not replace request-state handling in AnalyticsDashboard. Check the Angular version's documentation for the exact supported syntax and stability.

## 7. Review what leaves the browser

Before enabling telemetry, verify that it does not include passwords, access tokens, raw form values, full request bodies, personal identifiers, or sensitive URL parameters. Confirm the server validates payload size, applies access controls and retention, and does not return a failure that recursively triggers the same reporting pipeline.

## Further reading

- [Angular error handling best practices](https://angular.dev/best-practices/error-handling)
- [Angular ErrorHandler API](https://angular.dev/api/core/ErrorHandler)
- [Browser global error listeners](https://angular.dev/api/core/provideBrowserGlobalErrorListeners)
- [Angular template error boundaries](https://angular.dev/guide/templates/error-boundaries)
