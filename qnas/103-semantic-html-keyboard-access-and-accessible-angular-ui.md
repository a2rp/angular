# Semantic HTML, Keyboard Access, and Accessible Angular UI — Q&A

## 1. Why should I prefer native HTML controls?

Native buttons, links, inputs, and selects already provide semantics, keyboard behavior, focus behavior, and browser support. A custom element must recreate those correctly. Use the native element that matches the action.

## 2. When should I use a link and when a button?

Use a link when activating it navigates to a location. Use a button when it performs an action on the current page, such as saving or opening a panel. Styling should not change the semantic purpose.

## 3. Is placeholder text a label?

No. Placeholder text disappears or becomes harder to see after typing and is not a reliable substitute for a persistent label. Associate a visible label with the input.

## 4. When should I use aria-label?

Use it when the control has no visible text that provides a suitable name, such as an icon-only button. If visible text already exists, let it name the control or associate it with aria-labelledby rather than repeating the same words.

## 5. Does ARIA make an incorrect HTML control accessible?

No. ARIA can expose names, roles, states, and relationships, but it does not recreate all native interaction behavior. Start with appropriate HTML and add ARIA only when needed.

## 6. How should I expose an expanded or collapsed state?

Bind the appropriate ARIA state to the same application state that controls the content. For a disclosure, aria-expanded should reflect whether the panel is open, and aria-controls can reference the panel when that relationship is useful.

## 7. What does a visible focus indicator do?

It shows keyboard users which control will receive input. Removing the browser outline without an equally visible replacement makes navigation harder. Ensure focus remains obvious across themes and contrast modes.

## 8. Should I use positive tabindex values to control tab order?

Generally no. Positive values create an order separate from the DOM and can confuse keyboard users. Arrange elements in a meaningful document order and use normal focusability.

## 9. When should focus move after a route change?

Decide where the new page begins and move focus to an appropriate target, often the page heading or main content, after navigation. Do not move focus for every background data update; it can disorient users.

## 10. How should a modal dialog manage focus?

When it opens, move focus to a meaningful control and constrain focus within the modal when required by the interaction pattern. When it closes, return focus to the opening control if it still exists. Use a tested dialog implementation or CDK support rather than implementing all behavior ad hoc.

## 11. What is the difference between role=status and role=alert?

A status communicates a non-urgent change and is usually announced politely. An alert is for important information that should be announced promptly. Use them for meaning, not merely styling, and avoid making a whole large region announce every change.

## 12. When is an aria-live region useful?

It can announce asynchronous feedback such as a completed save or a short result count. Choose how frequently it updates. Repeated announcements of rapidly changing values can overwhelm the user.

## 13. How should form errors be connected to fields?

Give the control a persistent label. Associate help and error messages with aria-describedby, set aria-invalid when appropriate, and explain how to correct the error in text. Keep the user's input available after validation fails.

## 14. How should images and icons be named?

Provide useful alternative text for informative images and empty alt text for decorative images. Icon-only controls need a concise accessible name. If a decorative icon appears beside visible button text, avoid repeating the same name.

## 15. Are automated accessibility checks enough?

No. Automated tools can find some missing names, invalid attributes, and contrast issues. They cannot fully judge whether a workflow is understandable, spoken feedback is useful, or focus moves correctly. Combine automated checks with keyboard and screen-reader review.

## Further reading

- [Angular accessibility best practices](https://angular.dev/best-practices/a11y)
- [Angular Aria overview](https://angular.dev/guide/aria/overview)
- [Angular component guide](https://angular.dev/guide/components)
