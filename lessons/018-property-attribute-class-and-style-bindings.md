# 18. Property, attribute, class, and style bindings

Angular bindings connect a component value to something in the rendered view. Square-bracket syntax can target a DOM property, a component or directive input, or an explicit HTML attribute. Angular also provides focused syntax for dynamic classes and styles.

## Learning goals

After this lesson, you should be able to:

- Bind a value to a DOM property or component input.
- Explain when to use an explicit attribute binding with attr.
- Distinguish a DOM property from the corresponding HTML source attribute.
- Add or remove classes with class bindings.
- Set individual or multiple CSS styles, including units.
- Understand null handling, binding precedence, and immutable class/style values.
- Use Angular's security handling appropriately for bound values.

## 1. Property binding

Use square brackets to bind an expression to a target property:

~~~html
<button [disabled]="isSaving">Save</button>
~~~

This sets the disabled property of the button's DOM object. Angular reevaluates the expression and updates the target when change detection detects a new value.

Property bindings can target:

- A native DOM property, such as disabled, value, or title.
- An input of an Angular component.
- An input of an Angular directive.

The target determines what the binding means. For example, on a native button, disabled refers to a DOM property; on a custom component, a matching input can receive the value.

## 2. HTML attributes and DOM properties

An HTML attribute belongs to the markup and is represented in the DOM as an attribute. A DOM property belongs to the live JavaScript object for an element. Browsers often initialize a property from an attribute, but the two can later differ.

For example, an input's value attribute can describe its initial value, while the input element's value property represents its current value after user interaction. Use a property binding when you intend to update the live property:

~~~html
<input [value]="searchText">
~~~

Use attr. when you specifically need to set or remove an HTML attribute:

~~~html
<td [attr.colspan]="columnCount">Total</td>
<p [attr.aria-label]="accessibleDescription">Details</p>
~~~

The attr. prefix tells Angular to set the attribute by name. If the bound value is null or undefined, Angular removes the attribute.

Prefer a property binding for native properties such as disabled. Prefer an attribute binding when the target is an attribute that does not have a corresponding DOM property or when you specifically need attribute semantics.

## 3. Boolean attributes

HTML boolean attributes are generally true when present and false when absent. Their text value is not treated like an ordinary string boolean. For native controls, bind the boolean DOM property:

~~~html
<button [disabled]="isSaving">Save</button>
~~~

If you intentionally need to control the attribute itself, use attr. and provide either a value or null:

~~~html
<button [attr.disabled]="isSaving ? '' : null">Save</button>
~~~

Do not write [attr.disabled]="false" expecting that to remove the attribute. The string false is still a present attribute. With the property binding, pass the boolean directly.

## 4. Binding attributes for accessibility and SVG

Use attribute bindings for values such as ARIA attributes or SVG attributes when that is the intended target:

~~~html
<button
  [attr.aria-expanded]="isExpanded"
  [attr.aria-label]="actionLabel"
>
  Details
</button>
~~~

ARIA attributes are string-valued. Binding null or undefined removes the attribute. Use the correct native element and accurate accessible text; a binding cannot make an inappropriate control accessible by itself.

## 5. Class bindings

Use a single-class binding to add a class when its expression is truthy and remove it when false:

~~~html
<article [class.featured]="isFeatured">
  ...
</article>
~~~

Use the class binding to supply a set of classes:

~~~html
<section [class]="classNames">
  ...
</section>
~~~

The value can be a whitespace-separated string, an array of class names, or an object whose keys are class names and whose truthy values enable those classes.

~~~ts
classNames = {
  featured: true,
  archived: false,
};
~~~

Static class attributes, the [class] binding, and individual [class.name] bindings can be combined. Angular merges classes, but does not guarantee a particular order in the rendered class attribute.

For object or array bindings, Angular checks whether the reference changed. Create a new object or array when changing its contents; mutating the existing object in place may not trigger the class update.

Use specific class bindings when possible. For more advanced class handling, Angular also provides NgClass, but the built-in class bindings cover common cases.

## 6. Style bindings

Bind one CSS property with style.property syntax:

~~~html
<div [style.width.px]="panelWidth">
  ...
</div>
~~~

The optional unit suffix adds the unit to a numeric value. Other examples include style.height.rem or style.opacity.

Bind several CSS declarations at once with [style]. The value can be a CSS string or an object:

~~~html
<div [style]="panelStyles">
  ...
</div>
~~~

~~~ts
panelStyles = {
  'max-width': '40rem',
  color: '#172033',
  padding: '1rem',
};
~~~

As with class objects, replace an object with a new object when its contents change. Angular compares the current style object with the previous object by reference.

Prefer ordinary CSS classes for stable design rules, and use style bindings for values that genuinely depend on component data. For example, a class can define a card layout while a style binding supplies a dynamic width.

## 7. Binding precedence

An element can have static classes or styles and several dynamic bindings. Angular merges class and style values and applies documented precedence when multiple bindings target the same class or style property.

Avoid assigning the same class or style property from multiple dynamic sources. A clearer design gives each dynamic value one owner. For non-style properties and attributes, overlapping static and dynamic bindings can also lead to confusing results.

When a component host also defines its own host bindings, the template that uses the component and the component metadata can both affect the host. Keep those bindings from competing for the same dynamic target.

## 8. Security and bound values

Angular treats values supplied to template bindings as untrusted and applies security handling based on the target context. HTML, URL, style, and resource URL contexts have different risks and rules.

Do not bypass Angular sanitization just to make a value appear. In particular, do not trust user-controlled HTML, URLs, or resource URLs. Avoid direct DOM manipulation when a normal Angular binding can express the same view update.

## Common mistakes

- Using [attr.disabled]="false" and expecting the disabled attribute to disappear.
- Binding to an HTML attribute when the live DOM property is the intended target.
- Assuming every HTML attribute has a same-named DOM property.
- Forgetting the attr. prefix when the target is explicitly an attribute.
- Mutating a bound class or style object in place instead of replacing it.
- Expecting a class attribute's token order to remain fixed after Angular merges bindings.
- Writing the same dynamic class or style property from multiple sources.
- Using inline style bindings for stable design rules that belong in CSS.
- Assuming Angular sanitizes every kind of bound value in the same way.
- Using a bypass trust API for untrusted data.

## Practical checklist

- [ ] Identify whether the binding target is a DOM property, component input, directive input, or HTML attribute.
- [ ] Use property syntax for live DOM properties and attr. for explicit attributes.
- [ ] Bind native boolean properties with booleans.
- [ ] Use class and style binding syntax for dynamic presentation.
- [ ] Replace class/style arrays or objects instead of mutating them in place.
- [ ] Keep each dynamic class or style property under one clear owner.
- [ ] Treat bound values as untrusted and preserve Angular's security handling.

## Further reading

- [Binding dynamic text, properties, and attributes](https://angular.dev/guide/templates/binding)
- [Angular security best practices](https://angular.dev/best-practices/security)
- [Angular accessibility practices](https://angular.dev/best-practices/a11y)
- [Component host elements and binding precedence](https://angular.dev/guide/components/host-elements)
- [Angular style guide](https://angular.dev/style-guide)

## Next lesson

Continue with [Event bindings, event objects, and keyboard input](019-event-bindings-event-objects-and-keyboard-input.md).
