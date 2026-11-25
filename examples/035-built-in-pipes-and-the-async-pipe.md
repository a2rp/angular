# Example 35: Built-in pipes and the async pipe

This example is a small standalone Angular page that uses the built-in pipes in `@angular/common`. It formats product data for display, slices a list, renders object entries, selects plural/status messages, and displays values from both an Observable and a Promise.

## 1. Create a practice app

```sh
ng new pipes-demo --style=css
cd pipes-demo
ng serve --open
```

Use the generated standalone application. The exact filenames can vary by Angular CLI version; if the CLI generated `app.ts` and `app.html` instead of `app.component.ts` and `app.component.html`, place the same component code in the generated files and adjust `templateUrl`, `styleUrls`, and import paths to match. No third-party packages are needed; Angular CLI apps already include RxJS.

## 2. Component class

### `src/app/app.component.ts`

```ts
import {
  AsyncPipe,
  CurrencyPipe,
  DatePipe,
  DecimalPipe,
  I18nPluralPipe,
  I18nSelectPipe,
  JsonPipe,
  KeyValuePipe,
  LowerCasePipe,
  PercentPipe,
  SlicePipe,
  TitleCasePipe,
  UpperCasePipe,
} from '@angular/common';
import { Component } from '@angular/core';
import { interval, map } from 'rxjs';

interface Product {
  id: string;
  name: string;
  price: number;
  rating: number;
  releasedAt: Date;
}

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    AsyncPipe,
    CurrencyPipe,
    DatePipe,
    DecimalPipe,
    I18nPluralPipe,
    I18nSelectPipe,
    JsonPipe,
    KeyValuePipe,
    LowerCasePipe,
    PercentPipe,
    SlicePipe,
    TitleCasePipe,
    UpperCasePipe,
  ],
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export class AppComponent {
  readonly heading = 'built-in pipes in a standalone component';

  readonly featuredProduct: Product = {
    id: 'keyboard-01',
    name: 'wireless keyboard',
    price: 79.99,
    rating: 0.94,
    releasedAt: new Date('2026-02-12T12:00:00Z'),
  };

  readonly products: Product[] = [
    this.featuredProduct,
    {
      id: 'mouse-01',
      name: 'ergonomic mouse',
      price: 49.5,
      rating: 0.89,
      releasedAt: new Date('2026-01-08T09:30:00Z'),
    },
    {
      id: 'stand-01',
      name: 'aluminum laptop stand',
      price: 34,
      rating: 0.91,
      releasedAt: new Date('2025-11-20T15:00:00Z'),
    },
    {
      id: 'hub-01',
      name: 'usb-c hub',
      price: 59.95,
      rating: 0.86,
      releasedAt: new Date('2025-09-14T08:00:00Z'),
    },
  ];

  readonly completionRatio = 0.72;
  readonly measurements = {
    width: 12.5,
    height: 8,
    weight: 0.42,
  };

  readonly unreadCount = 3;
  readonly unreadMessages: Record<string, string> = {
    '=0': 'No unread messages',
    '=1': 'One unread message',
    other: '# unread messages',
  };

  readonly deliveryStatus = 'shipped';
  readonly deliveryMessages: Record<string, string> = {
    pending: 'The order is being prepared.',
    shipped: 'The order is on its way.',
    delivered: 'The order has arrived.',
    other: 'The delivery status is unavailable.',
  };

  // Keep async sources stable instead of creating them from template methods.
  readonly currentTime$ = interval(1000).pipe(map(() => new Date()));
  readonly welcomePromise = Promise.resolve(
    'This message was resolved from a Promise.',
  );
}
```

`AsyncPipe` is imported because the template uses `async`. The other pipe classes are also imported directly so the standalone component has exactly the template dependencies it needs.

## 3. Template

### `src/app/app.component.html`

