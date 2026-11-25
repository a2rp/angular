# Browser, DOM, HTTP, and JavaScript Foundations: Q&A

This Q&A accompanies [Chapter 3: Browser, DOM, HTTP, and JavaScript Foundations](../lessons/003-browser-dom-http-and-javascript-foundations.md).

## Browser and application layers

### 1. What is the browser responsible for in a web application?

The browser requests and displays documents and resources, runs client-side JavaScript, handles user interaction, applies browser security rules, and exposes web APIs such as the DOM, events, and Fetch.

### 2. What is the difference between a frontend and a backend?

A frontend is the part a user interacts with, commonly running in the browser. A backend runs on a server and can enforce application rules, protect credentials, and access databases or other services. A frontend can call a backend API, but it should not be trusted to protect secrets.

### 3. Does Angular replace the browser or the DOM?

No. Angular is a framework that helps create and update a web interface. The browser still parses documents, creates and renders DOM elements, dispatches events, and performs network requests.

### 4. What happens after a browser receives an HTML page?

The browser parses the HTML into a document structure, fetches referenced resources, applies CSS, runs scripts according to their loading behavior, renders the page, and reacts to events. The browser can overlap and optimize these activities, so this is a mental model rather than an exact sequence.

### 5. Is browser JavaScript the same environment as Node.js JavaScript?

The core language is JavaScript in both environments, but their host APIs differ. A browser exposes web APIs such as document and window; Node.js provides server-side APIs. Code must use APIs available in its runtime.

## DOM, events, and Angular views

### 6. What does DOM stand for, and what does it represent?

DOM stands for Document Object Model. It represents an HTML or XML document as a tree of objects and relationships that scripts and browsers can inspect and manipulate.

### 7. Is the DOM the same thing as the original HTML file?

No. HTML is source text that the browser parses. The DOM is the in-memory document structure produced from that source and modified during page execution.

### 8. What is an event?

An event is a signal that something happened, such as a click, key press, form submission, or page load. Code can register a listener to respond to it.

### 9. What is event bubbling?

For many events, the browser dispatches the event at the target and then lets it travel through ancestors. An ancestor listener can therefore observe an event that began on a child element. Some events have different propagation behavior.

### 10. What is a default action?

It is a browser-defined behavior associated with an interaction. Examples include submitting a form or following a link. Preventing a default action changes that expected behavior and should be done intentionally.

### 11. How does an Angular event binding relate to a browser event?

The browser still emits the event. Angular's template binding syntax connects that event to a component expression, such as a method call, and integrates that interaction with Angular's view updates.

### 12. Why should an Angular component normally use template bindings instead of manual DOM updates?

Angular's template and component state define the view. Manually changing the same nodes can put the DOM out of sync with Angular's model and can bypass framework behavior. Use direct DOM access only for specialized needs and through supported Angular patterns.

### 13. Should a clickable div be used instead of a button?

Usually no. A button already supports keyboard interaction, focus behavior, and semantics expected by assistive technologies. A generic div does not gain those behaviors merely because a click listener is attached.

## JavaScript and asynchronous operations

### 14. What does a Promise represent?

A Promise represents the eventual result of an asynchronous operation. It can fulfill with a value or reject with a reason. Code can attach handlers with then and catch or use async/await.

### 15. Does await stop the whole browser?

No. Await suspends the dependent async function until its promise settles. It does not freeze the browser's entire event loop; the browser can continue other work.

### 16. Does an async function return its final value directly?

An async function always returns a Promise. A returned value fulfills that Promise, and a thrown error rejects it.

### 17. Why use try/catch with await?

A try/catch can handle a rejected awaited operation and exceptions thrown in the try block. It provides a clear place to choose a user-visible error state, retry, or report the failure.

## HTTP and Fetch

### 18. What is HTTP?

HTTP is a client-server protocol used to transfer web resources and exchange requests and responses. A browser sends a request, and a server or intermediary returns a response.

### 19. What are the common parts of an HTTP request?

A request has a target URL and method, may include headers, and may have a body. The method describes the intended operation, while headers provide additional metadata.

### 20. What are the common parts of an HTTP response?

A response includes a status code and headers, and may include a body. A body might contain HTML, JSON, or other data.

### 21. What are GET, POST, PUT, PATCH, and DELETE commonly used for?

