# 21. Built-in control flow with if and switch

Angular's built-in control flow blocks decide which template content to render. Use @if, @else if, and @else for conditions. Use @switch, @case, and @default when one value selects among several alternatives.

~~~html
@if (isSignedIn) {
  <p>Welcome back.</p>
} @else {
  <p>Sign in to continue.</p>
}
~~~

These blocks are Angular template syntax, not structural directives. They are available directly in current Angular templates; they do not require importing NgIf or NgSwitch.

## Learning goals

After this lesson, you should be able to:

- Render a block conditionally with @if.
- Chain alternative conditions with @else if and @else.
- Reuse a truthy condition result with the as alias.
- Select among values with @switch and @case.
- Explain strict matching, no fall-through, and @default behavior.
- Use exhaustive checking for a discriminated union where supported.
- Choose conditions that handle meaningful values such as zero and empty strings correctly.

## 1. Render a conditional block with @if

An @if block renders its contents when its expression is truthy:

~~~html
@if (isSignedIn) {
  <p>Welcome back, {{ displayName }}.</p>
}
~~~

If the condition is false, Angular does not render that block's contents. Conditional blocks create and remove embedded views; they do not merely hide the contents with CSS.

Conditions use Angular template expressions. Keep the expression readable, and move complicated decisions into component code or a computed value.

## 2. Add alternative branches

Use @else for one fallback branch and @else if for additional tests:

~~~html
@if (accountStatus === 'active') {
  <p>Your account is ready.</p>
} @else if (accountStatus === 'pending') {
  <p>Your account is being reviewed.</p>
} @else {
  <p>Contact support for help.</p>
}
~~~

Angular checks branches in order. The first matching condition renders; later branches are skipped. Use a final @else when every possible state should show something.

There can be multiple @else if branches, but only one final @else.

## 3. Understand truthiness

@if tests truthiness, not only whether a value is exactly true. For example, null, undefined, false, zero, the empty string, and NaN are falsy in JavaScript.

This can make a condition misleading if zero or an empty string is a valid value:

~~~html
@if (itemCount) {
  <p>{{ itemCount }} items</p>
}
~~~

When itemCount is zero, the block is not rendered. If zero is meaningful, test the exact condition:

~~~html
@if (itemCount >= 0) {
  <p>{{ itemCount }} items</p>
}
~~~

For nullable values, compare explicitly or use a type guard that communicates the intended case.

## 4. Reuse the condition result with as

An @if block can assign the result of its condition to a local alias:

~~~html
@if (user.profile; as profile) {
  <h2>{{ profile.name }}</h2>
  <p>{{ profile.email }}</p>
} @else {
  <p>No profile is available.</p>
}
~~~

The block renders only when the expression is truthy, and profile can be used inside that block. This is useful when a longer expression should be evaluated once for readable template code.

The alias is local to the conditional block and its nested content. Do not try to access it after the block.

## 5. Choose between @if and @switch

Use @if for boolean conditions, ranges, and unrelated predicates:

~~~html
@if (score >= passingScore) {
  <p>Passed</p>
} @else {
  <p>Try again</p>
}
~~~

Use @switch when one value selects among a set of named alternatives, such as a status or role:

~~~html
@switch (status) {
  @case ('loading') {
    <p>Loading...</p>
  }
  @case ('ready') {
    <p>Ready.</p>
  }
  @case ('failed') {
    <p>Could not load the data.</p>
  }
  @default {
    <p>Status is unknown.</p>
  }
}
~~~

A switch makes the relationship between one value and its possible views explicit. Do not use it for complicated boolean conditions that are clearer as if/else branches.

## 6. Understand @case matching

Angular compares the switch expression with each case using strict equality, equivalent to ===:

~~~html
@switch (selectedPage) {
  @case (1) {
    <p>Page one</p>
  }
  @case ('1') {
    <p>The string value one</p>
  }
}
~~~