```html
<main class="page">
  <header class="page-header">
    <p class="eyebrow">Angular template transformations</p>
    <h1>{{ heading | titlecase }}</h1>
    <p>
      Each example changes the displayed value. The typed component data stays
      available in its original form.
    </p>
  </header>

  <section class="panel" aria-labelledby="text-pipes-heading">
    <h2 id="text-pipes-heading">Text pipes</h2>
    <dl class="examples-list">
      <dt>Lowercase</dt>
      <dd>{{ featuredProduct.name | lowercase }}</dd>

      <dt>Uppercase</dt>
      <dd>{{ featuredProduct.name | uppercase }}</dd>

      <dt>Title case</dt>
      <dd>{{ featuredProduct.name | titlecase }}</dd>

      <dt>Chained pipes</dt>
      <dd>{{ 'angular templates' | uppercase | lowercase }}</dd>
    </dl>
  </section>

  <section class="panel" aria-labelledby="formatting-heading">
    <h2 id="formatting-heading">Numbers, currency, and dates</h2>
    <dl class="examples-list">
      <dt>Currency</dt>
      <dd>{{ featuredProduct.price | currency:'USD':'symbol':'1.2-2' }}</dd>

      <dt>Rating as a ratio</dt>
      <dd>{{ featuredProduct.rating | percent:'1.0-1' }}</dd>

      <dt>Decimal precision</dt>
      <dd>{{ featuredProduct.price | number:'1.2-2' }}</dd>

      <dt>Localized date</dt>
      <dd>{{ featuredProduct.releasedAt | date:'mediumDate' }}</dd>

      <dt>Explicit UTC date and time</dt>
      <dd>{{ featuredProduct.releasedAt | date:'yyyy-MM-dd HH:mm':'UTC' }}</dd>

      <dt>Another percentage</dt>
      <dd>{{ completionRatio | percent:'1.0-0' }}</dd>
    </dl>
    <p class="hint">
      Output separators and date formats depend on the configured locale. The
      explicit timezone example uses UTC; the other date uses the viewer's
      local timezone by default.
    </p>
  </section>

  <section class="panel" aria-labelledby="collection-heading">
    <h2 id="collection-heading">Slice a collection</h2>
    <p>The source has {{ products.length }} products; this view shows two.</p>
    <ul class="product-list">
      @for (product of products | slice:0:2; track product.id) {
        <li>
          <strong>{{ product.name | titlecase }}</strong>
          <span>{{ product.price | currency:'USD' }}</span>
        </li>
      }
    </ul>
  </section>

  <section class="panel" aria-labelledby="keyvalue-heading">
    <h2 id="keyvalue-heading">Render object entries</h2>
    <p>`keyvalue` sorts keys by default.</p>
    <dl class="examples-list">
      @for (entry of measurements | keyvalue; track entry.key) {
        <dt>{{ entry.key | titlecase }}</dt>
        <dd>{{ entry.value | number:'1.0-2' }}</dd>
      }
    </dl>
  </section>

  <section class="panel" aria-labelledby="messages-heading">
    <h2 id="messages-heading">Select a message</h2>
    <p>{{ unreadCount | i18nPlural:unreadMessages }}</p>
    <p>{{ deliveryStatus | i18nSelect:deliveryMessages }}</p>
  </section>

  <section class="panel" aria-labelledby="async-heading">
    <h2 id="async-heading">AsyncPipe</h2>
    <p>Promise: {{ welcomePromise | async }}</p>

    @if (currentTime$ | async; as currentTime) {
      <p>Observable time: {{ currentTime | date:'mediumTime' }}</p>
    } @else {
      <p role="status">Waiting for the first time value…</p>
    }
  </section>

  <section class="panel debug-panel" aria-labelledby="json-heading">
    <h2 id="json-heading">JsonPipe for local debugging</h2>
    <pre>{{ featuredProduct | json }}</pre>
  </section>
</main>
```

The async `@if` aliases a `Date` object, which is truthy. Do not copy this truthiness pattern unchanged for a stream that can emit valid `0`, `false`, or an empty string; those values can select the fallback branch. `AsyncPipe` produces `null` until the Observable emits, so this template also has an explicit waiting message.

## 4. Component styles

### `src/app/app.component.css`

