# Reusable UI Components and Design-System Boundaries

Reusable components can improve consistency, but reuse is not a goal by itself. A component is worth sharing when its API expresses a stable concept that several consumers understand. A generic component with many flags and hidden assumptions can be harder to maintain than a few clear, feature-owned components.

## Learning outcomes

After this lesson, you should be able to:

- Design an Angular component around a clear, small public contract.
- Choose between inputs, outputs, content projection, and composition.
- Separate reusable presentation from feature-specific workflow behavior.
- Apply accessibility and native HTML semantics to shared UI.
- Use design tokens and component styles without leaking assumptions.
- Decide when a component belongs in an application feature or a design system.

## 1. Define the user-facing concept first

Start with the behavior and vocabulary consumers need. A notification banner may have a message, a severity, and an optional dismiss action. An order-status badge has domain meaning and may belong to the orders feature. A generic badge can be reusable only if it has a stable visual and behavioral contract across the application.

Ask what problem the component solves, who owns its data and state, what the consumer must configure, which actions it reports, and what it guarantees about semantics, keyboard use, and styling. A component is not reusable merely because its template contains HTML that appears twice.

## 2. Keep the public API small and typed

Inputs provide data or configuration. Outputs report events so the parent can decide what to do. Angular's signal-based input and output APIs are one current way to express this contract:

~~~ts
import { Component, input, output } from '@angular/core';

@Component({
  selector: 'app-notice',
  template: '<section class="notice"><p>{{ message() }}</p><button type="button" (click)="dismissed.emit()">Dismiss</button></section>'
})
export class NoticeComponent {
  readonly message = input.required<string>();
  readonly tone = input<'info' | 'warning' | 'success'>('info');
  readonly dismissed = output<void>();
}
~~~

The parent owns whether the notice remains visible. The child reports dismissal; it should not reach into parent state or make an unrelated server call. Prefer a few meaningful inputs over many boolean flags that create combinations no one has designed.

## 3. Use composition when content belongs to the consumer

Inputs suit simple data. Content projection is useful when a consumer supplies structured content that a visual container should place:

~~~html
<app-card>
  <h2 card-title>Account settings</h2>
  <p>Manage your profile and preferences.</p>
  <a routerLink="/settings">Open settings</a>
</app-card>
~~~

~~~ts
@Component({
  selector: 'app-card',
  template: '<article class="card"><header><ng-content select="[card-title]" /></header><div class="card__body"><ng-content /></div></article>'
})
export class CardComponent {}
~~~

Projected content remains owned by the component that declared it. Projection is not a mechanism for creating arbitrary dynamic component trees. Angular processes ng-content placeholders at compile time; do not conditionally create or destroy them with template control flow. For runtime-selected templates, use an appropriate template-fragment or dynamic-component pattern.

## 4. Prefer native semantics and accessibility

Shared components amplify both good and bad behavior. A button component should use a real button element so keyboard activation, focus behavior, and disabled semantics come from the platform. A disclosure should use a suitable button and expose its expanded state. A status message should be announced when appropriate.

Do not create clickable div elements when a native button or link expresses the action. Do not remove focus indication without a visible replacement. Labels, names, roles, and keyboard behavior are part of a reusable component's contract. Consider long text, missing optional content, keyboard use, zoom, narrow layouts, and assistive-technology-relevant states.

## 5. Keep styles deliberate

Angular component styles normally use component-scoped encapsulation, while global styles and CSS custom properties can provide shared tokens. Use tokens for values the design system intends to standardize, such as spacing, color roles, typography, and focus outlines.

~~~css
:host {
  display: block;
}

.notice {
  padding: var(--space-3);
  border-inline-start: 0.25rem solid var(--color-info);
  background: var(--surface-raised);
}

.notice--warning {
  border-inline-start-color: var(--color-warning);
}
~~~

Avoid depending on internal DOM structure from a consumer stylesheet. If consumers need a supported visual choice, expose an intentional variant or token. Do not expose implementation-specific selectors as stable API.

## 6. Separate shared presentation from feature behavior

A shared button can render a consistent control. It should not know how a particular feature saves a profile, authorizes a payment, or navigates through a business workflow. The feature owns those decisions and passes data or events through the component contract.

~~~text
Feature page owns workflow and state
              |
              v
Shared component receives data and emits intent
              |
              v
Native HTML renders accessible interaction
~~~

Avoid a giant shared component with feature-specific inputs, injected feature services, and conditional behavior. Split distinct concepts or keep them feature-local.

## 7. Reuse, variants, and breaking changes

A public component API includes more than TypeScript property names. It includes content slots, events, styling hooks, keyboard behavior, ARIA semantics, and documented defaults. Changes to those can break consumers even if the component still compiles.

Keep variants intentional and finite. If one variant changes meaning or behavior, consider a separate component. When a library is shared across apps, document migration steps and use release versions that reflect compatibility changes.

## 8. Decide where a component belongs

- Keep a component in a feature when its meaning or behavior is specific to that workflow.
- Keep it in an application shared area when a few parts of one application reuse a stable presentation pattern.
- Put it in a design-system library when multiple independent applications need the same documented contract and ownership.
- Do not generalize a component before its real consumers and use cases are understood.

The Angular component guide explains component structure. See also the guides for content projection and template control flow.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="098-organize-projects-libraries-and-shared-code.md">098 - Organize Projects, Libraries, and Shared Code</a></td><td><a href="../qnas/099-reusable-ui-components-and-design-system-boundaries.md">Q&amp;A</a></td><td><a href="../examples/099-reusable-ui-components-and-design-system-boundaries.md">Example</a></td><td><a href="100-dynamic-components-and-programmatic-rendering.md">100 - Dynamic Components and Programmatic Rendering</a></td>
</tr>
  </tbody>
</table>
