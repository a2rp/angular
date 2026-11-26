# Functional Interceptors and Request Context - Examples

These examples show functional interceptors and per-request metadata. They assume HttpClient is configured with provideHttpClient.

## 1. Log a request and its final response

An interceptor can inspect the outgoing request and observe the final HttpResponse:

~~~ts
// request-log.interceptor.ts
import {
  HttpEvent,
  HttpEventType,
  HttpInterceptorFn
} from '@angular/common/http';
import { tap } from 'rxjs';

export const requestLogInterceptor: HttpInterceptorFn = (req, next) => {
  const startedAt = Date.now();

  console.log('HTTP request:', req.method, req.urlWithParams);

  return next(req).pipe(
    tap((event: HttpEvent<unknown>) => {
      if (event.type === HttpEventType.Response) {
        console.log(
          'HTTP response:',
          req.urlWithParams,
          event.status,
          Date.now() - startedAt,
          'ms'
        );
      }
    })
  );
};
~~~

This is only a basic diagnostic example. Avoid logging authorization headers, cookies, request bodies, or sensitive response data.

## 2. Register an interceptor chain

~~~ts
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import {
  provideHttpClient,
  withInterceptors
} from '@angular/common/http';
import { requestIdInterceptor } from './request-id.interceptor';
import { loadingInterceptor } from './loading.interceptor';
import { requestLogInterceptor } from './request-log.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([
        requestIdInterceptor,
        loadingInterceptor,
        requestLogInterceptor
      ])
    )
  ]
};
~~~

For requests, the functions run in listed order. Response events pass back through the chain in reverse order. If one interceptor depends on another, document the order and keep the dependency narrow.

## 3. Add a request header without mutating the original request

~~~ts
// request-id.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';

export const requestIdInterceptor: HttpInterceptorFn = (req, next) => {
  const updatedRequest = req.clone({
    setHeaders: {
      'X-Request-Source': 'web-app'
    }
  });

  return next(updatedRequest);
};
~~~

The clone contains the added header. The original request is unchanged. In production, generate or obtain a request identifier according to the application's tracing design.

## 4. Create a typed context token

A token can control whether a global loading indicator applies to a particular request:

~~~ts
// http-context.tokens.ts
import { HttpContextToken } from '@angular/common/http';

export const SKIP_GLOBAL_LOADING =
  new HttpContextToken<boolean>(() => false);
~~~

The default-value factory returns false for requests that do not set the token.

## 5. Implement a loading counter service

A counter handles overlapping requests correctly; one request finishing should not hide the indicator while another request remains active:

~~~ts
// loading-indicator.service.ts
import { computed, Injectable, signal } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class LoadingIndicatorService {
  private readonly pendingCount = signal(0);

  readonly isLoading = computed(() => this.pendingCount() > 0);

  begin(): void {
    this.pendingCount.update(count => count + 1);
  }

  end(): void {
    this.pendingCount.update(count => Math.max(0, count - 1));
  }
}
~~~

A real application may track requests individually if it needs more detail than a shared count.

## 6. Use context in a loading interceptor

~~~ts
// loading.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { finalize } from 'rxjs';
import { LoadingIndicatorService } from './loading-indicator.service';
import { SKIP_GLOBAL_LOADING } from './http-context.tokens';

export const loadingInterceptor: HttpInterceptorFn = (req, next) => {
  if (req.context.get(SKIP_GLOBAL_LOADING)) {
    return next(req);
  }

  const loading = inject(LoadingIndicatorService);
  loading.begin();

  return next(req).pipe(
    finalize(() => loading.end())
  );
};
~~~

finalize runs when the request stream completes, errors, or is unsubscribed. This keeps the counter balanced across success, failure, and client-side cancellation.

## 7. Opt one request out of the global loading indicator

~~~ts
// background-job.service.ts
import {
  HttpClient,
  HttpContext
} from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { SKIP_GLOBAL_LOADING } from './http-context.tokens';

