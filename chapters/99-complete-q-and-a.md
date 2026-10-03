# 99. Complete Q&A

[Back to notes index](../README.md)

| [Previous: All code samples](98-all-code-samples.md) | [Notes index](../README.md) | [Next: Notes index](../README.md) |
|:--|:--:|--:|

This chapter collects common questions from the JavaScript notes in one place. Questions are grouped by topic so the related chapter is easy to find.

## JavaScript foundations and runtime

### Is JavaScript the same as ECMAScript?

ECMAScript is the language standard. JavaScript is the language implementation used by browsers and runtimes, with environment APIs added around that language.

### Does JavaScript run only in a browser?

No. Browsers run JavaScript for web pages, and Node.js runs it outside the browser for servers, scripts, and tools. Each environment supplies different APIs.

### Is the DOM part of JavaScript?

No. The DOM is a browser API that represents an HTML document. A Node.js process does not provide document by default.

### Why use a module script in a browser?

A module script supports import and export, has file-level scope, and is deferred until the page has been parsed. It is a good default for modern browser code.

### Why should a page be served locally instead of opened as a file?

Browser modules and security rules can behave differently under file URLs. A local server gives the page a normal origin and makes relative imports and requests behave more like a deployed site.

### What is the difference between a syntax error and a runtime error?

A syntax error prevents invalid source from being parsed. A runtime error happens while valid code is running, often only after a particular action or input.

### Where should I start when a page is blank?

Check the browser Console first. Then confirm the script path, the matching HTML selectors, and whether the script loaded successfully.

## Values, variables, and types

### What is the difference between a variable and a value?

A variable is a name that refers to a current value. The value has a type, and JavaScript lets a variable be assigned values of different types.

### Does const make an object immutable?

No. const prevents assigning a new value to the name. Properties inside an object or elements inside an array can still change unless the program prevents those changes.

### When should I use let instead of const?

Use let when the variable needs to be assigned again. Use const by default when the binding should stay fixed.

### What is the difference between null and undefined?

undefined often means a value is missing or has not been assigned. null is commonly used to represent an intentional absence, such as no selected item.

### Why does typeof null return object?

This is a historical behavior in JavaScript that remains for compatibility. Check null directly instead of relying on typeof for it.

### What does NaN mean?

NaN means a numeric operation did not produce a valid number. Use Number.isNaN to test for it and validate numeric input before further calculations.

### Why is an empty array truthy?

JavaScript treats objects as truthy, and arrays are objects. Check array.length when the program needs to know whether the array contains values.

## Operators and control flow

### What is the difference between || and ?? for defaults?

The || operator uses the fallback for any falsy value, including zero and an empty string. The ?? operator uses the fallback only for null or undefined.

### Do logical operators always return true or false?

No. They return one of their operands and may skip evaluating the other operand. This is called short-circuit evaluation.

### When should I use for...of?

Use for...of to visit values from arrays, strings, maps, sets, and other iterables.

### When should I use for...in?

Use for...in to enumerate enumerable string keys on an object. It is usually not the right way to read array values.

### Why should I use parentheses in a long expression?

Parentheses make the intended evaluation order visible. They reduce the need for readers to remember operator precedence.

### When is a ternary expression appropriate?

Use a ternary for a short choice between two values. Use if and else when either branch needs several statements or the condition is hard to understand at a glance.

### Why does switch sometimes need break?

Without break or return, execution continues into the next case. A return from the current function also exits the switch.

## Functions, scope, and closures

### What happens when a function does not return a value?

It returns undefined. Add a return statement when callers need a result.

### What is the difference between a parameter and an argument?

A parameter is a name in a function definition. An argument is the actual value supplied when the function is called.

### How is a function declaration different from a function expression?

A function declaration is initialized before statements in its scope run. A function expression is a value assigned when execution reaches that expression.

### Why does an arrow function behave differently with this?

An arrow function has no own this value. It reads this from the surrounding lexical scope, while a regular function's this depends on how it is called.

### What is a closure?

A closure is a function together with access to the variables in the scope where that function was created. The function can keep using those variables after the outer function returns.

### What is lexical scope?

Lexical scope means a function can access names from the place where the function was written, along with its own local names.

### Is a callback always asynchronous?

