# 39. Example: `linkedSignal` and dependent writable state

This standalone Angular page lets a user choose a shipping option. The selected option is writable, but it also depends on the current options list. When the list is replaced, the example keeps the selected option by ID if it still exists, selects the first option if it was removed, or uses `null` if the list is empty.

## What you will build

- A writable signal containing the current shipping options.
- A `linkedSignal()` that can be changed by the user and recalculates when the options change.
- A `previous`-aware computation that preserves a still-valid selection across refreshed objects.
- Controls to refresh the options, remove the current choice, clear the list, and restore the defaults.

## Requirements and setup

`linkedSignal()` is stable in Angular 20 and later. Create and run a standalone project if needed:

~~~sh
ng new linked-shipping-demo --standalone --style=css
cd linked-shipping-demo
ng serve
~~~

Use the generated root component. The files below use conventional `app.component.*` names; adjust them if your CLI generated shorter names such as `app.*`.

## File layout

~~~text
src/app/
  app.component.css
  app.component.html
  app.component.ts
~~~

## 1. Create the source and dependent writable signal

Replace or adapt `src/app/app.component.ts`:

~~~ts
import { CurrencyPipe } from '@angular/common';
import { Component, linkedSignal, signal } from '@angular/core';

interface ShippingOption {
  id: number;
  name: string;
  price: number;
}

const initialOptions: ShippingOption[] = [
  { id: 1, name: 'Ground (3–5 days)', price: 5 },
  { id: 2, name: 'Air (1–2 days)', price: 12 },
  { id: 3, name: 'Sea (7–10 days)', price: 3 },
];

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CurrencyPipe],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  readonly shippingOptions = signal<ShippingOption[]>(
    initialOptions.map((option) => ({ ...option })),
  );

  readonly selectedOption = linkedSignal<
    ShippingOption[],
    ShippingOption | null
  >({
    source: this.shippingOptions,
    computation: (newOptions, previous) => {
      const previousId = previous?.value?.id;

      return (
        newOptions.find((option) => option.id === previousId) ??
        newOptions[0] ??
        null
      );
    },
  });

  chooseOption(rawId: string): void {
    const option = this.shippingOptions().find(
      (item) => item.id === Number(rawId),
    );

    if (option) {
      this.selectedOption.set(option);
    }
  }

  refreshOptions(): void {
    // New objects arrive from the refreshed source, with the same stable IDs.
    this.shippingOptions.set([
      { id: 1, name: 'Ground (2–4 days)', price: 6 },
      { id: 2, name: 'Air (next day)', price: 14 },
      { id: 3, name: 'Sea (6–9 days)', price: 4 },
    ]);
  }

  removeSelectedOption(): void {
    const selectedId = this.selectedOption()?.id;

    if (selectedId === undefined) {
      return;
    }

    this.shippingOptions.update((current) =>
      current.filter((option) => option.id !== selectedId),
    );
  }

  clearOptions(): void {
    this.shippingOptions.set([]);
  }

  restoreOptions(): void {
    this.shippingOptions.set(
      initialOptions.map((option) => ({ ...option })),
    );
  }
}
~~~

The source signal owns the current list. The linked signal is writable: `chooseOption()` sets the user's choice. When the list changes, the computation uses the previous selected ID to find the corresponding option in the new array. This returns the refreshed object, not a stale object from the old array.

The explicit generic arguments are `<Source, Destination>`. Here, the source is an array and the destination is one option or `null`. That `null` result handles an empty list without returning `undefined` unexpectedly.

## 2. Add the template

Create or replace `src/app/app.component.html`:

~~~html
<main class="page">
  <header class="page-header">
    <p class="eyebrow">Angular example 39</p>
    <h1>Dependent writable state</h1>
    <p>
      The selected option can be changed directly, then reconciled when its
      source list changes.
    </p>
  </header>

  <section class="panel" aria-labelledby="shipping-title">
    <h2 id="shipping-title">Choose a shipping option</h2>

    <label for="shipping-option">Available options</label>
    <select
      id="shipping-option"
      [value]="selectedOption()?.id ?? ''"
      (change)="chooseOption($any($event.target).value)"
      [disabled]="shippingOptions().length === 0"
    >
      <option value="" disabled>Select a shipping option</option>
      @for (option of shippingOptions(); track option.id) {
        <option [value]="option.id">
          {{ option.name }} — {{ option.price | currency }}
        </option>
      }
    </select>

    @if (selectedOption(); as selected) {
      <div class="selection" aria-live="polite">
        <h3>Current selection</h3>
        <p><strong>{{ selected.name }}</strong></p>
        <p>Option ID: {{ selected.id }}</p>
        <p>Price: {{ selected.price | currency }}</p>
      </div>
    } @else {
      <p class="empty-state" aria-live="polite">
        No shipping options are available.
      </p>
    }
  </section>

  <section class="panel" aria-labelledby="source-title">
    <h2 id="source-title">Change the source list</h2>
    <p>
      Refreshing creates new option objects. The linked selection finds the
      matching ID again. Removing the selected option makes it fall back to the
      first remaining option.
    </p>

    <div class="button-row">
      <button type="button" (click)="refreshOptions()">
        Refresh option details
      </button>
      <button
        type="button"
        class="secondary"
        (click)="removeSelectedOption()"
        [disabled]="selectedOption() === null"
      >
        Remove selected option
      </button>
      <button type="button" class="secondary" (click)="clearOptions()">
        Clear all options
      </button>
      <button type="button" class="secondary" (click)="restoreOptions()">
        Restore defaults
      </button>
    </div>

    <p class="note">
      The list contains {{ shippingOptions().length }} options. The selection
      is stored separately because it can be changed by the user.
    </p>
  </section>
