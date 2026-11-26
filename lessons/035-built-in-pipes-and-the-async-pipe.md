# 35. Built-in pipes and the async pipe

Angular pipes transform values in template expressions. They are useful for presentation tasks such as formatting a date or currency, changing letter case, selecting part of a string or array, and displaying the latest value from an asynchronous source. A pipe keeps this presentation transformation close to the template without changing the value stored by the component.

## Learning goals

After this lesson, you should be able to:

- Explain pipe syntax, parameters, chaining, and operator precedence.
- Import pipes into a standalone component or make them available through `CommonModule` in an NgModule.
- Choose the appropriate built-in pipe for text, collections, debugging, dates, numbers, and currencies.
- Configure locale-aware output and understand locale-data requirements.
- Explain pure-pipe change detection and the behavior of the built-in impure pipes.
- Use `AsyncPipe` with an Observable or Promise and describe its subscription lifecycle.
- Avoid common formatting, mutation, subscription, and performance mistakes.

## What a pipe does

A pipe takes a value and returns a transformed value for display. In a template, the pipe operator is the vertical bar (`|`):

~~~html
<p>{{ amount | currency }}</p>
~~~

Angular evaluates `amount`, passes it to the `currency` pipe, and renders the returned string. A pipe does not rewrite `amount` in the component. If the application needs to store a normalized or calculated value, do that in the component, a computed signal, a service, or another appropriate layer.

Angular template pipe syntax is separate from JavaScript's bitwise OR operator. Angular template expressions do not support bitwise operators.

## Pipe parameters

Pass parameters after the pipe name with colons. Each additional parameter uses another colon:

~~~html
<p>{{ amount | currency:'USD':'symbol':'1.2-2' }}</p>
<p>{{ createdAt | date:'medium':'UTC' }}</p>
~~~

In an interpolation, the expression before the pipe is the input. The values after the pipe name are arguments in order. Use clear formatting and parentheses when an expression could be read in more than one way.

~~~html
<!-- The conditional result is passed into uppercase. -->
<p>{{ (isReady ? 'Ready' : 'Waiting') | uppercase }}</p>
~~~

The pipe operator has lower precedence than arithmetic, comparison, logical, and nullish-coalescing operators, and higher precedence than the conditional operator. Parentheses make intended grouping explicit.

## Chaining pipes

Pipes run from left to right. The output of one pipe becomes the input of the next:

~~~html
<p>{{ companyName | lowercase | titlecase }}</p>
<p>{{ scheduledAt | date:'medium' | uppercase }}</p>
~~~

Keep chains readable. If the sequence becomes difficult to understand, derive a clearly named presentation value in the component instead.

## Make a pipe available to a template

### Standalone components

Import each pipe used by the component from `@angular/common` and add it to the component's `imports` array:

~~~ts
import { Component } from '@angular/core';
import { CurrencyPipe, DatePipe, TitleCasePipe } from '@angular/common';

@Component({
  selector: 'app-purchase-summary',
  standalone: true,
  imports: [CurrencyPipe, DatePipe, TitleCasePipe],
  template: `
    <h1>{{ company | titlecase }} — {{ purchasedAt | date }}</h1>
    <p>Total: {{ amount | currency:'USD' }}</p>
  `,
})
export class PurchaseSummaryComponent {
  company = 'northwind supplies';
  purchasedAt = new Date();
  amount = 123.45;
}
~~~

Import only the directives and pipes the template needs. A standalone pipe import makes that pipe available to that component's template, not to every component in the application.

### NgModule-based applications

Import `CommonModule` into the NgModule that declares the component. `CommonModule` exports Angular's common pipes and template directives. `BrowserModule` imports `CommonModule` for the root browser module, but feature modules should import `CommonModule` themselves when needed.

Do not import `BrowserModule` into feature modules.

## Built-in pipes

Angular's built-in pipes are exported from `@angular/common`.

