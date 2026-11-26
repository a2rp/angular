# Examples: Selectors and the Component Host Element

This walkthrough adds a status indicator using a custom element selector and a save action using an attribute selector on a native button. It shows where Angular renders each component, and how host metadata applies attributes and classes to the matched host element.

Use the Angular workspace from earlier chapters. Keep the generated root component structure and adapt filenames or existing imports to the project.

## 1. Create a status component

Create src/app/status-indicator.ts:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-status-indicator',
  standalone: true,
  templateUrl: './status-indicator.html',
  styleUrl: './status-indicator.css',
  host: {
    role: 'status',
    '[attr.aria-label]': 'accessibleLabel',
    '[class.is-online]': 'online',
  },
})
export class StatusIndicator {
  readonly accessibleLabel = 'Service status: online';
  readonly online = true;
  readonly statusText = 'Online';
}
~~~

The app-status-indicator element is the component selector and becomes its host. The host metadata puts a role, accessible label, and conditional class on that outer element.

Create src/app/status-indicator.html:

~~~html
<span class="dot" aria-hidden="true"></span>
<span class="text">{{ statusText }}</span>
~~~

Create src/app/status-indicator.css:

~~~css
:host {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.4rem 0.7rem;
  border: 1px solid #a8d8bb;
  border-radius: 999px;
}

:host(.is-online) {
  background: #e4f5eb;
  color: #176b45;
}

.dot {
  width: 0.55rem;
  height: 0.55rem;
  border-radius: 50%;
  background: currentColor;
}
~~~

The :host rules target the app-status-indicator element. The .dot rule targets a span rendered by its template.

## 2. Create a component that enhances a native button

Create src/app/save-action.ts:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'button[appSaveAction]',
  standalone: true,
  template: '<span>Save</span>',
  styles: [
    ':host { padding: 0.55rem 0.9rem; border: 0; border-radius: 0.4rem; background: #155eef; color: white; font: inherit; cursor: pointer; }',
  ],
  host: {
    class: 'app-save-action',
  },
})
export class SaveAction {}
~~~

This component uses an attribute selector that matches a native button carrying appSaveAction. The matched button remains the host, so it retains native button behavior. The component supplies the label inside the button.

## 3. Use both components from the root template

Update the root component, commonly src/app/app.ts:

~~~ts
import { Component } from '@angular/core';
import { SaveAction } from './save-action';
import { StatusIndicator } from './status-indicator';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [StatusIndicator, SaveAction],
  templateUrl: './app.html',
})
export class App {}
~~~

Keep any existing root imports the app still uses. Add both components to its imports array because the root template uses both selectors.

Update the root template, commonly src/app/app.html:

~~~html
<main>
  <h1>Component host elements</h1>
  <app-status-indicator />
  <p>
    <button appSaveAction type="button"></button>
  </p>
</main>
~~~

The custom element selector is used as an element. The attribute selector is placed on a native button. The button has type="button" so it does not submit a surrounding form by default.

## 4. Run and inspect the page

From the Angular workspace root, start the development server:

~~~sh
ng serve
~~~

Open the local address printed by the CLI. In Developer Tools, inspect the Elements panel.

Conceptually, you should see DOM similar to:

~~~html
<app-status-indicator role="status" aria-label="Service status: online" class="is-online">
  <span class="dot" aria-hidden="true"></span>
  <span class="text">Online</span>
</app-status-indicator>

<button appSaveAction type="button" class="app-save-action">
  <span>Save</span>
</button>
~~~

Angular may add attributes used for style scoping and change formatting in the actual DOM. The important distinction is that the matching element is the host, and the component template is rendered inside it.

## 5. Experiment with selector matching

Try these changes separately, then restore the working code:

1. Change the custom element selector in StatusIndicator and update its use in app.html to match.
2. Remove StatusIndicator from App's imports array. The app template should report that its custom element is not available.
3. Remove the appSaveAction attribute from the button. SaveAction should no longer match that button.
4. Change the attribute selector to button[appSaveAction][type="button"], then confirm the current button still matches because it has both attributes.
5. Try adding appSaveAction to a div. It should not match because the selector requires a button element.
6. Inspect the host's class and accessible label separately from the spans in the component view.

Angular selector matching is based on the compiled template. Adding appSaveAction to a live DOM element later with ordinary JavaScript does not create a component instance.

## 6. Review the semantics

The status indicator uses a custom element host with role="status" and an accessible label. The save action enhances a native button, preserving its built-in keyboard activation and button semantics.

Use native elements where they already express the right behavior. Add host roles and attributes only when they accurately describe the component. A selector alone does not make a component accessible.

## Troubleshooting

- If app-status-indicator is unknown, check the selector spelling and App's imports array.
- If the status class or label is missing, confirm the matching host metadata and the property names.
- If appSaveAction does not match, check that the element is a button and includes the exact attribute from the selector.
- If the component's template does not render, check the template and stylesheet paths relative to their component files.
- If the native button does not look styled, inspect its class and the :host stylesheet rule.
- If the app reports more than one component matching the same element, check whether two component selectors overlap.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="014-standalone-components-and-template-imports.md">014 - 14. Standalone components and template imports</a></td><td><a href="../lessons/015-selectors-and-the-component-host-element.md">Lesson</a></td><td><a href="../qnas/015-selectors-and-the-component-host-element.md">Q&amp;A</a></td><td><a href="016-angular-templates-and-html-structure.md">016 - 16. Angular templates and HTML structure</a></td>
</tr>
  </tbody>
</table>
