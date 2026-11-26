# Examples: Angular Templates and HTML Structure

This walkthrough builds a small semantic event page from a root component and a child card. It demonstrates that an Angular template is a view fragment, uses ordinary HTML structure, and can include Angular component elements.

The filenames follow a common CLI layout. Adapt them to your workspace and preserve existing root component configuration that your app needs.

## 1. Create the event card component

Create src/app/event-card.ts:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-event-card',
  standalone: true,
  templateUrl: './event-card.html',
  styleUrl: './event-card.css',
})
export class EventCard {}
~~~

Create src/app/event-card.html:

~~~html
<article aria-labelledby="event-title">
  <header>
    <h2 id="event-title">Angular study session</h2>
    <p>Community learning</p>
  </header>

  <p>Bring a question and work through an example with the group.</p>

  <section aria-labelledby="topics-title">
    <h3 id="topics-title">Topics</h3>
    <ul>
      <li>Component templates</li>
      <li>Semantic HTML</li>
      <li>Accessible page structure</li>
    </ul>
  </section>
</article>
~~~

The file contains the event card's view, not a separate html, head, or body document shell. The article has its own heading and content, so it can be understood as a self-contained piece of information.

Create src/app/event-card.css:

~~~css
article {
  max-width: 42rem;
  padding: 1.25rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: #ffffff;
  color: #172033;
}

header p {
  color: #475569;
}

h2,
h3 {
  line-height: 1.25;
}

li + li {
  margin-block-start: 0.35rem;
}
~~~

## 2. Compose the card inside the page

Update the root component, commonly src/app/app.ts:

~~~ts
import { Component } from '@angular/core';
import { EventCard } from './event-card';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [EventCard],
  templateUrl: './app.html',
})
export class App {}
~~~

Keep any existing dependencies in the root component's imports array. EventCard is listed because the root template uses its selector.

Update src/app/app.html:

~~~html
<main>
  <header>
    <h1>Upcoming events</h1>
    <p>Learning activities for this month.</p>
  </header>

  <section aria-labelledby="events-title">
    <h2 id="events-title">Events</h2>
    <app-event-card />
  </section>

  <footer>
    <p>Questions? Contact help&#64;example.com.</p>
  </footer>
</main>
~~~

The main element identifies the page's primary content. The sections and headings communicate its organization. The app-event-card element is a self-closing Angular component tag; Angular compiles it because EventCard is in the root component's template imports.

The escaped at-sign displays a literal @ in the email address without confusing Angular's template syntax.

## 3. Check the document shell

Open the workspace index.html. It should contain the root component host, commonly:

~~~html
<body>
  <app-root></app-root>
</body>
~~~

Use the actual selector from your root component metadata. Do not copy the main page markup into index.html; the root component template is responsible for that view.

## 4. Run the application

From the directory containing angular.json, run:

~~~sh
ng serve
~~~

Open the local address reported in the terminal. The page should show the title, the event section, the card, and the contact line.

## 5. Inspect the rendered structure

Open the browser Developer Tools and inspect the Elements panel. Find the main element and the child component host. The DOM will be similar to:

~~~html
<app-root>
  <main>
    ...
    <section aria-labelledby="events-title">
      <h2 id="events-title">Events</h2>
      <app-event-card>
        <article aria-labelledby="event-title">
          ...
        </article>
      </app-event-card>
    </section>
    ...
  </main>
</app-root>
~~~

Angular may add comments or attributes used for rendering and style scoping. The Elements panel shows the rendered DOM, not a verbatim copy of the source file.

## 6. Review HTML void elements

Void elements do not have a closing tag. For example, an image can be written as:

~~~html
<img src="assets/event-icon.svg" alt="Calendar">
~~~

Use an existing asset path if you add an image to the page. Provide useful alternative text when the image conveys information; use an empty alt value when it is purely decorative. Do not write a separate closing img tag.

## 7. Experiment with structure and comments

Make one change at a time and restore a clean version afterward:

1. Add an HTML comment to event-card.html and confirm it is not visible as page text.
2. Change an h2 to a p and notice how the page's semantic heading structure changes even if CSS makes them look similar.
3. Move a closing section tag and observe the compiler diagnostic or changed DOM.
4. Add ordinary indentation and line breaks around elements; do not rely on them for exact visual spacing.
5. Try writing a literal @ directly in template text, then use &#64; or &commat; to display it safely.
6. Temporarily try putting a script element in a component template. Angular does not support script elements there; remove it and keep code in TypeScript or the workspace's configured scripts.

## 8. Optional Angular whitespace entity

Angular usually collapses or removes whitespace it considers unnecessary. If you need one explicit preserved space between adjacent inline text, Angular supports the special &ngsp; entity:

~~~html
<p>Blue&ngsp;Sky</p>
~~~

Use this only when a specific text space is required. For layout and spacing between blocks, use semantic structure and CSS.

## Troubleshooting

- If app-event-card is unknown, check that EventCard is imported by the root component and listed in its metadata imports array.
- If the child view is missing, confirm templateUrl points to event-card.html relative to event-card.ts.
- If Angular reports a template parse error, check the reported line and verify the nesting and closing tags.
- If visual spacing differs from source indentation, use CSS layout and spacing rules.
- If the browser DOM includes extra nodes or attributes, compare the rendered Elements panel with Angular's compilation behavior instead of treating every generated node as source markup.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="015-selectors-and-the-component-host-element.md">015 - 15. Selectors and the component host element</a></td><td><a href="../lessons/016-angular-templates-and-html-structure.md">Lesson</a></td><td><a href="../qnas/016-angular-templates-and-html-structure.md">Q&amp;A</a></td><td><a href="017-interpolation-and-template-expressions.md">017 - 17. Interpolation and template expressions</a></td>
</tr>
  </tbody>
</table>
