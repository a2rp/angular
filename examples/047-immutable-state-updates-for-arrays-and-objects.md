# Immutable state updates for arrays and objects — examples

This standalone Angular example manages an inventory. It demonstrates immutable array updates, replacing a nested object, updating a nested array, sorting a copied view, and keeping unchanged item references stable.

## File layout

~~~text
src/
├── main.ts
└── app/
    ├── inventory-state.ts
    └── inventory.component.ts
~~~

## 1. Create a state owner with immutable transitions

~~~ts
// src/app/inventory-state.ts
import { computed, Injectable, signal } from '@angular/core';

export interface InventoryItem {
  readonly id: number;
  readonly name: string;
  readonly quantity: number;
  readonly supplier: {
    readonly name: string;
  };
  readonly tags: readonly string[];
}

const initialItems: readonly InventoryItem[] = [
  {
    id: 1,
    name: 'Notebook',
    quantity: 12,
    supplier: { name: 'Paper House' },
    tags: ['stationery', 'popular'],
  },
  {
    id: 2,
    name: 'Desk lamp',
    quantity: 4,
    supplier: { name: 'Bright Goods' },
    tags: ['lighting'],
  },
  {
    id: 3,
    name: 'Water bottle',
    quantity: 9,
    supplier: { name: 'Trail Supply' },
    tags: ['outdoors'],
  },
];

@Injectable({ providedIn: 'root' })
export class InventoryState {
  readonly #items = signal<readonly InventoryItem[]>(initialItems);
  readonly items = this.#items.asReadonly();

  // Sorting is only for display, so sort a copied array.
  readonly alphabetizedItems = computed(() =>
    [...this.items()].sort((a, b) => a.name.localeCompare(b.name)),
  );

  addItem(item: InventoryItem): void {
    this.#items.update((items) => {
      // IDs are unique in this example; ignore a duplicate.
      if (items.some((current) => current.id === item.id)) {
        return items;
      }

      return [...items, item];
    });
  }

  updateQuantity(id: number, quantity: number): void {
    if (!Number.isFinite(quantity) || quantity < 0) {
      return;
    }

    this.#items.update((items) => {
      const current = items.find((item) => item.id === id);

      if (!current || current.quantity === quantity) {
        return items;
      }

      return items.map((item) =>
        item.id === id ? { ...item, quantity } : item,
      );
    });
  }

  updateSupplierName(id: number, name: string): void {
    this.#items.update((items) => {
      const current = items.find((item) => item.id === id);

      if (!current || current.supplier.name === name) {
        return items;
      }

      return items.map((item) =>
        item.id === id
          ? {
              ...item,
              supplier: {
                ...item.supplier,
                name,
              },
            }
          : item,
      );
    });
  }

  addTag(id: number, tag: string): void {
    const normalizedTag = tag.trim();

    if (normalizedTag.length === 0) {
      return;
    }

    this.#items.update((items) => {
      const current = items.find((item) => item.id === id);

      if (!current || current.tags.includes(normalizedTag)) {
        return items;
      }

      return items.map((item) =>
        item.id === id
          ? { ...item, tags: [...item.tags, normalizedTag] }
          : item,
      );
    });
  }

  removeTag(id: number, tag: string): void {
    this.#items.update((items) => {
      const current = items.find((item) => item.id === id);

      if (!current || !current.tags.includes(tag)) {
        return items;
      }

      return items.map((item) =>
        item.id === id
          ? {
              ...item,
              tags: item.tags.filter((currentTag) => currentTag !== tag),
            }
          : item,
      );
    });
  }

  removeItem(id: number): void {
    this.#items.update((items) => {
      if (!items.some((item) => item.id === id)) {
        return items;
      }

      return items.filter((item) => item.id !== id);
    });
  }
}
~~~

The writable signal is private. The public signal is read-only, and all supported changes go through methods that validate the transition. Each no-op returns the current array so it does not create a needless replacement.

## 2. Render the data and request changes

