# 34. Global styles, CSS variables, and theming

Angular applications use both **global styles**, which can affect the whole application, and **component styles**, which are scoped according to the component's view encapsulation mode. CSS custom properties (often called CSS variables) provide a useful bridge: define shared theme tokens globally, then let components consume them without reaching into each other's templates.

## Learning goals

After this lesson, you should be able to:

- Locate and configure an Angular application's global stylesheet.
- Choose which rules belong globally and which belong to a component.
- Define and consume CSS custom properties with fallbacks and inheritance.
- Build light and dark themes with semantic design tokens and a `data-theme` attribute.
- Respect the operating system's color preference when appropriate.
- Understand Angular's optional CSS custom-property namespacing and its limitations.

## Global styles in an Angular workspace

A CLI application commonly uses `src/styles.css` as its global stylesheet. The Angular CLI registers it in the application's build configuration, usually in the `styles` option in `angular.json`. Global styles can provide resets, base typography, page defaults, and application-wide tokens.

~~~css
/* src/styles.css */
*,
*::before,
*::after {
  box-sizing: border-box;
}

html {
  font-family: system-ui, sans-serif;
}

body {
  margin: 0;
}
~~~

Keep rules in a global stylesheet only when they should reach across the application. Generic global selectors such as `button`, `h2`, or `.title` can affect many components. Prefer deliberate base styles, semantic tokens, and namespaced utility classes over broad one-off rules.

A component's own `styles` or `styleUrl` remains associated with that component and follows its encapsulation setting. In the default `Emulated` mode, external global rules can still match elements in component templates; see [Component styles and view encapsulation](033-component-styles-and-view-encapsulation.md).

## CSS custom properties

CSS custom properties are runtime CSS values whose names begin with `--`. Define one on an element and read it with `var()`:

~~~css
:root {
  --color-brand: rebeccapurple;
}

button {
  background-color: var(--color-brand);
}
~~~

Unlike Sass variables, custom properties remain in the generated CSS and can change at runtime. They participate in the CSS cascade and normally inherit from an element to its descendants. A value declared on `:root` is available throughout the document unless a closer scope overrides it.

Provide a fallback when a token may not be defined:

~~~css
.card {
  border-color: var(--color-border, #cbd5e1);
}
~~~

A component can consume a token without knowing which theme supplied its value:

~~~css
/* profile-card.css */
:host {
  display: block;
  color: var(--color-text, #1f2937);
  background: var(--color-surface, white);
  border: 1px solid var(--color-border, #cbd5e1);
}
~~~

For a custom property to cross a component boundary, make it inherit through the DOM or set it on an ancestor or host that contains the component. Shadow DOM also allows inherited properties and custom properties to pass through its host; ordinary selectors still cannot cross that boundary.

## Use semantic design tokens

Name theme tokens by their role in the interface, such as `--color-page`, `--color-surface`, `--color-text`, and `--color-action`. Components can then use those roles without choosing between literal light and dark colors themselves.

~~~css
:root {
  --color-page: #f5f7fb;
  --color-surface: #ffffff;
  --color-text: #202531;
  --color-muted: #5b6473;
  --color-border: #d7dce5;
  --color-action: #5b36c8;
}

body {
  color: var(--color-text);
  background: var(--color-page);
}

.panel {
  color: var(--color-text);
  background: var(--color-surface);
  border: 1px solid var(--color-border);
}
~~~

A semantic token can map to a different color in another theme while the component's stylesheet remains unchanged. This reduces duplicated selectors and keeps visual roles consistent.

## Light and dark themes with `data-theme`

A theme can be selected by adding a `data-theme` attribute to an ancestor. The theme's custom properties then inherit into its descendants:

~~~css
:root {
  color-scheme: light;
  --color-page: #f5f7fb;
  --color-surface: #ffffff;
  --color-text: #202531;
  --color-muted: #5b6473;
  --color-border: #d7dce5;
  --color-action: #5b36c8;
}

[data-theme='light'] {
  color-scheme: light;
  --color-page: #f5f7fb;
  --color-surface: #ffffff;
  --color-text: #202531;
  --color-muted: #5b6473;
  --color-border: #d7dce5;
  --color-action: #5b36c8;
}

[data-theme='dark'] {
  color-scheme: dark;
  --color-page: #141820;
  --color-surface: #202632;
  --color-text: #f3f5f8;
  --color-muted: #b3bdcb;
  --color-border: #3a4656;
  --color-action: #c4b5fd;
}
~~~

The `color-scheme` property lets browser-provided controls and surfaces use a matching light or dark scheme. It does not theme your custom components by itself; those should use the tokens too.

Angular can bind a theme to a stable element in its template:

~~~ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <main [attr.data-theme]="theme()">
      <button type="button" (click)="toggleTheme()">
        Switch theme
      </button>
      <app-profile-card></app-profile-card>
    </main>
  `,
})
export class AppComponent {
  readonly theme = signal<'light' | 'dark'>('light');

  toggleTheme(): void {
    this.theme.update((current) => current === 'light' ? 'dark' : 'light');
  }
}
~~~

~~~css
/* app.css */
:host {
  display: block;
  min-height: 100vh;
}

main {
  min-height: 100vh;
  padding: 2rem;
  color: var(--color-text);
  background: var(--color-page);
}
~~~

Place the theme attribute on an ancestor of every region that should share the theme. A theme set on `<main>` affects that subtree; it does not change siblings outside `<main>` or the browser chrome. If the entire document must change, arrange for the document root to own the attribute through an appropriate application-level mechanism.

For a system preference default, use `prefers-color-scheme` as a CSS default and let an explicit theme attribute override it:

~~~css
:root {
  color-scheme: light;
  --color-page: #f5f7fb;
  --color-text: #202531;
}

@media (prefers-color-scheme: dark) {
  :root:not([data-theme]) {
    color-scheme: dark;
    --color-page: #141820;
    --color-text: #f3f5f8;
  }
}

:root[data-theme='light'],
[data-theme='light'] {
  color-scheme: light;
  --color-page: #f5f7fb;
  --color-text: #202531;
}

:root[data-theme='dark'],
[data-theme='dark'] {
  color-scheme: dark;
  --color-page: #141820;
  --color-text: #f3f5f8;
}
~~~

This gives a CSS-based system default while allowing the application to set an explicit light or dark choice. If the user's preference is stored, initialize the theme state carefully so server rendering and the first client render do not disagree.

## Theme tokens in component styles

Component styles can consume global tokens while keeping component-specific rules local:

~~~css
/* notice.css */
:host {
  display: block;
  padding: 1rem;
  color: var(--color-text);
  background: var(--color-surface);
  border-inline-start: 0.25rem solid var(--color-action);
}

.muted {
  color: var(--color-muted);
}
~~~

The component does not need separate `.dark` and `.light` rules. It responds to whichever token values it inherits from the nearest themed ancestor.

## Dynamic values from Angular

Use CSS for static theme maps and Angular bindings when a value comes from application state. A style binding can set one CSS custom property:

~~~html
<app-chart [style.--chart-accent]="accentColor()"></app-chart>
~~~

~~~css
/* chart.css */
:host {
  display: block;
  border-color: var(--chart-accent, var(--color-action, rebeccapurple));
}
~~~

Prefer setting a small number of meaningful custom properties over generating long inline style strings. Use class bindings when the value represents a finite visual state, and CSS custom properties when the value itself is configurable, such as a color or size.

## Optional Angular CSS custom-property namespacing

When a page contains multiple independently built applications or third-party markup, applications can accidentally reuse the same CSS custom-property names. Angular provides `provideCssVarNamespacing()` in `@angular/platform-browser` for applications that need to namespace custom properties in Angular-compiled styles.

~~~ts
// app.config.ts
import { APP_ID, ApplicationConfig } from '@angular/core';
import { provideCssVarNamespacing } from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    { provide: APP_ID, useValue: 'storefront' },
    provideCssVarNamespacing(),
  ],
};
~~~

Angular uses the `APP_ID` as the prefix and appends an underscore. For example, a component stylesheet's `--primary-color` becomes `--storefront_primary-color`. You may pass a namespace directly to `provideCssVarNamespacing('storefront')`; Angular appends the underscore. When several Angular applications share one page, give each app a distinct `APP_ID` or explicit namespace.

This API is optional. An application that owns its page usually does not need it. Before enabling it, check how global themes and installed libraries share CSS custom properties.

### Global styles and namespaced component styles

Angular rewrites custom-property names in styles it compiles for components, including component styles and `[style.--token]` bindings. The ordinary global stylesheet loaded by the build is not rewritten. Thus a global declaration `--color-text` may no longer match a component reference that Angular rewrites to `--storefront_color-text`.

To opt a custom property out of Angular namespacing inside a compiled component stylesheet, prefix its name with `--global--`. Angular removes the `--global` marker and leaves the custom property name unchanged:

~~~css
/* Compiled component stylesheet */
:host {
  color: var(--global--color-text, #202531);
}
~~~

That reference remains `var(--color-text)` and can read the global token. Use exactly two hyphens after `global`.

Namespacing has exceptions. Angular does not rewrite names in static `style` attributes, object or string `[style]` bindings, `ngStyle`, custom-property names embedded inside another binding's value, or `Renderer2.setStyle()` calls. Prefer a direct property binding such as `[style.--accent]="accent()"`, which Angular can process. If code must set a custom property through a DOM API, use Angular's `CssVarNamespacer` to resolve the runtime name when namespacing may be enabled.

## Common mistakes

- **Putting every rule in the global stylesheet:** broad rules can affect unrelated components; keep local presentation with its component.
- **Using literal colors throughout components:** semantic custom properties make theme changes consistent.
- **Setting the theme attribute too low in the tree:** CSS custom properties only inherit to descendants, not siblings or ancestors.
- **Expecting `color-scheme` to theme custom UI:** components still need to consume the relevant tokens.
- **Enabling variable namespacing without reviewing global tokens:** Angular-compiled names can differ from names in the build's global stylesheet.
- **Assuming namespacing rewrites every style path:** several runtime and inline style paths are explicitly not rewritten.
- **Giving multiple apps the same namespace:** shared-page applications need distinct IDs if their token names must be isolated.
- **Persisting a theme without considering server rendering:** the server and browser should agree on the initial theme to avoid a visible flash or hydration mismatch.

## Practical checklist

Before adding a global rule or theme token, ask:

- Should this apply to the whole application or only one component?
- Is the variable named by its semantic role rather than its literal color?
- Is the `data-theme` attribute on an ancestor of every component that should inherit the tokens?
- Do custom controls and components use the same tokens as the page?
- Is a system preference a default, and can an explicit user choice override it?
- If CSS-variable namespacing is enabled, are global styles, library styles, and dynamic style updates compatible with it?

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="033-component-styles-and-view-encapsulation.md">033 - 33. Component styles and view encapsulation</a></td><td><a href="../qnas/034-global-styles-css-variables-and-theming.md">Q&amp;A</a></td><td><a href="../examples/034-global-styles-css-variables-and-theming.md">Example</a></td><td><a href="035-built-in-pipes-and-the-async-pipe.md">035 - 35. Built-in pipes and the async pipe</a></td>
</tr>
  </tbody>
</table>
