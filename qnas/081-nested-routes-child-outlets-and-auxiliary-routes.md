# 81. Nested routes, child outlets, and auxiliary routes — Q&A

Questions and answers for [Lesson 81](../lessons/081-nested-routes-child-outlets-and-auxiliary-routes.md).

## Core concepts

### 1. What makes a route nested?

A route is nested when it is declared inside another route's `children` array. The parent route matches an earlier part of the URL; Angular then matches the remaining segments against the children.

~~~ts
{
  path: 'mail',
  component: MailShell,
  children: [
    { path: 'inbox', component: InboxPage },
  ],
}
~~~

The child URL is `/mail/inbox`.

### 2. Where does Angular render a child route?

Angular renders a child route in a `RouterOutlet` inside the parent component's template. The root outlet is not automatically reused as every nested level's outlet.

For the example above, `MailShell` needs its own outlet. Without it, the route configuration does not have the intended insertion point for the child page.

### 3. Does Angular recreate the parent when its child changes?

Usually, no. When navigation stays within the same parent route, Angular keeps the parent component active and updates the child component in the parent's outlet.

That is why a shell component can keep its navigation or layout visible while the outlet changes from Inbox to Sent.

### 4. How do parent and child URL segments combine?

Angular combines the matched segments in route-tree order. A parent with path `mail` and a child with path `inbox` match `/mail/inbox`.

The `children` array describes route structure; it does not mean that every child URL needs a special separator.

### 5. What is the difference between a nested route and a nested outlet?

A nested route is route configuration under `children`. A nested outlet is the `RouterOutlet` in the parent component's template where the child view is inserted. A working component child route generally needs both.

### 6. Can a route have children without a component?

Yes. A componentless route can group child routes and share route configuration such as guards, metadata, or providers. It does not add a parent component layout or a parent view of its own.

Use a component route when the parent needs to stay visible and provide an outlet for child views.

## Empty routes, redirects, and links

### 7. What does an empty child path match?

`path: ''` matches the parent URL without consuming an additional path segment. It can render a default child component or redirect to a named child.

### 8. Why should an empty-path redirect usually specify `pathMatch: 'full'`?

It makes the redirect apply only when the full remaining URL is empty. This prevents an empty route from matching more broadly than intended and helps avoid redirect loops or unexpectedly intercepting child URLs.

~~~ts
{ path: '', pathMatch: 'full', redirectTo: 'inbox' }
~~~

### 9. Should the default child be a redirect or a component?

Use a redirect when the URL should become the canonical child URL, such as changing `/mail` to `/mail/inbox`. Render a component at the empty path when the page should appear at `/mail` without adding another URL segment.

### 10. What is the difference between a relative and an absolute router link?

A relative link is resolved from the current activated route. An absolute link starts at the application root:

~~~html
<a routerLink="sent">Relative destination</a>
<a routerLink="/mail/sent">Root-relative destination</a>
~~~

Choose the form based on the location of the component and the desired route-tree level.

### 11. Why did a relative link append the wrong URL segment?

It may have been resolved from a deeper route than expected. Check which route activated the component containing the link. Use a leading slash for a root-relative destination, or adjust the relative commands to move to the correct parent level.

### 12. How should a child route link to a sibling?

From a child page such as Inbox, a route-relative command can move up to the parent and then to Sent:

~~~html
<a [routerLink]="['../sent']">Sent</a>
~~~

The exact relative command depends on the current activated route. If route levels are confusing, use an absolute route or inspect the activated route tree.

## Parameters and route structure

### 13. Where do I read a child route parameter?

Read it from the `ActivatedRoute` for the component whose route declares the parameter. For `path: 'message/:id'`, the Message component can read `paramMap.get('id')`.

~~~ts
const id = this.route.snapshot.paramMap.get('id');
~~~

### 14. When is a route snapshot insufficient?

A snapshot is a point-in-time view. If the same component instance can remain active while its route parameter changes, react to the route's parameter stream instead of assuming the component will be recreated with a new snapshot.

### 15. Are parent parameters automatically copied onto a child route?

Do not assume every parameter is available on the current route. Route parameters belong to route levels. Read the parameter from the route that declares it, or deliberately inspect its parent/child route relationship.

### 16. Can several levels of child routes be nested?

Yes. A parent can have children that themselves have children. Each component route that should render another routed view needs an outlet at the corresponding level. Nest only as deeply as the layout and URL structure require.

## Named outlets and auxiliary routes

### 17. What is a named outlet?

It is a router outlet with a name, for example:

~~~html
<router-outlet name="inspector"></router-outlet>
~~~

The default outlet name is `primary`. A named outlet creates another router insertion point so a secondary route branch can be active alongside the primary one.

