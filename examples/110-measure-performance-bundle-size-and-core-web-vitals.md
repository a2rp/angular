# Measure Performance, Bundle Size, and Core Web Vitals — Examples

The examples below cover a production build, a CLI size budget, image loading, and optional field metric collection. Adjust project names, endpoints, image paths, and thresholds for the application.

## 1. Build with the production configuration

~~~sh
ng build --configuration production
~~~

Record the build output for the same commit and inspect the initial total and any separate feature bundles. Compare with a known-good build when investigating growth. A larger feature bundle may not affect the initial load if it is only requested after navigation.

## 2. Add an initial bundle budget

In the build target's production configuration in angular.json, add a budget that matches the application's baseline:

~~~json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "configurations": {
            "production": {
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "300kb",
                  "maximumError": "450kb"
                }
              ]
            }
          }
        }
      }
    }
  }
}
~~~

This is a partial configuration example. Merge the budgets array into the existing production configuration; do not replace the rest of angular.json. Adjust the values from actual build output and product goals.

Build again:

~~~sh
ng build --configuration production
~~~

A warning is a prompt to review the change. An error threshold can fail the build. If a feature bundle needs its own limit, use a named bundle budget and match the bundle name reported by the builder.

## 3. Use Angular's image directive

A standalone component can import NgOptimizedImage and prioritize only the image that is actually the page's likely LCP element:

~~~ts
import { Component } from '@angular/core';
import { NgOptimizedImage } from '@angular/common';

@Component({
  selector: 'app-course-hero',
  standalone: true,
  imports: [NgOptimizedImage],
  template: `
    <main>
      <h1>Angular performance workshop</h1>
      <img
        ngSrc="assets/angular-workshop.webp"
        width="1600"
        height="900"
        sizes="(max-width: 700px) 100vw, 1200px"
        priority
        alt="Learner reviewing an Angular application on a laptop"
      />
    </main>
  `,
})
export class CourseHeroComponent {}
~~~

The image file and path must match the assets configured in the workspace. Supply its intrinsic dimensions and responsive sizes so the browser can choose an appropriate resource and reserve layout space. Mark only likely above-the-fold LCP candidates as priority. Non-critical images should generally remain lazy-loaded.

## 4. Measure a repeatable lab scenario

For a loading investigation:

1. Build the production configuration and serve it in a production-like environment.
2. Open browser developer tools and use the Performance or Lighthouse panel.
3. Capture a cold page load for the affected route.
4. Identify the LCP element and inspect the requests and main-thread work around it.
5. Save the browser, device, network, route, and cache conditions with the result.

For an interaction investigation:

1. Start a Performance recording.
2. Perform one representative interaction, such as opening a menu or filtering a list.
3. Stop recording and inspect the event, long tasks, rendering, and paint.
4. Use Angular DevTools when framework change detection may be a contributor.
5. Repeat the same action after the change.

Lab runs are useful for comparison, but field data is needed to understand the experience across real users.

## 5. Collect Core Web Vitals in the field (optional)

Install the web-vitals package with the package manager used by the project:

~~~sh
npm install web-vitals
~~~

A minimal browser-side collector can send metric values to a same-origin endpoint:

~~~ts
import { onCLS, onINP, onLCP } from 'web-vitals';

function reportMetric(metric: { name: string; value: number; id: string }): void {
  const payload = JSON.stringify({
    name: metric.name,
    value: metric.value,
    id: metric.id,
  });

  navigator.sendBeacon(
    '/analytics/web-vitals',
    new Blob([payload], { type: 'application/json' }),
  );
}

onLCP(reportMetric);
onINP(reportMetric);
onCLS(reportMetric);
~~~

The endpoint in this example is illustrative. Implement server-side validation, aggregation, retention limits, and consent handling appropriate to the product. Avoid collecting full URLs with sensitive query parameters or personal identifiers. Report field results at the 75th percentile and segment data only when the sample is large and the segmentation is useful.

## 6. Compare results without mixing conditions

Keep a small record for each before-and-after comparison:

~~~text
Commit:
Build configuration:
Route:
Browser and device profile:
Network and cache state:
Interaction or load scenario:
LCP / INP / CLS observation:
Bundle initial total:
Largest relevant chunk:
Change made:
Repeat measurement:
Field data window and sample:
~~~

Compare the same route and scenario. Do not compare a development-server run with a production build and call the difference an optimization.

## 7. Common interpretations

- **Initial budget warning:** inspect imports and dependencies in startup code, then consider whether a feature can be deferred.
- **Slow LCP:** identify the LCP element and its critical request chain; check server response and image priority.
- **Slow INP:** inspect the interaction handler and long tasks that block the next paint.
- **High CLS:** reserve space for media and asynchronous content, and investigate late style or font changes.
- **Large route chunk:** check whether it is expected and how often users need the route before trying to split it further.

## References

- [Angular CLI build budgets](https://angular.dev/tools/cli/build)
- [Angular performance best practices](https://angular.dev/best-practices/performance)
- [Angular image optimization](https://angular.dev/best-practices/performance/image-optimization)
- [Google Web Vitals](https://web.dev/articles/vitals)
- [web-vitals library](https://github.com/GoogleChrome/web-vitals)