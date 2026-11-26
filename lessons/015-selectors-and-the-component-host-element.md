# 15. Selectors and the component host element

A component selector tells Angular which template element should instantiate the component. The matching element becomes the component's host element, and the component's view is rendered inside it. Understanding both ideas helps you choose clear selectors, inspect rendered DOM, and put attributes or bindings on the correct element.

## Learning goals

After this lesson, you should be able to:

- Explain how Angular uses a component selector.
- Recognize the selector forms supported by Angular.
- Choose a readable selector for a reusable component.
- Distinguish the component host element from elements in its template.
- Bind appropriate attributes, properties, classes, styles, and events to the host.
- Avoid selector and host-binding patterns that produce confusing or inaccessible markup.

## 1. What a selector does

A component declares a selector in its @Component metadata:

~~~
@Component({
  selector: 'app-alert',
  standalone: true,
  template: '<p>The service is available.</p>',
})
export class Alert {}
~~~

A parent template uses that component by writing an element that matches its selector:

~~~html
<app-alert />
~~~

Angular matches component selectors against template elements at compile time. It does not repeatedly query the live DOM to discover components. Adding a matching class or attribute to an element later with ordinary DOM APIs does not create an Angular component instance.

Selectors are case-sensitive. Keep their spelling consistent between component metadata and template usage.

## 2. Selector forms Angular supports

Angular supports a limited set of selector forms for components and directives:

| Form | Example | Matches |
| --- | --- | --- |
| Element name | app-alert | An element named app-alert |
| Attribute presence | [appTooltip] | An element with the appTooltip attribute |
| Exact attribute value | [type="reset"] | An element whose type attribute is exactly reset |
| Class | .app-highlight | An element with the app-highlight class |
| Negation | [appDropzone]:not(textarea) | An appDropzone element that is not a textarea |

Selectors can combine simple forms, such as button[type="reset"], or use a comma-separated list to match more than one form. Angular supports only exact attribute-value matching, not CSS attribute operators such as ^= or *=. Component selectors do not support combinators such as descendant or child selectors, namespaces, or arbitrary pseudo-classes.

The CSS selector syntax used in stylesheets is broader than the selector syntax supported in @Component metadata. Do not assume a selector valid in CSS can be used as an Angular component selector.

## 3. Choosing a component selector

Most components should use a custom element name with a hyphen, such as app-alert or shop-product-card. A consistent prefix helps distinguish application components from standard HTML elements and components from other libraries. The Angular CLI uses app- by default.

Do not use ng- as a project prefix; Angular reserves it for framework APIs.

A custom element selector is a good default when a component represents a UI region:

~~~ts
@Component({
  selector: 'app-profile-card',
  standalone: true,
  template: '<article><h2>Profile</h2></article>',
})
export class ProfileCard {}
~~~

Use an attribute selector when a component intentionally enhances a standard HTML element and should retain that element's native behavior. For example, a design-system button may match button[appAction]. Choose the host element to preserve the right native keyboard and accessibility behavior.

Attribute selectors do not give the same typo protection as unknown custom element selectors. Review their spelling and imports carefully.

## 4. The host element and the component view

The element that matches the selector is the host element. The component's own template is rendered inside that element.

Given this component:

~~~ts
@Component({
  selector: 'app-profile-card',
  standalone: true,
  template: '<article><h2>Profile</h2></article>',
})
export class ProfileCard {}
~~~

And this parent template:

~~~html
<app-profile-card />
~~~

The rendered structure conceptually contains both levels:

~~~html
<app-profile-card>
  <article>
    <h2>Profile</h2>
  </article>
</app-profile-card>
~~~

The outer app-profile-card is the host. The article and heading come from the component's template. The host may appear in the DOM even when its template is a single nested element.

The root application component also has a host element, commonly written in the workspace's index.html. A component's host is not the same as the root element inside its template.

## 5. Host attributes and accessibility

Attributes can be written on a component's host element where the component is used. A component can also define host attributes or bindings in its metadata.

~~~ts
@Component({
  selector: 'app-status-badge',
  standalone: true,
  template: '<span class="badge">Online</span>',
  host: {
    role: 'status',
    '[attr.aria-label]': 'label',
    '[class.is-online]': 'online',
  },
})
export class StatusBadge {
  readonly label = 'Service status: online';
  readonly online = true;
}
~~~

Here, the role, accessible label, and is-online class are bound to the app-status-badge host element. They do not belong to the nested span unless the template places them there.

Use semantic HTML first. When a custom host needs semantics, choose an appropriate role and accessible name. Do not add an ARIA role or label that contradicts the element's actual behavior.

## 6. Binding to the host

The host metadata object can set static attributes and define dynamic property, attribute, class, style, and event bindings. For example:

~~~ts
host: {
  '[attr.aria-expanded]': 'expanded',
  '[class.open]': 'expanded',
  '(keydown)': 'onKeydown($event)',
}
~~~

Host bindings are useful when a property or behavior belongs to the host element itself. Template bindings are often clearer for markup inside the component's view.

Avoid defining competing dynamic bindings to the same host property or attribute both on the component and on the element where the component is used. Overlapping bindings can have surprising precedence. Keep ownership clear.

The @HostBinding and @HostListener decorators exist in older code. For new code, Angular recommends the host metadata property.

## 7. Styling the host

The special :host selector in a component stylesheet refers to that component's host element. It can set layout styles for the wrapper itself:

~~~css
:host {
  display: block;
}

.badge {
  padding: 0.25rem 0.6rem;
  border-radius: 999px;
}
~~~

The .badge rule targets an element in the component template; :host targets the element matching the component selector. Host styling works with Angular's component style behavior, but it is not a security boundary.

## 8. Common mistakes

- Treating the selector as a runtime DOM query instead of a compile-time component match.
- Using a CSS descendant selector such as app-panel app-alert as a component selector.
- Using a selector with unsupported attribute operators or pseudo-classes.
- Giving project components selectors without a consistent prefix or hyphenated custom element name.
- Using Angular's reserved ng- prefix for application components.
- Confusing the component host element with the first element inside its template.
- Putting an accessibility attribute on a nested template element when it belongs to the host, or vice versa.
- Applying competing dynamic host bindings to the same property.
- Using an attribute selector and forgetting to import the component into the consumer's template scope.
- Using a custom component host where a native button, link, or form control would provide clearer built-in behavior.

## Practical checklist

- [ ] The selector clearly identifies the component and follows the project's prefix convention.
- [ ] The selector uses syntax supported by Angular.
- [ ] The consumer template includes the selector and imports the component.
- [ ] Host attributes and bindings are placed on the intended element.
- [ ] Interactive behavior preserves native HTML semantics and keyboard access.
- [ ] Host metadata and consumer bindings do not compete for the same dynamic property.
- [ ] Styles distinguish :host from elements inside the component template.

## Next lesson

Continue with [Angular templates and HTML structure](016-angular-templates-and-html-structure.md).

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="014-standalone-components-and-template-imports.md">014 - 14. Standalone components and template imports</a></td><td><a href="../qnas/015-selectors-and-the-component-host-element.md">Q&amp;A</a></td><td><a href="../examples/015-selectors-and-the-component-host-element.md">Example</a></td><td><a href="016-angular-templates-and-html-structure.md">016 - 16. Angular templates and HTML structure</a></td>
</tr>
  </tbody>
</table>