</main>
~~~

The standalone component imports `CurrencyPipe` from `@angular/common`. In an NgModule-based app, import `CommonModule` in the declaring module instead.

## 3. Add the styles

Create or replace `src/app/app.component.css`:

~~~css
:host {
  display: block;
  min-height: 100vh;
  color: #172033;
  background: #f3f6fb;
  font-family: system-ui, sans-serif;
}

.page {
  width: min(100% - 2rem, 54rem);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header {
  margin-bottom: 2rem;
}

.eyebrow {
  color: #425b84;
  font-size: 0.875rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

h1,
h2,
h3 {
  line-height: 1.2;
}

h1 {
  margin-block: 0.5rem 1rem;
  font-size: clamp(2rem, 6vw, 3rem);
}

h2 {
  margin-top: 0;
}

.panel {
  margin-block: 1.25rem;
  padding: clamp(1rem, 4vw, 1.75rem);
  border: 1px solid #cbd5e1;
  border-radius: 0.9rem;
  background: #fff;
  box-shadow: 0 0.5rem 1.5rem rgb(15 23 42 / 6%);
}

label {
  display: block;
  margin-bottom: 0.45rem;
  font-weight: 650;
}

select {
  width: 100%;
  min-height: 2.8rem;
  padding: 0.55rem 0.75rem;
  border: 1px solid #8b9ab1;
  border-radius: 0.5rem;
  color: #172033;
  background: #fff;
  font: inherit;
}

.selection,
.empty-state {
  margin-top: 1.25rem;
  padding: 1rem;
  border: 1px solid #d7dfeb;
  border-radius: 0.65rem;
  background: #f8fafc;
}

.selection h3 {
  margin-top: 0;
}

.selection p:last-child {
  margin-bottom: 0;
}

.button-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.65rem;
  margin-block: 1rem;
}

button {
  min-height: 2.65rem;
  padding: 0.55rem 0.9rem;
  border: 1px solid #1d4ed8;
  border-radius: 0.5rem;
  color: #fff;
  background: #1d4ed8;
  font: inherit;
  font-weight: 650;
  cursor: pointer;
}

button.secondary {
  border-color: #8b9ab1;
  color: #172033;
  background: #fff;
}

button:disabled,
select:disabled {
  cursor: not-allowed;
  opacity: 0.65;
}

button:focus-visible,
select:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 3px;
}

.note {
  color: #43516a;
  font-size: 0.95rem;
}
~~~

## Try the linked behavior

1. Select **Sea (7–10 days)**. The linked signal now holds the Sea option chosen by the user.
2. Click **Refresh option details**. Angular replaces the list with new objects. The computation finds ID `3` and selects the refreshed Sea object with its updated name and price.
3. Click **Remove selected option**. The source list no longer contains ID `3`, so the computation selects the first remaining option.
4. Click **Clear all options**. The computation returns `null`, and the page displays the empty state.
5. Click **Restore defaults**. The first default becomes selected because the previous value is `null` and the refreshed source is non-empty.
6. Choose another option and change nothing else. The manually selected value remains until a linked dependency changes.

## How the computation works

- The source is `shippingOptions`, a writable signal containing the current array.
- `selectedOption` is a writable linked signal, so the user can choose an option with `.set()`.
- When the array changes, the computation receives the new array and the previous linked value.
- The computation compares a stable ID, not object identity, because refreshed API data commonly creates new objects.
- If the old ID remains, the computation returns the matching object from the new array.
- If that ID was removed, it uses the first remaining option. With no options, it returns `null`.

## Common problems

- **The selected option resets to the first item on every refresh:** Use the source/computation form and preserve the previous ID when it remains in the new list.
- **The selected object has stale details after a refresh:** Find and return the matching object from the new source array instead of returning `previous.value` directly.
- **The selection is `undefined` for an empty list:** Include `null` in the destination type and explicitly return it when there is no fallback.
- **The `previous` value is missing during initialization:** That is expected. Use optional chaining and define a valid initial fallback.
- **`linkedSignal` cannot be imported:** Confirm that the project uses Angular 20 or later.
- **The displayed value does not change after selection:** Make sure the UI reads `selectedOption()` and that the change handler calls `.set()` with an option from the current source list.

## Further reading

- [Lesson 39: `linkedSignal` and dependent writable state](../lessons/039-linkedsignal-and-dependent-writable-state.md)
- [Q&A 39](../qnas/039-linkedsignal-and-dependent-writable-state.md)
- [Angular linkedSignal guide](https://angular.dev/guide/signals/linked-signal)
- [linkedSignal API](https://angular.dev/api/core/linkedSignal)