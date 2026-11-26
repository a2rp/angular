# TypeScript Essentials: Values, Types, and Inference — Examples

These examples let you inspect inferred types, write basic annotations, handle uncertain values, and distinguish compile-time checking from runtime behavior.

Read [Chapter 4](../lessons/004-typescript-essentials-values-types-and-inference.md) and use the [Q&A](../qnas/004-typescript-essentials-values-types-and-inference.md) while working.

## Run the main example

Create a scratch folder with a file named values.ts. You can use the TypeScript Playground to inspect the code without installing anything.

To compile locally, install Node.js and npm if they are not already available, then run:

~~~sh
npm init -y
npm install --save-dev typescript
~~~

Save the following as values.ts:

~~~ts
const productName = 'Notebook';
let itemCount = 2;
itemCount += 1;

const unitPrice = 19.99;
const labels = ['TypeScript', 'Signals', 'Angular'];
const labelLengths = labels.map((label) => label.length);

function calculateLineTotal(price: number, quantity: number): number {
  return price * quantity;
}

function greet(name = 'friend'): string {
  return 'Hello, ' + name;
}

const lineTotal = calculateLineTotal(unitPrice, itemCount);
const greeting = greet('Mina');

const missingName: string | null = null;
const displayName =
  missingName === null ? 'Guest' : missingName.toUpperCase();

function extractTitle(value: unknown): string {
  if (
    typeof value !== 'object' ||
    value === null ||
    !('title' in value) ||
    typeof value.title !== 'string'
  ) {
    throw new Error('Expected an object with a string title.');
  }

  return value.title;
}

const responseText = '{"title":"Review inferred types"}';
const parsedResponse: unknown = JSON.parse(responseText);
const responseTitle = extractTitle(parsedResponse);

function parseFiniteNumber(text: string): number | null {
  const parsed = Number(text);
  return Number.isFinite(parsed) ? parsed : null;
}

const parsedCount = parseFiniteNumber('42');

console.log('Product:', productName);
console.log('Item count:', itemCount);
console.log('Label lengths:', labelLengths);
console.log('Line total:', lineTotal.toFixed(2));
console.log(greeting);
console.log('Name to display:', displayName);
console.log('Response title:', responseTitle);
console.log('Parsed count:', parsedCount);
~~~

Compile with strict checking:

~~~sh
npx tsc --strict --noEmit values.ts
~~~

If the compiler prints no diagnostics, the example type-checks. To emit JavaScript and run it with Node.js:

~~~sh
npx tsc --strict --target ES2022 --module commonjs --outDir dist values.ts
node dist/values.js
~~~

Expected output:

~~~text
Product: Notebook
Item count: 3
Label lengths: [ 10, 7, 7 ]
Line total: 59.97
Hello, Mina
Name to display: Guest
Response title: Review inferred types
Parsed count: 42
~~~

The input values and code determine these results. If you change the initial values, the output changes too.

## Inspect the inferred types

Hover over each variable in an editor with TypeScript support, or use the Playground:

| Expression | Inferred or declared type | Why |
| --- | --- | --- |
| productName | The literal type 'Notebook' | It is a const initialized with a string literal |
| itemCount | number | It is a let initialized with a numeric value |
| unitPrice | number | It is a numeric literal stored in a const |
| labels | string[] | Every initial array element is a string |
| label | string | The map callback receives a string from labels |
| labelLengths | number[] | The callback returns a number for each element |
| greeting | string | greet has a string return annotation |
| parsedResponse | unknown | The response is intentionally treated as untrusted |
| parsedCount | number or null | The parser returns a number only when it is finite |

The const unitPrice cannot be reassigned, but its type is number rather than the exact decimal literal. TypeScript generally widens numeric literal types for variables when the exact literal type is not needed.

## See a type error on purpose

Add this near the itemCount declaration:

~~~ts
// @ts-expect-error A string cannot be assigned to a number.
itemCount = 'three';
~~~

