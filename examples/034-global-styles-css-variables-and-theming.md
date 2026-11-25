# Example 034: Global styles, CSS variables, and theming

This small standalone Angular app demonstrates a useful division of responsibility:

- `src/styles.css` owns the global reset, default design tokens, system color preference, and explicit theme token sets.
- The root component owns the user's theme choice and puts a `data-theme` attribute on the application shell.
- A child component reads the inherited tokens without knowing which theme is active.

The initial `system` choice follows the operating system or browser preference. Choosing Light or Dark overrides that preference for this app shell.

## 1. Create a practice app

Use a current Angular CLI release and create a CSS-based standalone app:

```sh
ng new theme-demo --style=css
cd theme-demo
ng serve --open
```

Keep the CLI-generated workspace configuration and providers. Replace or add the following source files in the generated application. If your CLI uses different component filenames, keep its generated names and put the same code in those files.

Confirm `src/styles.css` is registered in the `styles` array for the application build target in `angular.json`. A newly generated CLI app normally registers it already; do not add it twice.

## 2. Bootstrap the root component

### `src/main.ts`

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
```

If your generated `main.ts` already bootstraps `AppComponent`, keep its existing bootstrap setup and update only the import path if needed.

## 3. Define global tokens and system defaults

### `src/styles.css`

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  color-scheme: light;
  font-family: system-ui, sans-serif;
  font-synthesis: none;
  text-rendering: optimizeLegibility;

  --page-background: #f1f5f9;
  --surface: #ffffff;
  --text: #172033;
  --muted-text: #526176;
  --border: #cbd5e1;
  --accent: #1d4ed8;
  --accent-contrast: #ffffff;
  --focus-ring: #7c3aed;
}

/* Follow the OS/browser preference while no explicit theme is selected. */
@media (prefers-color-scheme: dark) {
  [data-theme='system'] {
    color-scheme: dark;
    --page-background: #0b1220;
    --surface: #151f30;
    --text: #e5edf8;
    --muted-text: #afbdd0;
    --border: #344258;
    --accent: #93c5fd;
    --accent-contrast: #10213a;
    --focus-ring: #c4b5fd;
  }
}

/* Explicit themes override the inherited system default on the app shell. */
[data-theme='light'] {
  color-scheme: light;
  --page-background: #f1f5f9;
  --surface: #ffffff;
  --text: #172033;
  --muted-text: #526176;
  --border: #cbd5e1;
  --accent: #1d4ed8;
  --accent-contrast: #ffffff;
  --focus-ring: #7c3aed;
}

[data-theme='dark'] {
  color-scheme: dark;
  --page-background: #0b1220;
  --surface: #151f30;
  --text: #e5edf8;
  --muted-text: #afbdd0;
  --border: #344258;
  --accent: #93c5fd;
  --accent-contrast: #10213a;
  --focus-ring: #c4b5fd;
}

html {
  min-width: 320px;
  min-height: 100%;
  background: var(--page-background);
  color: var(--text);
}

body {
  min-height: 100vh;
}

button,
select {
  font: inherit;
}

:focus-visible {
  outline: 3px solid var(--focus-ring);
  outline-offset: 3px;
}
```

The system theme is scoped to the app shell. In System mode its CSS rule follows `prefers-color-scheme`; the Light and Dark options set their own token sets on that same shell.

## 4. Add the theme selector to the root component

### `src/app/app.component.ts`

```ts
import { Component, signal } from '@angular/core';
import { ThemeCardComponent } from './theme-card/theme-card.component';

type ThemeChoice = 'system' | 'light' | 'dark';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ThemeCardComponent],
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export class AppComponent {
  readonly theme = signal<ThemeChoice>('system');

  onThemeChange(event: Event): void {
    const value = (event.target as HTMLSelectElement).value;

    if (value === 'system' || value === 'light' || value === 'dark') {
      this.theme.set(value);
    }
  }
}
```

### `src/app/app.component.html`

```html
<main
  class="page"
  [attr.data-theme]="theme()"
>
  <div class="content">
    <header class="intro">
      <p class="eyebrow">Angular styling example</p>
      <h1>One token set, multiple themes</h1>
      <p>
        Change the theme here. The child card inherits the same semantic CSS
        variables without needing theme-specific component styles.
      </p>
    </header>

    <section class="theme-control" aria-labelledby="theme-heading">
      <h2 id="theme-heading">Color theme</h2>
      <label for="theme-choice">Choose how this app should look</label>
      <select
        id="theme-choice"
        [value]="theme()"
        (change)="onThemeChange($event)"
      >
        <option value="system">Use system preference</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
      <p class="selection-note">
        Current setting: <strong>{{ theme() }}</strong>.
        System mode follows <code>prefers-color-scheme</code>.
      </p>
    </section>

    <app-theme-card />
  </div>
</main>
```

