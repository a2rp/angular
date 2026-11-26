# Internationalization, Localization, and Formatting - Q&A

## 1. What is the difference between internationalization and localization?

Internationalization prepares an application so it can support multiple languages and regional conventions. Localization adapts it for a specific locale, including translated messages, formats, direction, and cultural expectations.

## 2. Does changing LOCALE_ID translate template strings?

No. LOCALE_ID affects locale-sensitive formatting, such as Angular date, number, and currency pipes. Template strings still need to be marked for translation and supplied with translated messages.

## 3. How do I mark template text for Angular translation?

Use the i18n attribute on the element whose static text should be translated. Use attributes such as i18n-title or i18n-alt for translatable attribute values. Add meaning or a description when translators need context.

## 4. How do I mark a string in component code?

Use Angular's $localize tagged message with the required package and build configuration. Prefer template messages when possible. Do not concatenate fragments of a sentence because word order can differ between languages.

## 5. What does ng extract-i18n do?

It extracts marked template text, attributes, and code messages into a source translation file. The team then maintains translated files and configures the desired localized builds. Extraction does not translate text automatically.

## 6. Why include message meaning and description?

Short messages can be ambiguous. A description tells translators where and how the text is used. A meaning distinguishes identical source words used differently. A stable custom ID can help retain identity across wording changes, but it needs a maintenance policy.

## 7. How are plurals translated?

Use a whole translation-aware plural expression and provide the categories required by the translation system, including a fallback. Do not append an English suffix or assume every locale has the same singular/plural rules.

## 8. Does Angular's built-in localization automatically switch language at runtime?

Angular's standard localization workflow supports configured localized builds. Selecting a different localized build may mean loading another locale-specific app. An instant user-controlled language switch generally needs a runtime localization strategy. Changing LOCALE_ID alone does not translate messages.

## 9. How do Angular date and number pipes choose a format?

They use locale information for their default output. That changes conventions such as date order, decimal separators, grouping, and currency symbol presentation. Explicitly override a locale only when the product requirement calls for it.

## 10. Does CurrencyPipe convert a value between currencies?

No. It formats a numeric value as a currency presentation. Currency conversion requires an exchange-rate source and business rules. Keep the currency code and amount semantics explicit.

## 11. Why should a date-only value be treated differently from a timestamp?

A date-only value such as a birthday represents a calendar day, while a timestamp represents a moment in time. Applying timezone conversion to a date-only value can shift it to a different day. Define storage and display semantics before formatting.

## 12. What should I check for right-to-left locales?

Check text direction, alignment, navigation order, form layout, icons with directional meaning, and mixed-direction strings. Use logical CSS properties where possible. Mirror only icons whose meaning is directional.

## 13. How can I test for text expansion?

Use long translations or pseudolocalized strings, then check wrapping, clipping, buttons, menus, tables, headings, and narrow layouts. A successful compile does not show whether the localized interface is usable.

## 14. Should localized display labels be stored as application keys?

No. Store stable internal codes, IDs, and data values. Translate display labels at the presentation boundary. This keeps business logic and persisted data independent from the active locale.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="101-global-error-handling-logging-and-diagnostics.md">101 - Global Error Handling, Logging, and Diagnostics</a></td><td><a href="../lessons/102-internationalization-localization-and-formatting.md">Lesson</a></td><td><a href="../examples/102-internationalization-localization-and-formatting.md">Example</a></td><td><a href="103-semantic-html-keyboard-access-and-accessible-angular-ui.md">103 - Semantic HTML, Keyboard Access, and Accessible Angular UI</a></td>
</tr>
  </tbody>
</table>
