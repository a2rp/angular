# Examples: View queries and content queries

Build a search panel that uses both query categories:

- A **view query** finds the panel's own search input so its button can focus it.
- A **content query** finds shortcut directives supplied by the parent and lists their labels.

The search panel does not inspect another component's private template. Its view query targets its own input; its content query targets directives on the parent's projected buttons.

## 1. Create a directive for projected shortcuts

Create `src/app/search-shortcut.ts`:

~~~ts
import { Directive, input } from '@angular/core';

@Directive({
  selector: 'button[app-search-shortcut]',
  standalone: true,
})
export class SearchShortcut {
  readonly shortcutLabel = input.required<string>({
    alias: 'app-search-shortcut',
  });
}
~~~

The directive identifies shortcut buttons and provides a required label that the panel can query.

## 2. Create the search panel

Create `src/app/search-panel.ts`:

~~~ts
import {
  Component,
  computed,
  contentChildren,
  ElementRef,
  viewChild,
} from '@angular/core';
import { SearchShortcut } from './search-shortcut';

@Component({
  selector: 'app-search-panel',
  standalone: true,
  templateUrl: './search-panel.html',
  styleUrl: './search-panel.css',
})
export class SearchPanel {
  readonly searchInput = viewChild<ElementRef<HTMLInputElement>>('searchInput');
  readonly shortcuts = contentChildren(SearchShortcut);
  readonly shortcutLabels = computed(() =>
    this.shortcuts().map((shortcut) => shortcut.shortcutLabel()),
  );

  focusSearch(): void {
    this.searchInput()?.nativeElement.focus();
  }
}
~~~

Create `src/app/search-panel.html`:

~~~html
<section class="panel" aria-labelledby="search-panel-title">
  <h2 id="search-panel-title">Search panel</h2>

  <label for="panel-search">Search</label>
  <input #searchInput id="panel-search" type="search" />

  <button class="focus-button" type="button" (click)="focusSearch()">
    Focus the search field
  </button>

  <div class="projected-content">
    <ng-content />
  </div>

  <section class="shortcut-list" aria-labelledby="shortcut-heading">
    <h3 id="shortcut-heading">Projected shortcuts</h3>

    @if (shortcutLabels().length === 0) {
      <p>No shortcut buttons were projected.</p>
    } @else {
      <ul>
        @for (label of shortcutLabels(); track label) {
          <li>{{ label }}</li>
        }
      </ul>
    }
  </section>
</section>
~~~

Create `src/app/search-panel.css`:

~~~css
.panel {
  display: grid;
  gap: 0.75rem;
  padding: 1.25rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: white;
  box-shadow: 0 0.25rem 0.75rem rgb(15 23 42 / 8%);
}

.panel h2,
.panel h3 {
  margin: 0;
}

.panel label {
  font-weight: 600;
}

.panel input {
  max-width: 24rem;
  padding: 0.6rem 0.7rem;
  border: 1px solid #94a3b8;
  border-radius: 0.4rem;
  font: inherit;
}

.panel input:focus-visible {
  outline: 3px solid #38bdf8;
  outline-offset: 2px;
}

.focus-button {
  justify-self: start;
  padding: 0.55rem 0.8rem;
  border: 0;
  border-radius: 0.4rem;
  background: #1d4ed8;
  color: white;
  font: inherit;
  cursor: pointer;
}

.shortcut-list ul {
  margin-bottom: 0;
}
~~~

The panel's `viewChild()` locator is the `#searchInput` reference in its own template. `contentChildren()` finds `SearchShortcut` directives in the projected content and tracks them as a signal.

## 3. Supply projected shortcut buttons from the parent

Create or adapt the root component at `src/app/app.ts`:

~~~ts
import { Component, signal } from '@angular/core';
import { SearchPanel } from './search-panel';
import { SearchShortcut } from './search-shortcut';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [SearchPanel, SearchShortcut],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class AppComponent {
  readonly message = signal('Choose a shortcut to see the parent handler run.');

  chooseShortcut(label: string): void {
    this.message.set('The parent handled shortcut: ' + label);
  }
}
~~~

Create `src/app/app.html`:

~~~html
<main class="page">
  <h1>View and content query demo</h1>
  <p class="intro">
    The panel focuses its own input and queries shortcut directives supplied here.
  </p>

  <app-search-panel>
    <p class="help-text">These buttons are projected from the parent template:</p>

    <div class="shortcut-buttons">
      <button
        app-search-shortcut="Ctrl+K"
        type="button"
        (click)="chooseShortcut('Ctrl+K')"
      >
        Search all pages
      </button>

      <button
        app-search-shortcut="Ctrl+/"
        type="button"
        (click)="chooseShortcut('Ctrl+/')"
      >
        Show keyboard help
      </button>
    </div>
  </app-search-panel>

  <p class="message" role="status" aria-live="polite">{{ message() }}</p>
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
.intro,
.message {
  margin: 0;
}

.intro {
  margin-top: -0.5rem;
  color: #475569;
}

.help-text {
  margin: 0 0 0.5rem;
}

.shortcut-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.shortcut-buttons button {
  padding: 0.5rem 0.75rem;
  border: 1px solid #94a3b8;
  border-radius: 0.4rem;
  background: #f8fafc;
  color: #172033;
  font: inherit;
  cursor: pointer;
}

.shortcut-buttons button:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 2px;
}

.message {
  color: #0f766e;
}
~~~

If your existing root component uses different filenames or a different class name, keep that setup and add the same imports, handlers, and `<app-search-panel>` markup. Keep the Angular CLI-generated `main.ts` and application configuration wired to your root component.

## 4. Run the app and observe each query

Start the development server using your Angular project's configured command, commonly:

~~~sh
ng serve
~~~

Open the local address printed by the CLI.

- Click **Focus the search field**. The `viewChild()` query finds the panel's internal input, and the panel focuses it.
- Read the **Projected shortcuts** list. The `contentChildren()` query found the directive instances on the buttons supplied by the parent.
- Click either projected shortcut. Its click binding calls `chooseShortcut()` in the parent, because the parent declared that projected markup.

## 5. Try these variations

- Remove one `app-search-shortcut` button and see the content query's labels update.
- Remove both shortcut directives and observe the empty-query message.
- Change the panel's `contentChildren(SearchShortcut)` to `contentChild(SearchShortcut)` if the component should query only one matching shortcut.
- Add a required `viewChild` only if the matching element is guaranteed to exist; keep an optional query when it may be conditionally rendered.
- Add another component inside the parent and put a shortcut directive in that component's private template. The search panel's content query will not pierce into that separate component template.
- Use `read: ElementRef` on a query when you need the DOM element for a browser operation; prefer component APIs for ordinary behavior.

## Expected behavior

- The view query focuses the search input declared by `SearchPanel`.
- The content query finds only matching shortcut directives supplied by the parent.
- The query result and displayed labels reflect the current projected content.
- Projected click handlers still run in the parent component's context.
- Queries retrieve references; they do not replace inputs, outputs, or ownership of application state.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="029-content-projection-and-multiple-slots.md">029 - 29. Content projection and multiple slots</a></td><td><a href="../lessons/030-view-queries-and-content-queries.md">Lesson</a></td><td><a href="../qnas/030-view-queries-and-content-queries.md">Q&amp;A</a></td><td><a href="031-lifecycle-hooks-and-component-initialization.md">031 - 31. Lifecycle hooks and component initialization</a></td>
</tr>
  </tbody>
</table>
