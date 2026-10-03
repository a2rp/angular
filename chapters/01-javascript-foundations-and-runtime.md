# 1. JavaScript foundations and runtime

[Back to notes index](../README.md)

| [Previous: Notes index](../README.md) | [Notes index](../README.md) | [Next: Values, variables, and types](02-values-variables-and-types.md) |
|:--|:--:|--:|

## What JavaScript is

JavaScript is a programming language standardized as ECMAScript. It can run in web browsers and in other environments such as Node.js. The language describes values, functions, objects, modules, and control flow. The environment provides additional capabilities.

In a browser, JavaScript can read and update the page, respond to user input, store data, and make network requests. Node.js provides APIs for server programs, command-line tools, file access, and other work outside the browser. The same language syntax can be used in both places, but their built-in APIs are different.

The browser DOM is a web platform API, not part of the JavaScript language itself. For example, console.log is available in many JavaScript environments, while document.querySelector is provided by a browser page.

## Source code becomes running behavior

A JavaScript engine parses source code, checks its syntax, and executes it. A syntax error prevents the affected file from running. A runtime error happens while valid code is executing, often after a particular path is reached.

~~~js
const greeting = 'Hello, JavaScript';
console.log(greeting);
~~~

This code creates a string value and writes it to the console. Open the browser developer tools and use the Console panel to try short expressions. The console is useful for inspecting values and errors, but it is not a replacement for keeping useful code in project files.

## Run a browser script

A web page can load a JavaScript file with a script element. A module script is a good default for new browser projects because it supports import and export and is deferred until the document has been parsed.

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>JavaScript practice</title>
    <script type="module" src="./main.js"></script>
  </head>
  <body>
    <main>
      <h1>JavaScript practice</h1>
      <p id="message">The page is ready.</p>
    </main>
  </body>
</html>
~~~

~~~js
const message = document.querySelector('#message');

if (message) {
  message.textContent = 'JavaScript updated this page.';
}
~~~

Save the files in the same directory and serve the page through a local development server. Module imports and browser security rules can behave differently when a page is opened directly from the file system.

The script runs after the document has been parsed because it is a module. With a classic script, defer is another option when the script needs elements that appear later in the document. Avoid placing a large script in the head without module or defer behavior when it depends on page elements.

## Use browser developer tools

The browser developer tools help inspect the page and understand what code is doing.

- **Console** shows logged values, warnings, and runtime errors.
- **Elements** shows the live DOM and styles.
- **Sources** lets you set breakpoints and step through execution.
- **Network** shows requests, responses, status codes, and timing.
- **Application or Storage** shows browser-managed storage and cache state.

Use console.log for values during practice. Remove noisy temporary logs before sharing or deploying an application. For a problem you cannot see, reproduce it, open the developer tools, and inspect the first relevant error.

~~~js
const price = 12.5;
const quantity = 3;
const total = price * quantity;

console.log({ price, quantity, total });
~~~

Passing an object to the console makes related values easier to inspect together.

## Run JavaScript with Node.js

Node.js runs JavaScript outside a browser. Install a supported Node.js release, then use node to run a file or open its interactive prompt.

~~~powershell
node --version
node
node practice.js
~~~

The interactive prompt evaluates expressions as you enter them. Use Ctrl+C twice or Ctrl+D to exit. In a Node.js file, browser globals such as document and window are not present. Node has its own APIs for files, paths, processes, and network servers.

The globalThis object refers to the global object of the current environment. Prefer explicit imports and APIs instead of relying on environment-specific global values.

~~~js
console.log(globalThis === globalThis);
~~~

This expression is true in both environments, but other properties available on globalThis differ. A browser page exposes window and document; a Node.js process exposes process and Node APIs.

## Make the first interactive example

A common first browser interaction is responding to a click. The HTML creates a button and a message area. JavaScript selects them and attaches an event listener.

~~~html
<button id="greet-button" type="button">Say hello</button>
<p id="greeting" aria-live="polite">Waiting for a click.</p>
~~~

~~~js
const button = document.querySelector('#greet-button');
const greeting = document.querySelector('#greeting');

if (button instanceof HTMLButtonElement && greeting) {
  button.addEventListener('click', () => {
    greeting.textContent = 'Hello from JavaScript.';
  });
}
~~~

The event listener function runs after the user clicks the button. The if statement checks that the page contains the expected button before trying to use it. DOM selection, event handling, and accessibility are covered in Chapter 8.

## Comments and readable source

A single-line comment begins with two forward slashes. A block comment begins with slash-star and ends with star-slash. Comments can explain why a choice is important, but clear names and small functions should explain most of what the code does.

~~~js
// Keep the user's original input for the form.
const enteredName = 'Ashish';

/*
  Normalize the value before comparing it with
  another name.
*/
const normalizedName = enteredName.trim().toLowerCase();
~~~

Do not leave a large disabled block of old code in a file. Version control keeps the history, and unused code makes the active behavior harder to find.

## JavaScript and the browser page are separate

A browser page typically combines HTML for structure, CSS for presentation, and JavaScript for behavior. A .js file contains JavaScript. A .html file contains document markup, though it can load scripts and styles.

Keep this separation visible while learning:

- Use HTML elements that express what the content is.
- Use CSS for layout and visual presentation.
- Use JavaScript for behavior, data changes, and responding to events.

The separation makes it easier to find an issue and test one part at a time. JavaScript can create or change DOM elements, but putting all markup into large strings is not usually the easiest starting point for a small page.

## Hands-on: run a local page

1. Create a folder named js-practice.
2. Add index.html and main.js from the browser example.
3. Start a local development server from the editor or a small static server.
4. Open the page and check the browser Console for errors.
5. Change the message text in main.js and reload.
6. Add the button example and click it.
7. Create practice.js and run it with node practice.js.
8. Compare which browser and Node.js globals are available.

If the page is blank, check the script URL, open the developer tools, and verify that the HTML element IDs match the selectors. If Node.js reports that it cannot find the file, confirm the current terminal directory and the spelling of the file name.

## Notes to remember

- ECMAScript standardizes the language, while each runtime supplies additional APIs.
- Browsers provide the DOM and browser developer tools.
- Node.js runs JavaScript outside the browser and provides different APIs.
- A module script supports imports and runs after the document is parsed.
- The Console and debugger help inspect values and locate errors.
- Separate page structure, presentation, and behavior while learning.

## References

- [MDN JavaScript Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)
- [MDN JavaScript first steps](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/What_is_JavaScript)
- [MDN JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [MDN Console API](https://developer.mozilla.org/en-US/docs/Web/API/Console)
- [Node.js introduction](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs)
- [ECMAScript language specification](https://tc39.es/ecma262/)

---

| [Previous: Notes index](../README.md) | [Notes index](../README.md) | [Next: Values, variables, and types](02-values-variables-and-types.md) |
|:--|:--:|--:|
