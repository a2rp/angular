# 2. Values, variables, and types

[Back to notes index](../README.md)

| [Previous: JavaScript foundations and runtime](01-javascript-foundations-and-runtime.md) | [Notes index](../README.md) | [Next: Operators and control flow](03-operators-and-control-flow.md) |
|:--|:--:|--:|

## JavaScript is dynamically typed

A variable can refer to values of different types during its lifetime. The value has a type, while the variable name refers to the current value.

~~~js
let result = 12;
result = 'complete';

console.log(result);
~~~

Dynamic typing is flexible, but it means code should check assumptions at important boundaries. Give variables descriptive names and avoid assigning unrelated kinds of values to the same variable.

## Choose const or let

Use const when a variable will not be assigned a new value. Use let when it will. Avoid var in new code because it is function-scoped and does not follow the block behavior most people expect.

~~~js
const courseName = 'JavaScript';
let completedChapters = 2;

completedChapters += 1;

console.log(courseName, completedChapters);
~~~

const prevents rebinding the name. It does not freeze an object or array stored in that variable.

~~~js
const learner = { name: 'Ashish' };
learner.name = 'Ashish Ranjan';

const topics = ['values'];
topics.push('variables');

console.log(learner.name, topics);
~~~

Use Object.freeze when you need a shallow frozen object, and remember it does not recursively freeze nested values. Immutability patterns are covered in Chapter 6.

## Understand scope and the temporal dead zone

let and const are block-scoped. A block is code between braces. A variable declared inside a block is not available outside it.

~~~js
if (true) {
  const message = 'Inside this block';
  console.log(message);
}

// console.log(message); // ReferenceError
~~~

A let or const name cannot be read before its declaration is initialized. The period before initialization is called the temporal dead zone.

~~~js
{
  // console.log(score); // ReferenceError
  const score = 10;
  console.log(score);
}
~~~

Prefer declaring a variable close to where it is used. This makes its lifetime and purpose easier to understand.

## The primitive types

JavaScript has seven primitive types:

- **string** represents text.
- **number** represents integers and floating-point values, plus NaN and infinities.
- **bigint** represents integers larger than the safe number range.
- **boolean** represents true or false.
- **undefined** usually means a value has not been assigned.
- **null** is an intentional absence of a value.
- **symbol** creates a unique identifier, often for specialized object keys.

Objects, arrays, and functions are reference values. Arrays and functions are also objects, though functions can be called.

~~~js
const language = 'JavaScript';
const chapterCount = 16;
const isPublished = true;
const missingValue = undefined;
const selectedNote = null;
const largeId = 9007199254740993n;
const internalKey = Symbol('internal');

console.log(typeof language); // string
console.log(typeof chapterCount); // number
console.log(typeof isPublished); // boolean
console.log(typeof largeId); // bigint
console.log(typeof internalKey); // symbol
~~~

Use undefined for an absent or not-yet-provided value when appropriate. Use null when the program intentionally sets a value to no selection or no result. Consistency across a project matters more than choosing one for every situation.

## Inspect values with typeof

typeof returns a string describing a value's broad type. There are two historical results that often surprise beginners: typeof null is "object", and arrays also return "object".

~~~js
console.log(typeof null); // object
console.log(typeof []); // object
console.log(typeof (() => {})); // function

console.log(Array.isArray([])); // true
~~~

Use Array.isArray to detect arrays. Use a direct null check for null. For many values, typeof is a first check, not a full validation of the value's shape.

## Work with numbers safely

The number type uses floating-point arithmetic. Some decimal fractions cannot be represented exactly in binary, so a calculation may have a small rounding difference.

~~~js
console.log(0.1 + 0.2); // 0.30000000000000004

const cents = Math.round((0.1 + 0.2) * 100);
console.log(cents); // 30
~~~

For money, store integer minor units such as cents or paise when that fits the application's needs, or use a decimal arithmetic library for calculations that need exact decimal behavior.

NaN means a numeric operation did not produce a valid number. Use Number.isNaN to check it. Use Number.isFinite when a value must be a finite number.

~~~js
const parsed = Number('not a number');

console.log(Number.isNaN(parsed)); // true
console.log(Number.isFinite(parsed)); // false
~~~

