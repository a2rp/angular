# Angular DevTools, Debugging, and Production Diagnostics

Angular DevTools and the browser developer tools help answer different debugging questions. Angular DevTools reveals Angular component, directive, injector, signal, and change-detection information. Browser tools reveal JavaScript execution, network requests, DOM, layout, storage, and browser errors. Use both to trace a problem from user-visible behavior to its likely cause.

## Learning outcomes

After this lesson, you should be able to:

- Install and open Angular DevTools for a supported browser.
- Inspect a component tree, component state, directives, and signals.
- Use the profiler to identify expensive change-detection work.
- Correlate Angular activity with browser-level performance data.
- Diagnose runtime errors, failed requests, and template mistakes.
- Understand why optimized production builds expose fewer debugging hooks.
- Use source maps and compiler diagnostics carefully.

## 1. Start with a reliable reproduction

Before changing code, record what the user did, what they expected, and what happened instead. Reduce the issue to the shortest repeatable sequence. Include the route, browser, viewport, application state, and relevant network conditions.

Check whether the problem occurs:

- On every reload or only after a particular interaction.
- In development, production, or both.
- For one route or across the application.
- With real backend responses or with a controlled fixture.

A small reproduction makes logs, breakpoints, and profiles easier to interpret. Remove sensitive data before sharing a console log, network capture, or profile.

## 2. Open Angular DevTools

Install the Angular DevTools browser extension for Chrome or Firefox, then open the browser's developer tools on a page running an Angular application. The Angular panel is not available on browser internal pages such as a new tab.

The Components view presents the rendered Angular component and directive hierarchy. Select a node to inspect its inputs, properties, and metadata. Use the view to verify which component owns a visible part of the page and whether the values supplied to it match expectations. The extension can also navigate from a selected component to its host DOM node or source when the browser and build expose that information.

Angular DevTools needs Angular's development debugging hooks. A normal development server build is suitable. Optimized production builds remove debugging code and metadata to reduce bundle size, so an extension may not show the same component information there. Do not disable production optimization on a public deployment just to make DevTools work. Reproduce the problem locally or use a controlled, access-restricted diagnostic build.

