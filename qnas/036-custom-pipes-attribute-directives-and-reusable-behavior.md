# 36. Q&A: Custom pipes, attribute directives, and reusable behavior

## Custom pipe fundamentals

### 1. What is a custom Angular pipe?

A custom pipe is a class decorated with `@Pipe`. Its template name is used after `|`, and its `transform()` method receives the value plus any arguments. It prepares a value for display; it should not change the component's source data.

### 2. What is the difference between the pipe name and the class name?

The pipe name is the lower-camel-case identifier used by templates, such as `truncate`. The class is a TypeScript symbol, conventionally named in PascalCase with a `Pipe` suffix, such as `TruncatePipe`.

### 3. What does `PipeTransform` do?

`PipeTransform` describes the expected `transform()` method shape. Implementing it helps TypeScript check that the class exposes the method Angular expects. It does not itself register or make the pipe available in a template.

### 4. How do I pass arguments to a custom pipe?

Separate the input and each argument with colons. For example, `{{ description | truncate:60:'…' }}` calls the pipe with the description, `60`, and the suffix. Declare the arguments in the same order in `transform()`.

### 5. Can a pipe argument have a default value?

Yes. Give the parameter a TypeScript default, such as `transform(value: string, maxLength = 40)`. Angular uses that default when the corresponding template argument is omitted.

### 6. How should a custom pipe handle `null` or `undefined`?

Choose and document a display policy. It might return an empty string, a placeholder, or `null`. Use a type such as `string | null | undefined` if those values are valid inputs; do not silence the type checker with a non-null assertion without a reason.

### 7. How do I use a standalone custom pipe in a standalone component?

Import the pipe class in the component file and add it to that component's `imports` array. A standalone pipe is not automatically available in every component template.

### 8. How does a non-standalone pipe become available in an NgModule app?

Declare it in exactly one NgModule. Export the declaring module if components in other modules need to use the pipe. A standalone pipe can instead be imported by a standalone component or an NgModule.

### 9. Are custom pipes pure by default?

Yes. Angular treats a pipe as pure unless it is configured with `pure: false`. Pure pipes are checked when Angular detects that the input or one of the pipe arguments changed.

### 10. Why does a pure pipe not update after I mutate an array in place?

The array reference is unchanged, so Angular does not see a changed argument for the pure pipe. Replace the array instead, for example `this.items = [...this.items, newItem]`, so the new reference makes the update explicit.

### 11. Does a pure pipe detect a mutated object or `Date` instance?

Not when the same object reference is passed again. Prefer immutable updates, such as creating a new object or `Date`, rather than mutating the existing instance and expecting the pipe to run.

### 12. When should I make a custom pipe impure?

Only when it truly needs to observe changes that are not represented by its arguments, and after considering a clearer alternative. An impure pipe can run during every change-detection pass, so its work must be very small and predictable.

### 13. Should `transform()` make HTTP calls, subscribe, or update component state?

No. A pipe should behave like a presentation transformation. Keep I/O and side effects in a service or component workflow. Use Angular's `AsyncPipe` to render an Observable or Promise instead of creating a custom pipe that subscribes.

### 14. Why extract a plain function instead of injecting a pipe into a service?

A pipe is designed for template use. A pure function can be called by the pipe, a service, or a component without coupling those TypeScript consumers to Angular's template-pipe API. It is also easier to reuse and reason about independently.

### 15. What should a truncation pipe do with a negative or invalid maximum length?

Define a deliberate rule and test the boundary cases. For example, the lesson's helper clamps negative limits to zero and treats a non-finite limit as zero. A different product may reject invalid configuration, but it should not produce accidental output silently.

### 16. Can `String.slice()` split an emoji when truncating text?

Yes. JavaScript string indexes count UTF-16 code units, not user-perceived grapheme clusters. For exact user-facing character limits across emoji and combining marks, use a grapheme-aware strategy such as `Intl.Segmenter` where the supported runtimes allow it.

### 17. Can I chain custom pipes with built-in pipes?

Yes. A pipe's output can be passed to the next pipe, for example `{{ title | titlecase | truncate:50 }}`. Pipes evaluate from left to right, so each pipe should accept the previous pipe's result type.

### 18. Should a custom pipe format domain data or replace its stored value?

Usually neither. Keep domain data in its meaningful type and use a pipe to format it at the template boundary. For example, a currency pipe formats a number for display without replacing the numeric amount in application state.

## Attribute directives

### 19. What does an attribute directive do?

It attaches reusable behavior or appearance to an element that already exists. It does not define its own template. A selector such as `[appHighlight]` matches Angular templates where that attribute is present.

### 20. How is an attribute directive different from a component?

A component has its own view and template. An attribute directive enhances an existing host element or component host. If the behavior needs a reusable UI view, a component is usually the clearer abstraction.

