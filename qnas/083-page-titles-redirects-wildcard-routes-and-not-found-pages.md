# 83. Page titles, redirects, wildcard routes, and not-found pages - Q&A

Questions and answers for [Lesson 83](../lessons/083-page-titles-redirects-wildcard-routes-and-not-found-pages.md).

## Page titles

### 1. What does the title property on a route do?

It supplies a document title for that route. When the route activates, Angular updates the browser's document title, which appears in the browser tab and helps assistive technology identify the current page.

### 2. Does a route title replace the page's visible heading?

No. The document title and the visible heading serve different places. Use a meaningful route title for the browser and a visible heading in the page content when appropriate.

### 3. Can a route title be static?

Yes. Set title to a string on the route:

~~~ts
{ path: 'about', component: AboutPage, title: 'About' }
~~~

### 4. Can a route title depend on route parameters?

Yes. Use a ResolveFn that returns a string:

~~~ts
export const profileTitle: ResolveFn<string> = (route) => {
  const username = route.paramMap.get('username');
  return username ? 'Profile: ' + username : 'Profile';
};
~~~

The resolver receives route information and can return a value or supported asynchronous result.

### 5. When should I use a title resolver?

Use one when the title depends on information available during route activation, such as a path parameter or data that the resolver obtains. Keep it focused on producing a title; use a data resolver or another data-loading pattern for complex page data.

### 6. What does the default title strategy do?

Angular's default strategy uses the deepest title it finds on the primary route branch. A title set only on a named auxiliary outlet is not included by the default strategy.

### 7. How can I add an application name to every route title?

Provide a custom service that extends TitleStrategy. In updateTitle, call buildTitle with the router snapshot, apply the app's formatting, and set the result with Angular's Title service.

~~~ts
const routeTitle = this.buildTitle(snapshot);
this.title.setTitle(routeTitle
  ? routeTitle + ' | My App'
  : 'My App');
~~~

Register the service with the TitleStrategy token in application providers.

### 8. Should each component set document.title manually?

Usually, route title metadata is clearer because it keeps the title beside the URL configuration. Use a custom TitleStrategy for shared formatting. Manual updates may be appropriate for specialized views, but avoid scattering inconsistent title logic across components.

## Redirects and path matching

### 9. What is a redirect route?

It is a route that sends a matched URL to a different URL instead of displaying its own component. Configure it with redirectTo.

### 10. What is the difference between an absolute and a relative redirect target?

A target beginning with a slash is absolute from the application root. A target without a leading slash is resolved relative to the matched route.

Choose deliberately, especially for redirects inside nested route groups.

### 11. What is pathMatch?

It controls how much of the remaining URL must match a route. The common values are prefix and full. Prefix is the default for redirects; full requires the entire remaining URL path to match.

### 12. Why should an empty-path root redirect use full?

An empty path is a prefix of every URL. Without full, it can match after the redirect destination is selected and create a loop or intercept other pages.

~~~ts
{ path: '', pathMatch: 'full', redirectTo: '/home' }
~~~

### 13. When is prefix matching useful for a redirect?

Use it when an old path prefix and all of its descendants should move to a new prefix. For example, /news/story can be redirected under /articles/story.

### 14. When should a redirect use full matching?

Use full when only one exact remaining path should redirect, especially for the empty root path. It prevents descendant segments from being treated as part of that redirect.

### 15. Can a redirect preserve a parameter?

Yes. Map the source parameter into the destination:

~~~ts
{ path: 'legacy-user/:id', redirectTo: '/users/:id' }
~~~

A visit to /legacy-user/42 redirects to /users/42.

### 16. Does a redirect route render a component first?

No. A redirect changes the target navigation. The destination route is the route that activates and supplies the resulting view and title.

### 17. Can redirectTo be conditional?

Yes. It can be a RedirectFunction that receives partial route-matching information and returns a string or UrlTree, including a supported asynchronous result.