~~~ts
// src/app/inventory.component.ts
import { Component, inject } from '@angular/core';
import { InventoryState } from './inventory-state';

@Component({
  selector: 'app-inventory',
  standalone: true,
  template: `
    <main>
      <h1>Inventory</h1>
      <p>{{ state.items().length }} items</p>

      @for (item of state.alphabetizedItems(); track item.id) {
        <article>
          <h2>{{ item.name }}</h2>
          <p>Quantity: {{ item.quantity }}</p>
          <p>Supplier: {{ item.supplier.name }}</p>

          <h3>Tags</h3>
          <ul>
            @for (tag of item.tags; track tag) {
              <li>
                {{ tag }}
                <button
                  type="button"
                  (click)="state.removeTag(item.id, tag)"
                >
                  Remove tag
                </button>
              </li>
            } @empty {
              <li>No tags</li>
            }
          </ul>

          <button
            type="button"
            (click)="state.updateQuantity(item.id, item.quantity + 1)"
          >
            Add one
          </button>

          <button
            type="button"
            (click)="state.updateSupplierName(
              item.id,
              item.supplier.name + ' (updated)'
            )"
          >
            Update supplier name
          </button>

          <button
            type="button"
            (click)="state.addTag(item.id, 'featured')"
          >
            Add featured tag
          </button>

          <button type="button" (click)="state.removeItem(item.id)">
            Remove item
          </button>
        </article>
      } @empty {
        <p>The inventory is empty.</p>
      }
    </main>
  `,
})
export class InventoryComponent {
  readonly state = inject(InventoryState);
}
~~~

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { InventoryComponent } from './app/inventory.component';

bootstrapApplication(InventoryComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

This assumes a standard standalone Angular application with the usual HTML document and build setup. The component reads the state and calls methods; it does not edit the collection or its objects directly.

## 3. Inspect which references change

Suppose the quantity of the first item changes. The state owner creates:

- A new inventory array.
- A new object for the notebook whose quantity changed.
- The same object references for the desk lamp and water bottle.

When a supplier name changes, the owner also creates a new supplier object for that item's changed path. When a tag is added, it creates a new tags array for that item. Other item objects remain shared.

The sorted view creates a new array for sorting, but it does not mutate the source array. The item objects inside that view are the same objects held by the source state unless an item was updated.

## 4. Compare with mutation

Do not update stored state like this:

~~~ts
// Both operations mutate values already held in state.
const items = this.state.items() as InventoryItem[];
items[0].quantity += 1;
items.sort((a, b) => a.name.localeCompare(b.name));
~~~

A read-only signal prevents calling signal.set() through that signal, but it does not deeply freeze the returned array. The cast only silences TypeScript; it does not make the update immutable or safe.

Instead, update through the state owner:

~~~ts
this.state.updateQuantity(itemId, currentQuantity + 1);
~~~

The owner creates replacement references and preserves its validation rules.

## 5. Pure array update in isolation

The same transition can be expressed as a pure function. It receives an array and returns the next array without changing the input:

~~~ts
function updateQuantity(
  items: readonly InventoryItem[],
  id: number,
  quantity: number,
): readonly InventoryItem[] {
  const current = items.find((item) => item.id === id);

  if (!current || current.quantity === quantity) {
    return items;
  }

  return items.map((item) =>
    item.id === id ? { ...item, quantity } : item,
  );
}
~~~

A pure helper is useful when the transformation is complex enough to reuse or reason about separately. Keep simple updates readable; extracting every one-line operation into a helper can make a small state transition harder to follow.

## What this example demonstrates

- Additions use a new array; removals use filter; replacements use map.
- A nested update copies every object along the changed path.
- An updated item's nested tags array is replaced instead of mutated.
- Sorting uses a copied array and does not alter canonical state.
- Unchanged items keep their references.
- No-op and invalid transitions do not create a new array.
- Read-only signals and readonly types help protect the boundary but do not deeply freeze JavaScript values.
