# Test Routing, Forms, Accessibility, and User Flows

Feature tests should confirm that navigation, forms, and feedback work together from a user's point of view. A route test should check the rendered destination, not only a URL string. A form test should exercise controls and submission behavior. Accessibility checks should verify names, states, and keyboard-friendly semantics in the DOM.

## Learning outcomes

After this lesson, you should be able to:

- Test routed components with Angular's RouterTestingHarness.
- Assert navigation outcomes and rendered route content.
- Exercise forms through their controls and submit events.
- Verify validation, failure, success, and duplicate-submit behavior.
- Check accessible labels, descriptions, states, and focus.
- Choose a user flow that adds confidence without duplicating every unit test.

## 1. Test router behavior with the router harness

RouterTestingHarness creates a test fixture for routed components and allows a test to navigate to a URL. Configure routes with provideRouter and navigate through the harness.

~~~ts
TestBed.configureTestingModule({
  providers: [
    provideRouter([
      { path: 'orders/:id', component: OrderDetailPage },
      { path: 'orders', component: OrderListPage }
    ])
  ]
});

const harness = await RouterTestingHarness.create();
const page = await harness.navigateByUrl('/orders/42', OrderDetailPage);
~~~

Assert both the final URL and the rendered destination behavior when those are relevant. A URL alone does not prove the correct component rendered or displayed the route data.

## 2. Cover route parameters, redirects, and guards

Test the public navigation result for important routes: route parameters reach the correct page, redirects lead to the intended destination, and guards allow or reject navigation according to their contract.

Avoid retesting every internal Router implementation detail. Configure a small route table containing the behavior under test. Use test providers for the dependencies a guard or resolver needs.

## 3. Exercise forms like a user

Set values through form controls and dispatch the relevant input or blur events. Submit the rendered form and observe the visible result. Do not call private validation handlers directly if the requirement is that the form responds to user input.

Test invalid submission, corrected values, server rejection, successful submission, and disabled or pending states. Preserve entered values when validation fails. Prevent duplicate submissions while an operation is in progress.

## 4. Verify accessible form behavior

Check that each control has a programmatic name, help and error messages are associated with the right control, and invalid state is exposed when relevant. A test can inspect labels and ARIA attributes, but it cannot fully prove that a screen reader experience is clear.

When a failed submission should move focus, focus the first invalid field and verify that the target is focusable. Do not move focus unexpectedly on every keystroke.

## 5. Test navigation focus and announcements

After route navigation, an application may move focus to the new page heading or main region. Test that behavior if the application implements it. For async success or error feedback, verify the visible message and any appropriate status semantics.

Do not use role=alert for every status. Use appropriate semantics and confirm announcements manually with assistive technology.

## 6. Test a complete user flow at the right boundary

A component or routed integration test can cover a short interaction: navigate to an edit page, change a value, submit, and confirm success or a validation message. A full end-to-end test can cover the same flow against the running application, but reserve it for high-value journeys because it is more expensive to maintain.

Reuse unit tests for detailed business rules. Broader tests should verify integration and user-visible results, not duplicate all internal calculations.

## 7. Keep flow tests reliable

Control API responses with HttpTestingController or an appropriate fake. Avoid live services and timing guesses. Wait for actual router or fixture stability. Use stable accessible queries or well-defined page objects rather than brittle nested selectors.

A broad test that waits for arbitrary timeouts is usually flaky. Find the event that signals readiness and await that.

## 8. Combine automated and manual accessibility checks

Automated DOM assertions can detect missing labels, incorrect state attributes, or absent error text. They do not fully test keyboard order, visible focus, screen-reader phrasing, zoom behavior, or whether the task makes sense.

Use automated checks, keyboard review, and screen-reader review together. Test validation and navigation states, not only the default page.

## Further reading

- [Testing routing and navigation](https://angular.dev/guide/routing/testing)
- [Component testing scenarios](https://angular.dev/guide/testing/component-scenarios)
- [Angular form validation](https://angular.dev/guide/forms/form-validation)
- [Angular accessibility best practices](https://angular.dev/best-practices/a11y)
- [RouterTestingHarness API](https://angular.dev/api/router/testing/RouterTestingHarness)
