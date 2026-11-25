# 34. Q&A: Global styles, CSS variables, and theming

## Global and component styles

### 1. What makes a stylesheet global in an Angular app?

A global stylesheet is included through the Angular workspace build configuration, commonly `src/styles.css` in the `styles` option of `angular.json`. Its selectors can match throughout the document. A component stylesheet instead follows that component's view encapsulation setting.

### 2. What belongs in a global stylesheet?

Application-wide foundations such as resets, base typography, broad page defaults, and shared design tokens. Keep component-specific layout and presentation next to the component that owns the template.

### 3. Can a global selector affect elements inside an emulated component?

Yes. `ViewEncapsulation.Emulated` scopes component styles, but it does not stop global CSS selectors from matching the component's elements. It also does not disable normal CSS inheritance.

### 4. How do component style files differ from global styles?

Component styles are attached to a component and are scoped according to `ViewEncapsulation`. Global styles are included across the application. A component set to `ViewEncapsulation.None` contributes styles that behave globally.

### 5. Why should I avoid broad global selectors?

Selectors such as `h2`, `button`, or `.title` can unexpectedly affect unrelated parts of the application, including third-party components. Keep intentional base styles clear and use semantic tokens or namespaced classes for shared patterns.

## CSS custom properties

### 6. What is a CSS custom property?

It is a CSS value defined with a name beginning with `--`, such as `--color-surface`, and read with `var(--color-surface)`. Custom properties are part of runtime CSS and participate in the cascade and inheritance.

### 7. How are CSS custom properties different from Sass variables?

Sass variables are resolved while styles are preprocessed. CSS custom properties remain in the emitted CSS and can take different values at runtime on different elements or under different themes.

### 8. Do custom properties inherit?

Normally, yes. A custom property declared on an ancestor can be read by its descendants unless a closer declaration overrides it. This is why defining theme tokens on `:root` or a themed subtree works.

### 9. How do I provide a fallback value?

Use the second argument to `var()`, for example `color: var(--color-text, #202531)`. The browser uses the fallback when the custom property has no usable value.

### 10. What is a semantic theme token?

A custom property named by its UI role, such as `--color-surface`, `--color-text`, or `--color-action`, rather than by one literal color. Each theme maps those roles to its own values, allowing components to remain theme-agnostic.

### 11. Should a component know whether it is in light or dark mode?

Usually not. A reusable component can consume semantic properties such as `var(--color-surface)` and `var(--color-text)`. An ancestor or theme layer supplies the current values.

### 12. What does `color-scheme` do?

It tells the browser which color scheme the element supports, helping browser-provided controls and surfaces such as form controls use an appropriate appearance. It does not automatically theme custom component UI; components still need to consume CSS tokens.

## Applying a theme

### 13. How can I switch themes without changing every component?

Override semantic custom properties on a common ancestor, for example `[data-theme='dark'] { --color-surface: ...; }`. Descendant components inherit the changed values while keeping their styles unchanged.

### 14. Where should I put the `data-theme` attribute?

On an ancestor of every region that should inherit the theme. Setting it on a `<main>` element themes that subtree, not its siblings, ancestors, or browser chrome. Put it higher if more of the document should respond.

### 15. Can Angular bind `data-theme` from application state?

Yes. Bind an attribute in the template, such as `<main [attr.data-theme]="theme()">`. A signal can hold the current theme and update the attribute reactively. This avoids manually changing the DOM for a simple theme switch.

### 16. How can I support the operating system's preferred color scheme?

Use CSS `@media (prefers-color-scheme: dark)` for the system default, then let an explicit `[data-theme='light']` or `[data-theme='dark']` selector override it. If Angular controls the theme state, define what `system`, `light`, and `dark` mean rather than assuming the media query will override an explicit theme.

### 17. How does `prefers-color-scheme` interact with explicit theme attributes?

It is a CSS media condition, not a persistent user preference mechanism. Arrange selectors so the system rule sets a default and a more specific explicit theme selector wins. Keep the theme attribute and token rules consistent.

### 18. Why might the page flash between themes on load?

The server may render the default theme before the browser loads a saved preference. If the client changes the theme after hydration, users can see a flash and server/client markup may disagree. Choose an initial strategy that keeps the server and first client render consistent, or apply the preference before rendering when the application architecture supports it.

### 19. Can CSS custom properties theme a Shadow DOM component?

Yes. Custom properties and other inherited values can pass through the shadow host. Outside selectors still cannot directly match internal shadow elements, so custom properties are a useful public theming interface.

## Angular CSS-variable namespacing

### 20. Does Angular namespace CSS custom properties by default?

No. `provideCssVarNamespacing()` is an optional API. An application that owns its page usually does not need it; it is useful when independent applications or external markup could reuse the same custom-property names.