The number 1 and the string '1' are different values and match different cases. Ensure the model's type and the case values agree.

## 7. Group cases without fall-through

Consecutive @case labels can share one block:

~~~html
@switch (userRole) {
  @case ('editor')
  @case ('reviewer') {
    <p>You can review content.</p>
  }
  @case ('viewer') {
    <p>You can read content.</p>
  }
}
~~~

Angular does not fall through from one case body to the next. No break statement is needed.

## 8. Use @default intentionally

An optional @default block renders when no case matches:

~~~html
@switch (status) {
  @case ('ready') {
    <p>Ready</p>
  }
  @default {
    <p>Waiting for a result</p>
  }
}
~~~

Without @default, no content is rendered when there is no matching case. Choose a fallback that is accurate and useful; do not label an unknown state as success.

## 9. Check union cases exhaustively

For a TypeScript union, a current Angular template checker can verify that all known cases are handled with @default never;:

~~~ts
type LoadState = 'loading' | 'ready' | 'failed';

export class DataPanel {
  state: LoadState = 'loading';
}
~~~

~~~html
@switch (state) {
  @case ('loading') {
    <p>Loading...</p>
  }
  @case ('ready') {
    <p>Data is ready.</p>
  }
  @case ('failed') {
    <p>Loading failed.</p>
  }
  @default never;
}
~~~

If LoadState later gains another member and the template has no matching case, Angular can report the missing case during template type checking. This syntax depends on Angular compiler support and TypeScript narrowing; verify it against the version used by the project.

Exhaustiveness checking works best when the switch expression is a typed variable. If the state is held in a signal or returned by a function, assign its value to an @let local first, then switch on that local where supported.

## 10. Scope and view lifecycle

A conditional block creates a nested template scope. Values declared inside it are available to that block and its children, not outside it.

When an @if branch becomes inactive, Angular removes its embedded view. When it becomes active again, Angular renders the branch again. Do not rely on the local DOM state of a removed element being preserved. Store durable application state in the component or an appropriate state service.

## 11. Migration from structural directives

Older Angular templates often use *ngIf and *ngSwitch with directive imports. Built-in control flow provides the newer block syntax:

~~~html
@if (isAvailable) {
  <p>Available</p>
} @else {
  <p>Unavailable</p>
}
~~~

Existing directive-based templates can continue to use their directives. When migrating, check that the new branch conditions and scopes preserve the intended behavior. The built-in blocks do not require NgIf or NgSwitch imports.

## Common mistakes

- Treating @if as an exact boolean check when it actually uses truthiness.
- Hiding valid values such as zero or an empty string with a truthiness condition.
- Expecting an alias from an @if block to be visible outside that block.
- Using @switch for ranges or unrelated boolean conditions.
- Assuming @case uses loose equality or converts strings to numbers.
- Adding break statements to @switch; there is no fall-through.
- Forgetting that no match and no @default means no content is rendered.
- Treating @default never; as a visual fallback; it is an exhaustiveness assertion.
- Assuming state inside a removed conditional view will remain in the DOM.
- Copying @default never; into a project whose Angular compiler does not support it.

## Practical checklist

1. Is the condition testing truthiness or a specific value?
2. Could zero, false, or an empty string be valid data here?
3. Would @if branches or a switch better communicate the states?
4. Are @case values the same type as the switch expression?
5. Does every possible state have a useful branch or fallback?
6. Is the alias used only inside its block?
7. Is exhaustive checking supported by this project's Angular version?
8. Does a branch need to preserve state after its view is removed?

## Further reading

- [Angular control flow](https://angular.dev/guide/templates/control-flow)
- [Angular variables in templates](https://angular.dev/guide/templates/variables)
- [Angular template expression syntax](https://angular.dev/guide/templates/expression-syntax)
- [Angular template syntax migration guide](https://angular.dev/reference/migrations/control-flow)