No. Array methods often call a callback immediately. Event listeners and timers call callbacks later, when their event or scheduled time occurs.

## Arrays and iteration

### What does map do?

map calls a function for each present array element and returns a new array containing the results. Use it when each input item should produce one output item.

### What does filter do?

filter returns a new array containing the elements whose callback result is truthy. It does not change the original array.

### What does reduce do?

reduce combines array values into one accumulated result. Supply an initial value, especially when the array might be empty.

### Which array methods mutate the original array?

Methods such as push, pop, shift, unshift, splice, and sort mutate the array. Methods such as map, filter, and slice return a new array.

### What is the difference between slice and splice?

slice returns a shallow copy of a selected range without changing the source. splice removes or inserts elements in the source array.

### Why can find return undefined?

find returns the first matching element, or undefined when no element matches. Check the result before reading its properties.

### Why does sorting numbers without a comparator look wrong?

sort compares values as strings by default. Provide a numeric comparator such as left minus right for numeric ordering.

## Objects, destructuring, and immutability

### When should I use dot notation or bracket notation?

Use dot notation when the property name is known in the source. Use brackets when the key is stored in a variable or the property name contains special characters.

### What does Object.hasOwn check?

It checks whether an object directly has a property. It does not report a property found only on the object's prototype.

### When does a destructuring default apply?

A default is used when the property is missing or undefined. It does not replace null, false, zero, or an empty string.

### Is object spread a deep copy?

No. Spread creates a shallow copy. Nested objects and arrays are still shared references unless they are copied separately.

### Does Object.freeze recursively freeze nested objects?

No. Object.freeze affects only the direct properties of the object. Nested values can still change unless they are frozen or otherwise protected.

### Why are two objects with the same fields not equal with ===?

Objects are compared by identity. Two separately created objects are different references even if their properties contain the same values.

### When is structuredClone useful?

It can make a deep copy of many supported data structures. It cannot clone every JavaScript value, such as functions, so check that it fits the data being copied.

## Strings, numbers, dates, and JSON

### Do string methods change the original string?

No. Strings are immutable. Methods such as trim and replace return a new string.

### What is a template literal?

It is a string delimited by backticks that can span lines and interpolate expressions using dollar-brace syntax.

### Why can 0.1 plus 0.2 be slightly different from 0.3?

JavaScript numbers use binary floating-point representation, and some decimal fractions cannot be represented exactly. Use an appropriate rounding strategy or decimal arithmetic for exact financial rules.

### Why use Intl formatters?

Intl.NumberFormat and Intl.DateTimeFormat format values according to locale conventions. They avoid hardcoding separators, currency placement, and date ordering.

### Why should I include a time zone in an ISO timestamp?

A time zone makes the represented instant explicit. Without one, date strings can be interpreted differently from the intended local time.

### Can JSON.stringify convert every JavaScript value?

No. JSON omits or transforms some values, does not support cycles, and cannot represent functions or symbols as ordinary JSON values.

### Does JSON.parse prove that a value has the expected fields?

No. It checks JSON syntax only. Validate the parsed shape before using it as application data.

## The DOM and browser events

### What if querySelector finds no element?

It returns null. Check the result before reading properties or calling methods on it.

### Why use textContent instead of innerHTML?

textContent treats the value as plain text. innerHTML parses markup, which can create a security problem when the string is untrusted.

### What is the difference between target and currentTarget?

target is where an event began. currentTarget is the element whose event listener is currently running.

### What is event delegation?

It is handling events from child elements with one listener on a shared parent. It works well for dynamic lists whose child buttons are created later.

### Why use a button instead of a clickable div?

A native button already supports keyboard activation, focus, and expected accessibility behavior. A div requires extra code to reproduce those features.

### Does querySelectorAll return an array?

No. It returns a static NodeList. It can be iterated directly, or converted with Array.from when array methods are needed.

### How do I remove an event listener?

Pass the same function reference and compatible options to removeEventListener. An AbortController signal can also stop a group of listeners together.

## Forms, validation, and browser storage

### Why does a form control need a name?

FormData and browser form submission use the name to identify each control's value.

### Why call preventDefault on submit?

The browser normally submits the form and may navigate or reload the page. preventDefault lets JavaScript handle the submission on the current page.

### Is browser validation enough to protect the server?

