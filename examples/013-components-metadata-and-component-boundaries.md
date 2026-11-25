# Examples: Components, Metadata, and Component Boundaries

This walkthrough creates a small Profile Summary component and places it inside a page component. It shows how the class, @Component metadata, template, and styles fit together, and how the rendered component appears in the browser DOM.

Use the Angular workspace from the earlier chapters. The file names below follow a common CLI layout. If your project uses a different root component filename or folder structure, adapt the paths and preserve the existing application configuration.

## 1. Create a focused child component

Create a folder such as src/app/profile-summary. Add src/app/profile-summary/profile-summary.ts:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-profile-summary',
  standalone: true,
  templateUrl: './profile-summary.html',
  styleUrl: './profile-summary.css',
})
export class ProfileSummary {
  readonly heading = 'Profile summary';
  readonly name = 'Avery Morgan';
  readonly role = 'Frontend developer';
  readonly status = 'Available for projects';
}
~~~

The class contains the data needed by this view. The metadata associates the class with its template and stylesheet and gives it a selector that a parent template can use.

## 2. Add the component template

Create src/app/profile-summary/profile-summary.html:

~~~html
<article class="profile-card">
  <h2>{{ heading }}</h2>
  <p class="name">{{ name }}</p>
  <p>{{ role }}</p>
  <p class="status">{{ status }}</p>
</article>
~~~

This template renders the view owned by ProfileSummary. Its bindings read values from that component's class.

## 3. Add component styles

Create src/app/profile-summary/profile-summary.css:

~~~css
.profile-card {
  max-width: 32rem;
  padding: 1.25rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: #ffffff;
  color: #172033;
}

.name {
  margin-block: 0.5rem;
  font-size: 1.25rem;
  font-weight: 700;
}

.status {
  color: #176b45;
}
~~~

These styles are associated with ProfileSummary. Under the default emulated encapsulation, Angular scopes component selectors so they normally apply to this component's view. They are not a security boundary.

## 4. Compose it into the root page

Open the existing root component TypeScript file. In a common CLI layout it is src/app/app.ts. Import ProfileSummary and add it to the root component's imports:

~~~ts
import { Component } from '@angular/core';
import { ProfileSummary } from './profile-summary/profile-summary';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ProfileSummary],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class App {}
~~~

If your root component already imports other template dependencies, keep them and add ProfileSummary to the existing imports array.

Update the root template, commonly src/app/app.html:

~~~html
<main class="page">
  <h1>Component boundaries</h1>
  <p>This page composes a focused profile view.</p>

  <app-profile-summary />
</main>
~~~

The parent template uses the selector declared by ProfileSummary. The root imports the standalone component because that component is a dependency of the root template.

## 5. Start the application

Run this command from the workspace root:

~~~sh
ng serve
~~~

Open the address reported by Angular CLI. The page should show a heading, a short introduction, and the profile card. If the application already has a different root selector or layout, keep it and adapt only the component composition.

## 6. Inspect the component boundary

Open browser Developer Tools and inspect the page in the Elements panel. Find the custom host element:

~~~html
<app-profile-summary>
  ...
</app-profile-summary>
~~~

Angular renders the child component's view inside its host element. The host element is where the parent uses the component; the article and its content come from the child's template.

The component boundary groups the profile markup, its display values, and its styles. The parent remains responsible for the page-level heading and composition.

## 7. Experiment with the metadata

Try each change separately, observe the result, and restore the working version:

1. Change the heading or name in the ProfileSummary class and observe the rendered text.
2. Change the selector in ProfileSummary metadata, then update the parent template to use the new selector.
3. Change templateUrl to a missing file path and observe the compiler diagnostic. Restore the correct path.
4. Remove ProfileSummary from the root imports array and observe the template diagnostic. Restore the import.
5. Move a card-specific rule into the root stylesheet and compare its scope with the component stylesheet.
6. Add a second app-profile-summary element to the root template and observe that the component can be composed more than once.

Do not change template and selector names independently and expect Angular to infer the relationship; the metadata and template usage must agree.

## 8. Review the responsibility

Ask whether the profile card has a clear purpose and whether the parent page has stayed focused on page composition. If the child component grows, its view and styling can evolve within that boundary. If it needs parent-provided data or needs to notify its parent about an action, use the input and output patterns covered later in the course.

## Troubleshooting

- If Angular reports that app-profile-summary is unknown, confirm the child selector spelling and that ProfileSummary is in the parent's template imports.
- If the template cannot be found, confirm profile-summary.html is beside the TypeScript file and that templateUrl matches its name.
- If the stylesheet cannot be found, confirm the filename and styleUrl path.
- If the browser shows a blank page, check the development server terminal for compilation diagnostics and the browser Console for runtime errors.
- If the card appears but styles do not, confirm styleUrl points to the stylesheet and inspect the browser's computed styles.

## Further reading

- [Angular component guide](https://angular.dev/guide/components)
- [Component decorator API](https://angular.dev/api/core/Component)
- [Component templates](https://angular.dev/guide/templates)
- [Component host elements](https://angular.dev/guide/components/host-elements)
- [Component styling](https://angular.dev/guide/components/styling)
