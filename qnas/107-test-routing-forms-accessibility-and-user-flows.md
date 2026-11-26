# Test Routing, Forms, Accessibility, and User Flows — Q&A

## 1. What does RouterTestingHarness help test?

It creates a test fixture for routed components and provides navigation helpers. It is useful for checking that a route resolves, route data reaches the destination, and the expected UI is rendered.

## 2. Should a route test only assert Router.url?

Usually not. The URL is useful, but it does not prove the correct component rendered or that it used route parameters correctly. Assert the destination content when that is part of the behavior.

## 3. What route behaviors deserve focused tests?

Test important route parameters, redirects, guards, resolvers, and navigation outcomes. Keep the configured route table small and include only the behavior relevant to the test.

## 4. How should I exercise form validation in a component test?

Set the form controls through their rendered inputs, dispatch the relevant events, submit the form, and inspect the resulting messages and state. This checks the user-facing template behavior as well as the form model.

## 5. Should I call a private validation method to test an invalid form?

Generally no. If the requirement is that a user entering invalid data sees an error, drive the form through its controls and submit action. Private-method tests do not verify the template wiring.

## 6. What form states should I test?

Cover invalid values, corrected values, pending submission, server rejection, success, and duplicate-submit prevention where those states are relevant. Ensure entered values are preserved when validation fails.

## 7. How can a test check accessible validation?

Check that the control has an associated label, error text is present and connected through aria-describedby when appropriate, and invalid state is reflected. Also review keyboard focus and spoken feedback manually.

## 8. When should focus move after an invalid submission?

If the user submits an invalid form, moving focus to the first invalid control can help them correct it. Avoid moving focus on each keystroke or in ways that interrupt the user's current interaction.

## 9. What is the difference between an integration flow test and an end-to-end test?

An integration flow test can use Angular's test environment and controlled dependencies to verify several connected parts. An end-to-end test exercises the running application and its configured environment. The latter offers broader coverage but is typically slower and more sensitive to environment changes.

## 10. How do I keep a route or form flow deterministic?

Control server responses with fakes or HttpTestingController, await actual navigation or fixture stability, and avoid arbitrary delays. Use stable, user-facing selectors where possible.

## 11. Can an automated accessibility assertion prove the page is accessible?

No. It can catch some missing names, attributes, or relationships. It cannot fully judge keyboard usability, focus order, screen-reader phrasing, zoom, or whether the workflow is understandable. Combine it with manual review.

## 12. Should broad user-flow tests repeat every unit test?

No. Unit tests should cover detailed rules. A broader flow test should check that important parts integrate and the user sees the expected outcome, without duplicating every internal calculation.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="106-test-services-dependency-injection-and-http-requests.md">106 - Test Services, Dependency Injection, and HTTP Requests</a></td><td><a href="../lessons/107-test-routing-forms-accessibility-and-user-flows.md">Lesson</a></td><td><a href="../examples/107-test-routing-forms-accessibility-and-user-flows.md">Example</a></td><td><a href="108-end-to-end-testing-coverage-and-test-strategy.md">108 - End-to-End Testing, Coverage, and Test Strategy</a></td>
</tr>
  </tbody>
</table>
