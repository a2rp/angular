# TypeScript Essentials: Values, Types, and Inference - Q&A

This Q&A accompanies [Chapter 4: TypeScript Essentials: Values, Types, and Inference](../lessons/004-typescript-essentials-values-types-and-inference.md).

## Values, types, and compilation

### 1. What is a value?

A value is data that a program can use while it runs, such as text, a number, true or false, an object, or a function.

### 2. What is a type?

A type describes which values are allowed in a particular position and which operations are valid there. TypeScript uses types to check code during development and compilation.

### 3. Does a TypeScript type exist in the browser at runtime?

Most TypeScript type annotations are removed when TypeScript is compiled to JavaScript. The browser runs the generated JavaScript and does not enforce those annotations.

### 4. Does adding a type annotation convert a value?

No. An annotation tells TypeScript what type is expected. To convert a runtime value, use an operation such as Number(value), String(value), or an application-specific parser.

### 5. Does TypeScript change how JavaScript operations work?

No. TypeScript checks JavaScript code and adds compile-time features, but ordinary JavaScript operations still have JavaScript runtime behavior. For example, adding a string and a number can concatenate them.

### 6. What is type erasure?

Type erasure is the removal of type-system syntax from the emitted JavaScript. For example, a variable annotation such as count: number does not become runtime validation code.

## Basic types and annotations

### 7. Which basic types should Angular beginners recognize?

Common types include string, number, boolean, arrays, object types, null, and undefined. TypeScript also has bigint, symbol, and other types used in more specialized code.

### 8. Does TypeScript have separate integer and floating-point types?

No. Ordinary numeric values use the number type, whether they represent integers, decimals, NaN, or other JavaScript number values.

### 9. Where does a variable type annotation go?

It follows the variable name after a colon, as in let count: number = 0.

### 10. Where do function parameter types go?

They follow each parameter name after a colon, as in function multiply(price: number, quantity: number).

### 11. Where does a function return type annotation go?

It follows the closing parenthesis of the parameter list and precedes the function body, as in function getLabel(): string.

### 12. Are type annotations required for every variable?

No. When TypeScript can infer an obvious type from an initializer or context, an extra annotation is often redundant. Add one when it clarifies intent, defines an API contract, or supplies information TypeScript cannot infer.

### 13. Why do function parameters usually need type annotations?

A standalone function parameter has no value or caller context from which to infer its intended type. Annotating parameters defines what callers can pass. A callback parameter can be inferred when its surrounding API supplies the context.

### 14. Can TypeScript infer a function return type?

Yes. TypeScript can infer the return type from the function's return expressions. A return annotation can still be helpful for an important or public function.

## Inference and arrays

### 15. What does type inference mean?

It is TypeScript's process of deriving a type from a value or context without an explicit annotation.

### 16. What type is inferred for let score = 10?

TypeScript normally infers number. The variable can be assigned another number, but assigning a string is a type error.

### 17. What type is inferred for const projectName = 'Task Board'?

The const variable is inferred with the literal string type 'Task Board', because it cannot be reassigned to a different value.

### 18. What type is inferred for an array initialized with strings?

Usually string[]. TypeScript infers the element type from the values in the array.

### 19. Why might an empty array need an annotation?

An empty array provides no elements from which to infer the intended item type. An explicit element type makes the intended collection clear and avoids unwanted inference.

### 20. What is contextual typing?

Contextual typing happens when a value's location provides type information. For example, when an arrow function is passed to an API that expects a string callback, TypeScript can infer the callback parameter as string.

### 21. Does TypeScript infer a standalone parameter type from all calls to a function?

Generally, no. Define the parameter type or place the function in a context that supplies it. Do not depend on TypeScript discovering a parameter's type from unrelated call sites.

## any, unknown, and nullish values

### 22. What does any do?

Any disables most type checking for that value. The checker permits arbitrary property access and function calls, even when those operations may fail at runtime.

### 23. Why should any be limited?

It removes type information and can allow mistakes to pass compilation. Use it only when there is a specific reason and a plan to restore safer types.

### 24. What is unknown?

Unknown can hold any value, but the checker requires code to inspect or narrow it before using type-specific operations. It is a safer starting point for data whose type has not been established.

### 25. How can code narrow an unknown value to a string?

Use a runtime check such as typeof value === 'string'. Inside the true branch, TypeScript knows the value is a string.

### 26. What is the difference between null and undefined?

Both are JavaScript values associated with absence, but APIs may use them differently. Null often represents an intentional empty value; undefined often represents a missing or uninitialized value. Check the specific API contract.

### 27. What does strictNullChecks do?

When enabled, it makes TypeScript account for null and undefined instead of silently treating them as values of every type. Code must check that a possibly absent value is present before using it as though it were not nullish.

### 28. Why is a null check useful even when the value has a type annotation?

The annotation may explicitly include null or undefined, or the value may come from an API that can be absent. A check protects the runtime behavior and helps TypeScript narrow the value.

## Runtime validation and assertions

### 29. Does declaring a type for JSON validate the server response?

No. The declaration tells TypeScript what the program expects, but it does not inspect the response bytes. Data from HTTP, local storage, or user input needs runtime validation when correctness or safety depends on its shape.

### 30. What does a type assertion do?

A type assertion tells the compiler to treat an expression as a chosen type. It does not convert, parse, or validate the runtime value.

### 31. When should an assertion be used?

Use one only when you have independent evidence that the value has the asserted type and the checker cannot represent that fact. Prefer runtime checks or parsing for untrusted input.

### 32. Why is Number(value) different from value as number?

Number(value) executes at runtime and attempts a conversion. An assertion changes only the compiler's view. Conversion can still produce an invalid numeric value, so check the result when input is uncertain.

### 33. Can an Angular form control's text input be typed as a number just because the user types digits?

No. A text input supplies text. Convert and validate it if the application needs a numeric value. Angular form APIs do not make arbitrary text become a number merely through a type annotation.

## Review scenarios

### 34. TypeScript infers a string for a variable, but the code later assigns a number. What should you do?

Decide what values the variable is intended to represent. If it should always be text, fix the assignment. If it intentionally has multiple possible types, define an appropriate type. Chapter 5 covers union types.

### 35. An editor reports that a value might be undefined. Should you add a non-null assertion immediately?

No. Trace where the value comes from and decide what should happen when it is absent. Add an explicit check, a fallback, or a documented invariant. Suppressing the warning does not make the value exist.

### 36. A type annotation makes an unsafe property access compile. Does that prove the value is safe?

No. The annotation may be incorrect or may describe only what the code expects. For external data, validate the value at runtime before relying on the declared shape.

### 37. Why is a callback parameter typed without an annotation in a common array example?

The array method provides contextual information. For example, the callback passed to a string array's map method receives a string element, so TypeScript can infer its parameter type.

### 38. Where should a beginner inspect an inferred type?

Hover over the variable, parameter, or expression in an editor that supports TypeScript, or use the TypeScript Playground. The displayed type helps explain what the compiler inferred.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="003-browser-dom-http-and-javascript-foundations.md">003 - Browser, DOM, HTTP, and JavaScript Foundations</a></td><td><a href="../lessons/004-typescript-essentials-values-types-and-inference.md">Lesson</a></td><td><a href="../examples/004-typescript-essentials-values-types-and-inference.md">Example</a></td><td><a href="005-typescript-interfaces-unions-generics-and-narrowing.md">005 - TypeScript Interfaces, Unions, Generics, and Narrowing</a></td>
</tr>
  </tbody>
</table>
