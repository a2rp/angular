# 12. Fetch and REST APIs

[Back to notes index](../README.md)

| [Previous: Asynchronous JavaScript and promises](11-asynchronous-javascript-and-promises.md) | [Notes index](../README.md) | [Next: Modules, npm, and project structure](13-modules-npm-and-project-structure.md) |
|:--|:--:|--:|

## Send a request with Fetch

The Fetch API sends network requests and returns a Promise for a Response. It is available in modern browsers and current Node.js releases.

~~~js
async function loadNotes() {
  const response = await fetch('/api/notes');

  if (!response.ok) {
    throw new Error('Request failed with status ' + response.status);
  }

  const notes = await response.json();
  return notes;
}
~~~

A fulfilled fetch Promise does not mean the server returned a successful status. Check response.ok or response.status. Fetch rejects for some network failures, but an HTTP error such as 404 still produces a Response.

## Parse and validate a JSON response

response.json returns a Promise that parses the response body as JSON. Parsing can fail, and valid JSON can still have an unexpected shape.

~~~js
function isNote(value) {
  return (
    typeof value === 'object' &&
    value !== null &&
    typeof value.id === 'string' &&
    typeof value.title === 'string' &&
    typeof value.reviewed === 'boolean'
  );
}

async function getNotes() {
  const response = await fetch('/api/notes');

  if (!response.ok) {
    throw new Error('Could not load notes: ' + response.status);
  }

  const data = await response.json();

  if (!Array.isArray(data) || !data.every(isNote)) {
    throw new Error('The server returned invalid note data.');
  }

  return data;
}
~~~

The validation checks only fields this example uses. Match the check to the API contract and reject malformed data before it reaches code that assumes those fields exist.

## Send a JSON request

To send JSON, serialize the value with JSON.stringify and set the content type. The server still decides which fields are allowed and validates every value.

