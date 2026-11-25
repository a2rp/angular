# Examples: Template references, ng-template, and ngTemplateOutlet

This example defines one person-card fragment and renders it twice with different context values. The fragment receives its person through `$implicit` and a heading through a named context key. It also reads a property from the component that declared the fragment, showing the difference between component scope and outlet context.

Use the Angular practice application from the setup chapters. File names for the generated root component can vary between Angular CLI versions.

## 1. Create the standalone component class

Create `src/app/template-gallery.ts`:

~~~ts
import { Component } from '@angular/core';
import { NgTemplateOutlet } from '@angular/common';

interface TeamMember {
  name: string;
  role: string;
  focus: string;
}

@Component({
  selector: 'app-template-gallery',
  standalone: true,
  imports: [NgTemplateOutlet],
  templateUrl: './template-gallery.html',
  styleUrl: './template-gallery.css',
})
export class TemplateGallery {
  readonly pageTitle = 'Product team';

  readonly teamLead: TeamMember = {
    name: 'Mina Patel',
    role: 'Team lead',
    focus: 'Planning and team coordination',
  };

  readonly designer: TeamMember = {
    name: 'Jordan Lee',
    role: 'Product designer',
    focus: 'Research and interface design',
  };
}
~~~

A standalone component imports the directives its template uses. Here, `NgTemplateOutlet` comes from `@angular/common`.

## 2. Declare and render the fragment

Create `src/app/template-gallery.html`:

~~~html
<section class="team-panel" aria-labelledby="team-title">
  <h1 id="team-title">Reusable team cards</h1>

  <p class="intro">
    The same template fragment renders each card with its own context.
  </p>

  <div class="card-list">
    <ng-container
      [ngTemplateOutlet]="memberCard"
      [ngTemplateOutletContext]="{
        $implicit: teamLead,
        heading: 'Team lead'
      }">
    </ng-container>

    <ng-container
      [ngTemplateOutlet]="memberCard"
      [ngTemplateOutletContext]="{
        $implicit: designer,
        heading: 'Design'
      }">
    </ng-container>
  </div>

  <ng-template #memberCard let-member let-heading="heading">
    <article class="member-card">
      <p class="eyebrow">{{ heading }}</p>
      <h2>{{ member.name }}</h2>
      <p class="role">{{ member.role }}</p>
      <p>{{ member.focus }}</p>
      <p class="declared-scope">Declared in: {{ pageTitle }}</p>
    </article>
  </ng-template>
</section>
~~~

The `#memberCard` reference points to the fragment. Each outlet renders it with different context values:

- `let-member` reads the context's `$implicit` value.
- `let-heading="heading"` reads the context's `heading` value.
- `pageTitle` comes from the component where the fragment was declared.

The two `ng-container` elements do not add wrapper elements to the rendered DOM. Each resulting card is an `article`, which gives the repeated content a meaningful semantic structure.

## 3. Add component styles

Create `src/app/template-gallery.css`:

~~~css
:host {
  display: block;
  max-width: 56rem;
  margin: 2rem auto;
  padding: 1.5rem;
  color: #172033;
  font-family: system-ui, sans-serif;
}

.team-panel {
  display: grid;
  gap: 1rem;
}

h1,
h2,
p {
  margin: 0;
}

.intro {
  color: #4b5565;
}

.card-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 16rem), 1fr));
  gap: 1rem;
}

.member-card {
  display: grid;
  align-content: start;
  gap: 0.65rem;
  min-height: 12rem;
  padding: 1.25rem;
  border: 1px solid #cbd3df;
  border-radius: 0.75rem;
  background: #f8fafc;
}

.eyebrow {
  color: #3155a4;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

.role,
.declared-scope {
  color: #4b5565;
}

.declared-scope {
  margin-top: auto;
  padding-top: 0.5rem;
  border-top: 1px solid #dbe1ea;
  font-size: 0.85rem;
}
~~~

## 4. Render the component from the root

Import `TemplateGallery` into the application's standalone root component and add its selector to the root template. For example:

~~~ts
import { Component } from '@angular/core';
import { TemplateGallery } from './template-gallery';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TemplateGallery],
  template: '<app-template-gallery />',
})
export class App {}
~~~

If the generated root component already imports other components, keep those imports and add `TemplateGallery` to the existing `imports` array. If its template is in a separate HTML file, place `<app-template-gallery />` in that file instead.

## 5. Run and inspect

From the Angular application directory, start the development server:

~~~sh
ng serve
~~~

Open the local address printed by the CLI. You should see two cards with different people and headings. Both cards show `Product team` under “Declared in” because the fragment's `pageTitle` expression belongs to the component where the fragment was declared.

## 6. Practice variations

1. Add a third `TeamMember` and render the same `memberCard` fragment again with a different context.
2. Change the local declaration to `let-member="person"` and pass `person: teamLead` in the context. Observe that a named key can be used instead of `$implicit`.
3. Add another named local such as `let-status="status"` and pass a matching `status` context property.
4. Replace one outlet with a semantic wrapper only if that wrapper is meaningful for the page structure.
5. Create a separate component for the card, then compare which version is easier to maintain when the card gains independent behavior or inputs.

## Expected result

The page contains two accessible article cards. Both use the same declared fragment, while their outlet contexts provide different people and headings. The fragment's own binding to `pageTitle` continues to refer to the declaring component.

## Troubleshooting

- If Angular does not recognize `ngTemplateOutlet`, confirm that `NgTemplateOutlet` is imported by `TemplateGallery`.
- If the template reference cannot be resolved, check the spelling of `#memberCard` and the outlet binding.
- If a context local is blank, ensure its context key matches the `let-` declaration. A plain `let-member` reads `$implicit`.
- If the page title is blank, confirm the fragment is declared in `TemplateGallery` and that the component property is named `pageTitle`.
- Keep HTML tags balanced and ensure each template reference is declared once in its template scope.

## Official references

- [Create template fragments with ng-template](https://angular.dev/guide/templates/ng-template)
- [Variables in templates](https://angular.dev/guide/templates/variables)
- [NgTemplateOutlet API](https://angular.dev/api/common/NgTemplateOutlet)
