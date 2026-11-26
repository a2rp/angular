# 81. Nested routes, child outlets, and auxiliary routes — Examples

This example builds a small mail workspace with:

- A persistent mail shell.
- Nested primary routes for Inbox and Sent.
- A named `inspector` outlet that displays a message beside the primary page.
- A close action that removes only the inspector route.

The snippets use standalone Angular components and the Angular Router.

## Project files

~~~text
src/
├── main.ts
└── app/
    ├── app.component.ts
    ├── app.routes.ts
    ├── mail-shell.component.ts
    ├── mail-shell.component.html
    ├── mail-shell.component.css
    ├── inbox-page.component.ts
    ├── sent-page.component.ts
    └── message-inspector.component.ts
~~~

## 1. Bootstrap the router

### `src/main.ts`

~~~ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideRouter } from '@angular/router';
import { AppComponent } from './app/app.component';
import { routes } from './app/app.routes';

bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes)],
}).catch((error: unknown) => console.error(error));
~~~

### `src/app/app.component.ts`

The root outlet receives the `MailShell` component when the URL begins with `/mail`.

~~~ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  standalone: true,
  selector: 'app-root',
  imports: [RouterOutlet],
  template: '<router-outlet></router-outlet>',
})
export class AppComponent {}
~~~

## 2. Configure the nested and named routes

### `src/app/app.routes.ts`

The Inbox and Sent routes use the default primary child outlet. The message route uses the named `inspector` outlet.

~~~ts
import type { Routes } from '@angular/router';
import { InboxPageComponent } from './inbox-page.component';
import { MailShellComponent } from './mail-shell.component';
import { MessageInspectorComponent } from './message-inspector.component';
import { SentPageComponent } from './sent-page.component';

export const routes: Routes = [
  {
    path: 'mail',
    component: MailShellComponent,
    children: [
      { path: '', pathMatch: 'full', redirectTo: 'inbox' },
      { path: 'inbox', component: InboxPageComponent },
      { path: 'sent', component: SentPageComponent },
      {
        path: 'message/:id',
        component: MessageInspectorComponent,
        outlet: 'inspector',
      },
    ],
  },
  { path: '', pathMatch: 'full', redirectTo: 'mail' },
  { path: '**', redirectTo: 'mail' },
];
~~~

The wildcard route is last because it matches any remaining path. The `mail` route's empty child redirects to Inbox, so loading `/mail` selects the primary child route at `/mail/inbox`.

## 3. Build the persistent shell and its two outlets

### `src/app/mail-shell.component.ts`

~~~ts
import { Component } from '@angular/core';
import { RouterLink, RouterOutlet } from '@angular/router';

@Component({
  standalone: true,
  selector: 'app-mail-shell',
  imports: [RouterLink, RouterOutlet],
  templateUrl: './mail-shell.component.html',
  styleUrl: './mail-shell.component.css',
})
export class MailShellComponent {}
~~~

### `src/app/mail-shell.component.html`

The first outlet is the default primary outlet. The second is named `inspector`, matching the route configuration.

~~~html
<header class="mail-header">
  <a class="brand" routerLink="/mail/inbox">Mail workspace</a>

  <nav aria-label="Mail folders">
    <a routerLink="inbox">Inbox</a>
    <a routerLink="sent">Sent</a>
  </nav>
</header>

<div class="mail-layout">
  <main class="mail-main">
    <router-outlet></router-outlet>
  </main>

  <aside class="mail-inspector" aria-label="Message inspector">
    <router-outlet name="inspector"></router-outlet>
  </aside>
</div>
~~~

### `src/app/mail-shell.component.css`

The outlets are routing insertion points. CSS gives their surrounding regions a two-column layout.

~~~css
:host {
  display: block;
  min-height: 100vh;
  color: #172033;
  font-family: system-ui, sans-serif;
}

.mail-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 1rem 1.5rem;
  border-bottom: 1px solid #d7deea;
}

.brand {
  color: inherit;
  font-weight: 700;
  text-decoration: none;
}

.mail-header nav {
  display: flex;
  gap: 1rem;
}

.mail-layout {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(16rem, 24rem);
  min-height: calc(100vh - 4rem);
}

.mail-main,
.mail-inspector {
  min-width: 0;
  padding: 1.5rem;
}

.mail-inspector {
  border-left: 1px solid #d7deea;
  background: #f6f8fb;
}

@media (max-width: 700px) {
  .mail-layout {
    grid-template-columns: 1fr;
  }

  .mail-inspector {
    border-top: 1px solid #d7deea;
    border-left: 0;
  }
}
~~~

## 4. List messages and open one in the auxiliary outlet

### `src/app/inbox-page.component.ts`

The link commands target the primary and named outlets together. The route base is the parent route, `mail`, because it owns both child outlets.

~~~ts
import { Component, inject } from '@angular/core';
import { ActivatedRoute, RouterLink } from '@angular/router';

interface MessageSummary {
  id: number;
  subject: string;
  sender: string;
}

@Component({
  standalone: true,
  selector: 'app-inbox-page',
  imports: [RouterLink],
  template: `
    <section aria-labelledby="inbox-heading">
      <h1 id="inbox-heading">Inbox</h1>
      <p>Select a message to open it in the inspector.</p>

      <ul class="message-list">
        @for (message of messages; track message.id) {
          <li>
            <a
              [routerLink]="[{
                outlets: {
                  primary: ['inbox'],
                  inspector: ['message', message.id]
                }
              }]"
              [relativeTo]="mailRoute"
            >
              <strong>{{ message.subject }}</strong>
              <span> from {{ message.sender }}</span>
            </a>
          </li>
        }
      </ul>
    </section>
  `,
  styles: [`
    .message-list {
      display: grid;
      gap: 0.75rem;
      padding-left: 1.25rem;
    }

    .message-list a {
      color: #1649a5;
    }
  `],
})
export class InboxPageComponent {
  readonly mailRoute = inject(ActivatedRoute).parent!;