GET commonly retrieves a representation, POST submits data for processing or creation, PUT replaces a representation, PATCH applies a partial modification, and DELETE requests removal. Actual endpoint behavior should be confirmed in the API documentation.

### 22. What do the HTTP status-code classes mean?

2xx means success, 3xx indicates redirection, 4xx indicates a request or client-side problem, and 5xx indicates a server-side failure. The exact code provides more specific meaning.

### 23. Is a 404 response a network failure?

No. A 404 is an HTTP response from a server indicating the requested resource was not found. A network failure means the browser could not obtain a usable HTTP response, for example because the connection failed or the browser blocked access.

### 24. Does Fetch reject when a server returns 404 or 500?

Usually no. Fetch fulfills with a Response for an HTTP error status. Code should check response.ok or response.status and handle unsuccessful statuses explicitly.

### 25. What does response.json() do?

It asynchronously reads the response body and parses it as JSON. It does not prove the HTTP status was successful, so check the status before or while processing the body.

### 26. Can the same response body be read more than once?

Normally the response body is a stream that is consumed when read. If code needs to read it in more than one way, it should deliberately clone the Response before consuming it or store the parsed result.

### 27. What does it mean that HTTP is stateless?

The protocol does not automatically remember the meaning of earlier requests when handling a new request. Applications can associate requests with a user or session using mechanisms such as cookies or tokens.

### 28. What does HTTPS provide?

HTTPS uses TLS to protect HTTP communication in transit and authenticate the server endpoint. It does not by itself make an application free from bugs or make client-side secrets private.

## Origins, security, and debugging

### 29. What is an origin?

An origin is based on a URL's scheme, host, and port. If one differs, the URL may have a different origin from the current page.

### 30. What is the same-origin policy?

It is a browser security rule that restricts scripts from one origin from reading certain resources belonging to another origin, unless a mechanism such as CORS permits it.

### 31. What does CORS do?

CORS is a server-controlled set of HTTP headers that tells browsers which cross-origin requests may be shared with scripts. The server serving the resource must allow the requesting origin; Angular cannot force a remote server to permit access.

### 32. Why can a local frontend and API on different ports cause a CORS issue?

The port is part of the origin. For example, localhost on port 4200 and localhost on port 3000 are different origins. The API may need to allow the frontend origin, or the development server may proxy the request.

### 33. Which developer-tools panel helps diagnose a failed API call?

The Network panel. It can show whether the request was sent, its URL and method, status, headers, timing, and response body. The Console can show JavaScript exceptions and browser security messages such as CORS errors.

### 34. Why should secrets stay on the server?

Code and data delivered to a browser can be inspected by the user. A key embedded in frontend source is not secret. Keep private credentials in a trusted server environment and let the server perform protected operations.

### 35. Is JSON safe because it is not JavaScript code?

JSON is a data format, but data still needs validation and safe handling. Do not execute untrusted strings as code or insert untrusted text as HTML.

## Review scenarios

### 36. A Fetch call gets a 500 response, but the catch block does not run. Is Fetch broken?

No. Fetch normally fulfills with a Response for an HTTP 500. Check response.ok or response.status and handle the unsuccessful result explicitly.

### 37. A request appears in the Network panel but Angular cannot read its response. What should you investigate?

Check the response status and browser console first. If the API is cross-origin, inspect its CORS response headers and confirm that the API allows the frontend's exact origin and request requirements.

### 38. A click handler runs, but the screen displays old data. What layers could be involved?

Inspect the event and handler, the component or service state change, the template binding, and any errors in the Console. In Angular, use framework-managed state and bindings instead of mutating the same rendered node manually.

### 39. A developer says “the browser downloaded my API key, but it is hidden in the Angular bundle.” Is it protected?

No. Bundling or minifying does not make client code private. Move privileged API calls to a server and keep the credential there.

### 40. What is a good first check when an Angular page fails to display API data?

Open the Network panel and verify that the request URL and method are expected, the response status is successful, and the response body has the expected shape. Then inspect the Console and the component's state and template bindings.

## References

- [HTTP overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)
- [HTTP methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)
- [HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)
- [Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)
- [DOM anatomy](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Anatomy_of_the_DOM)
- [Browser events](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Events)
- [Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
- [Angular templates](https://angular.dev/guide/templates)