### `src/app/app.component.css`

```css
:host {
  display: block;
  min-height: 100vh;
}

.page {
  min-height: 100vh;
  padding: clamp(1.5rem, 5vw, 4rem) 1rem;
  background: var(--page-background);
  color: var(--text);
}

.content {
  width: min(100%, 48rem);
  margin-inline: auto;
  display: grid;
  gap: 1.5rem;
}

.intro,
.theme-control {
  display: grid;
  gap: 0.75rem;
}

.intro h1 {
  font-size: clamp(2rem, 6vw, 3.25rem);
  line-height: 1.1;
}

.intro > p:not(.eyebrow),
.selection-note {
  color: var(--muted-text);
  line-height: 1.6;
}

.eyebrow {
  color: var(--accent);
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.theme-control {
  padding: 1.25rem;
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  background: var(--surface);
}

.theme-control h2 {
  font-size: 1.1rem;
}

.theme-control label {
  color: var(--muted-text);
}

.theme-control select {
  width: min(100%, 22rem);
  min-height: 2.75rem;
  padding: 0.5rem 0.75rem;
  border: 1px solid var(--border);
  border-radius: 0.4rem;
  background: var(--surface);
  color: var(--text);
}

.selection-note code {
  color: var(--text);
}
```

The component stylesheet uses semantic token names such as `--surface` and `--text`; it does not select a theme itself. The root template binds `data-theme` to the shell, and the custom properties inherit down to the card.

## 5. Make a child component that consumes the tokens

### `src/app/theme-card/theme-card.component.ts`

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-theme-card',
  standalone: true,
  templateUrl: './theme-card.component.html',
  styleUrls: ['./theme-card.component.css'],
})
export class ThemeCardComponent {}
```

### `src/app/theme-card/theme-card.component.html`

```html
<article class="card" aria-labelledby="card-heading">
  <p class="card-label">Child component</p>
  <h2 id="card-heading">The card reads inherited tokens</h2>
  <p>
    Its background, text, border, and link color come from global CSS custom
    properties. The card does not need a separate dark-mode selector.
  </p>
  <a href="https://angular.dev/guide/components/styling">
    Read Angular's component styling guide
  </a>
</article>
```

### `src/app/theme-card/theme-card.component.css`

```css
:host {
  display: block;
}

.card {
  display: grid;
  gap: 0.75rem;
  padding: 1.5rem;
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  background: var(--surface);
  color: var(--text);
}

.card-label {
  color: var(--accent);
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.card p:not(.card-label) {
  color: var(--muted-text);
  line-height: 1.6;
}

.card a {
  width: fit-content;
  color: var(--accent);
  font-weight: 650;
}
```

## 6. Try it and inspect the cascade

1. Start the app with `ng serve --open`.
2. Select **Use system preference** and change the OS/browser light or dark preference. The root CSS media query supplies the default tokens.
3. Select **Light** or **Dark**. Angular adds that value to the `<main>` element's `data-theme` attribute; the matching CSS rule overrides the inherited tokens for the shell and its descendants.
4. Inspect `<app-theme-card>` in DevTools. Its own stylesheet references the same variables, but it does not contain theme-specific selectors.
5. Focus the select with the keyboard and confirm the visible focus outline.

The choice is held in an in-memory signal, so a page reload returns to System mode. Persistence is a separate concern; add it deliberately if the app needs to remember a preference, and account for server rendering before reading browser-only storage.

## Common adjustments

- Add tokens for spacing, typography, shadows, and status colors only when the design system needs them. Keep names semantic (`--surface`, `--text`, `--danger`) so components describe a role rather than a specific palette.
- If the whole document must change theme, put the attribute on the document root through an appropriate application service or host binding. This example scopes the theme to `<main>` on purpose; content outside that shell keeps the root/system palette.
- Angular's optional `provideCssVarNamespacing()` changes handling of custom properties in Angular-compiled component styles. Global stylesheet assets are not rewritten the same way. This runnable example does not enable that provider. If adopting it, follow the official API guidance and test global-token references in component styles, including its documented escape syntax and exceptions.
- Keep global styles for resets, document defaults, and shared tokens. Keep component styles for the layout and appearance owned by each component.

## References

- [Angular component styling](https://angular.dev/guide/components/styling)
- [provideCssVarNamespacing API](https://angular.dev/api/platform-browser/provideCssVarNamespacing)
- [APP_ID API](https://angular.dev/api/core/APP_ID)
