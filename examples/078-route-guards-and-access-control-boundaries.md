# Route guards and access control boundaries: example

This standalone example demonstrates four route decisions:

1. A signed-out visitor is redirected to a login page by `CanActivateFn`.
2. An admin section checks both activation and its child routes.
3. The editor asks before leaving when its text differs from its saved value.
4. `CanMatchFn` selects a reports page when a feature flag is enabled and a fallback page when it is disabled.

The in-memory session and role are for demonstrating client navigation only. They are not authentication or authorization. A real backend must validate every protected request independently.

## File layout

~~~text
src/
├── main.ts
└── app/
    ├── app.component.ts
    ├── app.config.ts
    ├── app.routes.ts
    ├── route-guards.ts
    ├── session.service.ts
    └── pages/
        ├── editor.page.ts
        ├── login.page.ts
        ├── page.page.ts
        └── workspace.page.ts
~~~

Create a standalone Angular application, then add or adapt these files. The project should include Angular Router.

## 1. Keep demo session and feature state in a service

Create `src/app/session.service.ts`:

~~~ts
import {Injectable, signal} from '@angular/core';

export type DemoRole = 'member' | 'admin';

@Injectable({providedIn: 'root'})
export class SessionService {
  private readonly role = signal<DemoRole | null>(null);
  private readonly reportsFeatureEnabled = signal(true);

  isSignedIn(): boolean {
    return this.role() !== null;
  }

  hasRole(expectedRole: DemoRole): boolean {
    return this.role() === expectedRole;
  }

  signInAs(role: DemoRole): void {
    this.role.set(role);
  }

  signOut(): void {
    this.role.set(null);
  }

  reportsAreEnabled(): boolean {
    return this.reportsFeatureEnabled();
  }

  toggleReportsFeature(): void {
    this.reportsFeatureEnabled.update(enabled => !enabled);
  }
}
~~~

This service makes the navigation states easy to demonstrate. It stores no credential and must not be used as the model for production authentication.

## 2. Write functional guards

Create `src/app/route-guards.ts`:

~~~ts
import {inject} from '@angular/core';
import {Router} from '@angular/router';
import type {
  CanActivateChildFn,
  CanActivateFn,
  CanDeactivateFn,
  CanMatchFn,
  UrlTree,
} from '@angular/router';

import type {EditorPage} from './pages/editor.page';
import type {DemoRole} from './session.service';
import {SessionService} from './session.service';

export const signedInGuard: CanActivateFn = (_route, state) => {
  const session = inject(SessionService);
  const router = inject(Router);

  if (session.isSignedIn()) {
    return true;
  }

  return router.createUrlTree(['/login'], {
    queryParams: {returnUrl: state.url},
  });
};

function checkAdminAccess(url: string): boolean | UrlTree {
  const session = inject(SessionService);
  const router = inject(Router);

  if (!session.isSignedIn()) {
    return router.createUrlTree(['/login'], {
      queryParams: {returnUrl: url},
    });
  }

  return session.hasRole('admin')
    ? true
    : router.createUrlTree(['/forbidden']);
}

export const adminGuard: CanActivateFn = (_route, state) =>
  checkAdminAccess(state.url);

export const adminChildGuard: CanActivateChildFn = (_childRoute, state) =>
  checkAdminAccess(state.url);

export const unsavedChangesGuard: CanDeactivateFn<EditorPage> = page => {
  if (!page.hasUnsavedChanges()) {
    return true;
  }

  // Browser-only demonstration. Use an accessible dialog service in a
  // production application, especially if the app also renders on a server.
  return window.confirm('Discard the unsaved editor changes?');
};

export const reportsFeatureGuard: CanMatchFn = () =>
  inject(SessionService).reportsAreEnabled();
~~~

`checkAdminAccess()` is called synchronously from each guard, so its `inject()` calls still happen in the active injection context. It returns to the login page for a signed-out visitor and to an access-denied page for a signed-in non-admin.

If you change this helper to await a request, capture injected services before the first `await`. Never call `inject()` after asynchronous work has started.

## 3. Create small route page components

Create `src/app/pages/page.page.ts`. The same simple component renders route-specific headings and descriptions from route data:

~~~ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-page',
  standalone: true,
  template: `
    <h1>{{ title }}</h1>
    <p>{{ description }}</p>
  `,
})
export class PagePage {
  private readonly route = inject(ActivatedRoute);

  get title(): string {
    return this.route.snapshot.data['title'] ?? 'Angular route';
  }

  get description(): string {
    return this.route.snapshot.data['description'] ?? '';
  }
}
~~~

Create `src/app/pages/workspace.page.ts`. Its router outlet displays the active child route:

