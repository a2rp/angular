# 98. All code samples

[Back to notes index](../README.md)

| [Previous: Browser security, accessibility, and performance](16-browser-security-accessibility-and-performance.md) | [Notes index](../README.md) | [Next: Complete Q&A](99-complete-q-and-a.md) |
|:--|:--:|--:|

This chapter gathers the runnable examples from the topic chapters. Each sample keeps the language label and the section where it was introduced.

## 1. JavaScript foundations and runtime

### Source code becomes running behavior, example 1

~~~js
const greeting = 'Hello, JavaScript';
console.log(greeting);
~~~

### Run a browser script, example 2

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

### Run a browser script, example 3

~~~js
const message = document.querySelector('#message');

if (message) {
  message.textContent = 'JavaScript updated this page.';
}
~~~

### Use browser developer tools, example 4

~~~js
const price = 12.5;
const quantity = 3;
const total = price * quantity;

console.log({ price, quantity, total });
~~~

### Run JavaScript with Node.js, example 5

~~~powershell
node --version
node
node practice.js
~~~

### Run JavaScript with Node.js, example 6

~~~js
console.log(globalThis === globalThis);
~~~

### Make the first interactive example, example 7

~~~html
<button id="greet-button" type="button">Say hello</button>
<p id="greeting" aria-live="polite">Waiting for a click.</p>
~~~

### Make the first interactive example, example 8

~~~js
const button = document.querySelector('#greet-button');
const greeting = document.querySelector('#greeting');

if (button instanceof HTMLButtonElement && greeting) {
  button.addEventListener('click', () => {
    greeting.textContent = 'Hello from JavaScript.';
  });
}
~~~

### Comments and readable source, example 9

~~~js
// Keep the user's original input for the form.
const enteredName = 'Ashish';

/*
  Normalize the value before comparing it with
  another name.
*/
const normalizedName = enteredName.trim().toLowerCase();
~~~

## 2. Values, variables, and types

### JavaScript is dynamically typed, example 1

~~~js
let result = 12;
result = 'complete';

console.log(result);
~~~

### Choose const or let, example 2

~~~js
const courseName = 'JavaScript';
let completedChapters = 2;

completedChapters += 1;

console.log(courseName, completedChapters);
~~~

### Choose const or let, example 3

~~~js
const learner = { name: 'Ashish' };
learner.name = 'Ashish Ranjan';

const topics = ['values'];
topics.push('variables');

console.log(learner.name, topics);
~~~

### Understand scope and the temporal dead zone, example 4

~~~js
if (true) {
  const message = 'Inside this block';
  console.log(message);
}

// console.log(message); // ReferenceError
~~~

### Understand scope and the temporal dead zone, example 5

~~~js
{
  // console.log(score); // ReferenceError
  const score = 10;
  console.log(score);
}
~~~

### The primitive types, example 6

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

### Inspect values with typeof, example 7

~~~js
console.log(typeof null); // object
console.log(typeof []); // object
console.log(typeof (() => {})); // function

console.log(Array.isArray([])); // true
~~~

### Work with numbers safely, example 8

~~~js
console.log(0.1 + 0.2); // 0.30000000000000004

const cents = Math.round((0.1 + 0.2) * 100);
console.log(cents); // 30
~~~

### Work with numbers safely, example 9

~~~js
const parsed = Number('not a number');

console.log(Number.isNaN(parsed)); // true
console.log(Number.isFinite(parsed)); // false
~~~

### Convert values explicitly, example 10

~~~js
console.log(Number('42')); // 42
console.log(String(42)); // "42"
console.log(Boolean('')); // false
console.log(Boolean('false')); // true
~~~

### Convert values explicitly, example 11

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

### Convert values explicitly, example 12

~~~js
console.log(parseInt('101', 2)); // 5
console.log(parseInt('24px', 10)); // 24
~~~

### Compare values, example 13

~~~js
console.log(5 === 5); // true
console.log(5 === '5'); // false
console.log(5 == '5'); // true, because loose equality converts
~~~

### Compare values, example 14

~~~js
console.log(Object.is(NaN, NaN)); // true
console.log(Object.is(0, -0)); // false
~~~

### Compare values, example 15

~~~js
const first = { id: 1 };
const second = { id: 1 };
const sameReference = first;

console.log(first === second); // false
console.log(first === sameReference); // true
~~~

### Use truthiness deliberately, example 16

~~~js
if ([]) {
  console.log('An empty array is truthy.');
}

if ({}) {
  console.log('An empty object is truthy.');
}
~~~

### Use truthiness deliberately, example 17

~~~js
const oldPageSize = 0 || 20;
const selectedPageSize = 0 ?? 20;

console.log(oldPageSize); // 20
console.log(selectedPageSize); // 0
~~~

### Hands-on: validate a small input, example 18

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

## 3. Operators and control flow

### Operators produce or update values, example 1

~~~js
const subtotal = 4 * 25;
const remainder = 17 % 5;
const squared = 3 ** 2;

console.log(subtotal); // 100
console.log(remainder); // 2
console.log(squared); // 9
~~~

### Operators produce or update values, example 2

~~~js
console.log(2 + 3); // 5
console.log('2' + 3); // "23"
console.log(Number('2') + 3); // 5
~~~

### Operators produce or update values, example 3

~~~js
let total = 10;
total += 5;
total *= 2;

console.log(total); // 30
~~~

### Use precedence carefully, example 4

~~~js
const average = (8 + 10 + 12) / 3;
const isInRange = age >= 18 && age <= 65;
~~~

### Compare without accidental conversion, example 5

~~~js
const score = 82;

console.log(score >= 60); // true
console.log(score === 82); // true
console.log(score !== '82'); // true
~~~

### Combine conditions with logical operators, example 6

~~~js
const hasName = name !== '';
const canSubmit = hasName && isFormValid;
const displayName = name || 'Guest';
~~~

### Combine conditions with logical operators, example 7

~~~js
const pageSize = savedPageSize ?? 20;
const city = account.profile?.address?.city ?? 'Unknown';
~~~

### Choose a branch, example 8

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

### Choose a branch, example 9

~~~js
const statusLabel = isComplete ? 'Complete' : 'In progress';
~~~

### Use switch for one value with several cases, example 10

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

### Repeat work with loops, example 11

~~~js
const chapters = ['values', 'functions', 'objects'];

