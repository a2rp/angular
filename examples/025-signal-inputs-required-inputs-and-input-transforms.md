# Examples: Signal inputs, required inputs, and input transforms

This practice app shows a parent passing values into a reusable profile card. The card has required and optional signal inputs, a public input alias, pure text normalization, boolean and number transforms, and a computed display name.

## 1. Create the profile card component

Create `src/app/profile-card.ts`:

~~~ts
import {
  booleanAttribute,
  Component,
  computed,
  input,
  numberAttribute,
} from '@angular/core';

function trimText(value: string | undefined): string {
  return value?.trim() ?? '';
}

@Component({
  selector: 'app-profile-card',
  standalone: true,
  templateUrl: './profile-card.html',
  styleUrl: './profile-card.css',
})
export class ProfileCard {
  readonly firstName = input.required<string>();
  readonly lastName = input('');
  readonly active = input(false, { transform: booleanAttribute });
  readonly level = input(0, { transform: numberAttribute });

  // The parent-facing name is "caption"; TypeScript uses "description".
  readonly description = input('', {
    alias: 'caption',
    transform: trimText,
  });

  readonly displayName = computed(() =>
    [this.firstName(), this.lastName()].filter(Boolean).join(' ')
  );
}
~~~

`firstName` is required. The other inputs have defaults. The parent binds the `caption` alias, while the component reads the value from `description()`. The transform normalizes the incoming description before the component uses it.

## 2. Add the card template

Create `src/app/profile-card.html`:

~~~html
<article
  class="profile-card"
  [class.profile-card--active]="active()"
  [class.profile-card--inactive]="!active()">
  <div class="profile-card__header">
    <div>
      <p class="profile-card__eyebrow">Level {{ level() }}</p>
      <h2>{{ displayName() }}</h2>
    </div>

    @if (active()) {
      <span class="status status--active">Active</span>
    } @else {
      <span class="status status--inactive">Inactive</span>
    }
  </div>

  <p class="profile-card__description">
    {{ description() || 'No description provided.' }}
  </p>
</article>
~~~

Every signal input is read by calling it. The computed `displayName()` updates when either name input changes. The `@if` block is built-in Angular template syntax and needs no directive import.

## 3. Style the card

Create `src/app/profile-card.css`:

~~~css
:host {
  display: block;
}

.profile-card {
  display: grid;
  gap: 1rem;
  min-height: 10rem;
  padding: 1.25rem;
  border: 1px solid #cbd3df;
  border-radius: 0.75rem;
  background: #fff;
}

.profile-card--active {
  border-inline-start: 0.35rem solid #23734d;
}

.profile-card--inactive {
  border-inline-start: 0.35rem solid #7a8494;
  background: #f7f8fa;
}

.profile-card__header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 1rem;
}

.profile-card__eyebrow,
.profile-card__description {
  color: #4b5565;
}

