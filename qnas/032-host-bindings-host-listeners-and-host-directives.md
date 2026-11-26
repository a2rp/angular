# 32. Q&A: Host bindings, host listeners, and host directives

## Host element basics

### 1. What is a component's host element?

It is the DOM element that matches the component selector. If a component uses `selector: 'app-notice'`, then `<app-notice>` is its host. The component template renders inside that element; bindings in the component's `host` metadata target the `<app-notice>` element itself.

### 2. Do directives also have a host element?

Yes. For an attribute directive such as `<p appHighlight>`, the `<p>` element is the directive's host. The directive can bind classes, attributes, styles, properties, and events on that element.

### 3. When should I use a host binding instead of a template binding?

Use a host binding when the behavior belongs to the component's or directive's own element, such as reflecting its state with an ARIA attribute or listening for a key event on that element. Use a regular template binding for a specific element inside the component's template.

### 4. Which host-binding syntax should I use in new Angular code?

Prefer the `host` metadata property on `@Component` or `@Directive`. Angular's current guidance says `@HostBinding` and `@HostListener` are retained for backwards compatibility. Learn those decorators to maintain older code, but use `host` metadata for new bindings.

## Host properties, attributes, classes, and styles

### 5. What can the `host` metadata bind?

It can set static host values and bind DOM properties, HTML attributes, CSS classes, styles, and event listeners. Use static keys for literal values and bracketed keys for expressions, for example `'role': 'status'` and `'[attr.aria-busy]': 'loading() ? "true" : null'`.

### 6. What is the difference between `[tabIndex]` and `[attr.tabindex]`?

`tabIndex` is a DOM property that controls the element's keyboard focus order. `tabindex` is the corresponding HTML attribute. Prefer the DOM property when controlling the live element state; use an attribute binding when the attribute itself is the desired target. For ARIA attributes, use `[attr.aria-*]`.

### 7. Why do examples use strings for ARIA booleans?

ARIA state values are text such as `"true"` or `"false"`. A binding like `'[attr.aria-expanded]': 'expanded() ? "true" : "false"'` writes the appropriate attribute value. Returning `null` removes the attribute. This is distinct from native HTML Boolean attributes, whose meaning usually depends on whether the attribute is present.

### 8. What is the difference between removing an attribute and setting it to `false`?

With an attribute binding, `null` removes the attribute. The string `"false"` leaves the attribute present with the value `false`. For ARIA, that distinction is meaningful: some states support an explicit false value, while an absent attribute can mean the state is not specified.

### 9. Can host metadata bind a CSS custom property?

Yes. For example, `'[style.--notice-color]': 'color()'` can set a custom property on the host. The component's CSS can then read it with `var(--notice-color, steelblue)`. This lets consumers or component state configure styling without reaching into internal elements.

### 10. Should I bind `[class]` or a specific class?

For a single state, bind a specific class such as `'[class.is-invalid]': 'invalid()'`. This composes more clearly with other classes than replacing the entire class string. Use Angular's supported class and style bindings rather than assigning directly to `className` or `style` for simple state changes.

## Host event listeners

### 11. How do I listen to a host event with modern syntax?

Add an event key to `host` metadata:

```ts
host: {
  '(click)': 'activate()',
  '(keydown.enter)': 'activate()',
}
```

An expression can receive the event object with `$event`, for example `'(keydown)': 'onKeydown($event)'`.

### 12. How can a host listener listen to a global event?

Prefix the event with `window:`, `document:`, or `body:`, such as `'(window:keydown.escape)': 'closeMenu()'`. Use global listeners only when the behavior truly concerns that global target; keep element-specific interactions on the host or in its template.

### 13. What do keyboard event modifiers such as `.enter` mean?

They filter events by `KeyboardEvent.key`. For example, `(keydown.enter)` runs for the Enter key. `key` can vary with keyboard layout and input language; use a `.code` modifier such as `(keydown.code.Enter)` when you need to match the physical key position.

### 14. Does returning `false` from a host event handler stop event propagation?

No. In Angular's host event binding, a handler result of `false` requests `preventDefault()` for the event. It does not mean `stopPropagation()`. Prefer to call `event.preventDefault()` explicitly in a handler when that is part of the interaction's intended behavior, and call `stopPropagation()` only when there is a specific reason to stop bubbling.

### 15. Can a click inside a component's template trigger a host click listener?

Yes. DOM click events bubble. A click on an internal button can reach the host and invoke its click listener. If the component only needs the button's click, bind to that button in the template instead. If the host listener is intentional, account for `event.target` and `event.currentTarget`.

### 16. Does an interactive custom component become accessible just by setting `role="button"`?

No. A role communicates semantics but does not automatically add native button behavior. A custom control also needs sensible focus behavior, keyboard interaction, state, and disabled behavior. Prefer a native `<button>` whenever it fits the design.

### 17. Should I listen for keyboard and pointer events on the host or its internal element?

Choose the element that owns the interaction. A host listener is appropriate when the whole component host acts as the control. If only a particular internal button or input should respond, put the listener on that element in the component template.

