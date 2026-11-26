# Lazy Loading and Deferrable Views with defer - Examples

These examples show two different boundaries: lazy route files and deferred sections inside a route. Update component names and paths to match your workspace.

## 1. Lazy-load a route component

Use loadComponent when a route needs one page component only after navigation:

~~~ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./home/home-page').then((module) => module.HomePage),
  },
  {
    path: 'reports',
    loadChildren: () => import('./reports/reports.routes'),
  },
];
~~~

The reports route file can export child routes:

~~~ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./reports-page').then((module) => module.ReportsPage),
  },
  {
    path: 'detail/:id',
    loadComponent: () =>
      import('./report-detail-page').then((module) => module.ReportDetailPage),
  },
];
~~~

The dynamic imports let the router load the route code when navigation needs it. Confirm the project paths and exported symbol names.

## 2. Create a standalone section to defer

Place an optional feature in its own standalone component. This example can represent comments below an article:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-comments',
  standalone: true,
  templateUrl: './comments.html',
})
export class CommentsComponent {}
~~~

The comments.html file:

~~~html
<section aria-labelledby="comments-heading">
  <h2 id="comments-heading">Comments</h2>
  <p>Comments are ready to read.</p>
</section>
~~~

Import the component directly from its component file in the parent. Avoid importing it through an index barrel when verifying a separate chunk.

## 3. Defer comments until a user requests them

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

Parent template in article-page.html:

~~~html
<article>
  <h1>Building a reliable Angular feature</h1>
  <p>The article content is available immediately.</p>
</article>

<button #showComments type="button">
  Show comments
</button>

<div aria-live="polite" aria-atomic="true">
  @defer (on interaction(showComments); prefetch on idle) {
    <app-comments />
  } @placeholder {
    <p>Comments have not been loaded.</p>
  } @loading (after 100ms; minimum 300ms) {
    <p>Loading comments…</p>
  } @error {
    <p>Comments could not be loaded. Try again later.</p>
  }
</div>
~~~

The prefetch trigger can download the component when the browser is idle. The main interaction trigger still controls when Angular displays it. The live region is appropriate here because the user explicitly requested the section; adapt announcements to the experience and avoid noisy updates.

## 4. Defer content when it approaches the viewport

For a section below the fold, a viewport trigger can load it as the user approaches. A placeholder provides an element to observe and can reserve space:

~~~html
@defer (on viewport) {
  <app-related-courses />
} @placeholder {
  <section aria-label="Related courses" class="related-courses-placeholder">
    <p>Related courses will load here.</p>
  </section>
} @loading {
  <p aria-live="polite">Loading related courses…</p>
} @error {
  <p>Related courses could not be loaded.</p>
}
~~~

Give the placeholder a stable minimum height that resembles the final section so the transition does not move surrounding content unexpectedly.

## 5. Reserve space in CSS

Example style for a predictable placeholder area:

~~~css
.related-courses-placeholder {
  min-block-size: 18rem;
  display: grid;
  place-items: center;
  border: 1px dashed currentColor;
}
~~~

The final component may need a different size. Check the page at the viewport sizes used by users and measure CLS instead of assuming that one placeholder size is sufficient.

## 6. Verify the production chunk split

Build using the production configuration:

~~~sh
ng build --configuration production
~~~

Inspect the emitted bundle summary. The comments or related-courses code should be outside the initial bundle if the dependencies meet the defer requirements and are not referenced eagerly elsewhere. Open the built app in a browser, navigate to the page, then inspect the Network panel:

1. Load the article and confirm the deferred chunk has not been requested prematurely, allowing for configured prefetching.
2. Click **Show comments**.
3. Confirm the chunk request occurs and the comments appear.
4. Simulate or inspect a failed chunk request to verify the error state is understandable.

During development, HMR can fetch defer chunks eagerly. Use a production build or appropriate preview to confirm actual trigger behavior.

## 7. Keep eager references out of the parent file

A deferred dependency can become eager if the same dependency is used outside a defer block in that file. For example, displaying the component before the block defeats the intended split:

~~~html
<!-- This eager use can prevent the intended deferral. -->
<app-comments />

@defer {
  <app-comments />
}
~~~

Keep the optional component exclusively inside the defer block in the parent template. Also verify it is standalone and imported directly from its source file.

## 8. Choose a boundary from actual use

- Use a lazy route for a page or feature tree users reach through navigation.
- Use a defer block for an optional area inside a page that is already loaded.
- Keep critical visible content eager unless there is a strong reason to delay it.
- Avoid multiple nested deferred blocks that require several dependent requests.
- Measure startup and the first-use experience after adding a split.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="110-measure-performance-bundle-size-and-core-web-vitals.md">110 - Measure Performance, Bundle Size, and Core Web Vitals</a></td><td><a href="../lessons/111-lazy-loading-and-deferrable-views-with-defer.md">Lesson</a></td><td><a href="../qnas/111-lazy-loading-and-deferrable-views-with-defer.md">Q&amp;A</a></td><td><a href="112-rendering-performance-change-detection-and-profiling.md">112 - Rendering Performance, Change Detection, and Profiling</a></td>
</tr>
  </tbody>
</table>