  readonly messages: MessageSummary[] = [
    { id: 42, subject: 'Welcome to the workspace', sender: 'Ari' },
    { id: 57, subject: 'Project notes', sender: 'Sam' },
    { id: 63, subject: 'Friday review', sender: 'Lee' },
  ];
}
~~~

When a link is followed, the primary branch remains `inbox` and the `inspector` branch changes to the selected message. For message 42, the resulting URL is:

~~~text
/mail/(inbox//inspector:message/42)
~~~

The `@for` block uses each message ID as its tracking key, so Angular can identify each list item as the list changes.

## 5. Add a second primary child route

### `src/app/sent-page.component.ts`

This simple page is activated in the same primary outlet as Inbox. The parent shell remains in place during the switch.

~~~ts
import { Component } from '@angular/core';
import { RouterLink } from '@angular/router';

@Component({
  standalone: true,
  selector: 'app-sent-page',
  imports: [RouterLink],
  template: `
    <section>
      <h1>Sent</h1>
      <p>This is another child page inside the persistent mail shell.</p>
      <a routerLink="../inbox">Return to Inbox</a>
    </section>
  `,
})
export class SentPageComponent {}
~~~

The `../inbox` link is relative to the current child route. An absolute `/mail/inbox` link would also work.

## 6. Display the selected message and close the inspector

### `src/app/message-inspector.component.ts`

The inspector route declares the `id` parameter. This component reads the parameter stream so its displayed value can update if the same outlet is reused for another message.

~~~ts
import { AsyncPipe } from '@angular/common';
import { Component, inject } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { map } from 'rxjs';

@Component({
  standalone: true,
  selector: 'app-message-inspector',
  imports: [AsyncPipe],
  template: `
    <article>
      <button type="button" (click)="close()">Close message inspector</button>

      @if (messageId$ | async; as messageId) {
        <h2>Message {{ messageId }}</h2>
        <p>This message is displayed in the auxiliary inspector outlet.</p>
      }
    </article>
  `,
})
export class MessageInspectorComponent {
  private readonly route = inject(ActivatedRoute);
  private readonly router = inject(Router);

  readonly messageId$ = this.route.paramMap.pipe(
    map((params) => params.get('id')),
  );

  close(): void {
    void this.router.navigate(
      [{ outlets: { inspector: null } }],
      { relativeTo: this.route.parent },
    );
  }
}
~~~

The message route is a child of `mail`, so `this.route.parent` is the route that owns the primary and inspector outlets. Setting only `inspector` to `null` removes that branch and leaves the current primary page active.

## 7. Try the navigation

With the route configuration above:

1. Open `/mail`. The empty child redirect selects Inbox.
2. Select **Project notes**. Inbox remains visible, the inspector displays message 57, and the URL includes the `inspector` branch.
3. Select another message. The inspector updates to the newly selected message.
4. Select **Close message inspector**. The auxiliary branch disappears while Inbox stays active.
5. Open `/mail/sent`. Sent replaces Inbox in the primary outlet while the mail shell remains.
6. Open `/mail/(inbox//inspector:message/42)` directly. Both branches should be restored from the URL.

These are manual navigation checks; no automated test suite is included in this example.

## Route tree and outlet mapping

~~~text
Route config                          Template outlet
───────────────────────────────────   ─────────────────────────────
mail                                  AppComponent: primary outlet
├── inbox                             MailShell: primary outlet
├── sent                              MailShell: primary outlet
└── message/:id (inspector outlet)    MailShell: inspector outlet
~~~

The top-level `mail` route is activated in the app's primary outlet. Its children are resolved in the outlets inside `MailShell`: the ordinary child routes use the primary outlet, and the message route uses the named inspector outlet.

## Common adjustments

- **Show a placeholder when the inspector is closed:** add ordinary shell content around the named outlet and use route state to decide when a prompt should appear.
- **Use a different main page while inspecting:** target a different primary child in the same outlets command.
- **Open a message from another child page:** keep the same outlet command structure, but set `relativeTo` to the route that owns the outlets.
- **Use a full-page message view:** remove the `outlet: 'inspector'` setting and navigate to a regular primary child route.
- **Change the layout on small screens:** adjust the CSS while keeping the same route configuration.

## What this example demonstrates

- The parent component provides a persistent shell and nested outlets.
- Primary child routes replace one another inside the shell.
- A named outlet can render a second route branch at the same time.
- Outlet command links can target multiple branches in one navigation.
- Setting one named outlet to `null` closes that branch without clearing the primary view.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="080-lazy-load-routes-and-standalone-components.md">080 - Lazy-load routes and standalone components</a></td><td><a href="../lessons/081-nested-routes-child-outlets-and-auxiliary-routes.md">Lesson</a></td><td><a href="../qnas/081-nested-routes-child-outlets-and-auxiliary-routes.md">Q&amp;A</a></td><td><a href="082-route-level-providers-and-route-configuration.md">082 - 82. Route-level providers and route configuration</a></td>
</tr>
  </tbody>
</table>
