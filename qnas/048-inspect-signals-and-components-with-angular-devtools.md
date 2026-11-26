# Inspect signals and components with Angular DevTools — Q&A

## 1. What is Angular DevTools?

It is a browser extension for inspecting and profiling a running Angular application. Its panels can show the component tree, reactive signal graph, change-detection profile, injector hierarchy, and router hierarchy.

## 2. Does Angular DevTools edit my source files?

No. Its component properties panel can change some values in the live debugging session, but those experiments do not rewrite the source files. Normal application logic may also replace an edited value later.

## 3. Which build should I inspect?

Use a development build. Production optimization removes debug information required by Angular DevTools, so the extension cannot provide its normal component inspection features for a production-optimized build.

## 4. Why does ng serve usually work with DevTools?

The Angular CLI development server uses development settings by default. This keeps the debug information needed by DevTools available during local development.

## 5. Where do I find the Angular panel?

Open the browser developer tools while viewing a page running your Angular app, then select the Angular tab. The exact browser shortcut depends on the operating system and browser.

## 6. Why is there no Angular tab on a browser's new-tab page?

The new-tab page may not run installed extensions in the usual way. Navigate to a regular page with the running Angular application and inspect that tab instead.

## 7. What does the Components tab show?

It shows a hierarchical view of component and directive instances associated with the rendered application. Selecting an entry displays available properties and metadata.

## 8. Can I search a large component tree?

Yes. Use the tree's search box to find component or directive instances by name.

## 9. How can I select a component directly from the page?

Use the inspect-element control in Angular DevTools, then point to an element in the rendered application. DevTools can identify associated Angular components or directives.

## 10. How do I connect a component to its DOM node?

Select the component and use the host-node navigation action. The browser opens its Elements or Inspector panel and selects the associated DOM node.

## 11. Can DevTools navigate to component source?

It can offer source navigation for a selected component when source information is available in the development build. The action may be unavailable when that location cannot be determined.

## 12. Can I edit a property in the Components tab?

Some values can be edited in the live debugging session. Treat this as a temporary experiment to understand behavior. To test normal user behavior, reproduce the change with the app's own control.

## 13. What does the signal graph show?

It visualizes reactive nodes associated with the selected component or directive, such as signals, computed values, effects, linked signals, resources, and template reads.

## 14. What do the edges in the signal graph mean?

They show reactive dependencies. An upstream node is a value read by another node; a downstream node is a consumer that depends on it.

## 15. Which Angular versions support the signal graph?

The current guide requires the application to be built with Angular 19 or later and running in development mode. The basic Components panel may work even when the signal graph is unavailable.

## 16. Why is the signal graph action missing?

Check the app's Angular version, build mode, and DevTools version. Also select a component or directive and verify that the application is running in development mode.

## 17. Why does a signal not appear in the graph I selected?

The graph is scoped to the selected component or directive and its reactive relationships. Select the owner or consumer that actually reads the signal, then reproduce the state that activates conditional dependencies.

## 18. What are conditional signal dependencies?

A computed value can read different signals depending on its current branch. Angular tracks the signals read in the latest computation, so a branch change can add or remove dependencies. The signal graph can help inspect that relationship.

## 19. Why give signals or effects debug names?

Descriptive debug names make graph nodes easier to identify. This is especially useful for effects that are not naturally named through a class property.

## 20. What can I learn from the Profiler?

It records Angular execution during a reproduced interaction and shows change-detection cycles and the components or directives involved. It helps locate where time is being spent.

## 21. How do I record a profile?

Open the Profiler, start recording, reproduce the interaction, and stop recording. Select a relevant cycle to inspect its component costs or hierarchical visualization.

## 22. What does a tall cycle bar mean?

It represents a cycle that took longer than shorter bars in that recording. Select the bar to inspect which components and directives contributed to the recorded work.

## 23. What does the flame graph help me understand?

It shows execution in the render-tree hierarchy. Use it to locate a costly area and inspect its parent and child relationships; a highlighted component is a clue for investigation, not automatically the root cause.

## 24. Should I optimize every component that appears in a profile?

No. Start with the interaction that feels slow and inspect the relevant cycle. Focus on the specific component, directive, template expression, or lifecycle work associated with the cost.

## 25. Are DevTools timings identical to production timings?

No. Angular DevTools depends on development-mode debug features and does not profile a production-optimized Angular build. Use it to understand development execution and compare changes under the same setup; use production-like measurements when evaluating production performance.

## 26. Can I save and share a profile?

Yes. Angular DevTools can export a recorded profile as JSON and import a saved recording later.

## 27. What is the Injector Tree for?

It visualizes Environment and Element injector hierarchies and providers. Use it to inspect where a dependency is provided and why a component may receive a particular service instance.

## 28. Can the Injector Tree help diagnose duplicated service state?

Yes. Inspect the providers associated with the relevant injector levels. A service registered at separate component providers can have separate instances; compare the tree with the scope the application intended.

## 29. What does the Router Tree show?

It shows the application's runtime routing structure. It can help connect a rendered screen with the route hierarchy.

## 30. What is the Transfer State panel?

A Transfer State panel may appear when the application uses hydration. Its availability depends on the application setup and tool version.

## 31. What does “Angular application not detected” mean?

DevTools cannot communicate with an Angular app in the inspected page. Confirm that the correct tab is open, the app loaded successfully, and the page actually contains an Angular application.

## 32. Why can the Components tab work while the signal graph does not?

The signal graph has a newer version and development-mode requirement. The general component tree has different prerequisites, so check the requirements of the specific missing feature.

## 33. Should I disable production optimizations on a deployed site to use DevTools?

No. Run a development build locally or in an appropriate non-production environment. Do not weaken a public production build just to enable debugging features.

## 34. How can DevTools help with an unexpected computed value?

Select the component that consumes it, inspect the property, and trace its upstream dependencies in the signal graph. Reproduce the state that causes the unexpected result and check which branch and source values are active.

## 35. How can DevTools help when an OnPush view appears stale?

Inspect the component's inputs and signals, then reproduce the interaction. A profile can show which components participated in a cycle, while the component and signal views help identify the relevant data path. Compare the result with the application's update logic.

## 36. Which official Angular pages cover these features?

- Angular DevTools overview
- Inspect the component tree
- Debug the signal graph
- Profile your application

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="047-immutable-state-updates-for-arrays-and-objects.md">047 - Immutable state updates for arrays and objects</a></td><td><a href="../lessons/048-inspect-signals-and-components-with-angular-devtools.md">Lesson</a></td><td><a href="../examples/048-inspect-signals-and-components-with-angular-devtools.md">Example</a></td><td><a href="049-services-and-the-injectable-pattern.md">049 - Services and the injectable pattern</a></td>
</tr>
  </tbody>
</table>
