# Examples: Built-in directives and structural directive syntax

This practice component uses built-in `NgClass` and `NgStyle` attribute directives, Angular's current `@if` and `@for` template blocks, and a small custom structural directive. The custom directive makes the asterisk shorthand concrete: `*appShowWhen="showHints"` is shorthand for an input binding on an implicit `<ng-template>`.

The custom directive is included to demonstrate how structural syntax works. For ordinary conditional and repeated content in new Angular templates, use `@if` and `@for`.

## 1. Create a small structural directive

Create `src/app/show-when.directive.ts`:

~~~ts
import {
  Directive,
  Input,
  OnChanges,
  TemplateRef,
  ViewContainerRef,
  inject,
} from '@angular/core';

@Directive({
  selector: '[appShowWhen]',
  standalone: true,
})
export class ShowWhenDirective implements OnChanges {
  @Input() appShowWhen = false;

  private readonly templateRef = inject(TemplateRef<unknown>);
  private readonly viewContainer = inject(ViewContainerRef);
  private hasView = false;

  ngOnChanges(): void {
    if (this.appShowWhen && !this.hasView) {
      this.viewContainer.createEmbeddedView(this.templateRef);
      this.hasView = true;
      return;
    }

    if (!this.appShowWhen && this.hasView) {
      this.viewContainer.clear();
      this.hasView = false;
    }
  }
}
~~~

The directive receives the `TemplateRef` for the content and the `ViewContainerRef` where Angular should render it. It creates one embedded view when the input becomes true and clears that view when the input becomes false.

## 2. Create the standalone component

Create `src/app/directive-playground.ts`:

~~~ts
import { Component } from '@angular/core';
import { NgClass, NgStyle } from '@angular/common';
import { ShowWhenDirective } from './show-when.directive';

interface PracticeTask {
  id: number;
  title: string;
  priority: 'normal' | 'high';
  completed: boolean;
  accent: string;
}

@Component({
  selector: 'app-directive-playground',
  standalone: true,
  imports: [NgClass, NgStyle, ShowWhenDirective],
  templateUrl: './directive-playground.html',
  styleUrl: './directive-playground.css',
})
export class DirectivePlayground {
  showHints = true;

  tasks: PracticeTask[] = [
    {
      id: 1,
      title: 'Read the directive lesson',
      priority: 'high',
      completed: false,
      accent: '#b42318',
    },
    {
      id: 2,
      title: 'Try the template shorthand',
      priority: 'normal',
      completed: true,
      accent: '#23734d',
    },
  ];

  toggleTask(taskId: number): void {
    this.tasks = this.tasks.map((task) =>
      task.id === taskId
        ? { ...task, completed: !task.completed }
        : task
    );
  }
}
~~~

`NgClass`, `NgStyle`, and `ShowWhenDirective` are explicit template dependencies of this standalone component. The built-in `@if` and `@for` blocks need no imports.

## 3. Add the template

Create `src/app/directive-playground.html`:

~~~html
<main class="playground">
  <h1>Directive practice</h1>

  <p class="intro">
    Change a task to see its classes and inline styles respond to its state.
  </p>

  <button
    type="button"
    class="hint-toggle"
    [attr.aria-expanded]="showHints"
    (click)="showHints = !showHints">
    {{ showHints ? 'Hide' : 'Show' }} syntax hint
  </button>

  <ng-container *appShowWhen="showHints">
    <aside class="syntax-hint" aria-label="Structural syntax hint">
      <p><strong>Custom shorthand:</strong> <code>*appShowWhen="showHints"</code></p>
      <p>
        Angular expands it to an <code>&lt;ng-template&gt;</code> carrying
        <code>[appShowWhen]="showHints"</code>.
      </p>
    </aside>
  </ng-container>

  @if (tasks.length > 0) {
    <ul class="task-list" aria-label="Practice tasks">
      @for (task of tasks; track task.id) {
        <li
          class="task-card"
          [ngClass]="{
            'task-card--complete': task.completed,
            'task-card--urgent': task.priority === 'high'
          }"
          [ngStyle]="{
            'border-left-color': task.accent,
            'font-weight': task.completed ? '400' : '600'
          }">
          <div>
            <h2>{{ task.title }}</h2>
            <p>
              Priority: {{ task.priority }} ·
              {{ task.completed ? 'Complete' : 'In progress' }}
            </p>
          </div>

          <button
            type="button"
            [attr.aria-pressed]="task.completed"
            (click)="toggleTask(task.id)">
            {{ task.completed ? 'Mark in progress' : 'Mark complete' }}
          </button>
        </li>
      }
    </ul>
  } @else {
    <p class="empty-state">There are no practice tasks.</p>
  }
</main>
~~~

