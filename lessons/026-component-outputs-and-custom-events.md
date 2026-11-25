# 26. Component outputs and custom events

A component input carries a value from a parent into a child. A component output communicates an event from a child back to the parent. Outputs let a reusable component report that something meaningful happened while leaving the parent in control of application state.

For new Angular components, declare outputs with the `output()` function from `@angular/core`. It returns an `OutputEmitterRef<T>`: call `.emit(value)` to notify listeners. An output is not a signal and is not read by calling it.

~~~ts
import { Component, output } from '@angular/core';

@Component({
  selector: 'app-save-button',
  standalone: true,
  template: '<button type="button" (click)="save()">Save</button>',
})
export class SaveButton {
  readonly saved = output<string>();

  save(): void {
    this.saved.emit('Draft saved');
  }
}
~~~

A parent listens on the child component element with Angular event-binding syntax:

~~~html
<app-save-button (saved)="showMessage($event)" />
~~~

The value passed to `emit()` becomes `$event` in the parent listener. The child announces an event; the parent decides what to do with it.

## Learning goals

After this lesson, you should be able to:

- Declare a typed output with `output<T>()` and emit it with `.emit()`.
- Bind to an output in a parent template and use its `$event` payload.
- Choose useful payloads and output names for a component API.
- Explain how outputs differ from inputs and native DOM events.
- Use aliases and programmatic output subscriptions where they fit.
- Recognize the supported decorator-based output API in existing code.

## The direction of communication

Inputs and outputs describe opposite directions across a component boundary:

| Mechanism | Direction | Typical purpose |
| --- | --- | --- |
| Input | Parent → child | Give a child the data or configuration it needs |
| Output | Child → parent | Tell the parent that a user action or child-level event occurred |

The child should not reach into its parent's fields. Instead, it emits a well-defined event and lets the parent respond. This keeps the child reusable in different screens and makes ownership of application state clear.

An output does not update anything automatically. Declaring `saved` does not save data, and emitting it does not change the parent's state unless a listener handles the event. The output is the communication contract; the parent supplies the behavior.

## Declare outputs with `output()`

Import `output` from `@angular/core` and declare it as a component or directive class property initializer:

~~~ts
import { output } from '@angular/core';

export class SearchBox {
  readonly searchRequested = output<string>();
}
~~~

The generic type describes the event payload. It gives `.emit()` and template listeners useful type information:

~~~ts
readonly pageChanged = output<number>();
readonly itemSelected = output<{ id: number; label: string }>();
readonly closed = output<void>();
~~~

Use `output<void>()` for an event with no payload. Emit it with no argument:

~~~ts
this.closed.emit();
~~~

For data-bearing events, define a named type when it improves readability or the same shape is shared across files:

~~~ts
interface SelectionChange {
  id: string;
  selected: boolean;
}

readonly selectionChanged = output<SelectionChange>();
~~~

The `output()` function is an Angular compiler-recognized API and belongs in a component or directive property initializer. Do not create outputs by calling `output()` inside a constructor, method, or conditional branch. Prefer `readonly` so the output reference itself cannot accidentally be reassigned.

## Emit the event

Call `.emit()` at the point where the child has completed the action it reports:

~~~ts
export class SearchBox {
  readonly searchRequested = output<string>();
  query = '';

  submit(): void {
    const normalizedQuery = this.query.trim();
    if (normalizedQuery.length > 0) {
      this.searchRequested.emit(normalizedQuery);
    }
  }
}
~~~

An output is an event notification, not a two-way binding by itself. If a child needs a current value from its parent, give it an input as well. Do not emit on every internal change unless that is the documented meaning of the output; choose an event name that describes when the event occurs.

`OutputEmitterRef` is a small Angular output API, not an RxJS `Subject` or `EventEmitter`. Use `.emit()`, not `.next()`, and do not expect RxJS completion or error channels. Keep RxJS streams as streams and expose an output when a component API needs to notify its template consumer.

## Listen in the parent template

Use parentheses around the output name, just as for a native event binding:

~~~html
<app-search-box (searchRequested)="runSearch($event)" />
~~~

Angular calls the handler when the child emits. `$event` is the emitted value. A parent can pass that value to a method:

~~~ts
runSearch(query: string): void {
  this.searchService.search(query);
}
~~~

Or use a short expression for a simple state update:

~~~html
<app-counter (incremented)="total = total + $event" />
~~~

