# 32. Example: Host bindings, host listeners, and host directives

This standalone Angular app demonstrates three host-element patterns: a notice component binds its role, ARIA state, class, and style on its own host; a host event listens for Escape on `window`; and a reusable hover directive is composed into a card component with `hostDirectives`.

Use the files in a modern standalone Angular CLI app's `src` directory. Keep any providers configured by your generated `app.config.ts`; the small `main.ts` below is only for a project that bootstraps without extra providers.

## 1. Bootstrap the app

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

If the CLI project bootstraps through an application config, keep that setup and make it bootstrap `AppComponent` from `./app/app`.

## 2. Create the parent view

~~~ts
// src/app/app.ts
import { Component, signal } from '@angular/core';
import { DemoNotice } from './demo-notice';
import { HoverCard } from './hover-card';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [DemoNotice, HoverCard],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class AppComponent {
  readonly showNotice = signal(true);

  hideNotice(): void {
    this.showNotice.set(false);
  }

  showNoticeAgain(): void {
    this.showNotice.set(true);
  }
}
~~~

~~~html
<!-- src/app/app.html -->
<main class="page">
  <header>
    <p class="eyebrow">Angular host API demo</p>
    <h1>Host bindings and composed behavior</h1>
    <p>
      Host bindings affect a component's own element. Open the browser
      developer tools to inspect the elements as you interact with them.
    </p>
  </header>

  @if (showNotice()) {
    <app-demo-notice
      kind="success"
      message="Your settings were saved. Press Escape to dismiss this notice."
      (dismissed)="hideNotice()"
    ></app-demo-notice>
  } @else {
    <button type="button" class="restore" (click)="showNoticeAgain()">
      Show notice again
    </button>
  }

  <app-hover-card accessibleLabel="Example card with reusable hover behavior">
    <h2>Hover over this card</h2>
    <p>
      A host directive adds a class and accessible label to the
      <code>app-hover-card</code> element.
    </p>
    <button type="button">A button inside the card</button>
  </app-hover-card>

  <p class="tip">
    Inspect <code>&lt;app-demo-notice&gt;</code> and
    <code>&lt;app-hover-card&gt;</code> in the Elements panel to see their host
    attributes and classes change.
  </p>
</main>
~~~

~~~css
/* src/app/app.css */
:host {
  display: block;
  min-height: 100vh;
  color: #1f2937;
  background: #f3f6fb;
  font-family: system-ui, sans-serif;
}

.page {
  width: min(760px, calc(100% - 2rem));
  margin: 0 auto;
  padding: 3rem 0;
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
  margin: 0.35rem 0 0.75rem;
  font-size: clamp(1.8rem, 5vw, 2.6rem);
}

header p,
.tip {
  line-height: 1.6;
}

.restore {
  margin-bottom: 1rem;
  padding: 0.65rem 0.9rem;
  border: 0;
  border-radius: 0.45rem;
  color: white;
  background: #6d28d9;
  font: inherit;
  cursor: pointer;
}

.tip {
  margin-top: 1.5rem;
  color: #4b5563;
}
~~~

## 3. Bind state and listen on a component host

The notice component reflects its state through host classes and ARIA attributes. Its Escape listener is attached to `window`; clicking the internal button also emits a component output.

~~~ts
// src/app/demo-notice.ts
import { Component, input, output } from '@angular/core';

@Component({
  selector: 'app-demo-notice',
  standalone: true,
  templateUrl: './demo-notice.html',
  styleUrl: './demo-notice.css',
  host: {
    '[attr.role]': 'kind() === "error" ? "alert" : "status"',
    '[attr.aria-live]': 'kind() === "error" ? "assertive" : "polite"',
    '[attr.aria-atomic]': '"true"',
    '[class.notice--error]': 'kind() === "error"',
    '[style.--notice-accent]': 'kind() === "error" ? "crimson" : "seagreen"',
    '(window:keydown.escape)': 'dismiss()',
  },
})
export class DemoNotice {
  readonly kind = input<'success' | 'error'>('success');
  readonly message = input.required<string>();
  readonly dismissed = output<void>();

  dismiss(): void {
    this.dismissed.emit();
  }
}
~~~

~~~html
<!-- src/app/demo-notice.html -->
<p>{{ message() }}</p>
<button type="button" (click)="dismiss()" aria-label="Dismiss notice">
  Dismiss
</button>
~~~

~~~css
/* src/app/demo-notice.css */
:host {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1.25rem;
  padding: 1rem 1.25rem;
  border-inline-start: 0.35rem solid var(--notice-accent, seagreen);
  border-radius: 0.5rem;
  background: white;
  box-shadow: 0 6px 20px rgb(31 41 55 / 7%);
}

:host(.notice--error) {
  background: #fff5f5;
}

p {
  margin: 0;
  line-height: 1.5;
}

button {
  flex: none;
  padding: 0.5rem 0.75rem;
  border: 1px solid #9ca3af;
  border-radius: 0.4rem;
  color: inherit;
  background: transparent;
  font: inherit;
  cursor: pointer;
}

