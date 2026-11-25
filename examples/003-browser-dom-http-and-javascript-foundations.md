# Browser, DOM, HTTP, and JavaScript Foundations: Examples

This standalone browser exercise brings together HTML, the DOM, events, asynchronous JavaScript, Fetch, JSON, and HTTP status handling. It is plain browser JavaScript so the underlying web platform is visible. Angular-specific versions of these ideas appear in later chapters.

Read [Chapter 3](../lessons/003-browser-dom-http-and-javascript-foundations.md) first, and review the [Q&A](../qnas/003-browser-dom-http-and-javascript-foundations.md) as you work.

## What you will build

A small task list that:

- Loads starter tasks from a local JSON file using Fetch.
- Shows a loading message, task list, and useful error message.
- Adds a task when a form is submitted.
- Marks tasks complete with an accessible checkbox.
- Renders task titles as text, without interpreting them as HTML.

The added tasks are kept in memory. Reloading the page restores the original tasks from the JSON file.

## Create the files

Make a directory named browser-foundations-demo with these four files:

~~~text
browser-foundations-demo/
├── index.html
├── styles.css
├── app.js
└── tasks.json
~~~

### index.html

~~~html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Browser Foundations Task List</title>
  <link rel="stylesheet" href="./styles.css">
  <script src="./app.js" defer></script>
</head>
<body>
  <main class="page">
    <h1>Task list</h1>
    <p>Tasks are loaded from a local JSON file in this example.</p>

    <form id="task-form">
      <label for="task-title">New task</label>
      <div class="form-row">
        <input
          id="task-title"
          name="title"
          type="text"
          maxlength="80"
          autocomplete="off"
          required
        >
        <button type="submit">Add task</button>
      </div>
    </form>

    <p id="status" role="status" aria-live="polite">Loading tasks…</p>
    <ul id="task-list" class="task-list" aria-label="Tasks"></ul>
  </main>
</body>
</html>
~~~

The defer attribute makes the browser wait to run app.js until the HTML has been parsed. The label names its input, and the live status region can announce updates to assistive technology.

### styles.css

~~~css
:root {
  color-scheme: light;
  font-family: system-ui, sans-serif;
  line-height: 1.5;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: #f5f7fa;
  color: #172033;
}

.page {
  width: min(100% - 2rem, 42rem);
  margin: 3rem auto;
  padding: 1.5rem;
  background: white;
  border: 1px solid #d9e0ea;
  border-radius: 0.75rem;
}

.form-row {
  display: flex;
  gap: 0.5rem;
  margin-top: 0.4rem;
}

input,
button {
  min-height: 2.75rem;
  font: inherit;
}

input {
  flex: 1;
  min-width: 0;
  padding: 0.5rem;
  border: 1px solid #7a8798;
  border-radius: 0.3rem;
}

button {
  padding: 0.5rem 0.9rem;
  border: 0;
  border-radius: 0.3rem;
  background: #1649a0;
  color: white;
  cursor: pointer;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #e39b17;
  outline-offset: 2px;
}

.task-list {
  display: grid;
  gap: 0.5rem;
  padding: 0;
  list-style: none;
}

.task-list li {
  padding: 0.75rem;
  border: 1px solid #d9e0ea;
  border-radius: 0.3rem;
}

.task-label {
  display: flex;
  align-items: flex-start;
  gap: 0.6rem;
}

.task-label input {
  flex: 0 0 auto;
  width: 1.15rem;
  min-height: 1.15rem;
  margin-top: 0.25rem;
}

.is-complete .task-title {
  color: #526071;
  text-decoration: line-through;
}
~~~

### tasks.json

~~~json
[
  {
    "id": 1,
    "title": "Read the browser and DOM lesson",
    "completed": false
  },
  {
    "id": 2,
    "title": "Inspect an HTTP request in developer tools",
    "completed": false
  },
  {
    "id": 3,
    "title": "Try adding and completing a task",
    "completed": true
  }
]
~~~

JSON strings use double quotes, property names are quoted, and the file cannot contain JavaScript comments or trailing commas.

### app.js

~~~js
const taskForm = document.querySelector('#task-form');
const taskTitleInput = document.querySelector('#task-title');
const taskList = document.querySelector('#task-list');
const statusMessage = document.querySelector('#status');

let tasks = [];
let nextTaskId = 1;

function setStatus(message) {
  statusMessage.textContent = message;
}

function renderTasks() {
  taskList.replaceChildren();

  if (tasks.length === 0) {
    const emptyItem = document.createElement('li');
    emptyItem.textContent = 'No tasks yet. Add one above.';
    taskList.append(emptyItem);
    return;
  }

  for (const task of tasks) {
    const item = document.createElement('li');
    const label = document.createElement('label');
    const checkbox = document.createElement('input');
    const title = document.createElement('span');

    label.className = 'task-label';
    checkbox.type = 'checkbox';
    checkbox.checked = task.completed;
    title.className = 'task-title';
    title.textContent = task.title;

    checkbox.addEventListener('change', () => {
      task.completed = checkbox.checked;
      renderTasks();
      setStatus('Task updated.');
    });

    label.append(checkbox, title);
    item.append(label);
    item.classList.toggle('is-complete', task.completed);
    taskList.append(item);
  }
}

