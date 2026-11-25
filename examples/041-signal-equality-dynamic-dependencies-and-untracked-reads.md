# 41. Example: signal equality, dynamic dependencies, and `untracked()`

This standalone Angular page makes signal behavior observable with three small demos. One uses a custom comparator for a product signal, one uses a computed signal with a conditional dependency, and one uses `untracked()` so a diagnostic counter does not trigger an audit effect.

## What you will build

- A product signal that ignores equivalent object copies but accepts meaningful field changes.
- A rounded computed value whose equality suppresses downstream work when the rounded result stays the same.
- A profile computed signal whose `age` dependency is active only while details are visible.
- An audit effect that tracks the selected user but reads a counter without tracking it.
- Browser console messages that show which effects rerun.

## Requirements and setup

Create a practice project if needed:

~~~sh
ng new signal-tracking-demo --standalone --style=css
cd signal-tracking-demo
ng serve
~~~

Use the generated root component. The files below use conventional `app.component.*` names; adjust them if your CLI generated shorter names such as `app.*`.

## File layout

~~~text
src/app/
  app.component.ts
  app.component.html
  app.component.css
~~~

## 1. Create signals, computed values, and effects

Replace or adapt `src/app/app.component.ts`:

~~~ts
import {
  Component,
  computed,
  effect,
  signal,
  untracked,
} from '@angular/core';

interface Product {
  id: number;
  name: string;
  price: number;
}

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  readonly product = signal<Product>(
    { id: 1, name: 'Notebook', price: 12 },
    {
      equal: (previous, next) =>
        previous.id === next.id &&
        previous.name === next.name &&
        previous.price === next.price,
    },
  );

  readonly measurement = signal(2.1);
  readonly roundedMeasurement = computed(
    () => ({ value: Math.round(this.measurement()) }),
    { equal: (previous, next) => previous.value === next.value },
  );

  readonly showDetails = signal(false);
  readonly profileName = signal('Ada');
  readonly profileAge = signal(36);
  readonly profileSummary = computed(() => {
    if (this.showDetails()) {
      return `${this.profileName()}, age ${this.profileAge()}`;
    }

    return this.profileName();
  });

  readonly currentUser = signal('Ada');
  readonly diagnosticCounter = signal(0);

  constructor() {
    effect(() => {
      console.info('Product changed:', this.product());
    });

    effect(() => {
      console.info('Rounded display changed:', this.roundedMeasurement().value);
    });

    effect(() => {
      const user = this.currentUser();
      const diagnosticCount = untracked(this.diagnosticCounter);
      console.info(`Audit user: ${user}; diagnostic count: ${diagnosticCount}`);
    });
  }

  setEquivalentProduct(): void {
    // New identity, but every field used by the comparator is unchanged.
    this.product.set({ ...this.product() });
  }

  raiseProductPrice(): void {
    this.product.update((current) => ({ ...current, price: current.price + 1 }));
  }

  setMeasurement(value: number): void {
    this.measurement.set(value);
  }

  toggleDetails(): void {
    this.showDetails.update((visible) => !visible);
  }

  changeProfileAge(): void {
    this.profileAge.update((age) => age + 1);
  }

  switchUser(): void {
    this.currentUser.update((user) => (user === 'Ada' ? 'Grace' : 'Ada'));
  }

  incrementDiagnosticCounter(): void {
    this.diagnosticCounter.update((count) => count + 1);
  }
}
~~~

The product comparator checks all fields that this example displays. A copied product with the same field values is treated as equivalent; raising its price creates a meaningful change. The rounded computed recalculates from `measurement`, but its comparator says whether downstream consumers need to be notified about the new rounded result.

The profile computed always reads `showDetails()` and `profileName()`. It reads `profileAge()` only when details are visible. The audit effect tracks `currentUser()` and deliberately reads `diagnosticCounter` without tracking it.

## 2. Add the template

Create or replace `src/app/app.component.html`:

~~~html
<main class="page">
  <header class="page-header">
    <p class="eyebrow">Angular example 41</p>
    <h1>When do signals count as changed?</h1>
    <p>
      Try the controls and watch the browser console to see how equality and
      dependencies affect reactive updates.
    </p>
  </header>

  <section class="card" aria-labelledby="product-heading">
    <h2 id="product-heading">Custom equality for an object signal</h2>
    <p>
      {{ product().name }} costs {{ product().price }} credits.
    </p>
    <div class="button-row">
      <button type="button" (click)="setEquivalentProduct()">
        Set an equivalent copy
      </button>
      <button type="button" (click)="raiseProductPrice()">
        Raise price
      </button>
    </div>
    <p class="hint">
      The equivalent copy has a new object identity, but all compared fields
      match, so the product effect does not log another update. Raising the
      price changes a compared field and does trigger the effect.
    </p>
  </section>

  <section class="card" aria-labelledby="rounded-heading">
    <h2 id="rounded-heading">Equality for a computed result</h2>
    <p>
      Exact measurement: {{ measurement() }};
      rounded result: {{ roundedMeasurement().value }}.
    </p>
    <div class="button-row">
      <button type="button" (click)="setMeasurement(2.1)">Set 2.1</button>
      <button type="button" (click)="setMeasurement(2.4)">Set 2.4</button>
      <button type="button" (click)="setMeasurement(2.6)">Set 2.6</button>
    </div>
    <p class="hint">
      2.1 and 2.4 round to the same result, so the rounded-result effect does
      not need to run again. 2.6 rounds to 3 and changes the computed value.
    </p>
  </section>

  <section class="card" aria-labelledby="dynamic-heading">
    <h2 id="dynamic-heading">A dependency that changes with a branch</h2>
    <p>Profile: <strong>{{ profileSummary() }}</strong></p>
    <div class="button-row">
      <button type="button" (click)="toggleDetails()">
        {{ showDetails() ? 'Hide' : 'Show' }} age
      </button>
      <button type="button" (click)="changeProfileAge()">Increase age</button>
    </div>
    <p class="hint">
      While age is hidden, changing it does not invalidate the computed value.
      Show age to make it a dependency; hide age again to remove that dependency
      on the next evaluation.
    </p>
  </section>

  <section class="card" aria-labelledby="untracked-heading">
    <h2 id="untracked-heading">An incidental read with `untracked()`</h2>
    <p>Current audit user: <strong>{{ currentUser() }}</strong></p>
    <p>Diagnostic counter: <strong>{{ diagnosticCounter() }}</strong></p>
    <div class="button-row">
      <button type="button" (click)="switchUser()">Switch user</button>
      <button type="button" (click)="incrementDiagnosticCounter()">
        Increment diagnostic counter
      </button>
    </div>
    <p class="hint">
      The audit effect logs when the user changes. Incrementing only the
      diagnostic counter updates the template, but does not rerun that effect.
      The next user change logs the latest counter value.
    </p>
  </section>

  <footer class="page-footer">
    <p>Open the browser developer console to inspect effect messages.</p>
  </footer>
</main>
~~~

## 3. Add simple styles

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
  box-sizing: border-box;
  width: min(100% - 2rem, 52rem);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header,
.card,
.page-footer {
  margin-block-end: 1.25rem;
}

.eyebrow {
  color: #3156a6;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.card {
  padding: 1.25rem;
  border: 1px solid #d6deeb;
  border-radius: 0.75rem;
  background: #fff;
}

h1,
h2 {
  line-height: 1.2;
}

button {
  margin: 0.25rem 0.5rem 0.25rem 0;
  padding: 0.6rem 0.8rem;
  border: 0;
  border-radius: 0.4rem;
  color: #fff;
  background: #244b9a;
  font: inherit;
  cursor: pointer;
}

button:focus-visible {
  outline: 3px solid #f2a900;
  outline-offset: 2px;
}

.hint {
  color: #526078;
  line-height: 1.55;
}

.page-footer {
  padding-block: 0.5rem;
}
~~~

## 4. Run and explore

1. Start the app with `ng serve` and open the local page.
2. Open the browser developer console. Each effect logs once during its initial run.
3. Click **Set an equivalent copy**. The product comparator considers the object fields equivalent, so its effect does not log a new product update.
4. Click **Raise price**. A compared field changed, so the product signal updates and its effect logs the new product.
5. Set the measurement to `2.4`. The exact source changes, but the rounded result remains `2`, so the rounded-result effect does not need to log again. Set it to `2.6`; the result becomes `3` and the effect logs.
6. Keep the profile age hidden and click **Increase age**. The computed output remains the name because age is not a current dependency. Click **Show age** to evaluate the other branch and display the latest age.
7. Increment the diagnostic counter. The template updates, but the audit effect does not log. Switch the user; the effect runs and reads the latest counter without tracking it.

## 5. Try these changes

- Add a `description` field to `Product`. Decide whether it affects the comparator. If the template or any effect uses it, the comparator must account for it.
- Add another conditional signal to `profileSummary`, then test whether it is tracked in each branch.
- Remove `untracked()` from the audit effect. Incrementing the diagnostic counter should now trigger it, because that signal becomes a tracked dependency.
- Replace the custom product comparator with the default. Setting an equivalent object copy will now count as a change because it has a new identity.

## Concepts to notice

- Equality and dependency tracking are separate: equality decides whether a value update propagates; the current dependency set decides which updates can invalidate a consumer.
- The custom product comparator expresses which product fields count as observable changes.
- The computed equality compares derived results, not the source measurements.
- Conditional reads add and remove dependencies as the computed branch changes.
- `untracked()` reads the latest counter value but does not make the audit effect depend on it.
- Immutable updates create new object values rather than mutating a value held by a signal.

## Official references

- [Angular signals guide: equality and dynamic dependencies](https://angular.dev/guide/signals)
- [Angular `signal()` API](https://angular.dev/api/core/signal)
- [Angular `CreateSignalOptions` API](https://angular.dev/api/core/CreateSignalOptions)
- [Angular `CreateComputedOptions` API](https://angular.dev/api/core/CreateComputedOptions)
- [Angular `untracked()` API](https://angular.dev/api/core/untracked)