See the official [Angular DevTools overview](https://angular.dev/tools/devtools).

## 3. Inspect signals and component state

The Components view helps connect rendered output to the state that produced it. Follow the data from the selected component's inputs and fields to its template. When a template is stale, verify the state owner and update path before adding manual change-detection calls.

Angular DevTools can display a signal dependency graph for supported Angular versions in development mode. The graph can show signals, computed values, and effects associated with the selected component. Meaningful debug names make nodes easier to identify. Use the graph to trace a surprising value through its dependencies; do not treat the displayed graph as a substitute for understanding the application's state ownership.

Signal graph availability is version- and build-dependent. Consult the current [signals DevTools guide](https://angular.dev/tools/devtools/signals) when using it.

## 4. Profile change detection

The Angular DevTools Profiler records Angular execution such as change-detection cycles and lifecycle work. Start a recording, perform one representative interaction, and stop the recording. Select a slow cycle to see which components or directives consumed time. A component timeline or flame graph can reveal work concentrated in a template, lifecycle hook, or subtree.

Use the profiler to locate a cause before optimizing. Common follow-up actions include:

- Replacing repeated expensive template calculations with appropriately derived state.
- Reducing unnecessary updates or subscriptions.
- Using stable collection tracking and avoiding needless object recreation.
- Splitting large views or applying an appropriate change-detection strategy.
- Moving non-visual work out of frequently executed template expressions.

After a change, record the same interaction under comparable conditions. Profiling data is diagnostic evidence, not a universal performance score; development instrumentation and machine load affect measurements.

The official [Profiler guide](https://angular.dev/tools/devtools/profiler) explains recordings, cycle views, and component execution details.

## 5. Correlate framework and browser activity

Browser developer tools complement Angular DevTools:

- **Console:** inspect runtime exceptions, warnings, and application logs. Follow the first relevant stack frame into application code.
- **Sources/Debugger:** set a breakpoint where state changes or an error is thrown. Inspect local variables and the call stack.
- **Network:** check request URL, method, status, timing, payload, and response. A failed request can look like an Angular rendering issue.
- **Elements:** inspect the actual DOM and computed styles when the state is correct but the screen is wrong.
- **Performance:** record a representative interaction to distinguish JavaScript work from layout, paint, network waiting, and third-party scripts.

Angular can add framework-specific events to Chrome's Performance panel in development mode. Enable profiling before bootstrap when a recording must include startup:

~~~ts
import { enableProfiling } from '@angular/core';
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

enableProfiling();
bootstrapApplication(AppComponent);
~~~

Alternatively, when available in the browser console, call the Angular profiling hook there before recording. The integration is development-only; it is a way to inspect framework work alongside browser activity, not an always-on production monitor. See [Profiling with Chrome DevTools](https://angular.dev/best-practices/profiling-with-chrome-devtools) and [Angular performance best practices](https://angular.dev/best-practices/performance).

## 6. Diagnose errors at their source

Start with the earliest meaningful error. Later template or navigation failures may be consequences of an earlier request or state error. Read the stack trace and inspect the values at the point where the failure occurs.

Handle errors where the application still has enough context to recover. For example, a form submission can show a retryable message, while an unexpected programming error should be reported with enough context for diagnosis. Angular's ErrorHandler is a useful reporting boundary for errors Angular surfaces to it; it does not automatically catch every exception thrown by arbitrary application code. Use local error handling for expected failures and use global reporting for unexpected failures.

Angular's [error handling guidance](https://angular.dev/best-practices/error-handling) describes the framework boundary and recommended approach. Avoid logging credentials, access tokens, personal data, or full request bodies.

## 7. Turn on useful compiler diagnostics

Template type checking catches many mistakes before runtime. Keep strict template checking enabled in maintained applications. Angular extended diagnostics can warn about suspicious but syntactically valid template patterns, such as an uninvoked signal or an unused standalone import.

Diagnostics are version-sensitive and new checks may appear in minor releases. Treat warnings as actionable review items, and configure severity deliberately rather than suppressing all diagnostics. See [Angular extended diagnostics](https://angular.dev/extended-diagnostics).

## 8. Use source maps with care

Source maps make bundled JavaScript easier to relate to original TypeScript while debugging. Local development usually provides suitable mappings automatically. For deployed applications, decide whether source maps should be publicly accessible, privately uploaded to an error-monitoring service, or omitted. A source map can reveal source structure and names, so publication is a security and intellectual-property decision.

A diagnostic build should match the release configuration closely enough to reproduce the issue. Keep a record of the commit and build settings associated with the map; mismatched maps can point to the wrong source lines.

## A practical debugging loop

1. Reproduce the issue and capture the exact steps.
2. Check the console and network panel for the first error or failed request.
3. Inspect the responsible component and its state in Angular DevTools.
4. Add a breakpoint or a narrowly scoped log at the state transition.
5. Record a profiler trace only if the issue concerns slowness or repeated work.
6. Fix the cause at the layer that owns it.
7. Repeat the original steps and confirm the behavior.
8. Remove temporary logs and avoid retaining sensitive diagnostic data.

## Key takeaways

- Angular DevTools explains Angular structure and execution; browser DevTools adds DOM, network, JavaScript, and browser performance context.
- A short, stable reproduction is more useful than broad, noisy instrumentation.
- Use profiling to find expensive work before optimizing.
- Development debugging hooks are not a reason to expose an unoptimized production build.
- Handle expected failures locally and report unexpected errors with useful, privacy-conscious context.
- Keep strict template checks and review extended diagnostics as Angular versions evolve.

## Further reading

- [Angular DevTools](https://angular.dev/tools/devtools)
- [Angular DevTools Profiler](https://angular.dev/tools/devtools/profiler)
- [Inspect signals in Angular DevTools](https://angular.dev/tools/devtools/signals)
- [Angular performance best practices](https://angular.dev/best-practices/performance)
- [Profiling with Chrome DevTools](https://angular.dev/best-practices/profiling-with-chrome-devtools)
- [Angular error handling](https://angular.dev/best-practices/error-handling)
- [Extended diagnostics](https://angular.dev/extended-diagnostics)