No. Client validation gives fast feedback, but users can bypass browser code. The server must validate every submitted value.

### What does localStorage store?

It stores string values for an origin across browser sessions. Use JSON when storing structured data, and validate values when reading them back.

### What is the difference between localStorage and sessionStorage?

localStorage persists across browser sessions. sessionStorage is limited to the current tab's page session.

### Is localStorage safe for passwords or private credentials?

No. JavaScript running on the page can read it, and a script injection can expose its contents. Use a security design appropriate to the server and application.

### Can storage calls fail?

Yes. Storage may be unavailable or full, and reading or writing can throw. Catch failures and tell the user when a save did not complete.

## Errors, debugging, and testing

### What information should I read in an error?

Read the error name and message, then inspect the first relevant source line in the stack trace. Reproduce the same input or action before changing code.

### Should every error be caught?

No. Catch an error where code can recover, show a useful message, or add context. Otherwise return or throw it so an appropriate caller can handle it.

### Why use finally?

finally runs whether a try block succeeds or throws. It is useful for cleanup that must always happen.

### What does a unit test check?

A unit test checks a small function or module in isolation. Integration and browser tests cover interactions among larger parts of the application.

### Why are pure functions easier to test?

Their outputs depend on their inputs rather than hidden state, time, storage, or the DOM. Tests can provide input and compare the result directly.

### How do I run the built-in Node.js test runner?

Place tests in files that match the runner's discovery rules and run node --test from the project directory.

### What is the purpose of a breakpoint?

A breakpoint pauses execution so you can inspect variables and step through calls at the moment a behavior occurs.

## Asynchronous JavaScript and promises

### What does a Promise represent?

A Promise represents an operation that is pending, fulfilled with a value, or rejected with a reason.

### Does await block the entire browser?

No. It suspends the current async function until the Promise settles. Other event-loop work can continue.

### Does an async function return a Promise?

Yes. It returns a Promise even when the function returns a plain value. A thrown error becomes a rejection.

### Why do Promise callbacks run before a zero-delay timer?

Promise reactions are microtasks. They are processed after the current synchronous stack and before the next timer task.

### When should I use Promise.all?

Use it when independent operations must all succeed before the next step. It rejects if any input Promise rejects.

### When should I use Promise.allSettled?

Use it when every operation's result matters even if some operations reject. Each result reports whether it fulfilled or rejected.

### Does a zero-delay timer run immediately?

No. It schedules a callback for a later event-loop task, after the current synchronous work and pending microtasks.

## Fetch and REST APIs

### Does fetch reject when the server returns 404?

Usually no. It fulfills with a Response, so check response.ok or the status code yourself.

### Is response.json synchronous?

No. It returns a Promise because the response body must be read and parsed asynchronously.

### Why use URLSearchParams?

It safely encodes query names and values. This avoids manually concatenating untrusted text into the URL.

### What does CORS protect?

CORS controls which cross-origin responses a browser allows page scripts to read. It is not authentication or authorization for the server.

### How do I stop a fetch request?

Pass an AbortController signal to fetch and call abort when the response is no longer needed.

### Should I parse JSON from a 204 response?

No. A 204 response has no content. Check the endpoint contract and do not call response.json when the body is empty.

### Does fetch's generic usage validate response fields?

Fetch does not supply runtime validation. Read the response data and check the fields the application needs.

## Modules, npm, and project structure

### What does an ES module provide?

It provides file-level scope and explicit import and export statements. This helps organize code and show dependencies.

### What is the difference between a named and default export?

A module can have multiple named exports that are imported by their exported names. A module has at most one default export, which the importing file can name locally.

### What does the type field set to module do in Node.js?

It tells Node.js to interpret .js files in that package as ECMAScript modules. The .mjs extension also marks an individual module file.

### Should node_modules be committed?

No. Commit package.json and package-lock.json, then install dependencies from those files. node_modules is generated by the package manager.

### Why keep package-lock.json?

It records the resolved dependency tree so installs are more reproducible. npm ci uses it in automated or clean installations.

### What does dynamic import return?

It returns a Promise that fulfills with the loaded module's exports or rejects if loading fails.

### Why include .js on a browser module import?

Native browser modules resolve the specified file path. Include the actual extension so the browser can request the intended file.

