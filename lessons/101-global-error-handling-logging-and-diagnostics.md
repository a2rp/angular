# Global Error Handling, Logging, and Diagnostics

Applications need useful error handling at two levels. A feature handles failures it can recover from and explains them to the user. A global handler reports unexpected failures that the application or framework could not handle locally. Logging provides evidence for debugging, but it must not expose secrets or personal data.

## Learning outcomes

After this lesson, you should be able to:

- Handle expected request and user-action failures near their source.
- Explain what Angular's ErrorHandler is for.
- Configure global browser error forwarding when needed.
- Record structured, privacy-aware diagnostics.
- Keep logging failures from causing additional application failures.
- Choose a local fallback or error boundary for a broken region of UI.

## 1. Handle expected failures at the operation

A network request can fail because the user is offline, a session expired, or a server returned an error. The code that initiated the request has the context needed to decide whether to retry, display a validation message, preserve stale data, or offer a sign-in action.

For a request, handle the error where the request is made. For a form submission, show the failure beside the relevant action. For a route that cannot load, use route-level behavior. Do not rely on a global handler to infer how a particular feature should recover.

Expected errors are part of normal product behavior. Represent them as explicit state when the UI needs to respond:

~~~ts
type LoadState<T> =
  | { status: 'loading' }
  | { status: 'success'; value: T }
  | { status: 'error'; message: string };

function toLoadError<T>(): LoadState<T> {
  return {
    status: 'error',
    message: 'The information could not be loaded. Try again.'
  };
}
~~~

Do not show stack traces, raw server bodies, internal hostnames, or authentication details to end users.

## 2. Use ErrorHandler for unexpected errors

Angular reports framework-managed errors that are not handled by application code to the root ErrorHandler. The default handler writes errors to the console. A custom handler can forward unexpected errors to a diagnostics service.

The handler is not a replacement for try/catch or RxJS catchError at a call site. Directly invoked APIs do not all automatically route every failure to ErrorHandler. Handle an error locally when the application knows how to recover; report it globally when it is unexpected and important for investigation.

## 3. Forward browser-level unhandled errors

Angular provides provideBrowserGlobalErrorListeners to forward browser window error and unhandledrejection events to ErrorHandler. Recent Angular CLI applications may include this provider by default. Check the actual app configuration before adding it again or registering duplicate listeners.

For SSR, browser and server error sources differ. Angular has separate server-side handling behavior. Do not access window in code that can execute on the server.

## 4. Record structured, privacy-aware diagnostics

A useful diagnostic event might include an event name, a normalized error category, a safe route identifier, a release version, and a request correlation ID. Prefer structured fields over parsing a long free-form string later.

Avoid sending passwords, access tokens, full request or response bodies, personal identifiers, payment details, or sensitive query parameters. If a URL is useful, remove its query and fragment or map it to a safe route name. Apply retention, access control, and consent requirements to the logging system.

A logging service should avoid throwing. If the reporting network call fails, it must not create a second uncaught error or block the user's original action. Production reporting can be queued, sampled, rate-limited, or disabled based on application policy.

## 5. Keep a global handler small

The global handler should report and classify; it should not attempt to repair arbitrary application state. A repeated error loop can produce noisy telemetry, so consider deduplication and rate limiting.

Do not silently swallow the error. Logging it is not the same as recovering. If users need to see an error state or retry, handle that in the feature or a local rendering boundary.

## 6. Use a local error boundary for rendering failures

Newer Angular releases provide declarative template error boundaries for rendering subtrees. A boundary can show fallback content around a part of the page while allowing the rest of the application to remain available. Verify that @boundary is available and stable in the Angular version used by the project.

An error boundary is for rendering or view failures, not a general wrapper around every HTTP request. Requests still need local error state because the component knows the user action and recovery options.

## 7. Keep logs useful without over-logging

Logging every successful change can drown out useful failures and collect more data than needed. Define event names and levels such as debug, info, warn, and error. Include enough context to distinguish repeated incidents while keeping payloads small and safe.

Use a consistent correlation identifier across a client request and server logs when the server supports it. This makes a failure easier to trace without placing sensitive values in diagnostic messages.

## 8. Review your error strategy

For each failure, ask:

1. Is this an expected operational result or an unexpected programming failure?
2. Which code has enough context to choose the right recovery?
3. What should the user see?
4. What information helps a developer diagnose it?
5. Could the diagnostic contain secrets or personal data?
6. Can the reporting path itself fail safely?
7. Does the user need retry, sign-in, or a support path?

See Angular's [error handling best practices](https://angular.dev/best-practices/error-handling), [ErrorHandler API](https://angular.dev/api/core/ErrorHandler), and, on supported versions, [template error boundaries](https://angular.dev/guide/templates/error-boundaries).