### 21. How is an attribute directive different from a structural directive?

An attribute directive adds behavior or presentation to an existing host. A structural directive controls whether or how template content is instantiated. Choose based on whether the DOM structure itself must change.

### 22. How do I configure a directive with an input?

Declare an input on the directive and bind it where the selector is used. A common signal-input pattern uses the same name for the selector and primary input: `selector: '[appHighlight]'` with `readonly appHighlight = input('yellow')`, then `<p [appHighlight]="color">`.

### 23. Why use the selector name for a directive input?

It allows a bare attribute such as `<p appHighlight>` to match the selector and use a default, while `[appHighlight]="color"` both matches the selector and configures it. Other input names can expose additional options.

### 24. What does the `host` property in `@Directive` do?

It declares bindings and event listeners on the directive's host. For example, it can bind a host class or style to a signal and listen for an event without manually assigning DOM properties.

### 25. Can a host event binding access the browser event?

Yes. Use `$event` in the event expression or delegate to a method that accepts the event. Prefer a typed method when the handler needs branching or multiple statements.

### 26. Should new directives use `@HostListener` and `@HostBinding`?

For new code, Angular recommends the declarative `host` metadata option. The decorators remain supported for compatibility, but `host` keeps related host bindings together in the directive metadata.

### 27. When should I write a directive instead of a direct template binding?

Use a direct binding for a one-off behavior in one component. Consider a directive when the same behavior needs to be applied consistently in many places or to unrelated host elements. Avoid an abstraction that only hides one simple class or style binding.

### 28. Can a directive apply to an Angular component element?

Yes. An attribute directive can match a native element or a component host element. When used on a component, the directive attaches to that component's host; it does not directly modify the component's internal template.

### 29. How do I make a standalone directive available in a standalone component?

Import it in the component's `imports` array, then apply its selector in the template. For an NgModule-based app, import a standalone directive or declare a non-standalone directive in one NgModule and export it when needed elsewhere.

### 30. Should a hover directive listen only for mouse events?

Not if it represents an interaction that keyboard users must be able to use. Add the appropriate focus and keyboard behavior or use a native interactive element. For a purely decorative hover effect, do not imply that the element is interactive and keep visible focus styles intact.

### 31. Is it safe to set `innerHTML` from a directive?

Do not write untrusted strings into `innerHTML`. Prefer Angular template bindings for text, properties, attributes, classes, and styles. Low-level DOM access can bypass Angular's usual safety and should be limited to cases that need it and handled carefully.

### 32. Why should custom directive selectors have a prefix?

A prefix such as `app` lowers the chance of colliding with native attributes or another library's directive selector. Angular matches selectors during compilation; a selector is not a runtime CSS query string.

## Composing behavior with `hostDirectives`

### 33. What is `hostDirectives`?

It is the directive composition API. A component can apply standalone directives to its own host from its decorator, so the behavior is always present wherever that component is used.

### 34. Are a host directive's inputs and outputs public automatically?

No. They are private to the composed component unless explicitly exposed in `hostDirectives`. Expose and alias only the inputs and outputs that should be part of the component's public API.

### 35. Can `hostDirectives` be added dynamically at runtime?

No. Composition is declared statically in component metadata and is resolved as Angular compiles the component. Use a directive selector in the consuming template when the consumer needs to choose where the behavior applies.

### 36. When should I use a template selector versus `hostDirectives`?

Use a selector when a template author decides whether an element should receive the behavior. Use `hostDirectives` when the behavior is an implementation detail that every instance of a particular component should have.

### 37. Must a directive composed through `hostDirectives` be standalone?

Yes. Host directives must be standalone. They can expose selected inputs and outputs, but they do not become a new template or an independently rendered view.

## Troubleshooting and design

### 38. Why do I get an unknown pipe or directive error?

Check the template's compilation scope. Add a standalone pipe or directive to the using standalone component's `imports`, or declare/import it through the correct NgModule arrangement. Also check spelling and selector names.

### 39. What is a good way to organize reusable pipe and directive logic?

Keep the pipe and directive focused on their Angular integration. Move reusable transformations into plain functions and keep behavior configuration in typed inputs. This avoids duplicating logic and keeps each abstraction's responsibility clear.

### 40. Where can I find a complete working example for this chapter?

See [Example 36](../examples/036-custom-pipes-attribute-directives-and-reusable-behavior.md). The concepts and trade-offs are explained in [Lesson 36](../lessons/036-custom-pipes-attribute-directives-and-reusable-behavior.md).

## Further reading

- [Angular pipes guide](https://angular.dev/guide/templates/pipes)
- [Pipe API](https://angular.dev/api/core/Pipe)
- [Attribute directives guide](https://angular.dev/guide/directives/attribute-directives)
- [Directive composition API](https://angular.dev/guide/directives/directive-composition-api)