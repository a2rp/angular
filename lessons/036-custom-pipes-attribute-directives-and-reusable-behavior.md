# 36. Custom pipes, attribute directives, and reusable behavior

Angular gives you two useful ways to package presentation behavior: **custom pipes** transform values for a template, and **attribute directives** add behavior or appearance to an existing host element. A plain TypeScript function is often the reusable core behind a custom pipe. Use each tool when its responsibility fits; simple one-off bindings do not need a new abstraction.

## Learning goals

After this lesson, you should be able to:

- Create a custom pipe with `@Pipe`, a template name, and a typed `transform()` method.
- Give pipe arguments clear defaults and decide how null or invalid input should display.
- Understand pure pipes, immutable updates, and why impure pipes need restraint.
- Extract a reusable transformation function that can be called outside a template.
- Create an attribute directive with `@Directive` and apply it through an attribute selector.
- Use signal inputs and the `host` metadata for host classes, styles, attributes, and event listeners.
- Decide when a direct template binding is enough and when behavior deserves a reusable directive.
- Compose a directive into a component with `hostDirectives` when the behavior is an implementation detail of that component.

## Start with a clear responsibility

Use a **pipe** when a value needs a repeatable, synchronous presentation transformation in a template, such as turning a long label into a short preview.

Use an **attribute directive** when an existing element or component should gain reusable behavior or appearance, such as consistent host classes or hover styling.

Use a **plain function** for transformation logic that should also be used in services, event handlers, or other TypeScript code. Let a pipe call that function when templates need the same transformation.

Use Angular's existing **class, style, property, attribute, and event bindings** for behavior that is only needed once. Creating a directive for one small binding can add unnecessary indirection.

A component defines reusable UI with its own template. A structural directive changes which DOM elements exist. An attribute directive attaches behavior to an element that already exists.

## Create a custom pipe

A custom pipe is a TypeScript class decorated with `@Pipe`. Its `name` is the identifier used in a template and its `transform()` method receives the input value followed by any pipe arguments.

~~~ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'truncate',
  standalone: true,
  pure: true,
})
export class TruncatePipe implements PipeTransform {
  transform(value: string | null | undefined, maxLength = 40): string {
    if (value == null) {
      return '';
    }

    const limit = Number.isFinite(maxLength)
      ? Math.max(0, Math.floor(maxLength))
      : 0;

    if (value.length <= limit) {
      return value;
    }

    return `${value.slice(0, limit).trimEnd()}…`;
  }
}
~~~

Use it in a template like any built-in pipe:

~~~html
<p>{{ description | truncate:60 }}</p>
~~~

Pipe names use lower camel case and should not contain hyphens. A common class-name convention is PascalCase plus the `Pipe` suffix: `TruncatePipe` for the `truncate` pipe. Implementing `PipeTransform` helps TypeScript check the expected method shape.

The sample chooses to display null or undefined text as an empty string. That is a product decision: another pipe may intentionally return `null`, an em dash, or a localized placeholder. Define the behavior instead of letting a non-null assertion hide it.

### Pipe arguments

Each argument after the input is separated by a colon and is passed to `transform()` in the same order:

~~~html
<p>{{ description | truncate:60:'…' }}</p>
~~~

~~~ts
transform(
  value: string | null | undefined,
  maxLength = 40,
  suffix = '…',
): string {
  // transform the value using maxLength and suffix
}
~~~

Keep arguments simple and typed. If a pipe needs a large configuration object or many unrelated options, use a clearly named function or prepare the view value in the component instead.

## Make a custom pipe available

### Standalone components

Import the pipe class and list it in the component's `imports` array:

~~~ts
@Component({
  standalone: true,
  imports: [TruncatePipe],
  template: `{{ description | truncate:60 }}`,
})
export class PreviewComponent {
  description = 'A long description that needs a short preview.';
}
~~~

A standalone pipe must be imported by each standalone component template that uses it. It is not automatically available across the application.

### NgModule-based applications

A non-standalone pipe must be declared in exactly one NgModule before that module can make it available to templates. A standalone pipe can be imported by an NgModule. Follow the conventions used by the Angular version and architecture of the project; do not declare one pipe in several modules.

## Share transformation logic outside templates

A pipe is a template feature, not a service. If TypeScript code also needs the same transformation, put the logic in a plain function and call it from both the pipe and the TypeScript consumer:

~~~ts
// truncate-text.ts
export function truncateText(
  value: string | null | undefined,
  maxLength = 40,
  suffix = '…',
): string {
  if (value == null) {
    return '';
  }

  const limit = Number.isFinite(maxLength)
    ? Math.max(0, Math.floor(maxLength))
    : 0;

  if (value.length <= limit) {
    return value;
  }

  return `${value.slice(0, limit).trimEnd()}${suffix}`;
}
~~~

~~~ts
// truncate.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';
import { truncateText } from './truncate-text';

@Pipe({ name: 'truncate', standalone: true })
export class TruncatePipe implements PipeTransform {
  transform(
    value: string | null | undefined,
    maxLength = 40,
    suffix = '…',
  ): string {
    return truncateText(value, maxLength, suffix);
  }
}
~~~

The helper function can be called directly by a service or component without injecting the pipe. This sample treats a non-finite `maxLength` as zero and clamps a negative limit to zero. Keep such functions deterministic: the same arguments should produce the same result and should not perform side effects.

This example's `String.slice()` counts UTF-16 code units and may split a combined emoji or other multi-code-unit grapheme. If truncation must preserve user-perceived characters in multilingual text, implement a grapheme-aware policy, for example with `Intl.Segmenter` where the supported runtime and TypeScript libraries allow it.

## Pure and impure custom pipes

Pipes are pure by default. A pure pipe runs when Angular detects a change to one of its arguments: a new primitive value or, for objects and arrays, a new reference. A pure pipe will not notice an in-place mutation to an object or array whose reference stays the same.

~~~ts
// Replace the reference so pure consumers can detect the update.
this.names = [...this.names, newName];

// This mutates the existing array reference.
this.names.push(newName);
~~~

Prefer pure pipes for deterministic transformations. Angular can skip the transform while the arguments remain unchanged, which keeps presentation work predictable.

Setting `pure: false` makes a pipe run during change detection even when its arguments have not changed. This can be expensive because change detection may happen frequently. Avoid impure custom pipes unless the behavior truly requires observing state beyond the arguments, and keep any such work very small. Do not use an impure pipe just to compensate for mutable application state when an immutable update would make the change explicit.

A pipe should not perform I/O, change component state, subscribe to a stream, or mutate the input. Use `AsyncPipe` for Observable/Promise rendering and put side effects in an appropriate service or component workflow.

## Attribute directives

An attribute directive is a class decorated with `@Directive`. Its selector identifies where Angular should instantiate it. A selector in square brackets matches an attribute on an element:

~~~ts
@Directive({
  selector: '[appHighlight]',
  standalone: true,
})
export class HighlightDirective {}
~~~

~~~html
<p appHighlight>This element gets the directive's behavior.</p>
~~~

Use a project prefix such as `app` in custom selectors to reduce collisions. Directive selectors are Angular template selectors; they are not runtime CSS query strings and do not support namespaces.

A directive can apply to a native element or an Angular component host. The reusable behavior is defined once and is activated wherever the selector appears.

In a standalone component, import the standalone directive in that component's `imports` array, just like a standalone pipe. In a module-based app, import a standalone directive or declare a non-standalone directive in one NgModule and export it when other modules need it.

### Prefer a direct binding for one-off behavior

Angular template syntax already handles a one-off class, style, property, attribute, or event:

~~~html
<button
  [class.is-selected]="isSelected"
  [attr.aria-pressed]="isSelected"
  (click)="toggleSelected()"
>
  Select
</button>
~~~

If this behavior must be applied consistently to many unrelated elements, consider a directive. If it belongs to one component's own view, a normal component binding or component method is usually clearer.

### Use inputs to configure behavior

Like components, directives can accept inputs. Modern Angular supports signal inputs through `input()`:

~~~ts
import { Directive, input } from '@angular/core';

@Directive({
  selector: '[appHighlight]',
  standalone: true,
})
export class HighlightDirective {
  // The selector and input share a name.
  readonly appHighlight = input('#fff3bf');
}
~~~

~~~html
<p [appHighlight]="highlightColor">Configured highlight</p>
~~~

The binding both matches the selector and supplies the input value. A directive may expose additional inputs for options such as text color or a default value. Choose names that explain the behavior and provide defaults when the directive should work with a bare attribute.

## Bind the directive's host declaratively

The `host` property in `@Directive` metadata connects host properties, attributes, classes, styles, and events to class members:

~~~ts
import { Directive, input, signal } from '@angular/core';

