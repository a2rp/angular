# Inspect signals and components with Angular DevTools

Angular DevTools is a browser extension for inspecting and profiling a running Angular application. It provides a live view of the component tree, component and directive properties, signal dependencies, dependency-injection scopes, routes, and change-detection work.

DevTools helps you investigate what the application is doing. It does not replace reading the code or verifying user-visible behavior. Use it to form and check a hypothesis: find the relevant component or signal, reproduce the interaction, and inspect the resulting state or profile.

## Learning outcomes

After this lesson, you should be able to:

- Open Angular DevTools for a running development build.
- Inspect a component, directive, its properties, and its place in the component tree.
- Trace reactive dependencies in the signal graph.
- Record and interpret a change-detection profile.
- Use the Injector Tree and Router Tree to investigate application structure.
- Recognize common reasons the extension cannot detect an application.
- Treat DevTools edits and timing data as debugging aids, not production source of truth.

## Install and open the extension

Angular DevTools is available as an extension for supported Chromium browsers and Firefox. Install it from the browser's extension store, open the browser's developer tools, and choose the Angular tab while viewing a page that runs your Angular application.

A blank browser tab or a page without an Angular application will not show a useful component tree. Keep the application running in a development build while inspecting it. A normal Angular CLI development server started with ng serve uses development settings by default.

The information panel in Angular DevTools reports the Angular and DevTools versions detected on the page. Check it when a feature is missing: available panels and capabilities depend on the application and tool versions.

## Components tab: inspect the rendered application

The Components tab shows a hierarchy of component and directive instances associated with the rendered application.

A useful inspection flow is:

1. Expand the tree and select the component that owns the behavior you are investigating.
2. Read the properties panel to inspect its inputs, outputs, and other available state.
3. Use the tree search when the application has many components.
4. Use the inspect-element control to pick a component or directive from the page.
5. Open the host node in the browser's Elements or Inspector panel when you need to connect Angular state to rendered DOM.
6. Use source navigation when the development build provides a source location.

The properties panel can also edit some values in the live debugging session. Treat edits as temporary experiments: they do not edit source files, and application logic may replace the value on a later update. Prefer reproducing changes through the application's own controls when investigating normal behavior.

The tree can show deferred view blocks and, in hydrated applications, hydration information. The exact details depend on Angular and DevTools versions.

## Signal graph: follow reactive dependencies

Select a component or directive in the Components tab and open its signal graph. The graph visualizes reactive nodes associated with that component, including signals, computed values, effects, linked signals, resources, and template reads when supported by the application.

Read the graph as a dependency map:

- An upstream node is a value that the selected node reads.
- A downstream node is a value or consumer that depends on the selected node.
- The template node indicates which reactive values participate in rendering that component.
- Selecting a signal node can show its current value and available debugging actions.

The graph is useful for questions such as:

- Which source signals contribute to this computed value?
- Which template or effect reads this signal?
- Did a conditional computation stop depending on a signal after its branch changed?
- Which consumers might be affected when this source value changes?

Give important signals, computed values, and especially effects meaningful debug names where the API supports it. Descriptive names make a graph easier to read than anonymous nodes.

The signal graph requires an application built with Angular 19 or later and running in development mode. If the graph action is missing, check the app version, build mode, and DevTools version before assuming the signal code is broken.

## Profiler: find work before optimizing

The Profiler records Angular execution while you reproduce an interaction. Start a recording, perform the interaction that feels slow, and stop the recording. Angular DevTools displays change-detection cycles and details about components and directives involved in the selected cycle.

A disciplined profiling workflow is:

1. State the symptom precisely, such as “changing the filter makes the task list pause.”
2. Start recording before the interaction.
3. Reproduce the same interaction once or a small number of times.
4. Stop recording and select the relevant cycle.
5. Compare component costs and inspect a bar chart or flame graph.
6. Investigate the largest relevant cost in the code, then profile again after a change.

Each cycle appears as a bar; taller bars represent longer recorded cycles. The component view and flame graph help locate work in the component hierarchy. A component taking time is a clue, not by itself proof of the cause. The expensive work may be a template expression, a lifecycle hook, a directive, or a descendant.

A flame graph's hierarchy shows where execution occurred in the render tree. Use the profile to identify a specific slow path rather than optimizing every component that appears.

DevTools relies on development features and is not available for a production-optimized Angular build. Use its profile to understand development behavior and compare changes under the same setup; use browser performance tools and production-like measurements when evaluating production performance.

Profiles can be saved as JSON and imported later, which is useful for sharing or comparing a captured investigation.

## Injector Tree and Router Tree

The Injector Tree visualizes Environment and Element injector hierarchies and lets you inspect providers at different levels. Use it to investigate questions such as:

- Which injector provides this service?
- Why does one component receive a separate service instance?
- Which provider is visible from this component?

The Router Tree displays the application's route structure and helps connect a rendered view with the route configuration. A Transfer State panel may also appear when the application uses hydration.

These tools show configured runtime structure. They do not decide which provider scope or route design is correct; compare what you see with the intended ownership and navigation behavior.

## Common detection problems

### Angular application not detected

Confirm that the inspected tab is displaying the running Angular application and that the app loaded successfully. A page without Angular, a browser's new-tab page, or the wrong tab will not produce an Angular component tree.

### DevTools reports a production configuration

Run a development build for component inspection and signal-graph debugging. Production optimizations remove debug information that the extension needs. Do not disable production optimizations on a public deployment just to make DevTools connect.

### A panel or action is missing

Check the Angular version, DevTools version, build mode, browser, and feature prerequisites. For example, the signal graph has a newer version requirement than the basic component tree. Update the extension when appropriate and verify that the app is not running a production-optimized build.

### The graph does not show a value I expected

Check whether the selected component actually reads the value, directly or through a computed value. A signal that is never read by the selected component's template or reactive graph may not appear as an expected dependency. Select the component that owns or consumes the signal and reproduce the state that activates conditional dependencies.

## Practical debugging checklist

Before concluding that Angular is rendering incorrectly:

- Inspect the correct browser tab and confirm the Angular app is running.
- Select the component that owns the value, then inspect its inputs and state.
- Reproduce the issue instead of changing values arbitrarily in the DevTools panel.
- Use the signal graph to follow dependencies when a reactive value is unexpected.
- Use the profiler to locate measurable work before changing performance-sensitive code.
- Check injector scope when a service appears to have duplicate state.
- Confirm the build mode and feature version if a panel is unavailable.
- Reproduce the behavior again after a fix.

## Key takeaways

- Angular DevTools inspects a live Angular app; it does not edit source code.
- The Components tab connects component instances, properties, and DOM nodes.
- The signal graph shows reactive dependencies and requires Angular 19+ in development mode.
- The Profiler records change-detection work so you can investigate a measured interaction.
- Injector and Router trees expose runtime structure for debugging.
- Missing panels usually call for checking the inspected page, build mode, version, or feature prerequisites.
- Find the specific behavior or cost first, then make and verify a focused change.

## Further reading

- [Angular DevTools overview](https://angular.dev/tools/devtools)
- [Inspect the component tree](https://angular.dev/tools/devtools/component)
- [Debug the signal graph](https://angular.dev/tools/devtools/signals)
- [Profile your application](https://angular.dev/tools/devtools/profiler)
- [Angular performance guide](https://angular.dev/best-practices/performance)
