# 9. Forms, validation, and browser storage

[Back to notes index](../README.md)

| [Previous: The DOM and browser events](08-dom-and-browser-events.md) | [Notes index](../README.md) | [Next: Errors, debugging, and testing](10-errors-debugging-and-testing.md) |
|:--|:--:|--:|

## Read form values with FormData

Forms provide a standard way for users to submit related values. Give every field a label and a name. FormData reads named controls when the form is submitted.

~~~html
<form id="note-form">
  <label for="note-title">Note title</label>
  <input id="note-title" name="title" required minlength="3" />
  <button type="submit">Save note</button>
</form>
<p id="form-status" aria-live="polite"></p>
~~~

~~~js
const form = document.querySelector('#note-form');
const status = document.querySelector('#form-status');

form?.addEventListener('submit', (event) => {
  event.preventDefault();

  if (!(form instanceof HTMLFormElement)) {
    return;
  }

  const formData = new FormData(form);
  const title = formData.get('title');

  if (typeof title !== 'string' || title.trim().length < 3) {
    if (status) {
      status.textContent = 'Enter a title with at least three characters.';
    }
    return;
  }

  if (status) {
    status.textContent = 'Saved: ' + title.trim();
  }

  form.reset();
});
~~~

The submit event runs for both a button click and keyboard submission. preventDefault stops the browser's normal form navigation so this page can handle the value. If a server should receive the form, send the data to that server instead of only showing a local message.

## Use built-in browser validation

Attributes such as required, minlength, maxlength, min, max, and type provide built-in constraints. The browser can block an invalid form submission and show its own feedback.

~~~html
<label for="email">Email address</label>
<input id="email" name="email" type="email" required />
<label for="word-count">Word count</label>
<input id="word-count" name="wordCount" type="number" min="1" max="5000" required />
~~~

JavaScript can check validity before continuing. reportValidity asks the browser to show its validation feedback.

~~~js
const noteForm = document.querySelector('#note-form');

if (noteForm instanceof HTMLFormElement && !noteForm.reportValidity()) {
  console.log('The form has invalid fields.');
}
~~~

Use custom validity only when native constraints cannot express a real requirement. Clear the custom message when the value becomes valid.

~~~js
const password = document.querySelector('#password');

password?.addEventListener('input', () => {
  if (password instanceof HTMLInputElement) {
    password.setCustomValidity(
      password.value.length < 12 ? 'Use at least 12 characters.' : '',
    );
  }
});
~~~

Client-side validation improves feedback but is not a security check. Validate all submitted values on the server too.

## Show accessible validation feedback

- Associate a visible label with each field using matching for and id values.
- Use field-level help text to describe constraints before an error occurs.
- Connect help and error text with aria-describedby.
- Keep the message clear and actionable.
- Do not use color as the only indication of an error.
- Move focus to an invalid field only when it helps the user recover.

~~~html
<label for="topic">Topic</label>
<input
  id="topic"
  name="topic"
  required
  aria-describedby="topic-help topic-error"
/>
<p id="topic-help">Enter the subject of this note.</p>
<p id="topic-error" role="alert" hidden>Enter a topic before saving.</p>
~~~

A complete form should test both keyboard use and screen reader feedback. Native form controls provide keyboard behavior that a custom div would need to recreate.

## Store small values in the browser

localStorage persists string values for an origin across browser sessions. sessionStorage keeps values for a tab's page session. Both APIs store strings, so objects are commonly converted with JSON.stringify and JSON.parse.

~~~js
localStorage.setItem('theme', 'dark');

const savedTheme = localStorage.getItem('theme') ?? 'light';
console.log(savedTheme);

sessionStorage.setItem('current-tab', 'notes');
~~~

Storage is scoped by origin, which includes protocol, hostname, and port. It can be unavailable or full, so storage access may throw. Do not treat it as a database for large or sensitive information.

## Parse stored JSON defensively

Saved values can be missing, malformed, or from an older version of the application. Parsing should be inside a try block, and the result should be checked before it is used.

