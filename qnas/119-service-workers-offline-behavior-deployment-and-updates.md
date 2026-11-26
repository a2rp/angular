# Q&A: Service Workers, Offline Behavior, Deployment, and Updates

## 1. What is a service worker?

It is a browser-managed worker that runs separately from a page and can intercept requests within its scope. It can return cached resources when the network is unavailable and can help an application load more reliably.

## 2. Does a service worker run on any HTTP website?

No. Production service workers require a secure context, normally HTTPS. Browsers treat `localhost` as secure for local development.

## 3. How do I add Angular's service worker to a CLI project?

The Angular CLI schematic is `ng add @angular/pwa`. It installs and configures the service worker, generates `ngsw-config.json`, and adds PWA-related manifest and icon setup. Review the generated files and verify behavior with a production build.

## 4. What is the difference between `ngsw.json` and `manifest.webmanifest`?

`ngsw.json` is Angular's generated service-worker manifest with app resource metadata and hashes. `manifest.webmanifest` describes installable app metadata, such as the app name, icons, and display behavior. They serve different purposes.

## 5. What is an asset group?

An asset group describes build resources that belong to an application version, such as scripts, styles, the app shell, and local images. Angular hashes these resources and updates the application as a consistent set.

## 6. What is a data group?

A data group describes runtime URL responses that Angular's worker may cache under a manually selected policy. Data groups are not versioned together with the app; choose freshness, size, age, and matching patterns deliberately.

## 7. What is the difference between `performance` and `freshness` strategies?

`performance` prefers cached data for speed and may return stale content until its configured age expires. `freshness` prefers a network response and can fall back to cache after a timeout. Select based on how quickly the resource changes and what offline users may safely see.

## 8. Should I cache every API response for offline support?

No. Do not cache private account data or user-specific responses by default. Cache only data whose privacy, freshness, and offline meaning have been reviewed. A public catalog may be suitable; balances, medical records, or user-specific data need a different design.

## 9. Does caching a GET endpoint make offline writes work?

No. A cached read does not make a network mutation available offline. Queued writes require explicit persistence, replay, conflict handling, authorization on the server, and user feedback. Do not imply a failed request succeeded just because the UI queued it.

## 10. Does a successful offline page load prove the app works offline?

No. It only proves the requested app shell and resources were available from cache. Login, uncached APIs, search, uploads, checkout, and state-changing operations may still require the network.

## 11. Can the browser permanently keep everything in the service-worker cache?

No. Browser storage is subject to quotas and eviction. Treat cached resources as a performance and resilience aid, not as durable user data storage.

## 12. How does Angular detect an application update?

Angular's worker checks the generated `ngsw.json` manifest. When its content changes, the worker downloads the changed resources and prepares a new complete application version.

## 13. Why can a running tab keep showing the previous version?

Angular keeps an existing client on a consistent set of resources. A new version can be installed in the background, while the current tab continues with its current version until it reloads. This avoids mixing incompatible app code and lazy chunks.

## 14. What does `VERSION_READY` mean?

It means a new application version was downloaded and is ready to activate for clients. The app can notify the user and reload at an appropriate time, preferably after important unsaved work is preserved.

## 15. Should I call `activateUpdate()` as soon as an update is available?

Usually no. Angular warns that updating a tab without reloading can mix the new application shell with lazy-loaded chunks from another version. In most apps, prompt the user and reload the page to activate the installed version.

## 16. What is an unrecoverable service-worker state?

It is a state where the worker cannot serve a required resource for the app version and cannot recover without a full reload. `SwUpdate.unrecoverable` can notify the app so it presents a clear recovery action.

## 17. Why do releases need to deploy files coherently?

The service-worker manifest contains hashes for specific build resources. If a cache, CDN, or non-atomic deployment exposes a manifest and assets from different builds, integrity checks or lazy loading can fail. Publish a coherent build and configure caches to serve matching resources.

## 18. Is the Angular service worker enabled by ordinary `ng serve`?

Service-worker behavior is generally intended for a production build. Follow the current Angular getting-started guide to serve the production configuration locally. Use a clean browser session during diagnosis to avoid previous worker registrations and caches.

## 19. What happens if the browser does not support service workers?

The application should still work as a normal web app. Offline caching and worker update events are unavailable, and update checks may reject. Check `SwUpdate.isEnabled` before calling APIs that require a registered worker.

## 20. Can I use Angular's worker for advanced background synchronization or custom caching?

Angular describes its worker as a basic caching utility with a limited feature set and recommends native browser APIs for more advanced needs. Decide whether the built-in behavior is sufficient before adding a custom worker or offline-write architecture.

## 21. Why can update polling delay worker registration?

By default Angular waits for the app to stabilize before registering the worker, with a configured upper bound. A repeating timer started before stability can prevent the app from stabilizing. If polling is needed, start it after stability or choose a registration strategy intentionally.

## 22. How should I test an offline fallback?

Use a production build served on HTTPS or `localhost`, visit once to populate the cache, and then disable the network. Test cached and uncached routes separately, verify that unsupported features show an honest offline state, and test a new version with an existing tab.

## 23. Does the worker cache obey HTTP cache headers for every response?

No. Angular handles versioned resources and configured data groups according to its own policies. Unhashed resources have different behavior. Review Angular's configuration and deployment documentation instead of assuming ordinary HTTP cache semantics apply to every worker-managed resource.

## 24. What should a useful offline message say?

It should state that the network is unavailable, distinguish cached content from current server data, explain which actions need connectivity, and offer a retry. It should not present stale cached values as live or claim a server-side change succeeded when it did not.

## Further reading

- [Angular service worker overview](https://angular.dev/ecosystem/service-workers)
- [Angular service worker configuration](https://angular.dev/ecosystem/service-workers/config)
- [Angular service worker communications](https://angular.dev/ecosystem/service-workers/communications)
- [Angular service worker deployment guidance](https://angular.dev/ecosystem/service-workers/devops)
- [MDN: Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
