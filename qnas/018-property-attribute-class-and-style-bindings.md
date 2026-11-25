# Q&A: Property, Attribute, Class, and Style Bindings

## Property and attribute bindings

### 1. What does property binding do?

Square brackets bind an Angular expression to a target property. For a native element, this commonly updates a DOM property, such as the button's disabled state.

~~~html
<button [disabled]="isSaving">Save</button>
~~~

### 2. What can a property binding target?

Depending on the element and the name, it can target a native DOM property, an Angular component input, or a directive input. Angular resolves the target using the template and the directives or components in scope.

### 3. How do I bind to an HTML attribute explicitly?

Prefix the target with attr.:

~~~html
<div [attr.data-state]="state"></div>
~~~

Use this when the desired target is an HTML attribute rather than a DOM property or Angular input.

### 4. When is attr. especially useful?

It is useful for attributes that do not have a matching DOM property, including many ARIA, SVG, and custom data attributes. For example:

~~~html
<button [attr.aria-label]="accessibleName">Open</button>
~~~

### 5. What happens if an attr. binding evaluates to null?

Angular removes the attribute from the element. This is useful for optional attributes:

~~~html
<div [attr.data-reference]="reference"></div>
~~~

If reference is null, data-reference is absent.

### 6. Does binding false to a boolean HTML attribute disable it?

Not when using attr. A boolean HTML attribute is enabled by its presence; the string value false still leaves it present. Use a property binding for native boolean properties:

~~~html
<button [disabled]="isDisabled">Continue</button>
~~~

To conditionally add an attribute explicitly, return an empty string when it should be present and null when it should be absent.

### 7. What is the difference between an HTML attribute and a DOM property?

An attribute is part of an element's markup and is represented in the DOM attribute list. A DOM property is a field on the live element object. Some attributes initialize or reflect properties, but the two concepts are not interchangeable.

### 8. Why can an input's value attribute differ from its current value?

The value attribute generally represents the markup's initial or default value. The input element's value property represents its current editable value. Bind to the property when the UI should receive the current value.

### 9. Should I use interpolation or a property binding for a dynamic element property?

Use interpolation to produce text content. Use property binding when the value belongs to an element property, component input, or directive input. Use attr. when the target specifically needs to be an HTML attribute.

### 10. Are ARIA attributes boolean DOM properties?

ARIA attributes are attributes whose values are strings, often including the strings true or false. Bind them explicitly with attr. when needed:

~~~html
<button [attr.aria-expanded]="isExpanded">Details</button>
~~~

The string false is meaningful for aria-expanded; it does not remove the attribute.

### 11. How do I bind an SVG attribute?

Use attr. when the SVG target is an attribute:

~~~html
<svg>
  <use [attr.href]="iconHref"></use>
</svg>
~~~

Confirm the exact attribute and browser support required by the SVG feature you use.

## Class bindings

### 12. How do I toggle one class?

Use a class binding. Angular adds the class for a truthy expression and removes it for a falsy expression:

~~~html
<div [class.is-active]="isActive"></div>
~~~

### 13. Can I bind the whole class value?

Yes. The class binding accepts a string, an array of class names, or an object whose keys are class names and whose values decide whether each class is applied.

~~~html
<div [class]="classNames"></div>
~~~

For an object, a true value includes the class and a false value excludes it.

### 14. Can static classes and dynamic class bindings be used together?

Yes. Static classes, whole-class bindings, and individual class bindings can contribute classes to an element. Keep each class's owner clear and avoid competing bindings for the same class.

### 15. How do I remove a class conditionally?

Bind the class to a boolean expression. When it evaluates to false, Angular removes that bound class. For a whole class object, set the corresponding value to false.

### 16. Should I mutate a class object or array in place?

Prefer replacing the object or array when its contents change. A new reference makes the changed value clear and works well with reference-based update checks. Do not rely on mutating an existing object or array in place to be noticed by every binding form.

### 17. When would I use NgClass instead of class bindings?

For straightforward classes, built-in class binding syntax is usually sufficient. NgClass is an available directive for class changes driven by strings, arrays, sets, or maps and can be useful for more complex cases. Import it into a standalone component if the template uses it.

## Style bindings

### 18. How do I bind one CSS property?

Use style binding:

~~~html
<div [style.background-color]="backgroundColor"></div>
~~~

The property name can use kebab-case or the appropriate property form supported by Angular's template syntax.