~~~ts
import {Component} from '@angular/core';
import {RouterLink, RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-workspace',
  standalone: true,
  imports: [RouterLink, RouterOutlet],
  template: `
    <h1>Workspace</h1>
    <nav aria-label="Workspace">
      <a routerLink="overview">Overview</a>
      <a routerLink="admin/users">Admin users</a>
      <a routerLink="admin/settings">Admin settings</a>
    </nav>
    <router-outlet />
  `,
})
export class WorkspacePage {}
~~~

Create `src/app/pages/login.page.ts`. The return URL is accepted only when it is a local absolute path and does not start with `//`:

~~~ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute, Router} from '@angular/router';

import type {DemoRole} from '../session.service';
import {SessionService} from '../session.service';

@Component({
  selector: 'app-login',
  standalone: true,
  template: `
    <h1>Sign in (demo)</h1>
    <p>Choose a demo role to see the route behavior.</p>
    <button type="button" (click)="signIn('member')">Continue as member</button>
    <button type="button" (click)="signIn('admin')">Continue as admin</button>
  `,
})
export class LoginPage {
  private readonly route = inject(ActivatedRoute);
  private readonly router = inject(Router);
  private readonly session = inject(SessionService);

  async signIn(role: DemoRole): Promise<void> {
    this.session.signInAs(role);

    const returnUrl = this.route.snapshot.queryParamMap.get('returnUrl');

    if (returnUrl?.startsWith('/') && !returnUrl.startsWith('//')) {
      await this.router.navigateByUrl(returnUrl);
      return;
    }

    await this.router.navigateByUrl('/workspace/overview');
  }
}
~~~

A production sign-in flow would obtain and validate a real session with the server. This demo only changes an in-memory value.

Create `src/app/pages/editor.page.ts`:

~~~ts
import {Component, signal} from '@angular/core';

@Component({
  selector: 'app-editor',
  standalone: true,
  template: `
    <h1>Draft editor</h1>
    <label for="draft">Draft text</label>
    <textarea
      id="draft"
      rows="5"
      [value]="draft()"
      (input)="updateDraft($event)"
    ></textarea>
    <button type="button" (click)="save()">Save draft</button>
    <p aria-live="polite">
      {{ hasUnsavedChanges() ? 'Unsaved changes' : 'All changes saved' }}
    </p>
  `,
})
export class EditorPage {
  private readonly savedDraft = signal('');
  readonly draft = signal('');

  updateDraft(event: Event): void {
    const textarea = event.target as HTMLTextAreaElement;
    this.draft.set(textarea.value);
  }

  save(): void {
    this.savedDraft.set(this.draft());
  }

  hasUnsavedChanges(): boolean {
    return this.draft() !== this.savedDraft();
  }
}
~~~

The leave decision compares current text to saved text. Saving updates the baseline, so the guard allows navigation afterward.

## 4. Configure the routes

Create `src/app/app.routes.ts`:

~~~ts
import type {Routes} from '@angular/router';

import {EditorPage} from './pages/editor.page';
import {LoginPage} from './pages/login.page';
import {PagePage} from './pages/page.page';
import {WorkspacePage} from './pages/workspace.page';
import {
  adminChildGuard,
  adminGuard,
  reportsFeatureGuard,
  signedInGuard,
  unsavedChangesGuard,
} from './route-guards';

export const routes: Routes = [
  {
    path: 'login',
    component: LoginPage,
    title: 'Sign in',
  },
  {
    path: 'forbidden',
    component: PagePage,
    title: 'Access denied',
    data: {
      title: 'Access denied',
      description: 'This demo role cannot open that page.',
    },
  },
  {
    path: 'workspace',
    component: WorkspacePage,
    canActivate: [signedInGuard],
    title: 'Workspace',
    children: [
      {
        path: '',
        pathMatch: 'full',
        redirectTo: 'overview',
      },
      {
        path: 'overview',
        component: PagePage,
        title: 'Workspace overview',
        data: {
          title: 'Workspace overview',
          description: 'This page is available to signed-in demo users.',
        },
      },
      {
        path: 'admin',
        canActivate: [adminGuard],
        canActivateChild: [adminChildGuard],
        children: [
          {
            path: '',
            pathMatch: 'full',
            redirectTo: 'users',
          },
          {
            path: 'users',
            component: PagePage,
            title: 'Admin users',
            data: {
              title: 'Admin users',
              description: 'Only the demo admin role can open this route.',
            },
          },
          {
            path: 'settings',
            component: PagePage,
            title: 'Admin settings',
            data: {
              title: 'Admin settings',
              description: 'This child route is also checked by adminChildGuard.',
            },
          },
        ],
      },
    ],
  },
  {
    path: 'editor',
    component: EditorPage,
    canDeactivate: [unsavedChangesGuard],
    title: 'Draft editor',
  },
  {
    path: 'reports',
    component: PagePage,
    canMatch: [reportsFeatureGuard],
    title: 'Reports',
    data: {
      title: 'Reports',
      description: 'The feature flag selected this reports route.',
    },
  },
  {
    path: 'reports',
    component: PagePage,
    title: 'Reports unavailable',
    data: {
      title: 'Reports unavailable',
      description: 'CanMatch returned false, so the router used this fallback.',
    },
  },
  {
    path: '**',
    component: PagePage,
    title: 'Not found',
    data: {
      title: 'Not found',
      description: 'No configured route matches this URL.',
    },
  },
];
~~~