### 21. What does `provideCssVarNamespacing()` do?

It configures Angular to prefix custom-property names in Angular-compiled styles and supported style bindings. For example, `--primary-color` can become `--storefront_primary-color` when the namespace is `storefront`.

### 22. Where do I configure CSS-variable namespacing?

Add `provideCssVarNamespacing()` to the application providers, typically in `app.config.ts`. It uses the `APP_ID` as its namespace when no explicit namespace is supplied. You can also pass a namespace string directly.

### 23. Does the provider append an underscore?

Yes. Angular appends an underscore to the namespace. Passing `storefront` produces the `storefront_` prefix; passing `storefront_` produces a double underscore before the variable name. Usually pass the namespace without a trailing underscore.

### 24. What is `APP_ID` used for?

`APP_ID` identifies an Angular application, including for generated style and attribute prefixes. If several Angular applications share one page, give them distinct IDs or explicit namespaces so namespacing can distinguish their variables.

### 25. Which styles does Angular rewrite when namespacing is enabled?

Angular rewrites custom-property names in compiled component styles, including component styles, relevant inline styles and supported bindings. The ordinary global stylesheet loaded by the build is not rewritten. Installed component libraries are compiled too, so their custom-property references may also be namespaced.

### 26. Why can namespacing break a global theme token?

A global stylesheet might declare `--color-text`, while Angular rewrites a component's `var(--color-text)` reference to an application-prefixed name. The global declaration and compiled reference no longer match unless they use a compatible namespace or opt-out.

### 27. How do I reference a global custom property from a compiled component stylesheet?

Use Angular's `--global--` opt-out prefix in that compiled stylesheet, for example `color: var(--global--color-text, #202531)`. Angular removes the `--global` marker and leaves the property name as `--color-text`.

### 28. Does `--global--` work if namespacing is disabled?

Yes. Angular removes the marker from compiled styles even when no namespace is configured, leaving the normal custom-property name. Use exactly two hyphens after `global`.

### 29. Are all ways of setting a custom property rewritten?

No. Angular documents exceptions including static style attributes, object or string `[style]` bindings, `ngStyle`, custom-property names embedded inside another binding value, and `Renderer2.setStyle()`. Prefer a property-specific binding such as `[style.--accent]="accent()"` when possible.

### 30. What if code must use a DOM API to set a custom property?

Prefer a declarative Angular style binding when it fits. If a DOM API is required and the app may enable namespacing, use Angular's `CssVarNamespacer` to resolve the runtime property name rather than assuming the raw name is unchanged.

### 31. Should every Angular app enable CSS-variable namespacing?

No. It adds coordination requirements between compiled component styles, global styles, and library styles. Enable it only when avoiding collisions across independently owned apps or markup is a real requirement, then audit custom-property references and runtime style updates.

## Debugging and architecture

### 32. A component isn't changing when I toggle the theme. What should I check?

Check that the theme attribute is on an ancestor of the component, the theme selector matches that attribute, the component reads the same token name, and the token isn't overridden closer to the component. Inspect computed styles in browser developer tools.

### 33. Should I set the page background on `body` or on the themed app subtree?

It depends on the scope of the theme. If only the Angular app region is themed, set its background on the themed root or main element. If the whole document changes, arrange the theme on the document root and apply page-level tokens there.

### 34. Should I use classes or custom properties to represent theme values?

Use a theme attribute or class to choose a theme, and custom properties to store the theme's values. Components can then consume the same semantic properties regardless of which theme selector supplied them.

### 35. Can I define the same custom property at multiple levels?

Yes. CSS cascade and inheritance determine which value a descendant reads. A closer declaration overrides an inherited ancestor value, which can be useful for component-specific variants. Keep overrides intentional and documented.

### 36. What should I inspect when namespacing appears to have no effect?

Verify the provider is registered in the correct application, the code path is in styles Angular compiles, the exact custom-property name is eligible for rewriting, and the global declaration uses the matching namespaced or opt-out name. Remember that build-loaded global CSS and several runtime style APIs are not rewritten.

## Related material

- [Lesson 34: Global styles, CSS variables, and theming](../lessons/034-global-styles-css-variables-and-theming.md)
- [Example 34: Global styles, CSS variables, and theming](../examples/034-global-styles-css-variables-and-theming.md)
- [Angular styling guide](https://angular.dev/guide/components/styling)
- [`provideCssVarNamespacing` API](https://angular.dev/api/platform-browser/provideCssVarNamespacing)
- [`APP_ID` API](https://angular.dev/api/core/APP_ID)
- [Angular workspace file structure](https://angular.dev/reference/configs/file-structure)
