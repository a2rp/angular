# 5. Arrays and iteration

[Back to notes index](../README.md)

| [Previous: Functions, scope, and closures](04-functions-scope-and-closures.md) | [Notes index](../README.md) | [Next: Objects, destructuring, and immutability](06-objects-destructuring-and-immutability.md) |
|:--|:--:|--:|

## Store ordered values in an array

An array stores an ordered list of values. Its index starts at zero, and its length is the number of positions in the array.

~~~js
const topics = ['values', 'functions', 'objects'];

console.log(topics[0]); // values
console.log(topics.length); // 3
console.log(topics[topics.length - 1]); // objects
~~~

Reading an index that does not exist returns undefined. An array can contain values of different types, but a consistent shape is easier to process.

~~~js
const mixed = ['chapter', 5, true];
console.log(mixed[10]); // undefined
~~~

Avoid sparse arrays with empty slots. They behave differently from arrays that contain explicit undefined values and can make iteration methods surprising.

## Add and remove items

push adds one or more values to the end and returns the new length. pop removes and returns the last value. unshift and shift add or remove values at the start.

~~~js
const queue = ['review', 'practice'];

queue.push('summarize');
const firstTask = queue.shift();

console.log(firstTask); // review
console.log(queue); // ['practice', 'summarize']
~~~

These methods mutate the original array. Use them when changing that array is intended. If another part of the program relies on the original value, make a copy first or use a non-mutating method.

splice changes an array by removing or inserting values. slice returns a shallow copy of a selected range without changing the original.

~~~js
const original = ['a', 'b', 'c', 'd'];
const middle = original.slice(1, 3);
const removed = original.splice(1, 2);

console.log(middle); // ['b', 'c']
console.log(removed); // ['b', 'c']
console.log(original); // ['a', 'd']
~~~

The end index for slice is not included. splice takes a starting index and a delete count, then can accept values to insert.

## Iterate over values

Use for...of when each iteration needs the item value.

~~~js
const scores = [72, 88, 91];

for (const score of scores) {
  console.log(score);
}
~~~

forEach calls a callback for each present element and returns undefined. It is useful for straightforward side effects, but it cannot be stopped with break.

~~~js
scores.forEach((score, index) => {
  console.log(index, score);
});
~~~

Use a for loop or for...of when you need to break early, wait with await in sequence, or control the index. Use for...in for object keys, not for array values.

## Transform with map

map returns a new array containing the result of calling a function for every present element. It keeps the number and order of visited entries.

~~~js
const prices = [10, 20, 30];
const pricesWithTax = prices.map((price) => price * 1.05);

console.log(pricesWithTax); // [10.5, 21, 31.5]
console.log(prices); // [10, 20, 30]
~~~

Use map when every input item should produce one output item. Return the new value from the callback. If the goal is only a side effect, forEach communicates that purpose better.

~~~js
const notes = [
  { title: 'Arrays', reviewed: true },
  { title: 'Objects', reviewed: false },
];

const titles = notes.map((note) => note.title);
console.log(titles); // ['Arrays', 'Objects']
~~~

## Select with filter

filter returns a new array containing the elements whose callback result is truthy.

~~~js
const reviewedNotes = notes.filter((note) => note.reviewed);
const notesToReview = notes.filter((note) => !note.reviewed);

console.log(reviewedNotes);
console.log(notesToReview);
~~~

The original array is unchanged. An empty result is still an array, so check its length when the view needs an empty state.

## Find an item

find returns the first matching value or undefined. findIndex returns the index of the first match or -1.

~~~js
const selected = notes.find((note) => note.title === 'Objects');
const selectedIndex = notes.findIndex((note) => note.title === 'Objects');

console.log(selected);
console.log(selectedIndex);
~~~

Check the result before reading its properties because the item may not exist.

~~~js
if (selected) {
  console.log(selected.title);
}
~~~

Use some to check whether at least one element matches and every to check whether all elements match.

