# 19. Event bindings, event objects, and keyboard input

Event bindings let a template respond to browser events. Angular uses parentheses around an event name and runs a statement when that event occurs:

~~~html
<button (click)="saveTask()">Save</button>
~~~

A listener can call a component method, pass the event object through the special $event variable, or use Angular's keyboard modifiers to listen for a particular key. Prefer native interactive HTML elements so pointer, keyboard, and assistive technology behavior work together.

## Learning goals

After this lesson, you should be able to:

- Add a template listener with parentheses.
- Pass and type the event object with $event.
- Distinguish event.target from event.currentTarget.
- Read user input from a DOM event safely.
- Filter keyboard events by key and modifier.
- Explain event bubbling, preventDefault(), and stopPropagation().
- Use native controls instead of recreating their keyboard behavior.

## 1. Event binding syntax

Write the event name in parentheses and provide a template statement:

~~~html
<button (click)="saveTask()">Save</button>
<input (input)="updateSearch($event)" />
~~~

The event name is the native DOM event, such as click, input, change, keydown, or keyup. Angular calls the handler when the element emits that event.

Event statements are evaluated in the component and template context. They can call component methods and, unlike ordinary template expressions, event statements support assignments. Keep complex work in a named TypeScript method so the template remains easy to read.

Name handlers after the action they perform. saveTask() explains the intent more clearly than handleClick().

## 2. Pass the event object with $event

Angular provides the event object as $event inside a listener:

~~~html
<button (click)="saveTask($event)">Save</button>
~~~

Use the browser event type that matches the event:

~~~ts
saveTask(event: MouseEvent): void {
  console.log(event.button);
}
~~~

Common event types include:

| Event | Common TypeScript type | Useful information |
| --- | --- | --- |
| click | MouseEvent | Pointer button, coordinates, and modifier keys |
| input | Event or InputEvent | An editable control changed its value |
| change | Event | A control committed a changed value |
| keydown / keyup | KeyboardEvent | key, code, repeat, and modifier state |
| submit | SubmitEvent | The submitted form and submitter, where supported |

Use the platform's event types and check which properties belong to each type. A keyboard event does not have mouse coordinates, for example.

A custom Angular component output may pass its own payload as $event; that payload is not necessarily a browser Event. Component outputs are covered in the reusable component lessons.

## 3. target and currentTarget

A DOM event has two useful element references:

- target is the element where the event originated.
- currentTarget is the element whose listener is currently handling it.

They can differ when an event bubbles from a child to a parent. For example, clicking an icon inside a button can make the icon the target and the button the currentTarget.

The event target is typed as EventTarget, which does not promise input-specific properties. Narrow it before reading a value:

~~~ts
updateSearch(event: Event): void {
  const target = event.currentTarget;

  if (target instanceof HTMLInputElement) {
    this.searchText = target.value;
  }
}
~~~

This checks the actual element type before reading value. Use currentTarget when you mean the element on which this listener was registered. Avoid assuming that event.target is always an input.

## 4. Listen to keyboard events

Use keydown or keyup for keyboard-specific behavior:

~~~html
<input (keydown)="inspectKey($event)" />
~~~

~~~ts
inspectKey(event: KeyboardEvent): void {
  console.log(event.key);
}
~~~

KeyboardEvent.key describes the key's meaning in the current keyboard layout and modifier state. For example, the same physical key can produce different key values on different layouts.

KeyboardEvent.code describes the physical key position, such as KeyA. Use code only when the physical key matters; most application shortcuts should use key.

Use keydown when a response should happen as the key is pressed. Use keyup when the response should happen after release. Holding a key can cause repeated keydown events; KeyboardEvent.repeat reports whether an event is part of that repetition.

## 5. Filter keys with Angular modifiers

Angular can filter a keyboard listener by a named key:

~~~html
<input (keydown.enter)="submitSearch()" />
~~~

Angular supports key modifiers for combinations:

~~~html
<textarea (keydown.control.enter)="submitNotes()"></textarea>
<input (keydown.shift.enter)="insertLineBreak()">
~~~

The supported modifier names include alt, control, meta, and shift. The key names are matched against KeyboardEvent.key. Use the names and casing shown in Angular's current event binding documentation.

For physical-key matching, add the code suffix:

~~~html
<input (keydown.code.alt.shift.KeyA)="handlePhysicalShortcut()">
~~~

This listens for the physical KeyA together with Alt and Shift, regardless of which character those keys produce in the active layout. Prefer ordinary key matching unless physical position is specifically required.

