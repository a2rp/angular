# Browser, DOM, HTTP, and JavaScript Foundations

Angular applications run in a browser and communicate with servers through web platform APIs. Understanding the browser, the DOM, events, asynchronous JavaScript, and HTTP makes Angular code easier to reason about and helps you debug issues that are not caused by Angular itself.

This chapter introduces the foundations used throughout the course. It does not replace the dedicated JavaScript course or the later lessons on Angular templates, events, and HTTP.

## Learning objectives

After this lesson, you should be able to:

- Describe the browser's role when loading a web page.
- Explain how HTML is represented as a DOM tree.
- Describe the path from a browser event to an updated Angular view.
- Explain the basic request and response parts of HTTP.
- Use browser Fetch conceptually and explain why an HTTP error status needs to be checked.
- Describe promises, async/await, JSON, same-origin rules, and CORS at a beginner level.
- Use browser developer tools to investigate a page and its network requests.

## The browser, application, and server

A typical web application has several cooperating parts:

- **The browser** displays the page, runs client-side JavaScript, handles user input, and applies browser security rules.
- **The frontend application** describes what the user sees and how the interface responds. Angular is one framework for building this part.
- **The server** receives requests, applies application rules, reads or changes data, and sends responses.
- **The database or another service** may store or supply information the server needs.

The browser and server have different jobs. Angular code usually runs in the browser, while a backend API runs on a server. An Angular app can request data from an API, but it does not make the browser a database or make private server credentials safe to put in frontend code.

JavaScript can run in more than one environment. Browser JavaScript has web APIs such as document, fetch, and browser events. Server-side JavaScript, for example in Node.js, has a different set of environment APIs. The language is JavaScript in both places, but an API available in one environment is not automatically available in the other.

## What happens when a page loads

At a high level, a browser:

1. Requests a document from a URL.
2. Receives an HTTP response, commonly containing HTML.
3. Parses the HTML into a document structure.
4. Fetches referenced resources such as stylesheets, images, fonts, and scripts.
5. Applies CSS and runs scripts according to their loading rules.
6. Renders pixels and responds to events such as clicks and keyboard input.
7. May make more HTTP requests as scripts or user actions require data.

A real browser performs many of these activities with optimizations and overlapping work. The list is a useful mental model, not a strict serial trace.

Angular participates in this browser process. It uses components and templates to create and update part of the page. It does not remove the browser, HTML, CSS, HTTP, or JavaScript from the picture.

## HTML and the DOM

HTML describes the structure and meaning of a document. The **Document Object Model**, or **DOM**, is a tree of objects representing the document and its elements.

For example:

~~~html
<main>
  <h1>Tasks</h1>
  <button type="button">Add task</button>
</main>
~~~

The browser creates corresponding element objects with parent and child relationships. JavaScript can inspect or change the DOM through browser APIs such as document and Element. The MDN DOM anatomy guide explains the tree model.

In plain JavaScript, a script might find a button and attach a listener:

~~~js
const button = document.querySelector('button');

button.addEventListener('click', () => {
  console.log('The button was selected');
});
~~~

Angular also results in DOM elements, but Angular templates express the view declaratively. The framework tracks component data and template bindings and updates the rendered view as that data changes. The Angular template guide describes templates as the way components define the DOM they render.

When Angular owns a component's view, prefer Angular template bindings and component state for normal UI updates. Directly changing the same nodes with document.querySelector or innerHTML can conflict with Angular's view model, bypass expected behavior, and create security or accessibility problems. Direct DOM access has specialized uses; later lessons cover Angular's supported options.

## Events and user interaction

An event represents something that happens, such as a click, key press, form submission, or page load. Browser code can register a listener to react. For example, addEventListener attaches a listener to a DOM target.

Events can travel through ancestors in a process called **bubbling**. A click on a button inside a form can therefore be observed by the form or other ancestors. Some elements also have browser-defined default actions: submitting a form or following a link, for example. Preventing a default action should be deliberate, since it changes the expected behavior for keyboard and assistive technology users.