~~~js
const hasPendingNotes = notes.some((note) => !note.reviewed);
const allNotesReviewed = notes.every((note) => note.reviewed);

console.log(hasPendingNotes, allNotesReviewed);
~~~

For an empty array, some returns false and every returns true. This follows their logical meaning: no item satisfies some, and no item violates every.

## Combine values with reduce

reduce processes an array into one accumulated result. Provide an initial accumulator value so the behavior is clear, including for an empty array.

~~~js
const lineItems = [
  { price: 12, quantity: 2 },
  { price: 8, quantity: 3 },
];

const total = lineItems.reduce(
  (sum, item) => sum + item.price * item.quantity,
  0,
);

console.log(total); // 48
~~~

Do not reach for reduce just because it can express many operations. map, filter, and find are often easier to read for their specific jobs.

## Sort without changing the source list

sort changes the array it is called on and compares values as strings by default. For numbers, provide a comparator.

~~~js
const values = [100, 4, 25];
const ascending = [...values].sort((left, right) => left - right);

console.log(ascending); // [4, 25, 100]
console.log(values); // [100, 4, 25]
~~~

Modern JavaScript also provides toSorted, which returns a sorted copy. Check the environments supported by the application before relying on newer methods.

~~~js
const sortedTitles = notes
  .map((note) => note.title)
  .toSorted((left, right) => left.localeCompare(right));
~~~

For user-facing text, localeCompare can produce more appropriate alphabetic ordering than comparing strings with less-than. Sorting rules can depend on language and locale.

## Copy and combine arrays

Spread syntax can copy or combine arrays. The copy is shallow, so nested objects are still shared references.

~~~js
const firstGroup = ['values', 'functions'];
const allTopics = [...firstGroup, 'arrays'];
const copy = [...firstGroup];

console.log(allTopics);
console.log(copy);
~~~

Changing the outer array does not change the source array, but changing a nested object in a shallow copy can affect both arrays. Chapter 6 covers object references and immutability.

## Avoid mutating during iteration

Mutating the array being traversed can skip elements or change which values are visited. When removing items based on a condition, filter into a new array.

~~~js
const numbers = [1, 2, 3, 4, 5];
const evenNumbers = numbers.filter((number) => number % 2 === 0);

console.log(evenNumbers); // [2, 4]
~~~

If a mutation is required, iterate over a copy or carefully control the index. Prefer code that makes the resulting collection explicit.

## Hands-on: summarize note data

Use filter to keep unreviewed notes, map to read their titles, and reduce to count the total words.

~~~js
const studyNotes = [
  { title: 'Arrays', reviewed: false, words: 420 },
  { title: 'Functions', reviewed: true, words: 510 },
  { title: 'Objects', reviewed: false, words: 360 },
];

const pendingTitles = studyNotes
  .filter((note) => !note.reviewed)
  .map((note) => note.title);

const pendingWordCount = studyNotes
  .filter((note) => !note.reviewed)
  .reduce((total, note) => total + note.words, 0);

console.log(pendingTitles); // ['Arrays', 'Objects']
console.log(pendingWordCount); // 780
~~~

Try adding a reviewed note and an empty array. Check that the output still matches the expected result.

## Notes to remember

- Array indexes start at zero, and reading a missing index returns undefined.
- push, pop, shift, unshift, splice, and sort mutate an array.
- map transforms each item, while filter selects matching items.
- find can return undefined; check before using the result.
- Give reduce an initial value and prefer more specific methods when they are clearer.
- Use a comparator for numeric sorting and copy first when the source should remain unchanged.
- Spread makes a shallow copy, not a deep copy.

## References

- [MDN indexed collections](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections)
- [MDN Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [MDN Array methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array#instance_methods)
- [MDN sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)
- [MDN toSorted](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toSorted)

---

| [Previous: Functions, scope, and closures](04-functions-scope-and-closures.md) | [Notes index](../README.md) | [Next: Objects, destructuring, and immutability](06-objects-destructuring-and-immutability.md) |
|:--|:--:|--:|
