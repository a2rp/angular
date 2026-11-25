# Examples: Model inputs and component-level two-way binding

Build a reusable counter component whose `count` model is supplied by a parent and can also be changed by the child. The parent binds a writable signal with `[(count)]`, displays the same signal, and can reset it itself.

This example uses standalone components. Keep your Angular CLI-generated bootstrap and application configuration; connect the root component shown here using the setup already in your project.

## 1. Create the counter component

Create `src/app/counter.ts`:

~~~ts
import { Component, model } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  templateUrl: './counter.html',
  styleUrl: './counter.css',
})
export class Counter {
  readonly count = model(0);

  increment(): void {
    this.count.update((current) => current + 1);
  }

  decrement(): void {
    this.count.update((current) => Math.max(0, current - 1));
  }
}
~~~

Create `src/app/counter.html`:

~~~html
<section class="counter" aria-label="Counter controls">
  <button
    type="button"
    aria-label="Decrease count"
    [disabled]="count() === 0"
    (click)="decrement()"
  >
    −
  </button>

  <output class="value" aria-live="polite">{{ count() }}</output>

  <button type="button" aria-label="Increase count" (click)="increment()">
    +
  </button>
</section>
~~~

Create `src/app/counter.css`:

~~~css
.counter {
  display: inline-flex;
  align-items: center;
  gap: 1rem;
  padding: 0.75rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: #f8fafc;
}

button {
  width: 2.5rem;
  height: 2.5rem;
  border: 0;
  border-radius: 0.5rem;
  background: #1d4ed8;
  color: white;
  font: inherit;
  font-size: 1.25rem;
  cursor: pointer;
}

button:disabled {
  background: #94a3b8;
  cursor: not-allowed;
}

button:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 2px;
}

.value {
  min-width: 2ch;
  text-align: center;
  font-size: 1.25rem;
  font-variant-numeric: tabular-nums;
}
~~~

`count` is a model signal. The component reads it with `count()` and changes it with `.update()`. Angular also exposes the model as an input and creates a matching `countChange` output.

## 2. Bind it from a parent

Create or adapt your root component at `src/app/app.ts`:

~~~ts
import { Component, signal } from '@angular/core';
import { Counter } from './counter';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [Counter],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class AppComponent {
  readonly quantity = signal(3);

  resetCount(): void {
    this.quantity.set(0);
  }
}
~~~

Create `src/app/app.html`:

~~~html
<main class="page">
  <h1>Two-way binding with a component model</h1>
  <p>Use the child counter or change the same value from the parent.</p>

  <app-counter [(count)]="quantity" />

  <p class="parent-value">
    Parent signal value: <strong>{{ quantity() }}</strong>
  </p>

  <button class="reset" type="button" (click)="resetCount()">
    Reset from parent
  </button>
</main>
~~~

Create `src/app/app.css`:

~~~css
:host {
  display: block;
  color: #172033;
  font-family: system-ui, sans-serif;
}

.page {
  width: min(100% - 2rem, 42rem);
  margin: 2rem auto;
}

.parent-value {
  margin-block: 1.25rem 0.5rem;
}

button.reset {
  padding: 0.55rem 0.8rem;
  border: 0;
  border-radius: 0.4rem;
  background: #475569;
  color: white;
  font: inherit;
  cursor: pointer;
}

button.reset:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 2px;
}
~~~

If your existing root component has a different class or file name, keep that setup and apply the same `signal`, `Counter` import, model binding, and reset handler there. Keep the Angular CLI-generated `main.ts` and app configuration wired to your root component.

## 3. Run the example

Start the Angular development server using the script configured in your project, commonly:

~~~sh
ng serve
~~~

Open the local address printed by the CLI.

- Press **+** or **−** in the child. The child updates its `count` model, and the parent's `quantity` signal changes through the `countChange` output.
- Read the parent value below the counter. It matches the child value because both are connected by `[(count)]="quantity"`.
- Press **Reset from parent**. The parent updates `quantity`; Angular supplies the new value to the child's `count` model, and the child display changes too.
- At zero, the decrease button is disabled so the sample count stays nonnegative.

## 4. Compare the binding forms

With the parent's writable signal, use the signal instance in the two-way binding:

~~~html
<app-counter [(count)]="quantity" />
~~~

Call the signal to read its current value in the template:

~~~html
<p>{{ quantity() }}</p>
~~~

For a plain writable property, the same component model can be bound as follows:

~~~ts
quantity = 3;
~~~

~~~html
<app-counter [(count)]="quantity" />
<p>{{ quantity }}</p>
~~~

The shorthand is conceptually equivalent to a property binding and a matching change listener:

~~~html
<app-counter [count]="quantity" (countChange)="quantity = $event" />
~~~

Use `[(count)]` when the purpose is to keep one parent value and one child model synchronized. For a one-way value, use a regular input. For a separate action such as saving or submitting, use a semantic output instead of turning it into a model.

## Expected behavior

- The parent starts with a quantity of `3` and supplies it to the child.
- The child displays and changes the model value.
- The parent's signal and displayed value follow changes made by the child.
- A parent reset updates the value shown by the child.
- The two-way binding uses the signal instance as its writable target and calls the signal only when reading it.