Keep larger behavior in a parent method. This makes the template easier to scan and gives the handler a natural place for validation, state updates, and follow-up work.

The listener must be attached to a component or directive instance that declares the output. Angular outputs do not bubble through the DOM. If an ancestor needs to react to a nested child's event, an intermediate component can listen to the child and explicitly emit its own output, or the components can communicate through an appropriate shared service or state owner.

## Choose event names and payloads

Output names are case-sensitive. Use camelCase, avoid an `on` prefix, and choose a name that describes the semantic event:

~~~ts
readonly saved = output<SaveResult>();
readonly selectionChanged = output<SelectionChange>();
readonly dismissed = output<void>();
~~~

Names such as `saved`, `selectionChanged`, and `dismissed` describe what happened. Names such as `handleSave` describe an implementation detail, while a generic `click` can be confused with a native DOM event.

Avoid naming an output after a native event such as `click`, `input`, or `change`. If a component output and a native event share a name, template consumers can have difficulty telling which event they are binding to. Prefer a distinct component-level name such as `valueChanged` or `actionRequested`.

For payloads, send the smallest useful, stable description of the event. Use a typed object when several related values belong together. Do not send an entire component instance or rely on a listener knowing the child's private state. Document whether the payload is a value, a snapshot, an identifier, or a domain event.

## Alias an output when a public name must differ

An output may have a public template name that differs from its TypeScript property name:

~~~ts
readonly saved = output<SaveResult>({ alias: 'recordSaved' });
~~~

The TypeScript property remains `saved`, while a parent binds to `(recordSaved)`. Aliases can preserve an existing public API during a rename or avoid a collision with a native event name. Avoid aliases when there is no compatibility or naming need; having one name in TypeScript and another in templates adds a mapping readers must remember.

## Subscribe programmatically

Most components listen in templates. For dynamically created components, Angular also lets code subscribe to an output reference:

~~~ts
const subscription = componentRef.instance.saved.subscribe((result) => {
  this.handleSave(result);
});
~~~

The returned subscription has an `unsubscribe()` method. Angular automatically cleans up an output subscription when the component that owns the output is destroyed. Unsubscribe explicitly if the listener should stop earlier than that.

Programmatic subscription does not change the event contract: the child still emits a typed value with `.emit()`, and the consumer decides what to do with it.

## Inherited and decorator-based outputs

A component inherits outputs declared by its base class. This can be useful for a shared component contract, but keep inheritance intentional so consumers can still discover which events a concrete component exposes.

Existing applications may use the decorator API:

~~~ts
@Output() saved = new EventEmitter<SaveResult>();
~~~

It remains supported. For new code, Angular's `output()` API is the recommended signal-era declaration style. The official migration schematic can convert eligible decorator outputs:

~~~sh
ng generate @angular/core:output-migration
~~~

Review generated changes before adopting them, especially where an output is part of a public library API. Do not mix APIs without a reason in the same component.

## Common mistakes

- **Calling the output like a signal:** `saved()` is not how an output is read. Outputs notify listeners; call `saved.emit(value)` to send an event.
- **Using `.next()`:** `OutputEmitterRef` uses `.emit()`. It is not a `Subject` or `EventEmitter`.
- **Declaring `output()` in a method:** declare it in a class property initializer so Angular can recognize it.
- **Expecting DOM bubbling:** outputs are delivered to listeners on the component/directive that declares them; they do not bubble through ancestors.
- **Emitting an untyped or oversized payload:** define the event data shape and send only what consumers need.
- **Naming an output `click`:** use a distinct semantic name to avoid confusion with the browser's native click event.
- **Expecting an output to mutate parent state automatically:** add a parent listener and update state there.
- **Adding aliases without a reason:** keep the TypeScript property and template binding name aligned unless compatibility or collision avoidance calls for an alias.

## Practical checklist

Before exposing a component output, check that:

- The property uses `output<T>()` with the intended payload type.
- The output is declared as a class property initializer.
- The output name describes an event and is distinct from native DOM event names.
- The child emits only at the intended point and sends a documented payload.
- The parent binds to the correct component and handles `$event` deliberately.
- Any alias is needed and consistently used.
- A programmatic subscription is unsubscribed if it must end before owner destruction.

## Further reading

- [Component outputs](https://angular.dev/guide/components/outputs)
- [The `output` API](https://angular.dev/api/core/output)
- [Migrate to the `output()` function](https://angular.dev/reference/migrations/outputs)
