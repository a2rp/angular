# Measure Performance, Bundle Size, and Core Web Vitals — Q&A

## 1. Why should I measure before optimizing?

A slow page can be caused by a large startup bundle, slow server response, image loading, expensive JavaScript, layout, or an API request. Each cause needs a different fix. Measurement helps avoid making unrelated changes that add complexity without improving the user experience.

## 2. What are Core Web Vitals?

Core Web Vitals are Google's field metrics for loading performance, responsiveness, and visual stability. The current metrics are Largest Contentful Paint (LCP), Interaction to Next Paint (INP), and Cumulative Layout Shift (CLS).

## 3. What counts as a good LCP?

A good LCP is 2.5 seconds or less. LCP measures when the largest visible content element is rendered. The metric can be affected by the server response, critical resources, images, and how soon the browser discovers the main content.

## 4. What counts as a good INP?

A good INP is 200 milliseconds or less. INP measures responsiveness across user interactions. Slow event handlers, expensive synchronous work, long tasks, or delayed rendering can contribute to poor INP.

## 5. What counts as a good CLS?

A good CLS is 0.1 or less. CLS measures unexpected visual movement. Images or embeds without reserved dimensions, content inserted above existing content, and late font changes are common sources of layout shift.

## 6. How are Core Web Vitals evaluated for a site?

They are evaluated using the 75th percentile of real page visits, commonly segmented by mobile and desktop. A result from one Lighthouse run or one test device is useful evidence, but it is not the field percentile for all users.

## 7. What is the difference between lab and field data?

Lab data is collected in a controlled test environment. It is useful for repeatable debugging and comparing changes. Field data comes from real visits and captures the range of real users, devices, networks, and interactions. Use lab data to investigate and field data to assess actual user experience.

## 8. Does a high Lighthouse score guarantee good user performance?

No. A lab score describes a particular test run and configuration. Real users can have different devices, networks, route choices, cached assets, and interaction patterns. Field data is needed to understand whether real visits meet the target.

## 9. Why does the Angular CLI show a bundle size budget warning?

A configured bundle crossed its warning threshold. A budget is a size guardrail that identifies growth; it does not prove a user-visible regression. Inspect which bundle grew and whether it is part of startup or only loaded for a particular feature.

## 10. What does the initial bundle budget measure?

The Angular CLI initial budget covers JavaScript and CSS needed to bootstrap the application. A large initial bundle can delay the app becoming available. Lazy-loaded route or feature bundles should be examined separately because users may not download them during the initial visit.

## 11. How should I choose budget thresholds?

Start from the current production build, product requirements, and a desired growth margin. Set warning and error thresholds that surface meaningful increases. Avoid thresholds so small that every legitimate change causes noise, or so large that substantial regressions pass unnoticed.

## 12. Can a smaller bundle still produce a slower application?

Yes. Bundle size is only one factor. A smaller bundle can still run expensive computations, wait on a slow API, or cause layout shifts. Conversely, a larger lazy-loaded bundle may not affect initial startup if it is only downloaded after navigation.

## 13. What should I do when the initial bundle grows?

Compare the current build with a known-good build. Inspect new imports and dependencies, duplicate code, and whether a feature is included in startup unnecessarily. Consider splitting an infrequently used feature into a later chunk. Verify the change in the production build output.

## 14. What should I inspect when LCP is poor?

Identify the actual LCP element and inspect the document response, critical CSS and scripts, image request, and when the browser discovers that resource. If the LCP element is an image, verify it is sized correctly and prioritized appropriately. Angular's NgOptimizedImage can help apply image loading best practices.

## 15. What should I inspect when INP is poor?

Record the slow interaction in browser performance tools. Inspect the event handler, synchronous JavaScript, long tasks, and rendering work that follows the input. Angular DevTools can help identify Angular change-detection costs, while Chrome Performance can show the broader browser timeline.

## 16. What should I inspect when CLS is poor?

Use field or lab diagnostics to identify the layout-shift sources. Check for images, video, ads, embeds, and asynchronous content without reserved space. Also inspect font loading and elements inserted above existing content.

## 17. Should I optimize every route equally?

Prioritize routes and interactions that matter to users and have evidence of poor performance. A rarely visited administrative route may not deserve the same work as a public landing page or a core checkout flow. Measure route-specific behavior when possible.

## 18. Does development build output represent production bundle size?

No. Use the production build configuration when evaluating deployed bundle size. Development builds include different tooling and optimizations and are designed for development rather than production size measurement.

## 19. What does a bundle budget error mean for CI?

It means the build crossed an error threshold and the configured build target failed. This can help prevent accidental size regressions. Teams should agree on the threshold and review changes that add substantial code rather than blindly increasing the budget.

## 20. Are build size values the same as transferred compressed size?

Not necessarily. CLI output and budget values are based on the builder's reported output sizes and may not represent the exact bytes transferred after server compression and caching. Use browser network measurements or deployment telemetry for transfer behavior.

## 21. How do I compare two performance runs fairly?

Keep the route, build type, browser, device profile, network conditions, cache state, and user actions as similar as possible. Capture a baseline, change one likely cause, then repeat the same scenario. Use field data when enough visits have accumulated.

## 22. What is the role of Angular DevTools in performance work?

Angular DevTools can help find expensive change-detection cycles and components. It is one piece of evidence. Use browser Performance traces to see network, JavaScript, rendering, and other work that Angular's profiler does not describe by itself.

## 23. How can image optimization improve performance?

Correct dimensions can prevent layout shifts. Responsive sources can reduce unnecessary downloads. Prioritizing the actual LCP image can help it load sooner, while non-critical images can be lazy-loaded. Angular's NgOptimizedImage provides defaults and checks for common practices.

## 24. What should I record when reporting a performance regression?

Record the route, build and commit, device class, browser, network profile, reproduction steps, metric and percentile if available, and relevant trace. For field results, include the date range and sample context. Do not present one lab result as a universal field score.

## 25. What is the right performance workflow?

1. Define the user-visible problem.
2. Measure the affected route or interaction.
3. Identify the likely cause in a trace or build report.
4. Make one targeted change.
5. Repeat the same lab measurement.
6. Check field data when available.
7. Add or adjust budgets only with a clear baseline and purpose.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="109-angular-devtools-debugging-and-production-diagnostics.md">109 - Angular DevTools, Debugging, and Production Diagnostics</a></td><td><a href="../lessons/110-measure-performance-bundle-size-and-core-web-vitals.md">Lesson</a></td><td><a href="../examples/110-measure-performance-bundle-size-and-core-web-vitals.md">Example</a></td><td><a href="111-lazy-loading-and-deferrable-views-with-defer.md">111 - Lazy Loading and Deferrable Views with defer</a></td>
</tr>
  </tbody>
</table>
