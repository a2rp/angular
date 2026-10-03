# 10. Errors, debugging, and testing

[Back to notes index](../README.md)

| [Previous: Forms, validation, and browser storage](09-forms-validation-and-browser-storage.md) | [Notes index](../README.md) | [Next: Asynchronous JavaScript and promises](11-asynchronous-javascript-and-promises.md) |
|:--|:--:|--:|

## Read an error message

An error message usually includes a name, a description, and a stack trace. The stack trace shows the calls that led to the failure. Start with the first relevant application frame and inspect the values used there.

Common built-in error types include:

- **SyntaxError** means the source or parsed data has invalid syntax.
- **ReferenceError** means code tried to use a name that is not available.
- **TypeError** means an operation is not valid for the current value.
- **RangeError** means a value is outside an allowed range.

Do not guess from the final line alone. Read the error message, find the first source line in your code, and reproduce the same input or action.

## Throw and catch errors

Throw an Error when a function cannot complete its contract. Catch an error at a boundary where the program can recover, show a useful message, or add context.

~~~js
function calculateDiscount(price, rate) {
  if (!Number.isFinite(price) || price < 0) {
    throw new RangeError('Price must be a non-negative number.');
  }

  if (!Number.isFinite(rate) || rate < 0 || rate > 1) {
    throw new RangeError('Discount rate must be between zero and one.');
  }

  return price * (1 - rate);
}

try {
  console.log(calculateDiscount(100, 0.1));
} catch (error) {
  console.error('Could not calculate the price.', error);
}
~~~

Use specific error types when they help a caller distinguish cases. Do not catch an error only to ignore it; that can make a failed operation look successful.

~~~js
function readSavedSettings(text) {
  try {
    return JSON.parse(text);
  } catch (error) {
    if (error instanceof SyntaxError) {
      console.warn('Saved settings are invalid JSON.');
      return {};
    }

    throw error;
  }
}
~~~

This function recovers from malformed saved JSON but rethrows unexpected failures.

## Use finally for cleanup

A finally block runs whether the operation succeeds or throws. Use it for cleanup that must happen in either case.

~~~js
function doWork() {
  return 'done';
}

let isWorking = true;

try {
  console.log(doWork());
} catch (error) {
  console.error('Work failed.', error);
} finally {
  isWorking = false;
}
~~~

For asynchronous cleanup, use try, catch, and finally around await. Chapter 11 covers asynchronous error handling.

## Add useful debugging output

Use console methods according to their purpose. console.log prints information, console.warn highlights a concern, console.error reports a failure, and console.table displays rows of related data.

~~~js
const chapters = [
  { title: 'Values', complete: true },
  { title: 'Functions', complete: false },
];

console.table(chapters);
console.count('render');
console.time('filter');

const pending = chapters.filter((chapter) => !chapter.complete);

console.timeEnd('filter');
console.log('Pending chapters:', pending);
~~~

Remove temporary logs that expose private data or clutter production output. Do not log credentials, private form values, or sensitive API responses.

## Debug with breakpoints

A breakpoint pauses execution at a chosen line so you can inspect local variables and step through the program.

In browser developer tools:

1. Open Sources.
2. Find the loaded JavaScript file.
3. Click a line number to add a breakpoint.
4. Reproduce the action in the page.
5. Inspect local values and the call stack.
6. Step over, into, or out of a function.
7. Resume execution after checking the state.

A conditional breakpoint can pause only when an expression is true. This is useful for a loop that processes many records.

In Node.js, use the built-in inspector or run node with the inspect option, then connect a supported debugger. A single focused breakpoint often shows more than adding many logs.

## Write a small automated test

A unit test checks one small behavior in isolation. Node.js includes a test runner and assertion module, so a pure JavaScript function can be tested without an extra package.

~~~js
// math.mjs
export function add(left, right) {
  return left + right;
}
~~~

~~~js
// math.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import { add } from './math.mjs';

test('add returns the sum of two numbers', () => {
  assert.equal(add(2, 3), 5);
});
~~~

Run the test from the directory containing the files:

~~~powershell
node --test
~~~

A test names the expected behavior and makes an assertion about the result. If the assertion fails, Node reports the expected and actual values.

## Test normal cases and edge cases

A good test suite covers the normal result and important boundary conditions. The exact cases depend on the function contract.

~~~js
// discount.mjs
export function calculateDiscount(price, rate) {
  if (!Number.isFinite(price) || price < 0) {
    throw new RangeError('Price must be non-negative.');
  }

  if (!Number.isFinite(rate) || rate < 0 || rate > 1) {
    throw new RangeError('Rate must be between zero and one.');
  }

  return price * (1 - rate);
}
~~~

~~~js
// discount.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import { calculateDiscount } from './discount.mjs';

test('applies the discount rate', () => {
  assert.equal(calculateDiscount(100, 0.25), 75);
});

test('allows a zero discount', () => {
  assert.equal(calculateDiscount(80, 0), 80);
});

test('rejects a negative price', () => {
  assert.throws(
    () => calculateDiscount(-1, 0.1),
    { name: 'RangeError' },
  );
});
~~~

A test should not merely repeat the implementation line by line. It should check externally visible behavior that matters to the caller.

## Keep functions easy to test

Pure functions return values from their inputs without changing outside state. They are usually straightforward to test.

~~~js
function getPendingTitles(notes) {
  return notes
    .filter((note) => !note.reviewed)
    .map((note) => note.title);
}

const result = getPendingTitles([
  { title: 'Arrays', reviewed: false },
  { title: 'Objects', reviewed: true },
]);

console.log(result); // ['Arrays']
~~~

Code that reads the DOM, time, storage, or network has external dependencies. Keep those boundaries small so the main calculations can be tested with ordinary input values.

## Understand test levels

- A **unit test** checks a small function or module.
- An **integration test** checks that multiple parts work together.
- A **browser test** checks behavior in an actual browser page.

Start with the smallest useful test. Add broader tests for user workflows and important boundaries. Tests can verify expected behavior, but they do not prove that every possible input is correct.

## Notes to remember

- Read the error name, message, and first relevant stack frame.
- Throw Error objects when a function cannot meet its contract.
- Catch errors where there is a recovery or presentation decision.
- Do not swallow errors or log sensitive values.
- Breakpoints let you inspect values while code is paused.
- Test normal cases and important boundaries with assertions.
- Pure functions are easier to test than code coupled to browser globals.

## References

- [MDN JavaScript error handling](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling)
- [MDN Error](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error)
- [Node.js test runner](https://nodejs.org/api/test.html)
- [Node.js assert](https://nodejs.org/api/assert.html)
- [Chrome DevTools JavaScript debugging](https://developer.chrome.com/docs/devtools/javascript)

---

| [Previous: Forms, validation, and browser storage](09-forms-validation-and-browser-storage.md) | [Notes index](../README.md) | [Next: Asynchronous JavaScript and promises](11-asynchronous-javascript-and-promises.md) |
|:--|:--:|--:|
