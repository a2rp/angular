# 7. Strings, numbers, dates, and JSON

[Back to notes index](../README.md)

| [Previous: Objects, destructuring, and immutability](06-objects-destructuring-and-immutability.md) | [Notes index](../README.md) | [Next: The DOM and browser events](08-dom-and-browser-events.md) |
|:--|:--:|--:|

## Work with text

A string is an immutable sequence of text. String methods return a new string instead of changing the original value.

~~~js
const heading = '  JavaScript notes  ';
const cleaned = heading.trim();

console.log(cleaned);
console.log(heading); // The original still has its spaces
~~~

Common methods include includes, startsWith, endsWith, slice, split, replace, and replaceAll.

~~~js
const sentence = 'Functions make code reusable.';
const words = sentence.split(' ');

console.log(sentence.includes('code')); // true
console.log(sentence.slice(0, 9)); // Functions
console.log(words); // ['Functions', 'make', 'code', 'reusable.']
console.log(sentence.replace('reusable', 'clear')); // Functions make code clear.
~~~

String indexes count UTF-16 code units, not always user-perceived characters. A visible symbol can use multiple code units, especially emoji and some combined characters. Use Intl.Segmenter when an application needs language-aware word or grapheme boundaries.

~~~js
const text = 'JavaScript';
console.log(text.length); // 10
console.log([...text]); // Iterates Unicode code points
~~~

## Build text with template literals

Template literals use a backtick delimiter and can insert expressions with dollar-brace syntax. They are useful for readable messages and multi-line strings.

~~~js
const learnerName = 'Ashish';
const topicCount = 16;
const summary = `${learnerName} has ${topicCount} JavaScript topics to study.`;

console.log(summary);
~~~

An expression inside the braces can call a function or read a property. Keep complex work outside the string and store the result in a named variable first.

~~~js
const price = 1250;
const formattedPrice = price.toFixed(2);
const label = `Price: ₹${formattedPrice}`;

console.log(label);
~~~

A template literal does not escape HTML for safe insertion. Use textContent for text and follow safe DOM practices in Chapter 8.

## Understand number behavior

JavaScript's Number type uses floating-point arithmetic. Some decimal fractions cannot be represented exactly, so comparisons and sums can show a small difference.

~~~js
console.log(0.1 + 0.2); // 0.30000000000000004

const closeEnough = Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON;
console.log(closeEnough); // true
~~~

For fixed decimal amounts, consider storing integer minor units such as paise. For calculations that require exact decimal arithmetic, use a decimal library and understand how it rounds.

Useful methods include Number.isFinite, Number.isInteger, Number.isNaN, Number.parseInt, and Number.parseFloat.

~~~js
console.log(Number.isInteger(12)); // true
console.log(Number.isFinite(Infinity)); // false
console.log(Number.isNaN(Number('unknown'))); // true
~~~

Math.random is suitable for simple non-security random choices. It is not a secure source for passwords, tokens, or cryptographic values. Use the Web Crypto API for security-sensitive random bytes.

## Format numbers for people

Intl.NumberFormat formats numbers according to a locale and an optional style.

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

Keep numeric values numeric for calculations. Format them only for display. The same value can be rendered differently for another locale.

## Read and format dates

A Date represents a point in time as a timestamp. Date.now returns the current time in milliseconds since the Unix epoch.

~~~js
const now = new Date();
const timestamp = Date.now();

console.log(now.toISOString());
console.log(timestamp);
~~~

ISO strings with a time zone are safer for exchanging timestamps. Date-only strings and strings without an explicit time zone can be interpreted differently than expected. Decide whether the application value represents a calendar date or a precise instant, then parse and display it accordingly.

~~~js
const savedAt = new Date('2026-10-03T10:30:00Z');

if (!Number.isNaN(savedAt.getTime())) {
  console.log(savedAt.toISOString());
}
~~~

Use Intl.DateTimeFormat to display a date for a locale and time zone. Avoid manually joining date parts because ordering and month names vary.

~~~js
const dateFormatter = new Intl.DateTimeFormat('en-IN', {
  dateStyle: 'medium',
  timeStyle: 'short',
  timeZone: 'Asia/Kolkata',
});

console.log(dateFormatter.format(savedAt));
~~~

The time zone is an explicit choice. If an application displays the user's local time, omit timeZone and use the browser's configured zone. If a date represents a deadline in a specific zone, specify that zone.

## Match text with regular expressions

A regular expression describes a pattern used to search or validate text. The test method returns a boolean.

~~~js
const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

console.log(emailPattern.test('ashish@example.com')); // true
console.log(emailPattern.test('not-an-email')); // false
~~~

This small pattern checks a basic shape, not every valid email address. Do not use a short regular expression as proof that an address exists. For a form, use an email input and confirm the address through the appropriate service.

A global regular expression has state when test is called repeatedly. Reset its lastIndex or create a new expression when repeated checks behave unexpectedly.

~~~js
const globalPattern = /js/g;

console.log(globalPattern.test('js notes')); // true
console.log(globalPattern.test('js notes')); // false because lastIndex changed
~~~

Regular expressions are useful for simple text patterns. Prefer explicit parsing for complex structured input when the rules are easier to express as code.

## Convert values to JSON

JSON is a text format used to exchange data. JSON.stringify converts supported JavaScript values into JSON text. JSON.parse turns JSON text into JavaScript values.

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

JSON does not represent every JavaScript value. Functions and symbols are omitted from objects, undefined object properties are omitted, and circular references cause JSON.stringify to throw.

~~~js
const record = {
  title: 'Optional value',
  extra: undefined,
  callback: () => true,
};

console.log(JSON.stringify(record));
// {"title":"Optional value"}
~~~

JSON.parse can throw a SyntaxError when the input is not valid JSON. Catch that error when parsing user-provided, stored, or network text.

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

Valid JSON can still have the wrong shape for the application. Check important fields after parsing before using them.

## Hands-on: format a study note

Create a note object with a title, word count, and update time. Format the title with a template literal, count with Intl.NumberFormat, and date with Intl.DateTimeFormat. Then serialize the object and parse it again.

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

Try a malformed date and malformed JSON. Decide how your code should report each invalid value.

## Notes to remember

- String methods return new strings because strings are immutable.
- Template literals insert expressions into readable text.
- Number uses floating-point arithmetic, so decimal calculations can have rounding differences.
- Use Intl formatters for locale-aware numbers and dates.
- Specify the time zone when the meaning requires one.
- JSON cannot represent every JavaScript value and parsing can fail.
- Parsed JSON still needs shape checks before the application relies on it.

## References

- [MDN String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)
- [MDN template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)
- [MDN Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)
- [MDN Intl.NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat)
- [MDN Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
- [MDN Intl.DateTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat)
- [MDN regular expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions)
- [MDN JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON)

---

| [Previous: Objects, destructuring, and immutability](06-objects-destructuring-and-immutability.md) | [Notes index](../README.md) | [Next: The DOM and browser events](08-dom-and-browser-events.md) |
|:--|:--:|--:|