The two `reports` entries are intentional. When `reportsFeatureGuard` returns `false`, the first route is skipped and the next route with the same path can render its fallback.

The `admin` route is componentless and has child routes. Its `canActivate` checks access to the admin route; `canActivateChild` checks activation of its child pages. The surrounding `workspace` route has its own sign-in guard.

## 5. Add the top-level navigation and feature toggle

Create `src/app/app.component.ts`:

~~~ts
import {Component, inject} from '@angular/core';
import {Router, RouterLink, RouterOutlet} from '@angular/router';

import {SessionService} from './session.service';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterLink, RouterOutlet],
  template: `
    <header>
      <nav aria-label="Main navigation">
        <a routerLink="/workspace/overview">Workspace</a>
        <a routerLink="/editor">Editor</a>
        <a routerLink="/reports">Reports</a>
      </nav>
      <button type="button" (click)="toggleReports()">
        Toggle reports feature (currently {{ reportsAreEnabled() ? 'on' : 'off' }})
      </button>
      <button type="button" (click)="signOut()">Sign out</button>
    </header>

    <main>
      <router-outlet />
    </main>
  `,
})
export class AppComponent {
  private readonly router = inject(Router);
  private readonly session = inject(SessionService);

  reportsAreEnabled(): boolean {
    return this.session.reportsAreEnabled();
  }

  toggleReports(): void {
    this.session.toggleReportsFeature();
  }

  async signOut(): Promise<void> {
    this.session.signOut();
    await this.router.navigateByUrl('/login');
  }
}
~~~

Create `src/app/app.config.ts`:

~~~ts
import type {ApplicationConfig} from '@angular/core';
import {provideRouter} from '@angular/router';

import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)],
};
~~~

Create `src/main.ts`:

~~~ts
import {bootstrapApplication} from '@angular/platform-browser';

import {AppComponent} from './app/app.component';
import {appConfig} from './app/app.config';

bootstrapApplication(AppComponent, appConfig).catch(error => {
  console.error(error);
});
~~~

## 6. Try the navigation paths

Run the Angular development server and try these flows:

1. Open `/workspace/overview` while signed out. The guard sends the router to `/login` and includes the requested local URL as `returnUrl`.
2. Continue as a member. The overview loads. Open Admin users; the admin guard redirects to Access denied.
3. Sign in as admin and open Admin users or Admin settings. The child routes are allowed.
4. Open the editor, change the draft, and select another route. Canceling the browser confirmation keeps the editor active. Saving the draft makes the guard allow navigation.
5. Toggle the reports feature off, navigate away from Reports, then open Reports. `CanMatch` skips the first route definition and the fallback page appears. Toggle it on, navigate away again, and reopen Reports.
6. Sign out and try a workspace link again. The client router returns to the login page.

These steps demonstrate client navigation only. Add a server or API check and verify it rejects unauthorized requests even when a caller skips Angular routing.

## Common mistakes

- Returning `false` from a guard and separately calling `navigate()` instead of returning a `UrlTree` or `RedirectCommand`.
- Treating `CanMatch` false as a navigation denial when it actually skips that route configuration.
- Calling `inject()` after an `await`.
- Expecting asynchronous guards in one array to run as dependent, sequential setup steps.
- Assuming a hidden route or a client-side role signal protects backend data.
- Leaving a stale unsaved-changes flag set after a successful save.

## Related material

- [Lesson: Route guards and access control boundaries](../lessons/078-route-guards-and-access-control-boundaries.md)
- [Q&A: Route guards and access control boundaries](../qnas/078-route-guards-and-access-control-boundaries.md)
- [Angular route guards guide](https://angular.dev/guide/routing/route-guards)
- [CanActivateFn API](https://angular.dev/api/router/CanActivateFn)
- [CanActivateChildFn API](https://angular.dev/api/router/CanActivateChildFn)
- [CanDeactivateFn API](https://angular.dev/api/router/CanDeactivateFn)
- [CanMatchFn API](https://angular.dev/api/router/CanMatchFn)
