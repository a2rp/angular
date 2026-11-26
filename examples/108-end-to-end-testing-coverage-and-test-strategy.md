# End-to-End Testing, Coverage, and Test Strategy — Examples

This example configures a browser e2e target for an Angular application and tests an accessible help-search journey. Adapt the routes, labels, and setup to the application under test.

## 1. Add an e2e integration

Angular CLI can guide the setup for an e2e target. For a Playwright integration, one supported setup command is:

~~~sh
ng add playwright-ng-schematics
~~~

Follow the prompts and inspect the generated project configuration. The exact builder, scripts, and configuration files depend on the integration and its version.

Run the configured target through Angular CLI:

~~~sh
ng e2e
~~~

If the workspace already has an e2e target, use the existing setup rather than adding another runner.

## 2. Configure a local base URL

A Playwright config can point tests at the local development server. Use the generated configuration as the source of truth for your integration:

~~~ts
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  use: {
    baseURL: 'http://127.0.0.1:4200',
    ...devices['Desktop Chrome']
  },
  webServer: {
    command: 'ng serve',
    url: 'http://127.0.0.1:4200',
    reuseExistingServer: !process.env.CI
  }
});
~~~

The Angular CLI integration may generate its own config or wrap Playwright's options. Merge the intent into that file instead of replacing configuration blindly.

## 3. Test a user-visible search journey

~~~ts
// e2e/help-search.spec.ts
import { expect, test } from '@playwright/test';

test('a user can find a help article', async ({ page }) => {
  await page.goto('/help');

  await expect(
    page.getByRole('heading', { name: 'Help articles' })
  ).toBeVisible();

  await page
    .getByRole('searchbox', { name: 'Search help articles' })
    .fill('account');

  await page.getByRole('button', { name: 'Search' }).click();

  await expect(
    page.getByRole('link', { name: 'Protect your account' })
  ).toBeVisible();

  await expect(page.getByRole('status')).toContainText('result');
});
~~~

Role and label locators resemble the way users and assistive technology identify controls. Web-first assertions retry until the condition is met or the timeout expires, avoiding arbitrary sleep calls.

## 4. Control API data when the flow needs it

When a user journey depends on a backend response, intercept it with a deterministic test fixture or point the test at a dedicated test environment. For example, Playwright can fulfill a request locally:

~~~ts
await page.route('**/api/orders', async (route) => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify([
      { id: 'order-17', status: 'open' }
    ])
  });
});
~~~

Install the route before navigating or triggering the request. Do not intercept every API call if the purpose of the test is to verify the backend integration; choose the boundary deliberately.

## 5. Generate an Angular coverage report

For a Vitest-based Angular CLI project, install the coverage provider compatible with the project's Vitest version, then run:

~~~sh
npm install --save-dev @vitest/coverage-v8
ng test --coverage
~~~

Angular writes a coverage report to the configured output directory, commonly coverage/. Review uncovered branches for meaningful risk; do not add empty assertions only to raise a percentage.

## 6. A small layered strategy

| Layer | Example check | Purpose |
| --- | --- | --- |
| Unit | Search filtering or price calculation | Fast feedback on a focused rule |
| Component | Label, validation message, and submit state | Template and interaction behavior |
| Integration | Route, service, and controlled HTTP response | Angular pieces work together |
| End-to-end | Find a help article in a browser | Critical user journey works in a running app |

Avoid implementing the same detailed assertions in every layer. Broader tests should prove integration; focused tests should explain the local rule.

## 7. Keep CI data isolated

Use dedicated test accounts and repeatable data. Do not run destructive flows against production. Capture traces or screenshots on failure when useful, and ensure reports do not expose credentials or personal information.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="107-test-routing-forms-accessibility-and-user-flows.md">107 - Test Routing, Forms, Accessibility, and User Flows</a></td><td><a href="../lessons/108-end-to-end-testing-coverage-and-test-strategy.md">Lesson</a></td><td><a href="../qnas/108-end-to-end-testing-coverage-and-test-strategy.md">Q&amp;A</a></td><td><a href="109-angular-devtools-debugging-and-production-diagnostics.md">109 - Angular DevTools, Debugging, and Production Diagnostics</a></td>
</tr>
  </tbody>
</table>
