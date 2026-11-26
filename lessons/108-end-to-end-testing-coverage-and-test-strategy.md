# End-to-End Testing, Coverage, and Test Strategy

Unit and component tests give fast feedback about individual behaviors. End-to-end tests run a browser against the application and verify that important journeys work across routing, rendering, and configured integrations. They are valuable, but slower and more sensitive to environment, timing, and external systems.

A good test strategy balances confidence with maintenance cost. Test risk and user impact, not simply a target percentage.

## Learning outcomes

After this lesson, you should be able to:

- Explain which risks end-to-end tests cover.
- Set up an Angular e2e target with a browser testing tool.
- Design reliable user journeys with stable selectors and controlled data.
- Distinguish coverage metrics from test quality.
- Configure and interpret Angular coverage reports.
- Keep CI test suites useful and maintainable.
- Choose where unit, component, integration, and e2e tests belong.

## 1. What an end-to-end test checks

An e2e test drives a running application through a real browser interaction. It can reveal problems across route configuration, browser behavior, bundling, form submission, authentication integration, and deployed service wiring.

It does not replace unit tests. A failure in one long e2e flow can be harder to diagnose than a focused unit test, and e2e suites often run more slowly. Keep detailed business-rule assertions close to the relevant function or service; use e2e tests for a smaller set of critical user journeys.

## 2. Choose an e2e tool and configure the workspace

Angular CLI provides an ng e2e command that uses an e2e target. If the target is not configured, the CLI can guide setup with supported integrations. For example, a workspace can use Playwright, Cypress, Nightwatch, WebdriverIO, or Puppeteer through an appropriate package.

A typical setup flow is:

~~~sh
ng e2e
~~~

Use the Angular CLI's current end-to-end testing guide to select and configure a supported integration. Do not assume ng e2e is a built-in browser runner in every workspace; it invokes the project's configured target.

## 3. Pick high-value journeys

Select journeys that matter to users and cross important boundaries. Examples include:

- A user signs in and reaches a protected page.
- A customer searches, selects an item, and completes checkout.
- A user submits a form, receives validation, corrects it, and sees confirmation.
- A user without permission is blocked and shown a useful explanation.
- A core route loads correctly after deployment.

Avoid testing every button through e2e if a focused component test already covers its behavior. Prefer a small number of end-to-end flows that validate integration and release risk.

## 4. Make browser tests reliable

Use stable locators based on accessible role, label, or explicit test IDs that have a documented purpose. Avoid selectors tied to internal CSS classes or deeply nested DOM structure.

Wait for observable conditions instead of sleeping for an arbitrary number of milliseconds. Control test data with seeded fixtures or a dedicated test environment. Isolate browser contexts and restore the application state between tests.

Do not depend on public production services, shared accounts, uncontrolled email inboxes, or data that another test can change. If an external dependency must be used, make the limitation and retry policy explicit.

## 5. Keep test environments safe

Use dedicated test credentials, data, and backend resources. Never run destructive test flows against production. Avoid embedding secrets in source code, test reports, screenshots, or logs.

A test should clean up the state it creates. Parallel tests should not edit the same record unless the test is designed for that shared interaction. Prefer unique data and repeatable setup.

## 6. Understand coverage metrics

Code coverage reports estimate which statements, branches, functions, or lines executed during tests. Angular's CLI can generate a report with ng test --coverage. For Vitest coverage, the configured provider package may need to be installed.

Coverage can show unvisited code and guide investigation. It cannot prove that a test checks the correct outcome, uses realistic data, or would fail when behavior breaks. High coverage with weak assertions can provide false confidence.

Set thresholds thoughtfully. A threshold can prevent accidental declines, but teams should focus on high-risk branches and useful assertions rather than gaming the metric.

## 7. Use the right test layer

- **Unit tests** check a pure function or small class quickly.
- **Service and component tests** check Angular injection, templates, and interactions with controlled dependencies.
- **Integration tests** check several application parts working together.
- **End-to-end tests** check a user journey in a real browser and configured environment.

The exact naming varies between teams. The important choice is how much of the real application each test exercises and how difficult a failure is to diagnose.

## 8. Run tests deliberately in CI

A useful CI flow runs fast checks early and browser journeys in a controlled environment. Preserve failure exit codes and retain reports or screenshots when they help diagnosis. Do not automatically rerun every failing test until it passes; distinguish infrastructure flakes from real product failures.

Review test duration and failure history. Remove duplicate tests or replace brittle ones with clearer checks when the same confidence can be achieved at a lower maintenance cost.

## 9. Review the strategy regularly

Ask which high-impact journeys are not covered, whether failures are actionable, whether tests use isolated data, and which coverage gaps carry real risk. Update the suite when user flows or deployment architecture change.

Angular documents e2e testing setup and code coverage configuration. Playwright's official guide describes browser test installation and retrying web-first assertions.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="107-test-routing-forms-accessibility-and-user-flows.md">107 - Test Routing, Forms, Accessibility, and User Flows</a></td><td><a href="../qnas/108-end-to-end-testing-coverage-and-test-strategy.md">Q&amp;A</a></td><td><a href="../examples/108-end-to-end-testing-coverage-and-test-strategy.md">Example</a></td><td><a href="109-angular-devtools-debugging-and-production-diagnostics.md">109 - Angular DevTools, Debugging, and Production Diagnostics</a></td>
</tr>
  </tbody>
</table>
