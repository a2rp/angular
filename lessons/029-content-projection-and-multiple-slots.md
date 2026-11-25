# 29. Content projection and multiple slots

Content projection lets a component provide a wrapper or layout while letting its consumer supply part of the markup. The component's template places `<ng-content>` where that caller-provided content should appear.

~~~ts
@Component({
  selector: 'app-card',
  standalone: true,
  template: `
    <article class="card">
      <ng-content />
    </article>
  `,
})
export class Card {}
~~~

A consumer can then provide content inside the component element:

~~~html
<app-card>
  <h2>Account</h2>
  <p>Manage your profile and preferences.</p>
</app-card>
~~~

The `h2` and `p` are projected into the card's `<ng-content>` placeholder. This is useful when a reusable component controls structure and styling, while each use supplies its own content.

## Learning goals

After this lesson, you should be able to:

- Explain the difference between a component's view and its projected content.
- Use `<ng-content>` as a single content slot.
- Create named slots with the `select` attribute.
- Add fallback content and handle unmatched content deliberately.
- Use `ngProjectAs` to match a slot selector with static markup.
- Know when content projection is not a fit for dynamic or conditional templates.

## View content versus projected content

A component's **view** is the markup declared in its own template. Its **content** is the markup supplied between its component tags by the component's consumer. `<ng-content>` is a compile-time placeholder that tells Angular where to render that supplied content.

~~~html
<app-card>
  <!-- The consumer declares this content. -->
  <p>This paragraph is projected.</p>
</app-card>
~~~

The paragraph appears in the rendered location of the card's `<ng-content>`, but it remains part of the view of the component that declared it. Bindings and event handlers in projected markup are evaluated in the declaring component's context. The receiving component provides a place for the content; it does not take ownership of the markup's expressions.

`<ng-content>` is not a runtime component or a DOM element. Angular processes it at build time. You cannot attach directives, styles, or arbitrary attributes to the placeholder itself.

## Use one slot

A component with an unqualified `<ng-content>` has a default slot:

~~~ts
@Component({
  selector: 'app-panel',
  standalone: true,
  template: `
    <section class="panel">
      <ng-content />
    </section>
  `,
})
export class Panel {}
~~~

The consumer can provide any suitable child content:

~~~html
<app-panel>
  <p>Panel content comes from the consumer.</p>
  <a href="/settings">Open settings</a>
</app-panel>
~~~

Use a default slot when the component does not need to distinguish between different parts of the supplied markup.

## Create named slots with `select`

Use the `select` attribute when a component needs distinct areas, such as a heading, body, and actions:

~~~html
<article class="card">
  <header class="card-header">
    <ng-content select="[card-title]">Untitled card</ng-content>
  </header>

  <div class="card-body">
    <ng-content />
  </div>

  <footer class="card-actions">
    <ng-content select="[card-actions]" />
  </footer>
</article>
~~~

A consumer marks content for those slots with matching attributes:

~~~html
<app-info-card>
  <h2 card-title>Storage</h2>

  <p>You have used 4 GB of your 10 GB plan.</p>

  <div card-actions>
    <button type="button" (click)="upgradePlan()">Upgrade plan</button>
  </div>
</app-info-card>
~~~

Angular matches supplied elements against the slot's CSS selector. Slot selectors can use tag names, attributes, classes, and the `:not()` pseudo-class. For example:

~~~html
<ng-content select="h2" />
<ng-content select=".card-actions" />
<ng-content select="[card-title]" />
<ng-content select=":not([card-title])" />
~~~

Prefer stable, purpose-specific attributes such as `[card-title]` when a slot is part of your component's public API. They make intent clear and avoid tying the slot to a particular HTML tag or styling class.

## Default slots and unmatched content

When a component has named slots and one unqualified `<ng-content>`, the unqualified slot receives projected elements that did not match a named slot. In the card above, the paragraph goes into the default body slot, while the title and actions go to their named slots.

If a component declares only named slots and no default slot, content that matches none of them is not rendered into the component's DOM. Decide whether that is intended. Add an unqualified slot if other content should have a place.

Projection selectors describe the supplied nodes Angular is matching. Do not assume a selector searches arbitrarily through a nested subtree; put the slot marker on the node intended for projection.

## Provide fallback content