for (let index = 0; index < chapters.length; index += 1) {
  console.log(index + 1, chapters[index]);
}
~~~

### Repeat work with loops, example 12

~~~js
let attempts = 0;

while (attempts < 3) {
  console.log('Attempt', attempts + 1);
  attempts += 1;
}
~~~

### Choose the right collection loop, example 13

~~~js
const topics = ['arrays', 'objects', 'functions'];

for (const topic of topics) {
  console.log(topic);
}
~~~

### Choose the right collection loop, example 14

~~~js
const learner = { name: 'Ashish', city: 'Bengaluru' };

for (const key in learner) {
  if (Object.hasOwn(learner, key)) {
    console.log(key, learner[key]);
  }
}
~~~

### Use array methods for transformations, example 15

~~~js
const prices = [10, 25, 40];
const taxedPrices = prices.map((price) => price * 1.1);
const expensivePrices = prices.filter((price) => price >= 25);

console.log(taxedPrices);
console.log(expensivePrices);
~~~

### Hands-on: calculate a cart total, example 16

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

## 4. Functions, scope, and closures

### A function packages behavior, example 1

~~~js
function calculateArea(width, height) {
  return width * height;
}

const area = calculateArea(4, 6);
console.log(area); // 24
~~~

### A function packages behavior, example 2

~~~js
function logTopic(topic) {
  console.log('Topic:', topic);
}

const result = logTopic('functions');
console.log(result); // undefined
~~~

### Function declarations and expressions, example 3

~~~js
console.log(double(4)); // 8

function double(value) {
  return value * 2;
}
~~~

### Function declarations and expressions, example 4

~~~js
const triple = function (value) {
  return value * 3;
};

console.log(triple(4)); // 12
~~~

### Arrow functions, example 5

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

### Pass parameters and return values, example 6

~~~js
function greet(name = 'Guest') {
  return 'Hello, ' + name;
}

console.log(greet()); // Hello, Guest
console.log(greet('Ashish')); // Hello, Ashish
~~~

### Pass parameters and return values, example 7

~~~js
console.log(greet(null)); // Hello, null
console.log(greet('')); // Hello,
~~~

### Pass parameters and return values, example 8

~~~js
function sum(...values) {
  return values.reduce((total, value) => total + value, 0);
}

console.log(sum(2, 3, 5)); // 10
~~~

### Functions are values, example 9

~~~js
function applyOperation(value, operation) {
  return operation(value);
}

const result = applyOperation(5, (number) => number * number);
console.log(result); // 25
~~~

### Functions are values, example 10

~~~js
const topics = ['values', 'functions', 'objects'];
const labels = topics.map((topic) => topic.toUpperCase());

console.log(labels);
~~~

### Lexical scope, example 11

~~~js
const course = 'JavaScript';

function showCourse() {
  const message = 'Studying ' + course;
  return message;
}

console.log(showCourse());
~~~

### Lexical scope, example 12

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

### Closures keep access to outer variables, example 13

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

### Closures keep access to outer variables, example 14

~~~js
const firstCounter = createCounter();
const secondCounter = createCounter();

console.log(firstCounter()); // 1
console.log(firstCounter()); // 2
console.log(secondCounter()); // 1
~~~

### Scope and loops, example 15

~~~js
const callbacks = [];

for (let index = 0; index < 3; index += 1) {
  callbacks.push(() => index);
}

console.log(callbacks.map((getIndex) => getIndex())); // [0, 1, 2]
~~~

### Keep return paths clear, example 16

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

### Hands-on: create a configurable formatter, example 17

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

## 5. Arrays and iteration

### Store ordered values in an array, example 1

~~~js
const topics = ['values', 'functions', 'objects'];

console.log(topics[0]); // values
console.log(topics.length); // 3
console.log(topics[topics.length - 1]); // objects
~~~

### Store ordered values in an array, example 2

~~~js
const mixed = ['chapter', 5, true];
console.log(mixed[10]); // undefined
~~~

### Add and remove items, example 3

~~~js
const queue = ['review', 'practice'];

queue.push('summarize');
const firstTask = queue.shift();

console.log(firstTask); // review
console.log(queue); // ['practice', 'summarize']
~~~

### Add and remove items, example 4

~~~js
const original = ['a', 'b', 'c', 'd'];
const middle = original.slice(1, 3);
const removed = original.splice(1, 2);

console.log(middle); // ['b', 'c']
console.log(removed); // ['b', 'c']
console.log(original); // ['a', 'd']
~~~

### Iterate over values, example 5

~~~js
const scores = [72, 88, 91];

for (const score of scores) {
  console.log(score);
}
~~~

### Iterate over values, example 6

~~~js
scores.forEach((score, index) => {
  console.log(index, score);
});
~~~

### Transform with map, example 7

~~~js
const prices = [10, 20, 30];
const pricesWithTax = prices.map((price) => price * 1.05);

console.log(pricesWithTax); // [10.5, 21, 31.5]
console.log(prices); // [10, 20, 30]
~~~

### Transform with map, example 8

~~~js
const notes = [
  { title: 'Arrays', reviewed: true },
  { title: 'Objects', reviewed: false },
];

const titles = notes.map((note) => note.title);
console.log(titles); // ['Arrays', 'Objects']
~~~

### Select with filter, example 9

~~~js
const reviewedNotes = notes.filter((note) => note.reviewed);
const notesToReview = notes.filter((note) => !note.reviewed);

console.log(reviewedNotes);
console.log(notesToReview);
~~~

### Find an item, example 10

~~~js
const selected = notes.find((note) => note.title === 'Objects');
const selectedIndex = notes.findIndex((note) => note.title === 'Objects');

console.log(selected);
console.log(selectedIndex);
~~~

### Find an item, example 11

~~~js
if (selected) {
  console.log(selected.title);
}
~~~

### Find an item, example 12

~~~js
const hasPendingNotes = notes.some((note) => !note.reviewed);
const allNotesReviewed = notes.every((note) => note.reviewed);

console.log(hasPendingNotes, allNotesReviewed);
~~~

### Combine values with reduce, example 13

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

### Sort without changing the source list, example 14

~~~js
const values = [100, 4, 25];
const ascending = [...values].sort((left, right) => left - right);

console.log(ascending); // [4, 25, 100]
console.log(values); // [100, 4, 25]
~~~

### Sort without changing the source list, example 15

