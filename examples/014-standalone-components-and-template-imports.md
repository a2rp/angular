# Examples: Standalone Components and Template Imports

This walkthrough composes a page from standalone components. EventCard uses a standalone StatusPill component and the DatePipe. The root component imports only EventCard, showing that each component declares the dependencies used by its own template.

Use the Angular workspace from earlier chapters. The file names follow a common CLI layout; adapt them to your project's existing structure and preserve any generated component configuration you still need.

## 1. Create a small standalone component

Create src/app/status-pill.ts:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-status-pill',
  standalone: true,
  template: '<span class="pill">Scheduled</span>',
  styles: [
    '.pill { display: inline-block; padding: 0.25rem 0.65rem; border-radius: 999px; background: #e4f5eb; color: #176b45; font-size: 0.875rem; font-weight: 600; }',
  ],
})
export class StatusPill {}
~~~

This component has a selector and a small inline view. It has no other template dependencies.

## 2. Create an event card that uses a component and a pipe

Create src/app/event-card.ts:

~~~ts
import { DatePipe } from '@angular/common';
import { Component } from '@angular/core';
import { StatusPill } from './status-pill';

@Component({
  selector: 'app-event-card',
  standalone: true,
  imports: [DatePipe, StatusPill],
  templateUrl: './event-card.html',
  styleUrl: './event-card.css',
})
export class EventCard {
  readonly title = 'Angular study session';
  readonly startsAt = new Date(2026, 0, 15);
}
~~~

The EventCard template uses the date pipe and the app-status-pill selector. Both dependencies appear in EventCard's own imports array.

Create src/app/event-card.html:

~~~html
<article class="event-card">
  <div class="event-heading">
    <h2>{{ title }}</h2>
    <app-status-pill />
  </div>

  <p>Starts: {{ startsAt | date: 'mediumDate' }}</p>
</article>
~~~

Create src/app/event-card.css:

~~~css
.event-card {
  max-width: 36rem;
  padding: 1.25rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: #ffffff;
  color: #172033;
}

.event-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}

h2 {
  margin: 0;
}
~~~

## 3. Use the event card in the root component

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

If your generated root component already has other imports, preserve them and add EventCard to its imports array. If it already uses an inline template, keep that style and place the app-event-card element there.

Update its external template, commonly src/app/app.html:

~~~html
<main>
  <h1>Upcoming events</h1>
  <app-event-card />
</main>
~~~

The root template uses EventCard, so App imports EventCard. App does not import DatePipe or StatusPill: those belong to EventCard's template scope.

## 4. Run the app

From the directory containing angular.json, start the development server:

~~~sh
ng serve
~~~

The page should render an Upcoming events heading, a card, a formatted date, and a status pill. The exact displayed date format follows the active Angular locale.

If the app does not compile, read the CLI diagnostic and check the file paths and imports described below.

## 5. Trace template scope

Follow the dependency chain:

~~~text
App template
└── EventCard
    ├── DatePipe
    └── StatusPill
~~~

Each entry is imported where its template uses it:

- App imports EventCard.
- EventCard imports DatePipe and StatusPill.
- StatusPill needs no imports for its current template.

As an experiment, remove DatePipe from EventCard's imports array. The DatePipe name in its template should no longer be available. Restore it after observing the diagnostic.

Next, remove StatusPill from EventCard's imports. The app-status-pill element should no longer be recognized as the imported Angular component. Restore the import.

Finally, add DatePipe to App's imports while leaving it out of EventCard. This does not fix EventCard's missing dependency: App's imports apply to App's template, not to the EventCard template.

## 6. Legacy NgModule alternative

When maintaining an existing template that uses a dependency exported by CommonModule, a standalone component can import that module:

~~~ts
import { CommonModule } from '@angular/common';
import { Component } from '@angular/core';

@Component({
  selector: 'app-legacy-notice',
  standalone: true,
  imports: [CommonModule],
  template: '<p *ngIf="visible">This legacy notice is visible.</p>',
})
export class LegacyNotice {
  readonly visible = true;
}
~~~

For a new template, Angular's built-in @if syntax avoids the NgIf import. Use an NgModule import when it is the integration supported by the existing library or codebase; otherwise, prefer the specific standalone dependency when that is clearer.

## 7. Diagnose common mistakes

- If the root template cannot use app-event-card, confirm EventCard is in App's imports array.
- If event-card.html cannot use app-status-pill, confirm StatusPill is in EventCard's imports array.
- If the date pipe is unknown, confirm DatePipe is imported from @angular/common and appears in EventCard's metadata imports.
- If a class name is unresolved in TypeScript, check the top-level TypeScript import path and exported symbol name.
- If an imports property is rejected, check whether the component is marked standalone. A component declared in an NgModule cannot use standalone component imports metadata.
- If a dependency is not standalone, look for the NgModule that declares and exports it and import that module instead.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="013-components-metadata-and-component-boundaries.md">013 - 13. Components, metadata, and component boundaries</a></td><td><a href="../lessons/014-standalone-components-and-template-imports.md">Lesson</a></td><td><a href="../qnas/014-standalone-components-and-template-imports.md">Q&amp;A</a></td><td><a href="015-selectors-and-the-component-host-element.md">015 - 15. Selectors and the component host element</a></td>
</tr>
  </tbody>
</table>