The syntax hint is controlled by the custom structural directive. Each task uses `NgClass` for conditional classes and `NgStyle` for dynamic inline styles. The list and the outer conditional use current built-in control-flow blocks.

## 4. Add the component styles

Create `src/app/directive-playground.css`:

~~~css
:host {
  display: block;
  color: #172033;
  font-family: system-ui, sans-serif;
}

.playground {
  display: grid;
  gap: 1rem;
  width: min(calc(100% - 2rem), 48rem);
  margin: 2rem auto;
}

h1,
h2,
p {
  margin: 0;
}

.intro,
.task-card p {
  color: #4b5565;
}

.hint-toggle,
.task-card button {
  width: fit-content;
  min-height: 2.5rem;
  padding: 0.55rem 0.85rem;
  border: 1px solid #9aa7ba;
  border-radius: 0.4rem;
  background: white;
  color: #172033;
  font: inherit;
  cursor: pointer;
}

.hint-toggle:focus-visible,
.task-card button:focus-visible {
  outline: 3px solid #3155a4;
  outline-offset: 2px;
}

.syntax-hint {
  padding: 1rem;
  border-left: 0.25rem solid #3155a4;
  border-radius: 0.35rem;
  background: #eef3ff;
}

.syntax-hint code {
  font-family: ui-monospace, monospace;
}

.task-list {
  display: grid;
  gap: 0.75rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.task-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  padding: 1rem;
  border: 1px solid #cbd3df;
  border-left: 0.35rem solid;
  border-radius: 0.5rem;
  background: #fff;
}

.task-card--complete h2 {
  text-decoration: line-through;
  text-decoration-thickness: 0.08em;
}

.task-card--urgent {
  background: #fff8f6;
}

.empty-state {
  padding: 1rem;
  border: 1px dashed #9aa7ba;
  border-radius: 0.5rem;
}

@media (max-width: 34rem) {
  .task-card {
    align-items: flex-start;
    flex-direction: column;
  }
}
~~~

## 5. Render the component from the root

Import `DirectivePlayground` into the standalone root component and add its selector to the root template. For example:

~~~ts
import { Component } from '@angular/core';
import { DirectivePlayground } from './directive-playground';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [DirectivePlayground],
  template: '<app-directive-playground />',
})
export class App {}
~~~

If the generated root component already imports other components, add `DirectivePlayground` to its existing `imports` array. If its template is in a separate HTML file, place `<app-directive-playground />` in that file.

## 6. Run and explore

From the Angular application directory, run:

~~~sh
ng serve
~~~

Open the local address printed by the CLI.

- Use the syntax-hint button to create or remove the custom directive's embedded view.
- Mark a task complete or in progress to see `NgClass` add or remove classes.
- Observe how `NgStyle` changes the left border color and font weight.
- Inspect the template and identify which parts use directives and which use built-in control-flow blocks.

## 7. Compare shorthand with its expanded template

The hint describes the structural transformation. This shorthand:

~~~html
<p *appShowWhen="showHints">Read the hint.</p>
~~~

is conceptually expanded to:

~~~html
<ng-template [appShowWhen]="showHints">
  <p>Read the hint.</p>
</ng-template>
~~~

The directive is attached to the implicit template, and the paragraph is the content it controls. The directive's input name matches its selector, so the bare expression is bound to `appShowWhen`.

The custom directive handles one structural behavior. Do not combine another asterisk directive on the same element; put it on a nested element or use a control-flow block.

## Expected result

The page initially shows a syntax hint and two tasks. Hiding the hint removes its embedded view. Changing a task updates its classes and styles. The `@if` and `@for` blocks continue to handle the page's built-in conditional and repeated rendering.

## Troubleshooting

- If Angular reports `ngClass` or `ngStyle` as unknown, confirm they are imported in `DirectivePlayground`.
- If `appShowWhen` is unknown, import `ShowWhenDirective` into the standalone component.
- If the hint does not toggle, check the spelling of the input `appShowWhen` and the component property `showHints`.
- If a structural directive's local value is not available, review the directive's template context and its `let-` declaration. This example has no local context values.
- Keep built-in control-flow syntax separate from asterisk directive syntax; the control-flow blocks need no directive import.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="023-template-references-ng-template-and-ngtemplateoutlet.md">023 - 23. Template references, ng-template, and ngTemplateOutlet</a></td><td><a href="../lessons/024-built-in-directives-and-structural-directive-syntax.md">Lesson</a></td><td><a href="../qnas/024-built-in-directives-and-structural-directive-syntax.md">Q&amp;A</a></td><td><a href="025-signal-inputs-required-inputs-and-input-transforms.md">025 - 25. Signal inputs, required inputs, and input transforms</a></td>
</tr>
  </tbody>
</table>