~~~js
const sortedTitles = notes
  .map((note) => note.title)
  .toSorted((left, right) => left.localeCompare(right));
~~~

### Copy and combine arrays, example 16

~~~js
const firstGroup = ['values', 'functions'];
const allTopics = [...firstGroup, 'arrays'];
const copy = [...firstGroup];

console.log(allTopics);
console.log(copy);
~~~

### Avoid mutating during iteration, example 17

~~~js
const numbers = [1, 2, 3, 4, 5];
const evenNumbers = numbers.filter((number) => number % 2 === 0);

console.log(evenNumbers); // [2, 4]
~~~

### Hands-on: summarize note data, example 18

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

## 6. Objects, destructuring, and immutability

### Group related values in an object, example 1

~~~js
const note = {
  title: 'Objects',
  reviewed: false,
  wordCount: 420,
};

console.log(note.title);
console.log(note['wordCount']);
~~~

### Group related values in an object, example 2

~~~js
const fieldName = 'reviewed';
note[fieldName] = true;

const labels = {
  'last-updated': 'Today',
};

console.log(labels['last-updated']);
~~~

### Create and update properties, example 3

~~~js
const title = 'Property shorthand';
const field = 'category';

const entry = {
  title,
  [field]: 'JavaScript',
};

console.log(entry);
~~~

### Create and update properties, example 4

~~~js
const settings = { theme: 'dark' };
settings.fontSize = 16;
delete settings.theme;

console.log(settings);
~~~

### Create and update properties, example 5

~~~js
const profile = { name: 'Ashish' };

console.log(Object.hasOwn(profile, 'name')); // true
console.log(Object.hasOwn(profile, 'city')); // false
console.log('toString' in profile); // true through the prototype
~~~

### Read and enumerate properties, example 6

~~~js
const learner = {
  name: 'Ashish',
  city: 'Bengaluru',
};

console.log(Object.keys(learner));
console.log(Object.values(learner));
console.log(Object.entries(learner));
~~~

### Read and enumerate properties, example 7

~~~js
for (const [key, value] of Object.entries(learner)) {
  console.log(key, value);
}
~~~

### Destructure object properties, example 8

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

### Destructure object properties, example 9

~~~js
const options = { limit: null };
const { limit = 10 } = options;

console.log(limit); // null
~~~

### Destructure object properties, example 10

~~~js
const record = {};
const { label = 'Untitled' } = record;

console.log(label);
~~~

### Destructure object properties, example 11

~~~js
const user = {
  address: { city: 'Bengaluru' },
};

const {
  address: { city },
} = user;

console.log(city);
~~~

### Destructure arrays, example 12

~~~js
const coordinates = [12, 30, 48];
const [first, second] = coordinates;
const [start, , end] = coordinates;
const [head, ...tail] = coordinates;

console.log(first, second);
console.log(start, end);
console.log(head, tail);
~~~

### Copy and merge with spread, example 13

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

### Copy and merge with spread, example 14

~~~js
const original = {
  title: 'Objects',
  metadata: { reviewed: false },
};

const copy = { ...original };
copy.metadata.reviewed = true;

console.log(original.metadata.reviewed); // true
~~~

### Copy and merge with spread, example 15

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

### Gather remaining properties with rest, example 16

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

### Prevent accidental changes, example 17

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

### Prevent accidental changes, example 18

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

### Hands-on: update a nested note safely, example 19

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

## 7. Strings, numbers, dates, and JSON

### Work with text, example 1

~~~js
const heading = '  JavaScript notes  ';
const cleaned = heading.trim();

console.log(cleaned);
console.log(heading); // The original still has its spaces
~~~

### Work with text, example 2

~~~js
const sentence = 'Functions make code reusable.';
const words = sentence.split(' ');

console.log(sentence.includes('code')); // true
console.log(sentence.slice(0, 9)); // Functions
console.log(words); // ['Functions', 'make', 'code', 'reusable.']
console.log(sentence.replace('reusable', 'clear')); // Functions make code clear.
~~~

### Work with text, example 3

~~~js
const text = 'JavaScript';
console.log(text.length); // 10
console.log([...text]); // Iterates Unicode code points
~~~

### Build text with template literals, example 4

~~~js
const learnerName = 'Ashish';
const topicCount = 16;
const summary = `${learnerName} has ${topicCount} JavaScript topics to study.`;

console.log(summary);
~~~

### Build text with template literals, example 5

~~~js
const price = 1250;
const formattedPrice = price.toFixed(2);
const label = `Price: ₹${formattedPrice}`;

console.log(label);
~~~

### Understand number behavior, example 6

~~~js
console.log(0.1 + 0.2); // 0.30000000000000004

const closeEnough = Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON;
console.log(closeEnough); // true
~~~

### Understand number behavior, example 7

~~~js
console.log(Number.isInteger(12)); // true
console.log(Number.isFinite(Infinity)); // false
console.log(Number.isNaN(Number('unknown'))); // true
~~~

### Format numbers for people, example 8

~~~js
const currency = new Intl.NumberFormat('en-IN', {
  style: 'currency',
  currency: 'INR',
});

const percent = new Intl.NumberFormat('en-IN', {
  style: 'percent',
  maximumFractionDigits: 1,
});

console.log(currency.format(1250));
console.log(percent.format(0.725));
~~~

### Read and format dates, example 9

~~~js
const now = new Date();
const timestamp = Date.now();

console.log(now.toISOString());
console.log(timestamp);
~~~

### Read and format dates, example 10

~~~js
const savedAt = new Date('2026-10-03T10:30:00Z');

if (!Number.isNaN(savedAt.getTime())) {
  console.log(savedAt.toISOString());
}
~~~

### Read and format dates, example 11

~~~js
const dateFormatter = new Intl.DateTimeFormat('en-IN', {
  dateStyle: 'medium',
  timeStyle: 'short',
  timeZone: 'Asia/Kolkata',
});

console.log(dateFormatter.format(savedAt));
~~~

### Match text with regular expressions, example 12

~~~js
const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

console.log(emailPattern.test('ashish@example.com')); // true
console.log(emailPattern.test('not-an-email')); // false
~~~

### Match text with regular expressions, example 13

~~~js
const globalPattern = /js/g;

console.log(globalPattern.test('js notes')); // true
console.log(globalPattern.test('js notes')); // false because lastIndex changed
~~~

### Convert values to JSON, example 14