.profile-card__eyebrow {
  margin: 0 0 0.35rem;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

h2 {
  margin: 0;
  font-size: 1.25rem;
}

.profile-card__description {
  margin: 0;
}

.status {
  padding: 0.25rem 0.55rem;
  border-radius: 999px;
  font-size: 0.85rem;
  font-weight: 700;
}

.status--active {
  background: #e5f5ec;
  color: #155c39;
}

.status--inactive {
  background: #eceff3;
  color: #465161;
}
~~~

## 4. Create a parent that supplies and changes values

Create `src/app/input-demo.ts`:

~~~ts
import { Component } from '@angular/core';
import { ProfileCard } from './profile-card';

@Component({
  selector: 'app-input-demo',
  standalone: true,
  imports: [ProfileCard],
  templateUrl: './input-demo.html',
  styleUrl: './input-demo.css',
})
export class InputDemo {
  activeValue = 'true';
  levelValue = '7';

  toggleActiveValue(): void {
    this.activeValue = this.activeValue === 'true' ? 'false' : 'true';
  }

  increaseLevel(): void {
    this.levelValue = String(Number(this.levelValue) + 1);
  }
}
~~~

The parent properties intentionally hold strings so the transforms can convert them to the values the card uses.

## 5. Add the parent template

Create `src/app/input-demo.html`:

~~~html
<main class="demo">
  <h1>Signal input practice</h1>

  <p>
    The first card receives string values that are normalized by its input
    transforms. Change the parent values to see the card update.
  </p>

  <div class="controls" role="group" aria-label="Change profile inputs">
    <button
      type="button"
      [attr.aria-pressed]="activeValue === 'true'"
      (click)="toggleActiveValue()">
      Send active value: {{ activeValue }}
    </button>

    <button type="button" (click)="increaseLevel()">
      Increase level from {{ levelValue }}
    </button>
  </div>

  <p class="sent-values">
    Parent sends active as "{{ activeValue }}" and level as "{{ levelValue }}".
  </p>

  <section class="profiles" aria-label="Example profiles">
    <app-profile-card
      firstName="Mira"
      lastName="Shah"
      [active]="activeValue"
      [level]="levelValue"
      caption="  Coordinates the design system  " />

    <app-profile-card
      firstName="Noah"
      active="false"
      level="3"
      caption="  Builds accessible interfaces  " />
  </section>
</main>
~~~

Both cards provide the required `firstName`. The first card binds string values to `active` and `level`; the transforms convert them to a boolean and a number. The second card demonstrates static attribute values, including `active="false"`. Angular's boolean transform treats that literal string as false.

The whitespace around each `caption` value is removed by `trimText`. The transformed value is displayed as `description()` inside the component.

## 6. Add parent styles

Create `src/app/input-demo.css`:

~~~css
:host {
  display: block;
  color: #172033;
  font-family: system-ui, sans-serif;
}

.demo {
  display: grid;
  gap: 1rem;
  width: min(calc(100% - 2rem), 56rem);
  margin: 2rem auto;
}

h1,
p {
  margin: 0;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
}

button {
  min-height: 2.5rem;
  padding: 0.55rem 0.85rem;
  border: 1px solid #9aa7ba;
  border-radius: 0.4rem;
  background: #fff;
  color: #172033;
  font: inherit;
  cursor: pointer;
}

button:focus-visible {
  outline: 3px solid #3155a4;
  outline-offset: 2px;
}

.sent-values {
  color: #4b5565;
  font-family: ui-monospace, monospace;
}

.profiles {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 18rem), 1fr));
  gap: 1rem;
}
~~~

## 7. Render the demo from the root

Import `InputDemo` into the application's standalone root component and add its selector to the root template. For example:

~~~ts
import { Component } from '@angular/core';
import { InputDemo } from './input-demo';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [InputDemo],
  template: '<app-input-demo />',
})
export class App {}
~~~

If the generated root component already imports other components, add `InputDemo` to its existing `imports` array. If its template is in a separate HTML file, put `<app-input-demo />` in that file.

## 8. Run and explore

From the Angular application directory, start the dev server:

~~~sh
ng serve
~~~

Open the local address printed by the CLI.

- Use the first button to alternate the parent-provided string between `"true"` and `"false"`. The card reads a boolean from `active()`.
- Use the second button to increase the parent-provided level string. The card reads a number from `level()`.
- Compare the two descriptions: surrounding whitespace is trimmed by the transform.
- Notice that the first and last names combine through `computed`.
- Remove `firstName` from either card temporarily to see how Angular reports a missing required input.

## Expected result

Two profile cards appear. The first card responds to parent changes and displays a normalized name and description. The second card is inactive because its static boolean-like input is `"false"`. Both cards display numeric levels, even though their parent bindings are strings.

## Troubleshooting

- If Angular says a required input is missing, bind `firstName` at every `app-profile-card` use site.
- If `active` or `level` has the wrong displayed type, confirm the transform is declared in the input options and the component reads the signal by calling it.
- If the parent cannot bind a string to a transformed input, check the transform function's parameter type. It determines accepted template values.
- If `caption` is unknown, check that the alias matches the parent's attribute name. The component's TypeScript property is `description`.
- Keep transform functions pure; do not update component state or perform side effects from `trimText`.

## Official references

- [Accepting data with input properties](https://angular.dev/guide/components/inputs)
- [input API](https://angular.dev/api/core/input)
- [InputSignalWithTransform API](https://angular.dev/api/core/InputSignalWithTransform)
