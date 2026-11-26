# 33. Component styles and view encapsulation

Angular components can keep CSS next to the component that owns the template. **View encapsulation** determines how Angular scopes those component styles. Angular's default, `Emulated`, scopes selectors to a component's template without creating a browser Shadow DOM. `ShadowDom` uses a real shadow root, while `None` makes a component's styles global. Current Angular documentation also lists `ExperimentalIsolatedShadowDom`, an experimental mode that blocks styles from leaking into or out of a shadow root.

## Learning goals

After this lesson, you should be able to:

- Define component styles inline or in stylesheet files.
- Explain what Angular's default emulated style scoping does and does not isolate.
- Choose between `Emulated`, `None`, and `ShadowDom` for a component.
- Recognize `ExperimentalIsolatedShadowDom` as experimental.
- Use `:host` to style a component's own host element.
- Avoid new `::ng-deep` usage and understand its effect in older code.
- Diagnose global style bleed, CSS custom property inheritance, and cascade conflicts.

## Where component styles live

A component may keep a short stylesheet inline with `styles`:

~~~ts
@Component({
  selector: 'app-avatar',
  template: '<img src="avatar.png" alt="Profile" />',
  styles: `
    img {
      width: 3rem;
      aspect-ratio: 1;
      border-radius: 50%;
    }
  `,
})
export class AvatarComponent {}
~~~

For most components, use a separate stylesheet with `styleUrl`:

~~~ts
@Component({
  selector: 'app-profile-card',
  templateUrl: './profile-card.html',
  styleUrl: './profile-card.css',
})
export class ProfileCardComponent {}
~~~

Use `styleUrls` when referencing multiple component stylesheets. Paths are relative to the component file. Angular compiles component styles with the component, and includes them when the component is rendered, including when the component is lazy-loaded. Angular supports CSS and preprocessors such as Sass, Less, and Stylus when the project's build pipeline is configured for them.

Project-wide styles belong in the global stylesheet configured by the Angular CLI project, commonly `src/styles.css`. Use global styles for resets, base typography, and application-wide tokens. Keep component-specific selectors in that component's stylesheet so their ownership is clear.

## Default: `ViewEncapsulation.Emulated`

Unless configured otherwise, Angular uses `ViewEncapsulation.Emulated`. Angular adds generated attributes to the host and elements in the component template, then rewrites that component's CSS selectors to include those attributes. For example, a component selector such as `.title` is compiled into a selector that also matches the generated attribute for that component's view.

This is Angular's style-scoping mechanism; it is **not** a native Shadow DOM boundary. It usually prevents a component stylesheet from selecting elements outside that component's own template, but global CSS selectors can still match elements inside the component. Inherited properties and CSS custom properties also continue to follow normal CSS inheritance.

~~~ts
@Component({
  selector: 'app-message-card',
  template: '<h2 class="title">Saved</h2>',
  styles: `
    :host {
      display: block;
    }

    .title {
      color: seagreen;
    }
  `,
})
export class MessageCardComponent {}
~~~

The `.title` rule is scoped to this component's template in emulated mode. A global rule such as `h2 { font-family: sans-serif; }` can still affect the heading.

## The `:host` selector

In a component stylesheet, `:host` selects the component's own host element, such as `<app-message-card>`:

~~~css
:host {
  display: block;
  margin-block: 1rem;
}

:host(.compact) {
  margin-block: 0.5rem;
}
~~~

Use `:host` for the component's layout footprint and host-level state. A component stylesheet does not automatically give a custom element the block display of a `div`; set `display` when the component should participate as a block.

`:host-context(selector)` is also supported by Angular's emulated compiler, but the pseudo-class is deprecated in modern browsers. Prefer an explicit host class or inherited CSS custom property for theme and context styles in new designs.

## `ViewEncapsulation.None`

`ViewEncapsulation.None` disables encapsulation for that component's styles. Angular inserts the styles as global CSS; the selectors can affect matching elements anywhere in the application.

~~~ts
import { Component, ViewEncapsulation } from '@angular/core';

@Component({
  selector: 'app-global-label',
  template: '<span class="global-label">Component label</span>',
  styles: `
    .global-label {
      color: darkmagenta;
      font-weight: 700;
    }
  `,
  encapsulation: ViewEncapsulation.None,
})
export class GlobalLabelComponent {}
~~~

A `.global-label` element outside `app-global-label` can receive this style too. Use `None` only when global reach is intentional, and namespace selectors to reduce accidental collisions. For a shared global design system, a dedicated global stylesheet is often easier to discover than a component stylesheet that quietly changes the whole application.

## `ViewEncapsulation.ShadowDom`

`ViewEncapsulation.ShadowDom` asks Angular to attach a browser Shadow Root to the component host and render the component template and styles inside it:

~~~ts
import { Component, ViewEncapsulation } from '@angular/core';

@Component({
  selector: 'app-isolated-widget',
  template: '<h2 class="title">Widget heading</h2>',
  styles: `
    :host {
      display: block;
      padding: 1rem;
      border: 1px solid currentColor;
    }

    .title {
      color: darkslateblue;
    }
  `,
  encapsulation: ViewEncapsulation.ShadowDom,
})
export class IsolatedWidgetComponent {}
~~~

The component's stylesheet cannot select elements outside its shadow tree, and outside selectors do not cross into the tree to style its internal elements. CSS inheritance and custom properties on the host can still provide deliberate styling inputs. Shadow DOM also changes more than CSS: it affects event propagation and retargeting, browser inspection, and how content projection interacts with native slots. Use it when native encapsulation is part of the component's design, not simply as a way to increase selector priority.

## Experimental isolated Shadow DOM