## Prototypes, this, and classes

### What is a prototype chain?

It is the sequence of prototype objects JavaScript checks when a property is not found directly on an object.

### What decides this in a regular method?

The call site does. Calling object.method() supplies object as the receiver; detaching method removes that receiver unless the function is bound.

### What is special about this in an arrow function?

An arrow function inherits this from its surrounding scope and does not create a new this of its own.

### Where do class methods live?

Methods defined in the class body are stored on the class prototype and shared by instances. Instance fields are stored on each instance.

### What does a private field do?

A field whose name begins with # can only be accessed inside the declaring class body.

### When should a class extend another class?

Use inheritance when the subclass is a true specialized form of the base class and can stand in for it. Use composition when an object collaborates with another object.

### Do related functions always need a class?

No. A plain object or factory function may be simpler. Choose a class when shared instance behavior and state make the relationship clearer.

## Collections, iterators, and generators

### When should I use Map instead of an object?

Use Map when keys can be any value, insertion order and size are useful, or the values form a dynamic key-value collection. Use an object for a record with known named fields.

### What does Set guarantee?

A Set stores each value only once and preserves insertion order.

### Why can I not loop over a WeakMap?

WeakMap is intentionally not enumerable because the runtime can remove keys as objects become unreachable. Use Map when the application needs iteration or a size.

### What does an iterable provide?

It provides an iterator through Symbol.iterator. for...of, spread syntax, and other language features can consume iterable values.

### What does a generator function return?

Calling a generator function returns an iterator. It produces values when next is called or when a consumer such as for...of requests them.

### When should I use a typed array?

Use one when an API needs fixed-type binary data, such as bytes for a file, image, audio, or network format.

### Is Map automatically converted to JSON?

No. Convert its entries to an array or another explicit data structure before serializing them.

## Browser security, accessibility, and performance

### Why should plain user text go through textContent?

It is treated as text instead of being parsed as markup. This avoids turning user-controlled text into executable page content.

### Why validate a URL protocol?

A URL can use schemes beyond HTTP and HTTPS. Allow only the schemes the feature intends to open.

### Does CORS protect a private API?

No. The server must authenticate the caller and authorize each protected operation. CORS only affects browser access to cross-origin responses.

### Why use native buttons and links?

They provide expected keyboard, focus, and assistive technology behavior. Native controls also reduce the amount of custom behavior the page must maintain.

### When should a live region be used?

Use it to announce a meaningful status change that may otherwise be missed. Choose polite feedback for routine updates and an alert for urgent errors.

### How should I decide whether to optimize code?

Measure the behavior with representative input first. Improve the part shown to be a real bottleneck and measure again.

### Why clean up listeners and timers?

They can keep functions and data alive after a view is removed. Removing them prevents stale behavior and unnecessary memory retention.

## Notes index

- [JavaScript foundations and runtime](01-javascript-foundations-and-runtime.md)
- [Values, variables, and types](02-values-variables-and-types.md)
- [Operators and control flow](03-operators-and-control-flow.md)
- [Functions, scope, and closures](04-functions-scope-and-closures.md)
- [Arrays and iteration](05-arrays-and-iteration.md)
- [Objects, destructuring, and immutability](06-objects-destructuring-and-immutability.md)
- [Strings, numbers, dates, and JSON](07-strings-numbers-dates-and-json.md)
- [The DOM and browser events](08-dom-and-browser-events.md)
- [Forms, validation, and browser storage](09-forms-validation-and-browser-storage.md)
- [Errors, debugging, and testing](10-errors-debugging-and-testing.md)
- [Asynchronous JavaScript and promises](11-asynchronous-javascript-and-promises.md)
- [Fetch and REST APIs](12-fetch-and-rest-apis.md)
- [Modules, npm, and project structure](13-modules-npm-and-project-structure.md)
- [Prototypes, this, and classes](14-prototypes-this-and-classes.md)
- [Collections, iterators, and generators](15-collections-iterators-and-generators.md)
- [Browser security, accessibility, and performance](16-browser-security-accessibility-and-performance.md)
- [All code samples](98-all-code-samples.md)

---

| [Previous: All code samples](98-all-code-samples.md) | [Notes index](../README.md) | [Next: Notes index](../README.md) |
|:--|:--:|--:|
