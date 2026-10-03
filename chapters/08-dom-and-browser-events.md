# 8. The DOM and browser events

[Back to notes index](../README.md)

| [Previous: Strings, numbers, dates, and JSON](07-strings-numbers-dates-and-json.md) | [Notes index](../README.md) | [Next: Forms, validation, and browser storage](09-forms-validation-and-browser-storage.md) |
|:--|:--:|--:|

## The DOM represents the page

The Document Object Model, or DOM, is the browser's object representation of an HTML document. JavaScript can find DOM elements, read and update their properties, create new elements, and respond to browser events.

The DOM is a browser API. Node.js does not provide document by default. Keep browser-specific code in browser modules or provide a DOM implementation when testing it outside a browser.

## Find elements safely

Use querySelector to find the first matching element and querySelectorAll to find all matching elements. The first method returns null if there is no match, so check the result before using it.

~~~html
<h1 id="page-title">Study notes</h1>
<p class="status">Ready</p>
~~~

~~~js
const title = document.querySelector('#page-title');
const statusMessages = document.querySelectorAll('.status');

if (title instanceof HTMLHeadingElement) {
  console.log(title.textContent);
}

console.log(statusMessages.length);
~~~

querySelectorAll returns a static NodeList. It is not an Array, though it can be iterated with for...of. Use Array.from when an array method such as map is needed.

~~~js
const statusText = Array.from(
  document.querySelectorAll('.status'),
  (element) => element.textContent,
);

console.log(statusText);
~~~

Prefer specific selectors and stable IDs or data attributes. A selector that depends on a long chain of layout classes can break when the page structure changes.

## Change text and classes

Use textContent to set plain text. It treats the value as text instead of parsing it as HTML.

~~~js
const message = document.querySelector('#message');

if (message) {
  message.textContent = 'Your notes are saved.';
  message.classList.add('success');
  message.setAttribute('aria-live', 'polite');
}
~~~

classList provides add, remove, toggle, and contains methods. Use properties such as hidden, disabled, and checked when they match the element's behavior.

~~~js
const panel = document.querySelector('#help-panel');

if (panel instanceof HTMLElement) {
  panel.hidden = true;
  panel.classList.toggle('is-collapsed', panel.hidden);
}
~~~

Do not use innerHTML with text from a user, URL, or network response. It parses markup and can create a cross-site scripting vulnerability. If plain text is intended, use textContent.

## Create and remove elements

Use createElement to make an element, set its content and attributes, then add it to the document.

~~~html
<ul id="note-list"></ul>
~~~

~~~js
const list = document.querySelector('#note-list');

if (list instanceof HTMLUListElement) {
  const item = document.createElement('li');
  item.textContent = 'DOM basics';
  item.classList.add('note-item');
  list.append(item);
}
~~~

append adds nodes or text at the end of an element. Other useful methods include prepend, before, after, replaceWith, and remove.

~~~js
const oldMessage = document.querySelector('.temporary-message');
oldMessage?.remove();
~~~

Use optional chaining only when the element may be absent by design. If the element is required for the page, report the missing setup clearly instead of silently doing nothing.

## Listen for events

addEventListener registers a function that runs when an event occurs. Keep a reference to the function if it will need to be removed later.

~~~html
<button id="save-button" type="button">Save</button>
<p id="save-status" aria-live="polite"></p>
~~~

~~~js
const saveButton = document.querySelector('#save-button');
const saveStatus = document.querySelector('#save-status');

function showSavedMessage() {
  if (saveStatus) {
    saveStatus.textContent = 'Saved.';
  }
}

saveButton?.addEventListener('click', showSavedMessage);
~~~

An event listener receives an event object. target is the element where the event began, while currentTarget is the element whose listener is running.

~~~js
document.querySelector('#save-button')?.addEventListener('click', (event) => {
  console.log('Clicked element:', event.target);
  console.log('Listener element:', event.currentTarget);
});
~~~

For interactive controls, use the correct native element. A button already supports keyboard activation, focus, and accessibility behavior. Do not replace a button with a clickable div unless there is a compelling reason and all equivalent behavior is implemented.

## Understand bubbling and event delegation

Many DOM events bubble from the target through its ancestors. Event delegation uses one listener on a shared parent to handle events from its children. This is useful when list items are added after the listener is installed.

~~~html
<ul id="note-list">
  <li>
    <span>Functions</span>
    <button type="button" data-action="remove">Remove</button>
  </li>
</ul>
~~~

~~~js
const list = document.querySelector('#note-list');

