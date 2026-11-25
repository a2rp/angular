# Component Tests, Fixtures, and Template Behavior — Q&A

## 1. What does TestBed do in a component test?

TestBed configures Angular's testing environment, including imports, providers, and component compilation. It can create a component fixture so the test can inspect the instance and rendered view.

## 2. What does ComponentFixture represent?

It is a test handle for a component instance and its host view. It provides access to the instance, native host element, debug element, change detection, stability, and destruction.

## 3. Why might the DOM be empty immediately after createComponent?

Creating the fixture does not necessarily run the first change-detection pass synchronously. Bound template content may not appear until the test runs change detection or waits for Angular to become stable.

## 4. What is the difference between detectChanges and whenStable?

detectChanges explicitly asks Angular to run change detection now. whenStable waits for relevant asynchronous work and pending updates to settle. Use the one that matches the behavior under test; neither is a universal substitute for the other.

## 5. How should I set a required input in a test?

Use fixture.componentRef.setInput with the public input name and a suitable value, then render or wait as appropriate. This exercises Angular's input handling, including signal inputs, without mutating internal implementation state.

## 6. Why does setting input.value not update an Angular template?

Angular listens for DOM events. Assigning a DOM property alone does not dispatch the event the template uses. Set the value and then dispatch the relevant event, such as input, before checking the updated UI.

## 7. Should a component test call its click handler directly?

Prefer clicking the rendered button or dispatching the expected interaction. That verifies the template binding between the actual control and the behavior. Directly calling a method may be useful for a focused unit test, but it does not check the template connection.

## 8. What should component tests assert?

Check visible output, element presence, accessible names and states, disabled behavior, and public events or effects. Avoid private fields and incidental DOM structure unless those details are explicitly part of the component contract.

## 9. Should every child component be included in a parent test?

No. Include real children when their integration matters. Stub a child when the parent test only needs to verify its own composition, but do not stub away the behavior the test claims to cover.

## 10. When should I use a component harness?

A harness is useful for a shared or complex component whose consumers need stable operations such as selecting an option or clicking a button. It keeps tests from depending on internal DOM structure and can support multiple test environments.

## 11. How do I test timers or asynchronous updates?

Use async tests and await the relevant stability or promise. For controlled timers, use the runner's fake-timer API when suitable and restore real timers after each test. Avoid waiting for arbitrary wall-clock delays.

## 12. Should I use fakeAsync in a Vitest project?

Angular's fakeAsync utilities depend on Zone.js and require additional setup in Vitest. Current Angular guidance recommends native async or runner fake timers for many use cases. Follow the version-specific migration guide if the existing suite needs fakeAsync.

## 13. Can jsdom verify layout or focus behavior completely?

No. jsdom and happy-dom emulate parts of the DOM but do not fully reproduce browser rendering, layout, all focus behavior, or assistive technology. Use a real browser for those requirements.

## 14. What does a stable test look like?

It controls asynchronous work and external values, cleans up timers and mocks, and asserts on public behavior. It should not rely on test ordering or a particular internal sequence of method calls.

## Further reading

- [Component testing basics](https://angular.dev/guide/testing/components-basics)
- [Component testing scenarios](https://angular.dev/guide/testing/component-scenarios)
- [Angular testing utility APIs](https://angular.dev/guide/testing/utility-apis)
- [Creating component harnesses](https://angular.dev/guide/testing/creating-component-harnesses)
