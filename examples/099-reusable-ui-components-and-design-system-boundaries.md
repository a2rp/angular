# Reusable UI Components and Design-System Boundaries - Examples

This example builds two small presentation components: a Card that projects consumer content and a Notice that accepts typed data and reports a dismiss action. The consuming page owns the workflow state. Adapt file names to your Angular CLI conventions.

## 1. Set up a feature-local or design-system location

For reuse within one application, the components might live here:

~~~text
src/app/shared/ui/
  card/
    card.ts
    card.html
    card.css
  notice/
    notice.ts
    notice.html
    notice.css
~~~

If independent applications need these components, move them into a library only after agreeing on API ownership, compatibility, and accessibility behavior.

## 2. Create a Card with a title slot and body slot

~~~ts
// card.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-card',
  templateUrl: './card.html',
  styleUrl: './card.css'
})
export class CardComponent {}
~~~

~~~html
<!-- card.html -->
<article class="card">
  <header class="card__header">
    <ng-content select="[card-title]" />
  </header>
  <div class="card__body">
    <ng-content />
  </div>
</article>
~~~

~~~css
/* card.css */
:host {
  display: block;
}

.card {
  border: 1px solid var(--border-subtle);
  border-radius: var(--radius-card);
  background: var(--surface-raised);
  color: var(--text-primary);
}

.card__header {
  padding: var(--space-4);
  border-block-end: 1px solid var(--border-subtle);
}

.card__body {
  padding: var(--space-4);
}
~~~

The title slot is selected by an attribute. Remaining consumer content goes into the body. Projection placeholders are part of the component's template contract; do not put them inside conditional control-flow blocks.

## 3. Create a Notice with typed inputs and an event

~~~ts
// notice.ts
import { Component, input, output } from '@angular/core';

export type NoticeTone = 'info' | 'warning' | 'success';

@Component({
  selector: 'app-notice',
  templateUrl: './notice.html',
  styleUrl: './notice.css'
})
export class NoticeComponent {
  readonly message = input.required<string>();
  readonly tone = input<NoticeTone>('info');
  readonly dismissible = input(false);
  readonly dismissed = output<void>();
}
~~~

~~~html
<!-- notice.html -->
<aside class="notice" [attr.data-tone]="tone()" role="status">
  <p>{{ message() }}</p>
  @if (dismissible()) {
    <button type="button" (click)="dismissed.emit()">
      Dismiss
    </button>
  }
</aside>
~~~

~~~css
/* notice.css */
:host {
  display: block;
}

.notice {
  display: flex;
  align-items: start;
  justify-content: space-between;
  gap: var(--space-3);
  padding: var(--space-3);
  border-inline-start: 0.25rem solid var(--color-info);
  background: var(--surface-raised);
  color: var(--text-primary);
}

.notice[data-tone='warning'] {
  border-inline-start-color: var(--color-warning);
}

.notice[data-tone='success'] {
  border-inline-start-color: var(--color-success);
}

.notice button {
  min-height: 2.75rem;
}
~~~

The text is interpolated as text rather than treated as HTML. The button is a native button, so it receives standard keyboard activation and focus behavior. If the message needs an urgent alert rather than a polite status, define that behavior intentionally instead of applying the same role to every notice.

## 4. Compose the components in a page

~~~ts
// settings-page.ts
import { Component, signal } from '@angular/core';
import { CardComponent } from '../shared/ui/card/card';
import { NoticeComponent } from '../shared/ui/notice/notice';

@Component({
  selector: 'app-settings-page',
  imports: [CardComponent, NoticeComponent],
  templateUrl: './settings-page.html'
})
export class SettingsPage {
  readonly showNotice = signal(true);

  dismissNotice(): void {
    this.showNotice.set(false);
  }
}
~~~

~~~html
<!-- settings-page.html -->
<main>
  <h1>Settings</h1>

  @if (showNotice()) {
    <app-notice
      message="Your preferences have been saved."
      tone="success"
      [dismissible]="true"
      (dismissed)="dismissNotice()"
    />
  }

  <app-card>
    <h2 card-title>Profile preferences</h2>
    <p>Choose how the application should contact you.</p>
    <a href="/account/notifications">Manage notifications</a>
  </app-card>
</main>
~~~

The page owns whether the notice is visible. The Notice reports user intent through an output. The Card owns its surrounding structure, while the page owns its heading, copy, link, and workflow.

## 5. Define and consume design tokens

~~~css
/* styles.css */
:root {
  --space-3: 0.75rem;
  --space-4: 1rem;
  --radius-card: 0.5rem;
  --border-subtle: #707070;
  --surface-raised: #ffffff;
  --text-primary: #1b1b1b;
  --color-info: #145ea8;
  --color-warning: #8a4b00;
  --color-success: #176b36;
}
~~~

Choose tokens that preserve adequate text and focus contrast in every supported theme. CSS custom properties are a deliberate customization surface; internal class names should remain private implementation details.

## 6. Review the public contract

Before reusing or publishing these components, document:

- Which input values are accepted and what their defaults mean.
- Whether the consumer supplies title or body content.
- Which events are emitted and when.
- Keyboard and focus behavior.
- How each tone is announced and represented without relying on color alone.
- Which design tokens consumers may override.
- Which HTML content is trusted and which values are displayed as text.

Try the components with long messages, no optional projected content, keyboard-only navigation, and narrow layouts. Keep them in the settings feature if no other workflow needs them; move them into a shared library only when there is a stable cross-application contract.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="098-organize-projects-libraries-and-shared-code.md">098 - Organize Projects, Libraries, and Shared Code</a></td><td><a href="../lessons/099-reusable-ui-components-and-design-system-boundaries.md">Lesson</a></td><td><a href="../qnas/099-reusable-ui-components-and-design-system-boundaries.md">Q&amp;A</a></td><td><a href="100-dynamic-components-and-programmatic-rendering.md">100 - Dynamic Components and Programmatic Rendering</a></td>
</tr>
  </tbody>
</table>