button:focus-visible {
  outline: 3px solid #c4b5fd;
  outline-offset: 2px;
}
~~~

`kind` controls the host's `aria-live`, error class, and CSS custom property. The host switches between `role="status"` and `role="alert"` with matching `aria-live` politeness, and keeps `aria-atomic="true"`. The parent removes the component when it receives `dismissed`; Angular then removes its host listener with the component instance.

## 4. Define reusable hover behavior

This attribute directive owns the hover state and its host bindings. `hostDirectives` will apply it to `app-hover-card` without needing the directive selector in the consumer's template.

~~~ts
// src/app/hover-behavior.directive.ts
import { Directive, input, signal } from '@angular/core';

@Directive({
  selector: '[appHoverBehavior]',
  standalone: true,
  host: {
    'role': 'group',
    '[attr.aria-label]': 'label()',
    '[attr.data-hovered]': 'hovered() ? "true" : "false"',
    '[class.is-hovered]': 'hovered()',
    '(pointerenter)': 'hovered.set(true)',
    '(pointerleave)': 'hovered.set(false)',
  },
})
export class HoverBehaviorDirective {
  readonly label = input('Hoverable content');
  readonly hovered = signal(false);
}
~~~

~~~ts
// src/app/hover-card.ts
import { Component } from '@angular/core';
import { HoverBehaviorDirective } from './hover-behavior.directive';

@Component({
  selector: 'app-hover-card',
  standalone: true,
  hostDirectives: [
    {
      directive: HoverBehaviorDirective,
      inputs: ['label: accessibleLabel'],
    },
  ],
  templateUrl: './hover-card.html',
  styleUrl: './hover-card.css',
})
export class HoverCard {}
~~~

~~~html
<!-- src/app/hover-card.html -->
<ng-content></ng-content>
~~~

~~~css
/* src/app/hover-card.css */
:host {
  display: block;
  margin-top: 1.25rem;
  padding: 1.25rem;
  border: 1px solid #d1d5db;
  border-radius: 0.75rem;
  background: white;
  box-shadow: 0 6px 20px rgb(31 41 55 / 7%);
  transition: border-color 120ms ease, box-shadow 120ms ease;
}

:host(.is-hovered) {
  border-color: #7c3aed;
  box-shadow: 0 10px 28px rgb(109 40 217 / 18%);
}

h2 {
  margin-top: 0;
}

button {
  padding: 0.55rem 0.8rem;
  border: 0;
  border-radius: 0.4rem;
  color: white;
  background: #6d28d9;
  font: inherit;
  cursor: pointer;
}
~~~

The component explicitly exposes the host directive's `label` input as `accessibleLabel`. The consumer uses `<app-hover-card accessibleLabel="...">`; it does not need to apply `appHoverBehavior` itself. The directive's class, `data-hovered` attribute, group role, accessible label, and pointer listeners all belong to the card host.

## 5. Try it in the browser

1. Start the Angular app and inspect `<app-demo-notice>` in the Elements panel. It has `role`, `aria-live`, and `aria-atomic` host attributes.
2. Change the notice's `kind` input in `app.html` to `error`. The host switches to `role="alert"`, and its `aria-live`, error class, and accent custom property update together.
3. Press Escape anywhere in the page or use the Dismiss button. The parent removes the notice component; its window listener is no longer attached after destruction.
4. Move the pointer over the card. The host directive updates the card's `data-hovered` attribute and class, and the component's `:host(.is-hovered)` style responds.
5. Inspect the card's accessible label. The directive input is public on the component only because `hostDirectives` explicitly exposes it as `accessibleLabel`.

## What this example demonstrates

- `host` metadata binds a component's own attributes, classes, styles, and events.
- Property state and ARIA attributes are expressed with their appropriate binding forms.
- A global host event uses the `window:` prefix and is tied to the component's lifetime.
- A standalone directive packages reusable host behavior.
- `hostDirectives` applies that behavior statically and exposes only the aliased input the component chooses to publish.
- The directive selector is not needed where the composed component is used.

## Related material

- [Lesson 32: Host bindings, host listeners, and host directives](../lessons/032-host-bindings-host-listeners-and-host-directives.md)
- [Q&A 32: Host bindings, host listeners, and host directives](../qnas/032-host-bindings-host-listeners-and-host-directives.md)
- Angular component host elements
- Angular directive composition API

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="031-lifecycle-hooks-and-component-initialization.md">031 - 31. Lifecycle hooks and component initialization</a></td><td><a href="../lessons/032-host-bindings-host-listeners-and-host-directives.md">Lesson</a></td><td><a href="../qnas/032-host-bindings-host-listeners-and-host-directives.md">Q&amp;A</a></td><td><a href="033-component-styles-and-view-encapsulation.md">033 - 33. Component styles and view encapsulation</a></td>
</tr>
  </tbody>
</table>