### 18. How does a route target a named outlet?

Set the route's `outlet` property to the same name as the outlet:

~~~ts
{
  path: 'message/:id',
  component: MessageInspectorPage,
  outlet: 'inspector',
}
~~~

The name must match exactly, and the outlet must be present in the relevant component's template.

### 19. What is an auxiliary route?

An auxiliary route is a route branch rendered in a named outlet. It can keep the main view active while displaying another route-driven view, such as an inspector beside a list.

Auxiliary routes participate in router navigation and URL state; they are not just a visual panel hidden or shown by CSS.

### 20. How are named outlet branches represented in the URL?

Angular serializes the secondary branch in parentheses, with the outlet name before a colon. For the lesson's mail example:

~~~text
/mail/(inbox//inspector:message/42)
~~~

The primary child is Inbox, and the `inspector` outlet displays message 42. Parentheses group the parallel branches, and `//` separates them.

### 21. How can I open a message in the inspector without leaving the inbox?

Navigate to both child outlets from their common parent. Keep `inbox` in the primary branch and add the message route to `inspector`:

~~~ts
this.router.navigate(
  [{
    outlets: {
      primary: ['inbox'],
      inspector: ['message', id],
    },
  }],
  { relativeTo: this.route.parent },
);
~~~

Here, `relativeTo` points to the route that owns those child outlets. Change that base if your route configuration has a different shape.

### 22. How do I close only a named outlet?

Navigate with `null` for that outlet, using the route that owns it as the relative base:

~~~ts
this.router.navigate(
  [{ outlets: { inspector: null } }],
  { relativeTo: this.route },
);
~~~

This removes the inspector branch while keeping the primary route active. If the navigation clears more of the URL than expected, verify the `relativeTo` route and outlet ownership.

### 23. Can multiple named outlets be active at the same time?

A route tree can contain multiple named outlets, and a navigation can target multiple outlets together. Each outlet name is a distinct branch. Each outlet displays one active routed component at a time.

### 24. Does a named outlet automatically create a side-by-side layout?

No. The outlet controls where Angular inserts the routed component. CSS controls placement, dimensions, stacking, and responsive behavior.

### 25. When should I use an auxiliary route instead of a normal child route?

Use a normal primary child route when navigation should replace the main page. Consider an auxiliary route when a secondary view should coexist with the primary view and its state should be reflected in the URL and browser history.

Avoid adding auxiliary routing complexity to a panel that does not need route navigation or URL persistence.

## Troubleshooting and design

### 26. Why does a named route match but not appear?

Check that the corresponding named outlet exists in the component activated at that route level. Then verify that the outlet's `name` matches the route's `outlet` exactly.

Also inspect the URL and route tree to ensure the branch is being navigated relative to the correct parent.

### 27. Why does closing an inspector also navigate away from the main page?

The navigation may be relative to the wrong route, or it may replace the primary branch as well. Target only the named outlet with `null` and use the route that owns the outlets as `relativeTo`.

### 28. Can I hide a routed panel with CSS instead of closing its route?

You can control its visual appearance with CSS, but hiding it does not remove its route branch or URL state. If the panel is meant to be closed as navigation, update the router state by clearing its outlet.

### 29. What accessibility concerns apply to an auxiliary panel?

Give the panel a meaningful label. If it behaves like a dialog, manage focus, keyboard dismissal, and focus restoration. Provide a clear close control with an accessible name, and ensure the layout remains usable on narrow screens.

### 30. How do browser back and forward work with an auxiliary route?

They follow the navigations recorded by the Angular Router. Opening or closing an auxiliary branch changes router state and URL, so users can move through those states with browser history according to the navigations performed.

### 31. What should I check when nested routing fails?

Verify, in order:

1. The route is under the intended parent's `children`.
2. The URL has the expected parent and child segments.
3. The component activated by the parent contains the child outlet.
4. The outlet name matches the route's `outlet` value, if named.
5. A relative link or navigation uses the correct route base.
6. Empty-path redirects use the intended `pathMatch`.
7. The target component imports the router directives its template uses.

### 32. Is an outlet a wrapper around the routed component?

The outlet is an insertion anchor. Angular inserts the activated routed component next to that anchor in the DOM; the outlet itself does not provide a visual layout wrapper. Use a surrounding element when the layout needs one.

## Quick review

- **Nested child route:** configured under `children` and rendered in the parent's outlet.
- **Componentless route:** groups route configuration without adding a parent component view.
- **Relative link:** resolves from the current activated route.
- **Named outlet:** an additional outlet addressed by a matching route `outlet` name.
- **Auxiliary route:** a secondary router branch that can coexist with the primary branch.
- **Close one auxiliary branch:** navigate with that outlet set to `null`.
