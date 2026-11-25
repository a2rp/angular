# Angular Resources and Signal-Based Asynchronous Data: Q&A

## 1. What problem does a Resource solve?

A Resource represents asynchronous work and exposes its current value, status, loading state, and error as signals. This lets signal-driven templates respond to server reads without manually maintaining several request-state variables.

## 2. Are signals themselves asynchronous?

No. A signal read returns its current value synchronously. A Resource coordinates asynchronous work, then exposes the resulting state through synchronous signal reads.

## 3. When should I use resource()?

Use resource() for a read operation whose request depends on reactive params and whose loader naturally returns a Promise or async result.

## 4. What are params and loader?

params is a reactive function that describes the current request. loader is the asynchronous function that retrieves the value for those params.

## 5. How does a resource know when to reload?

Angular tracks signals read by the params function. When the resulting params value changes, Angular starts the loader for the new request.

## 6. Can the loader itself read signals?

It can, but request-defining dependencies should be read in params. That makes the resource's request inputs explicit and ensures changes trigger the correct load.

## 7. What happens if params returns undefined?

The loader does not run and the resource becomes idle. This is useful when a selection is missing or input is not yet valid.

## 8. Does returning { id: undefined } pause the resource?

No. That object is still a defined params value. Return undefined itself to pause the request.

## 9. What information does the loader receive?

The loader receives params, an AbortSignal, and previous status information. Use params for the requested data and pass abortSignal to APIs that support cancellation.

## 10. What does resource.value() return before data arrives?

By default it can be undefined until a value is available. A defaultValue can provide an initial value, but the resource status is still needed to tell a placeholder from a confirmed server result.

## 11. Why should I guard value() with hasValue()?

Reading value() when the resource is in an error state throws. hasValue() safely checks that a usable value exists and narrows the TypeScript type.

## 12. What statuses can a Resource report?

The current API statuses are idle, loading, reloading, resolved, local, and error. Use status() when the UI needs to distinguish these states.

## 13. What is the difference between loading and reloading?

Loading usually follows a change in params and has no current value for that request. Reloading follows reload() for the same params and keeps the previous value available while the refresh runs.

## 14. What does isLoading() include?

It is true while a new load or a reload is running. The exact status can distinguish loading from reloading when the UI needs to describe them differently.

## 15. How do I manually refresh a resource?

Call its reload() method. This runs the loader again for the current params.

## 16. Does changing params cancel the previous request?

Angular aborts the previous in-progress resource load and starts work for the new params. The loader needs to honor the provided AbortSignal for the underlying operation to stop promptly.

## 17. Does cancellation guarantee the server did no work?

No. The request may already have reached or been processed by the server. Client-side cancellation does not roll back server-side work.

## 18. Why are resources intended for reads rather than mutations?

A new params value or resource destruction can abort an in-progress load. Aborting a POST, PUT, or DELETE may leave the result ambiguous. Use an explicit action workflow for mutations.

## 19. What is httpResource?

httpResource is a reactive wrapper around HttpClient. It builds a request from reactive values and exposes the response and request state as resource signals.

## 20. Does httpResource use Angular interceptors?

Yes. It uses the Angular HTTP stack, including configured interceptors and HttpClient features.

## 21. How is httpResource different from HttpClient?

httpResource initiates a request eagerly when created and when its reactive request changes. HttpClient returns an Observable whose request begins when subscribed to.

## 22. Can httpResource skip a request?

Yes. Its URL or request factory can return undefined. This is useful for gating a request on input validity or a required selection.

## 23. Should I use httpResource for POST or PUT?

Usually no. Angular documents httpResource for reactive fetching and recommends the underlying HttpClient APIs for mutations such as POST or PUT.

## 24. Does a generic response type validate JSON at runtime?

No. A generic such as httpResource<Product[]>() helps TypeScript but does not inspect the server's actual JSON. Use the parse option, a parser, or a runtime schema when the shape must be verified.

## 25. What is rxResource?

rxResource creates a Resource from a stream factory that returns an RxJS Observable. It is useful when the data service already exposes Observables but the view should consume resource signals.

## 26. What is the difference between loader and stream in rxResource?

resource() with a promise-based loader returns an asynchronous result. rxResource uses a stream callback that returns an Observable for the current params.

## 27. What if an rxResource Observable completes without emitting?

The resource has no value to expose. The Observable must emit a value or error before it completes; returning EMPTY does not produce a valid resource result.

## 28. Should rxResource replace RxJS operators?

No. Use RxJS for stream composition, debounce, retries, event workflows, and other observable-specific behavior. rxResource adapts an Observable source to a Resource-style interface.

## 29. Which API should handle a WebSocket or server-sent event?

Use an approach that models a long-lived stream. Depending on the API and application needs, that may be a streaming resource, rxResource, or a service exposing an Observable. Plan connection cleanup, reconnection, event ordering, and buffering separately.

## 30. What does the resource streaming API return?

The streaming form returns a Signal<ResourceStreamItem<T>> or a Promise of one. Each signal value is either { value: T } or { error: Error }, and it can continue to change as new data arrives. This is distinct from a promise loader that resolves once for a request.

## 31. Can I cache a resource value through SSR?

A resource can use an id to transfer a resolved value through Angular's server rendering TransferState. The id must be unique in the application and identical on server and client.

## 32. Why is SSR transfer caching sensitive?

The transferred value is serialized into rendered HTML. Avoid sharing cached HTML that contains user-specific data, and review the application's authentication and cache policy.

## 33. Is an empty defaultValue the same as a successful empty response?

No. It may be present before a server response arrives. Check the resource status to distinguish a placeholder from a resolved response.

## 34. Should a resource display raw error.message?

Usually not. Error text can include implementation details. Map failures to a concise, safe message that follows the API's documented error contract.

## 35. Are resource APIs identical across every Angular release?

No. Their API status and availability vary by release. The current Angular documentation marks resource, httpResource, and rxResource stable since v22.0; check the version compatibility page for an older project.

## 36. What is the main takeaway?

Use a Resource when a read should react to changing signal-based params and expose its value and status as signals. Choose resource, httpResource, or rxResource based on the data source, guard value reads, and keep mutations in explicit HTTP workflows.

## Related references

- Lesson: lessons/095-angular-resources-and-signal-based-asynchronous-data.md
- Examples: examples/095-angular-resources-and-signal-based-asynchronous-data.md
- Angular async reactivity with resources: https://angular.dev/guide/signals/resource
- Angular resource API: https://angular.dev/api/core/resource`r`n- Angular streaming resource loader API: https://angular.dev/api/core/ResourceStreamingLoader
- Angular reactive data fetching with httpResource: https://angular.dev/guide/http/http-resource
- Angular httpResource API: https://angular.dev/api/common/http/httpResource
- Angular RxJS interop and rxResource: https://angular.dev/guide/signals/rxjs-interop
- Angular rxResource API: https://angular.dev/api/core/rxjs-interop/rxResource
- Angular version compatibility: https://angular.dev/reference/versions
