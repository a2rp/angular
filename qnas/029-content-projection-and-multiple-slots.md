# 29. Q&A: Content projection and multiple slots

## 1. What is content projection?

Content projection lets a component render markup supplied by its consumer. The receiving component places `<ng-content>` placeholders in its template, and Angular renders the consumer's matching content at those locations.

## 2. What does `<ng-content>` do?

It is a compile-time placeholder that tells Angular where projected content should appear. It is not a component or runtime DOM element.

## 3. What is the difference between a component's view and its content?

The view is the markup declared in that component's own template. Its content is the markup supplied between its component tags by the consumer. `<ng-content>` marks where some of that consumer content is rendered.

## 4. How do I create a single default slot?

Put an unqualified `<ng-content />` in the receiving component's template. It receives the projected content that is not assigned to a named slot.

## 5. How do I create named slots?

Use the `select` attribute with a CSS selector:

~~~html
<ng-content select="[card-title]" />
<ng-content select="[card-actions]" />
<ng-content />
~~~

The consumer marks the intended content with matching attributes, classes, or element names.

## 6. What selector forms can `select` use?

Angular supports selectors using tag names, attributes, CSS classes, and the `:not()` pseudo-class. Selectors can combine these forms. See the [`ng-content` API](https://angular.dev/api/core/ng-content) for the supported selector details.

## 7. How does the consumer mark content for a slot?

Put the matching marker on the element supplied inside the component:

~~~html
<app-card>
  <h2 card-title>Account</h2>
  <p>Card body text.</p>
  <div card-actions>Actions go here.</div>
</app-card>
~~~

The `[card-title]` and `[card-actions]` slots receive their marked nodes; the unqualified slot receives the remaining content.

## 8. What happens to content that matches no named slot?

If the component has an unqualified `<ng-content>`, unmatched projected content goes there. If it has only named slots, content that matches none of them is not rendered into that component's DOM.

## 9. Does a named slot automatically include all unmarked content?

No. A named slot receives content matching its `select` selector. Use an unqualified `<ng-content>` when remaining content needs a default location.

## 10. What is fallback content?

Fallback content is markup placed inside an `<ng-content>` placeholder. Angular shows it when no projected content matches that slot:

~~~html
<ng-content select="[card-title]">Untitled card</ng-content>
~~~

If the consumer supplies no matching title, `Untitled card` is rendered.

## 11. Does fallback content show when the consumer provides matching content?

No. When content matches the slot, Angular projects the supplied content and uses that instead of the fallback.

## 12. Can I use multiple unqualified `<ng-content>` slots?

Design a clear slot structure instead of relying on duplicate catch-all placeholders. Use selectors for distinct regions, and one unqualified slot for content that should go into the default region.

## 13. What is `ngProjectAs`?

It is a special static attribute that tells Angular to match an element against a specified selector for projection, even if the element's real tag or attributes do not match:

~~~html
<h2 ngProjectAs="[card-title]">Account</h2>
~~~

This element can be projected into `<ng-content select="[card-title]">`.

## 14. Can `ngProjectAs` be dynamic?

No. `ngProjectAs` supports static values and cannot be bound to a dynamic expression. Use normal marker attributes when you control the consumer markup.

## 15. Can I put `@if` around `<ng-content>` in the receiving component?

Do not conditionally include the `<ng-content>` placeholder with `@if`, `@for`, or `@switch`. Angular processes projection placeholders at compile time and can instantiate the projected nodes even if the placeholder is hidden. Use a template fragment for content the receiving component must instantiate conditionally.

## 16. Can the consumer conditionally supply projected markup?

The receiving component should keep its projection placeholders unconditional. The consumer can decide what markup to provide in its own template. If the receiver itself must decide when or how often to render caller-provided content, accept a template fragment instead.

## 17. How do I dynamically render caller-provided markup?

Use an `<ng-template>` fragment and render it with `NgTemplateOutlet` or `ViewContainerRef`. These APIs are designed for dynamic instantiation; `<ng-content>` is designed for fixed projection locations.

## 18. Is `<ng-content>` a real DOM element that I can style?

No. It is a compiler placeholder, not a runtime DOM element. Style a real wrapper around it, or style the actual projected markup using an approach appropriate to the component's styling contract.

## 19. Can I add a directive or arbitrary attribute to `<ng-content>`?

No. Angular's guide says the placeholder cannot receive directives, styles, or arbitrary attributes. Apply these to a real wrapper or to consumer-provided content as appropriate.

## 20. Which component owns projected markup?

The component that declared the projected markup owns it. It remains part of that component's view even though Angular renders it inside the receiving component's template.

## 21. In which component context do projected bindings run?

Bindings and event handlers in projected content are evaluated in the context of the component that declared the content. For example, a click handler on a projected button calls the declaring component's method.

## 22. Which injector does projected content use?

Projected content uses the injector of the component that declared it, not the receiving component's `viewProviders`.

## 23. How does `OnPush` affect projected content?

Projected content belongs to the declaring component's view and is checked as part of that parent's view. Do not assume that placing expensive projected content inside an `OnPush` wrapper automatically makes the parent's projected content skip checking.

## 24. Is content projection the same as passing an input?

No. Use an input to pass data such as a title string, status, or object. Use projection when the consumer supplies markup to appear in a fixed region of a wrapper or layout component.

## 25. Is content projection the same as a template fragment?

No. Projection places supplied content into statically declared slots. A template fragment such as `<ng-template>` can be instantiated conditionally, repeatedly, or at a location selected by the receiving component.

## 26. Can projected content contain Angular bindings and components?

Yes. The consumer can provide Angular markup, including bindings and components, subject to the receiving component's projection slots and any library-specific requirements. The markup continues to be owned by the declaring view.

## 27. Does a component always support arbitrary projected children?

No. A library component may query and manage its children for behaviors such as keyboard navigation, focus management, or accessibility. Check its documentation before projecting custom wrappers or children into it.

## 28. Can `ng-content` project content through nested wrappers?

Projection is based on Angular's matching of the content supplied at the component boundary. Put slot markers on the actual nodes intended for projection; do not expect a selector to search arbitrarily through a nested subtree.

## 29. When should I use fallback content instead of an input default?

Use fallback content for small markup defaults, such as a short placeholder title. Use an input when the default is data that should be configured, validated, transformed, or read as part of the component's explicit data API.

## 30. Can I use projection to make an entire custom card component?

Yes. A card is a common use case: the card defines a shell and slots for title, body, and actions, while each consumer supplies the appropriate content.

## 31. What is a reliable slot API design?

Use stable, purpose-specific selectors such as `[card-title]` and `[card-actions]`, document whether unmatched content has a default slot, and provide fallbacks for optional regions when useful.

## 32. What is the most common projection bug?

A consumer's content appears to vanish because it matches no named slot and the receiver has no unqualified default slot. Check the selectors and add a default slot if unmatched content should render.

## 33. What is the key difference between projection and a web component `<slot>`?

They are similar composition ideas, but Angular `<ng-content>` is Angular's compile-time content projection mechanism with Angular-specific matching and ownership rules. It is not the native Web Components slot element.

## 34. Where can I read Angular's official documentation?

See [Content projection with `ng-content`](https://angular.dev/guide/components/content-projection), the [`ng-content` API reference](https://angular.dev/api/core/ng-content), and [Create template fragments with `ng-template`](https://angular.dev/guide/templates/ng-template).
