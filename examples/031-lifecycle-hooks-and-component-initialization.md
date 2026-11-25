# 31. Example: Lifecycle hooks and component initialization

This small standalone Angular app lets you watch a component's lifecycle in the browser console. It demonstrates initial input setup, a later input change, post-render work, and cleanup when Angular destroys and recreates the child component.

The example uses the modern standalone component and signal APIs. Put these files into a modern Angular CLI application's `src` directory. Keep any providers your existing `app.config.ts` needs; the `main.ts` below is a minimal bootstrap for an app that does not need extra providers.

## 1. Bootstrap the root component

~~~ts
// src/main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app';

bootstrapApplication(AppComponent).catch((error: unknown) => {
  console.error(error);
});
~~~

If your CLI project already bootstraps `AppComponent` through `app.config.ts`, keep that setup and make sure it points to this root component instead of replacing it with the minimal bootstrap above.

## 2. Create the parent component

~~~ts
// src/app/app.ts
import { Component, signal } from '@angular/core';
import { LifecycleDemo } from './lifecycle-demo';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [LifecycleDemo],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class AppComponent {
  readonly label = signal('Initial input');
  readonly showDemo = signal(true);

  updateLabel(): void {
    this.label.set(`Updated at ${new Date().toLocaleTimeString()}`);
  }

  toggleDemo(): void {
    this.showDemo.update((visible) => !visible);
  }
}
~~~

~~~html
<!-- src/app/app.html -->
<main class="page">
  <header>
    <p class="eyebrow">Angular lifecycle demo</p>
    <h1>Watch a component's lifecycle</h1>
    <p>
      Open the browser developer tools and select the Console tab. The child
      component logs its lifecycle hooks there.
    </p>
  </header>

  <div class="controls">
    <button type="button" (click)="updateLabel()">Change input</button>
    <button type="button" (click)="toggleDemo()">
      @if (showDemo()) {
        Destroy child
      } @else {
        Create child
      }
    </button>
  </div>

  @if (showDemo()) {
    <app-lifecycle-demo [label]="label()">
      <p class="projected-message">
        This paragraph is projected into the child with ng-content.
      </p>
    </app-lifecycle-demo>
  } @else {
    <p class="status">The child component has been destroyed.</p>
  }

  <p class="tip">
    Clear the Console before each action if you want to follow one lifecycle
    sequence at a time. Checked hooks can log more than once during a render.
  </p>
</main>
~~~

~~~css
/* src/app/app.css */
:host {
  display: block;
  min-height: 100vh;
  color: #1f2937;
  background: #f3f6fb;
  font-family: system-ui, sans-serif;
}

.page {
  width: min(760px, calc(100% - 2rem));
  margin: 0 auto;
  padding: 3rem 0;
}

header,
.controls {
  margin-bottom: 1.5rem;
}

.eyebrow {
  color: #7c3aed;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1 {
  margin: 0.35rem 0 0.75rem;
  font-size: clamp(1.8rem, 5vw, 2.6rem);
}

header p,
.tip,
.status {
  line-height: 1.6;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
}

button {
  padding: 0.7rem 1rem;
  border: 0;
  border-radius: 0.5rem;
  color: white;
  background: #6d28d9;
  font: inherit;
  font-weight: 650;
  cursor: pointer;
}

button:hover {
  background: #5b21b6;
}

button:focus-visible {
  outline: 3px solid #c4b5fd;
  outline-offset: 3px;
}

.tip {
  margin-top: 1.5rem;
  color: #4b5563;
}
~~~

## 3. Create the child component that logs lifecycle hooks

~~~ts
// src/app/lifecycle-demo.ts
import {
  AfterContentChecked,
  AfterContentInit,
  AfterViewChecked,
  AfterViewInit,
  Component,
  DestroyRef,
  DoCheck,
  OnChanges,
  OnDestroy,
  OnInit,
  SimpleChanges,
  afterNextRender,
  inject,
  input,
} from '@angular/core';

