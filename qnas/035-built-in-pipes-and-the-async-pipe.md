# 35. Q&A: Built-in pipes and the async pipe

## Pipe fundamentals

### 1. What is an Angular pipe?

A pipe is a declarative transformation used in a template expression. It receives the value to the left of `|`, applies a named transformation, and renders the result. It does not change the value stored in the component.

### 2. Is the Angular pipe operator the JavaScript bitwise OR operator?

No. In Angular templates, `|` introduces a pipe. Angular template expressions do not support JavaScript's bitwise OR operator.

### 3. How do I pass arguments to a pipe?

Put a colon after the pipe name and provide the argument. Separate additional arguments with more colons: `{{ amount | currency:'USD':'symbol':'1.2-2' }}`. Argument order is defined by that pipe's API.

### 4. In what order do chained pipes run?

From left to right. For `{{ name | lowercase | titlecase }}`, `lowercase` runs first and `titlecase` receives its output.

### 5. How do I avoid pipe-precedence surprises?

Use parentheses when combining a pipe with a conditional or another operator. For example, `{{ (isReady ? 'Ready' : 'Waiting') | uppercase }}` clearly sends the conditional result to `uppercase`.

### 6. Why does a pipe I used in a standalone component show as unknown?

A standalone component must import each pipe used by its own template. Import the pipe class from `@angular/common` and add it to the component's `imports` array, such as `imports: [CurrencyPipe, DatePipe]`.

### 7. How do pipes become available in an NgModule component?

Import `CommonModule` into the NgModule that declares the component. `CommonModule` exports Angular's common pipes and template directives. Do not import `BrowserModule` into feature modules.

### 8. Which built-in pipes does Angular provide?

The common built-ins are `async`, `currency`, `date`, `number` (`DecimalPipe`), `i18nPlural`, `i18nSelect`, `json`, `keyvalue`, `lowercase`, `percent`, `slice`, `titlecase`, and `uppercase`. Their classes are exported from `@angular/common`.

### 9. Does a pipe update the component property it receives?

No. The pipe transforms a value for display. For example, `{{ price | currency }}` renders formatted text but leaves `price` as its original number. Keep domain data in its proper type and format it at the presentation boundary.

## Formatting and locale

### 10. How does `DecimalPipe` work in a template?

Use its template name, `number`, for example `{{ score | number:'1.1-2' }}`. The digits string is `{minIntegerDigits}.{minFractionDigits}-{maxFractionDigits}`. Its defaults are one integer digit, zero minimum fractional digits, and up to three fractional digits.

### 11. Does `PercentPipe` expect `25` for 25 percent?

No. It expects a ratio, so use `0.25` to represent 25 percent. Passing `25` means 2,500 percent. Use `digitsInfo` when the displayed fractional precision needs to be controlled.

### 12. How should I format a currency amount?

Pass the amount's ISO 4217 code, such as `USD`, `EUR`, or `INR`: `{{ total | currency:'USD':'symbol':'1.2-2' }}`. The optional display can be `code`, `symbol`, `symbol-narrow`, or a custom string. Avoid relying on an implicit currency when the currency is part of the business data.

### 13. Is a currency symbol enough to identify the currency?

Not always. Symbols can be shared or vary by locale. Keep the currency code with the amount in application data and specify the code in the formatting pipe when the user needs an unambiguous value. Choose `code` display where clarity matters more than a compact symbol.

### 14. Which locale do Angular's formatting pipes use?

They use `LOCALE_ID` unless a pipe supports and receives an explicit locale argument. Angular defaults to `en-US`. Locale affects separators, grouping, date patterns, currency presentation, and plural rules.

### 15. How do I set a non-default locale?

Import and register the locale data you need with `registerLocaleData`, then provide its identifier through `LOCALE_ID`. A pipe's explicit locale argument can override the global choice where supported, but the corresponding locale data still needs to be registered.

### 16. Does setting `LOCALE_ID` translate my interface text?

No. It configures locale-sensitive formatting and related locale rules. Translating labels, sentences, and other interface content is a separate localization task.

### 17. What date values can `DatePipe` format?

It accepts a JavaScript `Date`, a number of milliseconds since the Unix epoch, or an ISO date string. Prefer a well-defined timestamp or ISO value; ambiguous strings can be interpreted differently before they reach the pipe.

### 18. Does `DatePipe` use UTC by default?

No. Without an explicit timezone or configured default, it uses the end user's local timezone. Pass a timezone such as `'UTC'` if the display requirement calls for a consistent zone: `{{ timestamp | date:'yyyy-MM-dd HH:mm':'UTC' }}`.

### 19. Why did changing a `Date` object not refresh its formatted text?

`DatePipe` is pure and observes a changed primitive or object reference. Mutating the same `Date` instance does not change its reference. Assign a new `Date` instance when the date changes.

### 20. What does `i18nPlural` do?

It selects a message from a mapping based on a numeric value and the locale's plural rules. Exact keys such as `=0` can override plural categories; `other` is the general fallback. The `#` marker in the selected message is replaced with the number.

### 21. What does `i18nSelect` do?

