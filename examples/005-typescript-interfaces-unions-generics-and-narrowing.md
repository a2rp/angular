# TypeScript Interfaces, Unions, Generics, and Narrowing - Examples

This exercise combines an object interface, a literal union, a generic helper, a discriminated state union, an exhaustive switch, and runtime validation for unknown input.

Read [Chapter 5](../lessons/005-typescript-interfaces-unions-generics-and-narrowing.md) and refer to its [Q&A](../qnas/005-typescript-interfaces-unions-generics-and-narrowing.md).

## Create a TypeScript scratch file

Create a file named task-model.ts. You can paste the code into the TypeScript Playground to inspect inferred types.

To compile it locally with strict checks, set up a small folder:

~~~sh
npm init -y
npm install --save-dev typescript
~~~

Save the following source as task-model.ts:

~~~ts
interface Task {
  id: number;
  title: string;
  completed: boolean;
  description?: string;
}

type TaskFilter = 'all' | 'active' | 'completed';

type LoadState<T> =
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; message: string };

interface TaskSummary extends Task {
  category: string;
}

type TimestampedTask = Task & {
  createdAt: Date;
};

function formatTaskId(id: number | string): string {
  if (typeof id === 'number') {
    return 'Task #' + id.toFixed(0);
  }

  return 'Task ' + id.toUpperCase();
}

function findById<T extends { id: number }>(
  items: readonly T[],
  id: number,
): T | undefined {
  return items.find((item) => item.id === id);
}

function describeLoadState(state: LoadState<Task[]>): string {
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

function isTask(value: unknown): value is Task {
  if (typeof value !== 'object' || value === null) {
    return false;
  }

  if (!('id' in value) || typeof value.id !== 'number') {
    return false;
  }

  if (!('title' in value) || typeof value.title !== 'string') {
    return false;
  }

  if (!('completed' in value) || typeof value.completed !== 'boolean') {
    return false;
  }

  if (
    'description' in value &&
    typeof value.description !== 'string'
  ) {
    return false;
  }

  return true;
}

function parseTasks(value: unknown): Task[] {
  if (!Array.isArray(value)) {
    throw new Error('Expected the response to be an array.');
  }

  const candidates: unknown[] = value;
  const tasks: Task[] = [];

  for (const candidate of candidates) {
    if (!isTask(candidate)) {
      throw new Error('The response contains an invalid task.');
    }

    tasks.push(candidate);
  }

  return tasks;
}

const payload: unknown = JSON.parse(
  '[{"id":1,"title":"Write the interface","completed":true},{"id":2,"title":"Model loading and error states","completed":false,"description":"Use a discriminated union"}]',
);

const tasks = parseTasks(payload);
const selectedTask = findById(tasks, 2);

const filter: TaskFilter = 'active';
const successState: LoadState<Task[]> = {
  status: 'success',
  data: tasks,
};
const loadingState: LoadState<Task[]> = {
  status: 'loading',
};
const errorState: LoadState<Task[]> = {
  status: 'error',
  message: 'The service is unavailable.',
};

console.log(formatTaskId(7));
console.log(formatTaskId('A-7'));
console.log('Filter:', filter);
console.log(describeLoadState(loadingState));
console.log(describeLoadState(successState));
console.log(describeLoadState(errorState));

if (selectedTask !== undefined) {
  console.log('Selected:', selectedTask.title);
}

for (const task of tasks) {
  if (task.completed) {
    console.log('Complete:', task.title);
  } else {
    console.log('Still active:', task.title);
  }
}
~~~

Compile the file without emitting JavaScript:

~~~sh
npx tsc --strict --noEmit task-model.ts
~~~

No compiler diagnostics means the declared types, generic helper, runtime guard, and state handling are consistent with one another.

## Expected output

~~~text
Task #7
Task A-7
Filter: active
Loading tasks
Loaded 2 tasks
Could not load tasks: The service is unavailable.
Selected: Model loading and error states
Complete: Write the interface
Still active: Model loading and error states
~~~

## Follow the types

Hover over these values in an editor or in the Playground:

| Expression | Type | What it shows |
| --- | --- | --- |
| tasks | Task[] | Runtime-checked array with a named object shape |
| filter | TaskFilter | One of three allowed literal values |
| successState | LoadState<Task[]> | Success state carrying an array of tasks |
| loadingState | LoadState<Task[]> | Loading alternative with no data field |
| selectedTask | Task or undefined | Lookup can fail to find a match |
| candidate inside parseTasks | unknown | External values need checking |
| result of findById | The caller's specific item type or undefined | Generic input/output relationship is preserved |

## Try invalid values

Add these lines after the filter declaration to see how the checker catches unsupported values:

~~~ts
// @ts-expect-error 'archived' is not a supported filter.
const invalidFilter: TaskFilter = 'archived';
~~~

The error directive marks the assignment as intentionally invalid for this exercise. Remove these lines from normal application code.

Try removing one required property from an object typed as Task. TypeScript reports the missing property. Then remove the optional description property; the object remains valid.

## Try the narrowing

In formatTaskId, TypeScript allows toFixed only in the branch where typeof id is number. The other branch is narrowed to string.

In the task loop, checking task.completed selects the true or false case of the boolean. In more complex unions, a discriminant such as state.status identifies which fields are available.

In describeLoadState, each switch case narrows state to one alternative. The never assignment provides an exhaustiveness check. Add another status to LoadState without adding a matching case; the default assignment should then become a compiler error.

## Try the generic lookup

findById accepts an array of any item type that has a numeric id. It still returns that specific item type rather than a generic object:

~~~ts
const found = findById(tasks, 1);
// found is Task or undefined
~~~

Try passing an array of objects with an id and an extra field. The result preserves that extra field's type too. Try passing objects without a numeric id; they do not meet the generic constraint.

## Try invalid runtime data

Change the JSON payload so a task has a numeric title:

~~~json
[
  {
    "id": 3,
    "title": 123,
    "completed": false
  }
]
~~~

parseTasks throws because the runtime guard checks the title's actual type. Changing only a TypeScript annotation would not perform that check.

The guard validates the shape used by this exercise. A production validator may also need to check business rules, such as whether an id is positive or a title is non-empty.

## What this example does not prove

A type predicate communicates a claim to TypeScript, but the compiler does not prove that its implementation checked every required invariant. If isTask returned true without checking title or completed, the predicate would be unsound even if the file compiled.

A generic constraint describes static input expectations; it does not inspect JSON. Runtime guards handle that boundary.

## Connect the model to Angular

An Angular feature could use Task as the item type in a component or service, TaskFilter for a filter control, and LoadState<Task[]> for its loading view. A template can branch on the state's discriminant and show the data or error fields appropriate to the selected state.

The TypeScript types help the editor and compiler check the code. Angular renders the chosen state, while runtime validation remains responsible for untrusted response data.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="004-typescript-essentials-values-types-and-inference.md">004 - TypeScript Essentials: Values, Types, and Inference</a></td><td><a href="../lessons/005-typescript-interfaces-unions-generics-and-narrowing.md">Lesson</a></td><td><a href="../qnas/005-typescript-interfaces-unions-generics-and-narrowing.md">Q&amp;A</a></td><td><a href="006-classes-modules-decorators-and-compiler-options.md">006 - Classes, Modules, Decorators, and Compiler Options</a></td>
</tr>
  </tbody>
</table>