Content between `<ng-content>` and `</ng-content>` is fallback content. Angular renders it when no supplied content matches that slot:

~~~html
<header>
  <ng-content select="[card-title]">Untitled card</ng-content>
</header>
~~~

If the consumer provides `<h2 card-title>Storage</h2>`, that title is projected. If it provides no matching title, the fallback text `Untitled card` appears.

Fallback content is useful for concise defaults. For a substantial or configurable default, an input can be clearer because it exposes the value directly in the component API.

## Match alternate markup with `ngProjectAs`

Sometimes the consumer cannot add the attribute or tag your slot selector expects. Add the static `ngProjectAs` attribute to tell Angular which selector the element should match:

~~~html
<ng-content select="[card-title]" />
<ng-content />
~~~

~~~html
<app-info-card>
  <h2 ngProjectAs="[card-title]">Storage</h2>
  <p>Plan details go in the default slot.</p>
</app-info-card>
~~~

The `h2` is matched as though it had the `[card-title]` selector for projection. `ngProjectAs` accepts a static value; it cannot be bound to a dynamic expression. Prefer a real marker attribute when you control the consumer markup.

## Projection does not dynamically create slots

Angular compiles all `<ng-content>` placeholders. Do not put an `<ng-content>` inside `@if`, `@for`, or `@switch` to try to create or remove a slot at runtime. Angular can still instantiate the projected nodes even when that placeholder is hidden, and `<ng-content>` is not a dynamic insertion API.

For dynamic or conditional reusable markup, use a template fragment such as `<ng-template>` and render it with `NgTemplateOutlet` or `ViewContainerRef`. Use projection when the consumer supplies actual content that belongs in a fixed slot of the receiving component.

## Important ownership behavior

Projected content continues to belong to the component that declared it:

- Bindings in the projected markup use the declaring component's context.
- The projected content uses the declaring component's injector, not the receiver's `viewProviders`.
- The parent view is responsible for checking projected content during change detection.

This is one reason projection works naturally for caller-authored markup. It is also a reason to consult a library component's documentation before projecting into components that manage their own children, such as menus, tabs, or lists. Those components may rely on queries and internal setup for keyboard interaction or accessibility.

## Choose the right content API

| Need | Common choice |
| --- | --- |
| Consumer supplies markup for a fixed area in a wrapper | `<ng-content>` |
| Consumer supplies distinct heading, body, or action areas | Multiple `<ng-content select="...">` slots |
| Component needs a value such as a label, title, or status | Input |
| Component needs to show a caller-provided template conditionally or repeatedly | `<ng-template>` / template fragments |

Projection is a layout and composition mechanism. Use inputs for data, outputs for events, and template fragments when the receiving component must decide when or how often to instantiate caller-provided markup.

## Common mistakes

- **Confusing content with the component view:** projected markup is declared and evaluated by the consumer.
- **Forgetting `select` markers:** named slots only receive content that matches their selectors.
- **Assuming unmatched content appears automatically:** without an unqualified slot, unmatched content is not rendered.
- **Conditionally wrapping `<ng-content>`:** projection placeholders are compiled up front; use template fragments for dynamic rendering.
- **Putting `ngProjectAs` on a dynamic binding:** it supports static values only.
- **Treating `<ng-content>` as an element you can style or add directives to:** it is a compiler placeholder, not a DOM node.
- **Using content projection with a library component without checking its contract:** projected children may not work with that component's queries or accessibility behavior.
- **Using projection to pass simple data:** expose data as an input instead.

## Practical checklist

When designing a component that accepts content, confirm that:

- A fixed `<ng-content>` placeholder is the right composition mechanism.
- Each named slot has a clear, stable selector.
- You have decided where unmatched content should go.
- Useful fallback content is provided for optional slots.
- `ngProjectAs` is static and only used when normal slot markers are impractical.
- The component does not try to conditionally instantiate `<ng-content>`.
- A dynamic template fragment would not fit the requirement better.
- Any library component's projected-child requirements are understood.

## Further reading

- [Content projection with `ng-content`](https://angular.dev/guide/components/content-projection)
- [`ng-content` API reference](https://angular.dev/api/core/ng-content)
- [Create template fragments with `ng-template`](https://angular.dev/guide/templates/ng-template)
