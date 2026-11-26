# Service Workers, Offline Behavior, Deployment, and Updates

A service worker is a browser-managed worker that can intercept requests and serve cached responses. Angular's service worker adds versioned application caching and basic offline support to a CLI application. It can make a deployed app more reliable on slow or interrupted networks, but it does not automatically make every feature work offline or make stale server data safe.

Angular describes its service worker as a basic caching utility with a limited feature set and recommends browser APIs directly for more advanced caching or offline behavior. Choose it for the behavior it provides, and document the product's actual offline guarantees.

## Learning outcomes

- Add Angular's service worker and understand the generated files.
- Distinguish versioned app assets from runtime data caches.
- Choose cache policies based on freshness, privacy, and offline needs.
- Explain how Angular installs and activates application updates.
- Handle update prompts and unrecoverable application states.
- Deploy the worker, manifest, and assets as a consistent build.
- Test offline behavior without assuming every workflow is available.

## 1. What a service worker does

A service worker runs separately from the page and can intercept requests in its registration scope. The browser can keep it available after a tab closes, so it can respond with cached resources when a network request is unavailable. Service workers require a secure context: production sites should use HTTPS, while browsers treat `localhost` as secure for development.

Angular's worker is generated and managed by the Angular CLI. Its `ngsw.json` manifest records the files and content hashes for a build. Do not confuse this generated service-worker manifest with the PWA web app manifest, typically `manifest.webmanifest`, which describes installable app metadata and icons.

Angular's service worker is not enabled in the normal development configuration by default. Add it to a CLI project with:

~~~bash
ng add @angular/pwa
~~~

The schematic installs the worker package, configures the build and registration, adds PWA manifest and icons, and creates `ngsw-config.json`. Review the generated configuration rather than treating it as a complete offline product design.

## 2. App assets and runtime data use different policies

**Asset groups** describe files that belong to a particular application version, such as the app shell, scripts, styles, and local assets. The worker hashes the build resources and updates the application as a consistent version. A running tab continues using its current version; it should not suddenly combine old application code with new lazy-loaded chunks.

**Data groups** describe runtime URL responses that are cached according to explicit policies. These resources are not versioned together with the application. Angular supports `performance`, which can serve cached data quickly and accept some staleness, and `freshness`, which prefers the network and can fall back to cache after a configured timeout.

Use cache rules only where the product can explain what a user may see while offline and how old that content can be. Never cache account details, private API responses, or other sensitive data by default. Do not cache a mutation as though it succeeded; offline writes require a separate, explicit queue, conflict, replay, and authorization design.

A `dataGroups` rule might cache a public catalog whose content is safe to show offline:

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

This fragment belongs inside `ngsw-config.json`'s top-level object. Verify URL patterns, limits, and behavior against the installed Angular CLI version. More specific data groups should appear before broader matching groups.

## 3. Decide what “offline” means for the feature

The worker can serve previously cached app files and only the runtime data explicitly covered by a rule. It cannot make a server API available when the network is gone. For each user journey, decide whether the app should:

- show a clear offline state and disable server actions;
- display a bounded-age copy of public, read-only data; or
- support queued writes with a carefully designed synchronization and conflict flow.

A successful offline page load does not prove login, search, checkout, form submission, uploads, or account changes work offline. Service-worker cache contents may be evicted by the browser, and available storage varies by device. Treat offline support as a resilience feature, not durable data storage.

## 4. Application versions and updates

Angular's worker checks the generated manifest for a new build, downloads changed versioned resources, and makes the complete new version available. It serves a consistent version to an existing client until the client reloads; this avoids mixing incompatible bundles. A later navigation or refresh can load the latest fully installed version.

`SwUpdate` exposes `versionUpdates`, `checkForUpdate()`, `activateUpdate()`, and `unrecoverable`. `VERSION_READY` means a new version has been downloaded and is ready. A friendly update prompt can let a user save work and reload. In most cases, reload the page to switch versions. Angular warns that calling `activateUpdate()` without a reload can mismatch the app shell with lazy chunks from another build.

If an old required asset disappears from both the cache and server, the active application version can become unrecoverable. Subscribe to `SwUpdate.unrecoverable` and offer a clear recovery action such as a full reload. Avoid an automatic reload loop that could repeatedly discard user work.

## 5. Deployment must publish one coherent build

Build the application with the production configuration so the CLI emits the worker script and manifest. Deploy the generated output at the intended base path, including `ngsw-worker.js`, `ngsw.json`, `index.html`, and every hashed resource referenced by that manifest.

Publish the output as a coherent release. A non-atomic deploy or stale CDN response can expose a new manifest with old files, or an old running version whose lazy chunk is no longer available. Use an atomic release strategy where possible, configure intermediary caches to serve matching files, and retain old content-hashed assets for an appropriate overlap when the hosting platform requires it. Verify that the worker script is reachable within the intended scope and that SPA navigation fallback does not return an incorrect response for worker or asset URLs.

Test the deployed path, not only the development server. A subpath deployment changes the worker scope and asset paths; the CLI base href and server routing must agree.

## 6. Updates and registration need deliberate user experience

Angular delays service-worker registration until the app stabilizes by default, to avoid delaying startup. Starting an endless polling timer before stabilization can delay registration until the configured upper bound. If you schedule manual checks, start after stability and handle rejected promises. Check `SwUpdate.isEnabled` before calling update APIs because unsupported or disabled service workers do not emit worker events and can reject checks.

Do not force a reload while a user is midway through an important workflow without warning. Show an update prompt at an appropriate point and let the user save or finish work first. Some products can refresh immediately, but that is a product decision with data-loss implications.

## 7. Safe testing and support

Test a production build served from HTTPS or `localhost`, since ordinary service-worker behavior is not available on an arbitrary insecure HTTP host. Use a clean browser profile or remove old registrations during diagnosis so previous cached builds do not confuse results.

Check:

- first install and the first controlled page load;
- a cached page reload with the network disabled;
- a public data URL in each configured cache strategy;
- an uncached API while offline;
- a deployed update with an existing tab and a new tab;
- a failed update download and an unrecoverable missing chunk;
- the exact production base path, CDN, and routing configuration;
- privacy: no sensitive or user-specific responses are cached unexpectedly.

An offline fallback page can explain that the network is unavailable, but it cannot guarantee access to uncached features. Provide a visible retry path and avoid presenting stale information as current.

## Key takeaways

- Angular's service worker provides versioned asset caching and basic offline support.
- Asset groups version with the app; data groups need explicit freshness and privacy choices.
- Cached application files do not make backend features available offline.
- Prompt and reload to activate updates in most cases; avoid mixing versions.
- Publish manifest and matching build resources coherently.
- Test deployed offline and update behavior, including failures and user data safety.
- For advanced offline workflows, consider whether the limited Angular worker is sufficient before building a custom strategy.

## Further reading

- [Angular service worker overview](https://angular.dev/ecosystem/service-workers)
- [Getting started with Angular service workers](https://angular.dev/ecosystem/service-workers/getting-started)
- [Angular service worker configuration](https://angular.dev/ecosystem/service-workers/config)
- [Angular service worker communications and updates](https://angular.dev/ecosystem/service-workers/communications)
- [Angular service worker deployment guidance](https://angular.dev/ecosystem/service-workers/devops)
- [MDN: Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [MDN: Secure contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Secure_Contexts)
