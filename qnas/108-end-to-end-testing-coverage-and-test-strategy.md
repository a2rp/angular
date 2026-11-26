# End-to-End Testing, Coverage, and Test Strategy - Q&A

## 1. What does an end-to-end test verify?

It drives a running application through a real browser and checks an important user journey across integrated parts such as routing, rendering, and configured services. It helps find issues that isolated unit tests cannot reveal.

## 2. Does Angular CLI include an e2e browser runner automatically?

Not necessarily. The CLI can invoke an e2e target, but the workspace must configure a tool or builder. If no target exists, the CLI can guide setup. Check the workspace's current Angular e2e documentation.

## 3. Which journeys belong in e2e coverage?

Choose high-impact journeys that cross important boundaries, such as checkout, protected-page access, or completing a critical form. Keep detailed calculations and local component behavior in faster focused tests.

## 4. Why are e2e tests often more expensive to maintain?

They involve a running app, browser, test data, environment configuration, and sometimes external services. Failures can be slower to diagnose and can be affected by timing or infrastructure.

## 5. How can I reduce flaky browser tests?

Use stable accessible locators, wait for observable conditions, isolate browser contexts, control test data, and avoid fixed sleeps. Do not share mutable records between parallel tests unless that sharing is intentional.

## 6. Should e2e tests use production credentials or data?

No. Use dedicated test accounts, safe test environments, and isolated data. Never put secrets in source code, logs, reports, or screenshots, and do not run destructive test flows against production.

## 7. What does a coverage percentage measure?

It estimates which statements, branches, functions, or lines ran during tests. It does not measure whether the assertions were meaningful or whether the tests would catch a real regression.

## 8. How do I generate Angular coverage?

For supported Angular CLI configurations, run ng test with the coverage option. Vitest coverage may require its configured coverage provider package. Check the installed CLI and project's coverage options.

## 9. Should a team enforce an 80 percent threshold?

Only if the threshold supports a real quality goal and the team understands what it measures. A threshold can prevent accidental drops, but a percentage is not a substitute for testing high-risk behavior with useful assertions.

## 10. Can high coverage create false confidence?

Yes. Tests may execute many lines without checking important results. A test suite should be reviewed for the failures it would catch, not only the percentage it reports.

## 11. How do unit and e2e tests complement each other?

Unit tests give fast feedback on focused rules. E2e tests verify that a smaller number of important user journeys work across the browser and configured application. Neither layer should duplicate the entire purpose of the other.

## 12. Should CI automatically retry every failed e2e test?

No. Repeated retries can hide product defects and make failures seem harmless. Identify whether the cause is infrastructure or application behavior, and make retries deliberate and bounded.

## 13. What should happen to browser reports and screenshots?

Retain them when they make failures easier to diagnose, while avoiding secrets or sensitive user data. Use the CI system's access controls and retention policies.

## 14. How often should a test strategy be reviewed?

Review it as the product, deployment, and risk profile change. Look for high-impact gaps, duplicate tests, noisy flakes, and tests that no longer reflect real user behavior.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="107-test-routing-forms-accessibility-and-user-flows.md">107 - Test Routing, Forms, Accessibility, and User Flows</a></td><td><a href="../lessons/108-end-to-end-testing-coverage-and-test-strategy.md">Lesson</a></td><td><a href="../examples/108-end-to-end-testing-coverage-and-test-strategy.md">Example</a></td><td><a href="109-angular-devtools-debugging-and-production-diagnostics.md">109 - Angular DevTools, Debugging, and Production Diagnostics</a></td>
</tr>
  </tbody>
</table>