The assignment is a type error. The directive tells TypeScript that this error is intentional in the teaching example, so the compiler does not report it as an unexpected diagnostic. Remove these two lines before using the program as normal application code.

Try changing the directive to a spelling that TypeScript does not recognize. The compiler will then report the assignment error. This demonstrates that types catch some mistakes before execution.

## Compare any with unknown

This snippet compiles, but the call is unsafe:

~~~ts
const unchecked: any = { name: 42 };
unchecked.name.toUpperCase();
~~~

The checker permits the call because unchecked is any. At runtime, name is a number and does not have the expected string method.

The main example treats parsedResponse as unknown. The extractTitle function checks that the input is a non-null object, has a title property, and that the property is a string before returning it. Those runtime checks both protect the operation and allow TypeScript to narrow the type.

## Compare conversion with assertion

Number conversion executes at runtime:

~~~ts
const rawText = '42';
const convertedNumber = Number(rawText);

console.log(convertedNumber);
console.log(typeof convertedNumber); // number
~~~

An assertion only changes the compiler's view:

~~~ts
const uncertainText: unknown = '42';
const claimedNumber = uncertainText as number;

console.log(typeof claimedNumber); // string
~~~

The last output is string. No conversion happened. If the code later calls a number-only method on claimedNumber, it can fail at runtime.

## Empty arrays and explicit types

An initialized array gives TypeScript values to inspect:

~~~ts
const chapterNumbers = [1, 2, 3];
// inferred as number[]
~~~

An empty array gives no element examples. State the intended element type when it is important:

~~~ts
const completedTitles: string[] = [];
completedTitles.push('Type checking');
~~~

## Try these changes

1. Change itemCount to a string and observe the type error.
2. Add a label to the labels array and see how labelLengths changes.
3. Change missingName to a real string and inspect displayName.
4. Pass a JSON object whose title is a number to extractTitle. The runtime check should throw.
5. Pass invalid numeric text to parseFiniteNumber. The result should be null.
6. Remove the type annotation from parsedResponse and inspect how JSON.parse is typed by the library. Consider why explicitly treating external data as unknown is safer.
7. Add an explicit return type to greet and compare it with the inferred type.

## Connect this to Angular

Angular components and services use the same TypeScript rules:

- Component fields have inferred or declared types.
- Function parameters define what a method accepts.
- Template expressions use the component's type information during compilation.
- Form and HTTP data may still need conversion or runtime validation.
- A type annotation does not make data from a user or server trustworthy.

As the course moves into interfaces and unions, you will learn how to define reusable object shapes and represent values that can have more than one type.

## Common issues

### TypeScript says the return value may be null

parseFiniteNumber explicitly returns number or null. Handle both cases before using the result as a definite number.

### TypeScript says a value is unknown

That is a prompt to check what the value actually is. Use runtime checks to establish the facts your code needs.

### The Playground displays a type you did not expect

Check whether the variable is mutable, whether it has an initializer, and whether an API provides a contextual type. Inference depends on those surrounding facts.

### The code compiles but fails at runtime

Static checking cannot prove that every external value is valid or that the program has no runtime errors. Inspect the failing value and add appropriate runtime validation.

## Key takeaway

Inference avoids redundant annotations, explicit types describe important contracts, and runtime checks protect data the type checker cannot inspect. Use these tools together rather than expecting a type declaration to convert or validate a value.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="003-browser-dom-http-and-javascript-foundations.md">003 - Browser, DOM, HTTP, and JavaScript Foundations</a></td><td><a href="../lessons/004-typescript-essentials-values-types-and-inference.md">Lesson</a></td><td><a href="../qnas/004-typescript-essentials-values-types-and-inference.md">Q&amp;A</a></td><td><a href="005-typescript-interfaces-unions-generics-and-narrowing.md">005 - TypeScript Interfaces, Unions, Generics, and Narrowing</a></td>
</tr>
  </tbody>
</table>
