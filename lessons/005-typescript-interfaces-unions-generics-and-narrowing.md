# TypeScript Interfaces, Unions, Generics, and Narrowing

Angular applications pass structured data through components, services, forms, and HTTP calls. TypeScript interfaces and type aliases describe that data, unions represent alternatives, generics preserve relationships between types, and narrowing lets code safely work with one possibility at a time.

## Learning objectives

After this lesson, you should be able to:

- Describe an object shape with an interface or type alias.
- Use required, optional, and readonly properties appropriately.
- Explain structural typing and the purpose of excess-property checks.
- Choose a union for alternatives and an intersection for combined requirements.
- Use literal unions and discriminated unions to model finite choices and states.
- Write a generic function that preserves information about its input.
- Constrain a generic when its implementation needs a particular property.
- Narrow a value with runtime checks and make a switch exhaustive.
- Explain why declared types do not validate untrusted runtime data.

## Object shapes with interfaces

An interface describes the properties and operations that an object is expected to provide:

~~~ts
interface Task {
  id: number;
  title: string;
  completed: boolean;
  description?: string;
  readonly createdAt: Date;
}
~~~

Here, id, title, and completed are required. The question mark makes description optional. Reading an optional property may produce undefined, so code must handle the case where it is absent.

The readonly modifier prevents ordinary assignments to createdAt through this TypeScript type:

~~~ts
function markComplete(task: Task): void {
  task.completed = true;
  // task.createdAt = new Date(); // type error
}
~~~

Readonly is a compile-time restriction. It does not freeze an object at runtime.

Interfaces can describe methods and function-valued properties too:

~~~ts
interface TaskStore {
  find(id: number): Task | undefined;
  save(task: Task): void;
}
~~~

This gives a useful contract to code that uses a task store without tying that code to one particular implementation.

## Type aliases and interfaces

A type alias gives a name to a type expression:

~~~ts
type TaskId = number;

type Point = {
  x: number;
  y: number;
};
~~~

For ordinary object shapes, interfaces and type aliases can both work well. Interfaces can be extended and participate in declaration merging. Type aliases can name unions, intersections, tuples, primitives, and other type expressions.

~~~ts
interface Timestamped {
  createdAt: Date;
}

interface AuditedTask extends Task {
  updatedAt: Date;
}

type TaskWithTimestamp = Task & Timestamped;
~~~

Choose the form that communicates the model clearly and follows the project's conventions. They are related tools, not two different runtime object formats.

## Structural typing

TypeScript generally checks whether a value has the required structure, rather than whether it was created from a particular named interface. This is often called structural typing.

~~~ts
interface Coordinates {
  x: number;
  y: number;
}

function printCoordinates(point: Coordinates): string {
  return point.x + ', ' + point.y;
}

const origin = { x: 0, y: 0, label: 'origin' };
printCoordinates(origin);
~~~

The origin object has the required x and y properties, so it can be used where Coordinates is expected. Its additional label property does not prevent assigning an existing variable to that parameter.

A fresh object literal can receive an excess-property check:

~~~ts
// This may be rejected because the fresh literal includes an unknown property.
// printCoordinates({ x: 0, y: 0, label: 'origin' });
~~~

This check helps catch misspelled or unexpected properties in common situations. It does not mean TypeScript uses exact object types everywhere.

## Unions: one of several possibilities

A union type uses the vertical bar to say that a value may be one of several types:

~~~ts
type TaskIdInput = number | string;

function displayId(id: TaskIdInput): string {
  return String(id);
}
~~~

The value is either a number or a string. Until code narrows it, TypeScript only permits operations that are valid for both possibilities.

Literal unions are useful when a value must be one of a known set:

~~~ts
type TaskFilter = 'all' | 'active' | 'completed';

function setFilter(filter: TaskFilter): void {
  console.log('Selected filter:', filter);
}
~~~

A misspelled or unsupported filter value is rejected during type checking.

## Intersections: combine requirements

An intersection uses an ampersand to require a value to satisfy multiple types at once:

~~~ts
type Identified = {
  id: number;
};

type Labeled = {
  label: string;
};

type IdentifiedLabel = Identified & Labeled;

const item: IdentifiedLabel = {
  id: 7,
  label: 'Priority task',
};
~~~

The value must have both properties. An intersection is not the same idea as a union: a union allows alternatives; an intersection combines requirements.

## Narrowing a union

When a value has a union type, runtime checks can narrow it to a more specific type:

~~~ts
function formatId(id: number | string): string {
  if (typeof id === 'number') {
    return 'Task #' + id.toFixed(0);
  }

  return 'Task ' + id.toUpperCase();
}
~~~

Inside the first branch, TypeScript knows id is a number. After that branch returns, TypeScript knows the remaining possibility is a string. The typeof check is both JavaScript code and information the type checker understands.

Equality checks, instanceof, the in operator, truthiness checks, and user-defined type predicates can also narrow values. Use checks that are precise for the actual data; truthiness alone can accidentally treat valid values such as zero or an empty string as absent.

## Discriminated unions model state

A discriminated union gives each alternative a shared property with a distinct literal value. That property lets TypeScript determine which shape is present:

~~~ts
type LoadState<T> =
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; message: string };
~~~

