# Hierarchical dependency injection — example

This standalone Angular example demonstrates three provider-scope behaviors:

1. A root-provided ThemeService is available in both editor trees.
2. Each EditorComponent has its own EditorSessionService instance, while the toolbar and preview inside that editor share it.
3. A provider in viewProviders is visible to a component in the owner’s view, but projected content falls back to the root token value.

The service decorator shown here uses the current Angular v22 @Service() API. In projects using an older Angular version, replace @Service() with the version-appropriate @Injectable() metadata; keep EditorSessionService registered in the component’s providers array.

## File layout

~~~text
src/
├── main.ts
└── app/
    ├── app.component.ts
    ├── app.component.html
    ├── theme.service.ts
    ├── editor-session.service.ts
    ├── editor-toolbar.component.ts
    ├── editor-preview.component.ts
    ├── editor.component.ts
    ├── editor.component.html
    ├── boundary-label.token.ts
    ├── scope-probe.component.ts
    ├── provider-boundary.component.ts
    └── provider-boundary.component.html
~~~

## 1. Create a root-provided service

ThemeService has application-wide scope. Both editor components can inject it from the root EnvironmentInjector.

~~~ts
// src/app/theme.service.ts
import { Service } from '@angular/core';

@Service()
export class ThemeService {
  readonly name = 'Shared root theme';
}
~~~

## 2. Create state that should be isolated per editor

This service is not automatically registered in the root. EditorComponent will register it in its own providers array.

~~~ts
// src/app/editor-session.service.ts
import { Service, signal } from '@angular/core';

@Service({ autoProvided: false })
export class EditorSessionService {
  readonly draft = signal('Original draft');

  appendLocalEdit(): void {
    this.draft.update((text) => text + ' | local edit');
  }
}
~~~

## 3. Add child components that share the editor’s instance

The toolbar and preview each request EditorSessionService. They are rendered inside an EditorComponent, so both resolve the instance registered by that editor.

~~~ts
// src/app/editor-toolbar.component.ts
import { Component, inject } from '@angular/core';
import { EditorSessionService } from './editor-session.service';

@Component({
  selector: 'app-editor-toolbar',
  standalone: true,
  template: '<button type="button" (click)="session.appendLocalEdit()">Add a local edit</button>',
})
export class EditorToolbarComponent {
  readonly session = inject(EditorSessionService);
}
~~~

~~~ts
// src/app/editor-preview.component.ts
import { Component, inject } from '@angular/core';
import { EditorSessionService } from './editor-session.service';

@Component({
  selector: 'app-editor-preview',
  standalone: true,
  template: '<p>Draft: {{ session.draft() }}</p>',
})
export class EditorPreviewComponent {
  readonly session = inject(EditorSessionService);
}
~~~

## 4. Scope the session service to each editor

The providers array creates an EditorSessionService instance for each EditorComponent. The editor itself and its eligible descendants use that instance. skipSelf starts a second request at the parent; because the service is not root-provided, that optional lookup returns null.

~~~ts
// src/app/editor.component.ts
import { Component, Input, inject } from '@angular/core';
import { EditorPreviewComponent } from './editor-preview.component';
import { EditorSessionService } from './editor-session.service';
import { EditorToolbarComponent } from './editor-toolbar.component';
import { ThemeService } from './theme.service';

@Component({
  selector: 'app-editor',
  standalone: true,
  imports: [EditorToolbarComponent, EditorPreviewComponent],
  providers: [EditorSessionService],
  templateUrl: './editor.component.html',
})
export class EditorComponent {
  @Input({ required: true }) name = '';

  readonly theme = inject(ThemeService);
  readonly parentSession = inject(EditorSessionService, {
    optional: true,
    skipSelf: true,
  });
}
~~~

~~~html
<!-- src/app/editor.component.html -->
<section>
  <h3>{{ name }}</h3>
  <p>Theme: {{ theme.name }}</p>
  <app-editor-toolbar />
  <app-editor-preview />
  <p>Parent editor session exists: {{ parentSession !== null }}</p>
</section>
~~~

The local service is available to the EditorComponent template’s toolbar and preview. The parent AppComponent cannot inject this provider by searching downward.

## 5. Define a token with a root fallback

The token represents a string label. Its root factory supplies a fallback value, which makes it possible to observe which scope a consumer resolves.

~~~ts
// src/app/boundary-label.token.ts
import { InjectionToken } from '@angular/core';

export const BOUNDARY_LABEL = new InjectionToken<string>(
  'boundary.label',
  {
    providedIn: 'root',
    factory: () => 'Root EnvironmentInjector fallback',
  },
);
~~~

## 6. Add a probe component

This component displays whichever BOUNDARY_LABEL provider is visible at its injection site.

~~~ts
// src/app/scope-probe.component.ts
import { Component, inject } from '@angular/core';
import { BOUNDARY_LABEL } from './boundary-label.token';

@Component({
  selector: 'app-scope-probe',
  standalone: true,
  template: '<p>Resolved label: {{ label }}</p>',
})
export class ScopeProbeComponent {
  readonly label = inject(BOUNDARY_LABEL);
}
~~~

## 7. Provide a view-only override

