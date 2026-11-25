# Loading, Empty, Success, and Error UI States: Q&A

## 1. Why model asynchronous UI states explicitly?

A request can be idle, loading, successful with data, successful with no data, or failed. Explicit states prevent the template from guessing what a missing value means.

## 2. What is an idle state?

It means the feature has not started a request yet. It can show an initial prompt or remain visually quiet.

## 3. What is the difference between empty and error?

Empty means the request succeeded and returned no items. Error means the request failed and the application did not receive a successful result. They need different user feedback.

## 4. Why is a loading boolean often insufficient?

A loading flag does not say whether the last request succeeded, returned no items, or failed. Separate booleans can also create contradictory combinations such as loading and error both being true.

## 5. What is a discriminated union?

It is a TypeScript union whose variants share a field such as status or kind with a distinct literal value. TypeScript can narrow the available fields based on that value.

## 6. Why use a generic LoadState<T>?

It lets success carry the expected data type while the other variants carry only the fields they need, such as an error message or retry flag.

## 7. Should every request use an empty state?

No. An empty state is useful for collections with no items. A detail request that returns no resource may instead be a not-found state, depending on the API contract and product flow.

## 8. Where can I map an HTTP result into UI state?

A service or a feature-level stream can map successful values and failures into a state union. Keep the mapping near the boundary where technical results become user-visible states.

## 9. What does catchError do?

It handles an Observable error by returning another Observable or rethrowing an error. A returned fallback becomes part of the downstream stream.

## 10. Why catch a request error inside switchMap?

If requests are triggered by a long-lived stream such as a retry button, an inner catchError converts one request's failure into a state while allowing future triggers to continue.

## 11. What can happen if catchError wraps the entire trigger stream?

If it replaces the failed outer stream with a one-value fallback, that replacement may complete. Later trigger values then cannot start new requests.

## 12. Should every error show a retry button?

No. Retry should fit the failure and operation. Validation errors, authorization failures, and malformed requests may need a different action than a temporary network failure.

## 13. How should a network error be presented?

Use a short message that suggests an action, such as checking the connection and trying again. Do not display internal exception details.

## 14. Should raw server error bodies be shown?

Usually not. A server payload may contain internal details or untrusted text. Map documented error codes to safe, useful messages.

## 15. What should happen to old data while refreshing?

Choose intentionally. The application may clear it and show loading, or keep it visible with a refreshing state. If the refresh fails while old data remains, make clear that the data may be stale.

## 16. How do I represent stale data?

Add a state such as refreshing that carries the previous data, or model data and request status separately with an explicit stale indicator. Do not label old data as the latest successful response without qualification.

## 17. Should a spinner replace the whole page?

It depends on the experience. For an initial load, a page-level indicator may fit. During a background refresh, preserving useful content and showing a smaller status may reduce disruption.

## 18. What does aria-busy communicate?

It indicates that a region is being updated. Bind it to the loading state when the region's content is in progress.

## 19. When is role=status useful?

It provides a polite announcement for non-urgent status updates such as loading progress. It should not repeatedly interrupt the user.

## 20. When is role=alert useful?

It announces important new content that may require immediate attention, such as a request failure. Do not use it for routine changes that would create excessive interruptions.

## 21. Should an error message automatically move keyboard focus?

Not for every background failure. Move focus only when the interaction requires it, such as a form submission that needs correction. Otherwise, use an appropriate live region and preserve the user's current focus.

## 22. How does the async pipe help?

It subscribes to an Observable used by the template and cleans up the view subscription when the view is destroyed or the bound Observable reference changes.

## 23. When should I use @switch?

Use it when one state field selects among distinct UI branches. It keeps each state rendering path together and avoids nested, potentially overlapping booleans.

## 24. Can @for @empty replace an explicit empty state?

It can render an empty list fallback when iterating over a collection. An explicit empty state is often more useful when it needs a heading, explanation, or action. In either case, do not use it to represent request failure.

## 25. Is retrying a GET usually safe?

GET is intended to retrieve data, so retrying it is commonly safe. The endpoint must still follow safe HTTP semantics, and retries should remain limited and appropriate.

## 26. Should I automatically retry a POST?

Not blindly. A timeout does not prove the server did not process the mutation. Follow the API's idempotency and retry contract.

## 27. Why avoid infinite automatic retries?

They can leave the interface stuck in loading, waste network and server resources, and make it unclear whether the operation is progressing. Use a finite retry policy and clear user feedback.

## 28. How should a 204 No Content response be rendered?

Treat it according to the endpoint's contract. For a collection endpoint, it may represent an empty result; for a command endpoint, it may mean successful completion. Do not infer meaning from status alone when the API contract says otherwise.

## 29. How should I represent a 404?

It depends on the resource and product flow. A missing detail record may deserve a not-found page; a filtered list returning no matches should normally be a successful empty state.

## 30. What if several requests load one page?

Model the page's required outcomes deliberately. You can combine streams into one page state or model independently loaded sections if one section's failure should not block the others.

## 31. Should the component or service own the state?

A service can own shared data loading and caching policy. A component or feature layer often owns view-specific states and messages. Keep state near the scope where it is consumed and avoid duplicating conflicting sources of truth.

## 32. Should a successful empty response be converted into an error?

No, unless the API contract defines it as invalid. Empty success and failure are different outcomes and should remain distinguishable.

## 33. What is the main accessibility rule for state changes?

Make loading and errors perceivable without forcing unexpected focus changes. Use appropriate status semantics, concise messages, and visible actions.

## 34. What is the main rule for asynchronous UI state?

Represent each meaningful outcome explicitly. Render empty, success, and error separately, and make refresh, stale-data, and retry behavior intentional.

## Related references

- Lesson: lessons/094-loading-empty-success-and-error-ui-states.md
- Examples: examples/094-loading-empty-success-and-error-ui-states.md
- Angular template control flow: https://angular.dev/guide/templates/control-flow
- Angular making HTTP requests: https://angular.dev/guide/http/making-requests
- RxJS catchError: https://rxjs.dev/api/operators/catchError
