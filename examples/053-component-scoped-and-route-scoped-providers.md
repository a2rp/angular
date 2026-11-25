# Component-scoped and route-scoped providers — example

This standalone app demonstrates two provider scopes:

- Two editor component instances each get an independent EditorDraft service.
- The admin route provides one AdminWorkspace service and one route label shared by its guard, layout, and child page.

The guard is only a dependency-injection demonstration. It always allows navigation and is not authentication or authorization.

## Create the project

Create a standalone Angular project with routing:

~~~sh
ng new angular-provider-scopes --standalone --routing --style=css
cd angular-provider-scopes
~~~

Create or replace the files shown below. The snippets target current Angular and use the Angular v22 Service decorator. In projects using Injectable, use Injectable() without providedIn for these manually scoped services.

## File layout

~~~text
src/
├── main.ts
└── app/
    ├── app.component.ts
    ├── app.config.ts
    ├── app.routes.ts
    ├── admin-area-name.ts
    ├── admin.guard.ts
    ├── admin-workspace.ts
    ├── editor-draft.ts
    ├── editor-panel.component.ts
    ├── editor-panel.component.html
    ├── home.page.ts
    ├── home.page.html
    ├── admin-layout.page.ts
    ├── admin-layout.page.html
    └── admin-dashboard.page.ts
~~~

## 1. Create a component-scoped editor state

~~~ts
// src/app/editor-draft.ts
import { Service, signal } from '@angular/core';

@Service({ autoProvided: false })
export class EditorDraft {
  readonly text = signal('');

  setText(value: string): void {
    this.text.set(value);
  }
}
~~~

~~~ts
// src/app/editor-panel.component.ts
import { Component, inject, Input } from '@angular/core';
import { EditorDraft } from './editor-draft';

@Component({
  selector: 'app-editor-panel',
  standalone: true,
  providers: [EditorDraft],
  templateUrl: './editor-panel.component.html',
})
export class EditorPanelComponent {
  @Input({ required: true }) label = '';

  protected readonly draft = inject(EditorDraft);

  protected updateText(event: Event): void {
    const input = event.target as HTMLInputElement;
    this.draft.setText(input.value);
  }
}
~~~

~~~html
<!-- src/app/editor-panel.component.html -->
<section>
  <h2>{{ label }}</h2>
  <label>
    Draft text
    <input
      [value]="draft.text()"
      (input)="updateText($event)"
    />
  </label>
  <p>Saved in this editor: {{ draft.text() || 'Nothing yet' }}</p>
</section>
~~~

Because EditorDraft is provided by EditorPanelComponent, each editor panel instance gets its own draft.

## 2. Render two independent component instances

~~~ts
// src/app/home.page.ts
import { Component } from '@angular/core';
import { RouterLink } from '@angular/router';
import { EditorPanelComponent } from './editor-panel.component';

@Component({
  standalone: true,
  imports: [RouterLink, EditorPanelComponent],
  templateUrl: './home.page.html',
})
export class HomePage {}
~~~

~~~html
<!-- src/app/home.page.html -->
<main>
  <h1>Component-scoped provider demo</h1>
  <p>Type different text in each editor. Each component owns a separate draft.</p>

  <app-editor-panel label="Editor A" />
  <app-editor-panel label="Editor B" />

  <p><a routerLink="/admin/dashboard">Open the admin feature</a></p>
</main>
~~~

Try typing into both fields. Each editor displays its own value because the nearest EditorDraft provider belongs to that editor component instance.

## 3. Create route-scoped feature dependencies

~~~ts
// src/app/admin-area-name.ts
import { InjectionToken } from '@angular/core';

export const ADMIN_AREA_NAME =
  new InjectionToken<string>('ADMIN_AREA_NAME');
~~~

~~~ts
// src/app/admin-workspace.ts
import { Service, signal } from '@angular/core';

@Service({ autoProvided: false })
export class AdminWorkspace {
  readonly updateCount = signal(0);

  canOpen(): boolean {
    // Demo only: this is not an authorization check.
    return true;
  }

  recordUpdate(): void {
    this.updateCount.update((count) => count + 1);
  }
}
~~~

The route will provide both AdminWorkspace and ADMIN_AREA_NAME. The service is not automatically registered in the application root.

## 4. Inject the route provider in a functional guard

~~~ts
// src/app/admin.guard.ts
import { inject } from '@angular/core';
import { CanActivateFn } from '@angular/router';
import { AdminWorkspace } from './admin-workspace';

export const adminGuard: CanActivateFn = () =>
  inject(AdminWorkspace).canOpen();
~~~

The Router runs this guard in the route's injection context, so it can resolve the dependency registered by the admin route.