## Legacy decorators

### 18. What does `@HostBinding` do?

It connects a class member, getter, or supported input to a host property, class, style, or attribute. For example, `@HostBinding('attr.aria-expanded')` binds the decorated value to the host's `aria-expanded` attribute.

### 19. What does `@HostListener` do?

It declares an event listener on the host and routes the event to a class method. It can pass event arguments, for example `@HostListener('keydown', ['$event'])`. Angular recommends `host` metadata for new code, while the decorator remains relevant when reading or maintaining existing projects.

### 20. Should I combine a decorator and `host` metadata for the same property or event?

Avoid it. Multiple declarations for the same host binding make ownership and update behavior harder to reason about. Choose one form for each binding; for new code, use `host` metadata.

## Host directives

### 21. What is a host directive?

It is a directive applied to a component's or another directive's host through the `hostDirectives` metadata option. Angular creates the directive instance and applies its host bindings, so the component can reuse behavior without duplicating its implementation.

### 22. When is `hostDirectives` useful?

Use it when a component should always include a reusable behavior as part of its implementation—for example, shared focus handling or a common host class. Use an ordinary attribute directive when consumers should independently choose where to apply behavior. Use a component when the behavior also owns a view and its own UI.

### 23. Does a host directive need to match its selector?

No. Angular ignores the selector when a directive is listed in `hostDirectives`; it is applied through the composition metadata. The directive must be standalone. Host directives are resolved statically at compile time, so they cannot be attached dynamically at runtime.

### 24. Are a host directive's inputs and outputs automatically public on the component?

No. They are hidden by default. To include one in the composed component's API, list it explicitly, for example `inputs: ['label: accessibleLabel']` or `outputs: ['activated: activate']`. The name before the colon is declared by the directive; the name after it is the public name exposed by the component.

### 25. Can host directives be composed together?

Yes. A directive can list host directives, and another directive or component can compose that directive. This lets behavior be assembled from smaller pieces. Keep the dependency structure understandable and avoid exposing implementation details that consumers do not need.

### 26. Which runs first: a host directive or the component that applies it?

The host directive runs its constructor, lifecycle hooks, and host bindings before the component or directive that applies it. As a result, the composing component can override host bindings specified by its host directive. Do not rely on unrelated directives on the same element having a particular ordering unless Angular documents it.

### 27. Can the composing component inject its host directive?

Yes. A component or directive that applies a host directive can inject that host directive instance, and the host directive can inject the composing component or directive. Use this only when that direct collaboration is part of a clear design.

### 28. What happens if the same directive appears more than once in a host-directive composition tree?

Angular deduplicates repeated host-directive instances in the resolved composition tree. If repeated paths expose an input or output with conflicting aliases, Angular reports a compile-time error. Keep aliases consistent across the composition.

## Conflicts and debugging

### 29. Which class or style value wins if the template and host both bind it?

Angular's class/style priority is: dynamic template binding, static template value, directive host binding, component host binding, and static host value. Prefer not to rely on a conflict for normal configuration; define one clear owner for the value.

### 30. Which non-style binding wins if both the template and host set it?

For static values, a static attribute from the template wins over a static host value. If one value is static and one is dynamic, the dynamic value wins. Avoid duplicate dynamic bindings to the same non-style property or attribute: Angular does not reconcile them, and updates can overwrite each other.

### 31. What is the difference between `null` and `undefined` for class/style host bindings?

For Angular's documented class and style precedence, `undefined` falls back to the next value in the priority order. `null` removes that class or style without falling back. Use the distinction intentionally when multiple class/style sources are involved.

### 32. A host binding is not updating. What should I inspect?

Check that the binding is in the component or directive metadata that owns the intended host, that its expression uses the right property or signal call, and that it is not competing with another dynamic binding. For events, confirm the event is emitted on that element and consider bubbling from descendants. For host directives, verify that the directive is listed and that any needed inputs or outputs are explicitly exposed.

### 33. Should I use a host binding to update an element inside the template?

Usually no. Host bindings target only the component or directive host. For a child element inside the template, use a normal Angular binding in that template. This keeps the ownership of the markup and state together.

## Related material

- [Lesson 32: Host bindings, host listeners, and host directives](../lessons/032-host-bindings-host-listeners-and-host-directives.md)
- [Example 32: Host bindings, host listeners, and host directives](../examples/032-host-bindings-host-listeners-and-host-directives.md)
- Angular component host elements
- Angular directive composition API
- [`HostBinding` API](`https://angular.dev/api/core/HostBinding`)
- [`HostListener` API](`https://angular.dev/api/core/HostListener`)

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="031-lifecycle-hooks-and-component-initialization.md">031 - 31. Lifecycle hooks and component initialization</a></td><td><a href="../lessons/032-host-bindings-host-listeners-and-host-directives.md">Lesson</a></td><td><a href="../examples/032-host-bindings-host-listeners-and-host-directives.md">Example</a></td><td><a href="033-component-styles-and-view-encapsulation.md">033 - 33. Component styles and view encapsulation</a></td>
</tr>
  </tbody>
</table>
