# 33. Q&A: Component styles and view encapsulation

## Component styles

### 1. Where can I define styles for an Angular component?

Use `styles` for short inline CSS, or `styleUrl` for a separate stylesheet. `styleUrls` can reference multiple files. Component stylesheet paths are relative to the component source file. Application-wide styles usually belong in the global stylesheet configured by the Angular CLI project.

### 2. Does Angular include a component's stylesheet when the component is lazy-loaded?

Yes. Angular compiles component styles with the component and includes them when the component is rendered, including when a lazy-loaded component is rendered.

### 3. Can I use Sass or Less in a component stylesheet?

Yes, when the Angular project's build pipeline is configured for that preprocessor. Angular supports tools that compile to CSS, including Sass, Less, and Stylus.

### 4. Should I put a global reset in every component stylesheet?

No. Resets, base typography, and application-wide tokens belong in the configured global stylesheet. Repeating them in component styles duplicates rules and can cause unexpected cascade behavior.

## Emulated encapsulation

### 5. What is Angular's default view encapsulation mode?

`ViewEncapsulation.Emulated`. Angular adds generated attributes to the component host and template elements, then rewrites the component's selectors to include those attributes. This scopes the component's CSS without creating a native Shadow Root.

### 6. Does `Emulated` mean styles are completely isolated from global CSS?

No. The component's own selectors are scoped so they do not normally select elements outside its template, but global selectors can still match elements inside it. Inherited values and CSS custom properties also follow normal CSS inheritance.

### 7. Is `Emulated` a security boundary?

No. It is a style-scoping mechanism. It does not sandbox HTML, JavaScript, or untrusted content, and it must not be treated as an application security feature.

### 8. Does `Emulated` use browser Shadow DOM?

No. It emulates style scoping by rewriting selectors and adding generated attributes. Use `ViewEncapsulation.ShadowDom` when a real browser Shadow Root is required.

### 9. Can styles outside a component override its emulated styles?

They can conflict. Angular does not guarantee that component styles always override outside styles when specificity is equal. The normal CSS cascade, specificity, source order, and `!important` still matter.

## Host selectors and legacy selectors

### 10. What does `:host` select?

`:host` selects the component's own host element, such as `<app-card>`. It is useful for host layout, display, border, and state-specific host styling. A custom host is inline by default unless its styles change the display behavior.

### 11. What does `:host(.compact)` mean?

It selects the host only when the host has the `compact` class. For example, a component can use `:host(.compact) { padding: 0.5rem; }` when that class is part of its documented styling API.

### 12. Is `:host-context()` still supported?

Angular's emulated compiler supports `:host-context()`, but the pseudo-class is deprecated in modern browsers. For new theme designs, prefer an explicit host class or an inherited CSS custom property.

### 13. What does `::ng-deep` do?

In emulated mode, it stops Angular's selector rewriting after that point, so the rest of the selector can match content outside the component's normal style boundary. Depending on its position, it can act globally or reach projected and child-component content.

### 14. Should I use `::ng-deep` in new code?

No. Angular strongly discourages new use and retains it for backwards compatibility. Prefer a public styling hook, a CSS custom property, a documented class or input, or a global theme rule when that is the intended contract.

### 15. Can I use `::part` or `::shadow` with Angular emulated styles?

Angular's emulated mode does not implement the other Shadow DOM pseudo-elements such as `::part` or `::shadow`. It supports `:host` and its compiler-specific support for `:host-context()` instead.

## `None` and Shadow DOM modes

### 16. What does `ViewEncapsulation.None` do?

It disables style encapsulation for that component. Its stylesheet behaves as global CSS and can style matching elements anywhere in the application. Use it only when global reach is intentional.

### 17. Why can a component with `None` unexpectedly change another page element?

Because its selectors are global. A generic selector such as `.title` or `h2` can match unrelated markup after the component is rendered. Namespace selectors or move intentionally global rules into a clearly owned global stylesheet.

### 18. What does `ViewEncapsulation.ShadowDom` do?

Angular attaches a native Shadow Root to the component host and renders the component's template and styles within the shadow tree. Component styles cannot select elements outside that tree, and ordinary outside selectors cannot reach its internal elements.

### 19. Do inherited CSS values and custom properties cross a Shadow DOM host?

