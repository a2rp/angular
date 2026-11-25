# 33. Example: Component styles and view encapsulation

This standalone Angular app shows how three encapsulation modes affect CSS: the default `Emulated`, global `None`, and native `ShadowDom`. It also shows a global stylesheet, a selector scoped to one component, and a `None` selector that affects an element outside the component that declared it.

Use these files in a modern Angular CLI app's `src` directory. The example assumes the app's global stylesheet is `src/styles.css`; if your project uses another path, add that path to the styles list in `angular.json`. Keep any providers from your generated `app.config.ts`; the `main.ts` below is a minimal bootstrap.

## 1. Bootstrap the app

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

## 2. Add application-wide styles

~~~css
/* src/styles.css */
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  color: #1f2937;
  background: #f3f6fb;
  font-family: system-ui, sans-serif;
}

/* This global selector reaches into emulated components, but not Shadow DOM. */
h2 {
  text-decoration: underline;
  text-underline-offset: 0.2em;
}

app-emulated-preview,
app-global-preview,
app-shadow-preview {
  display: block;
  margin-block: 1rem;
}
~~~

The `h2` rule is deliberately global so you can compare its effect in the emulated and shadow components.

## 3. Create the parent view

~~~ts
// src/app/app.ts
import { Component } from '@angular/core';
import { EmulatedPreview } from './emulated-preview';
import { GlobalPreview } from './global-preview';
import { ShadowPreview } from './shadow-preview';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [EmulatedPreview, GlobalPreview, ShadowPreview],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class AppComponent {}
~~~

~~~html
<!-- src/app/app.html -->
<main class="page">
  <header>
    <p class="eyebrow">Angular styling demo</p>
    <h1>View encapsulation in practice</h1>
    <p>
      Inspect the headings and matching paragraphs in the browser's Elements
      panel. Each card explains where its styles can reach.
    </p>
  </header>

  <app-emulated-preview></app-emulated-preview>

  <p class="scope-marker outside-example">
    This paragraph has the same class as an element styled inside the emulated
    component. It should not receive that component's local purple style.
  </p>

  <app-global-preview></app-global-preview>

  <p class="none-style-proof outside-example">
    This paragraph is outside the None component, but its global selector still
    styles it.
  </p>

  <app-shadow-preview></app-shadow-preview>

  <p class="tip">
    The global font still inherits through the document. A Shadow Root blocks
    selector matching across its boundary; it does not turn off normal CSS
    inheritance.
  </p>
</main>
~~~

~~~css
/* src/app/app.css */
:host {
  display: block;
  min-height: 100vh;
}

.page {
  width: min(800px, calc(100% - 2rem));
  margin-inline: auto;
  padding-block: 2.5rem;
}

header {
  margin-bottom: 1.5rem;
}

.eyebrow {
  color: #6d28d9;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1 {
  margin-block: 0.35rem 0.75rem;
  font-size: clamp(1.8rem, 5vw, 2.6rem);
}

header p,
.tip,
.outside-example {
  line-height: 1.6;
}

.outside-example {
  padding: 0.75rem 1rem;
  border-radius: 0.4rem;
  background: white;
}

.tip {
  margin-top: 1.5rem;
  color: #4b5563;
}
~~~

## 4. Default `Emulated` encapsulation

This component does not set `encapsulation`, so Angular uses the default `ViewEncapsulation.Emulated` mode.

~~~ts
// src/app/emulated-preview.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-emulated-preview',
  standalone: true,
  templateUrl: './emulated-preview.html',
  styleUrl: './emulated-preview.css',
})
export class EmulatedPreview {}
~~~

~~~html
<!-- src/app/emulated-preview.html -->
<section class="panel">
  <p class="mode">Default: Emulated</p>
  <h2 class="scope-marker">This local title is purple</h2>
  <p>
    The component stylesheet scopes <code>.scope-marker</code> to this
    component's template. The global <code>h2</code> rule still underlines this
    heading.
  </p>
</section>
~~~

~~~css
/* src/app/emulated-preview.css */
:host {
  display: block;
}

.panel {
  padding: 1rem 1.25rem;
  border: 1px solid #c4b5fd;
  border-radius: 0.65rem;
  background: white;
}

.mode {
  margin: 0;
  color: #6d28d9;
  font-size: 0.8rem;
  font-weight: 700;
  text-transform: uppercase;
}

.scope-marker {
  color: #6d28d9;
}
~~~

