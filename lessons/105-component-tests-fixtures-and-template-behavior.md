# Component Tests, Fixtures, and Template Behavior

A component's class is only part of its behavior. Its template binds data, responds to events, renders conditional content, and exposes semantics to users. A component test creates the component in Angular's test environment and checks the resulting DOM through behavior that matters to the consumer.

## Learning outcomes

After this lesson, you should be able to:

- Configure TestBed for a component and create a ComponentFixture.
- Distinguish the component instance, debug element, and rendered native element.
- Trigger change detection and wait for asynchronous stability.
- Set inputs and simulate user events correctly.
- Assert on visible output and accessible behavior.
- Avoid tests that depend on private implementation details.
- Choose a harness when a component has a reusable interaction contract.

## 1. Create the test environment

TestBed configures the providers, imports, and declarations for a test. For a standalone component, add it to the testing module's imports:

~~~ts
TestBed.configureTestingModule({
  imports: [ProfileCard]
});
~~~

Then create a fixture:

~~~ts
const fixture = TestBed.createComponent(ProfileCard);
const component = fixture.componentInstance;
const hostElement: HTMLElement = fixture.nativeElement;
~~~

The fixture is Angular's handle for the rendered component. It exposes the instance, host element, debug information, change detection, stability, and destruction APIs.

## 2. Know when bindings update

TestBed.createComponent creates the host and instance but does not necessarily synchronously perform the first change-detection pass. A test that reads the DOM immediately may see no bound content.

Use fixture.detectChanges() to run a change-detection pass directly. Use await fixture.whenStable() to wait for Angular's asynchronous work and pending change detection to settle. Choose one based on what the test is controlling; do not add repeated calls without knowing why.

A typical setup can be:

~~~ts
const fixture = TestBed.createComponent(ProfileCard);
fixture.detectChanges();
~~~

For asynchronous work:

~~~ts
await fixture.whenStable();
~~~

## 3. Set inputs through Angular

If a component has a required input, set it before asserting on the rendered output. Fixture.setInput is useful for testing a component's public input contract:

~~~ts
fixture.componentRef.setInput('profile', testProfile);
fixture.detectChanges();
~~~

For signal inputs, set the input through the component ref rather than assigning to the signal created by Angular. This exercises Angular's input behavior and allows the template to update normally.

## 4. Simulate real DOM events

Setting an input element's value property alone does not notify Angular. Dispatch the DOM event the template listens for:

~~~ts
input.value = 'Ada';
input.dispatchEvent(new Event('input'));
fixture.detectChanges();
~~~

Likewise, click the button element to trigger its click handler. Prefer user-visible interactions over directly calling private methods. Use the event type the browser would send for the interaction.

For higher-level component libraries, a harness or user-event utility can make interactions more realistic and protect tests from internal DOM changes.

## 5. Assert on observable behavior

Check rendered text, element presence, accessible attributes, disabled state, and emitted effects. For example, a submit button should become disabled when the form is invalid if that is the user-visible requirement.

Avoid assertions on private fields or exact DOM nesting unless the structure is a documented contract. A test should usually survive an internal refactor that leaves the interaction unchanged.

## 6. Keep asynchronous tests understandable

Use async tests and await the relevant fixture or promise stability. For timers, use the configured runner's fake timers when appropriate and restore real timers after each test.

Angular's legacy fakeAsync utilities depend on Zone.js. Current Angular guidance recommends native async approaches or runner fake timers for many cases. If using fakeAsync with Vitest, follow the Angular migration guide's setup and compatibility notes.

## 7. Choose stubs deliberately

A component test can replace a child component or provider when the test's concern is the parent behavior. Avoid stubbing so much that the test no longer verifies the behavior you intended to protect. A shallow test can check composition, while a focused child test can check that child's contract.

If a test repeatedly queries deep implementation markup, a component harness can expose stable operations such as clicking a button or reading a selected value. Harnesses are especially useful for shared components used by many consumers.

## 8. Beware of DOM emulation limits

Vitest with jsdom or happy-dom provides a fast DOM-like environment, not a full browser. It cannot fully verify layout, actual rendering, all focus behavior, or assistive technology announcements. Use browser-level checks when behavior depends on a real browser.

## 9. A component-test review checklist

- Did the test configure the component's required imports and providers?
- Were required inputs set before the first assertion?
- Did change detection or async stability run at the right time?
- Did user input dispatch the appropriate DOM event?
- Are assertions about user-visible or public behavior?
- Is asynchronous work controlled and cleaned up?
- Would a harmless template refactor break the test?
- Does the feature need a real-browser check in addition to a DOM test?

See Angular's component testing basics, component testing scenarios, and testing utility APIs.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="104-angular-testing-fundamentals-and-the-cli-test-runner.md">104 - Angular Testing Fundamentals and the CLI Test Runner</a></td><td><a href="../qnas/105-component-tests-fixtures-and-template-behavior.md">Q&amp;A</a></td><td><a href="../examples/105-component-tests-fixtures-and-template-behavior.md">Example</a></td><td><a href="106-test-services-dependency-injection-and-http-requests.md">106 - Test Services, Dependency Injection, and HTTP Requests</a></td>
</tr>
  </tbody>
</table>
