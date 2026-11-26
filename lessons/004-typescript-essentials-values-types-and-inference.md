# TypeScript Essentials: Values, Types, and Inference

TypeScript is JavaScript with a static type checker and additional language features. Angular applications commonly use TypeScript for components, services, and application code. Learning how values and types relate helps you read Angular APIs, understand editor feedback, and catch mistakes before code runs.

## Learning objectives

After this lesson, you should be able to:

- Explain the difference between a runtime value and a TypeScript type.
- Add basic type annotations to variables and functions.
- Recognize common primitive types and array types.
- Read types that TypeScript inferred from a value or context.
- Decide when an explicit annotation helps.
- Explain what any, unknown, and strict null checking mean.
- Explain why TypeScript types do not validate incoming data at runtime.

## What TypeScript does

TypeScript checks relationships between values in your program while you develop and compile it. It can report that a string is being used where a number is required, or that a property is missing from a value.

TypeScript source is transformed into JavaScript for execution. Type annotations and most type-system syntax are removed; browsers run the resulting JavaScript, not TypeScript's static types. The JavaScript runtime behavior remains important: TypeScript does not make JavaScript operations behave differently just because a type was written beside them.

~~~ts
const quantity: number = 3;
const label: string = 'notebook';

console.log(label + ': ' + quantity);
~~~

The annotation tells the type checker that quantity should be a number. The value 3 is the runtime value. The type number is not a runtime wrapper around that value.

See the official TypeScript Handbook for the language's type-checking model.

## Values and types

A **value** is data available while a program runs: a string, number, boolean, object, function, or another JavaScript value.

A **type** describes which values are allowed in a particular place and which operations are valid there. Types help a developer and the checker reason about code. They do not, by themselves, sanitize, convert, or validate data arriving from outside the program.

~~~ts
let pageTitle: string = 'Tasks';
pageTitle = 'Projects'; // valid: this value is a string
// pageTitle = 42;      // type error: this value is a number
~~~

The checker reports the last assignment as an error before the program is built. JavaScript would not know about the string annotation at runtime.

## Basic types you will see often

| Type | Values it describes | Example |
| --- | --- | --- |
| string | Text | 'Angular' |
| number | JavaScript numbers, including integers and decimals | 12 or 3.5 |
| boolean | true or false | true |
| bigint | Large integer values | 123n |
| symbol | Unique symbol values | Symbol('key') |
| null | An intentional absence value | null |
| undefined | A missing or not-yet-assigned value | undefined |
| object | Non-primitive values | An object or array |

JavaScript has one general number type for ordinary numeric values; TypeScript does not split it into integer and floating-point types. Special numeric values such as NaN are also values of type number.

In everyday Angular code, you will frequently see string, number, boolean, arrays, object types, and values that may be null or undefined. Later chapters cover object shapes, interfaces, unions, and other reusable type patterns.

## Type annotations

A type annotation follows a variable or parameter name after a colon:

~~~ts
let username: string = 'Ada';
let retryCount: number = 0;
let isSaving: boolean = false;
~~~

Annotations also describe function parameters and return values:

~~~ts
function formatPrice(amount: number, currency: string): string {
  return currency + ' ' + amount.toFixed(2);
}

const displayPrice = formatPrice(12.5, 'USD');
~~~

The parameter annotations state what callers may pass. The return annotation states what the function promises to return. If the implementation returns a number instead of a string, TypeScript reports a mismatch.

A default parameter can provide an inferred type:

~~~ts
function greet(name = 'there'): string {
  return 'Hello, ' + name;
}

greet();
greet('Mina');
~~~

The default value gives name a string type, while the explicit return annotation documents the function result.

## Type inference

TypeScript often figures out a type from nearby information. This is called **type inference**.

~~~ts
let score = 10;
score = 12;
// score = 'twelve'; // type error

const projectName = 'Task Board';
~~~

TypeScript infers score as number and projectName as the literal string type 'Task Board'. The const variable cannot be reassigned, so the checker can preserve that specific literal type.

Inference appears in several common places:

- An initialized variable gets a type from its initial value.
- A function return type is inferred from its return expressions when no return annotation is given.
- An array's element type is inferred from its elements.
- A callback parameter can be inferred from the function or API that receives that callback.
- A default parameter receives a type from its default value.

~~~ts
const scores = [8, 10, 9];
// scores is inferred as number[]

const doubledScores = scores.map((score) => score * 2);
// score is inferred as number
// doubledScores is inferred as number[]
~~~

The map callback gets its context from the array. You do not need to repeat number on score.

Inference can sometimes be broader than you want. A mutable object property may be inferred as a general string even when it starts with one particular string. When the intended type is not clear, add a deliberate annotation or use a more specific type pattern covered in a later chapter.

The official Type Inference guide explains initializer inference and contextual typing.

## When to write a type annotation

Prefer inference when the initializer or surrounding API makes the type obvious:

~~~ts
const isReady = true;
const total = 4 + 6;
~~~

An annotation is useful when it improves clarity or provides information that cannot be inferred from the initializer:

~~~ts
let selectedTaskId: number | undefined;
const names: string[] = [];
~~~

The union syntax in the first example means the variable may hold a number or undefined; Chapter 5 develops union types in detail. The annotation on the empty array tells TypeScript what will be stored later.

Function parameter types are generally important because they define how callers may use the function:

~~~ts
function calculateSubtotal(unitPrice: number, quantity: number): number {
  return unitPrice * quantity;
}
~~~

Return types are often inferred for small private functions. Teams may still annotate public APIs or important functions to make contracts explicit. Follow a project's conventions rather than adding the same obvious annotation to every local variable.