The root template also contains a paragraph with `class="scope-marker"`. Angular's emulated component selector rewriting prevents this component stylesheet from selecting that outside paragraph.

## 5. Global styles from `ViewEncapsulation.None`

This component deliberately disables encapsulation. Its `.none-style-proof` selector becomes global and also matches the paragraph in `app.html` outside this component.

~~~ts
// src/app/global-preview.ts
import { Component, ViewEncapsulation } from '@angular/core';

@Component({
  selector: 'app-global-preview',
  standalone: true,
  templateUrl: './global-preview.html',
  styleUrl: './global-preview.css',
  encapsulation: ViewEncapsulation.None,
})
export class GlobalPreview {}
~~~

~~~html
<!-- src/app/global-preview.html -->
<section class="panel">
  <p class="mode">ViewEncapsulation.None</p>
  <h2>These component styles are global</h2>
  <p class="none-style-proof">
    This paragraph is inside the component and receives its global rule.
  </p>
</section>
~~~

~~~css
/* src/app/global-preview.css */
.panel {
  padding: 1rem 1.25rem;
  border: 1px solid #f0abfc;
  border-radius: 0.65rem;
  background: white;
}

.mode {
  margin: 0;
  color: #a21caf;
  font-size: 0.8rem;
  font-weight: 700;
  text-transform: uppercase;
}

.none-style-proof {
  padding: 0.75rem;
  border-inline-start: 0.25rem solid #c026d3;
  background: #fdf4ff;
}
~~~

The root page's outside paragraph has that same class. It gets the border and background because the component's stylesheet is global in `None` mode. Generic global selectors can affect unrelated components, so namespace them carefully.

## 6. Native `ShadowDom` encapsulation

This component asks Angular to render its template and CSS inside a browser Shadow Root.

~~~ts
// src/app/shadow-preview.ts
import { Component, ViewEncapsulation } from '@angular/core';

@Component({
  selector: 'app-shadow-preview',
  standalone: true,
  templateUrl: './shadow-preview.html',
  styleUrl: './shadow-preview.css',
  encapsulation: ViewEncapsulation.ShadowDom,
})
export class ShadowPreview {}
~~~

~~~html
<!-- src/app/shadow-preview.html -->
<section class="panel">
  <p class="mode">ViewEncapsulation.ShadowDom</p>
  <h2>This title is inside a Shadow Root</h2>
  <p>
    The application's global <code>h2</code> selector does not reach this
    heading. The document's inherited font still reaches the host's contents.
  </p>
</section>
~~~

~~~css
/* src/app/shadow-preview.css */
:host {
  display: block;
}

.panel {
  padding: 1rem 1.25rem;
  border: 1px solid #86efac;
  border-radius: 0.65rem;
  background: white;
}

.mode {
  margin: 0;
  color: #15803d;
  font-size: 0.8rem;
  font-weight: 700;
  text-transform: uppercase;
}

h2 {
  color: #166534;
  text-decoration: none;
}
~~~

Inspect `<app-shadow-preview>` in the Elements panel to see its shadow root. The global `h2` selector cannot enter that root, while inheritable values such as the document font can still flow through the host.

## 7. Optional experimental isolation mode

Current Angular documentation also lists `ExperimentalIsolatedShadowDom`. It is marked experimental. On Angular versions that expose it, you can evaluate the stronger boundary by changing the component metadata:

~~~ts
encapsulation: ViewEncapsulation.ExperimentalIsolatedShadowDom,
~~~

This option prevents external styles from affecting elements inside the component's shadow root as well as preventing the component's styles from leaking out. Verify the API's status and compatibility for your Angular version before relying on it.

## What to observe

1. The emulated card's local `.scope-marker` rule styles its own heading, but does not style the matching paragraph in the parent template.
2. The global `h2` rule affects the emulated and `None` headings. It does not match the heading inside the Shadow Root.
3. The `.none-style-proof` rule declared by the `None` component also styles the same class outside that component.
4. The shadow component's local CSS works inside its Shadow Root. Inherited font values still pass through the host.
5. `:host` styles the component host itself, while `.panel` styles an element in that component's template.

## Related material

- [Lesson 33: Component styles and view encapsulation](../lessons/033-component-styles-and-view-encapsulation.md)
- [Q&A 33: Component styles and view encapsulation](../qnas/033-component-styles-and-view-encapsulation.md)
- [Angular styling guide](https://angular.dev/guide/components/styling)
- [`ViewEncapsulation` API](https://angular.dev/api/core/ViewEncapsulation)
