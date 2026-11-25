# RouterOutlet, RouterLink, and active route links

Angular Router connects the browser URL to the view shown by the application. The route table maps paths to components, an outlet marks where the selected component is displayed, and router links let visitors navigate between those paths.

This lesson builds on the router setup from the previous chapter and focuses on displaying routed components and creating accessible navigation in a standalone application.

## Learning outcomes

After this lesson, you should be able to:

- Explain the role of `RouterOutlet`.
- Use `RouterLink` on semantic anchor elements.
- Explain how router navigation updates the URL without reloading the Angular app.
- Mark active navigation with `RouterLinkActive`.
- Expose the active page to assistive technology with `aria-current`.
- Choose exact or subset matching for active links.

## How the pieces fit

A simple routed application uses three pieces:

1. The route configuration maps a path such as `about` to a component such as `AboutPage`.
2. `RouterLink` connects an anchor to a destination route.
3. `RouterOutlet` marks where the component for the active route appears.

`RouterLinkActive` is an optional addition. It adds a CSS class and can set an accessibility state while its associated link matches the current route.

## Render a route with `RouterOutlet`

`RouterOutlet` is a directive from `@angular/router`. Import it in the standalone component whose template should display routed views:

~~~ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet],
  template: `
    <header>
      <a routerLink="/">Trail Notes</a>
    </header>

    <main>
      <router-outlet />
    </main>

    <footer>Learning Angular routing</footer>
  `,
})
export class AppComponent {}
~~~

When the URL matches a configured route, Angular creates that route's component immediately after the outlet. The outlet stays in the DOM as the reference point for later navigations. Shared layout such as a header and footer stays in place while the routed view changes.

An outlet is only a rendering location. It does not choose the route; the current URL and route configuration do that. A component that hosts child routes can have another outlet for its nested view, such as the content area inside a settings page.

## Navigate with `RouterLink`

`RouterLink` connects an anchor to a route. Import it in each standalone component template that uses it:

~~~ts
import {Component} from '@angular/core';
import {RouterLink} from '@angular/router';

@Component({
  selector: 'app-site-nav',
  imports: [RouterLink],
  template: `
    <nav aria-label="Main navigation">
      <a routerLink="/">Home</a>
      <a routerLink="/guides">Guides</a>
      <a routerLink="/about">About</a>
    </nav>
  `,
})
export class SiteNavComponent {}
~~~

Use an `<a>` for navigation. Anchors retain familiar link behavior, including keyboard focus and browser link actions. Use a `<button>` for an action on the current page, such as saving a form or opening a dialog.

For a normal internal router link, Angular navigates through the configured routes and updates the browser URL without requesting a new document for each page. Browser history is updated, so Back and Forward continue to navigate through visited routes.

The `routerLink` value can be a path string. A leading slash starts at the application root:

~~~html
<a routerLink="/guides">All guides</a>
~~~

A path without a leading slash is resolved relative to the current activated route. Relative links can be useful inside nested route areas, but their meaning depends on where they are rendered. For a small top-level menu, root-relative paths make each destination unambiguous. Route parameters and advanced relative navigation are covered later.

Avoid imitating ordinary navigation with a click handler:

~~~html
<!-- Avoid for normal navigation -->
<a (click)="goToAbout()">About</a>
~~~

That pattern has to recreate link semantics and browser behavior manually. Use `RouterLink` for navigation; reserve event handlers for actions that are not simply moving to another route.

## Style the active link with `RouterLinkActive`

`RouterLinkActive` observes the route associated with its link and applies one or more CSS classes while that route is active. Import it alongside `RouterLink`:

~~~ts
import {Component} from '@angular/core';
import {RouterLink, RouterLinkActive} from '@angular/router';

@Component({
  selector: 'app-site-nav',
  imports: [RouterLink, RouterLinkActive],
  template: `
    <nav aria-label="Main navigation">
      <a
        routerLink="/"
        routerLinkActive="active"
        [routerLinkActiveOptions]="{exact: true}"
        ariaCurrentWhenActive="page"
      >
        Home
      </a>

      <a
        routerLink="/guides"
        routerLinkActive="active"
        ariaCurrentWhenActive="page"
      >
        Guides
      </a>
    </nav>
  `,
})
export class SiteNavComponent {}
~~~

Angular adds the `active` class while a link is active and removes it when it becomes inactive. Style the class in the component stylesheet:

~~~css
nav a {
  color: #24415c;
  text-decoration: none;
}

nav a.active {
  color: #0b5cab;
  font-weight: 700;
  text-decoration: underline;
  text-underline-offset: 0.2em;
}
~~~

By default, active matching uses a **subset match**. For example, a `/guides` link can stay active on a guide detail route below `/guides`. This is useful when a section link should remain selected throughout its pages.

Use `[routerLinkActiveOptions]="{exact: true}"` when the class should apply only for an exact route match. The root link is a common case: exact matching prevents Home from appearing active on every URL under `/`.

`RouterLinkActive` can also be put on a parent element to style a group when one of its links is active. For precise per-link styling and `aria-current`, placing the directive on each anchor makes the relationship clear. An ancestor directive cannot inspect links hidden inside another component's template.

## Communicate the active page accessibly

A color change alone may not communicate the current page to everyone. `ariaCurrentWhenActive="page"` tells assistive technology that the active link represents the current page. Angular adds `aria-current="page"` while active and removes it when inactive.

Keep the visual style clear as well. Combine color with another cue such as an underline or heavier font weight. Give the navigation landmark an accessible label, for example `<nav aria-label="Main navigation">`, and use descriptive link text.

## Common mistakes

- **Forgetting the outlet:** the URL may change but no routed component appears. Import `RouterOutlet` and place it in the intended layout.
- **Forgetting a template import:** standalone components must import `RouterLink` and `RouterLinkActive` when their templates use them.
- **Using a button as a destination:** use an anchor with `RouterLink` for navigation; buttons represent actions.
- **Using only a click handler:** this discards native link behavior and adds accessibility work.
- **Making Home active on every route:** set exact matching for the root link.
- **Using color as the only active cue:** add a non-color visual cue and the `aria-current` state.
- **Expecting the outlet to choose a page:** the route configuration and URL select the component.

## Navigation cycle

1. A visitor activates an anchor containing `RouterLink`.
2. Angular resolves the destination against configured routes.
3. The router updates the URL and active route state.
4. `RouterOutlet` displays the route component.
5. `RouterLinkActive` updates its class and `aria-current` value.

## Official references

- [Show routes with outlets](https://angular.dev/guide/routing/show-routes-with-outlets)
- [Navigate to routes](https://angular.dev/guide/routing/navigate-to-routes)
- [RouterLink API](https://angular.dev/api/router/RouterLink)
- [RouterLinkActive API](https://angular.dev/api/router/RouterLinkActive)
- [Router reference](https://angular.dev/guide/routing/router-reference)
