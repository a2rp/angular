# Examples: Property, Attribute, Class, and Style Bindings

This walkthrough builds a small task panel that demonstrates bindings to DOM properties, HTML attributes, CSS classes, and styles. It uses Angular's standalone component format. File names in an existing app may differ; adapt the import and root component names to your workspace.

## 1. Create the component class

Create src/app/binding-demo.ts:

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-binding-demo',
  standalone: true,
  templateUrl: './binding-demo.html',
  styleUrl: './binding-demo.css',
})
export class BindingDemo {
  readonly taskTitle = 'Prepare the release notes';
  readonly isSaving = true;

  readonly status = 'In progress';
  readonly descriptionId = 'task-status-help';
  readonly optionalReference: string | null = null;

  readonly isUrgent = true;

  classNames: Record<string, boolean> = {
    'task-status': true,
    'task-status--complete': false,
  };

  readonly panelWidth = 480;
  readonly accentColor = '#9a3412';

  panelStyles: Record<string, string> = {
    padding: '1rem',
    'border-radius': '0.75rem',
  };

  markComplete(): void {
    this.isUrgent = false;
    this.status = 'Complete';
    this.classNames = {
      ...this.classNames,
      'task-status--complete': true,
    };
    this.panelStyles = {
      ...this.panelStyles,
      padding: '1.5rem',
    };
  }
}
~~~

This component has a boolean for a native disabled property, text for an input value, values for attributes, and collections for classes and styles. The markComplete method shows how to replace the class and style objects instead of mutating them.

## 2. Add the template

Create src/app/binding-demo.html:

~~~html
<section
  class="task-card"
  [class.task-card--urgent]="isUrgent"
  [style.width.px]="panelWidth"
  [style]="panelStyles"
  [style.--task-accent]="accentColor"
  [attr.data-status]="status"
  [attr.data-reference]="optionalReference"
  aria-labelledby="task-title"
>
  <h2 id="task-title">Task details</h2>

  <label for="task-title-input">Title</label>
  <input
    id="task-title-input"
    type="text"
    [value]="taskTitle"
    [disabled]="isSaving"
    [attr.aria-describedby]="descriptionId"
  />
  <p [id]="descriptionId">
    The title field is disabled while a save is in progress.
  </p>

  <p [class]="classNames">
    Status: {{ status }}
  </p>

  <button type="button" [disabled]="isSaving">
    Save task
  </button>
</section>
~~~

Notice what each binding targets:

- [disabled] updates the native boolean property on the input and button.
- [value] supplies the input's current value as a one-way property binding.
- attr.aria-describedby and attr.data-status explicitly set HTML attributes.
- optionalReference is null, so Angular removes data-reference from the rendered section.
- [class.task-card--urgent] toggles one class.
- [class] applies the class names enabled in the classNames object.
- [style.width.px] adds px to the numeric width.
- [style] applies the style object, and the custom property binding provides --task-accent.

The status text is also interpolated. Interpolation creates text; it does not select a DOM property or attribute.

## 3. Add component styles

Create src/app/binding-demo.css:

~~~css
:host {
  display: block;
  max-width: 100%;
}

.task-card {
  box-sizing: border-box;
  color: #292524;
  background: #fffaf5;
}

.task-card--urgent {
  border-inline-start: 0.35rem solid var(--task-accent);
}

.task-status {
  display: inline-block;
  margin-block: 1rem;
}

.task-status--complete {
  color: #166534;
  font-weight: 700;
}

label {
  display: block;
  margin-block-end: 0.35rem;
  font-weight: 600;
}

input {
  box-sizing: border-box;
  width: 100%;
  padding: 0.6rem;
  font: inherit;
}

button {
  padding: 0.6rem 0.9rem;
  font: inherit;
}

button:disabled,
input:disabled {
  cursor: not-allowed;
  opacity: 0.65;
}
~~~

The CSS class gives the task card its general appearance. The class binding adds the urgent accent, while the custom property supplies its color.

## 4. Render the component

Import BindingDemo into the root component and include its selector in the root template. For example, if the root component is src/app/app.ts:

~~~ts
import { Component } from '@angular/core';
import { BindingDemo } from './binding-demo';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [BindingDemo],
  template: '<app-binding-demo />',
})
export class App {}
~~~

If the root component already has imports or a separate template, add BindingDemo to its imports and place app-binding-demo in that template. Keep the app's existing providers and other imports.

Start the Angular development server:

~~~sh
ng serve
~~~

The input and button start disabled because isSaving is true. The section has data-status="In progress"; data-reference is absent because its bound value is null.

## 5. Inspect property and attribute behavior

Open the browser's developer tools and inspect the rendered input and button:

- Check the live disabled property. It is true.
- Inspect the input's value property. It contains the task title.
- Check the aria-describedby attribute. It points to the help text.
- Check data-reference on the section. It is absent because optionalReference is null.

A present boolean attribute is not the same thing as a string attribute whose value is false. For a native disabled state, [disabled] is the intended binding.

## 6. Try replacing the class and style collections

From TypeScript code, replace the existing values with new objects:

~~~ts
this.markComplete();
~~~

The method changes the status, removes the urgent class, enables the completion class, and adjusts the padding. The example intentionally uses object spreads so Angular sees new class and style object references.

This method is not wired to a button in this chapter's example because event bindings are the next topic. It can be called from a debugger, a temporary test harness, or after studying the event-binding lesson.

## 7. Practice variations

1. Set isSaving to false and confirm that the input and button become enabled.
2. Set optionalReference to a non-null string and inspect the resulting data-reference attribute.
3. Change isUrgent to false and observe the urgent border disappear.
4. Change panelWidth and inspect the computed width.
5. Add a CSS class key to classNames and toggle its boolean value.
6. Add a property to panelStyles and replace the object rather than mutating it.
7. Replace the data-status binding with an ARIA state only when the value and semantics match the relevant control.
8. Compare [disabled]="false" with [attr.disabled]="false" on a temporary native button, then remove the experiment. The property binding enables the button; the attribute form still emits a present boolean attribute.

Do not insert untrusted strings as HTML or bypass Angular's security checks to make arbitrary values render. Prefer text bindings and fixed CSS classes for user-facing content and design states.
