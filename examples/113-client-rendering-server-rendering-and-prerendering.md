# Client Rendering, Server Rendering, and Prerendering — Examples

These examples sketch a route-by-route rendering plan for a small public learning site with stable articles, a fresh public catalog, and a client-rendered workspace. Angular's setup details can vary by version; follow the generated files in the current workspace.

## 1. Add Angular's rendering support

For an existing Angular project, the CLI can add the SSR package and generated server configuration:

~~~sh
ng add @angular/ssr
~~~

Review the generated configuration and deployment output before selecting a production platform. The command may change project files and scripts based on the installed Angular version.

## 2. Assign rendering modes by route

A server route configuration can assign a mode to each route:

~~~ts
import { RenderMode, ServerRoute } from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  {
    path: '',
    renderMode: RenderMode.Client,
  },
  {
    path: 'lessons/**',
    renderMode: RenderMode.Prerender,
  },
  {
    path: 'catalog/**',
    renderMode: RenderMode.Server,
  },
  {
    path: 'workspace/**',
    renderMode: RenderMode.Client,
  },
  {
    path: '**',
    renderMode: RenderMode.Server,
  },
];
~~~

The order and syntax of route patterns should match the current Angular API and application routes. The final wildcard is a deliberate fallback here; choose the correct not-found behavior for the product rather than copying it blindly.

This example assumes lesson pages are public and stable enough to generate at build time, catalog pages need fresh request-time HTML, and the workspace is an interactive client-side area. Those assumptions should be validated for a real application.

## 3. Connect server routes in the server configuration

Generated projects commonly register server rendering features from the server application configuration:

~~~ts
import { ApplicationConfig } from '@angular/core';
import { provideServerRendering, withRoutes } from '@angular/ssr';
import { serverRoutes } from './app.routes.server';

export const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering(withRoutes(serverRoutes)),
  ],
};
~~~

Use the configuration structure created by ng add @angular/ssr for your project. Do not replace existing providers or generated settings without understanding their role.

## 4. Choose static output for a static deployment

If the application should be fully static and does not need a rendering server at request time, configure the build output mode in angular.json:

~~~json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "options": {
            "outputMode": "static"
          }
        }
      }
    }
  }
}
~~~

This is a partial snippet; merge the option into the existing project and build target. Static output can host prerendered routes without an Angular server. Request-time SSR routes cannot be served by a static file host alone, so make sure the selected modes and output mode agree.

## 5. Generate known parameterized pages

A prerendered route with a parameter needs a build-time list of parameter values. For example, a public lesson service could return the IDs to generate:

~~~ts
import { inject } from '@angular/core';
import { RenderMode, ServerRoute } from '@angular/ssr';
import { LessonCatalog } from './lesson-catalog';

export const serverRoutes: ServerRoute[] = [
  {
    path: 'lessons/:id',
    renderMode: RenderMode.Prerender,
    async getPrerenderParams() {
      const catalog = inject(LessonCatalog);
      const ids = await catalog.getPublicLessonIds();

      return ids.map((id) => ({ id }));
    },
  },
];
~~~

Call inject synchronously before the first await, as shown. Build-time route generation must not depend on a visitor's browser or private session. If lessons are too numerous or too frequently updated, consider whether request-time rendering or a different fallback is more suitable.

## 6. Compare the initial responses

Build the application using its configured production target:

~~~sh
ng build --configuration production
~~~

Then verify each route in the browser or deployed preview:

1. Request a prerendered lesson directly, not only by navigating from the home page.
2. Inspect the response HTML to confirm the expected public content is present.
3. Request a server-rendered catalog route and verify it reflects the expected fresh data.
4. Request the client-rendered workspace and confirm its shell loads as expected.
5. Check the browser console and Network panel for client startup errors.
6. Verify cache headers do not make private output publicly reusable.

On Windows, curl.exe can inspect an HTTP response, for example:

~~~sh
curl.exe -i https://example.test/lessons/signals
~~~

Replace the domain and route with the local preview or deployment URL. A successful HTML response does not by itself prove that hydration or client interactivity works; those are verified in later chapters.

## 7. Keep a route decision table

Before shipping the configuration, document why each route uses its mode:

| Route pattern | Mode | Data timing | Hosting requirement | Cache notes |
| --- | --- | --- | --- | --- |
| /lessons/** | Prerender | Build time | Static host or compatible server | Public content can be CDN cached |
| /catalog/** | Server | Request time | SSR runtime | Define freshness and cache policy |
| /workspace/** | Client | Browser session | Static assets plus app APIs | Keep private data in the client flow |

Adjust the entries to match the project. Avoid putting private data into prerendered HTML or public caches.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="112-rendering-performance-change-detection-and-profiling.md">112 - Rendering Performance, Change Detection, and Profiling</a></td><td><a href="../lessons/113-client-rendering-server-rendering-and-prerendering.md">Lesson</a></td><td><a href="../qnas/113-client-rendering-server-rendering-and-prerendering.md">Q&amp;A</a></td><td><a href="114-set-up-server-side-rendering-and-hybrid-route-rendering.md">114 - Set Up Server-Side Rendering and Hybrid Route Rendering</a></td>
</tr>
  </tbody>
</table>