## Convert values explicitly

Use constructors such as Number, String, and Boolean when you want an explicit conversion. Be aware of the input and reject values that do not meet the application's requirements.

~~~js
console.log(Number('42')); // 42
console.log(String(42)); // "42"
console.log(Boolean('')); // false
console.log(Boolean('false')); // true
~~~

Boolean conversion follows truthiness, not the words inside a string. The non-empty string "false" is truthy. Do not use Boolean(input) to parse a text field that should contain only "true" or "false"; compare against the allowed strings instead.

Number can convert an empty string to zero and invalid text to NaN. For user input, check that the result is finite and within the expected range.

~~~js
function parsePositiveCount(input) {
  const value = Number(input);

  if (!Number.isInteger(value) || value < 1) {
    return null;
  }

  return value;
}

console.log(parsePositiveCount('4')); // 4
console.log(parsePositiveCount('four')); // null
~~~

parseInt can read an integer prefix from text. Pass the radix explicitly when parsing a particular base.

~~~js
console.log(parseInt('101', 2)); // 5
console.log(parseInt('24px', 10)); // 24
~~~

For validation, Number(input) plus a finite or integer check is often clearer than accepting a valid prefix and ignoring the rest.

## Compare values

Use strict equality with === and !== by default. Strict equality compares without converting between types.

~~~js
console.log(5 === 5); // true
console.log(5 === '5'); // false
console.log(5 == '5'); // true, because loose equality converts
~~~

Loose equality has special conversion rules and is easy to misread. Use it only when a specific conversion is intended and the behavior is understood.

Object.is is useful for its treatment of NaN and signed zero:

~~~js
console.log(Object.is(NaN, NaN)); // true
console.log(Object.is(0, -0)); // false
~~~

Objects are compared by identity, not by their contents.

~~~js
const first = { id: 1 };
const second = { id: 1 };
const sameReference = first;

console.log(first === second); // false
console.log(first === sameReference); // true
~~~

To compare object data, compare the relevant fields or use a deliberate comparison function. Converting to JSON is not a general deep equality solution because JSON omits or transforms some values.

## Use truthiness deliberately

Falsy values include false, 0, -0, 0n, an empty string, null, undefined, and NaN. Most other values are truthy, including empty arrays and empty objects.

~~~js
if ([]) {
  console.log('An empty array is truthy.');
}

if ({}) {
  console.log('An empty object is truthy.');
}
~~~

The logical OR operator returns the first truthy value, not necessarily a boolean. This can be useful for defaults but can incorrectly replace a meaningful zero or empty string.

~~~js
const oldPageSize = 0 || 20;
const selectedPageSize = 0 ?? 20;

console.log(oldPageSize); // 20
console.log(selectedPageSize); // 0
~~~

Use the nullish coalescing operator ?? when the fallback should apply only to null or undefined. Use || when any falsy value should trigger the fallback.

## Hands-on: validate a small input

Create a function that receives a value from a form or command line and returns a positive whole number or null. Test a whole number, decimal, blank input, negative number, and non-numeric text.

~~~js
function readPageNumber(input) {
  const value = Number(input);

  if (!Number.isInteger(value) || value < 1) {
    return null;
  }

  return value;
}

for (const input of ['3', '2.5', '', '-1', 'many']) {
  console.log(input, readPageNumber(input));
}
~~~

A blank string converts to zero, which is rejected by this function. If blank input needs a different result, check input.trim() before converting it.

## Notes to remember

- Variables refer to values, and a value has a type.
- Use const by default and let when a name must be reassigned.
- Prefer strict equality and explicit conversions.
- Arrays are objects, so use Array.isArray to detect them.
- Check numeric input with Number.isFinite or Number.isInteger.
- Empty arrays and objects are truthy.
- Use ?? for a fallback only when a value is null or undefined.

## References

- [MDN JavaScript data types and data structures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures)
- [MDN const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
- [MDN let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)
- [MDN typeof](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof)
- [MDN equality comparisons](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness)
- [MDN Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)

---

| [Previous: JavaScript foundations and runtime](01-javascript-foundations-and-runtime.md) | [Notes index](../README.md) | [Next: Operators and control flow](03-operators-and-control-flow.md) |
|:--|:--:|--:|