async function loadTasks() {
  setStatus('Loading tasks…');

  try {
    const response = await fetch('./tasks.json');

    if (!response.ok) {
      throw new Error('The server returned status ' + response.status);
    }

    const data = await response.json();

    if (!Array.isArray(data)) {
      throw new Error('The JSON response must be an array.');
    }

    tasks = data.map((item) => ({
      id: Number(item.id),
      title: String(item.title),
      completed: Boolean(item.completed),
    }));

    nextTaskId = tasks.reduce(
      (largestId, task) => Math.max(largestId, task.id),
      0,
    ) + 1;

    renderTasks();
    setStatus('Loaded ' + tasks.length + ' tasks.');
  } catch (error) {
    console.error('Could not load tasks:', error);
    tasks = [];
    renderTasks();
    setStatus('Could not load tasks. Check the local server and tasks.json.');
  }
}

taskForm.addEventListener('submit', (event) => {
  event.preventDefault();

  const title = taskTitleInput.value.trim();

  if (title.length === 0) {
    setStatus('Enter a task before submitting.');
    taskTitleInput.focus();
    return;
  }

  tasks.push({
    id: nextTaskId,
    title,
    completed: false,
  });
  nextTaskId += 1;

  taskForm.reset();
  renderTasks();
  setStatus('Task added.');
  taskTitleInput.focus();
});

loadTasks();
~~~

## Serve the files over HTTP

Fetch needs a web origin. Do not open index.html directly as a file, because file URLs do not behave like a normal HTTP origin for local resource requests.

Open a terminal in the browser-foundations-demo directory and start Python's built-in local static server:

~~~sh
py -m http.server 8000
~~~

On systems where the command is python3 instead, use:

~~~sh
python3 -m http.server 8000
~~~

Open http://localhost:8000 in a browser. Keep the terminal process running while you use the page. The page and tasks.json are now served from the same origin.

## Follow the request in developer tools

1. Open the browser's Network panel.
2. Reload the page.
3. Find the request for tasks.json.
4. Inspect its method, status, response headers, and JSON body.
5. Select a task checkbox and add a task.
6. Notice that these local interactions do not send another HTTP request.

The browser requests index.html, then the script requests tasks.json. The response body is read and parsed as JSON. The JavaScript uses that data to create DOM elements.

## Trace one task from data to screen

For a starter task, the data travels through these steps:

1. The server returns an array from tasks.json.
2. response.json() parses the body into JavaScript values.
3. The code stores the values in the tasks array.
4. renderTasks creates li, label, input, and span DOM elements.
5. The title is assigned using textContent.
6. The browser displays the resulting DOM and CSS.

When the checkbox changes, the browser emits a change event. The listener updates the task object, calls renderTasks, and updates the status message.

## Why use textContent?

The task title comes from JSON or user input. Assigning it to textContent displays it as text. If someone enters text that looks like HTML, the browser shows those characters instead of treating them as markup.

Avoid inserting untrusted task text with innerHTML. Treating data as markup can create cross-site scripting vulnerabilities. In Angular, use template interpolation for text and follow Angular's security guidance for values that must be rendered as HTML.

## Explore HTTP and failure states

### Return a not-found status

Temporarily rename tasks.json and reload the page. The Network panel should show an unsuccessful response. The code checks response.ok and shows the error message instead of treating the response body as a successful task list.

The exact status shown depends on the local server. The key idea is to inspect the response rather than assume the request succeeded.

### Break the JSON

Put invalid JSON into tasks.json and reload. The HTTP response can still be successful, but response.json() will fail to parse its body. The catch block handles that parsing failure.

This shows two separate checks:

- HTTP status says whether the server reported a successful request.
- JSON parsing says whether the response body can be read as valid JSON.

### Stop the local server

Stop the Python process and reload. The browser cannot get a usable response, so the request fails at the network level. The catch block reports the failure.

## Connect this example to Angular

The page above uses direct DOM APIs because it is a plain JavaScript demonstration. An Angular version would normally:

- Keep the task collection in a component or service.
- Load data through an Angular HTTP service in the later HTTP lessons.
- Display tasks through template control flow and bindings.
- Handle form submission and checkbox changes through Angular event bindings.
- Use Angular's state and template system to update the rendered view.

The browser still performs the HTTP request and renders DOM elements; Angular provides a structured way to describe and update the application interface.

## Exercises

1. Add a task counter that displays the number of tasks.
2. Add a filter to show all, active, or completed tasks.
3. Add a Delete button with an accessible name for each task.
4. Add an empty-state message when the filter has no matches.
5. Add a loading failure retry button.
6. Use the Network panel to compare the successful JSON response with the not-found response.
7. Type a string containing an image element with a nonexistent source and an error handler as a task title. Confirm that it appears as text and does not execute.

## Troubleshooting

### The page loads but the task list reports a failure

Confirm the local server is running, open the page through its HTTP address, and make sure tasks.json is beside index.html.

### The Network panel shows a 404 for tasks.json

Check the file name and directory. Paths and file names may be case-sensitive on some hosts.

### The browser shows a CORS or file-origin error

Do not open the page using a file URL. Start the local HTTP server and open the localhost URL printed above.

### A task title appears as literal angle-bracket text

That is expected. The example uses textContent to render untrusted text safely rather than interpret it as HTML.

### New tasks disappear after reload

That is expected. The page changes only its in-memory array. Saving data across reloads requires a backend or a deliberate browser-storage design, which later chapters can cover.

## Key takeaway

HTML gives the page structure, the DOM represents that structure in the browser, events report user actions, JavaScript manages behavior, and HTTP moves data between the browser and a server. Angular builds on these browser foundations with components, templates, services, and its own state and HTTP APIs.