It chooses a string from a mapping based on an input key. Provide an `other` entry for values without a matching key. Use it for small template-level selection; use a translation workflow for complete localized messages.

## Other built-in pipes

### 22. What does `SlicePipe` return?

`slice` returns the selected part of a string or array using start and optional end indexes, following JavaScript `slice` behavior. The end index is excluded, negative indexes count from the end, and an array result is a copy. The source array is not modified.

### 23. What does `KeyValuePipe` accept and how does it order entries?

It accepts an object or `Map` and returns entries with `key` and `value`. By default it sorts by key using Unicode point order. Pass a comparator for a custom ordering, or pass `null` to preserve the input's natural order.

### 24. Should I use `JsonPipe` in a production page?

Usually no. `json` is intended for inspecting a value during development. It is not a stable API format, and exposing an object can reveal fields that should not be shown to users.

### 25. When should I use `titlecase`?

Use it for simple presentation text when Angular's title-case transformation is appropriate. Do not use it to rewrite stored names, brand spellings, or culturally specific person names; those need intentional text and localization rules.

## Pure pipes and change detection

### 26. What is a pure pipe?

A pure pipe runs when Angular detects a changed input value or reference. It avoids repeating a transformation while its inputs remain unchanged. Most built-in pipes are pure.

### 27. Why did a pure pipe not notice an array mutation?

The array still has the same reference. Prefer immutable updates, such as `this.items = [...this.items, item]`, when a pure pipe or other reference-based change detection must observe a collection change.

### 28. Which built-in pipes are impure?

`AsyncPipe`, `JsonPipe`, `KeyValuePipe`, and `SlicePipe` are marked impure. They need to react to asynchronous emissions or changes that are not represented solely by a new input reference. Impure pipes can run during change detection, so avoid expensive work in templates.

### 29. Does a pure pipe cache every value it has ever formatted?

No. Purity lets Angular skip the transformation while the current input values and references are unchanged. It is not a general-purpose memoization cache over historical inputs.

### 30. Should I write an impure pipe to detect every object mutation?

Usually not. Impure pipes can be reevaluated during change detection and cause repeated work. Prefer immutable state updates or an explicit derived value. Build an impure pipe only when its behavior is justified and its cost is understood.

## AsyncPipe

### 31. What can `AsyncPipe` consume?

It can consume an Observable or a Promise. It unwraps and renders the latest emitted or resolved value, so a template can display asynchronous data without a manual subscription just for rendering.

### 32. What happens before the first async value arrives?

`AsyncPipe` can return `null` before its source emits or resolves. Provide a loading state or otherwise make the template safe for a null value.

### 33. Does `AsyncPipe` unsubscribe automatically?

Yes. It cleans up when its view is destroyed. If the bound expression changes to a different Observable or Promise reference, it stops using the old source and follows the new one.

### 34. Does `AsyncPipe` update an `OnPush` component?

Yes. When the source emits, the pipe marks its component to be checked. That makes it suitable for templates using `OnPush` and for Angular's supported zoneless change-detection notification flow.

### 35. Does `AsyncPipe` show a loading or error message automatically?

No. It exposes the async value and manages its subscription. The template or application state must decide how to present loading, empty, and error states; handle source errors in the appropriate stream or application error flow.

### 36. Why should an Observable be stored in a field instead of created by a template method?

A template method can run repeatedly during change detection and may return a new Observable reference each time. `AsyncPipe` then switches subscriptions to follow that reference. Keep the source stable in component state, a signal, or a service.

### 37. Can I write `{{ data$ | async }}` in several places?

You can, but each `async` pipe occurrence manages its own subscription. With a cold Observable, those subscriptions can start separate executions. Unwrap the source once and reuse the value with a template variable when it is needed in multiple places.

### 38. Is `@if (data$ | async; as data)` safe for every value type?

It is useful when an emitted value is guaranteed to be truthy, but the `@if` condition tests truthiness. Valid values such as `0`, `false`, or `''` can choose the fallback branch. Use a template pattern that distinguishes null/not-yet-loaded from valid false-like data.

### 39. Should I manually subscribe instead of using `AsyncPipe`?

For rendering an Observable or Promise value, `AsyncPipe` usually keeps template code simpler and handles cleanup. Subscribe in TypeScript when the component needs imperative side effects or orchestration that belongs outside the template, and manage that subscription's lifecycle deliberately.

### 40. Can I use a pipe class to format data in a service?

Pipes are designed for template expressions. For non-template formatting, use appropriate functions exported by `@angular/common`, such as `formatDate`, `formatCurrency`, `formatNumber`, or `formatPercent`, with the required locale data.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="034-global-styles-css-variables-and-theming.md">034 - 34. Global styles, CSS variables, and theming</a></td><td><a href="../lessons/035-built-in-pipes-and-the-async-pipe.md">Lesson</a></td><td><a href="../examples/035-built-in-pipes-and-the-async-pipe.md">Example</a></td><td><a href="036-custom-pipes-attribute-directives-and-reusable-behavior.md">036 - 36. Custom pipes, attribute directives, and reusable behavior</a></td>
</tr>
  </tbody>
</table>
