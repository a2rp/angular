# RouterOutlet, RouterLink, and active route links: Q&A

## 1. What does `RouterOutlet` do?

`RouterOutlet` marks the location in a template where Angular should render the component for the currently activated route. The outlet does not select a route; the router uses the URL and route configuration to choose the component.

## 2. Does the outlet element disappear after Angular renders a route?

No. The outlet remains in the DOM as the reference point for later navigations. Angular inserts the routed component immediately after it as a sibling. Shared elements before and after the outlet can remain visible while the route view changes.

## 3. Where should `RouterOutlet` be placed?

Place it in the template of the component that owns the routed view. A root application commonly puts it inside its main content area, between shared layout elements such as a header and footer. A parent route can have another outlet for its child views.

## 4. What must a standalone component do before using `RouterOutlet`?

Import `RouterOutlet` from `@angular/router` in that component's `imports` array:

~~~ts
@Component({
  imports: [RouterOutlet],
  template: '<router-outlet />',
})
export class AppComponent {}
~~~

The route configuration and router provider must also be set up elsewhere in the application.

## 5. What does `RouterLink` provide?

`RouterLink` connects a template element, usually an anchor, to a route. It provides declarative navigation through the Angular Router, updates the application URL, and works with browser history.

## 6. Why should a navigation destination use an anchor instead of a button?

A destination is a link, so use `<a routerLink="/about">About</a>`. Anchors provide expected link semantics and browser behavior, including keyboard access and link actions. A button is appropriate for an action such as submitting a form, saving data, or opening a dialog.

## 7. Does a basic router link cause a full document reload?

For a normal internal route handled by the configured Angular Router, navigation occurs within the running application. Angular updates the URL and activates the matching route instead of requesting a new HTML document for every page. Browser Back and Forward continue to work.

## 8. What is the difference between a root-relative and a relative router link?

A path beginning with `/` starts at the application root, for example `routerLink="/guides"`. A path without a leading slash is resolved relative to the current activated route. Relative links are useful in nested route areas, but their target depends on the route context.

## 9. What does `RouterLinkActive` do?

It tracks whether a related router link matches the current route. It can add CSS classes while active and remove them when inactive. It can also set `aria-current` for the active link.

## 10. Does `RouterLinkActive` have to be on the anchor?

No. It can be placed on a parent element to apply a class when one of its contained links is active. For a navigation menu, placing it on each anchor makes the active state and accessibility state specific to that link. An ancestor directive cannot inspect links that are inside another component's template.

## 11. What is the default active-link matching behavior?

By default, `RouterLinkActive` uses subset matching. A link may remain active when the current URL has the link's path plus additional segments. This is useful when a section link should stay highlighted on its child pages.

## 12. When should a link use exact matching?

Use exact matching when the active class should be applied only when the current URL exactly matches the link. The Home link to `/` is a common example; without exact matching, it can appear active on routes beneath the root.

~~~html
<a
  routerLink="/"
  routerLinkActive="active"
  [routerLinkActiveOptions]="{exact: true}"
>
  Home
</a>
~~~

## 13. How does `ariaCurrentWhenActive="page"` help?

It asks Angular to add `aria-current="page"` while that router link is active. Assistive technology can then identify the link for the current page. Angular removes the attribute when the link stops being active.

## 14. Is a color change enough to show an active link?

No. Some visitors cannot distinguish color differences. Pair color with another visual cue, such as an underline or font weight, and set an appropriate `aria-current` value for assistive technology.

## 15. What is the difference between `RouterLink` and a click handler that navigates?

`RouterLink` expresses navigation in the template and preserves link semantics and browser behavior. A click handler is application code for responding to an event; using one to imitate a normal link requires recreating expected navigation and accessibility behavior. Use the router directive for ordinary route navigation.

## 16. Why might the URL change but no page component appear?

The root template may be missing `<router-outlet />`, or the component may have forgotten to import `RouterOutlet`. Also confirm that the router is configured and that a route matches the URL (or that a fallback route is configured).

## 17. Why might an active class not appear?

Check that:

- The component imports `RouterLink` and `RouterLinkActive`.
- The link points to a route configured in the router.
- The current URL matches the link according to its matching options.
- The expected CSS class is spelled the same way in the template and stylesheet.

## 18. What is a common cause of a Home link appearing active on every page?

The root path is a prefix of application paths under `/`. Use `[routerLinkActiveOptions]="{exact: true}"` on the Home link when it should be active only at the root URL.

## 19. Should a regular navigation link use `href` or `routerLink`?

Use `routerLink` when the destination is an internal Angular route handled by the app. Use a normal `href` for external destinations or document navigation that should be handled by the browser. Do not put a click handler on a link simply to duplicate the router directive.

## 20. What is the relationship between the outlet and active link directives?

The outlet renders the component for the active route. Router links request navigation to routes. `RouterLinkActive` observes the route state to reflect the current destination in the navigation UI. They cooperate, but each has a distinct responsibility.

## 21. How can I check the navigation manually?

Run the practice app and confirm that:

1. Each link leads to its intended URL.
2. The routed page appears at the outlet while the shared layout remains.
3. Browser Back and Forward restore the previous routes.
4. The active class moves to the expected link.
5. The root link uses exact matching when necessary.
6. The active link exposes `aria-current="page"`.
7. Link text and visual active styling remain understandable.

## 22. Where can I verify the APIs?

Use the official Angular [outlet guide](https://angular.dev/guide/routing/show-routes-with-outlets), [navigation guide](https://angular.dev/guide/routing/navigate-to-routes), [RouterLink API](https://angular.dev/api/router/RouterLink), and [RouterLinkActive API](https://angular.dev/api/router/RouterLinkActive).