```css
:host {
  display: block;
  min-height: 100vh;
  color: #172033;
  background: #f1f5f9;
  font-family: system-ui, sans-serif;
}

.page {
  display: grid;
  gap: 1rem;
  width: min(100% - 2rem, 52rem);
  margin-inline: auto;
  padding-block: 2rem 4rem;
}

.page-header,
.panel {
  padding: clamp(1rem, 3vw, 1.5rem);
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: #ffffff;
}

.page-header {
  display: grid;
  gap: 0.75rem;
}

.page-header h1 {
  font-size: clamp(1.75rem, 5vw, 2.5rem);
  line-height: 1.15;
}

.page-header > p:not(.eyebrow),
.hint {
  color: #526176;
  line-height: 1.6;
}

.eyebrow {
  color: #1d4ed8;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.panel {
  display: grid;
  gap: 0.9rem;
}

.panel h2 {
  font-size: 1.2rem;
}

.examples-list {
  display: grid;
  grid-template-columns: minmax(9rem, 0.8fr) minmax(0, 1.2fr);
  gap: 0.55rem 1rem;
}

.examples-list dt {
  color: #526176;
}

.examples-list dd {
  min-width: 0;
  overflow-wrap: anywhere;
  font-weight: 600;
}

.product-list {
  display: grid;
  gap: 0.5rem;
  list-style: none;
}

.product-list li {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  gap: 0.5rem 1rem;
  padding: 0.75rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.5rem;
}

.debug-panel pre {
  overflow-x: auto;
  padding: 1rem;
  border-radius: 0.5rem;
  background: #f1f5f9;
  color: #172033;
  white-space: pre-wrap;
  overflow-wrap: anywhere;
}

@media (max-width: 32rem) {
  .examples-list {
    grid-template-columns: 1fr;
    gap: 0.25rem;
  }

  .examples-list dd + dt {
    margin-top: 0.75rem;
  }
}
```


## 5. Understand the output

- The text pipes format strings for display; the `featuredProduct.name` property remains lowercase in TypeScript.
- Currency, decimal, percentage, and date output use Angular's locale-aware formatting rules. Date output may vary with the machine's local timezone unless a timezone is specified.
- `slice:0:2` displays the first two products without removing entries from `products`.
- `keyvalue` turns the measurements object into entries with `key` and `value`; the default order is by key.
- `i18nPlural` uses the numeric count and mapping. `i18nSelect` matches the status key and falls back to `other` if needed.
- `AsyncPipe` subscribes to `currentTime$`, renders new values, and cleans up when its view is destroyed. The Promise is also unwrapped by the same pipe.
- `JsonPipe` makes a development value easier to inspect. Do not leave internal data exposed in a public production view.

## 6. Try these changes

1. Change `completionRatio` to `0.25`; observe that `percent` renders 25 percent.
2. Change `unreadCount` to `0` and `1` to see the exact plural-map entries.
3. Add a product and see how `slice:0:2` still limits the rendered list.
4. Change a key in `measurements` and observe the default key ordering.
5. Change `deliveryStatus` to a key not present in `deliveryMessages`; the `other` entry is used.
6. Remove `AsyncPipe` from the component's imports and observe the template error, then restore it.
7. Remove the `UTC` argument from the explicit date and compare it with the local-time output.

The template stores `currentTime$` and `welcomePromise` as stable component properties. The Observable starts when `AsyncPipe` subscribes and stops when the view is destroyed; no manual subscription or cleanup code is needed for this display.

## References

- [Angular pipes guide](https://angular.dev/guide/templates/pipes)
- [AsyncPipe API](https://angular.dev/api/common/AsyncPipe)
- [Format data based on locale](https://angular.dev/guide/i18n/format-data-locale)
- [DatePipe API](https://angular.dev/api/common/DatePipe)
- [CurrencyPipe API](https://angular.dev/api/common/CurrencyPipe)
- [KeyValuePipe API](https://angular.dev/api/common/KeyValuePipe)
- [SlicePipe API](https://angular.dev/api/common/SlicePipe)
- [Angular template control flow](https://angular.dev/guide/templates/control-flow)
