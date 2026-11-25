# Server-Compatible Code, Metadata, and Search Visibility — Examples

These snippets show route titles, route-specific metadata, guarded browser-only code, and a direct-response review. Adapt paths and public content for the application.

## 1. Define page titles in the route table

Angular Router can set the document title when a route activates:

~~~ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: 'courses/angular',
    title: 'Angular Course',
    loadComponent: () =>
      import('./course-page').then((module) => module.CoursePageComponent),
  },
  {
    path: 'articles/:slug',
    title: 'Article',
    loadComponent: () =>
      import('./article-page').then((module) => module.ArticlePageComponent),
  },
];
~~~

Use a ResolveFn when a title should use resolved route data, or a custom TitleStrategy when all titles need a common suffix. Keep titles unique and useful for the route.

## 2. Set route-specific description and social tags

An article component can update meta tags from public article data:

~~~ts
import { Component, inject } from '@angular/core';
import { Meta } from '@angular/platform-browser';

type Article = {
  title: string;
  summary: string;
  canonicalUrl: string;
  imageUrl: string;
};

@Component({
  selector: 'app-article-page',
  standalone: true,
  templateUrl: './article-page.html',
})
export class ArticlePageComponent {
  private readonly meta = inject(Meta);

  setArticleMetadata(article: Article): void {
    this.meta.updateTag({
      name: 'description',
      content: article.summary,
    });
    this.meta.updateTag({
      property: 'og:title',
      content: article.title,
    });
    this.meta.updateTag({
      property: 'og:description',
      content: article.summary,
    });
    this.meta.updateTag({
      property: 'og:image',
      content: article.imageUrl,
    });
  }
}
~~~

Call setArticleMetadata after the public article data has been resolved for the route, before the SSR response is produced. Make sure values are escaped and use a trusted, absolute public image URL. Meta tags alone do not guarantee how search or social platforms display the page.

## 3. Set a canonical link with a trusted URL

If the application needs a canonical link, use a trusted configured origin and a normalized route path. For example, the server or a route resolver can provide a validated canonical URL to a metadata service. Avoid composing an absolute URL from unvalidated Host or forwarded headers.

Keep canonical metadata consistent with redirects and the URL that actually serves the page.

## 4. Guard browser-only storage access

A service can avoid referencing localStorage during server rendering:

~~~ts
import { inject, Injectable, PLATFORM_ID } from '@angular/core';
import { isPlatformBrowser } from '@angular/common';

@Injectable({ providedIn: 'root' })
export class BrowserPreferences {
  private readonly platformId = inject(PLATFORM_ID);

  readTheme(): string | null {
    if (!isPlatformBrowser(this.platformId)) {
      return null;
    }

    return window.localStorage.getItem('theme');
  }
}
~~~

This guard belongs in imperative service code, not in a template that renders different server and client trees. If a browser preference changes visible state, apply it after the initial compatible render and handle the resulting UI transition deliberately.

## 5. Initialize a DOM-dependent library after rendering

When a library needs a real element, isolate initialization to browser rendering:

~~~ts
import {
  afterNextRender,
  Component,
  ElementRef,
  viewChild,
} from '@angular/core';

@Component({
  selector: 'app-chart',
  standalone: true,
  templateUrl: './chart.html',
})
export class ChartComponent {
  readonly chartHost =
    viewChild.required<ElementRef<HTMLElement>>('chartHost');

  constructor() {
    afterNextRender({
      write: () => {
        initializeChartLibrary(this.chartHost().nativeElement);
      },
    });
  }
}
~~~

Use a matching template:

~~~html
<div #chartHost aria-label="Course progress chart"></div>
~~~

The callback runs on browser platforms, not the server. It is not a guarantee that the component has completed hydration. Ensure the library does not unexpectedly restructure Angular-owned DOM, and dispose of the chart when the component is destroyed.

## 6. Add safe defaults to the document shell

Set a useful default title and description in the document template for routes that do not provide a more specific value:

~~~html
<title>Learning Notes</title>
<meta
  name="description"
  content="Practical lessons and reference notes for web development."
/>
~~~

Route metadata can update these defaults. Public pages should provide accurate titles and descriptions, while personalized pages should not expose private account information in shared HTML.

## 7. Verify the response HTML

Request a page directly from the SSR or static preview:

~~~sh
curl.exe -sS https://example.test/articles/angular-basics
~~~

Inspect the returned document for:

- The correct page title.
- The description and Open Graph values.
- The expected public page content.
- A canonical URL, if the application uses one.
- Any personal data that should not appear in shared output.

A browser's live DOM after Angular runs can differ from the original document response. Inspect the response itself when verifying SSR or prerendered metadata.

## 8. Review a server compatibility checklist

Before shipping a route:

~~~text
Route:
Rendering mode:
Browser-only dependencies:
Server-safe data source:
Page title:
Meta description:
Social preview values:
Canonical URL source:
Expected initial HTML:
Hydration behavior:
Private values excluded:
Direct-response verification:
~~~

## References

- [Angular SSR best practices](https://angular.dev/best-practices/performance/ssr)
- [Angular route titles](https://angular.dev/guide/routing/define-routes)
- [Angular Meta service](https://angular.dev/api/platform-browser/Meta)
- [Angular Title service](https://angular.dev/api/platform-browser/Title)
- [afterNextRender API](https://angular.dev/api/core/afterNextRender)
- [Angular hydration](https://angular.dev/guide/hydration)