### 19. How do I bind a numeric CSS value with a unit?

Add a unit suffix to the binding name:

~~~html
<div [style.width.px]="panelWidth"></div>
<div [style.font-size.rem]="fontSize"></div>
~~~

Here the component values are numbers; Angular adds the requested unit.

### 20. What if the numeric style binding has no unit suffix?

A number without an explicit suffix is appropriate only for CSS properties that accept unitless numbers, such as opacity or line-height. Length properties such as width need a unit.

### 21. Can I bind several styles at once?

Yes. The whole-style binding accepts a style string or an object mapping CSS property names to values:

~~~html
<div [style]="panelStyles"></div>
~~~

Use a clear source of truth and keep the styles in a component value when the set is substantial.

### 22. How can I set a CSS custom property?

Use a style binding with the custom property name:

~~~html
<div [style.--accent-color]="accentColor"></div>
~~~

Then the stylesheet can consume it with var(--accent-color).

### 23. How do I remove a dynamically bound style?

A nullish value for a style binding removes that style contribution. For a whole-style object, omit the property or set its value to null where supported by the binding form.

### 24. How should a bound style object be updated?

Create a replacement object when its properties change rather than mutating it in place:

~~~ts
this.panelStyles = {
  ...this.panelStyles,
  padding: '1.5rem',
};
~~~

This makes the changed reference visible to Angular's binding update checks.

### 25. Can I use a class instead of an inline style binding?

Yes. For reusable or substantial styling, prefer CSS classes and toggle those classes from the template. Use style bindings for values that genuinely vary at runtime, such as a measured width or a theme custom property.

## Precedence and safe use

### 26. What if static class or style values overlap with dynamic bindings?

Keep overlapping values intentional. Angular defines precedence rules for static values and bindings, and template bindings can take priority over host values. Avoid assigning the same class or style through several competing sources; that makes the result harder to reason about.

### 27. Can I put the same dynamic class in both [class] and [class.name]?

Avoid duplicate ownership. Choose one binding form for a given class where possible, so the condition controlling the class is unambiguous.

### 28. Does Angular sanitize every bound value in exactly the same way?

No. Angular applies security handling according to the binding context, such as HTML, URL, or resource URL. A value safe in one context is not automatically safe in another.

### 29. Is it safe to bind an untrusted string as HTML with innerHTML?

Angular sanitizes values in HTML contexts, but rendering untrusted HTML still needs careful review. Prefer text interpolation when the content is meant to be text.

### 30. Should I use bypassSecurityTrust... to make a binding work?

Only when the value has been independently verified as safe and the specific bypass is genuinely required. Bypass APIs disable Angular's normal protection for that value; never use them to silence a warning or to accept arbitrary user input.

### 31. Does Angular sanitize ResourceURL values?

Angular cannot sanitize a resource URL into a safe value because it may load executable content. Treat such URLs as trusted configuration, validate them at the application boundary, and avoid accepting them from untrusted sources.

### 32. Are style bindings a replacement for input validation?

No. Validate and constrain application data according to its source and purpose. Use a CSS class for fixed design choices and bind only the runtime values the UI actually needs.

### 33. What accessibility details should I check when binding attributes?

Use native HTML properties and semantics when they fit. For ARIA, provide the exact string value required by the relevant role or state, keep it synchronized with the actual UI, and ensure the control remains operable by keyboard.

### 34. What is a good way to debug a binding?

Inspect the rendered DOM and the live element property separately. Check the expression's current value, the actual attribute list, the relevant DOM property, and any CSS rule that may override the visible style.

### 35. What should I remember about binding syntax?

- Square brackets bind expressions to properties, component inputs, or directive inputs.
- Use attr. when an actual HTML attribute is the target.
- For native boolean controls, bind the DOM property rather than emitting a string attribute.
- Use class bindings for conditional classes and style bindings for genuinely dynamic values.
- Return null to remove optional attributes or styles where the binding form supports it.
- Keep binding values immutable when replacing class or style collections.
- Let Angular apply context-specific security handling; do not bypass it casually.

## Further reading

- [Angular template binding](https://angular.dev/guide/templates/binding)
- [Angular security best practices](https://angular.dev/best-practices/security)
- [Angular accessibility](https://angular.dev/best-practices/a11y)
- [Angular component host elements and binding collisions](https://angular.dev/guide/components/host-elements)
