# Internationalization, Localization, and Formatting — Examples

This example marks user-facing messages, extracts translations, formats localized data, and lays out a summary card with logical CSS properties. It uses Angular's build-time localization workflow. A runtime language switch needs a separate strategy.

## 1. Add localization support and mark messages

If the workspace does not already include Angular's localize package, add it:

~~~sh
ng add @angular/localize
~~~

Mark complete messages and translatable attributes in a template:

~~~html
<main>
  <h1 i18n="dashboard title|Heading on the account dashboard@@dashboardTitle">
    Account overview
  </h1>

  <p i18n="dashboard greeting|Greeting shown above the summary@@dashboardGreeting">
    Welcome back
  </p>

  <img
    src="/assets/account.svg"
    alt="Account illustration"
    i18n-alt="account illustration|Alternative text for the account illustration@@accountIllustrationAlt"
  />

  <p i18n>
    {messageCount, plural,
      =0 {You have no new messages}
      =1 {You have one new message}
      other {You have {{ messageCount }} new messages}
    }
  </p>
</main>
~~~

Give translators context when a short phrase is ambiguous. Keep the whole sentence in one translatable message instead of joining language fragments in code.

## 2. Mark a code message

~~~ts
import { $localize } from '@angular/localize/init';

export class SaveButton {
  readonly savedMessage =
    $localize`:save confirmation|Shown after profile changes are saved@@savedConfirmation:Your preferences were saved.`;
}
~~~

The custom message ID is optional. Use IDs deliberately and keep them stable if the translation workflow relies on them. Re-extract messages when source text changes and review the resulting translation changes.

## 3. Extract the source translation file

Run extraction after marking template text, translatable attributes, and code messages:

~~~sh
ng extract-i18n --output-path src/locale
~~~

Review the generated source file and provide the context translators need. Create or update one translated file for each supported locale. The extraction command creates translation units; it does not translate them.

## 4. Configure localized builds

A simplified workspace configuration can declare a source locale and a French translation file:

~~~json
{
  "projects": {
    "storefront": {
      "i18n": {
        "sourceLocale": "en-US",
        "locales": {
          "fr": "src/locale/messages.fr.xlf"
        }
      }
    }
  }
}
~~~

Place the i18n configuration in the relevant project configuration in angular.json. Preserve the rest of the generated workspace settings. Then build the configured localized applications:

~~~sh
ng build storefront --localize
~~~

Check the Angular CLI version's i18n documentation for supported translation formats, locale IDs, and deployment paths.

## 5. Format data using the selected locale

~~~ts
import { Component } from '@angular/core';
import { CurrencyPipe, DatePipe, DecimalPipe } from '@angular/common';

@Component({
  selector: 'app-invoice-summary',
  imports: [CurrencyPipe, DatePipe, DecimalPipe],
  templateUrl: './invoice-summary.html'
})
export class InvoiceSummary {
  readonly invoiceDate = new Date('2026-10-18T15:30:00Z');
  readonly total = 12345.67;
  readonly itemCount = 12345;
  readonly currencyCode = 'EUR';
}
~~~

~~~html
<section>
  <h2 i18n="invoice summary title|Heading for the invoice summary">
    Invoice summary
  </h2>

  <p>
    <span i18n>Invoice date:</span>
    <time>{{ invoiceDate | date: 'longDate' }}</time>
  </p>

  <p>
    <span i18n>Total:</span>
    {{ total | currency: currencyCode }}
  </p>

  <p>
    <span i18n>Items:</span>
    {{ itemCount | number: '1.0-0' }}
  </p>
</section>
~~~

The locale used by the Angular application controls default date, decimal, and currency presentation. The currency code identifies the displayed currency; it does not convert the amount. If the application explicitly overrides its locale instead of serving localized builds, register the needed locale data and configure LOCALE_ID according to Angular's locale documentation.

## 6. Use logical styles for direction-aware layout

~~~css
.invoice-summary {
  padding-inline: 1rem;
  margin-inline-start: auto;
  border-inline-start: 0.25rem solid var(--accent-color);
}

.invoice-summary__actions {
  display: flex;
  gap: 0.75rem;
  justify-content: flex-end;
}
~~~

Logical properties adapt to left-to-right and right-to-left writing directions. Verify each icon and navigation pattern individually; some should mirror and others should not.

## 7. Review the localized interface

Use a long translation or pseudolocalized build and inspect:

- Button and navigation width.
- Text wrapping in cards, tables, dialogs, and validation messages.
- Date, number, and currency output.
- Right-to-left direction and mixed-direction values.
- Image alternative text and accessible names.
- Whether a user's locale selection matches displayed messages and formats.

Do not use the source-language page as the only layout check. A localized build can compile successfully while still clipping important text.

## Further reading

- [Angular internationalization guide](https://angular.dev/guide/i18n)
- [Prepare components for translation](https://angular.dev/guide/i18n/prepare)
- [Extract and manage translation files](https://angular.dev/guide/i18n/translation-files)
- [Format data based on locale](https://angular.dev/guide/i18n/format-data-locale)