CSS custom properties and inherited properties can be passed through the host according to normal CSS inheritance. This is a common way to make a shadow component themeable without allowing outside selectors to style its internals.

### 20. Does Shadow DOM affect anything besides CSS selectors?

Yes. It affects event propagation and retargeting, how content projection interacts with native slots, and what browser developer tools show. Consider these consequences before enabling it.

### 21. What is `ExperimentalIsolatedShadowDom`?

It is an experimental Angular encapsulation mode that behaves like Shadow DOM and additionally prevents external styles from leaking into the shadow root. The API is explicitly marked experimental, so verify the Angular version and compatibility needs before using it.

### 22. Which mode should I normally choose?

Start with the default `Emulated`, which fits most Angular components. Choose `None` only when styles should be global, and `ShadowDom` when native Shadow DOM semantics are an intentional part of the component. Treat `ExperimentalIsolatedShadowDom` as experimental rather than a default choice.

## Files and external styles

### 23. Are styles in a `<style>` element inside a component template encapsulated?

Angular applies the component's encapsulation mode to styles declared in a template `<style>` element. Angular does not support template bindings inside that style element; use component metadata and template bindings for dynamic values.

### 24. Does emulated encapsulation scope styles loaded with `<link>` or `@import`?

No. Angular treats those as external styles and does not rewrite them for emulated encapsulation. Review them as potentially global CSS.

### 25. Do styles inside a child component's stylesheet normally select its parent's elements?

Not in `Emulated` mode. A component stylesheet is scoped to that component's own template. Use a public styling interface instead of reaching into another component's private markup.

### 26. Can a parent component's global styles affect a child component?

Yes. Global selectors can match child component elements, including elements inside an emulated component template. This is why global styles need deliberate selectors and why encapsulation should not be mistaken for immunity from global CSS.

## CSS custom properties and the cascade

### 27. Why are CSS custom properties useful for component theming?

They provide a public styling input that can inherit through the DOM. A component can read `var(--card-border, #cbd5e1)` while a consumer sets `--card-border` on the host. This avoids selectors that reach into the component's internals.

### 28. Does encapsulation guarantee that a component's CSS always wins?

No. Encapsulation controls where selectors can match; it does not replace CSS specificity and cascade rules. If two rules can match the same element, inspect their origin, specificity, source order, cascade layer, and `!important` use.

### 29. How do I debug a style that appears to leak?

Inspect the element in browser developer tools. Check which rule matched it, where that stylesheet came from, whether the component uses `None`, and whether the rule is global, external, inherited, or a custom property. Check the cascade before adding more selectors.

### 30. Is `ViewEncapsulation.ShadowDom` a good way to fix a specificity conflict?

Usually not. It changes event and slot behavior in addition to CSS boundaries. First clarify which component owns the style and use a documented class, input, or CSS custom property where appropriate.

### 31. Can I style a child component's internal DOM from its parent stylesheet?

With default emulated encapsulation, parent component styles are scoped and should not be used to reach a child's private template. Use the child's documented CSS custom properties, parts, classes, or other supported styling API. Avoid adding `::ng-deep` as a shortcut.

### 32. What should I use for a dynamic visual state?

Use a class or style binding for dynamic state, and keep the actual presentation rules in CSS. For example, bind `[class.is-invalid]` and define `.is-invalid` in the relevant stylesheet. CSS custom properties are useful for dynamic theme values.

### 33. What is the difference between a component stylesheet and `src/styles.css`?

A component stylesheet belongs to one component and follows its encapsulation setting. The project's global stylesheet applies across the application. Put reusable page-wide foundations and resets in the global stylesheet; keep local presentation beside the owning component.

### 34. If a component stylesheet uses `ViewEncapsulation.None`, is the rule inserted at app startup?

Angular associates component styles with the component and includes them when the component is rendered. Once active, `None` means those selectors act globally, so a selector may affect matching elements beyond that component.

## Related material

- [Lesson 33: Component styles and view encapsulation](../lessons/033-component-styles-and-view-encapsulation.md)
- [Example 33: Component styles and view encapsulation](../examples/033-component-styles-and-view-encapsulation.md)
- [Angular styling guide](https://angular.dev/guide/components/styling)
- [`ViewEncapsulation` API](https://angular.dev/api/core/ViewEncapsulation)
- [Component host elements](https://angular.dev/guide/components/host-elements)