## 5. Create a parent layout and child page

~~~ts
// src/app/admin-layout.page.ts
import { Component, inject } from '@angular/core';
import { RouterLink, RouterOutlet } from '@angular/router';
import { ADMIN_AREA_NAME } from './admin-area-name';
import { AdminWorkspace } from './admin-workspace';

@Component({
  standalone: true,
  imports: [RouterLink, RouterOutlet],
  templateUrl: './admin-layout.page.html',
})
export class AdminLayoutPage {
  protected readonly areaName = inject(ADMIN_AREA_NAME);
  protected readonly workspace = inject(AdminWorkspace);
}
~~~

~~~html
<!-- src/app/admin-layout.page.html -->
<main>
  <h1>{{ areaName }}</h1>
  <p>Updates recorded by this route: {{ workspace.updateCount() }}</p>
  <p><a routerLink="dashboard">Open dashboard</a></p>

  <router-outlet />
</main>
~~~

~~~ts
// src/app/admin-dashboard.page.ts
import { Component, inject } from '@angular/core';
import { AdminWorkspace } from './admin-workspace';

@Component({
  standalone: true,
  template: '<section><h2>Admin dashboard child route</h2><p>Shared updates: {{ workspace.updateCount() }}</p><button type="button" (click)="workspace.recordUpdate()">Record an update</button></section>',
})
export class AdminDashboardPage {
  protected readonly workspace = inject(AdminWorkspace);
}
~~~

The parent layout and child dashboard are in the admin route subtree. Both resolve the same route-provided AdminWorkspace instance, so the count displayed in both places changes together.

## 6. Register the route providers

~~~ts
// src/app/app.routes.ts
import type { Routes } from '@angular/router';
import { ADMIN_AREA_NAME } from './admin-area-name';
import { adminGuard } from './admin.guard';
import { AdminWorkspace } from './admin-workspace';

export const routes: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./home.page').then((module) => module.HomePage),
  },
  {
    path: 'admin',
    providers: [
      AdminWorkspace,
      { provide: ADMIN_AREA_NAME, useValue: 'Admin workspace' },
    ],
    canActivate: [adminGuard],
    loadComponent: () =>
      import('./admin-layout.page').then((module) => module.AdminLayoutPage),
    children: [
      {
        path: 'dashboard',
        loadComponent: () =>
          import('./admin-dashboard.page').then(
            (module) => module.AdminDashboardPage,
          ),
      },
      {
        path: '',
        pathMatch: 'full',
        redirectTo: 'dashboard',
      },
    ],
  },
  {
    path: '**',
    redirectTo: '',
  },
];
~~~

The home route is a sibling of the admin route. It cannot resolve the AdminWorkspace or ADMIN_AREA_NAME providers registered only on the admin route. The admin route's child dashboard can resolve both.

## 7. Configure the router and bootstrap

~~~ts
// src/app/app.config.ts
import type { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)],
};
~~~

~~~ts
// src/app/app.component.ts
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  template: '<router-outlet />',
})
export class AppComponent {}
~~~

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig).catch((error: unknown) => {
  console.error(error);
});
~~~

## 8. Run and inspect the behavior

~~~sh
ng serve
~~~

Open the local address printed by the Angular CLI.

1. On the home page, type different values in Editor A and Editor B. Their values remain independent.
2. Open the admin dashboard. The guard, layout, and dashboard resolve the same AdminWorkspace from the admin route.
3. Select **Record an update**. The shared count changes in both the layout and dashboard.
4. Return to the home page. Its components do not inject the admin route providers.

## What this example demonstrates

| Observation | Provider scope |
| --- | --- |
| Editor A and Editor B hold different text | Component providers create separate instances per component instance |
| Admin layout and dashboard show the same count | Route providers are shared with the route and its child routes |
| The admin guard can inject AdminWorkspace | Route providers are available in the route's guard injection context |
| Home does not use AdminWorkspace | A sibling route cannot resolve a provider registered only on the admin route |

A route provider controls dependency scope inside the client application. The demonstration guard returns true and is not a security boundary. Real authorization must be enforced by the server for protected data and actions.

## Further reading

- [The lesson](../lessons/053-component-scoped-and-route-scoped-providers.md)
- [The Q&A](../qnas/053-component-scoped-and-route-scoped-providers.md)
- [Hierarchical dependency injection](https://angular.dev/guide/di/hierarchical-dependency-injection)
- [Defining dependency providers](https://angular.dev/guide/di/defining-dependency-providers)
- [Route-level providers](https://angular.dev/guide/routing/define-routes#route-level-providers-for-dependency-injection)
- [Route API](https://angular.dev/api/router/Route)