| Pipe | Template name | What it does |
| --- | --- | --- |
| `AsyncPipe` | `async` | Reads the latest value from a Promise or Observable and manages its subscription. |
| `CurrencyPipe` | `currency` | Formats a number as a currency using locale rules. |
| `DatePipe` | `date` | Formats a date value according to a format, timezone, and locale. |
| `DecimalPipe` | `number` | Formats a number with locale-aware grouping and decimal separators. |
| `I18nPluralPipe` | `i18nPlural` | Selects a message based on a number and pluralization mapping. |
| `I18nSelectPipe` | `i18nSelect` | Selects a message from a mapping based on a string key. |
| `JsonPipe` | `json` | Converts a value to JSON text, mainly for debugging. |
| `KeyValuePipe` | `keyvalue` | Converts an object or `Map` into key/value entries for display. |
| `LowerCasePipe` | `lowercase` | Converts a string to lowercase. |
| `PercentPipe` | `percent` | Formats a ratio as a locale-aware percentage. |
| `SlicePipe` | `slice` | Returns a subset of a string or array. |
| `TitleCasePipe` | `titlecase` | Converts text to title case. |
| `UpperCasePipe` | `uppercase` | Converts a string to uppercase. |

Choose a pipe for a specific view transformation. These pipes do not replace application-specific formatting rules or localization of full messages.

## Text pipes

`lowercase`, `uppercase`, and `titlecase` are convenient for short presentation transformations:

~~~html
<p>{{ username | lowercase }}</p>
<p>{{ status | uppercase }}</p>
<h2>{{ product.name | titlecase }}</h2>
~~~

Do not use `titlecase` as a substitute for carefully authored names, brand capitalization, or culturally aware person-name formatting. Text casing is presentation logic; it should not silently change the underlying data.

## Slice and key/value pipes

### `SlicePipe`

`slice` follows JavaScript's `slice` behavior for strings and arrays. Its `start` and optional `end` arguments define the range; the end index is excluded. Negative indexes count from the end.

~~~html
<p>{{ message | slice:0:20 }}</p>

<ul>
  @for (item of items | slice:0:5; track item.id) {
    <li>{{ item.name }}</li>
  }
</ul>
~~~

When the input is an array, `SlicePipe` returns a copy; it does not mutate the source array. It is an impure pipe so changes within a mutable array can be observed, but avoid using it repeatedly on large collections when filtering or pagination should instead be implemented as explicit state or derived data.

### `KeyValuePipe`

`keyvalue` converts an object or `Map` into entries with `key` and `value` properties. By default, entries are sorted by key using Unicode point order. Pass a comparator to choose a different ordering; pass `null` to preserve the input's natural order.

~~~html
<dl>
  @for (entry of settings | keyvalue; track entry.key) {
    <div>
      <dt>{{ entry.key }}</dt>
      <dd>{{ entry.value }}</dd>
    </div>
  }
</dl>
~~~

`KeyValuePipe` is impure. It is useful for a small object or map rendered as a list, but it is not a replacement for a well-typed array when the view needs a specific order or richer item behavior.

## Plural and string-selection pipes

`I18nPluralPipe` picks a message for a numeric value using exact-value entries such as `=0` and plural categories such as `one` and `other`. The `#` placeholder is replaced by the number:

~~~ts
readonly unreadCount = 3;
readonly unreadMessages = {
  '=0': 'No unread messages',
  '=1': 'One unread message',
  other: '# unread messages',
};
~~~

~~~html
<p>{{ unreadCount | i18nPlural:unreadMessages }}</p>
~~~

`I18nSelectPipe` picks a message for a string key and uses `other` when there is no matching key:

~~~ts
readonly deliveryStatus = 'shipped';
readonly deliveryMessages: Record<string, string> = {
  pending: 'The order is being prepared.',
  shipped: 'The order is on its way.',
  delivered: 'The order has arrived.',
  other: 'The delivery status is unavailable.',
};
~~~

~~~html
<p>{{ deliveryStatus | i18nSelect:deliveryMessages }}</p>
~~~

These pipes help with small template-level message selections. For a translated application, use Angular's localization workflow and keep complete, context-aware messages in translation resources rather than assembling sentences from fragments.
## JSON pipe for debugging

`json` converts a value to JSON-formatted text and is useful while inspecting a value in a development template:

~~~html
<pre>{{ selectedProduct | json }}</pre>
~~~

`JsonPipe` is impure and can run during change detection. Remove diagnostic output from user-facing templates and avoid treating this debug representation as a stable serialization or API format. It can reveal fields that should not be shown to a user.

## Locale-aware formatting pipes