In Angular templates, common event bindings use parentheses around the event name:

~~~html
<button type="button" (click)="addTask()">Add task</button>
<form (submit)="saveTask($event)">
  ...
</form>
~~~

An event binding connects an event to a component expression. The button still emits a browser click event; Angular provides a declarative way to handle it. The MDN events guide covers browser events, listeners, event objects, bubbling, and default actions.

Use semantic HTML controls where possible. A real button supports keyboard interaction and accessibility behavior that a clickable generic div does not provide automatically.

## JavaScript execution and asynchronous work

JavaScript statements normally execute in order. Some operations, such as waiting for a network response, take time. Browsers allow other work to continue while such operations are pending; the result is delivered later through an asynchronous API.

A **Promise** represents the eventual completion or failure of an operation. An async function returns a promise. The await keyword lets code inside that function pause until a promise settles, without blocking the browser's entire event loop.

~~~js
async function loadMessage() {
  try {
    const response = await fetch('/api/message');

    if (!response.ok) {
      throw new Error('The server returned an unsuccessful status');
    }

    const message = await response.json();
    console.log(message);
  } catch (error) {
    console.error('Could not load the message', error);
  }
}
~~~

The try/catch handles rejected operations and errors thrown in the try block. Checking response.ok handles HTTP error statuses such as a missing resource. The distinction is important and is explained in the HTTP section.

Angular applications use the same JavaScript runtime concepts. Depending on the Angular API, asynchronous results may be represented as promises, RxJS observables, or signals that hold the latest state. Later chapters explain these Angular patterns.

For more about promises and async/await, see Using promises and async functions.

## HTTP: requests and responses

**HTTP** is a client-server protocol for transferring resources. A browser commonly sends a request to a server and receives a response. Requests and responses contain metadata called headers and may contain a body.

A simplified request might contain:

- A URL identifying the server and resource.
- A method describing the requested operation.
- Request headers with additional information.
- An optional body, often used when submitting data.

A simplified response might contain:

- A status code describing the result.
- Response headers such as content type and caching information.
- An optional body containing HTML, JSON, an image, or another representation.

Common methods include:

| Method | Common purpose |
| --- | --- |
| GET | Retrieve a representation of a resource |
| POST | Submit data for processing or resource creation |
| PUT | Replace a resource representation |
| PATCH | Apply a partial change |
| DELETE | Request removal of a resource |

These are protocol semantics, not a guarantee that every API is designed well. Check the API's documentation before relying on what an endpoint does. The HTTP overview describes the request and response model, and the HTTP methods reference describes method semantics.

Status codes are grouped into classes:

- **2xx**: the request was handled successfully.
- **3xx**: more navigation or redirection is involved.
- **4xx**: the request has a client-side problem, such as missing authentication or a resource that does not exist.
- **5xx**: the server failed while handling the request.

A response with a 404 status is still an HTTP response. It is different from a network failure where the browser could not obtain a usable response at all. See the HTTP status reference.

HTTP is described as stateless: each request is interpreted on its own unless an application uses additional mechanisms such as cookies or tokens to associate requests with a session. HTTPS is HTTP carried over a TLS-protected connection; it protects data in transit but does not by itself guarantee that an application handles data safely.

## JSON and Fetch

**JSON** is a text format commonly used to represent structured data. JavaScript can parse JSON into values such as objects and arrays. JSON is data, not executable JavaScript.

The browser's Fetch API makes HTTP requests and returns a promise for a Response. Reading a JSON response is itself asynchronous:

~~~js
async function getTasks() {
  const response = await fetch('/api/tasks');

  if (!response.ok) {
    throw new Error('Request failed with status ' + response.status);
  }

  const tasks = await response.json();
  return tasks;
}
~~~

Calling response.json() reads and parses the response body. It does not itself mean the server returned a successful status. Always consider the status and handle failures.

Fetch commonly rejects for network-level failures, but an HTTP response with a 404 or 500 status does not automatically reject the fetch promise. Check response.ok or response.status. The Fetch API guide explains this distinction and response-body handling.