@Directive({
  selector: '[appHighlight]',
  standalone: true,
  host: {
    '(mouseenter)': 'isHovered.set(true)',
    '(mouseleave)': 'isHovered.set(false)',
    '[class.is-highlighted]': 'isHovered()',
    '[style.background-color]': 'isHovered() ? appHighlight() : null',
  },
})
export class HighlightDirective {
  readonly appHighlight = input('yellow');
  protected readonly isHovered = signal(false);
}
~~~

`host` bindings are declarative and Angular updates the host element when their values change. For event handlers, `$event` is available in the expression when needed. Prefer host metadata over the older `@HostBinding` and `@HostListener` decorators for new code; those decorators remain for compatibility.

Prefer Angular bindings over directly assigning arbitrary DOM properties. Do not use `ElementRef.nativeElement.innerHTML` with untrusted values. If a low-level DOM operation is unavoidable, constrain it carefully and use Angular's supported abstractions rather than bypassing sanitization or accessibility requirements.

### Keep directive behavior accessible

Do not make an interaction depend only on a mouse event. Add keyboard and focus behavior when the directive's purpose requires it, or use a native control that already has the correct keyboard semantics. A style-only hover directive should not pretend to be an interactive control. Keep focus indicators visible and do not remove accessible names or states.

## Compose behavior with host directives

The directive composition API lets a component apply standalone directives to its own host from its decorator:

For a concrete example, this directive binds an input to the host's accessible busy state:

~~~ts
import { Directive, input } from '@angular/core';

@Directive({
  selector: '[busyState]',
  standalone: true,
  host: {
    '[attr.aria-busy]': 'busy()',
  },
})
export class BusyStateDirective {
  readonly busy = input(false);
}
~~~

~~~ts
import { Component } from '@angular/core';
import { BusyStateDirective } from './busy-state.directive';

@Component({
  selector: 'app-save-button',
  standalone: true,
  hostDirectives: [
    {
      directive: BusyStateDirective,
      inputs: ['busy: isBusy'],
    },
  ],
  template: `<button type="button">Save</button>`,
})
export class SaveButtonComponent {}
~~~

~~~html
<app-save-button [isBusy]="saving"></app-save-button>
~~~

The host directive is applied statically when Angular compiles the component. Its input is private by default; the component deliberately exposes it as `isBusy`. Outputs can be exposed and aliased through the corresponding `outputs` option when the component should make them public.

Host directives must be standalone and cannot be added dynamically at runtime through `hostDirectives`. Use ordinary template selectors when the consuming template should choose whether to apply a directive. Use composition when a component always needs the behavior as part of its implementation.

## Common mistakes

- Creating a pipe but forgetting to import it into a standalone component or declare/import it in an NgModule.
- Naming a pipe with a hyphen or using a class name that does not make its purpose clear.
- Assuming a pure pipe detects an in-place array, object, or date mutation.
- Marking a pipe impure just to avoid immutable updates.
- Putting side effects, network access, or subscriptions inside `transform()`.
- Injecting a pipe into a service instead of extracting its logic into a reusable function.
- Writing a directive for a one-off class or style binding.
- Using an overly broad directive selector that may collide with another library.
- Mutating the DOM directly when a host class, style, property, or event binding would express the behavior.
- Using `@HostListener` or `@HostBinding` in new code instead of the `host` metadata option.
- Making a directive respond only to mouse input when keyboard or focus behavior is required.
- Assuming `hostDirectives` inputs and outputs are public automatically.
- Assuming `hostDirectives` can be attached dynamically after compilation.

## Key takeaways

- A pipe transforms a value for template presentation; a directive adds reusable behavior to an existing host.
- Prefer small, typed, pure pipes and direct template bindings for one-off behavior.
- Extract shared transformation logic into a plain function when TypeScript code also needs it.
- Use signal inputs and `host` metadata to configure a directive and bind host behavior declaratively.
- Design directive interactions with native semantics, keyboard support, and visible focus in mind.
- Use `hostDirectives` to compose behavior that belongs to a component's implementation, exposing only intentional inputs and outputs.

## Further reading

- [Angular pipes guide](https://angular.dev/guide/templates/pipes)
- [Pipe API](https://angular.dev/api/core/Pipe)
- [Attribute directives guide](https://angular.dev/guide/directives/attribute-directives)
- [Directive overview](https://angular.dev/guide/directives)
- [Host elements and host bindings](https://angular.dev/guide/components/host-elements)
- [Directive composition API](https://angular.dev/guide/directives/directive-composition-api)