`DatePipe`, `CurrencyPipe`, `DecimalPipe` (used as `number`), and `PercentPipe` format values according to locale data. The active locale comes from the `LOCALE_ID` injection token unless a pipe supports and receives an explicit locale argument. Angular's default locale is `en-US`.

### Decimal numbers

`number` accepts a `digitsInfo` argument with this structure:

~~~text
{minIntegerDigits}.{minFractionDigits}-{maxFractionDigits}
~~~

For example:

~~~html
<p>{{ score | number:'1.1-2' }}</p>
<p>{{ measurement | number:'3.0-0' }}</p>
~~~

The defaults are one integer digit, zero minimum fractional digits, and up to three fractional digits. Locale controls separators and grouping. The formatting operation rounds the value to the permitted precision; it does not provide financial decimal arithmetic.

### Percentages

`percent` treats a value as a ratio: `0.25` represents 25 percent.

~~~html
<p>{{ completionRatio | percent:'1.0-1' }}</p>
~~~

Passing `25` means 2,500 percent. Store ratios in the expected scale and format them with the pipe.

### Currency

`currency` accepts an ISO 4217 currency code, a display option, a `digitsInfo` string, and an optional locale:

~~~html
<p>{{ total | currency:'USD':'symbol':'1.2-2' }}</p>
<p>{{ total | currency:'EUR':'code':'1.2-2':'de-DE' }}</p>
~~~

The display can be `code`, `symbol`, `symbol-narrow`, or a custom string. For money, pass the currency explicitly instead of relying on a locale's default currency. Currency-specific minor-unit defaults can differ; specify `digitsInfo` when the UI requires a fixed precision.

### Dates and timezones

`date` accepts a `Date`, a number of milliseconds since the Unix epoch, or an ISO date string. Its parameters are format, timezone, and locale:

~~~html
<p>{{ createdAt | date:'medium' }}</p>
<p>{{ createdAt | date:'yyyy-MM-dd HH:mm':'UTC' }}</p>
~~~

Without an explicit timezone or configured default, date formatting uses the end user's local timezone. Use an explicit timezone when the product requirement calls for one consistent zone, such as UTC. Prefer ISO date strings or well-defined timestamps; ambiguous date strings can be interpreted differently before the pipe receives them.

`DatePipe` is pure. Mutating an existing `Date` object does not count as a changed input reference, so the displayed value may remain unchanged. Assign a new `Date` instance when the date value changes.

## Locale data and application configuration

Angular includes locale data for `en-US` by default. To use another locale globally, register its locale data and provide the matching `LOCALE_ID`. A standalone application can configure this in application setup:

~~~ts
import { ApplicationConfig, LOCALE_ID } from '@angular/core';
import { registerLocaleData } from '@angular/common';
import localeFr from '@angular/common/locales/fr';

registerLocaleData(localeFr);

export const appConfig: ApplicationConfig = {
  providers: [{ provide: LOCALE_ID, useValue: 'fr' }],
};
~~~

Import and register the locale data your app actually needs. A locale affects dates, numbers, currencies, and plural rules; translating interface messages is a separate localization task. Individual pipes can also accept a locale argument where their API supports it, but the required locale data must still be available.

## Pure and impure pipes

A pure pipe runs when Angular detects that its input primitive value or object reference has changed. For arrays, objects, and dates, changing the reference is the dependable way to trigger a pure pipe:

~~~ts
// A new reference is visible to a pure pipe.
this.items = [...this.items, newItem];

// Mutating the existing array does not create a new reference.
this.items.push(newItem);
~~~

Most built-in pipes are pure. The built-in `AsyncPipe`, `JsonPipe`, `KeyValuePipe`, and `SlicePipe` are impure because they must respond to asynchronous emissions, object/collection changes, or other behavior beyond a simple changed input reference. Angular can reevaluate an impure pipe during change detection, so avoid putting expensive work into repeated template transformations.

A pure pipe is not a memoized computation for every possible input. It avoids calling its transform again while its input values are unchanged. Keep template transformations inexpensive and move business rules or large data processing into explicit, testable application code.

## AsyncPipe

`AsyncPipe` is the `async` template pipe. It accepts an Observable or Promise and returns the latest emitted or resolved value:

~~~html
<p>Current time: {{ currentTime$ | async | date:'mediumTime' }}</p>
<p>Greeting: {{ greetingPromise | async }}</p>
~~~

