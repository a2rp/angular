# 3. Operators and control flow

[Back to notes index](../README.md)

| [Previous: Values, variables, and types](02-values-variables-and-types.md) | [Notes index](../README.md) | [Next: Functions, scope, and closures](04-functions-scope-and-closures.md) |
|:--|:--:|--:|

## Operators produce or update values

Operators combine values or change a variable. Arithmetic operators include addition, subtraction, multiplication, division, remainder, and exponentiation.

~~~js
const subtotal = 4 * 25;
const remainder = 17 % 5;
const squared = 3 ** 2;

console.log(subtotal); // 100
console.log(remainder); // 2
console.log(squared); // 9
~~~

The plus operator adds numbers and joins strings. If one operand is a string, the result may be string concatenation. Convert values deliberately when working with form input.

~~~js
console.log(2 + 3); // 5
console.log('2' + 3); // "23"
console.log(Number('2') + 3); // 5
~~~

Assignment operators store a value. Compound assignment updates a variable using its current value.

~~~js
let total = 10;
total += 5;
total *= 2;

console.log(total); // 30
~~~

## Use precedence carefully

JavaScript applies operators according to precedence rules. Parentheses make the intended order visible and can prevent mistakes when an expression combines several operations.

~~~js
const average = (8 + 10 + 12) / 3;
const isInRange = age >= 18 && age <= 65;
~~~

Do not rely on memorizing every precedence level. Use parentheses when they clarify a non-obvious expression, and split a long expression into named variables.

## Compare without accidental conversion

Comparison operators include greater than, less than, and their inclusive forms. Strict equality checks whether values are equal without converting between different types.

~~~js
const score = 82;

console.log(score >= 60); // true
console.log(score === 82); // true
console.log(score !== '82'); // true
~~~

Comparisons with NaN are always false, including NaN === NaN. Use Number.isNaN or validate a number before comparing it.

## Combine conditions with logical operators

The logical AND operator && returns the first falsy operand or the final operand. The logical OR operator || returns the first truthy operand or the final operand. They can short-circuit, so the second expression only runs when needed.

~~~js
const hasName = name !== '';
const canSubmit = hasName && isFormValid;
const displayName = name || 'Guest';
~~~

Nullish coalescing returns the right operand only when the left side is null or undefined. Optional chaining stops a property access when the value to its left is null or undefined.

~~~js
const pageSize = savedPageSize ?? 20;
const city = account.profile?.address?.city ?? 'Unknown';
~~~

Use optional chaining when a missing value is an expected possibility. It should not hide a programming error where a required object should always exist.

## Choose a branch

Use if and else when a condition controls which statements run. Add early returns to keep exceptional or invalid cases near the top of a function.

~~~js
function getResultLabel(score) {
  if (!Number.isFinite(score)) {
    return 'Score is missing';
  }

  if (score >= 80) {
    return 'Strong result';
  }

  if (score >= 50) {
    return 'Keep practicing';
  }

  return 'Review the basics';
}
~~~

A ternary expression chooses between two values. Use it for a short value selection, not as a replacement for a long if statement.

~~~js
const statusLabel = isComplete ? 'Complete' : 'In progress';
~~~

## Use switch for one value with several cases

A switch statement compares one expression to case values. Include break after a case unless execution should continue into the next case. A return inside a function also exits the switch.

~~~js
function getDayType(day) {
  switch (day) {
    case 'Saturday':
    case 'Sunday':
      return 'Weekend';
    case 'Monday':
    case 'Tuesday':
    case 'Wednesday':
    case 'Thursday':
    case 'Friday':
      return 'Weekday';
    default:
      return 'Unknown day';
  }
}
~~~

Group cases with the same result. A default case provides behavior for values that were not listed.

## Repeat work with loops

Use a for loop when the number of iterations or index matters. The initialization runs once, the condition is checked before each iteration, and the update runs after each iteration.

~~~js
const chapters = ['values', 'functions', 'objects'];

for (let index = 0; index < chapters.length; index += 1) {
  console.log(index + 1, chapters[index]);
}
~~~

A while loop repeats as long as its condition is true. Make sure something in the loop changes the condition, or the program can run forever.

~~~js
let attempts = 0;

while (attempts < 3) {
  console.log('Attempt', attempts + 1);
  attempts += 1;
}
~~~

Use break to leave a loop and continue to skip the rest of the current iteration. Prefer a clear condition over many break or continue statements.

## Choose the right collection loop

Use for...of to read values from an iterable such as an array or string.

~~~js
const topics = ['arrays', 'objects', 'functions'];

for (const topic of topics) {
  console.log(topic);
}
~~~

Use for...in to enumerate an object's enumerable string keys. It is usually not the right loop for an array because it iterates keys rather than values and can include inherited enumerable properties.

~~~js
const learner = { name: 'Ashish', city: 'Bengaluru' };

for (const key in learner) {
  if (Object.hasOwn(learner, key)) {
    console.log(key, learner[key]);
  }
}
~~~

Object.keys, Object.values, and Object.entries return arrays of an object's own enumerable properties and can be easier to work with than for...in. Chapter 6 covers these object operations.

## Use array methods for transformations

When the goal is to transform or select array values, methods such as map and filter describe the result directly.

~~~js
const prices = [10, 25, 40];
const taxedPrices = prices.map((price) => price * 1.1);
const expensivePrices = prices.filter((price) => price >= 25);

console.log(taxedPrices);
console.log(expensivePrices);
~~~

Use a loop when the work has side effects or complex control flow. Use array methods when a transformation or selection reads more clearly. Chapter 5 covers the main array methods.

## Hands-on: calculate a cart total

Build a function that ignores invalid prices, adds the remaining values, and applies a discount only when the subtotal is high enough.

~~~js
function getCartTotal(prices, discountThreshold, discountRate) {
  let subtotal = 0;

  for (const price of prices) {
    if (!Number.isFinite(price) || price < 0) {
      continue;
    }

    subtotal += price;
  }

  if (subtotal >= discountThreshold) {
    return subtotal * (1 - discountRate);
  }

  return subtotal;
}

console.log(getCartTotal([20, 35, -4, 15], 60, 0.1)); // 63
~~~

Try different inputs: an empty cart, a cart below the threshold, and values containing a negative number or text. Decide whether invalid prices should be ignored or rejected, then make the behavior explicit.

## Notes to remember

- Parentheses make expression order easier to verify.
- Use strict comparisons and convert input before comparing it.
- Logical operators short-circuit and return operand values.
- Use if for branching and a short ternary for simple value selection.
- Use for...of for values in arrays and other iterables.
- Use for...in for object keys only when its behavior is intended.
- Use map and filter when they clearly express an array transformation.

## References

- [MDN expressions and operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_operators)
- [MDN conditional statements](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling)
- [MDN loops and iteration](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration)
- [MDN for...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)
- [MDN for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in)

---

| [Previous: Values, variables, and types](02-values-variables-and-types.md) | [Notes index](../README.md) | [Next: Functions, scope, and closures](04-functions-scope-and-closures.md) |
|:--|:--:|--:|
