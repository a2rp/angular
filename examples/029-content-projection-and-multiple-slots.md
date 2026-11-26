# Examples: Content projection and multiple slots

Build a reusable card with three named slots: title, body, and actions. The consumer supplies different markup to each slot. A second card leaves some slots empty so you can see fallback content. A projected button calls a method in the component that declared the button.

## 1. Create the card component

Create `src/app/content-card.ts`:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-content-card',
  standalone: true,
  templateUrl: './content-card.html',
  styleUrl: './content-card.css',
})
export class ContentCard {}
~~~

Create `src/app/content-card.html`:

~~~html
<article class="card">
  <header class="card-header">
    <ng-content select="[card-title]">Untitled card</ng-content>
  </header>

  <section class="card-body">
    <ng-content select="[card-body]">No body content was provided.</ng-content>
  </section>

  <footer class="card-footer">
    <ng-content select="[card-actions]">No actions available.</ng-content>
  </footer>
</article>
~~~

Create `src/app/content-card.css`:

~~~css
.card {
  overflow: hidden;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: white;
  box-shadow: 0 0.25rem 0.75rem rgb(15 23 42 / 8%);
}

.card-header,
.card-body,
.card-footer {
  padding: 1rem 1.25rem;
}

.card-header {
  border-bottom: 1px solid #e2e8f0;
  background: #f8fafc;
}

.card-body {
  min-height: 3rem;
}

.card-footer {
  border-top: 1px solid #e2e8f0;
  background: #f8fafc;
}
~~~

The `select` attributes match markers on the consumer's projected elements. The text inside each `<ng-content>` is fallback content for that slot.

## 2. Supply content from the parent

Create or adapt the root component at `src/app/app.ts`:

~~~ts
import { Component, signal } from '@angular/core';
import { ContentCard } from './content-card';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ContentCard],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class AppComponent {
  readonly notice = signal('No card action has been used yet.');

  upgradePlan(): void {
    this.notice.set('The upgrade action was handled by the parent.');
  }
}
~~~

Create `src/app/app.html`:

~~~html
<main class="page">
  <h1>Content projection demo</h1>
  <p>Each card supplies its own title, body, and optional actions.</p>

  <app-content-card>
    <h2 id="storage-title" card-title>Storage</h2>

    <p card-body>
      You have used 4 GB of your 10 GB plan. Projected content appears in the
      matching card slots.
    </p>

    <div card-actions>
      <button type="button" (click)="upgradePlan()">Upgrade plan</button>
    </div>
  </app-content-card>

  <app-content-card>
    <p card-body>This card has no title or action, so those slots use fallback content.</p>
  </app-content-card>

  <p class="notice" role="status" aria-live="polite">{{ notice() }}</p>
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
  display: grid;
  gap: 1rem;
  width: min(100% - 2rem, 44rem);
  margin: 2rem auto;
}

.page h1,
.page > p {
  margin: 0;
}

.page > p:not(.notice) {
  margin-top: -0.5rem;
  color: #475569;
}

[card-title] {
  margin: 0;
  font-size: 1.2rem;
}

[card-body] {
  margin: 0;
  line-height: 1.6;
}

[card-actions] button {
  padding: 0.55rem 0.8rem;
  border: 0;
  border-radius: 0.4rem;
  background: #1d4ed8;
  color: white;
  font: inherit;
  cursor: pointer;
}

[card-actions] button:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 2px;
}

.notice {
  min-height: 1.5em;
  color: #0f766e;
}
~~~

If your existing root component uses different filenames or a different class name, keep that setup and add the `ContentCard` import and the same projected markup. Keep the Angular CLI-generated `main.ts` and application configuration wired to your root component.

## 3. Run the app and inspect the slots

Start the development server using the command configured by your Angular project, commonly:

~~~sh
ng serve
~~~

Open the local address printed by the CLI.

- The first card shows the supplied title, body, and action in separate regions.
- The second card shows `Untitled card` and `No actions available.` because those named slots have no matching projected content.
- Click **Upgrade plan**. The projected button invokes `upgradePlan()` from the parent component, because that is where the button markup was declared.

## 4. Try these variations

- Remove the `card-body` slot marker from the card template and see what happens to the paragraph. With no matching slot or default `<ng-content>`, it is not projected into the card.
- Add an unqualified `<ng-content />` to the card template to receive content that does not match the named slots.
- Change the order of the three placeholders to change where the supplied content appears in the card layout.
- Replace `card-title` on the `h2` with `ngProjectAs="[card-title]"` and keep the title slot selector unchanged.
- Put `@if` around a caller-provided child in the parent template. Keep the card's `<ng-content>` placeholders unconditional; use an `ng-template` fragment if the card itself must instantiate caller content conditionally.
- Add a consumer-supplied CSS class or component to the body slot and observe that its bindings still refer to the parent that declared it.

## Expected behavior

- `[card-title]`, `[card-body]`, and `[card-actions]` content appears in the matching slots.
- Missing title and action content displays the slot fallback.
- Content that matches no named slot has no destination unless the card adds a default `<ng-content>`.
- The projected action button calls the parent component's handler.
- The card controls layout while the consumer controls the supplied markup.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="028-parent-child-communication-and-one-way-data-flow.md">028 - 28. Parent-child communication and one-way data flow</a></td><td><a href="../lessons/029-content-projection-and-multiple-slots.md">Lesson</a></td><td><a href="../qnas/029-content-projection-and-multiple-slots.md">Q&amp;A</a></td><td><a href="030-view-queries-and-content-queries.md">030 - 30. View queries and content queries</a></td>
</tr>
  </tbody>
</table>
