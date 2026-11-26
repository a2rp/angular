# Q&A: Accessible labels, instructions, and validation messages

These questions review how to give form controls clear names, associate instructions and errors, and present Angular validation state accessibly. Read the [lesson](../lessons/069-accessible-labels-instructions-and-validation-messages.md) for the complete guide and build the [example](../examples/069-accessible-labels-instructions-and-validation-messages.md) to see the patterns in a standalone Reactive Form.

## Labels and names

### 1. What is the purpose of a form label?

A label identifies what information a control expects. It gives the control an accessible name and helps people understand the form visually.

### 2. How do you explicitly associate a label and input?

Give the input a unique **id** and set the label's **for** attribute to exactly that value.

### 3. What happens when a user activates a correctly associated label?

The browser focuses or activates its associated control. This also provides a larger click target for small controls such as checkboxes and radio buttons.

### 4. Can a control be nested inside its label?

Yes. Wrapping the control in a **label** is an implicit association. Use a clear label structure and do not also wrap another labelable control inside that label.

### 5. Is a placeholder a replacement for a label?

No. Placeholder text can disappear while typing and does not reliably supply the control's accessible name. Keep a label and use a placeholder only as optional supplementary guidance.

### 6. Is the **title** attribute a good general replacement for a visible label?

No. It is not a dependable substitute for a visible, associated label. Use a visible label whenever possible.

### 7. Does **formControlName** automatically label a reactive input?

No. **formControlName** connects the DOM control to Angular's form model. The HTML template still needs accessible naming and description relationships.

### 8. Why must IDs be unique?

A label or ARIA reference resolves an ID to an element. Duplicate IDs can make it point to the wrong control or description.

### 9. How should radio options that form one question be grouped?

Use a **fieldset** with a **legend** for the shared question and give every radio option its own label.

### 10. Can checkboxes also use fieldset and legend?

Yes, when they are related choices under one question or heading. Each checkbox still needs its own label.

## Instructions and required fields

### 11. What kinds of instructions should a form provide?

Explain relevant requirements such as input format, length, allowed values, required or optional status, and other constraints before users need that information.

### 12. Where should whole-form instructions go?

Place them before the form or near its heading so they are available before the user starts entering values.

### 13. Where should field-specific instructions go?

Place them close to the field and associate them with **aria-describedby** when the extra text is important to understanding or completing that control.

### 14. What does **aria-describedby** do?

It associates one or more descriptive elements with a control. Assistive technology can announce that description in addition to the control's accessible name.

### 15. How do you reference more than one description?

Provide the referenced IDs as a space-separated string, such as **aria-describedby="email-help email-error"**.

### 16. Must an **aria-describedby** target exist at all times?

Each referenced ID should resolve to the intended element while it is referenced. When an error is conditional, ensure the error element is present whenever that error is being described.

### 17. How should a required field be identified?

Make required status clear in text, such as “Email address (required),” and use the HTML **required** attribute. Apply the matching Angular validator when using Reactive Forms.

### 18. Does **Validators.required** add the HTML **required** attribute automatically?

No. The Angular validator controls Angular's validity. The template's HTML **required** attribute provides browser semantics; keep both aligned.

### 19. Is an asterisk enough to indicate a required field?

Not by itself. Explain what the symbol means, and make required status understandable without relying only on a symbol, color, or visual convention.

### 20. Can a form say “All fields required” instead of marking every label?

A clear form-level instruction can help, especially when most fields share the same rule. Each field's status should still be understandable when encountered, and exceptions should be clearly marked.

## Angular validation messages

### 21. Which Angular states are commonly used to decide when to show an error?

Use the control's **invalid** state together with an interaction state such as **touched**, **dirty**, or a form-level **submitted** flag.

### 22. Why not show all required errors on initial page load?

The model may correctly be invalid before the user interacts with it, but showing every error immediately can make the page noisy. Show instructions first and reveal errors at an intentional point.

### 23. What does **markAllAsTouched()** do during a submit attempt?