~~~js
const note = {
  title: 'JSON',
  reviewed: false,
  tags: ['data', 'browser'],
};

const jsonText = JSON.stringify(note, null, 2);
const restored = JSON.parse(jsonText);

console.log(jsonText);
console.log(restored.title);
~~~

### Convert values to JSON, example 15

~~~js
const record = {
  title: 'Optional value',
  extra: undefined,
  callback: () => true,
};

console.log(JSON.stringify(record));
// {"title":"Optional value"}
~~~

### Convert values to JSON, example 16

~~~js
function parseJson(text) {
  try {
    return JSON.parse(text);
  } catch (error) {
    console.error('The saved text is not valid JSON.', error);
    return null;
  }
}
~~~

### Hands-on: format a study note, example 17

~~~js
const note = {
  title: 'Strings and dates',
  wordCount: 760,
  updatedAt: new Date().toISOString(),
};

const title = `Study note: ${note.title}`;
const words = new Intl.NumberFormat('en-IN').format(note.wordCount);
const updated = new Intl.DateTimeFormat('en-IN', {
  dateStyle: 'medium',
}).format(new Date(note.updatedAt));

console.log(title, words, updated);

const saved = JSON.stringify(note);
const loaded = JSON.parse(saved);
console.log(loaded.title);
~~~

## 8. The DOM and browser events

### Find elements safely, example 1

~~~html
<h1 id="page-title">Study notes</h1>
<p class="status">Ready</p>
~~~

### Find elements safely, example 2

~~~js
const title = document.querySelector('#page-title');
const statusMessages = document.querySelectorAll('.status');

if (title instanceof HTMLHeadingElement) {
  console.log(title.textContent);
}

console.log(statusMessages.length);
~~~

### Find elements safely, example 3

~~~js
const statusText = Array.from(
  document.querySelectorAll('.status'),
  (element) => element.textContent,
);

console.log(statusText);
~~~

### Change text and classes, example 4

~~~js
const message = document.querySelector('#message');

if (message) {
  message.textContent = 'Your notes are saved.';
  message.classList.add('success');
  message.setAttribute('aria-live', 'polite');
}
~~~

### Change text and classes, example 5

~~~js
const panel = document.querySelector('#help-panel');

if (panel instanceof HTMLElement) {
  panel.hidden = true;
  panel.classList.toggle('is-collapsed', panel.hidden);
}
~~~

### Create and remove elements, example 6

~~~html
<ul id="note-list"></ul>
~~~

### Create and remove elements, example 7

~~~js
const list = document.querySelector('#note-list');

if (list instanceof HTMLUListElement) {
  const item = document.createElement('li');
  item.textContent = 'DOM basics';
  item.classList.add('note-item');
  list.append(item);
}
~~~

### Create and remove elements, example 8

~~~js
const oldMessage = document.querySelector('.temporary-message');
oldMessage?.remove();
~~~

### Listen for events, example 9

~~~html
<button id="save-button" type="button">Save</button>
<p id="save-status" aria-live="polite"></p>
~~~

### Listen for events, example 10

~~~js
const saveButton = document.querySelector('#save-button');
const saveStatus = document.querySelector('#save-status');

function showSavedMessage() {
  if (saveStatus) {
    saveStatus.textContent = 'Saved.';
  }
}

saveButton?.addEventListener('click', showSavedMessage);
~~~

### Listen for events, example 11

~~~js
document.querySelector('#save-button')?.addEventListener('click', (event) => {
  console.log('Clicked element:', event.target);
  console.log('Listener element:', event.currentTarget);
});
~~~

### Understand bubbling and event delegation, example 12

~~~html
<ul id="note-list">
  <li>
    <span>Functions</span>
    <button type="button" data-action="remove">Remove</button>
  </li>
</ul>
~~~

### Understand bubbling and event delegation, example 13

~~~js
const list = document.querySelector('#note-list');

list?.addEventListener('click', (event) => {
  if (!(event.target instanceof Element)) {
    return;
  }

  const removeButton = event.target.closest('[data-action="remove"]');

  if (!removeButton || !list.contains(removeButton)) {
    return;
  }

  removeButton.closest('li')?.remove();
});
~~~

### Prevent a browser default action, example 14

~~~js
const form = document.querySelector('#search-form');

form?.addEventListener('submit', (event) => {
  event.preventDefault();
  console.log('Handle the search in JavaScript.');
});
~~~

### Remove listeners and clean up, example 15

~~~js
const button = document.querySelector('#toggle-button');

function togglePanel() {
  console.log('Toggle the panel');
}

button?.addEventListener('click', togglePanel);
button?.removeEventListener('click', togglePanel);
~~~

### Remove listeners and clean up, example 16

~~~js
const controller = new AbortController();

window.addEventListener(
  'resize',
  () => console.log(window.innerWidth),
  { signal: controller.signal },
);

controller.abort();
~~~

### Work with data attributes, example 17

~~~html
<button type="button" data-note-id="42" data-action="open-note">
  Open note
</button>
~~~

### Work with data attributes, example 18

~~~js
const openButton = document.querySelector('[data-action="open-note"]');

if (openButton instanceof HTMLButtonElement) {
  console.log(openButton.dataset.noteId); // String value: "42"
}
~~~

### Hands-on: add and remove list items, example 19

~~~html
<label for="topic-input">New topic</label>
<input id="topic-input" />
<button id="add-topic" type="button">Add topic</button>
<ul id="topic-list"></ul>
~~~

### Hands-on: add and remove list items, example 20

~~~js
const input = document.querySelector('#topic-input');
const addButton = document.querySelector('#add-topic');
const topicList = document.querySelector('#topic-list');

addButton?.addEventListener('click', () => {
  if (!(input instanceof HTMLInputElement)) {
    return;
  }

  const title = input.value.trim();

  if (title.length === 0 || !(topicList instanceof HTMLUListElement)) {
    return;
  }

  const item = document.createElement('li');
  const label = document.createElement('span');
  const removeButton = document.createElement('button');

  label.textContent = title;
  removeButton.type = 'button';
  removeButton.textContent = 'Remove';
  removeButton.dataset.action = 'remove';

  item.append(label, removeButton);
  topicList.append(item);
  input.value = '';
});

topicList?.addEventListener('click', (event) => {
  if (!(event.target instanceof Element)) {
    return;
  }

  if (event.target.closest('[data-action="remove"]')) {
    event.target.closest('li')?.remove();
  }
});
~~~