For an Observable, `AsyncPipe` subscribes and displays each new value. For a Promise, it displays the resolved value. It marks the component to be checked when a new value arrives. When the view is destroyed, Angular cleans up the subscription; if the expression begins referring to a different Observable or Promise, the pipe stops using the old reference and follows the new one.

The pipe returns `null` before its first value is available, and also for a null or undefined source. Account for this loading state in the template. The pipe handles subscription cleanup; it does not decide what loading, empty, or error UI your screen should show.

### Use a stable asynchronous source

Create the stream or Promise as component state, not by calling a factory method in the template:

~~~ts
import { interval, map } from 'rxjs';

// Stable reference created once for this component instance.
readonly currentTime$ = interval(1000).pipe(map(() => new Date()));
~~~

Avoid:

~~~html
<!-- A method could create a new Observable every time Angular checks the view. -->
<p>{{ createCurrentTimeStream() | async }}</p>
~~~

A new source reference makes `AsyncPipe` unsubscribe from the previous source and subscribe to the new one. This can restart work and create confusing behavior. Store the stream or Promise in a field, signal, or another stable source instead.

### Avoid duplicate subscriptions in a template

Each `async` pipe occurrence manages its own subscription. If multiple parts of a template need the same emitted value, bind once and reuse a template variable with the template features supported by the Angular version in the project. Be especially careful with cold Observables: separate subscriptions can start separate executions.

For a source that can emit a false-like value such as `0`, do not use a truthiness check as a substitute for a null check. A loading branch based on `@if (value$ | async; as value)` is appropriate when the emitted value is always truthy, but can hide valid values such as `0`, `false`, or an empty string. Choose a template pattern that distinguishes “not yet available” from valid data for the value's type.

### AsyncPipe and change detection

`AsyncPipe` marks its component for checking when a source emits, which works with modern change-detection strategies including `OnPush`. It also fits zoneless applications when the configured Angular notification mechanism observes the async-pipe update. It is usually the clearest template option when a template needs to render an Observable or Promise value.

Use an explicit subscription in TypeScript only when the component must perform imperative side effects, coordinate multiple streams, or manage a non-template workflow. For template display, prefer `AsyncPipe` or the signal/RxJS interop APIs that fit the state model.

## Common mistakes

- Using a pipe in a standalone component without importing that pipe.
- Assuming a pipe changes the component's stored value.
- Mutating an object or `Date` and expecting a pure pipe to notice the same reference.
- Forgetting that `AsyncPipe` can return `null` before data arrives.
- Creating a new Observable from a template method, causing reference churn and resubscriptions.
- Writing `{{ stream$ | async }}` in several places and unknowingly opening several subscriptions.
- Treating 25 as 25 percent instead of representing 25 percent as a ratio such as `0.25`.
- Omitting a currency code when the amount's currency is part of the business data.
- Assuming a date uses UTC when the default is the viewer's local timezone.
- Assuming a non-default locale works without registering its locale data.
- Using `JsonPipe` as a production data format or leaving internal object details on a public screen.
- Using `SlicePipe` or `KeyValuePipe` as a substitute for deliberately shaped, ordered view data.
- Using pipes for expensive filtering, repeated business calculations, or side effects.

## Key takeaways

- A pipe is a declarative transformation for a template value; it does not mutate the original value.
- Import each required pipe in a standalone component, or import `CommonModule` in an NgModule.
- Locale-aware pipes follow `LOCALE_ID` and locale data; set currency and timezone deliberately when the requirement needs them.
- Pure pipes react to changed values or references. Some built-ins, including `AsyncPipe`, are impure for a reason.
- `AsyncPipe` subscribes, exposes the latest value, marks the view for checking, and cleans up when the view or source changes.
- Model loading and errors explicitly, keep async sources stable, and avoid accidental duplicate subscriptions.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="034-global-styles-css-variables-and-theming.md">034 - 34. Global styles, CSS variables, and theming</a></td><td><a href="../qnas/035-built-in-pipes-and-the-async-pipe.md">Q&amp;A</a></td><td><a href="../examples/035-built-in-pipes-and-the-async-pipe.md">Example</a></td><td><a href="036-custom-pipes-attribute-directives-and-reusable-behavior.md">036 - 36. Custom pipes, attribute directives, and reusable behavior</a></td>
</tr>
  </tbody>
</table>