## Origins, same-origin policy, and CORS

A web **origin** is made from a scheme, host, and port. For example, differences in HTTPS versus HTTP, hostname, or port can make two URLs different origins.

Browsers restrict scripts from reading many responses from a different origin. This is part of the same-origin policy. **CORS**, or **Cross-Origin Resource Sharing**, is a server-controlled mechanism that can allow selected cross-origin responses to be read by browser JavaScript. CORS is not a frontend switch that can force another server to allow access; the server must send appropriate response headers.

For local development, a frontend may run on one port while an API runs on another. Those are different origins, so the API may need a development CORS configuration or the frontend dev server may need a proxy. Never solve a CORS issue by disabling browser security for normal browsing. Read MDN's CORS guide before changing server policy.

## How Angular connects these foundations

A simplified Angular data flow looks like this:

1. The user interacts with an HTML element.
2. The browser emits an event.
3. An Angular template binding calls component behavior.
4. The component or a service updates application state.
5. Angular reflects the state in its template and updates the corresponding DOM.
6. If data is needed, Angular code calls an HTTP API and handles the asynchronous result.
7. The resulting state is reflected in the template.

Angular provides APIs for these steps, but the browser's event system, DOM, networking, and JavaScript runtime are still underneath. When a page is blank or a request fails, knowing these layers helps narrow down the cause.

## Browser developer tools

The browser's developer tools are useful for investigating the whole stack:

- **Elements/Inspector**: inspect the rendered DOM and computed styles.
- **Console**: view JavaScript errors and run small diagnostic expressions.
- **Network**: inspect request URLs, methods, status codes, headers, timing, and response bodies.
- **Sources/Debugger**: set breakpoints and inspect JavaScript execution.
- **Accessibility tools**: inspect accessibility properties and run available audits.

For an API problem, check the Network panel before changing Angular code. Confirm the request was sent, the URL and method are correct, the response status and body are expected, and the browser did not report a CORS or mixed-content issue.

## Common misunderstandings

- **“Angular is the DOM.”** Angular renders and updates DOM nodes, but the DOM is a browser web API and exists independently of Angular.
- **“A 404 makes fetch reject automatically.”** Fetch normally fulfills with a Response; code should check the status.
- **“CORS is fixed in Angular.”** The server's CORS response headers control whether browser JavaScript may read a cross-origin response.
- **“JSON is JavaScript code.”** JSON is a data format. Parse it as data and do not execute untrusted strings.
- **“await freezes the browser.”** It pauses the dependent async function while the browser can continue other work.
- **“Anything available in the browser is safe to keep secret.”** Code and data sent to a browser can be inspected by its user. Secrets belong on a trusted server.
- **“Angular should update the page by querying elements manually.”** For ordinary UI, use Angular state and templates so the framework can manage the view consistently.

## Knowledge check

1. What does the DOM represent?
2. What is the difference between an HTTP status error and a network failure?
3. Why should code check response.ok after a Fetch request?
4. What does CORS control, and which side must allow the origin?
5. What happens when an event bubbles?
6. Does await block the browser's entire event loop?
7. Why should Angular templates normally manage component DOM updates?
8. Which developer-tools panel helps inspect a request's response status and headers?

Answers are in [Chapter 3 Q&A](../qnas/003-browser-dom-http-and-javascript-foundations.md). Follow the [Chapter 3 examples](../examples/003-browser-dom-http-and-javascript-foundations.md) to build a small page using DOM events, Fetch, JSON, and error states.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="002-angular-compared-with-angularjs.md">002 - Angular Compared with AngularJS</a></td><td><a href="../qnas/003-browser-dom-http-and-javascript-foundations.md">Q&amp;A</a></td><td><a href="../examples/003-browser-dom-http-and-javascript-foundations.md">Example</a></td><td><a href="004-typescript-essentials-values-types-and-inference.md">004 - TypeScript Essentials: Values, Types, and Inference</a></td>
</tr>
  </tbody>
</table>
