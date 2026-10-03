# 4. Functions, scope, and closures

[Back to notes index](../README.md)

| [Previous: Operators and control flow](03-operators-and-control-flow.md) | [Notes index](../README.md) | [Next: Arrays and iteration](05-arrays-and-iteration.md) |
|:--|:--:|--:|

## A function packages behavior

A function is a reusable block of code. It can receive input through parameters and send a result back with return. Names should describe the action or result.

~~~js
function calculateArea(width, height) {
  return width * height;
}

const area = calculateArea(4, 6);
console.log(area); // 24
~~~

When a function reaches return, it stops running and returns that value. A function with no return statement returns undefined.

~~~js
function logTopic(topic) {
  console.log('Topic:', topic);
}

const result = logTopic('functions');
console.log(result); // undefined
~~~

Keep one function focused on one task. Smaller functions are easier to test, reuse, and understand.

## Function declarations and expressions

A function declaration names a function directly. Function declarations are initialized before their scope's statements run, so they can be called earlier in the same scope.

~~~js
console.log(double(4)); // 8

function double(value) {
  return value * 2;
}
~~~

A function expression stores a function in a variable. With const, the name cannot be used before the declaration has initialized.

~~~js
const triple = function (value) {
  return value * 3;
};

console.log(triple(4)); // 12
~~~

Both forms are useful. Use declarations for named functions that form part of a module's main API. Use expressions when a function is a value passed into another operation or assigned conditionally.

## Arrow functions

Arrow functions provide concise function expressions. They are common for callbacks and short transformations.

~~~js
const double = (value) => value * 2;
const add = (left, right) => left + right;
const createLabel = (name) => {
  const cleanedName = name.trim();
  return 'Learner: ' + cleanedName;
};

console.log(double(5));
console.log(add(2, 3));
console.log(createLabel(' Ashish '));
~~~

When an arrow function has one parameter, parentheses may be omitted, though keeping them can make code more consistent. Use braces and return for multiple statements. Arrow functions do not have their own this value; Chapter 14 explains that difference.

## Pass parameters and return values

Parameters receive values supplied by a caller. A default parameter is used when the argument is missing or undefined.

~~~js
function greet(name = 'Guest') {
  return 'Hello, ' + name;
}

console.log(greet()); // Hello, Guest
console.log(greet('Ashish')); // Hello, Ashish
~~~

A default is not used for null or an empty string.

~~~js
console.log(greet(null)); // Hello, null
console.log(greet('')); // Hello,
~~~

Use rest parameters when a function accepts a variable number of arguments. The rest parameter gathers them into an array and must be last.

~~~js
function sum(...values) {
  return values.reduce((total, value) => total + value, 0);
}

console.log(sum(2, 3, 5)); // 10
~~~

Do not rely on the older arguments object in new functions. Rest parameters make the input shape clear and work with arrow functions.

## Functions are values

A function can be stored in a variable, passed to another function, or returned from a function. A function that receives or returns another function is often called a higher-order function.

~~~js
function applyOperation(value, operation) {
  return operation(value);
}

const result = applyOperation(5, (number) => number * number);
console.log(result); // 25
~~~

Callbacks are functions given to another operation to run at a chosen time. Array methods such as map and filter receive callbacks. Browser event listeners also receive callbacks.

~~~js
const topics = ['values', 'functions', 'objects'];
const labels = topics.map((topic) => topic.toUpperCase());

console.log(labels);
~~~

A callback may run immediately or later. Keep that timing in mind when reading code that handles events, promises, or timers.

## Lexical scope

Scope determines where a name can be accessed. JavaScript uses lexical scope: a function can access names from the place where it was written, plus its own local names.

~~~js
const course = 'JavaScript';

function showCourse() {
  const message = 'Studying ' + course;
  return message;
}

console.log(showCourse());
~~~

A nested function can read variables in its outer function.

~~~js
function makeLabel(topic) {
  const prefix = 'Study: ';

  function format() {
    return prefix + topic;
  }

  return format();
}

console.log(makeLabel('scope'));
~~~

Prefer local variables and explicit parameters. Avoid depending on mutable global variables because any part of the program may change them.

## Closures keep access to outer variables

A closure is a function together with access to the lexical environment where that function was created. The inner function can continue to read variables from an outer call even after that outer function has returned.

~~~js
function createCounter() {
  let count = 0;

  return function increment() {
    count += 1;
    return count;
  };
}

const nextCount = createCounter();

console.log(nextCount()); // 1
console.log(nextCount()); // 2
~~~

Each call to createCounter creates a separate count.

~~~js
const firstCounter = createCounter();
const secondCounter = createCounter();

console.log(firstCounter()); // 1
console.log(firstCounter()); // 2
console.log(secondCounter()); // 1
~~~

Closures are useful for callbacks, private state, and functions configured with a value. They can also keep data in memory as long as a function still references it, so avoid retaining large unused objects through long-lived handlers.

## Scope and loops

A loop that declares its index with let creates a separate binding for each iteration. This is useful when a callback runs later.

~~~js
const callbacks = [];

for (let index = 0; index < 3; index += 1) {
  callbacks.push(() => index);
}

console.log(callbacks.map((getIndex) => getIndex())); // [0, 1, 2]
~~~

Using var would create one function-scoped binding shared by those callbacks. Prefer const or let in modern code and keep the declaration in the smallest scope that needs it.

## Keep return paths clear

A function can return early when input is missing or invalid. This avoids nesting the main logic inside several conditions.

~~~js
function getInitials(name) {
  if (typeof name !== 'string') {
    return '';
  }

  const words = name.trim().split(/\s+/).filter(Boolean);

  if (words.length === 0) {
    return '';
  }

  return words
    .map((word) => word[0].toUpperCase())
    .join('');
}

console.log(getInitials('Ashish Ranjan')); // AR
~~~

A function should communicate its result for every expected input. Decide whether invalid input should return a fallback, return a structured result, or throw an error.

## Hands-on: create a configurable formatter

Write a function that accepts a prefix and returns another function. The returned function formats any supplied topic using the saved prefix.

~~~js
function createFormatter(prefix) {
  return function format(topic) {
    return prefix + ': ' + topic.trim();
  };
}

const noteLabel = createFormatter('Study note');

console.log(noteLabel(' functions '));
console.log(noteLabel(' closures '));
~~~

Then create a second formatter with another prefix. Notice that each returned function keeps its own prefix because each call created a separate closure.

## Notes to remember

- A function can receive parameters and return a result.
- Function declarations and function expressions initialize at different times.
- Arrow functions are concise and use lexical this.
- Functions are values and can be passed as callbacks.
- Lexical scope follows where code is written.
- Closures retain access to outer variables.
- Prefer local state and clear inputs over mutable globals.

## References

- [MDN functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions)
- [MDN function declarations](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)
- [MDN arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [MDN default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)
- [MDN rest parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)
- [MDN closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)

---

| [Previous: Operators and control flow](03-operators-and-control-flow.md) | [Notes index](../README.md) | [Next: Arrays and iteration](05-arrays-and-iteration.md) |
|:--|:--:|--:|