ProviderBoundaryComponent overrides the root token in viewProviders. The probe created in this component’s own template sees the local value. The probe supplied through ng-content is projected content, so it cannot see that viewProviders entry and resolves the root fallback instead.

~~~ts
// src/app/provider-boundary.component.ts
import { Component, inject } from '@angular/core';
import { BOUNDARY_LABEL } from './boundary-label.token';
import { ScopeProbeComponent } from './scope-probe.component';

@Component({
  selector: 'app-provider-boundary',
  standalone: true,
  imports: [ScopeProbeComponent],
  viewProviders: [
    {
      provide: BOUNDARY_LABEL,
      useValue: 'ProviderBoundary own view',
    },
  ],
  templateUrl: './provider-boundary.component.html',
})
export class ProviderBoundaryComponent {
  readonly ownLabel = inject(BOUNDARY_LABEL);
}
~~~

~~~html
<!-- src/app/provider-boundary.component.html -->
<section>
  <h2>Provider visibility boundary</h2>

  <h3>Requested by ProviderBoundary itself</h3>
  <p>{{ ownLabel }}</p>

  <h3>Requested by a child in its own view</h3>
  <app-scope-probe />

  <h3>Requested by projected content</h3>
  <ng-content />
</section>
~~~

## 8. Assemble the application

The root component imports the reusable components and renders two editors. Its template also projects a probe into ProviderBoundaryComponent.

~~~ts
// src/app/app.component.ts
import { Component, inject } from '@angular/core';
import { EditorComponent } from './editor.component';
import { ProviderBoundaryComponent } from './provider-boundary.component';
import { ScopeProbeComponent } from './scope-probe.component';
import { ThemeService } from './theme.service';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    EditorComponent,
    ProviderBoundaryComponent,
    ScopeProbeComponent,
  ],
  templateUrl: './app.component.html',
})
export class AppComponent {
  readonly theme = inject(ThemeService);
}
~~~

~~~html
<!-- src/app/app.component.html -->
<main>
  <h1>Hierarchical dependency injection</h1>
  <p>Root theme: {{ theme.name }}</p>

  <section>
    <h2>Independent editor sessions</h2>
    <app-editor name="Editor A" />
    <app-editor name="Editor B" />
  </section>

  <app-provider-boundary>
    <app-scope-probe />
  </app-provider-boundary>
</main>
~~~

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

## 9. Run the application

Place the files in an Angular standalone application and start its development server:

~~~sh
ng serve
~~~

Open the local address printed by Angular CLI.

## Expected behavior

### Editor instances

- Both editors display the same root theme.
- Each editor initially displays its own draft state.
- Click Add a local edit in Editor A. Only Editor A’s preview changes.
- Click the button in Editor B. Its draft changes independently.
- Each editor reports that no parent editor session exists because the service is provided locally, not in the root.

The toolbar and preview inside one editor share its instance because both look upward to the EditorComponent provider. The two EditorComponent instances do not share their component-level providers.

### viewProviders boundary

You should see:

- ProviderBoundary itself resolves ProviderBoundary own view.
- The probe created in ProviderBoundary’s template resolves ProviderBoundary own view.
- The projected probe resolves Root EnvironmentInjector fallback.

This shows that viewProviders is visible to the component’s own view but not to projected content. If the local registration were moved to providers, projected content could resolve the component-level value instead.

## What this example demonstrates

| Observation | Injector rule |
| --- | --- |
| Both editor subtrees see the root theme | The root EnvironmentInjector is an ancestor for both requests |
| Each editor draft changes independently | Each EditorComponent has its own ElementInjector provider instance |
| Toolbar and preview in one editor see the same draft | Both resolve the nearest EditorSessionService provider |
| Parent lookup for EditorSessionService is null | skipSelf skips the editor’s provider and no parent provider exists |
| Internal probe sees the local boundary label | The provider is registered in the component’s viewProviders |
| Projected probe sees the root fallback | Projected content cannot see the component’s viewProviders |

## Common adjustments

- To share EditorSessionService between both editors, move its provider to AppComponent or another common ancestor and remove it from EditorComponent.
- To give every editor its own state, keep the provider on EditorComponent.
- To make projected content resolve the component-level BOUNDARY_LABEL, replace viewProviders with providers.
- To check only the current element injector, use self: true. Pair it with optional: true if absence is valid.
- To read a parent’s provider while skipping a local one, use skipSelf: true. Pair it with optional: true if the parent may not provide the token.
- Use host: true when a request should stop at its host component boundary.

## Related files

- Lesson: [Hierarchical dependency injection](../lessons/051-hierarchical-dependency-injection.md)
- Q&A: [Hierarchical dependency injection](../qnas/051-hierarchical-dependency-injection.md)
- Next lesson: [Application and environment providers](../lessons/052-application-and-environment-providers.md)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="050-inject-dependencies-with-inject-and-constructor-injection.md">050 - Inject dependencies with `inject()` and constructor injection</a></td><td><a href="../lessons/051-hierarchical-dependency-injection.md">Lesson</a></td><td><a href="../qnas/051-hierarchical-dependency-injection.md">Q&amp;A</a></td><td><a href="052-application-and-environment-providers.md">052 - Application and environment providers</a></td>
</tr>
  </tbody>
</table>