~~~ts
export const campaignRedirect: RedirectFunction = (snapshot) =>
  snapshot.queryParams['campaign'] === 'spring'
    ? '/offers/spring'
    : '/offers';
~~~

### 18. Does a RedirectFunction receive the fully activated route?

No. It runs during route matching, before the router has completed activation. It receives partial information, so do not rely on resolved data or a fully activated route tree.

### 19. Can inject be called from a RedirectFunction?

Yes, the function runs in an injection context and can call inject synchronously. Do not assume that injection context remains available later in an unrelated callback or after an asynchronous boundary.

## Wildcards and not-found pages

### 20. What does the wildcard path match?

The path written as two asterisks matches URLs that no earlier route handles at its level. A common use is a not-found page.

### 21. Where should the wildcard route go?

At the end of that route array, after the valid specific routes. Angular uses first-match-wins matching, so an early wildcard could capture a URL before its intended route is considered.

### 22. Should static routes come before parameter routes?

Usually, yes, when a static segment could be interpreted as a parameter. Put products/new before products/:id so the word new is treated as the static route instead of an identifier.

### 23. What is a root-level wildcard for?

It handles top-level URLs that do not match any application route. It can activate a global not-found page.

### 24. Can a feature have its own wildcard?

Yes. A wildcard in a feature's children can handle unknown paths within that feature and preserve its shell. A root wildcard can then handle unmatched paths outside that section.

### 25. Should every unknown URL redirect to the home page?

Not if users need to know that the requested page does not exist. Redirecting unknown URLs can hide stale links and typos. A not-found page communicates the problem and can offer useful next steps.

### 26. Does an Angular wildcard page return an HTTP 404 status?

No. It selects a client-side view. The HTTP response status is controlled by the server or SSR setup, which is separate from the Angular route configuration.

### 27. What should a not-found page include?

A clear explanation, a useful next action such as a home link, and a meaningful document title. Make the message understandable without depending only on color or an icon.

## Route matching and troubleshooting

### 28. Why is my route being swallowed by another route?

Check earlier route definitions. A parameter route or broad prefix route may match before the intended route. Reorder routes so more specific paths are evaluated before broad fallbacks.

### 29. Why does my root redirect loop?

Check the empty-path redirect. It should normally use full path matching so it only matches the exact root URL.

### 30. Why is the browser title not changing?

Check that the activated route has a title or title resolver, that the router is configured, and that any custom TitleStrategy is registered under the TitleStrategy token and calls the Title service.

### 31. Why does a title on a named outlet not appear?

The default strategy follows the deepest primary route title. A title on an auxiliary outlet is not automatically combined with it. Use a custom TitleStrategy if the app needs a combined title policy.

### 32. Should authentication use a wildcard redirect?

Usually no. Use a guard or explicit navigation decision for access control. A wildcard is intended as a fallback for URLs that do not match. Server-side authorization is still required for protected resources.

### 33. Should a not-found component have a route title?

Yes. A title such as Page not found makes the browser tab and assistive technology reflect the current state.

## Quick review

- **Route title:** document title associated with an activated route.
- **ResolveFn title:** route title computed from route information or supported asynchronous work.
- **TitleStrategy:** central control over how route titles are composed.
- **Redirect:** URL rule that navigates to a destination path.
- **full path matching:** the entire remaining path must match.
- **Wildcard:** final fallback for paths not matched earlier.
- **Not-found view:** client-side page; server HTTP status is configured separately.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="082-route-level-providers-and-route-configuration.md">082 - 82. Route-level providers and route configuration</a></td><td><a href="../lessons/083-page-titles-redirects-wildcard-routes-and-not-found-pages.md">Lesson</a></td><td><a href="../examples/083-page-titles-redirects-wildcard-routes-and-not-found-pages.md">Example</a></td><td><a href="084-preloading-scroll-restoration-and-router-testing.md">084 - 84. Preloading, scroll restoration, and router testing</a></td>
</tr>
  </tbody>
</table>
