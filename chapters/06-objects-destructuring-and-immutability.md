# 6. Objects, destructuring, and immutability

[Back to notes index](../README.md)

| [Previous: Arrays and iteration](05-arrays-and-iteration.md) | [Notes index](../README.md) | [Next: Strings, numbers, dates, and JSON](07-strings-numbers-dates-and-json.md) |
|:--|:--:|--:|

## Group related values in an object

An object stores values under property keys. A property can hold a primitive, another object, an array, or a function.

~~~js
const note = {
  title: 'Objects',
  reviewed: false,
  wordCount: 420,
};

console.log(note.title);
console.log(note['wordCount']);
~~~

Use dot notation when the property name is known. Use bracket notation when the property name is stored in a variable or contains characters that cannot be written after a dot.

~~~js
const fieldName = 'reviewed';
note[fieldName] = true;

const labels = {
  'last-updated': 'Today',
};

console.log(labels['last-updated']);
~~~

Object keys are strings or symbols. A number used as a key is converted to a string.

## Create and update properties

Object literals can use shorthand when a property name matches an existing variable. A computed property name uses an expression between square brackets.

~~~js
const title = 'Property shorthand';
const field = 'category';

const entry = {
  title,
  [field]: 'JavaScript',
};

console.log(entry);
~~~

Assigning to a property adds it when it does not exist. delete removes a property, though replacing a value or using an explicit state is often easier to understand.

~~~js
const settings = { theme: 'dark' };
settings.fontSize = 16;
delete settings.theme;

console.log(settings);
~~~

Use Object.hasOwn to check whether an object directly has a property. The in operator also checks the object's prototype chain.

~~~js
const profile = { name: 'Ashish' };

console.log(Object.hasOwn(profile, 'name')); // true
console.log(Object.hasOwn(profile, 'city')); // false
console.log('toString' in profile); // true through the prototype
~~~

For data objects, check own properties when that is what the code requires.

## Read and enumerate properties

Object.keys returns an array of an object's own enumerable string keys. Object.values returns the values, and Object.entries returns key-value pairs.

~~~js
const learner = {
  name: 'Ashish',
  city: 'Bengaluru',
};

console.log(Object.keys(learner));
console.log(Object.values(learner));
console.log(Object.entries(learner));
~~~

This is useful when a task needs to inspect data rather than call a method.

~~~js
for (const [key, value] of Object.entries(learner)) {
  console.log(key, value);
}
~~~

Property enumeration does not include non-enumerable or symbol properties. Use the specific reflection API required when those properties matter.

## Destructure object properties

Destructuring assigns object properties to local variables. A local variable can be renamed and can have a default value.

~~~js
const note = {
  title: 'Destructuring',
  reviewed: true,
};

const { title, reviewed } = note;
const { title: noteTitle, category = 'General' } = note;

console.log(title, reviewed);
console.log(noteTitle, category);
~~~

The default is used when a property is missing or undefined, but not when it is null.

~~~js
const options = { limit: null };
const { limit = 10 } = options;

console.log(limit); // null
~~~

Destructuring a missing property creates an undefined local variable. Use a default when the later code needs a specific fallback.

~~~js
const record = {};
const { label = 'Untitled' } = record;

console.log(label);
~~~

Nested destructuring can be concise, but too many levels make code difficult to read. Extract an intermediate value when it helps explain the structure.

~~~js
const user = {
  address: { city: 'Bengaluru' },
};

const {
  address: { city },
} = user;

console.log(city);
~~~

## Destructure arrays

Array destructuring reads values by position. It can skip a position with an empty slot and gather remaining values with a rest element.

~~~js
const coordinates = [12, 30, 48];
const [first, second] = coordinates;
const [start, , end] = coordinates;
const [head, ...tail] = coordinates;

console.log(first, second);
console.log(start, end);
console.log(head, tail);
~~~

Use array destructuring when position has meaning, such as a pair returned by an operation. For named fields, an object is usually clearer.

## Copy and merge with spread