## 9. Forms, validation, and browser storage

### Read form values with FormData, example 1

~~~html
<form id="note-form">
  <label for="note-title">Note title</label>
  <input id="note-title" name="title" required minlength="3" />
  <button type="submit">Save note</button>
</form>
<p id="form-status" aria-live="polite"></p>
~~~

### Read form values with FormData, example 2

~~~js
const form = document.querySelector('#note-form');
const status = document.querySelector('#form-status');

form?.addEventListener('submit', (event) => {
  event.preventDefault();

  if (!(form instanceof HTMLFormElement)) {
    return;
  }

  const formData = new FormData(form);
  const title = formData.get('title');

  if (typeof title !== 'string' || title.trim().length < 3) {
    if (status) {
      status.textContent = 'Enter a title with at least three characters.';
    }
    return;
  }

  if (status) {
    status.textContent = 'Saved: ' + title.trim();
  }

  form.reset();
});
~~~

### Use built-in browser validation, example 3

~~~html
<label for="email">Email address</label>
<input id="email" name="email" type="email" required />
<label for="word-count">Word count</label>
<input id="word-count" name="wordCount" type="number" min="1" max="5000" required />
~~~

### Use built-in browser validation, example 4

~~~js
const noteForm = document.querySelector('#note-form');

if (noteForm instanceof HTMLFormElement && !noteForm.reportValidity()) {
  console.log('The form has invalid fields.');
}
~~~

### Use built-in browser validation, example 5

~~~js
const password = document.querySelector('#password');

password?.addEventListener('input', () => {
  if (password instanceof HTMLInputElement) {
    password.setCustomValidity(
      password.value.length < 12 ? 'Use at least 12 characters.' : '',
    );
  }
});
~~~

### Show accessible validation feedback, example 6

~~~html
<label for="topic">Topic</label>
<input
  id="topic"
  name="topic"
  required
  aria-describedby="topic-help topic-error"
/>
<p id="topic-help">Enter the subject of this note.</p>
<p id="topic-error" role="alert" hidden>Enter a topic before saving.</p>
~~~

### Store small values in the browser, example 7

~~~js
localStorage.setItem('theme', 'dark');

const savedTheme = localStorage.getItem('theme') ?? 'light';
console.log(savedTheme);

sessionStorage.setItem('current-tab', 'notes');
~~~

### Parse stored JSON defensively, example 8

~~~js
function loadNotes() {
  try {
    const text = localStorage.getItem('study-notes');

    if (text === null) {
      return [];
    }

    const value = JSON.parse(text);

    if (!Array.isArray(value)) {
      return [];
    }

    return value.filter(
      (note) =>
        typeof note === 'object' &&
        note !== null &&
        typeof note.id === 'string' &&
        typeof note.title === 'string',
    );
  } catch (error) {
    console.error('Could not read saved notes.', error);
    return [];
  }
}
~~~

### Parse stored JSON defensively, example 9

~~~js
function saveNotes(notes) {
  try {
    localStorage.setItem('study-notes', JSON.stringify(notes));
    return true;
  } catch (error) {
    console.error('Could not save notes.', error);
    return false;
  }
}
~~~

### Hands-on: save a small note list, example 10

~~~html
<form id="study-note-form">
  <label for="study-note-title">Note title</label>
  <input id="study-note-title" name="title" required minlength="3" />
  <button type="submit">Add note</button>
</form>
<p id="notes-status" aria-live="polite"></p>
<ul id="study-note-list"></ul>
~~~

### Hands-on: save a small note list, example 11

~~~js
const storageKey = 'study-notes';
const notesForm = document.querySelector('#study-note-form');
const notesList = document.querySelector('#study-note-list');
const notesStatus = document.querySelector('#notes-status');
let notes = loadNotes();

function renderNotes() {
  if (!(notesList instanceof HTMLUListElement)) {
    return;
  }

  notesList.replaceChildren();

  for (const note of notes) {
    const item = document.createElement('li');
    item.textContent = note.title;
    notesList.append(item);
  }
}

notesForm?.addEventListener('submit', (event) => {
  event.preventDefault();

  if (!(notesForm instanceof HTMLFormElement)) {
    return;
  }

  const title = new FormData(notesForm).get('title');

  if (typeof title !== 'string' || title.trim().length < 3) {
    return;
  }

  notes = [
    ...notes,
    { id: crypto.randomUUID(), title: title.trim() },
  ];

  if (saveNotes(notes)) {
    renderNotes();
    notesForm.reset();

    if (notesStatus) {
      notesStatus.textContent = 'Note saved.';
    }
  }
});

renderNotes();
~~~

## 10. Errors, debugging, and testing

### Throw and catch errors, example 1

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

### Throw and catch errors, example 2

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

### Use finally for cleanup, example 3

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

### Add useful debugging output, example 4

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

### Write a small automated test, example 5

~~~js
// math.mjs
export function add(left, right) {
  return left + right;
}
~~~

### Write a small automated test, example 6

~~~js
// math.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import { add } from './math.mjs';

test('add returns the sum of two numbers', () => {
  assert.equal(add(2, 3), 5);
});
~~~

### Write a small automated test, example 7

~~~powershell
node --test
~~~

### Test normal cases and edge cases, example 8

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

### Test normal cases and edge cases, example 9

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

### Keep functions easy to test, example 10

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

## 11. Asynchronous JavaScript and promises

### Synchronous and asynchronous work, example 1

~~~js
console.log('First');

setTimeout(() => {
  console.log('Later');
}, 0);

console.log('Second');
~~~

### Understand the event loop, example 2

~~~js
console.log('Start');

setTimeout(() => console.log('Timer task'), 0);
Promise.resolve().then(() => console.log('Promise reaction'));

console.log('Finish');
~~~

### A Promise represents a future result, example 3

~~~js
const savedResult = new Promise((resolve) => {
  setTimeout(() => {
    resolve('Saved');
  }, 100);
});

savedResult.then((message) => {
  console.log(message);
});
~~~

### A Promise represents a future result, example 4

~~~js
Promise.resolve(5)
  .then((value) => value * 2)
  .then((value) => {
    console.log(value); // 10
  })
  .catch((error) => {
    console.error('The operation failed.', error);
  });
~~~

### Use async and await, example 5