@Component({
  selector: 'app-lifecycle-demo',
  standalone: true,
  templateUrl: './lifecycle-demo.html',
  styleUrl: './lifecycle-demo.css',
})
export class LifecycleDemo
  implements
    OnChanges,
    OnInit,
    DoCheck,
    AfterContentInit,
    AfterContentChecked,
    AfterViewInit,
    AfterViewChecked,
    OnDestroy
{
  readonly label = input.required<string>();

  constructor() {
    this.log('constructor');

    inject(DestroyRef).onDestroy(() => {
      this.log('DestroyRef.onDestroy');
    });

    afterNextRender({
      read: () => this.log('afterNextRender (browser only)'),
    });
  }

  ngOnChanges(changes: SimpleChanges<LifecycleDemo>): void {
    const change = changes.label;

    if (change) {
      const previous = change.firstChange
        ? '(no previous value)'
        : String(change.previousValue);

      this.log(`ngOnChanges: ${previous} → ${change.currentValue}`);
    }
  }

  ngOnInit(): void {
    this.log(`ngOnInit: initial label = ${this.label()}`);
  }

  ngDoCheck(): void {
    this.log('ngDoCheck');
  }

  ngAfterContentInit(): void {
    this.log('ngAfterContentInit');
  }

  ngAfterContentChecked(): void {
    this.log('ngAfterContentChecked');
  }

  ngAfterViewInit(): void {
    this.log('ngAfterViewInit');
  }

  ngAfterViewChecked(): void {
    this.log('ngAfterViewChecked');
  }

  ngOnDestroy(): void {
    this.log('ngOnDestroy');
  }

  private log(message: string): void {
    console.log(`[LifecycleDemo] ${message}`);
  }
}
~~~

~~~html
<!-- src/app/lifecycle-demo.html -->
<section class="card">
  <h2>Child component</h2>
  <p>
    Current input: <strong>{{ label() }}</strong>
  </p>

  <div class="projected-content">
    <ng-content></ng-content>
  </div>
</section>
~~~

~~~css
/* src/app/lifecycle-demo.css */
.card {
  padding: 1.25rem;
  border: 1px solid #d8dce7;
  border-radius: 0.75rem;
  background: white;
  box-shadow: 0 8px 24px rgb(31 41 55 / 7%);
}

h2 {
  margin-top: 0;
}

.projected-content {
  margin-top: 1rem;
  padding: 0.75rem 1rem;
  border-left: 3px solid #8b5cf6;
  background: #f5f3ff;
}

.projected-content p {
  margin: 0;
}
~~~

## 4. Follow the lifecycle

1. Start the Angular app and open the browser console. The child logs `constructor`, its first `ngOnChanges`, `ngOnInit`, and the initial content and view hooks. `afterNextRender` logs after Angular completes the next browser render.
2. Select **Change input**. The parent updates its signal, which changes the child's input. The child logs another `ngOnChanges`; `ngOnInit` does not run again. Check hooks may log as Angular checks the view.
3. Select **Destroy child**. The `@if` removes the child. Angular calls `ngOnDestroy` and runs the callback registered with `DestroyRef` as part of destruction. Do not rely on an ordering between these two cleanup logs.
4. Select **Create child**. Angular creates a new instance, so the constructor and initialization hooks run again with the latest input value.

The initial hook sequence in this child is approximately:

~~~text
constructor
ngOnChanges: (no previous value) → Initial input
ngOnInit: initial label = Initial input
ngDoCheck
ngAfterContentInit
ngAfterContentChecked
ngAfterViewInit
ngAfterViewChecked
afterNextRender (browser only)
~~~

Angular can perform additional checks, so the checked hooks may appear more than once. The render callback is browser-only; it does not run during server-side rendering or prerendering. This example only logs from that callback and does not directly manipulate the DOM.

## What this example demonstrates

- Initial inputs are ready by `ngOnChanges` and `ngOnInit`, not the constructor.
- The first `ngOnChanges` runs before `ngOnInit`.
- `ngOnInit` runs once for each component instance, while input changes are reported through `ngOnChanges`.
- Content hooks correspond to the projected `<ng-content>` and view hooks to the component's own template.
- `afterNextRender` runs after a completed browser render.
- `ngOnDestroy` and `DestroyRef` provide destruction cleanup points.
- `ngDoCheck` and checked hooks can run often, so production logic should not do expensive work there.

## Related material

- [Lesson 31: Lifecycle hooks and component initialization](../lessons/031-lifecycle-hooks-and-component-initialization.md)
- [Q&A 31: Lifecycle hooks and component initialization](../qnas/031-lifecycle-hooks-and-component-initialization.md)
- [Angular component lifecycle guide](https://angular.dev/guide/components/lifecycle)
- [`afterNextRender` API](https://angular.dev/api/core/afterNextRender)
- [`DestroyRef` API](https://angular.dev/api/core/DestroyRef)