Keyboard shortcuts should not block normal typing or override browser and assistive technology commands without a clear reason. Consider platform differences: Control is common for shortcuts on Windows and Linux, while Meta is commonly used on macOS.

## 6. Use semantic controls for keyboard access

A native button already supports focus, Enter, and Space activation. Bind its click event rather than making a div imitate a button:

~~~html
<button type="button" (click)="toggleDetails()">
  Toggle details
</button>
~~~

A text input already supports text editing, selection, and keyboard navigation. A form with a submit button already allows Enter to submit in common form configurations.

Avoid adding custom keyboard handlers to controls whose native behavior already fits. If you build a custom widget, it must deliberately provide the correct focus behavior, keyboard interaction, role, and accessible name. That is substantially more work than using a native control.

## 7. Default actions and event propagation

Some browser events trigger a default action. Examples include following a link, submitting a form, or inserting typed text. If a handler must replace that action, explicitly call preventDefault():

~~~html
<a href="#details" (click)="openPanel($event)">Open details</a>
~~~

~~~ts
openPanel(event: MouseEvent): void {
  event.preventDefault();
  this.showPanel = true;
}
~~~

Do not prevent a default action unless the application is intentionally replacing it. For a form submit handled by the component, prevent navigation or reload as needed. Angular's current guide recommends calling preventDefault() explicitly instead of relying on a listener returning false.

stopPropagation() is different: it stops the event from continuing through the bubbling path. It does not cancel the browser's default action. Use it only when the event should not reach ancestor listeners.

## 8. Event bubbling and delegation

Many DOM events bubble from their target through ancestor elements. A listener on a parent may therefore receive events that began on a child. The event's target identifies the origin; currentTarget identifies the parent's listener element.

This can be useful for event delegation on a large collection, but the handler must determine which child initiated the event. For ordinary controls, put the listener on the element whose action it represents.

Not every event bubbles, and event behavior depends on the browser event type. Check the relevant platform documentation instead of assuming every event behaves the same way.

## 9. Global and host event listeners

For events on a component's own host element, prefer the component host metadata where appropriate:

~~~ts
@Component({
  selector: 'app-dialog',
  host: {
    '(keydown.escape)': 'closeDialog()',
  },
  template: '<ng-content />',
})
export class Dialog {
  closeDialog(): void {
    // Close the dialog through the owning application state.
  }
}
~~~

Angular's host event syntax also supports the global targets window, document, and body. Use global listeners only when the component truly needs them, and consider focus management and cleanup when the component starts listening imperatively. Avoid adding global listeners just to intercept ordinary key presses.

## 10. Input events are not two-way binding

An input event tells the application that the user edited a control. It does not automatically assign the input's value to a component field. The component must read the event and update its state, or use an appropriate forms or two-way binding API in a later lesson.

For a simple text field:

~~~html
<input [value]="query" (input)="updateQuery($event)">
~~~

This is a one-way property binding plus an event listener. The property supplies the current displayed value; the input handler copies user edits back to the component.

## Common mistakes

- Writing [click] instead of (click). Square brackets are for property binding.
- Writing an event handler with an untyped or incorrectly typed event object.
- Reading target.value without narrowing EventTarget.
- Using target when the listener element is meant; currentTarget is often the intended element.
- Using keyup or keydown to recreate a button's normal click activation.
- Using preventDefault() when the browser's normal action should happen.
- Expecting stopPropagation() to cancel the default action.
- Treating a custom component output payload as a MouseEvent or KeyboardEvent.
- Listening globally when a local listener on a semantic control is sufficient.
- Putting complicated behavior directly in a template statement.

## Practical checklist

Before adding an event binding, ask:

1. Which element owns this interaction?
2. Is there a native control that already provides the expected keyboard behavior?
3. Does the handler need $event, or can it call a method without it?
4. If it needs an event, what is the correct event type?
5. Is target or currentTarget the element I actually need?
6. Should the browser's default action continue?
7. Should the event bubble to an ancestor listener?
8. Does this keyboard shortcut respect normal typing, platform conventions, and accessibility?

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="018-property-attribute-class-and-style-bindings.md">018 - 18. Property, attribute, class, and style bindings</a></td><td><a href="../qnas/019-event-bindings-event-objects-and-keyboard-input.md">Q&amp;A</a></td><td><a href="../examples/019-event-bindings-event-objects-and-keyboard-input.md">Example</a></td><td><a href="020-two-way-binding-and-ngmodel.md">020 - 20. Two-way binding and ngModel</a></td>
</tr>
  </tbody>
</table>