Spread syntax copies enumerable values into a new object. Properties later in the expression override earlier values with the same key.

~~~js
const defaults = {
  theme: 'dark',
  pageSize: 20,
};

const userSettings = {
  pageSize: 10,
};

const settings = {
  ...defaults,
  ...userSettings,
};

console.log(settings); // { theme: 'dark', pageSize: 10 }
~~~

Spread is shallow. A nested object remains shared unless that nested value is also copied.

~~~js
const original = {
  title: 'Objects',
  metadata: { reviewed: false },
};

const copy = { ...original };
copy.metadata.reviewed = true;

console.log(original.metadata.reviewed); // true
~~~

To update a nested property without mutating the source, copy each changed level.

~~~js
const updated = {
  ...original,
  metadata: {
    ...original.metadata,
    reviewed: false,
  },
};

console.log(original.metadata.reviewed); // true
console.log(updated.metadata.reviewed); // false
~~~

structuredClone can make a deep copy of many structured values supported by the platform. It cannot clone every value, including functions, and should not be used as a substitute for understanding which data is shared.

## Gather remaining properties with rest

Object rest collects properties that were not selected by destructuring.

~~~js
const account = {
  id: 7,
  name: 'Ashish',
  email: 'ash.ranjan09@gmail.com',
};

const { id, ...publicProfile } = account;

console.log(id);
console.log(publicProfile);
~~~

This creates a shallow object without id. It does not remove id from account.

## Prevent accidental changes

Object.freeze prevents adding, removing, or changing the direct properties of an object. It is shallow, so nested objects can still change.

~~~js
const configuration = Object.freeze({
  mode: 'production',
  features: ['search'],
});

try {
  configuration.mode = 'development';
} catch (error) {
  console.log(error instanceof TypeError); // true in strict mode
}

configuration.features.push('export');

console.log(configuration.mode); // production
console.log(configuration.features); // ['search', 'export']
~~~

In strict mode, changing a frozen property throws a TypeError. Modules and class bodies are strict by default. Freezing only helps when the code also avoids changing nested values or uses a deliberate deep-freeze strategy.

Prefer patterns where a function creates and returns an updated object rather than changing an object unexpectedly.

~~~js
function markReviewed(note) {
  return {
    ...note,
    reviewed: true,
  };
}

const before = { title: 'Objects', reviewed: false };
const after = markReviewed(before);

console.log(before.reviewed); // false
console.log(after.reviewed); // true
~~~

## Hands-on: update a nested note safely

Start with a note object containing a title, reviewed flag, and metadata object. Create a new value with the title changed and the reviewed flag set without changing the original object.

~~~js
const sourceNote = {
  id: 1,
  title: 'Object references',
  reviewed: false,
  metadata: {
    category: 'language',
  },
};

const revisedNote = {
  ...sourceNote,
  title: 'Objects and references',
  reviewed: true,
  metadata: {
    ...sourceNote.metadata,
    category: 'core JavaScript',
  },
};

console.log(sourceNote);
console.log(revisedNote);
~~~

Check that the original title and nested category remain unchanged. Then try changing the copy using a shallow spread only and observe which nested values are still shared.

## Notes to remember

- Objects group related values under property keys.
- Use dot notation for known keys and bracket notation for computed keys.
- Destructuring reads properties or array positions into local variables.
- Later spread properties override earlier ones.
- Spread and Object.freeze are shallow.
- Copy every changed object level when an immutable nested update is needed.
- Use Object.hasOwn to test for a direct property.

## References

- [MDN working with objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects)
- [MDN destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
- [MDN spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)
- [MDN Object.hasOwn](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwn)
- [MDN Object.freeze](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)
- [MDN structuredClone](https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone)

---

| [Previous: Arrays and iteration](05-arrays-and-iteration.md) | [Notes index](../README.md) | [Next: Strings, numbers, dates, and JSON](07-strings-numbers-dates-and-json.md) |
|:--|:--:|--:|
