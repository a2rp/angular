# Example: A Small Component-Based Angular App

This exercise builds a tiny Angular application with two components. The root component owns a page title and a counter. A child greeting component owns its own display. You will see how components compose into a tree and how an event updates component state and the rendered view.

## What you will build

~~~text
App
└── GreetingCard
~~~

The app will display a page heading, a counter, a button, and a greeting card. Clicking the button increments the counter. The greeting card is a separate component with a focused purpose.

## 1. Create a practice project

Use a separate practice workspace, not this notes repository. Check the Angular version compatibility page for the Node.js version supported by the Angular release you install.

~~~sh
npm install -g @angular/cli
ng new component-playground --style=css
cd component-playground
ng serve --open
~~~

If the CLI asks setup questions, choose the simple defaults for this exercise. Wait for the starter application to run, then stop the development server with Ctrl+C before editing files.

The current CLI uses concise file names such as app.ts and app.html by default. Older workspaces may use names such as app.component.ts and app.component.html. Use your project's generated root component file names, and make sure the class exported by the root file matches the import in src/main.ts.

## 2. Create the child greeting component

Create **src/app/greeting-card.ts**:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-greeting-card',
  standalone: true,
  templateUrl: './greeting-card.html',
  styleUrl: './greeting-card.css',
})
export class GreetingCard {
  name = 'Angular learner';
}
~~~

Create **src/app/greeting-card.html**:

~~~html
<article class="greeting-card">
  <h2>Welcome, {{ name }}!</h2>
  <p>This message belongs to the greeting-card component.</p>
</article>
~~~

Create **src/app/greeting-card.css**:

~~~css
.greeting-card {
  border: 1px solid #b8c4d8;
  border-radius: 0.75rem;
  padding: 1rem;
  background: #f4f7fb;
}

.greeting-card h2 {
  margin-block: 0 0.5rem;
}
~~~

The component metadata connects the **app-greeting-card** selector to its class and files. Its own template and CSS keep the greeting's markup and presentation together.

## 3. Update the root component

Replace the generated root component class in **src/app/app.ts** with:

~~~ts
import { Component, signal } from '@angular/core';
import { GreetingCard } from './greeting-card';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [GreetingCard],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class App {
  title = 'Component playground';
  clickCount = signal(0);

  increment(): void {
    this.clickCount.update((currentCount) => currentCount + 1);
  }
}
~~~

If your generated project uses a different root class name, keep that name or update the import in **src/main.ts** to match. The component selector remains app-root because it is normally referenced by the generated host page.

The **imports** array makes GreetingCard available to this standalone component's template. A standalone component imports the components, directives, and pipes its own template uses.

## 4. Replace the root template

Set **src/app/app.html** to:

~~~html
<main class="page">
  <h1>{{ title }}</h1>
  <p class="description">
    This page is assembled from focused Angular components.
  </p>

  <section aria-labelledby="counter-heading">
    <h2 id="counter-heading">Button clicks</h2>
    <p aria-live="polite">Count: {{ clickCount() }}</p>
    <button type="button" (click)="increment()">
      Add one
    </button>
  </section>

  <app-greeting-card />
</main>
~~~

## 5. Add page styles

Set **src/app/app.css** to:

~~~css
:host {
  display: block;
  color: #182230;
  font-family: system-ui, sans-serif;
}

.page {
  max-width: 42rem;
  margin-inline: auto;
  padding: 2rem 1rem;
}

.description {
  color: #46566b;
}

button {
  border: 0;
  border-radius: 0.5rem;
  padding: 0.65rem 1rem;
  background: #2457c5;
  color: white;
  font: inherit;
  cursor: pointer;
}

button:focus-visible {
  outline: 3px solid #f2a900;
  outline-offset: 3px;
}
~~~

If your root stylesheet has an existing starter rule that hides or changes the host, remove or adapt that rule so the page remains visible. Component CSS is scoped according to Angular's style encapsulation.

## 6. Run and inspect

~~~sh
ng serve --open
~~~

You should see:

- A heading that reads **Component playground**.
- A counter beginning at **0**.
- A button labeled **Add one**.
- A greeting card that says **Welcome, Angular learner!**.
- The counter increasing by one each time you activate the button.

The button is a native HTML button, so it works with keyboard activation. The live region announces the updated count to assistive technology.

## How the pieces work together

1. Angular bootstraps the root **App** component from the application's entry point.
2. The **App** template displays values from its class and includes an app-greeting-card element.
3. The App metadata imports **GreetingCard**, so Angular can use it in this template.
4. Angular creates a GreetingCard instance for the matching element and renders its template inside the child component's host.
5. The click event calls **increment()** on App.
6. increment updates **clickCount**, and the template reads the new signal value with **clickCount()**.
7. Angular updates the rendered count without code manually changing a DOM node.

The title and count are owned by App; the greeting text is owned by GreetingCard. This is a small example of keeping each piece of state with the component responsible for it.

## Check your understanding

- **Which file defines the root component behavior?**
  The TypeScript file, app.ts.

- **Where is the page structure declared?**
  In app.html, the root component's template.

- **Why can app.html use app-greeting-card?**
  The root component imports GreetingCard, whose selector is app-greeting-card.

- **Which component owns the click count?**
  App owns it because the counter and its button are part of the root page in this example.

- **What causes the displayed number to change?**
  The event calls a method, the method updates a signal, and Angular refreshes the template that reads that signal.

- **Does this example call the backend?**
  No. It uses local component state only. Backend requests and services come in later lessons.

## Try these extensions

1. Change the greeting's name and observe which component owns that value.
2. Add a second child component, such as an app-footer, and draw the updated component tree.
3. Add a **Reset** button in App and a method that sets clickCount back to zero.
4. Open browser developer tools and inspect the rendered elements. Notice that the browser sees DOM output, not the original Angular template source.
5. Remove GreetingCard from the root component's imports and observe the Angular template diagnostic. Restore the import afterward.

## Troubleshooting

- **Unknown element error:** Check the selector spelling and confirm GreetingCard appears in the root component's imports.
- **Cannot find module error:** Confirm the greeting-card.ts path and the relative import spelling.
- **Missing template or stylesheet error:** Confirm the file names match templateUrl and styleUrl exactly. File-name casing matters on some systems and deployment environments.
- **Root component import error:** Check which class src/main.ts imports and make it match the class exported by app.ts.
- **Counter does not change:** Check that the button has (click)="increment()", the method updates clickCount, and the template reads clickCount().
- **Unexpected starter content remains:** Confirm you edited the root component template used by the project and saved the file.

## Further reading

- [Anatomy of components](https://angular.dev/guide/components)
- [Template syntax](https://angular.dev/guide/templates)
- [Signals](https://angular.dev/guide/signals)
- [Angular CLI: ng new](https://angular.dev/cli/new)
- [Version compatibility](https://angular.dev/reference/versions)
