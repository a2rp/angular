# Hydration, Event Replay, and Incremental Hydration — Examples

These snippets demonstrate the app provider, a section that hydrates on demand, a mismatch workaround, and a direct-response verification workflow. Use the Angular CLI generated SSR structure for your project version.

## 1. Provide hydration in the application configuration

A standalone application's shared app configuration can provide hydration:

~~~ts
import { ApplicationConfig } from '@angular/core';
import { provideClientHydration } from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    provideClientHydration(),
  ],
};
~~~

The generated SSR setup may already include this provider. Keep it in the configuration used by both server and browser bootstraps, as required by the generated project. Current Angular documentation enables incremental hydration and its event replay support through the hydration setup; check the installed Angular version for exact defaults.

## 2. Render a deferred component on the server and hydrate on interaction

The deferred component should be standalone and imported directly in the parent component.

comments.component.ts:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-comments',
  standalone: true,
  templateUrl: './comments.html',
})
export class CommentsComponent {}
~~~

comments.html:

~~~html
<section aria-labelledby="comments-title">
  <h2 id="comments-title">Comments</h2>
  <p>Read the latest discussion about this lesson.</p>
  <button type="button">Reply</button>
</section>
~~~

Parent component class:

~~~ts
import { Component } from '@angular/core';
import { CommentsComponent } from './comments.component';

@Component({
  selector: 'app-article-page',
  standalone: true,
  imports: [CommentsComponent],
  templateUrl: './article-page.html',
})
export class ArticlePageComponent {}
~~~

article-page.html:

~~~html
<article>
  <h1>Understanding Angular hydration</h1>
  <p>The article body is available as server-rendered HTML.</p>
</article>

@defer (hydrate on interaction) {
  <app-comments />
} @placeholder {
  <section aria-label="Comments">
    <h2>Comments</h2>
    <p>Comments are available when you open this section.</p>
  </section>
}
~~~

With incremental hydration configured, the server can render the main content of the defer block. On the initial client load, the section remains dehydrated until a supported interaction triggers hydration. The placeholder is still useful for later client-side navigations where normal defer behavior applies.

## 3. Hydrate below-the-fold content near the viewport

A section that is not needed immediately can hydrate when it enters view:

~~~html
@defer (hydrate on viewport) {
  <app-related-lessons />
} @placeholder {
  <section class="related-lessons-placeholder" aria-label="Related lessons">
    <h2>Related lessons</h2>
    <p>More lessons are available below.</p>
  </section>
}
~~~

Reserve a stable placeholder size with CSS and check the transition on different viewport sizes. Do not delay the hydration of a control that a user is likely to activate immediately.

## 4. Enable Event Replay without incremental hydration

If a project deliberately opts out of incremental hydration but still wants early event capture, current Angular APIs support this configuration:

~~~ts
import { ApplicationConfig } from '@angular/core';
import {
  provideClientHydration,
  withEventReplay,
  withNoIncrementalHydration,
} from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    provideClientHydration(
      withNoIncrementalHydration(),
      withEventReplay(),
    ),
  ],
};
~~~

Use only APIs supported by the project's Angular version. Do not add this configuration alongside a separate duplicate provideClientHydration call. Check the generated app configuration first.

## 5. Use ngSkipHydration narrowly

A legacy widget that mutates its own host DOM may temporarily need to skip hydration:

~~~html
<app-legacy-map ngSkipHydration />
~~~

Apply ngSkipHydration only to a component host. Angular recreates that subtree in the browser, so it may add work or cause visual movement. Keep the workaround isolated and remove it after replacing the direct DOM mutation with a hydration-compatible approach.

## 6. Inspect hydration in the browser

1. Build and serve an SSR or prerendered route.
2. Load the route directly in a new browser navigation.
3. Inspect the page source or document response to confirm server-rendered content.
4. Open the browser console and look for hydration mismatch warnings or errors.
5. Interact with a deferred control before hydration completes when testing Event Replay.
6. Use Angular DevTools to inspect component hydration status.
7. Confirm that client-side navigation follows the normal defer trigger behavior.

A client-side navigation from an already bootstrapped route does not exercise the same initial hydration path as a direct page load.

## 7. Check for DOM mismatch sources

If hydration fails, compare the server HTML with what reaches the browser. Remove code that inserts or moves nodes before hydration, validate HTML nesting, and check third-party scripts and CDN settings that rewrite the document.

For example, avoid rendering structurally different content on the server and client with a browser check in the template. Keep the initial markup consistent and initialize browser-only behavior after rendering using an appropriate Angular render hook.

## References

- [Angular hydration](https://angular.dev/guide/hydration)
- [Incremental hydration](https://angular.dev/guide/incremental-hydration)
- [Deferrable views](https://angular.dev/guide/templates/defer)
- [Event Replay API](https://angular.dev/api/platform-browser/withEventReplay)