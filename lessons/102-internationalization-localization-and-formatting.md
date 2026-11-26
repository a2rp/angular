# Internationalization, Localization, and Formatting

Internationalization prepares an application to support multiple languages and regional conventions. Localization adapts the interface for a specific locale: translated text, date and number formats, currency display, direction, and cultural expectations.

Translation is not just replacing words. A layout that fits English may overflow in another language, and a date or number that looks familiar in one locale may be ambiguous in another.

## Learning outcomes

After this lesson, you should be able to:

- Distinguish internationalization from localization.
- Mark template text and code strings for translation.
- Add context and stable IDs that help translation teams.
- Use Angular's extraction and localized-build workflow.
- Format dates, numbers, and currencies using locale data.
- Handle plural forms, text expansion, and right-to-left layouts.
- Decide whether the product needs build-time localization or runtime language switching.

## 1. Prepare messages for translation

Mark static template text with Angular's i18n attribute. Mark translatable attributes such as image alt text with an i18n-prefixed attribute.

~~~html
<h1 i18n="page title|Heading on the account page">
  Account settings
</h1>

<img
  src="/assets/profile.svg"
  alt="Profile"
  i18n-alt="profile illustration|Alternative text for the profile illustration"
/>
~~~

Use a description to clarify context. The word "Open" could be a verb on a button or an adjective describing a status. Add translator context when the meaning is not obvious. A custom ID can keep a message identified across source-text edits, but IDs need a maintenance policy.

In TypeScript, mark user-facing strings with Angular's $localize tagged message when the package and build setup supports it:

~~~ts
const savedMessage = $localize`:notice|Shown after preferences are saved:Your preferences were saved.`;
~~~

The metadata supplies translator context; the text after the second colon is the source message. Prefer keeping user-facing copy in templates where possible.

Do not assemble translated sentences by concatenating fragments. Word order and grammar vary. Mark the full sentence and use named placeholders for values when needed.

## 2. Extract and translate messages

A practical Angular build-time workflow is:

1. Add Angular's localize package to the application if it is not already configured.
2. Mark template text, attributes, and code messages.
3. Run ng extract-i18n to create or update a source translation file.
4. Give translators context and the source meaning.
5. Translate the file for each target locale.
6. Configure the locale and translation file in the workspace.
7. Build or serve the localized application and review the result.

Use the translation file format and configuration supported by your Angular CLI. Treat source text and translations as maintained assets; re-extract and review changes when messages are added or altered.

See Prepare component for translation, Work with translation files, and Merge translations into the app.

## 3. Understand build-time and runtime localization

Angular's built-in localization workflow can create localized application builds for configured locales. A deployment can serve the matching locale version, often using a locale-specific URL. This is a good fit when the application can select a locale before loading the app.

A user-controlled instant language switch without a full localized application load is a different requirement. Choose a runtime translation system and design how it loads catalogs, updates text, caches bundles, and handles missing keys. Do not assume changing LOCALE_ID alone translates template strings.

Keep translation and formatting in agreement. The selected language, number conventions, currency, date, timezone, text direction, and accessibility labels should represent one coherent locale choice.

## 4. Format dates, numbers, and currencies

Angular's formatting pipes use the application locale for their default output. A locale determines conventions such as decimal separators, grouping, and date order.

~~~html
<p>{{ amount | number: '1.2-2' }}</p>
<p>{{ completion | percent: '1.0-0' }}</p>
<p>{{ createdAt | date: 'mediumDate' }}</p>
<p>{{ amount | currency: 'USD' }}</p>
~~~

Currency formatting communicates presentation, not financial conversion. Store monetary values in an appropriate numeric representation and use a currency code. Converting one currency to another requires an exchange-rate source and explicit business rules.

Dates also require a timezone decision. A date-only concept such as a birthday is not the same as a timestamp such as a payment event. Avoid shifting a date-only value across timezones by treating it as a local timestamp without a clear policy.

Angular's built-in data pipes use locale information. See Format data based on locale. When overriding the app locale, also ensure the required locale data is registered.

## 5. Translate plural and alternate forms as whole messages

Plural rules differ between languages. Do not choose an English singular or plural suffix in TypeScript. Use a translation-aware plural message with all categories needed by the translation system, including a fallback.

A message such as "You have 3 messages" should be translated as a whole expression. The translator may need to change word order or use more plural categories than English. The same applies to select expressions and gendered or status-dependent alternatives.

## 6. Design for text expansion and direction

Translated text may be substantially longer or shorter than its source. Allow buttons, menus, cards, and tables to grow. Avoid clipping labels or using fixed widths for text unless the design explicitly handles overflow.

Use logical CSS properties such as margin-inline-start instead of assuming left always means the start. Verify right-to-left layout, icons with directional meaning, form alignment, navigation order, and mixed-direction text. Not every icon should be mirrored; arrows representing physical direction may need different treatment than decorative marks.

Review line wrapping, truncation, headings, screen-reader announcements, and keyboard navigation in each supported locale. Check long text and pseudolocalized strings before release.

## 7. Avoid common localization failures

- Do not concatenate translatable sentence fragments.
- Do not hardcode date, number, or currency conventions in display strings.
- Do not assume every language has singular and plural only.
- Do not use a country as a substitute for a language or locale identifier.
- Do not infer translation success from a successful build alone.
- Do not forget accessible labels, validation messages, tooltips, and title or alt attributes.
- Do not use localized display text as a database key or internal status value.

## 8. Choose a locale strategy deliberately

Before adding localization infrastructure, define supported locales, URL and language-switch behavior, catalog ownership, translation review, number and currency policy, timezone behavior, right-to-left requirements, and fallback rules.

Angular's built-in localization is suitable for marked messages and localized builds. Runtime switching, user-specific catalogs, or frequent remote translation updates may require a dedicated runtime localization solution. Choose based on the product requirement and document how it affects loading and caching.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="101-global-error-handling-logging-and-diagnostics.md">101 - Global Error Handling, Logging, and Diagnostics</a></td><td><a href="../qnas/102-internationalization-localization-and-formatting.md">Q&amp;A</a></td><td><a href="../examples/102-internationalization-localization-and-formatting.md">Example</a></td><td><a href="103-semantic-html-keyboard-access-and-accessible-angular-ui.md">103 - Semantic HTML, Keyboard Access, and Accessible Angular UI</a></td>
</tr>
  </tbody>
</table>