export interface JobStatus {
  state: 'queued' | 'running' | 'complete';
}

@Injectable({ providedIn: 'root' })
export class BackgroundJobService {
  private readonly http = inject(HttpClient);

  getStatus() {
    const context = new HttpContext()
      .set(SKIP_GLOBAL_LOADING, true);

    return this.http.get<JobStatus>('/api/jobs/status', {
      context
    });
  }
}
~~~

The token is read by the interceptor only. The server receives the normal request and does not receive the context value.

## 8. Display the shared loading state

A component can read the service's signal:

~~~ts
// loading-indicator.component.ts
import { Component, inject } from '@angular/core';
import { LoadingIndicatorService } from './loading-indicator.service';

@Component({
  selector: 'app-loading-indicator',
  standalone: true,
  templateUrl: './loading-indicator.component.html'
})
export class LoadingIndicatorComponent {
  readonly loading = inject(LoadingIndicatorService);
}
~~~

~~~html
<!-- loading-indicator.component.html -->
@if (loading.isLoading()) {
  <div role="status" aria-live="polite">Loading…</div>
}
~~~

Avoid announcing every small request as a disruptive alert. Choose an accessible presentation that matches the application and its request duration.

## 9. Use an object token default safely

For object or array defaults, provide a factory that creates a fresh value:

~~~ts
import { HttpContextToken } from '@angular/common/http';

export interface RequestMetadata {
  labels: string[];
}

export const REQUEST_METADATA =
  new HttpContextToken<RequestMetadata>(() => ({
    labels: []
  }));
~~~

If code changes the object returned by get, keep in mind that context values are mutable and cloned requests share their context unless a new context is supplied. Prefer explicit updates and avoid using mutable metadata as hidden cross-request state.

## 10. Skip a specific interceptor behavior without changing the API

A context flag can be read by one interceptor and ignored by others:

~~~ts
export const cacheInterceptor: HttpInterceptorFn = (req, next) => {
  if (!req.context.get(CACHE_RESPONSE)) {
    return next(req);
  }

  // Cache behavior would be implemented here.
  return next(req);
};
~~~

Define CACHE_RESPONSE as a HttpContextToken<boolean> and set it in request options when desired. Do not use a custom header just to signal behavior that is only relevant inside the Angular client pipeline.

## 11. Important security boundary for auth interceptors

When adding authentication headers, check that the request targets the trusted API before attaching a token. A global interceptor that blindly adds credentials to every URL can leak them to a third-party or attacker-controlled destination.

An interceptor is not server authorization. The server must verify permissions for each protected operation.

## 12. Observable subscriptions and repeated requests

HttpClient Observables are cold. Subscribing twice sends two requests, and each request traverses the interceptor chain:

~~~ts
const status$ = this.http.get('/api/status');

status$.subscribe();
status$.subscribe(); // A second request and interceptor pass.
~~~

Avoid duplicate subscriptions when the feature intends to make one request. If multiple consumers should share a request result, use an explicit sharing or caching approach.

## Checklist

- Register functions through provideHttpClient(withInterceptors(...)).
- Keep interceptor ordering intentional.
- Clone requests to change headers or options.
- Use HttpContextToken for client-side per-request metadata.
- Remember context is not sent to the server and is mutable.
- Keep credentials limited to the intended API.
- Use finalize for cleanup that must also happen on errors or unsubscribe.
- Do not log secrets or sensitive payloads.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="088-upload-and-download-progress-events.md">088 - Upload and Download Progress Events</a></td><td><a href="../lessons/089-functional-interceptors-and-request-context.md">Lesson</a></td><td><a href="../qnas/089-functional-interceptors-and-request-context.md">Q&amp;A</a></td><td><a href="090-http-errors-retry-policy-and-timeout-handling.md">090 - HTTP Errors, Retry Policy, and Timeout Handling</a></td>
</tr>
  </tbody>
</table>