list?.addEventListener('click', (event) => {
  if (!(event.target instanceof Element)) {
    return;
  }

  const removeButton = event.target.closest('[data-action="remove"]');

  if (!removeButton || !list.contains(removeButton)) {
    return;
  }

  removeButton.closest('li')?.remove();
});
~~~

The listener checks that the event came from a matching remove button inside the list. This prevents an unrelated click inside the parent from triggering the action.

Some events can be handled during the capture phase by passing { capture: true }. Most common click handling uses the default bubbling phase. Use event.stopPropagation only when preventing an ancestor from receiving the event is part of the intended behavior.

## Prevent a browser default action

Some events have a default browser behavior. A form submit normally navigates or reloads the page. preventDefault cancels that default so JavaScript can handle the submission.

~~~js
const form = document.querySelector('#search-form');

form?.addEventListener('submit', (event) => {
  event.preventDefault();
  console.log('Handle the search in JavaScript.');
});
~~~

Do not cancel default behavior without replacing the action in a usable way. Form input, validation, and submission are covered in Chapter 9.

## Remove listeners and clean up

removeEventListener requires the same function reference and matching capture option that were used to register the listener.

~~~js
const button = document.querySelector('#toggle-button');

function togglePanel() {
  console.log('Toggle the panel');
}

button?.addEventListener('click', togglePanel);
button?.removeEventListener('click', togglePanel);
~~~

An AbortController can remove several listeners together. This is convenient when a view or feature needs to stop listening.

~~~js
const controller = new AbortController();

window.addEventListener(
  'resize',
  () => console.log(window.innerWidth),
  { signal: controller.signal },
);

controller.abort();
~~~

Use event listeners on the elements that own the interaction. A global listener for every button can make behavior harder to trace.

## Work with data attributes

Data attributes attach application-specific values to HTML elements. JavaScript reads them through dataset.

~~~html
<button type="button" data-note-id="42" data-action="open-note">
  Open note
</button>
~~~

~~~js
const openButton = document.querySelector('[data-action="open-note"]');

if (openButton instanceof HTMLButtonElement) {
  console.log(openButton.dataset.noteId); // String value: "42"
}
~~~

Dataset values are strings. Convert and validate them before using them as numbers or identifiers.

## Hands-on: add and remove list items

Create a text input, an add button, and a list. When the button is clicked, add a list item with the input text. Use event delegation to remove an item when its remove button is clicked.

~~~html
<label for="topic-input">New topic</label>
<input id="topic-input" />
<button id="add-topic" type="button">Add topic</button>
<ul id="topic-list"></ul>
~~~

~~~js
const input = document.querySelector('#topic-input');
const addButton = document.querySelector('#add-topic');
const topicList = document.querySelector('#topic-list');

addButton?.addEventListener('click', () => {
  if (!(input instanceof HTMLInputElement)) {
    return;
  }

  const title = input.value.trim();

  if (title.length === 0 || !(topicList instanceof HTMLUListElement)) {
    return;
  }

  const item = document.createElement('li');
  const label = document.createElement('span');
  const removeButton = document.createElement('button');

  label.textContent = title;
  removeButton.type = 'button';
  removeButton.textContent = 'Remove';
  removeButton.dataset.action = 'remove';

  item.append(label, removeButton);
  topicList.append(item);
  input.value = '';
});

topicList?.addEventListener('click', (event) => {
  if (!(event.target instanceof Element)) {
    return;
  }

  if (event.target.closest('[data-action="remove"]')) {
    event.target.closest('li')?.remove();
  }
});
~~~

The item title is assigned with textContent, so user input is treated as text. Try entering angle brackets and confirm that they appear as characters rather than becoming markup.

## Notes to remember

- The DOM is a browser API that represents the page.
- querySelector can return null, and querySelectorAll returns a NodeList.
- Use textContent for plain text and createElement for new nodes.
- Use native buttons and links for interactive controls.
- Event delegation handles events from dynamic children.
- target is where the event began; currentTarget is where the listener runs.
- Untrusted strings inserted as HTML can create a security vulnerability.

## References

- [MDN DOM introduction](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction)
- [MDN selecting elements](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector)
- [MDN EventTarget.addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
- [MDN event bubbling](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Event_bubbling)
- [MDN Node.textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent)
- [MDN HTMLElement.dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset)
- [MDN DOM security](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML#security_considerations)

---

| [Previous: Strings, numbers, dates, and JSON](07-strings-numbers-dates-and-json.md) | [Notes index](../README.md) | [Next: Forms, validation, and browser storage](09-forms-validation-and-browser-storage.md) |
|:--|:--:|--:|
