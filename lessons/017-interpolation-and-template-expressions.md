# 17. Interpolation and template expressions

Interpolation displays a value from a component in template text. The expression inside the interpolation is evaluated in the component's template context, and Angular converts its result to text. Angular expressions resemble JavaScript, but they have their own supported syntax and should stay small and focused.

## Learning goals

After this lesson, you should be able to:

- Display component values with interpolation.
- Explain where a template expression gets its values.
- Use common literals, operators, optional chaining, and nullish coalescing.
- Recognize Angular expression syntax that differs from full JavaScript.
- Keep expressions readable and free of side effects.
- Explain how text interpolation treats nullish values and HTML-looking strings.

## 1. Text interpolation

Write an expression between double curly braces to display its value:

~~~html
<p>Welcome, {{ userName }}.</p>
~~~

If the component has a userName property with the value Mina, Angular renders the corresponding text in the paragraph. Interpolation can appear within ordinary text, and multiple interpolations can appear in one text node.

A component template might contain:

~~~html
<h1>{{ pageTitle }}</h1>
<p>{{ description }}</p>
<p>{{ itemName }} costs {{ unitPrice }}.</p>
~~~

Angular evaluates these expressions as part of rendering and updates the text when the expression's value changes and change detection runs.

## 2. The expression context

Expressions can read the component's accessible members, such as properties and methods. The component instance is the default context, so this is usually implicit:

~~~html
<p>{{ pageTitle }}</p>
~~~

This is equivalent in intent to referring to the component member explicitly with this.pageTitle.

Angular templates can also use template-local values in contexts that define them. A local name takes precedence over a component member with the same name. In current Angular, this.pageTitle can distinguish the component member when a template-local variable shadows it.

If a component property is a signal, call the signal to read its current value:

~~~html
<p>{{ count() }}</p>
~~~

A regular property is read without parentheses. A method call is written with parentheses.

## 3. Common expression forms

Angular expressions support common values and operations, including:

- String, number, boolean, and null literals.
- Array and object literals.
- Property access and method calls.
- Arithmetic, comparison, and logical operators.
- Conditional expressions using the ternary operator.
- Optional chaining with ?.
- Nullish coalescing with ??.
- Angular pipes, written with the pipe character.

For example:

~~~html
<p>{{ firstName + ' ' + lastName }}</p>
<p>{{ unitPrice * quantity }}</p>
<p>{{ isAvailable ? 'Available' : 'Unavailable' }}</p>
<p>{{ customer?.displayName ?? 'Guest' }}</p>
~~~

Keep calculations short enough to understand in the template. Put multi-step calculations and application decisions in the component or an appropriate service.

## 4. Nullish values and fallbacks

When a text interpolation evaluates to null or undefined, Angular renders an empty string. This can make a missing value appear as a blank area.

Use optional chaining when an intermediate object might be null or undefined:

~~~html
<p>{{ customer?.displayName }}</p>
~~~

Use nullish coalescing when the expression should display a fallback only for null or undefined:

~~~html
<p>{{ customer?.displayName ?? 'Guest' }}</p>
~~~

This differs from the logical OR operator. The expression value 0, false, or an empty string can be meaningful. The ?? operator keeps those values; || uses JavaScript truthiness and falls back for them too.

Starting with Angular 22, optional chaining in template expressions follows standard JavaScript behavior and returns undefined when its left side is null or undefined. Earlier Angular versions returned null for this case. Avoid code that depends on the difference between those two values where possible.

## 5. Angular expressions are not full JavaScript

Angular expressions are intentionally limited. They are not a place to write arbitrary JavaScript statements.

Angular expressions do not support many JavaScript features, including:

- Declaring variables or functions inside an expression.
- The new operator.
- Bitwise operators.
- Destructuring declarations.
- Most browser and JavaScript globals.

The expression globals Angular provides include undefined and $any. Do not expect window, document, console, Math, Number, or parseInt to be available as globals in a template expression. Put that work in TypeScript and expose the needed result through the component.

Assignment and increment/decrement belong to event-handler statements rather than interpolation and ordinary bindings. Event-handler syntax is covered in a later lesson.

## 6. Keep expressions pure and inexpensive

Expressions can be evaluated repeatedly as Angular checks a view. A method called from a template should therefore be quick and free of side effects.

Avoid expressions that:

- Mutate component state.
- Start network requests.
- Write to storage.
- Perform large computations on every check.
- Hide a multi-step decision inside nested ternaries.

For derived state, calculate it in the component or use an appropriate computed value. Keep the template focused on displaying the result.

## 7. What interpolation renders

Interpolation converts its result to a string. Null and undefined render as empty text. Other values use string conversion; for example, a plain object commonly becomes a generic string such as [object Object], which is rarely useful to a reader.

Format values deliberately instead of interpolating whole objects:

~~~html
<p>{{ customer.displayName }}</p>
~~~

Text interpolation displays a string as text. If a string contains markup-like characters, Angular escapes them for text rendering rather than interpreting them as HTML. For example, interpolating a string containing <strong>Sale</strong> shows those characters as text. Binding to an HTML property such as innerHTML is a different operation with separate security considerations.

## 8. Interpolation in attributes and properties

Interpolation can also appear in some element attributes, for example an image alt value. Angular treats that syntax as a binding:

~~~html
<img alt="Photo of {{ displayName }}">
~~~

For many dynamic element properties, bracket binding is clearer:

~~~html
<button [disabled]="!isAvailable">Continue</button>
~~~

Attribute, property, class, and style binding rules are covered in the next lesson. Do not assume every HTML attribute and DOM property behave the same way.

## Common mistakes

- Writing a component property as a JavaScript global instead of accessing it through the component context.
- Calling a signal as if it were a regular property, or adding parentheses to an ordinary property.
- Expecting null or undefined to show visible text.
- Using || when 0, false, or an empty string should remain a valid value.
- Writing a statement or variable declaration inside interpolation.
- Using browser globals such as window directly in an expression.
- Calling a slow or state-changing method during template evaluation.
- Interpolating an entire object and expecting Angular to format its fields.
- Assuming an HTML-looking string in interpolation will become real markup.
- Depending on optional chaining's null-versus-undefined result across Angular versions.

## Practical checklist

- [ ] The expression reads a value from the component or an in-scope template local.
- [ ] A signal is called to read its value.
- [ ] Nullish data has an intentional display or fallback.
- [ ] A fallback uses ?? or || according to the intended meaning.
- [ ] The expression is short, quick, and side-effect free.
- [ ] Values are formatted as useful text rather than raw objects.
- [ ] HTML-looking text is not confused with HTML property binding.
- [ ] Version-sensitive syntax is checked against the installed Angular release.

## Further reading

- [Angular expression syntax](https://angular.dev/guide/templates/expression-syntax)
- [Binding dynamic text and properties](https://angular.dev/guide/templates/binding)
- [Angular security best practices](https://angular.dev/best-practices/security)
- [Signals](https://angular.dev/guide/signals)

## Next lesson

Continue with [Property, attribute, class, and style bindings](018-property-attribute-class-and-style-bindings.md).