It marks every descendant control touched so the UI can reveal validation messages consistently after submission is attempted. It does not make any control valid.

### 24. When should **aria-invalid** be set to true?

When the UI has identified an error and is presenting it to the user. A blank untouched required field does not always need to be announced as an error immediately on page load.

### 25. How can Angular bind **aria-invalid**?

Use an attribute binding such as **[attr.aria-invalid]="showError(control) ? 'true' : null"**. Returning **null** removes the attribute when no presented error exists.

### 26. Should an error be indicated only by a red border?

No. Color can reinforce the problem, but provide text that identifies the issue and gives useful correction guidance.

### 27. What makes an error message useful?

It identifies which value needs attention and how to correct it. “Enter a valid email address, for example name@example.com” is more actionable than “Invalid.”

### 28. Should the same error be associated through multiple ARIA mechanisms?

Avoid duplicate descriptions that may be read twice. Choose a clear primary association, such as **aria-describedby**, and ensure the message is available where the user needs it.

### 29. What should the UI show while an async validator is pending?

A concise status that explains the check is still in progress. Do not announce a final validation result until it finishes.

### 30. Is a pending control the same as an invalid control?

No. **PENDING** means the asynchronous result is unresolved. Treat pending as its own state instead of presenting it as either valid or invalid.

## Error summaries and announcements

### 31. When is a form-level error summary useful?

It is useful after a submit attempt in longer forms or when errors may be difficult to locate. A summary can list the problems and link to the affected fields.

### 32. Should every short form have an error summary?

No. Clear inline messages may be sufficient. Choose the approach that makes errors easiest to find without duplicating information.

### 33. What should happen to focus after a form is submitted with errors?

For a longer or complex form, move focus to the summary or first invalid field after it appears. This helps keyboard and screen-reader users find where to start correcting the form.

### 34. Should every changing validation message use **role="alert"**?

No. An assertive live announcement on each keystroke can interrupt users. Use inline descriptions for field context and reserve live announcements for meaningful state changes that need to be announced without a focus move.

### 35. What is a good use for **role="status"** or a polite live region?

Announce updates such as “Checking availability,” “Saving,” or a completed result when the user otherwise would not hear that state change.

### 36. Is **aria-describedby** itself a live announcement?

No. It associates descriptive text with a control. It is typically available when the user focuses the control; it does not automatically turn every update into a live notification.

### 37. Should a success or server-error message be associated with a field?

Only if it specifically describes that field. Form-level request results should be presented as form-level status or error feedback, not attached to an unrelated control.

## Repeated controls and testing

### 38. How should IDs work in a repeated Angular control list?

Generate unique IDs for each row and use the same row identity for its label, help text, and error message. Do not duplicate a fixed ID inside an **@for** loop.

### 39. What should be checked with a keyboard?

Confirm that each control is reachable, labels work, instructions and errors are available, submit feedback is findable, and focus moves sensibly after an invalid submission.

### 40. What should be checked with a screen reader or accessibility tree?

Confirm that each control has the intended name, role, and state; its instructions and active error are associated; grouped options have context; and status or error announcements are useful rather than repetitive.

### 41. Does accessible client-side validation replace server validation?

No. Client-side validation improves feedback, but the server must validate submitted data and enforce business rules.

### 42. What is the core checklist for an accessible Reactive Form?

Use meaningful visible labels, provide persistent instructions, associate help and errors with the correct inputs, reveal errors at a deliberate time, expose **aria-invalid** consistently, and make submit feedback easy to find and understand.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="068-submission-flows-pending-state-and-reset-behavior.md">068 - Submission flows, pending state, and reset behavior</a></td><td><a href="../lessons/069-accessible-labels-instructions-and-validation-messages.md">Lesson</a></td><td><a href="../examples/069-accessible-labels-instructions-and-validation-messages.md">Example</a></td><td><a href="070-custom-controls-with-controlvalueaccessor.md">070 - Custom controls with ControlValueAccessor</a></td>
</tr>
  </tbody>
</table>
