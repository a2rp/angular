# Semantic HTML, Keyboard Access, and Accessible Angular UI

Accessible interfaces provide meaningful structure, predictable interaction, and feedback that works for people using different input methods and assistive technologies. Angular templates produce HTML, so native web semantics are the foundation. Angular bindings can keep names, states, and messages synchronized as the interface changes.

Accessibility is not a final ARIA pass. Choose the right element, expose the interaction model, preserve visible focus, and verify the experience with real keyboard use and assistive technology.

## Learning outcomes

After this lesson, you should be able to:

- Choose native HTML elements that match user intent.
- Structure pages with landmarks, headings, labels, and useful names.
- Make interactive Angular controls work from the keyboard.
- Bind changing ARIA state to component state.
- Manage focus when dialogs, routes, and dynamic content change.
- Announce important updates without overwhelming screen readers.
- Review accessibility beyond automated checks.

## 1. Start with semantic HTML

Use a button for an action and a link for navigation. Use headings to describe sections, lists for lists, labels for controls, and main, nav, header, and footer landmarks where they fit the page structure.

A clickable div does not automatically behave like a button. It does not provide the same keyboard activation, role, focus behavior, or disabled state. Reuse native HTML rather than rebuilding platform behavior.

~~~html
<a href="/orders">View orders</a>
<button type="button" (click)="saveChanges()">Save changes</button>
~~~

Avoid placeholder text as the only label. A visible label remains available after a user starts typing and helps everyone understand the field.

## 2. Give controls accessible names and descriptions

A form control needs a programmatic name. The most reliable default is a label associated with the input:

~~~html
<label for="email">Email address</label>
<input id="email" type="email" autocomplete="email" />
~~~

Use aria-label only when visible text is unavailable, and use aria-labelledby when an existing visible element supplies the name. Do not assign roles and names that contradict the native element.

Descriptions can be connected with aria-describedby. Error messages should identify the field and explain how to fix the problem. A required field should be indicated in text or through the form's instructions, not only by color or an asterisk.

## 3. Keep state and ARIA attributes synchronized

When a control changes state, the template should update its corresponding semantics. A disclosure button can expose whether its panel is expanded:

~~~html
<button
  type="button"
  [attr.aria-expanded]="isOpen()"
  aria-controls="help-panel"
  (click)="toggleHelp()"
>
  Help
</button>

@if (isOpen()) {
  <section id="help-panel" aria-labelledby="help-heading">
    <h2 id="help-heading">Help</h2>
    <p>Instructions for this task.</p>
  </section>
}
~~~

ARIA supplements native semantics; it does not make an otherwise incorrect interaction accessible. Prefer native controls and add ARIA only when a state or relationship is not already communicated.

## 4. Make keyboard behavior complete

Every interactive control must be reachable and usable with a keyboard. Preserve a visible focus indicator. Use a logical tab order and avoid positive tabindex values, which create a separate order from the document.

Do not add custom key handling when a native button, link, checkbox, or select already provides the desired behavior. Composite patterns such as menus, comboboxes, and tablists need deliberate arrow-key, Escape, selection, and focus behavior. Use a tested library or Angular's accessible directives where appropriate rather than inventing a pattern casually.

## 5. Manage focus when the interface changes

When a dialog opens, move focus to an appropriate element and keep interaction within the modal while it is open. When it closes, return focus to the control that opened it if that control still exists.

After route navigation, decide where focus should move so keyboard and screen-reader users know the main content changed. A common approach is to focus the new page heading or main region after navigation. Do not move focus on every small data update; unexpected focus changes can disorient users.

For dynamically inserted content, focus it only when the user action and interaction pattern call for it. A new notification in the background should usually be announced rather than stealing focus.

## 6. Announce important asynchronous updates

An aria-live region can announce status text that appears after an asynchronous action. Choose the politeness level and update frequency carefully. A short save confirmation can be polite. A repeated progress counter may overwhelm users if announced every moment.

For complex announcements, use Angular CDK's LiveAnnouncer where it fits. Keep the visible message and the announcement in sync. Do not announce content that is already obvious and continuously changing unless it helps the user.

## 7. Use semantic status and error messages

Use role=status for non-urgent status updates and role=alert for important errors that need prompt announcement. These roles are not styling hooks. Do not attach alert semantics to large sections of content or every field error.

For form validation:

- Associate each field with its label.
- Connect help and error text with aria-describedby.
- Set aria-invalid when a value is invalid.
- Explain the error in text and provide a correction path.
- Avoid clearing the user's input after a validation failure.

## 8. Review accessible images and icons

Provide meaningful alternative text when an image contributes information. Use empty alt text for decorative images so they can be ignored by assistive technology. Icon-only buttons need an accessible name, even if the icon is visually clear.

If a decorative icon sits inside a button that already has visible text, avoid an extra repeated accessible name. Keep the control's name concise and consistent with its visible label.

## 9. Treat accessibility as a continuous review

Automated tools can catch some missing names, invalid relationships, and contrast concerns. They cannot determine whether the interaction makes sense, whether focus moves appropriately, or whether spoken feedback is understandable.

Review pages with:

- Keyboard only, including visible focus and a logical sequence.
- Browser zoom and narrow viewport widths.
- A screen reader for key user journeys.
- Automated checks and accessibility linting.
- Realistic validation, loading, empty, error, and success states.

Angular's accessibility guide covers ARIA, native elements, and CDK tools. Angular Aria provides headless interaction patterns for supported controls; use those when a custom widget needs more than basic native HTML.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="102-internationalization-localization-and-formatting.md">102 - Internationalization, Localization, and Formatting</a></td><td><a href="../qnas/103-semantic-html-keyboard-access-and-accessible-angular-ui.md">Q&amp;A</a></td><td><a href="../examples/103-semantic-html-keyboard-access-and-accessible-angular-ui.md">Example</a></td><td><a href="104-angular-testing-fundamentals-and-the-cli-test-runner.md">104 - Angular Testing Fundamentals and the CLI Test Runner</a></td>
</tr>
  </tbody>
</table>