~~~js
async function createNote(note) {
  const response = await fetch('/api/notes', {
    method: 'POST',
    headers: {
      Accept: 'application/json',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(note),
  });

  if (!response.ok) {
    throw new Error('Could not create note: ' + response.status);
  }

  return response.json();
}
~~~

A server can return a different status and response body for success and failure. Read and parse the response according to the endpoint contract.

## Use query strings safely

URLSearchParams encodes query parameter names and values.

~~~js
async function searchNotes(query) {
  const params = new URLSearchParams({ q: query });
  const response = await fetch('/api/notes?' + params);

  if (!response.ok) {
    throw new Error('Search failed: ' + response.status);
  }

  return response.json();
}
~~~

Do not concatenate raw user input into a URL. Use URLSearchParams for query values and URL when building or validating complete addresses.

~~~js
const endpoint = new URL('/api/notes', window.location.origin);
endpoint.searchParams.set('status', 'to-review');

console.log(endpoint.href);
~~~

The window example runs in a browser. In Node.js, use an explicit base URL or configuration value instead of window.location.

## Handle network and HTTP errors

A request can fail because of a network problem, an unsuccessful HTTP status, invalid JSON, or unexpected data. Catch errors at the boundary where the application can show a useful message or retry.

~~~js
async function showNotes() {
  try {
    const notes = await getNotes();
    renderNotes(notes);
  } catch (error) {
    console.error('Could not display notes.', error);
    showMessage('Notes could not be loaded. Try again.');
  }
}
~~~

A retry can be appropriate for a temporary network failure or an idempotent read. Do not automatically retry every operation. Repeating a create request can create duplicate records unless the server supports a safe idempotency strategy.

Do not show raw stack traces or internal server messages to users. Keep detailed diagnostics for development logs and display a concise recovery message.

## Cancel a request

AbortController can cancel a fetch request. This helps when a user starts a new search or leaves a view before the previous response arrives.

~~~js
const controller = new AbortController();

fetch('/api/notes', { signal: controller.signal })
  .then((response) => {
    if (!response.ok) {
      throw new Error('Request failed: ' + response.status);
    }

    return response.json();
  })
  .then((notes) => console.log(notes))
  .catch((error) => {
    if (error.name === 'AbortError') {
      console.log('Request cancelled.');
      return;
    }

    console.error('Request failed.', error);
  });

controller.abort();
~~~

For a real interface, call abort when the request is no longer needed, not immediately after starting it. A new AbortController is needed for a new request.

## Add a timeout

A timeout can cancel a request that takes too long. Clear the timer when the operation finishes so it does not remain scheduled.

~~~js
async function fetchWithTimeout(url, timeoutMilliseconds) {
  const controller = new AbortController();
  const timerId = setTimeout(
    () => controller.abort(),
    timeoutMilliseconds,
  );

  try {
    return await fetch(url, { signal: controller.signal });
  } finally {
    clearTimeout(timerId);
  }
}
~~~

The function returns the Response. The caller must still check response.ok and handle errors. Distinguish a timeout from user cancellation if the interface needs to explain why the operation stopped.

## Understand same-origin and CORS

A browser restricts requests between different origins unless the server allows them through Cross-Origin Resource Sharing (CORS). An origin consists of the scheme, host, and port. A development proxy can simplify local requests, but the production server still needs the correct policy.

CORS is enforced by browsers. It is not authentication and does not protect an API from direct requests. The server must authenticate and authorize sensitive operations.

Use HTTPS for production API traffic. Keep server credentials on the server. Values in JavaScript files sent to a browser can be read by users.

## Send updates and delete records

A PUT request commonly replaces a resource. PATCH commonly changes selected fields. DELETE asks the server to remove a resource. Use the server's documented contract for the exact payload and status behavior.

~~~js
async function updateNote(id, changes) {
  const response = await fetch('/api/notes/' + encodeURIComponent(id), {
    method: 'PATCH',
    headers: {
      Accept: 'application/json',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(changes),
  });

  if (!response.ok) {
    throw new Error('Could not update note: ' + response.status);
  }

  return response.json();
}

async function deleteNote(id) {
  const response = await fetch('/api/notes/' + encodeURIComponent(id), {
    method: 'DELETE',
  });

  if (!response.ok) {
    throw new Error('Could not delete note: ' + response.status);
  }
}
~~~

A DELETE endpoint may return 204 No Content. Do not call response.json when there is no response body.

## Hands-on: load and render a list

Use the getNotes function to load a note list, handle errors, and create DOM elements using textContent.

~~~js
async function displayNotes() {
  const list = document.querySelector('#note-list');
  const status = document.querySelector('#load-status');

  if (!(list instanceof HTMLUListElement)) {
    return;
  }

  if (status) {
    status.textContent = 'Loading notes...';
  }

  try {
    const notes = await getNotes();
    list.replaceChildren();

    for (const note of notes) {
      const item = document.createElement('li');
      item.textContent = note.title;
      list.append(item);
    }

    if (status) {
      status.textContent = notes.length + ' notes loaded.';
    }
  } catch (error) {
    if (status) {
      status.textContent = 'Notes could not be loaded. Try again.';
    }

    console.error(error);
  }
}

displayNotes();
~~~

Add a retry button that calls displayNotes again. Inspect the Network panel and compare the request URL, status, headers, and response with the API's documentation.

## Notes to remember

- Fetch returns a Promise for a Response.
- Check response.ok because HTTP error statuses do not reject fetch by themselves.
- response.json is asynchronous and can fail to parse.
- Validate important fields before relying on a network response.
- Use URLSearchParams for query values and encode path segments.
- AbortController cancels requests that are no longer needed.
- CORS is a browser policy, not authentication.
- Do not expose secrets or internal error details in browser code.

## References

- [MDN Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [MDN Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)
- [MDN AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)
- [MDN URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)
- [MDN CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

---

| [Previous: Asynchronous JavaScript and promises](11-asynchronous-javascript-and-promises.md) | [Notes index](../README.md) | [Next: Modules, npm, and project structure](13-modules-npm-and-project-structure.md) |
|:--|:--:|--:|