~~~js
function wait(milliseconds) {
  return new Promise((resolve) => {
    setTimeout(resolve, milliseconds);
  });
}

async function showProgress() {
  console.log('Working...');
  await wait(100);
  console.log('Finished.');
}

showProgress();
~~~

### Use async and await, example 6

~~~js
async function loadProfile() {
  try {
    const profile = await getProfile();
    console.log(profile);
  } catch (error) {
    console.error('Could not load the profile.', error);
  }
}
~~~

### Run independent work in parallel, example 7

~~~js
async function loadDashboard() {
  const [profile, notifications] = await Promise.all([
    getProfile(),
    getNotifications(),
  ]);

  return { profile, notifications };
}
~~~

### Run independent work in parallel, example 8

~~~js
async function reportDashboardResults() {
  const results = await Promise.allSettled([
    getProfile(),
    getNotifications(),
  ]);

  for (const result of results) {
    if (result.status === 'fulfilled') {
      console.log('Value:', result.value);
    } else {
      console.error('Failure:', result.reason);
    }
  }
}

reportDashboardResults();
~~~

### Avoid unhandled rejections, example 9

~~~js
async function saveNote(note) {
  const response = await sendNote(note);
  return response;
}

saveNote({ title: 'Promises' }).catch((error) => {
  console.error('Could not save the note.', error);
});
~~~

### Use timers for scheduling, not exact timing, example 10

~~~js
const timerId = setTimeout(() => {
  console.log('Reminder');
}, 500);

clearTimeout(timerId);
~~~

### Use timers for scheduling, not exact timing, example 11

~~~js
let intervalId = setInterval(() => {
  console.log('Check for updates');
}, 5000);

clearInterval(intervalId);
~~~

### Hands-on: compare sequential and parallel work, example 12

~~~js
async function loadPageData() {
  console.time('sequential');
  const note = await getNote();
  const profile = await getProfile();
  console.timeEnd('sequential');

  console.time('parallel');
  const [parallelNote, parallelProfile] = await Promise.all([
    getNote(),
    getProfile(),
  ]);
  console.timeEnd('parallel');

  return { note, profile, parallelNote, parallelProfile };
}
~~~

## 12. Fetch and REST APIs

### Send a request with Fetch, example 1

~~~js
async function loadNotes() {
  const response = await fetch('/api/notes');

  if (!response.ok) {
    throw new Error('Request failed with status ' + response.status);
  }

  const notes = await response.json();
  return notes;
}
~~~

### Parse and validate a JSON response, example 2

~~~js
function isNote(value) {
  return (
    typeof value === 'object' &&
    value !== null &&
    typeof value.id === 'string' &&
    typeof value.title === 'string' &&
    typeof value.reviewed === 'boolean'
  );
}

async function getNotes() {
  const response = await fetch('/api/notes');

  if (!response.ok) {
    throw new Error('Could not load notes: ' + response.status);
  }

  const data = await response.json();

  if (!Array.isArray(data) || !data.every(isNote)) {
    throw new Error('The server returned invalid note data.');
  }

  return data;
}
~~~

### Send a JSON request, example 3

