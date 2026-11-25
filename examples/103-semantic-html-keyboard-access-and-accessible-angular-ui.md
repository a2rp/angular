# Semantic HTML, Keyboard Access, and Accessible Angular UI — Examples

This small Angular page demonstrates a labeled search form, text-based validation, an accessible error relationship, semantic result markup, and a polite status announcement. It uses native HTML elements so basic keyboard behavior comes from the browser.

## 1. Component state and behavior

~~~ts
// accessible-search.ts
import { Component, signal } from '@angular/core';
import { FormsModule } from '@angular/forms';

interface HelpArticle {
  id: string;
  title: string;
  summary: string;
  url: string;
}

const ARTICLES: HelpArticle[] = [
  {
    id: 'profile',
    title: 'Update your profile',
    summary: 'Change your name and contact details.',
    url: '/help/profile'
  },
  {
    id: 'security',
    title: 'Protect your account',
    summary: 'Review sign-in and recovery options.',
    url: '/help/security'
  },
  {
    id: 'billing',
    title: 'Review billing details',
    summary: 'Find invoices and payment information.',
    url: '/help/billing'
  }
];

@Component({
  selector: 'app-accessible-search',
  imports: [FormsModule],
  templateUrl: './accessible-search.html'
})
export class AccessibleSearch {
  query = '';
  readonly invalid = signal(false);
  readonly results = signal<HelpArticle[]>([]);
  readonly status = signal('Enter at least two characters to search.');

  search(): void {
    const term = this.query.trim();

    if (term.length < 2) {
      this.invalid.set(true);
      this.status.set('Search needs at least two characters.');
      return;
    }

    this.invalid.set(false);

    const matches = ARTICLES.filter((article) =>
      (article.title + ' ' + article.summary)
        .toLowerCase()
        .includes(term.toLowerCase())
    );

    this.results.set(matches);
    this.status.set(
      matches.length === 1
        ? 'One result found.'
        : matches.length + ' results found.'
    );
  }
}
~~~

## 2. Accessible template

~~~html
<!-- accessible-search.html -->
<main>
  <h1>Help articles</h1>

  <form (ngSubmit)="search()" novalidate>
    <label for="help-search">Search help articles</label>
    <p id="query-help">Enter at least two characters.</p>

    <input
      id="help-search"
      name="query"
      type="search"
      [(ngModel)]="query"
      aria-describedby="query-help"
      [attr.aria-invalid]="invalid() ? 'true' : null"
      [attr.aria-describedby]="invalid() ? 'query-help query-error' : 'query-help'"
    />

    @if (invalid()) {
      <p id="query-error">Enter at least two characters before searching.</p>
    }

    <button type="submit">Search</button>
  </form>

  <p role="status" aria-live="polite">{{ status() }}</p>

  @if (results().length > 0) {
    <section aria-labelledby="results-heading">
      <h2 id="results-heading">Search results</h2>
      <ul>
        @for (article of results(); track article.id) {
          <li>
            <h3><a [href]="article.url">{{ article.title }}</a></h3>
            <p>{{ article.summary }}</p>
          </li>
        }
      </ul>
    </section>
  }
</main>
~~~

The example uses a real label, input, submit button, headings, list, and links. The error is text and is referenced from the input. The result count is exposed through a polite status region without moving keyboard focus.

## 3. Make the error relationship conditional

The template above binds aria-describedby to the help text and, when invalid, the error text as well. Avoid duplicate static and bound attributes with the same name. Use this corrected input element in the final template:

~~~html
<input
  id="help-search"
  name="query"
  type="search"
  [(ngModel)]="query"
  [attr.aria-invalid]="invalid() ? 'true' : null"
  [attr.aria-describedby]="invalid() ? 'query-help query-error' : 'query-help'"
/>
~~~

Keep keyboard focus on the search field after an invalid submission so the user can correct it. Do not clear their entry. The status region reports the result count after a successful search; it is not necessary to announce every internal calculation.

## 4. Review the interaction manually

- Tab to the search field and button; confirm focus is visible.
- Use Enter from the field to submit the form.
- Submit an empty or one-character value and confirm the error is readable and associated with the input.
- Search for matching and non-matching terms.
- Confirm results use a heading and a list of real navigation links.
- Check the status announcement with a screen reader.
- Zoom in and test a narrow viewport.

## 5. Further patterns

For a modal, menu, combobox, or tab system, use the matching native element or a tested accessibility implementation. Angular CDK and Angular Aria provide helpers for several interaction patterns. After router navigation, decide where focus should move so users can find the new page content.

## Further reading

- [Angular accessibility best practices](https://angular.dev/best-practices/a11y)
- [Angular Aria overview](https://angular.dev/guide/aria/overview)
- [Angular template bindings](https://angular.dev/guide/templates/binding)
