# 81. Nested routes, child outlets, and auxiliary routes

## What you will learn

After this lesson, you should be able to:

- Build a route tree with parent and child routes.
- Place a child route in the correct nested `RouterOutlet`.
- Understand how nested route URLs and relative links work.
- Use empty child routes and redirects without creating redirect loops.
- Explain componentless route groups.
- Add a named outlet for a secondary route branch.
- Open and close an auxiliary route while keeping the primary page visible.
- Choose between ordinary nested navigation and auxiliary routes.

## 1. Routes form a tree

A route does more than match a URL. A component route also identifies the outlet where Angular should display its component.

A simple application has one primary outlet:

~~~html
<router-outlet></router-outlet>
~~~

A route such as `{ path: 'about', component: AboutPage }` displays `AboutPage` in that outlet. A nested route adds another level to the route tree. Its component is displayed in an outlet inside the parent component.

For example, a mail application might have this hierarchy:

~~~text
AppComponent
└── primary outlet
    └── MailShell
        ├── primary outlet
        │   └── InboxPage or SentPage
        └── inspector outlet
            └── MessageInspectorPage
~~~

The root outlet and the outlet inside `MailShell` are different insertion points. Defining a child route does not create the child outlet for you: the parent template must contain it.

## 2. Configure child routes

Put child route definitions in the parent route's `children` array:

~~~ts
import type { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: 'mail',
    component: MailShell,
    children: [
      { path: '', pathMatch: 'full', redirectTo: 'inbox' },
      { path: 'inbox', component: InboxPage },
      { path: 'sent', component: SentPage },
    ],
  },
  { path: '**', component: NotFoundPage },
];
~~~

The route for `mail` matches the parent segment. Angular then matches the remaining URL against its children. The URLs are therefore:

| URL | Parent component | Child component |
| --- | --- | --- |
| `/mail` | `MailShell` | Redirects to `InboxPage` |
| `/mail/inbox` | `MailShell` | `InboxPage` |
| `/mail/sent` | `MailShell` | `SentPage` |

The parent's component stays active as Angular switches between its child routes. This makes the parent a useful place for a persistent header, navigation, or page layout.

The parent template must provide the child outlet:

~~~html
<header class="mail-header">
  <h1>Mail</h1>
  <nav aria-label="Mail folders">
    <a routerLink="inbox">Inbox</a>
    <a routerLink="sent">Sent</a>
  </nav>
</header>

<main>
  <router-outlet></router-outlet>
</main>
~~~

In a standalone component, import the directives used by its template:

~~~ts
import { Component } from '@angular/core';
import { RouterLink, RouterOutlet } from '@angular/router';

@Component({
  standalone: true,
  selector: 'app-mail-shell',
  imports: [RouterLink, RouterOutlet],
  templateUrl: './mail-shell.html',
})
export class MailShell {}
~~~

A standalone root component also imports `RouterOutlet`. A child component that uses links imports `RouterLink`; add `RouterLinkActive` when its template uses that directive.

## 3. Default child routes and redirects

An empty child path matches the parent URL without an additional segment. It is often used to choose the parent's initial page:

~~~ts
{ path: '', pathMatch: 'full', redirectTo: 'inbox' }
~~~

The `pathMatch: 'full'` option means the empty route only matches when the entire remaining URL is empty. It is important on empty-path redirects: without it, a redirect can match too broadly and prevent the intended child route from being selected.

You can render a component directly instead:

~~~ts
{ path: '', component: MailHomePage }
~~~

Use either a default component or a redirect based on the desired URL and page behavior. A redirect changes the URL to the destination; rendering a component at the empty path does not add a new segment.

## 4. Relative and absolute links

A link inside a routed component is resolved relative to that component's activated route unless it starts with a slash.

Inside `MailShell`, the child links are relative to `mail`:

~~~html
<a routerLink="inbox">Inbox</a>
<a routerLink="sent">Sent</a>
~~~

Inside `InboxPage`, a link to a sibling child route can move up one route level:

~~~html
<a [routerLink]="['../sent']">Open sent mail</a>
~~~

An absolute link starts at the application root:

~~~html
<a routerLink="/mail/inbox">Inbox</a>
~~~

Relative links are convenient inside reusable route sections. Absolute links make the destination explicit and are often easier to read in global navigation. When a link unexpectedly appends to the current URL, check whether it needs a leading slash or a different relative route level.

## 5. Read parameters from a child route

A parameter belongs to the route that declares it. Add a parameterized child route like this:

~~~ts
{ path: 'message/:id', component: MessagePage }
~~~

The corresponding URL is `/mail/message/42`. Read the parameter from the activated route for `MessagePage`:

~~~ts
import { Component, inject } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  standalone: true,
  selector: 'app-message-page',
  template: '<p>Message ID: {{ messageId }}</p>',
})
export class MessagePage {
  private readonly route = inject(ActivatedRoute);
  readonly messageId = this.route.snapshot.paramMap.get('id');
}
~~~

Use the snapshot when the component can be treated as a single route activation. If the same component instance can stay active while the parameter changes, subscribe to or otherwise react to the route's parameter stream so the displayed data updates.

Parent and child route information is represented at separate levels of the activated route tree. Read a value from the route that declares it, or walk through `parent` and `children` deliberately; do not assume every parameter is on the current route.

## 6. Componentless route groups

A route can have children and shared route configuration without a component:

~~~ts
{
  path: 'account',
  canActivate: [signedInGuard],
  children: [
    { path: 'profile', component: ProfilePage },
    { path: 'security', component: SecurityPage },
  ],
}
~~~

This groups related paths and can share route configuration such as guards, metadata, or providers. Because there is no parent component, the group does not add a layout of its own. The rendered child is placed in the nearest applicable outlet.

