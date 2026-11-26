# 32. Host bindings, host listeners, and host directives

A component's **host element** is the DOM element that matches its selector. For `<app-notice>`, the `<app-notice>` element is the host; the component template is rendered inside it. Directives also have a host: the element on which the directive matches.

Angular lets a component or directive bind properties, attributes, classes, styles, and events on that host. A directive can also be composed onto a component's host with `hostDirectives`, allowing reusable behavior to travel with a component.

## Learning goals

After this lesson, you should be able to:

- Identify a component or directive's host element.
- Use `host` metadata for host attributes, properties, classes, styles, and event listeners.
- Choose between host property binding and attribute binding.
- Handle host keyboard and global events deliberately.
- Recognize legacy `@HostBinding` and `@HostListener` code and prefer modern `host` metadata in new code.
- Compose reusable behavior with `hostDirectives` and expose only the inputs and outputs a component should publish.
- Avoid host-binding collisions and accessibility mistakes.

## The host element

Given this component:

~~~ts
@Component({
  selector: 'app-notice',
  template: '<p>Saved.</p>',
})
export class NoticeComponent {}
~~~

and this template usage:

~~~html
<app-notice></app-notice>
~~~

`<app-notice>` is the host element. Bindings in `NoticeComponent`'s template affect elements inside the host. Host bindings affect the `<app-notice>` element itself.

Host bindings are useful when behavior belongs to the component's public element: for example, setting its role, reflecting its state through ARIA, exposing a CSS custom property, or handling a keyboard event on the component host. Use ordinary template bindings for elements that live inside the component template.

## Use the `host` metadata

The `host` property on `@Component` or `@Directive` metadata declares static host values and dynamic bindings. It supports DOM properties, attributes, classes, styles, and events.

~~~ts
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-toggle',
  standalone: true,
  template: '<ng-content></ng-content>',
  host: {
    'role': 'button',
    '[attr.aria-pressed]': 'pressed() ? "true" : "false"',
    '[attr.aria-disabled]': 'disabled() ? "true" : "false"',
    '[class.is-pressed]': 'pressed()',
    '[style.--toggle-accent]': 'accent()',
    '[tabIndex]': 'disabled() ? -1 : 0',
    '(click)': 'activate()',
    '(keydown.enter)': 'activate()',
    '(keydown.code.Space)': 'onSpaceKeydown($event)',
    '(keyup.code.Space)': 'onSpaceKeyup($event)',
  },
})
export class ToggleComponent {
  readonly pressed = signal(false);
  readonly disabled = signal(false);
  readonly accent = signal('rebeccapurple');

  activate(): void {
    if (!this.disabled()) {
      this.pressed.update((value) => !value);
    }
  }

  onSpaceKeydown(event: KeyboardEvent): void {
    event.preventDefault();
  }

  onSpaceKeyup(event: KeyboardEvent): void {
    event.preventDefault();
    this.activate();
  }
}
~~~

The values in `host` are Angular template expressions. Dynamic expressions use square brackets; event names use parentheses. Event expressions can refer to `$event`, just like template event bindings.

A host metadata object can also name global event targets:

~~~ts
host: {
  '(window:keydown.escape)': 'closeMenu()',
  '(document:click)': 'onDocumentClick($event)',
}
~~~

The supported global target prefixes are `window:`, `document:`, and `body:`. A key modifier such as `.enter` matches `KeyboardEvent.key`, which depends on keyboard layout and input language. Use a `.code` modifier, such as `(keydown.code.Enter)`, when the physical key matters regardless of layout.

## Property bindings and attribute bindings

Use a property binding when you mean a DOM property, such as `tabIndex`, `id`, or `title`:

~~~ts
host: {
  '[tabIndex]': 'disabled() ? -1 : 0',
}
~~~

Use an attribute binding for HTML attributes that do not have a matching DOM property, and for accessibility attributes such as `aria-*`:

~~~ts
host: {
  '[attr.aria-expanded]': 'expanded() ? "true" : "false"',
  '[attr.aria-describedby]': 'descriptionId() || null',
}
~~~

