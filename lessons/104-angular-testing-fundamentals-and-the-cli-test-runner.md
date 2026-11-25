# Angular Testing Fundamentals and the CLI Test Runner

Tests give a repeatable way to check behavior and protect changes. An Angular test can exercise a plain function, a service with dependency injection, a component and its template, or a full user journey. Choose the smallest test boundary that can prove the behavior you care about, then add broader coverage for important interactions.

New Angular CLI projects currently use Vitest by default. Existing projects may use Karma and Jasmine; inspect the project configuration instead of assuming the runner from the Angular version alone.

## Learning outcomes

After this lesson, you should be able to:

- Identify the runner configured for an Angular project.
- Run tests through the Angular CLI.
- Structure a focused test with Arrange, Act, and Assert.
- Choose among pure unit, service, component, and end-to-end tests.
- Use TestBed when Angular dependency injection or rendering is part of the behavior.
- Keep tests deterministic and focused on observable behavior.
- Understand the limits of test coverage and DOM emulation.

## 1. Find the configured test runner

The Angular CLI provides an ng test command that runs the project's test target. In new CLI applications, Vitest is the default runner; Karma remains supported for existing projects. The CLI configuration and installed dependencies determine the actual behavior.

Inspect the project's angular.json test target and package.json. Newer projects commonly use a unit-test builder integrated with Vitest and a DOM emulator such as jsdom or happy-dom. Legacy projects may use the Karma builder and Jasmine.

~~~sh
ng test
~~~

For a non-interactive CI run, use the supported one-shot options for the installed CLI or the CI environment setting it detects. Check ng test --help for the version in your project.

The Angular testing guide documents the current [CLI and Vitest setup](https://angular.dev/guide/testing) and the [Karma setup](https://angular.dev/guide/testing/karma).

## 2. Choose the test boundary

A **unit test** checks a small function or class in isolation. It is usually fast and has few dependencies.

A **service test** checks behavior that may involve dependency injection, HTTP, or application state.

A **component test** checks the component together with its template and DOM behavior. TestBed and ComponentFixture are common tools for this boundary.

An **end-to-end test** checks a user journey in a running application. It can catch integration problems, but usually takes longer and has more environmental failure modes.

Use a layered strategy. Many small tests give fast feedback; a smaller number of broader tests cover important user journeys. A test's value depends on whether it detects meaningful regressions, not how many lines it executes.

## 3. Arrange, Act, Assert

A focused test usually has three steps:

1. Arrange the input and starting state.
2. Act by calling the behavior under test.
3. Assert on the observable result.

~~~ts
import { describe, expect, it } from 'vitest';
import { formatDisplayName } from './format-display-name';

describe('formatDisplayName', () => {
  it('trims surrounding whitespace', () => {
    const result = formatDisplayName('  Ada Lovelace  ');

    expect(result).toBe('Ada Lovelace');
  });
});
~~~

Use names that state the behavior and condition. Avoid tests whose only assertion is that an object exists when important outcomes matter.

## 4. Test behavior rather than private implementation

A test should protect a requirement, not dictate every internal line. Prefer checking returned values, rendered text, accessible roles, enabled states, and emitted effects that users or other code can observe.

Avoid assertions against private fields, exact implementation call order, or framework internals unless they are part of a documented contract. Tests coupled to implementation details fail during harmless refactoring and discourage useful improvements.

## 5. Use TestBed when Angular behavior matters

TestBed configures an Angular testing environment. Use it when the test needs Angular dependency injection, component compilation, a fixture, providers, or template rendering.

For a plain function with no Angular dependencies, importing TestBed adds unnecessary setup. Keep pure logic independent when that improves reuse and makes tests smaller.

A component test often creates a fixture, triggers change detection, interacts through the DOM, and checks the result. Component testing details are covered in the next lessons.

## 6. Keep tests deterministic

A deterministic test gives the same result under the same conditions. Control time, random values, network responses, and external state. Avoid depending on the developer's machine, live production services, current timezone, or uncontrolled animation timers.

Restore mocks and fake timers after each test. Ensure each test owns and cleans up the state it creates. Shared mutable state can make tests pass or fail depending on execution order.

## 7. Understand browser emulation and real browsers

Vitest commonly runs in Node with jsdom or happy-dom to simulate a browser DOM. Emulation is fast, but it is not a full browser. Layout, rendering, native dialogs, focus nuances, browser APIs, and accessibility behavior may differ.

Use real-browser testing for behavior that depends on actual rendering engines, platform APIs, or browser-specific interaction. Do not conclude that a site is visually correct because a DOM-based test passed.

## 8. Use coverage as a signal, not a target

Coverage reports show which code paths ran during tests. They do not prove that assertions are meaningful or that the user journey is correct. A test that executes every line but checks nothing useful provides weak protection.

Prioritize high-risk business rules, validation, permissions, state transitions, error handling, and accessible interaction. Review whether tests would fail if the behavior regressed.

## 9. Use the CLI consistently

Use ng test so Angular's configured builder prepares files and integrates the runner. For CI, run the project's non-watch command and preserve the failing exit status. Keep runner-specific options and custom configurations in the workspace's test target.

Vitest is not the only supported choice. If the application uses Karma, follow its setup and test semantics rather than mixing configuration styles casually. Angular's migration guide currently describes Karma-to-Vitest migration as experimental; review it before changing a mature suite.

## 10. Build a useful test suite

For each behavior, ask:

- What user-visible requirement should this protect?
- What is the smallest useful test boundary?
- Which values or dependencies need to be controlled?
- Does the assertion check a meaningful result?
- Does the test clean up its state?
- Is a real browser needed for this behavior?

A maintainable suite makes ordinary changes safer without encoding accidental implementation details.
