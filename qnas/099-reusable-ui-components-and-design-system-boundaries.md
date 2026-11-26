# Reusable UI Components and Design-System Boundaries - Q&A

## 1. When should I create a reusable component?

Create one when several consumers share a stable concept, contract, and behavior. Repeated markup alone is not enough. If the component only makes sense inside one workflow, keep it in that feature until a broader use case is clear.

## 2. What should a component's public API contain?

Only the data, configuration, events, and content slots that consumers need. Include meaningful types, predictable defaults, accessible behavior, and documented styling choices. Treat inputs, outputs, projected content, and keyboard behavior as public API.

## 3. What is the difference between an input and an output?

An input supplies data or configuration from a parent to a child. An output reports an event from the child so the parent can decide what happens next. The child should not secretly mutate unrelated parent state or perform a feature workflow.

## 4. When is content projection useful?

Use content projection when a consumer supplies structured markup that a container should place into a known slot, such as a card title and body. It lets the consumer own the content while the component owns the surrounding structure and styling.

## 5. Can ng-content be conditionally created with template control flow?

No. Angular processes projection placeholders at compile time and projected content is instantiated even if a placeholder is visually hidden. Use a template-fragment or dynamic-component approach when content must be selected or instantiated at runtime.

## 6. Should a shared button implement feature-specific behavior?

Usually no. A shared button should provide a consistent accessible control and expose a small contract. The owning feature decides what saving, payment, or navigation action means and handles that workflow.

## 7. How do native HTML elements help accessibility?

Native buttons, links, inputs, and headings provide built-in semantics, keyboard behavior, and focus behavior. A custom clickable div must recreate much of that correctly. Prefer the native element that matches the user's action.

## 8. What accessibility behavior belongs to the shared component?

Its contract should state how it is named, operated by keyboard, focused, disabled, and announced to assistive technology. Components should preserve visible focus, appropriate semantics, and clear label relationships. Consumers should not have to repair basic accessibility each time they use it.

## 9. How should component styles be shared?

Use component-scoped styles for implementation details and design tokens or documented variants for intentional shared customization. Avoid asking consumers to target private internal DOM selectors because markup changes can break those styles.

## 10. Does Angular style encapsulation mean styles can never affect other components?

No. Component style encapsulation is a mechanism for scoping component styles, but global styles, inherited properties, CSS custom properties, and explicitly configured encapsulation still matter. Define which values are public tokens and keep internal selectors private.

## 11. When should a component go into a design-system library?

When multiple independent applications need the same documented contract and a team owns its compatibility, accessibility, visual language, and release process. A component used several times within one app may only need an application shared folder.

## 12. Why avoid many boolean inputs?

Many boolean inputs can create a large number of combinations, some of which have never been designed or tested. Prefer a small finite variant type when the choices are intentional, or separate components when the concepts differ.

## 13. Can changing HTML structure break a component API?

Yes. Consumers may rely on projected slots, style hooks, focus behavior, ARIA semantics, or documented defaults. Those are part of the supported contract even if TypeScript compilation succeeds.

## 14. How can I tell whether a generic component is too generic?

It often has many feature-specific flags, injected services, conditionals, or exceptions. If two uses require different meanings or interaction rules, split the component or keep it feature-owned instead of expanding one universal abstraction.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="098-organize-projects-libraries-and-shared-code.md">098 - Organize Projects, Libraries, and Shared Code</a></td><td><a href="../lessons/099-reusable-ui-components-and-design-system-boundaries.md">Lesson</a></td><td><a href="../examples/099-reusable-ui-components-and-design-system-boundaries.md">Example</a></td><td><a href="100-dynamic-components-and-programmatic-rendering.md">100 - Dynamic Components and Programmatic Rendering</a></td>
</tr>
  </tbody>
</table>