~~~js
async function createNote(note) {
  const response = await fetch('/api/notes', {
    method: 'POST',
    headers: {
      Accept: 'application/json',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(note),
  });

  if (!response.ok) {
    throw new Error('Could not create note: ' + response.status);
  }

  return response.json();
}
~~~

### Use query strings safely, example 4

~~~js
async function searchNotes(query) {
  const params = new URLSearchParams({ q: query });
  const response = await fetch('/api/notes?' + params);

  if (!response.ok) {
    throw new Error('Search failed: ' + response.status);
  }

  return response.json();
}
~~~

### Use query strings safely, example 5

~~~js
const endpoint = new URL('/api/notes', window.location.origin);
endpoint.searchParams.set('status', 'to-review');

console.log(endpoint.href);
~~~

### Handle network and HTTP errors, example 6

~~~js
async function showNotes() {
  try {
    const notes = await getNotes();
    renderNotes(notes);
  } catch (error) {
    console.error('Could not display notes.', error);
    showMessage('Notes could not be loaded. Try again.');
  }
}
~~~

### Cancel a request, example 7

~~~js
const controller = new AbortController();

fetch('/api/notes', { signal: controller.signal })
  .then((response) => {
    if (!response.ok) {
      throw new Error('Request failed: ' + response.status);
    }

    return response.json();
  })
  .then((notes) => console.log(notes))
  .catch((error) => {
    if (error.name === 'AbortError') {
      console.log('Request cancelled.');
      return;
    }

    console.error('Request failed.', error);
  });

controller.abort();
~~~

### Add a timeout, example 8

~~~js
async function fetchWithTimeout(url, timeoutMilliseconds) {
  const controller = new AbortController();
  const timerId = setTimeout(
    () => controller.abort(),
    timeoutMilliseconds,
  );

  try {
    return await fetch(url, { signal: controller.signal });
  } finally {
    clearTimeout(timerId);
  }
}
~~~

### Send updates and delete records, example 9

~~~js
async function updateNote(id, changes) {
  const response = await fetch('/api/notes/' + encodeURIComponent(id), {
    method: 'PATCH',
    headers: {
      Accept: 'application/json',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(changes),
  });

  if (!response.ok) {
    throw new Error('Could not update note: ' + response.status);
  }

  return response.json();
}

async function deleteNote(id) {
  const response = await fetch('/api/notes/' + encodeURIComponent(id), {
    method: 'DELETE',
  });

  if (!response.ok) {
    throw new Error('Could not delete note: ' + response.status);
  }
}
~~~

### Hands-on: load and render a list, example 10

~~~js
async function displayNotes() {
  const list = document.querySelector('#note-list');
  const status = document.querySelector('#load-status');

  if (!(list instanceof HTMLUListElement)) {
    return;
  }

  if (status) {
    status.textContent = 'Loading notes...';
  }

  try {
    const notes = await getNotes();
    list.replaceChildren();

    for (const note of notes) {
      const item = document.createElement('li');
      item.textContent = note.title;
      list.append(item);
    }

    if (status) {
      status.textContent = notes.length + ' notes loaded.';
    }
  } catch (error) {
    if (status) {
      status.textContent = 'Notes could not be loaded. Try again.';
    }

    console.error(error);
  }
}

displayNotes();
~~~

## 13. Modules, npm, and project structure

### Why split code into modules, example 1

~~~html
<script type="module" src="./src/main.js"></script>
~~~

### Export and import named values, example 2

~~~js
// src/notes/note-utils.js
export function countPending(notes) {
  return notes.filter((note) => !note.reviewed).length;
}

export const maximumTitleLength = 80;
~~~

### Export and import named values, example 3

~~~js
// src/main.js
import { countPending, maximumTitleLength } from './notes/note-utils.js';

console.log(countPending([]));
console.log(maximumTitleLength);
~~~

### Use a default export, example 4

~~~js
// src/notes/note-store.js
export default class NoteStore {
  constructor() {
    this.notes = [];
  }

  getAll() {
    return [...this.notes];
  }
}
~~~

### Use a default export, example 5

~~~js
// src/main.js
import NoteStore from './notes/note-store.js';

const store = new NoteStore();
console.log(store.getAll());
~~~

### Re-export related values, example 6

~~~js
export { countPending } from './note-utils.js';
export { default as NoteStore } from './note-store.js';
~~~

### Load a module dynamically, example 7

~~~js
async function openExportTools() {
  const tools = await import('./export-tools.js');
  tools.downloadNotes();
}
~~~

### Use npm to manage a project, example 8

~~~powershell
mkdir js-notes-practice
cd js-notes-practice
npm init -y
~~~

### Use npm to manage a project, example 9

~~~json
{
  "name": "js-notes-practice",
  "private": true,
  "type": "module",
  "scripts": {
    "test": "node --test"
  }
}
~~~

### Use npm to manage a project, example 10

~~~powershell
npm install package-name
npm uninstall package-name
npm test
~~~

### Understand dependencies and node_modules, example 11

~~~gitignore
node_modules/
coverage/
.env
~~~

### Example project structure, example 12

~~~text
js-notes-practice/
├── package.json
├── package-lock.json
├── index.html
├── src/
│   ├── main.js
│   ├── notes/
│   │   ├── note-store.js
│   │   └── note-utils.js
│   └── ui/
│       └── render-notes.js
└── test/
    └── note-utils.test.js
~~~

### Module values are live bindings, example 13

~~~js
// counter.js
export let count = 0;

export function increment() {
  count += 1;
}
~~~

### Module values are live bindings, example 14

~~~js
// main.js
import { count, increment } from './counter.js';

increment();
console.log(count); // 1
~~~

## 14. Prototypes, this, and classes

### Objects can delegate through prototypes, example 1

~~~js
const baseNote = {
  describe() {
    return 'A study note';
  },
};

const note = Object.create(baseNote);
note.title = 'Prototypes';

console.log(note.title);
console.log(note.describe());
console.log(Object.hasOwn(note, 'describe')); // false
console.log(Object.getPrototypeOf(note) === baseNote); // true
~~~

### Objects can delegate through prototypes, example 2

~~~js
const dictionary = Object.create(null);
dictionary['safe-key'] = 'value';

console.log(Object.getPrototypeOf(dictionary)); // null
~~~

### Understand this from the call site, example 3

~~~js
const learner = {
  name: 'Ashish',
  introduce() {
    return 'I am ' + this.name;
  },
};

console.log(learner.introduce());
~~~

### Understand this from the call site, example 4

~~~js
const introduce = learner.introduce;

// Calling introduce() has no learner receiver.
~~~

### Understand this from the call site, example 5

~~~js
const boundIntroduce = learner.introduce.bind(learner);
console.log(boundIntroduce());
~~~

### Understand this from the call site, example 6

~~~js
function describeRole(role) {
  return this.name + ' is a ' + role;
}

const person = { name: 'Ashish' };

console.log(describeRole.call(person, 'developer'));
console.log(describeRole.apply(person, ['developer']));
~~~

### Arrow functions capture this, example 7

~~~js
const counter = {
  value: 0,
  start() {
    setTimeout(() => {
      this.value += 1;
      console.log(this.value);
    }, 100);
  },
};

counter.start();
~~~

### Create instances with class, example 8

~~~js
class StudyNote {
  constructor(title, body) {
    this.title = title;
    this.body = body;
    this.reviewed = false;
  }

  markReviewed() {
    this.reviewed = true;
  }

  get summary() {
    return this.title + ': ' + this.body;
  }
}

const note = new StudyNote('Classes', 'Class methods use prototypes.');
note.markReviewed();

console.log(note.summary);
~~~

### Create instances with class, example 9

~~~js
console.log(Object.hasOwn(note, 'title')); // true
console.log(Object.hasOwn(note, 'markReviewed')); // false
console.log(Object.hasOwn(StudyNote.prototype, 'markReviewed')); // true
~~~

### Add private fields and static methods, example 10

~~~js
class StudyCounter {
  #count = 0;

  increment() {
    this.#count += 1;
    return this.#count;
  }

  get value() {
    return this.#count;
  }

  static create() {
    return new StudyCounter();
  }
}

const counter = StudyCounter.create();
counter.increment();

console.log(counter.value); // 1
~~~

### Extend a class carefully, example 11

~~~js
class ReferenceNote extends StudyNote {
  constructor(title, body, sourceUrl) {
    super(title, body);
    this.sourceUrl = sourceUrl;
  }

  openSource() {
    return this.sourceUrl;
  }
}

const reference = new ReferenceNote(
  'Prototypes',
  'Objects delegate property lookup.',
  'https://developer.mozilla.org/',
);

console.log(reference instanceof StudyNote); // true
console.log(reference.openSource());
~~~

### Choose an object, factory, or class, example 12

~~~js
function createProgress(initialValue = 0) {
  let value = initialValue;

  return {
    increment() {
      value += 1;
      return value;
    },
    getValue() {
      return value;
    },
  };
}

const progress = createProgress();
progress.increment();

console.log(progress.getValue());
~~~

### Hands-on: create a progress object, example 13

~~~js
class Progress {
  #value = 0;

  constructor(maximum) {
    this.maximum = maximum;
  }

  get value() {
    return this.#value;
  }

  increment() {
    if (this.#value < this.maximum) {
      this.#value += 1;
    }

    return this.#value;
  }

  reset() {
    this.#value = 0;
  }
}

const progress = new Progress(2);

console.log(progress.increment()); // 1
console.log(progress.increment()); // 2
console.log(progress.increment()); // Still 2
~~~

## 15. Collections, iterators, and generators

### Choose a collection for the data, example 1

~~~js
const tags = new Set();

tags.add('browser');
tags.add('arrays');
tags.add('browser');

console.log(tags.size); // 2
console.log(tags.has('arrays')); // true
~~~

### Choose a collection for the data, example 2

~~~js
const names = ['Ashish', 'Maya', 'Ashish'];
const uniqueNames = [...new Set(names)];

console.log(uniqueNames); // ['Ashish', 'Maya']
~~~

### Use Map for key-value collections, example 3

~~~js
const noteById = new Map();

noteById.set(1, { title: 'Collections' });
noteById.set(2, { title: 'Iterators' });

console.log(noteById.get(1));
console.log(noteById.has(2));
console.log(noteById.size);
~~~

### Use Map for key-value collections, example 4

~~~js
for (const [id, note] of noteById) {
  console.log(id, note.title);
}
~~~

### Use Map for key-value collections, example 5

~~~js
const plainEntries = Array.from(noteById.entries());
const jsonText = JSON.stringify(plainEntries);

console.log(jsonText);
~~~

### Understand weak collections, example 6

~~~js
const privateMetadata = new WeakMap();

function attachMetadata(object, metadata) {
  privateMetadata.set(object, metadata);
}

const element = {};
attachMetadata(element, { inspected: true });

console.log(privateMetadata.get(element));
~~~

### What an iterable provides, example 7

~~~js
const word = 'JS';

for (const character of word) {
  console.log(character);
}

console.log([...word]);
~~~

### What an iterable provides, example 8

~~~js
const iterator = ['first', 'second'][Symbol.iterator]();

console.log(iterator.next()); // { value: 'first', done: false }
console.log(iterator.next()); // { value: 'second', done: false }
console.log(iterator.next()); // { value: undefined, done: true }
~~~

### Make a custom iterable with a generator, example 9

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

### Make a custom iterable with a generator, example 10

~~~js
function* countTo(maximum) {
  for (let value = 1; value <= maximum; value += 1) {
    yield value;
  }
}

const values = [...countTo(4)];
console.log(values); // [1, 2, 3, 4]
~~~

### Make a custom iterable with a generator, example 11

~~~js
function* combineTopics() {
  yield 'values';
  yield* ['functions', 'objects'];
  yield 'browser APIs';
}

console.log([...combineTopics()]);
~~~

### Use generators for paged values, example 12

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

### Use typed arrays for binary data, example 13

~~~js
const bytes = new Uint8Array([65, 66, 67]);

console.log(bytes[0]); // 65
console.log(new TextDecoder().decode(bytes)); // ABC
~~~

### Hands-on: count note tags, example 14

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

## 16. Browser security, accessibility, and performance

### Treat input as untrusted, example 1

~~~js
const message = document.querySelector('#message');
const userText = new URLSearchParams(location.search).get('message');

if (message) {
  message.textContent = userText ?? '';
}
~~~

### Treat input as untrusted, example 2

~~~js
const heading = document.createElement('h2');
heading.textContent = 'A safe note title';
document.querySelector('main')?.append(heading);
~~~

### Validate URL schemes, example 3

~~~js
function getSafeWebUrl(value) {
  try {
    const url = new URL(value, location.origin);

    if (url.protocol !== 'https:' && url.protocol !== 'http:') {
      return null;
    }

    return url.href;
  } catch {
    return null;
  }
}

const safeUrl = getSafeWebUrl(userProvidedUrl);

if (safeUrl) {
  const link = document.createElement('a');
  link.href = safeUrl;
  link.textContent = 'Open reference';
  document.querySelector('main')?.append(link);
}
~~~

### Avoid unsafe dynamic code and data merging, example 4

~~~js
function makePreferences(input) {
  return {
    theme: input.theme === 'light' ? 'light' : 'dark',
    pageSize:
      Number.isInteger(input.pageSize) && input.pageSize > 0
        ? input.pageSize
        : 20,
  };
}
~~~

### Build accessible interactions, example 5

~~~html
<button id="save-note" type="button">Save note</button>
<p id="save-status" aria-live="polite"></p>
~~~

### Build accessible interactions, example 6

~~~js
const saveButton = document.querySelector('#save-note');
const saveStatus = document.querySelector('#save-status');

saveButton?.addEventListener('click', () => {
  if (saveStatus) {
    saveStatus.textContent = 'Note saved.';
  }
});
~~~

### Announce dynamic changes carefully, example 7

~~~html
<p id="save-status" aria-live="polite"></p>
<p id="error-message" role="alert"></p>
~~~

### Announce dynamic changes carefully, example 8

~~~js
const status = document.querySelector('#save-status');

if (status) {
  status.textContent = 'Three notes loaded.';
}
~~~

### Measure before optimizing, example 9

~~~js
const start = performance.now();

const visibleNotes = notes.filter((note) =>
  note.title.toLowerCase().includes(query.toLowerCase()),
);

const elapsed = performance.now() - start;
console.log('Filter took', elapsed, 'milliseconds');
~~~

### Reduce unnecessary repeated work, example 10

~~~js
let searchTimer;

function scheduleSearch(query) {
  clearTimeout(searchTimer);

  searchTimer = setTimeout(() => {
    performSearch(query);
  }, 250);
}
~~~

### Reduce unnecessary repeated work, example 11

~~~js
const fragment = document.createDocumentFragment();

for (const note of notes) {
  const item = document.createElement('li');
  item.textContent = note.title;
  fragment.append(item);
}

document.querySelector('#note-list')?.replaceChildren(fragment);
~~~

### Schedule visual updates with requestAnimationFrame, example 12

~~~js
let frameId;

function scheduleVisualUpdate() {
  cancelAnimationFrame(frameId);

  frameId = requestAnimationFrame(() => {
    document.documentElement.classList.add('updated');
  });
}
~~~

### Avoid leaks from listeners and retained data, example 13

~~~js
const controller = new AbortController();

window.addEventListener('resize', handleResize, {
  signal: controller.signal,
});

function cleanupView() {
  controller.abort();
  clearTimeout(searchTimer);
}
~~~

---

| [Previous: Browser security, accessibility, and performance](16-browser-security-accessibility-and-performance.md) | [Notes index](../README.md) | [Next: Complete Q&A](99-complete-q-and-a.md) |
|:--|:--:|--:|