~~~js
function loadNotes() {
  try {
    const text = localStorage.getItem('study-notes');

    if (text === null) {
      return [];
    }

    const value = JSON.parse(text);

    if (!Array.isArray(value)) {
      return [];
    }

    return value.filter(
      (note) =>
        typeof note === 'object' &&
        note !== null &&
        typeof note.id === 'string' &&
        typeof note.title === 'string',
    );
  } catch (error) {
    console.error('Could not read saved notes.', error);
    return [];
  }
}
~~~

JSON.parse checks JSON syntax, not the shape of each object. The filter checks the fields this example needs. A larger application should validate the complete data contract and decide how to recover invalid storage.

~~~js
function saveNotes(notes) {
  try {
    localStorage.setItem('study-notes', JSON.stringify(notes));
    return true;
  } catch (error) {
    console.error('Could not save notes.', error);
    return false;
  }
}
~~~

Handle the false return value so the user knows that saving did not work. A quota error can happen when storage has no room.

## Do not store secrets in browser storage

JavaScript running on the page can read localStorage and sessionStorage. A cross-site scripting vulnerability can expose their contents. Do not store passwords, private keys, or long-lived sensitive credentials there.

For sensitive authentication, use a security design appropriate to the server and application, such as secure, HttpOnly cookies with correct SameSite and Secure settings. Browser storage is suitable for non-sensitive preferences and small local drafts, with an understanding that users can inspect or clear it.

## Hands-on: save a small note list

Create a form with a title input, an add button, a status message, and an empty unordered list. Store note objects in localStorage and re-render the list after a successful submit.

~~~html
<form id="study-note-form">
  <label for="study-note-title">Note title</label>
  <input id="study-note-title" name="title" required minlength="3" />
  <button type="submit">Add note</button>
</form>
<p id="notes-status" aria-live="polite"></p>
<ul id="study-note-list"></ul>
~~~

~~~js
const storageKey = 'study-notes';
const notesForm = document.querySelector('#study-note-form');
const notesList = document.querySelector('#study-note-list');
const notesStatus = document.querySelector('#notes-status');
let notes = loadNotes();

function renderNotes() {
  if (!(notesList instanceof HTMLUListElement)) {
    return;
  }

  notesList.replaceChildren();

  for (const note of notes) {
    const item = document.createElement('li');
    item.textContent = note.title;
    notesList.append(item);
  }
}

notesForm?.addEventListener('submit', (event) => {
  event.preventDefault();

  if (!(notesForm instanceof HTMLFormElement)) {
    return;
  }

  const title = new FormData(notesForm).get('title');

  if (typeof title !== 'string' || title.trim().length < 3) {
    return;
  }

  notes = [
    ...notes,
    { id: crypto.randomUUID(), title: title.trim() },
  ];

  if (saveNotes(notes)) {
    renderNotes();
    notesForm.reset();

    if (notesStatus) {
      notesStatus.textContent = 'Note saved.';
    }
  }
});

renderNotes();
~~~

This example uses loadNotes and saveNotes from the earlier snippets. Run it from a secure local development server so crypto.randomUUID and browser storage are available. Reload the page to check that saved notes return. Use the browser's Application or Storage panel to inspect and remove the saved value.

## Notes to remember

- FormData reads controls by their name.
- Listen for the form submit event instead of handling only a button click.
- Native attributes provide useful browser validation.
- Validate again on the server when data is submitted remotely.
- localStorage persists strings across sessions; sessionStorage is limited to a tab session.
- Handle storage and JSON errors and check parsed values.
- Never treat browser storage as a safe place for secrets.

## References

- [MDN form validation](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)
- [MDN FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)
- [MDN constraint validation](https://developer.mozilla.org/en-US/docs/Web/HTML/Constraint_validation)
- [MDN localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)
- [MDN sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)
- [MDN HTML forms accessibility](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Your_first_form)

---

| [Previous: The DOM and browser events](08-dom-and-browser-events.md) | [Notes index](../README.md) | [Next: Errors, debugging, and testing](10-errors-debugging-and-testing.md) |
|:--|:--:|--:|
