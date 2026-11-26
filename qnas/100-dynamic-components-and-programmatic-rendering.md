# Dynamic Components and Programmatic Rendering — Q&A

## 1. When should I use dynamic component rendering?

Use it when the component type is not known in the template until runtime or when code must insert, replace, or remove Angular views programmatically. If the component is known and always belongs on the page, a normal component element is simpler.

## 2. How do I choose between @defer and dynamic rendering?

Use @defer when the component is known but its code should load later. Use NgComponentOutlet or ViewContainerRef when the component type itself must be selected at runtime. Dynamic rendering is not automatically a better lazy-loading technique.

## 3. What is NgComponentOutlet useful for?

It renders a component type supplied by a template expression. It is useful when the choice is declarative and its required configuration fits the outlet API. Keep the possible component types in an explicit, validated registry.

## 4. What does ViewContainerRef do?

It represents a location in Angular's view tree where embedded views or component host views can be inserted. A component created through that container is attached at the container's location and managed as part of its view hierarchy.

## 5. What is the difference between ViewContainerRef.createComponent and createComponent?

ViewContainerRef.createComponent inserts the host view into the container automatically. The standalone createComponent API returns a ComponentRef without attaching it to an existing view location; the caller must attach the host element and view and own cleanup. Use the latter only when that extra control is needed.

## 6. How should I provide input values to a dynamically created component?

Use ComponentRef.setInput for a dynamic component input. It lets Angular process the input through its normal input and change-detection behavior. Newer Angular versions also have binding helpers for programmatic rendering; check the API for the exact version in use.

## 7. How should I respond to a dynamic component's output?

Subscribe to the output exposed by the component instance or use a supported output-binding helper. Keep the subscription under an explicit owner and unsubscribe or destroy the component when it is removed.

## 8. What happens when ViewContainerRef.clear is called?

It removes and destroys the views managed by that container. This is useful when a single dynamic view should be replaced. Do not call it before every insertion if the container is supposed to keep multiple components.

## 9. How do I destroy one dynamically created component?

Keep its ComponentRef and call destroy when the component is no longer needed. Destroying the reference tears down the component and its view. Clear the container when all of its current views should be removed.

## 10. Is it safe to select a component from a server-provided string?

Do not turn an untrusted string into an import path or executable component. Map validated data to a finite set of application-owned component types. A server may provide a known widget identifier, but the application must enforce an allowlist.

## 11. How should dynamic lazy loading handle failures?

Handle dynamic import failure and show a useful fallback or retry action. A loaded component can also fail during construction or rendering. Use an appropriate error boundary or error handling mechanism and confirm the available API for the Angular release you use.

## 12. Does programmatic rendering work with SSR and hydration automatically?

Angular rendering APIs participate in Angular's view model, but rendering outside the view tree or mutating raw DOM can lead to server/client mismatches. Verify dynamic components on server-rendered routes and give every view an explicit owner and cleanup policy.

## 13. Can I create an unlimited number of dynamic components?

You can, but unbounded creation can retain views, subscriptions, DOM nodes, and memory. Decide whether new components append, replace, or reuse old ones, and destroy them when they are no longer needed.

## 14. What does a ComponentRef represent?

It is a handle to a dynamically created component instance and its host view. It provides ways to set inputs, inspect the instance, access the location and change detector, and destroy the component.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="099-reusable-ui-components-and-design-system-boundaries.md">099 - Reusable UI Components and Design-System Boundaries</a></td><td><a href="../lessons/100-dynamic-components-and-programmatic-rendering.md">Lesson</a></td><td><a href="../examples/100-dynamic-components-and-programmatic-rendering.md">Example</a></td><td><a href="101-global-error-handling-logging-and-diagnostics.md">101 - Global Error Handling, Logging, and Diagnostics</a></td>
</tr>
  </tbody>
</table>