Use a component route when you need a persistent shell and a nested outlet. Use a componentless route when the route hierarchy is useful for configuration or organization but no additional component view is needed.

## 7. Named outlets and auxiliary routes

A primary outlet displays the main route branch. A named outlet provides another insertion point for a second branch that can be active alongside the primary branch.

Add a uniquely named outlet to the shell:

~~~html
<section class="mail-layout">
  <main>
    <router-outlet></router-outlet>
  </main>

  <aside aria-label="Message inspector">
    <router-outlet name="inspector"></router-outlet>
  </aside>
</section>
~~~

The implicit outlet name is `primary`. Named outlets need unique names within their outlet context. Configure the route with the matching `outlet` value:

~~~ts
{
  path: 'message/:id',
  component: MessageInspectorPage,
  outlet: 'inspector',
}
~~~

Place this route in the `children` of `mail`, alongside `inbox` and `sent`. The shell's primary outlet can then keep the inbox visible while the inspector outlet displays a selected message.

An auxiliary branch is serialized in the URL in parentheses, with the outlet name before a colon. For example:

~~~text
/mail/(inbox//inspector:message/42)
~~~

Here, `inbox` is the primary child and `message/42` is the branch for the `inspector` outlet. Auxiliary routes are still part of router navigation: their URL can be bookmarked, restored with browser history, and shared.

### Navigate to both branches

From `InboxPage`, target the shell route as the relative base and provide commands for both outlets:

~~~ts
import { Component, inject } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';

@Component({
  standalone: true,
  selector: 'app-inbox',
  template: '<button type="button" (click)="openMessage(42)">Inspect message 42</button>',
})
export class InboxPage {
  private readonly router = inject(Router);
  private readonly route = inject(ActivatedRoute);

  openMessage(id: number): void {
    void this.router.navigate(
      [{
        outlets: {
          primary: ['inbox'],
          inspector: ['message', id],
        },
      }],
      { relativeTo: this.route.parent },
    );
  }
}
~~~

The command explicitly keeps `inbox` in the primary outlet while opening the message in `inspector`. The parent route is used as the navigation base because both branches are children of `mail`. If your route tree differs, adjust `relativeTo` to the route that owns those outlets.

### Close the auxiliary branch

To close only the inspector, navigate with `null` for its outlet. In this example, the handler belongs to `MailShell`, whose activated route is the parent of both child outlets:

~~~ts
closeInspector(): void {
  void this.router.navigate(
    [{ outlets: { inspector: null } }],
    { relativeTo: this.route },
  );
}
~~~

Provide a visible close button in the inspector and give it an accessible name:

~~~html
<button type="button" (click)="closeInspector()">Close message inspector</button>
~~~

Clearing a named outlet removes that branch from the URL while leaving the primary child route active. Use the Angular router to perform the change rather than manually editing the address bar or hiding the outlet with CSS.

## 8. Layout and navigation design

A named outlet is useful when two route-driven views should coexist, such as:

- A list and a selected-item inspector.
- A workspace and a contextual help panel.
- A main page and a route-addressable dialog.

It adds URL and route-tree complexity. For a simple page transition, use ordinary primary child routes. For a panel that should be shareable, respond to browser back and forward, and survive a refresh, an auxiliary route can be a good fit.

Outlets choose where routed components are inserted; CSS chooses how those regions are arranged. Make sure a narrow-screen layout still presents the secondary content clearly. If the auxiliary content behaves like a dialog, also handle focus, keyboard dismissal, and an accessible label.

## 9. Common mistakes

- **Defining children but seeing a blank area:** the parent template is missing its child `RouterOutlet`.
- **Putting the child outlet in the wrong component:** the outlet must be in the component activated by the parent route.
- **Expecting nested routes to recreate the shell:** Angular keeps the parent active while switching children.
- **Using a relative link from the wrong level:** inspect the activated route tree and choose the correct base.
- **Using the wrong outlet name:** the outlet's `name` and route's `outlet` must match exactly.
- **Forgetting the named outlet in the layout:** a route can match but has no matching outlet context to display in.
- **Clearing the whole route while closing a panel:** set only the auxiliary outlet to `null`.
- **Creating a redirect loop:** use a precise empty-path redirect with `pathMatch: 'full'`.
- **Using auxiliary routes for every panel:** reserve them for secondary views that should participate in URL navigation and browser history.

## 10. A practical debugging checklist

1. Read the route configuration from the root down to the target child.
2. Confirm that every component route with children has the needed outlet in its template.
3. Confirm that the outlet is in the component activated at that parent level.
4. Compare the URL segments with the route tree, including parentheses and named outlet branches.
5. Check the spelling of the route's `outlet` and the template outlet's `name`.
6. Check the navigation base when using relative links or `relativeTo`.
7. Verify that an empty-path redirect uses an appropriate `pathMatch`.
8. Test direct URL loading and browser back/forward, not only button clicks.

## Key takeaways

- Nested routes are configured with `children` and rendered into an outlet inside the parent component.
- A parent can remain visible while Angular changes the child displayed in its nested outlet.
- Relative navigation depends on the current activated route; a leading slash makes a link root-relative.
- Componentless routes can group route configuration without introducing a parent view.
- A named outlet and a route configured with the same `outlet` name create an auxiliary route branch.
- Use auxiliary routes when the secondary view should be represented in navigation and browser history.

## Further reading

- [Show routes with outlets](https://angular.dev/guide/routing/show-routes)
- [Define routes](https://angular.dev/guide/routing/define-routes)
- [RouterOutlet API](https://angular.dev/api/router/RouterOutlet)
