# TypeScript Interfaces, Unions, Generics, and Narrowing — Q&A

This Q&A accompanies [Chapter 5: TypeScript Interfaces, Unions, Generics, and Narrowing](../lessons/005-typescript-interfaces-unions-generics-and-narrowing.md).

## Interfaces, aliases, and object types

### 1. What does an interface describe?

An interface describes the properties and operations that a value is expected to provide. It is a compile-time type contract; it does not create an object or runtime constructor.

### 2. What is a type alias?

A type alias gives a name to a type expression. It can name an object shape, a union, an intersection, a tuple, a primitive type, or another type expression.

### 3. Should an object shape always use an interface instead of a type alias?

No. Both can describe object shapes. Interfaces support extension and declaration merging; aliases can also represent unions and other combined types. Choose what best communicates the model and follows project conventions.

### 4. What does an optional property mean?

A property marked optional may be absent. When code reads it, the result may be undefined, so code should check for its presence before using it as a definite value.

### 5. Does readonly freeze an object?

No. Readonly restricts assignments through a TypeScript type view. It does not freeze the object at runtime or prevent another reference from changing it.

### 6. What is structural typing?

TypeScript usually checks whether a value has the required properties and compatible types, rather than whether it was declared with one exact interface name. A value with extra properties can still satisfy a type that requires only a subset of them.

### 7. What is an excess-property check?

It is a check TypeScript applies in certain cases, especially to fresh object literals, to catch unexpected or misspelled properties. It does not mean TypeScript requires exact object shapes for every assignment.

### 8. Does a named interface affect the runtime object?

No. The interface is type information used by the checker. The runtime object contains only the JavaScript properties and values that were created.

## Unions, intersections, and narrowing

### 9. What does a union type represent?

A union represents a value that may be one of several listed types. For example, string | number means either a string value or a number value.

### 10. What operations are allowed on an unnarrowed union?

Only operations that are valid for every possible union member. For a string-or-number union, code cannot call a string-only method until it checks that the value is a string.

### 11. What is a literal union?

It restricts a value to a set of specific literal choices, such as 'all' | 'active' | 'completed'. It helps catch unsupported or misspelled options.

### 12. What does an intersection type represent?

An intersection combines requirements from multiple types. A value of type A & B must satisfy both A and B.

### 13. How does a union differ from an intersection?

A union represents alternatives: a value may be one member. An intersection combines requirements: a value must satisfy all members.

### 14. What is narrowing?

Narrowing is TypeScript's process of using runtime checks and control flow to refine a broad type into a more specific type in a particular code branch.

### 15. Which checks can narrow a type?

Checks such as typeof, equality comparisons, instanceof, the in operator, discriminant properties, and user-defined type predicates can narrow types when their logic supplies useful evidence.

### 16. Why can truthiness checks be risky?

Valid values such as zero, an empty string, and false are falsy. If those values are allowed, a truthiness test may treat valid data as missing. Check the exact condition the program cares about.

### 17. What is a discriminated union?

It is a union whose alternatives share a property with distinct literal values. Checking that property lets TypeScript identify the matching alternative and its associated fields.

### 18. Why model loading states with a discriminated union?

It can represent loading, success, and error as distinct states with different required data. The success alternative carries data, while the error alternative carries an error message. This avoids loosely related booleans and optional fields that allow contradictory combinations.

### 19. What does never mean in an exhaustive switch?

After every possible union member has been handled, there should be no remaining value in the default branch. TypeScript represents that impossible state as never.

### 20. How does assigning a switch value to never help?

If a new union alternative is added and the switch does not handle it, the default branch is no longer never and the assignment produces a compiler error. That prompts the developer to update the handler.

## Generics

### 21. What problem do generics solve?

Generics let a function, interface, or type work with multiple types while preserving information about which type was used. They can express a relationship between inputs and outputs without replacing useful type information with any.

### 22. What does a generic identity function preserve?

It preserves the type of the input as the type of the output. Passing a number produces a number result; passing a string produces a string result.

### 23. Does a generic always need an explicit type argument?

No. TypeScript often infers a type argument from the values passed to a generic function. Explicit arguments are useful when inference is insufficient or when they make intent clearer.

### 24. Why constrain a generic with extends?

A constraint states what every accepted type must provide. For example, T extends { id: number } lets the function read an id while still preserving the specific T type.

### 25. Is a generic the same as any?

No. A generic preserves a type relationship and allows checking based on that type. Any largely disables checking and does not preserve a useful connection between input and output.

### 26. When is a generic unnecessary?

If a function works with only one known model and does not need to preserve a relationship across different types, a concrete type is often simpler and more readable.

### 27. Does a generic constraint prove external JSON is valid?

No. A generic constraint checks TypeScript values according to their declared types. It does not inspect network data at runtime. Validate external data before treating it as a trusted model.

## Runtime checks and Angular

### 28. Does defining an interface for an HTTP response validate the response?

No. The declaration is removed from emitted JavaScript and does not inspect the response body. Runtime validation or a suitable schema validator is needed when the application must trust that data.

### 29. What is a user-defined type predicate?

It is a function whose return type states that a true result means the checked parameter satisfies a specified type. The implementation must perform the checks accurately; TypeScript does not prove the predicate's logic is correct.

### 30. How do these types help Angular component state?

Interfaces or aliases can describe component data, unions can represent alternatives, generics can preserve data types through reusable helpers, and narrowing can render each state safely in a template or method.

### 31. Should every API service method use a generic response type?

Use generics where they preserve a real and clearly understood relationship. A generic response declaration still does not validate external data; do not make an unverified API response appear safe simply by annotating it.

### 32. What should a component show for each discriminated state?

A loading state should show progress, a success state should use the available data, and an error state should explain the failure and any available recovery action. The exact user experience depends on the application.

## Review scenarios

### 33. A function receives a string-or-number value. Why does calling toUpperCase fail?

The value may be a number. Narrow it with a runtime check, such as typeof value === 'string', then call the string method only in that branch.

### 34. A type guard returns true whenever a value is non-null. Can it claim that the value is a Task?

No. A predicate claiming Task must check the properties and value types that Task requires. A non-null check alone does not establish that shape.

### 35. A Task interface has an optional description. Is description always a string?

No. It can be absent, and reading the property can produce undefined. The code must handle both cases.

### 36. A switch handles loading, success, and error. A new cancelled state is later added. How can the compiler help?

If the switch uses an exhaustive never assignment in its default branch, TypeScript reports that the cancelled alternative is not handled.

### 37. Why can a variable with extra fields be passed to a function expecting a smaller interface?

Structural typing checks that the required fields are present and compatible. Extra fields on an existing variable do not generally prevent it from satisfying the narrower requirement.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="004-typescript-essentials-values-types-and-inference.md">004 - TypeScript Essentials: Values, Types, and Inference</a></td><td><a href="../lessons/005-typescript-interfaces-unions-generics-and-narrowing.md">Lesson</a></td><td><a href="../examples/005-typescript-interfaces-unions-generics-and-narrowing.md">Example</a></td><td><a href="006-classes-modules-decorators-and-compiler-options.md">006 - Classes, Modules, Decorators, and Compiler Options</a></td>
</tr>
  </tbody>
</table>
