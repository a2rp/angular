# Angular Testing Fundamentals and the CLI Test Runner — Q&A

## 1. Which test runner does Angular CLI use?

New Angular CLI projects currently use Vitest by default. Existing projects may use Karma and Jasmine, which remain supported. Check angular.json and package.json to see what your project actually uses.

## 2. What does ng test do?

It invokes the project's configured test target through the Angular CLI. The builder prepares and runs the tests using the configured runner. Watch behavior and flags depend on the installed Angular CLI version and project configuration.

## 3. What is the difference between Vitest and TestBed?

Vitest is a test runner and assertion library. TestBed is Angular's utility for configuring dependency injection, compiling components, and creating fixtures. They solve different problems and are commonly used together when testing Angular behavior.

## 4. Should I use TestBed for a pure function?

Usually no. A function with no Angular dependencies can be imported and tested directly. TestBed adds setup that is only useful when the behavior depends on Angular injection, rendering, providers, or other framework features.

## 5. What is Arrange, Act, Assert?

Arrange the input and initial state, act by invoking the behavior, and assert on the observable result. This structure helps a reader understand what the test proves.

## 6. Should tests check private fields?

Usually not. Prefer assertions on behavior that a user or collaborating code can observe. Tests tied to private implementation details often fail during harmless refactoring without detecting a real regression.

## 7. What is a component fixture?

A ComponentFixture is a test handle for a created Angular component. It gives access to the component instance, rendered native element, and change-detection controls. Component testing lessons cover fixture use in detail.

## 8. Why can a test pass in Vitest but fail in a browser?

Vitest commonly uses a DOM emulator such as jsdom or happy-dom. These environments do not fully reproduce browser layout, rendering engines, focus behavior, native APIs, or assistive technology. Use real-browser testing for behavior that depends on those details.

## 9. What makes a test deterministic?

It produces the same result under the same conditions. Control time, random values, network responses, and external dependencies. Clean up mocks, timers, subscriptions, and shared state.

## 10. Does high code coverage mean the application is well tested?

No. Coverage shows which code ran, not whether assertions detect meaningful failures. Prioritize important behavior and confirm the test would fail if the requirement regressed.

## 11. Should every method have its own test?

No. Test meaningful behavior and boundaries. Some tiny implementation details are better covered through the larger behavior that uses them. Excessive one-assertion tests can make the suite brittle without improving protection.

## 12. Can a unit test replace an end-to-end test?

No. Unit tests quickly check focused behavior; end-to-end tests check an important journey through a running application. Use a balanced suite, with only as many broad tests as needed for critical workflows.

## 13. Should a team migrate every Karma suite to Vitest immediately?

Not automatically. Karma remains supported. Angular's current migration guidance describes migration as experimental and has prerequisites. Compare the benefits, configuration, and suite compatibility before changing a mature test setup.

## 14. How do tests fit into CI?

Run the configured test target in a non-watch mode in CI and fail the pipeline when tests fail. Use the flags supported by the project's installed CLI and avoid a command that silently ignores the test exit code.

## Further reading

- [Angular testing overview](https://angular.dev/guide/testing)
- [Angular Karma testing](https://angular.dev/guide/testing/karma)
- [Karma-to-Vitest migration](https://angular.dev/guide/testing/migrating-to-vitest)
- [Testing utility APIs](https://angular.dev/guide/testing/utility-apis)
