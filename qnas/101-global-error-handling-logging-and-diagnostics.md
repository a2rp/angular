# Global Error Handling, Logging, and Diagnostics — Q&A

## 1. What is Angular's ErrorHandler for?

It is the application-level hook for reporting errors Angular surfaces when framework-managed code runs and the error is not handled locally. The default implementation writes errors to the console. A custom implementation can forward unexpected failures to a diagnostics system.

## 2. Should ErrorHandler replace local error handling?

No. The code that initiates a request or user action usually has the context needed to show a useful message, retry, or preserve data. Handle expected operational failures at that call site. Use the global handler mainly to report unexpected errors.

## 3. Are all rejected promises and thrown errors automatically sent to ErrorHandler?

No. Angular forwards errors according to the framework API contract and execution context. Errors from code directly invoked by application code may need explicit try/catch or RxJS handling. Check the behavior of the specific API rather than assuming all asynchronous failures are caught.

## 4. Should a network request failure be treated as an unexpected crash?

Usually not. Network failure, authorization expiry, and server validation are operational outcomes that a feature may need to display or recover from. Represent them in feature state and report them separately if the product needs diagnostics.

## 5. What does provideBrowserGlobalErrorListeners do?

It adds browser-level listeners for window error and unhandledrejection events and forwards those errors to ErrorHandler. Recent CLI-generated apps may already include it. Inspect the application providers before adding another listener or a duplicate custom handler.

## 6. Does browser error-listener configuration also configure SSR?

No. Browser window listeners and server process error handling are different environments. Avoid using browser globals in server-executed code and check Angular's SSR guidance for server-side behavior.

## 7. What data should a diagnostic event include?

Include only fields needed to understand and correlate the failure, such as a normalized category, safe route name, release version, and a request correlation ID. Use structured fields with stable names.

## 8. What data should a diagnostic event avoid?

Do not include passwords, tokens, full request or response bodies, payment data, personal identifiers, or sensitive URL query parameters. Redact or normalize data before it leaves the client, and apply retention and access controls to the telemetry service.

## 9. Should the global handler show an error dialog?

Not by default. A global handler often lacks context to decide what recovery is appropriate. If a user needs a retry or explanation, the feature or a local error boundary should present it. An unexpected crash may need a global fallback, but avoid showing a noisy dialog for every internal error.

## 10. What if the logging service fails?

The error-reporting path should fail safely and avoid throwing a second error. It should not block the original user action. Consider a bounded queue, rate limiting, sampling, or a fallback that avoids recursive reporting.

## 11. What is an Angular template error boundary for?

On Angular versions that support it, a boundary can show fallback UI for errors during rendering of a subtree. It can isolate a broken visual region so the entire page does not have to disappear. Check availability and stability in the project's Angular version.

## 12. Can an error boundary replace catchError for HTTP requests?

No. A rendering boundary is for view errors. The component or service that owns the request should still model loading, success, and failure so the user can understand the request result and retry when appropriate.

## 13. Should every successful operation be logged?

Not necessarily. Excessive success logging can obscure failures and collect unnecessary data. Choose event levels and retention based on operational need, and sample or rate-limit noisy events.

## 14. Why use a correlation ID?

A correlation ID can connect client diagnostics with server logs for the same request. It helps trace a failure across systems without putting credentials or personal data into free-form messages.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="100-dynamic-components-and-programmatic-rendering.md">100 - Dynamic Components and Programmatic Rendering</a></td><td><a href="../lessons/101-global-error-handling-logging-and-diagnostics.md">Lesson</a></td><td><a href="../examples/101-global-error-handling-logging-and-diagnostics.md">Example</a></td><td><a href="102-internationalization-localization-and-formatting.md">102 - Internationalization, Localization, and Formatting</a></td>
</tr>
  </tbody>
</table>
