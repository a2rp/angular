# Examples: Service Workers, Offline Behavior, Deployment, and Updates

These examples use Angular's built-in service worker for versioned app assets and selected public data. Verify generated configuration against the Angular CLI version used by the project. Do not put private API responses into cache rules without an explicit privacy and freshness review.

## 1. Add Angular service-worker support

For an Angular CLI project, the schematic creates the package, registration, build configuration, PWA manifest files, and an initial `ngsw-config.json`:

~~~bash
ng add @angular/pwa
~~~

The service worker is intended for production output. Build and serve that output over HTTPS or `localhost` when checking registration and cache behavior.

## 2. Register only outside development

The CLI schematic normally configures registration. If you manage root providers manually, the configuration can look like this:

~~~ts
import { ApplicationConfig, isDevMode } from '@angular/core';
import { provideServiceWorker } from '@angular/service-worker';

export const appConfig: ApplicationConfig = {
  providers: [
    provideServiceWorker('ngsw-worker.js', {
      enabled: !isDevMode(),
      registrationStrategy: 'registerWhenStable:30000',
    }),
  ],
};
~~~

Do not add a second registration if the schematic already registered the worker. Review the generated `app.config.ts` and keep one deliberate setup.

## 3. Cache a public, read-only catalog

This example is a fragment for the top-level `dataGroups` property in `ngsw-config.json`. It is suitable only if the endpoint is public, read-only, and safe to show for up to an hour while offline.

~~~json
{
  "dataGroups": [
    {
      "name": "public-catalog",
      "urls": ["/api/public/catalog/**"],
      "cacheConfig": {
        "strategy": "freshness",
        "maxSize": 50,
        "maxAge": "1h",
        "timeout": "3s"
      }
    }
  ]
}
~~~

`freshness` prefers the network and falls back to a cached response after the configured timeout. For data that rarely changes and can safely be stale, `performance` may be more appropriate. Put specific URL patterns before broader ones. Check the actual request paths and matching behavior with the installed Angular version.

Never add a broad rule such as `/api/**` without proving that it cannot match private, personalized, or state-changing data. Caching is not a substitute for API authorization.

## 4. Show an update prompt and reload after the user chooses

This service records when a complete new version is ready. The UI can expose a button such as “Update available” and call `reloadForUpdate()` only after the user confirms or saves current work.

~~~ts
import { Injectable, inject, signal } from '@angular/core';
import { SwUpdate } from '@angular/service-worker';
import { filter } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class UpdateNoticeService {
  private readonly updates = inject(SwUpdate);

  readonly updateReady = signal(false);
  readonly reloadRequired = signal(false);

  constructor() {
    if (!this.updates.isEnabled) {
      return;
    }

    this.updates.versionUpdates
      .pipe(filter((event) => event.type === 'VERSION_READY'))
      .subscribe(() => this.updateReady.set(true));

    this.updates.unrecoverable.subscribe(() => this.reloadRequired.set(true));
  }

  reloadForUpdate(): void {
    window.location.reload();
  }
}
~~~

Instantiate this root-provided service from a component or another startup service so its subscriptions begin. Show `updateReady()` in the interface. Do not call `activateUpdate()` and continue using the existing page unless the app has a reasoned compatibility design for mixing resource versions.

A component template might provide explicit actions:

~~~html
@if (updateNotice.updateReady()) {
  <aside role="status">
    <p>A new version is ready. Save your work before updating.</p>
    <button type="button" (click)="updateNotice.reloadForUpdate()">
      Reload to update
    </button>
  </aside>
}

@if (updateNotice.reloadRequired()) {
  <aside role="alert">
    <p>This page cannot load one of its files. Reload to continue.</p>
    <button type="button" (click)="updateNotice.reloadForUpdate()">
      Reload page
    </button>
  </aside>
}
~~~

The component must inject the service as `updateNotice`. Consider preserving or warning about unsaved work before reloading.

## 5. Handle manual update checks safely

Angular performs update checks during normal worker operation. If the product needs a scheduled check, wait until the app stabilizes, avoid frequent polling, and handle errors. Check that service-worker support is enabled first.

~~~ts
if (this.updates.isEnabled) {
  try {
    const available = await this.updates.checkForUpdate();
    if (available) {
      this.updateReady.set(true);
    }
  } catch (error) {
    // Report a non-sensitive diagnostic; keep the current app usable.
    this.logUpdateCheckFailure(error);
  }
}
~~~

This snippet belongs in a method with access to `updates`, `updateReady`, and `logUpdateCheckFailure`. Avoid starting a repeating timer before Angular's stability-based registration has had a chance to run.

## 6. Test the deployed offline experience

1. Build the production application.
2. Serve it over HTTPS or on `localhost` and open it once while online.
3. Confirm the browser registered `ngsw-worker.js` and fetched `ngsw.json`.
4. Reload, disable the network, and verify only the intended shell and cached public catalog are available.
5. Check that uncached APIs and mutations show an honest offline error.
6. Deploy another build and verify the existing tab gets an update prompt while a new tab can load the installed version.
7. Try a failed asset request and verify the unrecoverable state offers a recovery action.
8. Confirm private API data, cookies, and credentials are not in Cache Storage.

Use a clean browser profile or unregister the worker during local diagnosis if an old build is confusing results. Do not ask users to clear their browser data as the normal recovery procedure.

## 7. Publish a coherent release

Treat `index.html`, `ngsw.json`, `ngsw-worker.js`, and the hashed assets referenced by the manifest as one release. An atomic release is preferable. If a deployment platform cannot switch the whole build at once, publish new hashed files first, switch the manifest and entry document only when all referenced files are available, and keep the prior hashed resources for an appropriate overlap.

Verify that:

- the worker script is reachable at the intended registration scope;
- the deployed base path matches the built asset paths;
- SPA fallback does not return HTML for a missing JavaScript chunk;
- CDN or intermediary caches do not serve a manifest from one release with files from another;
- the previous active build can still fetch needed lazy chunks during rollout.

## Further reading

- [Angular service worker getting started](https://angular.dev/ecosystem/service-workers/getting-started)
- [Angular service worker configuration](https://angular.dev/ecosystem/service-workers/config)
- [Angular service worker communications](https://angular.dev/ecosystem/service-workers/communications)
- [Angular service worker deployment guidance](https://angular.dev/ecosystem/service-workers/devops)
- [Angular `SwUpdate` API](https://angular.dev/api/service-worker/SwUpdate)