The status property is the discriminator. Each state contains only the information relevant to that state. A loading state does not pretend to contain successful data, and an error state includes a message.

~~~ts
function describeTasks(state: LoadState<Task[]>): string {
  switch (state.status) {
    case 'loading':
      return 'Loading tasks';

    case 'success':
      return 'Loaded ' + state.data.length + ' tasks';

    case 'error':
      return 'Could not load tasks: ' + state.message;
  }
}
~~~

Inside each case, TypeScript narrows state to the matching alternative. This pattern is useful for Angular component state such as loading, loaded, and error views.

## Exhaustive checks with never

A switch can be made to report when a new state was added but not handled:

~~~ts
function describeTasksExhaustively(state: LoadState<Task[]>): string {
  switch (state.status) {
    case 'loading':
      return 'Loading tasks';

    case 'success':
      return 'Loaded ' + state.data.length + ' tasks';

    case 'error':
      return 'Could not load tasks: ' + state.message;

    default: {
      const impossibleState: never = state;
      return impossibleState;
    }
  }
}
~~~

When every union alternative has been handled, the default branch has type never. If another state is added later and not handled, assigning it to never produces a compiler error. This turns an incomplete state renderer into a visible maintenance task.

## Generics preserve type relationships

A generic lets a function or type work with different types while preserving information about which type was used.

~~~ts
function identity<T>(value: T): T {
  return value;
}

const numericId = identity(42);
const userLabel = identity('Asha');
~~~

TypeScript infers T from each call, so numericId is a number and userLabel is a string. The same T appears in the input and output, expressing that the function returns the same kind of value it received.

A generic collection helper can preserve the element type:

~~~ts
function firstItem<T>(items: T[]): T | undefined {
  return items[0];
}

const firstTask = firstItem<Task>([]);
~~~

The result is Task or undefined. In many calls, TypeScript can infer the type argument from the provided value, so explicit angle-bracket arguments are often unnecessary.

## Constrain a generic

A generic function can require every accepted type to provide a capability:

~~~ts
function findById<T extends { id: number }>(
  items: T[],
  id: number,
): T | undefined {
  return items.find((item) => item.id === id);
}
~~~

The constraint says T must have a numeric id. The function can safely read item.id and still returns the original specific item type T, including any additional fields it has.

Do not use a generic when the relationship does not need to be preserved. A concrete Task type is often simpler when a function only works with tasks.

## Runtime data needs runtime checks

An interface or type alias is removed from emitted JavaScript. It does not verify an HTTP response or other external value. Treat external data as unknown and check the properties the application relies on:

~~~ts
function isTask(value: unknown): value is Task {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    typeof value.id === 'number' &&
    'title' in value &&
    typeof value.title === 'string' &&
    'completed' in value &&
    typeof value.completed === 'boolean'
  );
}

function readTask(value: unknown): Task {
  if (!isTask(value)) {
    throw new Error('The response did not contain a valid task.');
  }

  return value;
}
~~~

The return type value is Task tells TypeScript that a true result from isTask narrows the checked value. The function body must actually perform adequate runtime checks; a type predicate is not automatically verified as logically correct.

For larger or more complex API contracts, a runtime schema-validation library may be useful. Type declarations alone are not validation.

## Common misunderstandings

- **“An interface creates a runtime class.”** It describes a type during checking; it does not create an object or constructor.
- **“An optional property always exists with an empty value.”** It may be absent, and reading it can produce undefined.
- **“Readonly freezes an object.”** It restricts assignments through a TypeScript view; it does not freeze the runtime object.
- **“A union allows every operation from every member.”** Only operations valid for the current narrowed member, or all members, are safe.
- **“An intersection means one of the types.”** It combines requirements; a union represents alternatives.
- **“A generic is the same as any.”** A generic preserves a relationship to a specific type; any discards checking.
- **“An interface validates an API response.”** It does not inspect runtime data. Add runtime checks at trust boundaries.
- **“A type predicate is proof just because it compiles.”** The predicate's implementation must correctly check the properties it claims.

## Knowledge check

1. When can an interface and a type alias both describe an object shape?
2. What does structural typing check?
3. How does a union differ from an intersection?
4. Why is a discriminated union useful for loading and error states?
5. What does a generic identity function preserve?
6. Why constrain a generic with extends?
7. How does a typeof check narrow a union?
8. What does assigning a switch value to never help detect?
9. Does readonly prevent changes to an object at runtime?
10. Why should an HTTP response still be validated even if a type describes it?

Answers are in [Chapter 5 Q&A](../qnas/005-typescript-interfaces-unions-generics-and-narrowing.md). The [Chapter 5 examples](../examples/005-typescript-interfaces-unions-generics-and-narrowing.md) put these patterns together in a small task-state model.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="004-typescript-essentials-values-types-and-inference.md">004 - TypeScript Essentials: Values, Types, and Inference</a></td><td><a href="../qnas/005-typescript-interfaces-unions-generics-and-narrowing.md">Q&amp;A</a></td><td><a href="../examples/005-typescript-interfaces-unions-generics-and-narrowing.md">Example</a></td><td><a href="006-classes-modules-decorators-and-compiler-options.md">006 - Classes, Modules, Decorators, and Compiler Options</a></td>
</tr>
  </tbody>
</table>