Current Angular docs also list `ViewEncapsulation.ExperimentalIsolatedShadowDom`. It behaves like `ShadowDom` and additionally prevents global styles from affecting elements inside the component's shadow root. The API is explicitly marked **experimental**, so verify its support and behavior for the Angular version you use before adopting it in application or library APIs.

~~~ts
@Component({
  selector: 'app-strict-widget',
  template: '<p>Isolated widget content</p>',
  styles: ':host { display: block; }',
  encapsulation: ViewEncapsulation.ExperimentalIsolatedShadowDom,
})
export class StrictWidgetComponent {}
~~~

## Avoid new `::ng-deep` usage

Angular's emulated mode still supports the custom `::ng-deep` pseudo-class for backward compatibility, but Angular strongly discourages new use. It stops Angular's selector rewriting after that point, allowing the rest of the selector to reach beyond the component's style boundary. Depending on its placement, a rule can effectively become global or reach projected and child-component content.

~~~css
/* Legacy pattern: avoid adding this to new component styles. */
:host ::ng-deep .third-party-label {
  color: teal;
}
~~~

Prefer a supported styling API: a CSS custom property, a documented class or input, a global theme rule, or a style API provided by the child component or library. Do not use `::ng-deep` as a routine way to reach into another component's private template.

## External styles and inline `<style>` elements

A `<style>` element in a component template is subject to that component's encapsulation, but Angular does not support Angular template bindings inside the `<style>` element. Use component metadata or a stylesheet for component styles, and template bindings for dynamic values.

A component template may also reference a stylesheet with `<link>`, or a stylesheet may use CSS `@import`. Angular treats these as external styles, and they are not rewritten by emulated encapsulation. Review their selectors as potentially global rather than assuming the component's `Emulated` mode scopes them.

## Encapsulation does not replace the CSS cascade

View encapsulation scopes selectors; it is not a guarantee that a component's CSS always wins. CSS specificity and the cascade still matter, and Angular does not guarantee that component styles will always override outside styles in `Emulated` or `ShadowDom` mode when specificity is equal.

When styles conflict:

1. Inspect the rendered element and matched selectors in browser developer tools.
2. Check whether the rule is component-scoped, global, inherited, or coming from an external stylesheet.
3. Compare specificity, source order, cascade layers, and `!important` declarations.
4. Prefer a clear public styling hook over reaching into another component's internal markup.

Do not increase specificity blindly or use `::ng-deep` to hide an unclear ownership problem.

## CSS custom properties as component styling inputs

Custom properties inherit through the DOM and are useful for theming a component without penetrating its template:

~~~css
/* app-card.css */
:host {
  display: block;
  border-color: var(--card-border, #cbd5e1);
  color: var(--card-text, #1f2937);
}
~~~

~~~html
<app-card style="--card-border: rebeccapurple; --card-text: #241044">
  Content
</app-card>
~~~

The same technique can provide values across a Shadow DOM host boundary. If an application deliberately enables Angular's CSS custom-property namespacing feature, review which tokens cross between global styles and component styles; namespacing can change the property names Angular rewrites in compiled component CSS and bindings.

## Choosing an encapsulation mode

| Mode | How styles are scoped | Typical use |
| --- | --- | --- |
| `Emulated` | Angular rewrites selectors with generated attributes; default | Most Angular components |
| `None` | Styles are global | Intentionally global styles with carefully namespaced selectors |
| `ShadowDom` | Browser Shadow Root provides a native boundary | Components designed to use native Shadow DOM semantics |
| `ExperimentalIsolatedShadowDom` | Shadow DOM plus isolation from external styles; experimental | Evaluate only when the experimental API's stronger boundary is specifically needed |

Start with the default `Emulated`. Choose a different mode only when its global or native-boundary behavior is a deliberate requirement.

## Common mistakes

- **Assuming emulated mode is native Shadow DOM:** global CSS and inheritance can still affect component content.
- **Using `None` for a local component fix:** its styles become global and may unexpectedly change unrelated elements.
- **Using `ShadowDom` only to beat CSS specificity:** it changes event and slot behavior as well as style scoping.
- **Using `::ng-deep` in new code:** it bypasses style boundaries and is retained for compatibility only.
- **Styling another component's private elements:** use a public styling hook such as a CSS custom property or documented class instead.
- **Expecting encapsulation to beat every conflicting selector:** inspect the normal CSS cascade and specificity.
- **Forgetting `:host { display: block; }`:** a custom host remains inline by default unless styled otherwise.
- **Treating encapsulation as a security boundary:** it is a CSS scoping feature, not an HTML or JavaScript security mechanism.

## Practical checklist

Before choosing where a style belongs, ask:

- Is it a global reset/theme rule or a component-specific rule?
- Does it target the host (`:host`) or an element inside the component template?
- Is the default `Emulated` mode sufficient?
- If styles appear to leak, are they global, inherited, external, or using `ViewEncapsulation.None`?
- Am I about to use `::ng-deep` where a public styling hook would work?
- Would `ShadowDom` introduce event, slot, or tooling changes for this component?
- Is the experimental isolated mode worth its compatibility and stability tradeoffs?

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="032-host-bindings-host-listeners-and-host-directives.md">032 - 32. Host bindings, host listeners, and host directives</a></td><td><a href="../qnas/033-component-styles-and-view-encapsulation.md">Q&amp;A</a></td><td><a href="../examples/033-component-styles-and-view-encapsulation.md">Example</a></td><td><a href="034-global-styles-css-variables-and-theming.md">034 - 34. Global styles, CSS variables, and theming</a></td>
</tr>
  </tbody>
</table>
