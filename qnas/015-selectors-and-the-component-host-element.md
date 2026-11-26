# Q&A: Selectors and the Component Host Element

## Selectors and matching

### 1. What is a component selector?

It is the selector in @Component metadata that tells Angular which template element should instantiate the component.

### 2. How does Angular match component selectors?

Angular matches selectors against template elements at compile time. It does not continuously search the live DOM for matching elements.

### 3. Can adding a class or attribute with JavaScript create an Angular component?

No. Changing the live DOM with ordinary JavaScript or a runtime binding does not cause Angular to compile and instantiate a component that was not part of the template.

### 4. Are Angular component selectors case-sensitive?

Yes. Match the case used in component metadata.

### 5. Which selector types are supported?

Angular supports element-name, attribute, and class selectors, along with the :not pseudo-class and certain combinations of those forms.

### 6. Can a selector match an attribute's exact value?

Yes. An attribute selector can match a specific exact value, such as [type="reset"]. Angular does not support the other CSS attribute value operators such as ^=, $=, or *= in component selectors.

### 7. Can I use a descendant selector in @Component metadata?

No. Component selectors do not support CSS combinators such as descendant or child selectors.

### 8. Can I use any CSS pseudo-class in a component selector?

No. Angular supports :not for narrowing a selector, but does not support arbitrary pseudo-classes or pseudo-elements there.

### 9. Can I provide multiple selectors for one component?

Yes. A comma-separated selector list can match more than one selector form. Angular creates the component for each template element matching any one of those alternatives.

### 10. Can two components match the same element?

No. Angular permits only one component instance per element. If multiple component selectors match the same element, Angular reports an error.

### 11. Should most components use a custom element selector?

Yes. A custom element name is the recommended default for most components. Use a hyphen and a consistent project prefix, for example app-profile-card.

### 12. Why should a custom component selector contain a hyphen?

Custom element names are expected to contain a hyphen by the HTML standard. It also helps distinguish the component from a built-in HTML element.

### 13. Why shouldn't I use the ng- prefix?

Angular reserves ng- for its own framework APIs. Use a project-specific prefix such as app- or another consistent team prefix.

### 14. When might an attribute selector be useful?

It can be useful when a component is designed to enhance a native element, for example a button[appAction] component that retains the native button host and its standard behavior.

### 15. What is a drawback of an attribute selector?

A misspelled custom attribute may not produce the same unknown-element error as a misspelled custom element tag. Check the selector spelling and make sure the component is in the template scope.

### 16. Does an Angular component selector work like a CSS selector in a stylesheet?

No. Both use selector-like syntax, but Angular component selectors support only a limited subset and are used for compile-time component matching. CSS stylesheet selectors are used to match rendered elements for styling.

## Host elements and component views

### 17. What is a component host element?

It is the template element that matches the component's selector and gets the component instance. The component's view is rendered inside the host.

### 18. Is the host element the same as the root element inside the component template?

No. The host is outside the component's template markup. The template's elements are rendered inside the host.

### 19. Does an Angular component host appear in the DOM?

Yes. The element matching the component selector remains as the host element, and the component's template content is rendered inside it.

### 20. What is the host element of the root component?

It is the element in the application's index.html that matches the root component's selector, commonly app-root.

### 21. Where do attributes written on a component tag go?

They are attributes or bindings on the component's host element. They are not automatically placed on an inner element in the component template.

### 22. What does :host mean in component CSS?

It selects the host element of the component whose stylesheet contains that rule. It is useful for styling the component wrapper itself.

### 23. Does :host select the first element inside the template?

No. It selects the host element outside the template content. Use an ordinary CSS selector for an element defined inside the component template.

## Host metadata and accessibility

### 24. What can the host metadata property configure?

It can set static host attributes and declare bindings for host properties, attributes, classes, styles, and events.

### 25. Should I use @HostBinding and @HostListener in new code?

Angular recommends the host metadata property for new code. The decorators are supported for backwards compatibility with existing code.

### 26. When should I put a binding in host metadata?

Use it when the value or behavior belongs to the host element itself, such as its accessible role, state class, or keyboard handler. Use template bindings for elements inside the component's template.

### 27. Can host bindings conflict with bindings on the component tag?

Yes. If both the host metadata and the consuming template dynamically bind the same property or attribute, they can compete. Avoid overlapping dynamic bindings and give each value a clear owner.

### 28. Should every custom component host get an ARIA role?

No. Use native semantics where possible. Add a role only when the custom host needs that semantic and the chosen role accurately describes its behavior.

### 29. Is a custom element selector automatically accessible?

No. The selector only identifies the component. Ensure the host and its view provide appropriate semantics, labels, keyboard behavior, and focus behavior for their purpose.

### 30. Should I make a clickable custom element instead of using a button?

Usually no. Use a native button for button actions and a native link for navigation. Native elements provide expected keyboard and accessibility behavior.

### 31. Does a host role affect an inner element in the template?

No. A role set in host metadata is placed on the host element. If a nested element needs a role, configure that element in the component template instead.

## Troubleshooting and design

### 32. Why does a custom element appear unknown to Angular?

Check that its selector spelling matches the component metadata and that the component is available in the template scope through standalone imports or the relevant NgModule.

### 33. Why does a component with an attribute selector not render?

Check that the attribute selector is present in the consumer template in the form Angular expects and that the component has been imported into that template's scope.

### 34. Why does my attribute component replace or change the host's inner content?

A component has its own template, which renders within the matched host. If the component needs to render consumer-provided content, use Angular content projection deliberately; it is covered in a later lesson.

### 35. Can component selector matching respond to a runtime state change?

No. Selector matching is static at compile time. Use Angular template control flow to conditionally render a component based on state.

### 36. How can I inspect a component's host in the browser?

Open Developer Tools and inspect the Elements panel. Find the custom element or native element that matched the selector, then inspect its attributes and the child DOM rendered by its template.

### 37. Are host selectors and component selectors interchangeable?

No. The component selector chooses which host element instantiates the component. The host metadata configures bindings on that element after the component is instantiated.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="014-standalone-components-and-template-imports.md">014 - 14. Standalone components and template imports</a></td><td><a href="../lessons/015-selectors-and-the-component-host-element.md">Lesson</a></td><td><a href="../examples/015-selectors-and-the-component-host-element.md">Example</a></td><td><a href="016-angular-templates-and-html-structure.md">016 - 16. Angular templates and HTML structure</a></td>
</tr>
  </tbody>
</table>