## Arrays and collections

An array type describes the type of its elements. Two common spellings are:

~~~ts
const lessonNames: string[] = ['Setup', 'Components'];
const retryDelays: Array<number> = [100, 250, 500];
~~~

Both examples describe arrays of one element type. TypeScript can infer the same types when the array is initialized with values:

~~~ts
const lessonNumbers = [1, 2, 3];
// lessonNumbers is inferred as number[]
~~~

An empty array contains no values from which to infer an element type. In an important or non-obvious case, annotate it before adding values so the intended element type is clear.

## Type checking and JavaScript operations

TypeScript checks that an operation is allowed by the types it knows. It does not change the runtime result of a JavaScript operation.

~~~ts
const first = '10';
const second = 2;

const combined = first + second; // JavaScript concatenates: '102'
const numericTotal = Number(first) + second; // 12
~~~

The first expression is valid JavaScript because plus can concatenate strings. If the application expects a numeric addition, convert and validate the input deliberately. Type annotations alone do not turn text into numbers.

This matters for Angular forms: a text input's value is text, even if the user types digits. Convert it to a number only when the application needs numeric data, and decide how invalid or empty input should be handled.

## any and unknown

The **any** type opts out of most type checking for a value. It lets you access arbitrary properties and call it without the checker knowing whether those operations are safe.

~~~ts
let unchecked: any = 'hello';
unchecked.notARealMethod();
~~~

The code above type-checks, but it can fail at runtime. Avoid adding any just to silence an error; it removes useful checks from that part of the program.

The **unknown** type can hold a value whose type is not yet known, but the checker requires you to inspect it before using type-specific operations:

~~~ts
function printIfText(value: unknown): void {
  if (typeof value === 'string') {
    console.log(value.toUpperCase());
  }
}
~~~

The typeof check narrows value to string inside that branch. This gives you a safer starting point for handling uncertain input. Later lessons go deeper into narrowing and validation.

## null, undefined, and strict checking

JavaScript has both null and undefined. They commonly indicate that a value is absent, but they do not mean exactly the same thing in every API.

With strict null checking enabled, TypeScript makes you account for a value that might be null or undefined before using it as though it were present.

~~~ts
function showName(name: string | null): string {
  if (name === null) {
    return 'Name not provided';
  }

  return name.toUpperCase();
}
~~~

After the explicit check, TypeScript knows name is a string in the final line. Checking an optional or missing value is ordinary program logic, not something to bypass with a non-null assertion.

Angular projects commonly enable strict checking. If a compiler error says a value might be null or undefined, inspect how that value is produced and handle the absent case intentionally.

## TypeScript types do not validate external data

A type annotation describes what the program expects. It does not verify that a network response, browser storage entry, query parameter, or user-provided value actually has that shape.

~~~ts
type ApiTask = {
  id: number;
  title: string;
};

const task = JSON.parse(responseText) as ApiTask;
~~~

The assertion in the example does not parse or validate the task. It only tells the compiler to trust the programmer. If the JSON is malformed or contains a title of the wrong type, the assertion does not repair it.

At a system boundary, parse and validate data with runtime checks or a schema-validation library appropriate to the application. Later lessons cover object types and narrowing; the Angular HTTP chapters will apply those ideas to API responses.

## Type annotations versus type assertions

An annotation checks that an expression is compatible with a declared type:

~~~ts
const count: number = 3;
~~~

A type assertion uses the as syntax to tell the compiler to treat an expression as a chosen type. It does not convert or validate the runtime value. Assertions can hide real problems, so use them only when you have independent evidence the value is safe.

For conversion, use a runtime operation such as Number(value), and validate the result if the input is uncertain. Conversion and type assertions solve different problems.

## Common misunderstandings

- **“TypeScript types exist at runtime.”** Most type annotations are erased when TypeScript is emitted as JavaScript.
- **“A type annotation converts a value.”** It does not; conversion requires runtime code.
- **“An assertion proves data is valid.”** It only changes the compiler's view of the expression.
- **“Inference means every function parameter can be omitted.”** Standalone function parameters generally need annotations unless they receive a contextual type.
- **“any and unknown are equivalent.”** any disables checking; unknown requires checks before type-specific use.
- **“number means integer.”** It represents JavaScript's ordinary number values, including decimals and special numeric values.
- **“A typed API response is validated automatically.”** Type declarations do not inspect the bytes received from a server.

## Knowledge check

1. What is the difference between a value and a type?
2. What does TypeScript inference do?
3. Why can a callback parameter have a type without an explicit annotation?
4. What type would TypeScript infer for an array initialized with only strings?
5. Why is unknown safer than any for an input whose type has not been checked?
6. Does a type assertion convert a string to a number?
7. Why should code handle null and undefined explicitly?
8. Does declaring an HTTP response as an interface validate the JSON sent by a server?

Answers are in [Chapter 4 Q&A](../qnas/004-typescript-essentials-values-types-and-inference.md). Work through the [Chapter 4 examples](../examples/004-typescript-essentials-values-types-and-inference.md) to inspect inference, annotations, and runtime checks.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="003-browser-dom-http-and-javascript-foundations.md">003 - Browser, DOM, HTTP, and JavaScript Foundations</a></td><td><a href="../qnas/004-typescript-essentials-values-types-and-inference.md">Q&amp;A</a></td><td><a href="../examples/004-typescript-essentials-values-types-and-inference.md">Example</a></td><td><a href="005-typescript-interfaces-unions-generics-and-narrowing.md">005 - TypeScript Interfaces, Unions, Generics, and Narrowing</a></td>
</tr>
  </tbody>
</table>
