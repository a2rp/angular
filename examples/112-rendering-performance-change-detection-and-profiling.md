# Rendering Performance, Change Detection, and Profiling - Examples

These snippets demonstrate derived state, immutable updates, stable list tracking, and a repeatable profiler workflow. Adapt the files and imports to the Angular version and workspace.

## 1. Derive filtered data with a computed signal

ProductListComponent class:

~~~ts
import { ChangeDetectionStrategy, Component, computed, signal } from '@angular/core';
import { ProductRowComponent } from './product-row.component';

export type Product = {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
};

@Component({
  selector: 'app-product-list',
  standalone: true,
  imports: [ProductRowComponent],
  changeDetection: ChangeDetectionStrategy.OnPush,
  templateUrl: './product-list.html',
})
export class ProductListComponent {
  readonly query = signal('');
  readonly products = signal<Product[]>([
    { id: 1, name: 'Keyboard', price: 79, inStock: true },
    { id: 2, name: 'Monitor', price: 249, inStock: false },
    { id: 3, name: 'Desk lamp', price: 35, inStock: true },
  ]);

  readonly visibleProducts = computed(() => {
    const query = this.query().trim().toLowerCase();

    return this.products().filter((product) =>
      product.name.toLowerCase().includes(query),
    );
  });

  setQuery(event: Event): void {
    const input = event.target as HTMLInputElement;
    this.query.set(input.value);
  }

  toggleStock(id: number): void {
    this.products.update((products) =>
      products.map((product) =>
        product.id === id
          ? { ...product, inStock: !product.inStock }
          : product,
      ),
    );
  }
}
~~~

product-list.html:

~~~html
<section aria-labelledby="products-heading">
  <h1 id="products-heading">Products</h1>

  <label for="product-search">Search</label>
  <input
    id="product-search"
    type="search"
    [value]="query()"
    (input)="setQuery($event)"
  />

  <ul>
    @for (product of visibleProducts(); track product.id) {
      <li>
        <app-product-row [product]="product" />
        <button type="button" (click)="toggleStock(product.id)">
          Toggle stock status
        </button>
      </li>
    } @empty {
      <li>No products match this search.</li>
    }
  </ul>
</section>
~~~

The computed list recalculates when the query or products signal changes. Tracking by product ID preserves the association between data and rendered rows. The stock update creates a new object and array, so the child receives a new input reference.

## 2. Keep an OnPush child focused

product-row.component.ts:

~~~ts
import { ChangeDetectionStrategy, Component, input } from '@angular/core';
import { CurrencyPipe } from '@angular/common';
import type { Product } from './product-list.component';

@Component({
  selector: 'app-product-row',
  standalone: true,
  imports: [CurrencyPipe],
  changeDetection: ChangeDetectionStrategy.OnPush,
  templateUrl: './product-row.html',
})
export class ProductRowComponent {
  readonly product = input.required<Product>();
}
~~~

product-row.html:

~~~html
<article>
  <h2>{{ product().name }}</h2>
  <p>Price: {{ product().price | currency }}</p>
  <p>{{ product().inStock ? 'In stock' : 'Out of stock' }}</p>
</article>
~~~

The component imports CurrencyPipe explicitly because it uses that pipe in a standalone template. The strategy is explicit for clarity. In Angular v22 and later, OnPush is the documented default, but explicit declaration can communicate the intended boundary.

## 3. Handle an external callback with reactive state

When a browser or third-party callback updates state, a signal read by the template provides an Angular notification:

~~~ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-connection-status',
  standalone: true,
  template: '<p>{{ status() }}</p>',
})
export class ConnectionStatusComponent {
  readonly status = signal('Connecting');

  connect(socket: WebSocket): void {
    socket.addEventListener('open', () => {
      this.status.set('Connected');
    });

    socket.addEventListener('close', () => {
      this.status.set('Disconnected');
    });
  }
}
~~~

Adapt lifecycle and cleanup for the actual socket owner. If a callback changes plain fields instead of reactive state, ensure it notifies Angular through an appropriate mechanism, such as markForCheck in a scenario that requires it.

## 4. Configure zoneless change detection in Angular v20.2

In Angular v20.2, a standalone application can opt into the stable zoneless provider:

~~~ts
import { ApplicationConfig, provideZonelessChangeDetection } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [provideZonelessChangeDetection()],
};
~~~

Angular v21 and later use zoneless change detection by default according to the current Angular documentation, so check the project version before adding this provider. A provider cannot make an unnotified plain-field update render; state still needs to use Angular notifications.

## 5. Profile the product list

1. Start the app with a representative dataset large enough to reproduce the slow behavior.
2. Open Angular DevTools Profiler and record typing into the search field.
3. Select the longest change-detection cycle.
4. Identify whether filtering, row rendering, or another component dominates.
5. If needed, record the same action in Chrome Performance to inspect layout and paint.
6. Make one focused change, then repeat the same recording.

For a realistic large list, include network and device conditions in the notes. Do not use a tiny demo list to claim that a production-scale bottleneck is solved.

## 6. Recognize an expensive template call

A template that invokes a sorting method can repeatedly sort the same collection:

~~~html
@for (product of getSortedProducts(); track product.id) {
  <app-product-row [product]="product" />
}
~~~

If profiling confirms this is expensive, move the derivation into state that changes only when its inputs change:

~~~ts
readonly sortedProducts = computed(() =>
  [...this.products()].sort((a, b) => a.name.localeCompare(b.name)),
);
~~~

~~~html
@for (product of sortedProducts(); track product.id) {
  <app-product-row [product]="product" />
}
~~~

The copy before sort is important because Array.sort mutates its receiver. This still sorts when products change, so use a more appropriate data structure or algorithm if profiling shows that operation remains the bottleneck.

## 7. Compare before and after

Record the same conditions for each profile:

~~~text
Route and commit:
Angular version and build mode:
Browser and device profile:
Dataset size:
Action performed:
Longest change-detection cycle:
Slowest component or expression:
Browser layout/paint contribution:
Change made:
Repeat measurement:
Visual behavior verified:
~~~

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="111-lazy-loading-and-deferrable-views-with-defer.md">111 - Lazy Loading and Deferrable Views with defer</a></td><td><a href="../lessons/112-rendering-performance-change-detection-and-profiling.md">Lesson</a></td><td><a href="../qnas/112-rendering-performance-change-detection-and-profiling.md">Q&amp;A</a></td><td><a href="113-client-rendering-server-rendering-and-prerendering.md">113 - Client Rendering, Server Rendering, and Prerendering</a></td>
</tr>
  </tbody>
</table>
