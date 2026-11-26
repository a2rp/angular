# Measure Performance, Bundle Size, and Core Web Vitals

Performance work starts with measurement. A slow page can be limited by network delivery, JavaScript download and execution, server response, rendering, images, or interaction handlers. Identify the user-visible symptom and its cause before choosing an optimization.

This chapter introduces bundle measurement and Core Web Vitals. Later chapters focus on lazy loading, rendering behavior, and server rendering.

## Learning outcomes

After this lesson, you should be able to:

- Distinguish lab measurements from field measurements.
- Explain LCP, INP, and CLS and their current good thresholds.
- Read Angular CLI build output and set meaningful bundle budgets.
- Identify whether startup size, images, network, or main-thread work is the likely bottleneck.
- Use browser performance tools and field data together.
- Re-measure after each targeted optimization.

## 1. Define the user-visible problem

Begin with the experience that is slow or unstable. Examples include:

- The primary content appears late after navigation.
- The page downloads a large amount of JavaScript before it becomes usable.
- A button responds late after it is clicked.
- Content moves when an image or font loads.
- One route is much slower than others.

Measure the affected route and interaction, including the device class and network conditions. A development server and a fast laptop can hide issues that users experience on mobile devices.

Do not treat every metric as interchangeable. A large transfer can delay first render, but a poor interaction can be caused by long JavaScript tasks after loading. A visual shift can happen even when the page loads quickly.

## 2. Lab data and field data answer different questions

**Lab data** comes from a controlled test run, such as a local browser recording or Lighthouse. It is useful for reproducing a scenario, comparing changes, and finding a likely cause. Its results depend on the simulated or local device, cache, browser, and network.

**Field data** comes from real page visits and reflects the range of devices, networks, routes, and user interactions. It helps determine whether users experience a problem at scale. Field data can be reported through real-user monitoring, the Chrome User Experience Report, and tools that consume field data.

Use lab data to investigate; use field data to understand real user experience. A good lab run does not guarantee good field performance, and a single lab score is not a diagnosis.

## 3. Core Web Vitals

The current Core Web Vitals describe loading, responsiveness, and visual stability:

| Metric | What it measures | Good threshold |
| --- | --- | --- |
| Largest Contentful Paint (LCP) | Time until the main visible content is rendered | 2.5 seconds or less |
| Interaction to Next Paint (INP) | Responsiveness across user interactions | 200 milliseconds or less |
| Cumulative Layout Shift (CLS) | Unexpected visual movement during a page's lifetime | 0.1 or less |

Evaluate these metrics at the 75th percentile of real visits, generally segmented by mobile and desktop. A value for one test run is not the site's field result.

LCP can be affected by server response, render-blocking resources, image delivery, and when the main content is discovered. INP can be affected by long JavaScript tasks and expensive work triggered by an interaction. CLS can be affected by media without reserved space, late content insertion, and font changes.

See Google's current Web Vitals overview for metric definitions and thresholds.

## 4. Measure a representative page

Choose a route that users actually visit and record a predictable scenario:

1. Use a production build or a performance test environment that resembles production.
2. Record the browser, device, network, cache state, route, and actions.
3. Capture a cold page load when investigating startup.
4. Capture a representative interaction when investigating responsiveness.
5. Note both the measured symptom and browser trace evidence.
6. Repeat the same scenario after a change.

Chrome DevTools Performance can show JavaScript tasks, rendering, layout, paint, network timing, and Angular-specific activity where enabled. Angular DevTools and browser profiling were covered in the previous chapter. The Angular performance guide maps common symptoms to tools and techniques.

## 5. Read Angular CLI build output

Build the application using its production configuration:

~~~sh
ng build --configuration production
~~~

Inspect the output for the initial JavaScript and CSS required to start the app, as well as separate lazy-loaded bundles. Compare the same application configuration across commits. A total artifact size alone can hide a growing initial bundle or a particularly large feature chunk.

A size increase is a clue, not proof that users will see a regression. Consider whether the code is loaded at startup, how often the route is visited, compression, caching, and runtime execution.

## 6. Add bundle budgets

Angular CLI budgets set size boundaries in the workspace build configuration. For example, a production configuration can define an initial budget:

~~~json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "configurations": {
            "production": {
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "300kb",
                  "maximumError": "450kb"
                }
              ]
            }
          }
        }
      }
    }
  }
}
~~~

Replace the project name and thresholds with values appropriate for the application. The initial budget covers JavaScript and CSS needed for application startup. Other budget types can cover all scripts, individual files, component styles, or a named bundle. A named bundle budget is useful for watching a substantial feature chunk.

Treat budgets as guardrails that prompt investigation. Set thresholds from a measured baseline and the product's constraints; arbitrary tiny limits can create noisy warnings, while overly generous limits fail to catch regressions. Review the Angular CLI build documentation for the supported budget types and configuration shape.

## 7. Investigate bundle growth

When a budget warns or a bundle grows unexpectedly:

- Compare the current build with the previous known-good build.
- Check whether a new dependency is included in the startup path.
- Look for duplicate libraries or imports of large utility modules.
- Check whether a route or feature can be loaded later.
- Confirm that development-only tooling is not part of production.
- Inspect generated chunk names and the dependency graph provided by the current build tooling.

Avoid optimizing by package name alone. A dependency may be small, shared, or already tree-shaken. Measure its effect in the built application.

## 8. Connect symptoms to likely causes

| Symptom | First evidence to inspect |
| --- | --- |
| Main content appears late | LCP element, document response, critical requests, image priority, render-blocking CSS |
| Initial download is large | Initial bundle summary, production budgets, startup dependencies |
| A click or keypress feels slow | INP or interaction trace, long tasks, event-handler work, synchronous computation |
| Content jumps after rendering | CLS entries, image dimensions, inserted content, font loading |
| One route is slow | Route-specific chunk, its API requests, rendering cost, and route-level field data |
| Build output suddenly grows | Commit diff, dependency changes, chunk composition, budget warnings |

These are starting points. Confirm the cause with measurements before changing architecture or adding another dependency.

## 9. Re-measure and document the result

A useful performance change has a stated hypothesis and a comparable measurement:

1. Capture a baseline for the affected route or interaction.
2. Identify one likely source of cost.
3. Make one focused change.
4. Re-run the same lab scenario.
5. Check field data when enough real visits are available.
6. Confirm the fix did not worsen another metric or route.

Record the device, browser, build, network profile, metric, and percentile alongside the result. This makes the comparison understandable later.

## Key takeaways

- Measure the specific user-visible problem before selecting an optimization.
- Use lab data to diagnose repeatable scenarios and field data to understand real visits.
- Current Core Web Vitals are LCP, INP, and CLS, evaluated at the 75th percentile.
- Inspect Angular CLI's initial and lazy bundle output, then use budgets to catch growth.
- Budgets are investigation guardrails; tune them from real baselines.
- Repeat comparable measurements after the change.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="109-angular-devtools-debugging-and-production-diagnostics.md">109 - Angular DevTools, Debugging, and Production Diagnostics</a></td><td><a href="../qnas/110-measure-performance-bundle-size-and-core-web-vitals.md">Q&amp;A</a></td><td><a href="../examples/110-measure-performance-bundle-size-and-core-web-vitals.md">Example</a></td><td><a href="111-lazy-loading-and-deferrable-views-with-defer.md">111 - Lazy Loading and Deferrable Views with defer</a></td>
</tr>
  </tbody>
</table>
