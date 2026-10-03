# 15. Collections, iterators, and generators

[Back to notes index](../README.md)

| [Previous: Prototypes, this, and classes](14-prototypes-this-and-classes.md) | [Notes index](../README.md) | [Next: Browser security, accessibility, and performance](16-browser-security-accessibility-and-performance.md) |
|:--|:--:|--:|

## Choose a collection for the data

JavaScript has several built-in collection types. Arrays are ordered lists. Objects are commonly used for records with named fields. Map and Set provide collection behavior for other use cases.

~~~js
const tags = new Set();

tags.add('browser');
tags.add('arrays');
tags.add('browser');

console.log(tags.size); // 2
console.log(tags.has('arrays')); // true
~~~

A Set stores unique values and preserves insertion order. It is useful for removing duplicates or checking membership.

~~~js
const names = ['Ashish', 'Maya', 'Ashish'];
const uniqueNames = [...new Set(names)];

console.log(uniqueNames); // ['Ashish', 'Maya']
~~~

## Use Map for key-value collections

A Map stores key-value pairs and allows keys of any type. It preserves insertion order and has a size property.

~~~js
const noteById = new Map();

noteById.set(1, { title: 'Collections' });
noteById.set(2, { title: 'Iterators' });

console.log(noteById.get(1));
console.log(noteById.has(2));
console.log(noteById.size);
~~~

Map is useful when keys are not just fixed string property names, or when insertion order and direct size are important. An object remains a good choice for a record with a known set of named fields.

~~~js
for (const [id, note] of noteById) {
  console.log(id, note.title);
}
~~~

Map entries are iterable and can be converted to an array. JSON.stringify does not automatically serialize a Map as its entries. Convert it deliberately when the data needs a JSON representation.

~~~js
const plainEntries = Array.from(noteById.entries());
const jsonText = JSON.stringify(plainEntries);

console.log(jsonText);
~~~

## Understand weak collections

WeakMap and WeakSet hold object keys or values weakly. They are not enumerable and do not expose a size because garbage collection can remove entries at any time.

A WeakMap can associate private metadata with an object without keeping that object alive solely because of the metadata.

~~~js
const privateMetadata = new WeakMap();

function attachMetadata(object, metadata) {
  privateMetadata.set(object, metadata);
}

const element = {};
attachMetadata(element, { inspected: true });

console.log(privateMetadata.get(element));
~~~

Use weak collections only when their lifetime behavior is useful. Use Map or Set when the data must be iterated, measured, or explicitly cleared.

## What an iterable provides

An iterable exposes an iterator through the well-known Symbol.iterator property. for...of and spread syntax work with iterable values such as arrays, strings, maps, and sets.

~~~js
const word = 'JS';

for (const character of word) {
  console.log(character);
}

console.log([...word]);
~~~

An iterator has a next method that returns an object with value and done fields. Most code should use for...of or built-in collection methods instead of calling next manually.

~~~js
const iterator = ['first', 'second'][Symbol.iterator]();

console.log(iterator.next()); // { value: 'first', done: false }
console.log(iterator.next()); // { value: 'second', done: false }
console.log(iterator.next()); // { value: undefined, done: true }
~~~

## Make a custom iterable with a generator

A generator function uses function* and yield. Calling it returns an iterator. Each next call runs until the next yield or the function finishes.

~~~js
function* createSequence() {
  yield 'first';
  yield 'second';
  yield 'third';
}

for (const value of createSequence()) {
  console.log(value);
}
~~~

Generators produce values on demand. This can be useful when generating a sequence or processing values incrementally instead of creating a complete array first.

~~~js
function* countTo(maximum) {
  for (let value = 1; value <= maximum; value += 1) {
    yield value;
  }
}

const values = [...countTo(4)];
console.log(values); // [1, 2, 3, 4]
~~~

A generator can delegate to another iterable with yield*.

~~~js
function* combineTopics() {
  yield 'values';
  yield* ['functions', 'objects'];
  yield 'browser APIs';
}

console.log([...combineTopics()]);
~~~

## Use generators for paged values

A generator can model a sequence of batches and pause between them. This example uses already available arrays. Real network pagination is asynchronous and is covered with Fetch in Chapter 12.

~~~js
function* readBatches(batches) {
  for (const batch of batches) {
    yield batch;
  }
}

const batches = [
  ['note one', 'note two'],
  ['note three'],
];

for (const batch of readBatches(batches)) {
  console.log(batch);
}
~~~

For data that arrives asynchronously over time, JavaScript also has async iterators and for await...of. Use them when an API exposes an asynchronous iterable, not merely because a loop has several steps.

## Use typed arrays for binary data

Typed arrays provide views over binary data using numeric element types. They are useful for file formats, image data, audio, network protocols, and other binary work.

~~~js
const bytes = new Uint8Array([65, 66, 67]);

console.log(bytes[0]); // 65
console.log(new TextDecoder().decode(bytes)); // ABC
~~~

Typed arrays are not ordinary arrays. They have fixed element types and convert assigned values to their element representation. Use them when an API expects binary data, not as a default replacement for regular application lists.

## Hands-on: count note tags

Use a Map to count tags and a Set to list unique tags. Then iterate over the results.

~~~js
const notes = [
  { title: 'DOM', tags: ['browser', 'events'] },
  { title: 'Events', tags: ['browser', 'accessibility'] },
  { title: 'Collections', tags: ['data', 'arrays'] },
];

const tagCounts = new Map();

for (const note of notes) {
  for (const tag of note.tags) {
    tagCounts.set(tag, (tagCounts.get(tag) ?? 0) + 1);
  }
}

const uniqueTags = new Set(tagCounts.keys());

for (const tag of uniqueTags) {
  console.log(tag, tagCounts.get(tag));
}
~~~

Add another note and check how the counts and unique tags change. Then replace the Map with a plain object and compare which API makes the purpose clearer.

## Notes to remember

- Array is an ordered list; Map is an ordered key-value collection.
- Set stores unique values.
- WeakMap and WeakSet are not enumerable and do not keep keys alive.
- An iterable provides values through Symbol.iterator.
- for...of consumes iterables.
- Generators create iterators that yield values on demand.
- Typed arrays are specialized for binary data.

## References

- [MDN Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- [MDN Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
- [MDN WeakMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)
- [MDN iteration protocols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols)
- [MDN generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)
- [MDN typed arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays)

---

| [Previous: Prototypes, this, and classes](14-prototypes-this-and-classes.md) | [Notes index](../README.md) | [Next: Browser security, accessibility, and performance](16-browser-security-accessibility-and-performance.md) |
|:--|:--:|--:|