Returning `null` from an attribute binding removes that attribute. ARIA state values are strings such as `"true"` and `"false"`; omitting an ARIA attribute is different from setting it to the string `"false"`. Native HTML Boolean attributes have their own presence-based rules, so use an appropriate DOM property or Angular binding for the element and attribute involved.

## Classes, styles, and CSS custom properties

A host binding can expose a state as a class or style:

~~~ts
host: {
  '[class.is-invalid]': 'invalid()',
  '[style.opacity]': 'disabled() ? 0.6 : 1',
  '[style.--notice-color]': 'color()',
}
~~~

CSS custom properties make a component's styles configurable without reaching into its internal DOM:

~~~css
:host {
  display: block;
  border-inline-start: 0.25rem solid var(--notice-color, steelblue);
}
~~~

~~~ts
readonly color = signal('teal');
~~~

Avoid assigning directly to `className` or `style` if you only need to toggle one class or style. Class and style bindings preserve Angular's ability to combine individual values.

## Host event listeners

A host event binding listens on the component or directive's host element. Keep the handler focused on the behavior of that component or directive:

~~~ts
host: {
  '(pointerenter)': 'setHovered(true)',
  '(pointerleave)': 'setHovered(false)',
  '(keydown.escape)': 'dismiss()',
}
~~~

~~~ts
setHovered(value: boolean): void {
  this.hovered.set(value);
}
~~~

Events from descendants can bubble to the host. If the host listens for `click`, a click on a button inside its template may also reach that listener. Consider whether that is intended; inspect `event.target` and `event.currentTarget` where needed, or attach the event to the specific internal element in the template instead.

Do not add `preventDefault()` or stop propagation automatically. Do so only when the component's interaction contract requires it. Keyboard interaction should complement the semantics of the host element. For a control that behaves like a button, a native `<button>` is usually safer than recreating button semantics on a custom element; when a custom host is necessary, define its role, keyboard behavior, focusability, and disabled behavior together.

## Prefer `host` over `@HostBinding` and `@HostListener`

Older code may declare host behavior on class members:

~~~ts
@HostBinding('attr.aria-expanded')
get ariaExpanded(): string {
  return this.expanded() ? 'true' : 'false';
}

@HostListener('keydown.escape', ['$event'])
onEscape(event: KeyboardEvent): void {
  this.dismiss();
}
~~~

`@HostBinding` binds a class member to a host property, class, style, or attribute. `@HostListener` declares a host event and calls a method. Angular retains these decorators for backwards compatibility, but its current guidance is to prefer the `host` metadata property for new code:

~~~ts
@Component({
  selector: 'app-disclosure',
  host: {
    '[attr.aria-expanded]': 'expanded() ? "true" : "false"',
    '(keydown.escape)': 'dismiss()',
  },
  template: '...',
})
export class DisclosureComponent {}
~~~

The decorator form remains useful to recognize while reading and maintaining existing Angular applications. Avoid mixing both forms for the same host binding.

## Host binding collisions

A component is used on an element where its consumer can also write bindings. The consumer, the component, and directives on that element can therefore target the same property, attribute, class, or style.

For class and style bindings, Angular's documented priority is:

1. Dynamic template binding.
2. Static template value.
3. Directive host binding.
4. Component host binding.
5. Static host value.

For example, a consumer's `class` or `[class.active]` can take priority over a component's host class binding. A `null` style/class binding removes that binding without falling back; an `undefined` value falls back to the next priority.

For non-style properties and attributes, static template values win over static host values, and a dynamic value wins over a static value. Avoid two dynamic bindings to the same DOM property or attribute, such as the consumer and component both dynamically binding `id`. Angular does not reconcile those duplicates; updates can overwrite one another depending on change-detection order.

Choose a clear owner for each host property. Let consumers configure a component through documented inputs rather than making them compete with internal bindings on the same attribute.

## Compose reusable behavior with `hostDirectives`

A host directive applies a standalone directive to a component's host element from the component's metadata. It is useful when several components should share behavior such as hover state, focus state, or a common keyboard interaction.

~~~ts
import { Directive, input, signal } from '@angular/core';

@Directive({
  standalone: true,
  host: {
    '[class.is-hovered]': 'hovered()',
    '[attr.aria-label]': 'label()',
    '(pointerenter)': 'hovered.set(true)',
    '(pointerleave)': 'hovered.set(false)',
  },
})
export class HoverBehaviorDirective {
  readonly label = input('Interactive item');
  readonly hovered = signal(false);
}
~~~

A component can apply this behavior without repeating its host bindings:

~~~ts
import { Component } from '@angular/core';
import { HoverBehaviorDirective } from './hover-behavior.directive';

@Component({
  selector: 'app-hover-card',
  standalone: true,
  hostDirectives: [HoverBehaviorDirective],
  template: '<ng-content></ng-content>',
})
export class HoverCardComponent {}
~~~

The applied directive's selector is ignored; Angular applies it because it appears in `hostDirectives`. The composition is static at compile time, not something to add conditionally at runtime. Host directives cannot be non-standalone.

### Expose a deliberate component API

A host directive's inputs and outputs are private to that composition by default. A component selectively exposes them in its own metadata and can rename them:

~~~ts
@Component({
  selector: 'app-hover-card',
  standalone: true,
  hostDirectives: [
    {
      directive: HoverBehaviorDirective,
      inputs: ['label: accessibleLabel'],
    },
  ],
  template: '<ng-content></ng-content>',
})
export class HoverCardComponent {}
~~~

A consumer can then write:

~~~html
<app-hover-card accessibleLabel="Account actions">
  <button type="button">Open account</button>
</app-hover-card>
~~~

To expose an output, list it in `outputs`, for example `outputs: ['activated: activate']`. The value before the colon is the host directive's input or output name; the value after it is the public alias on the component.

Host directives can also be applied to other directives to build reusable behavior from smaller pieces. A host directive and the component that composes it can inject one another. Host directives run their constructors, hooks, and host bindings before the composing component's corresponding work; this allows the component's host binding to override one supplied by its host directive.

## Accessibility and design guidance

- Prefer a native element that already has the required semantics, such as `<button>` for an action.
- Keep ARIA state synchronized with the actual component state. Use the correct role and keyboard behavior; a `role` alone does not make a custom element behave like a native control.
- Use host bindings for behavior and state belonging to the host. Use template bindings for internal elements.
- Make host behavior reusable with a directive when the same behavior belongs on unrelated elements or several components.
- Use `hostDirectives` when a component should always include a behavior as part of its implementation. Expose only the inputs and outputs that belong in the component's public API.
- Avoid overlapping dynamic bindings and avoid direct DOM mutation when Angular bindings can express the behavior.

## Common mistakes

- **Binding to the wrong element:** host bindings target the component's selector element, not an element inside its template.
- **Using decorators for new host bindings:** prefer the `host` metadata property; keep decorators for compatibility with existing code.
- **Confusing attributes with properties:** use `[attr.aria-*]` for ARIA attributes and a DOM property binding for properties such as `tabIndex`.
- **Adding click behavior without keyboard support:** interactive controls need suitable semantics, focus, and keyboard behavior. Prefer native controls.
- **Assuming a child click cannot reach a host listener:** bubbling events from the template can invoke a listener on the host.
- **Binding the same property dynamically in multiple places:** select one owner or redesign the component API.
- **Expecting host directive inputs to be public automatically:** explicitly expose and alias inputs/outputs in `hostDirectives` when consumers need them.
- **Trying to add host directives dynamically:** host directive composition is static.

## Practical checklist

Before adding host behavior, ask:

- Does this state belong to the host, or to an internal element in the template?
- Is the target a DOM property or an HTML attribute?
- Can the host use a native semantic element instead of a custom interactive element?
- Could an event bubble from a child and trigger this host listener?
- Is another template or host binding already targeting this property?
- Would this behavior be clearer as a reusable attribute directive?
- If I compose a host directive, which inputs and outputs should become part of the component's API?

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="031-lifecycle-hooks-and-component-initialization.md">031 - 31. Lifecycle hooks and component initialization</a></td><td><a href="../qnas/032-host-bindings-host-listeners-and-host-directives.md">Q&amp;A</a></td><td><a href="../examples/032-host-bindings-host-listeners-and-host-directives.md">Example</a></td><td><a href="033-component-styles-and-view-encapsulation.md">033 - 33. Component styles and view encapsulation</a></td>
</tr>
  </tbody>
</table>
