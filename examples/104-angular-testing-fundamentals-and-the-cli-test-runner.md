# Angular Testing Fundamentals and the CLI Test Runner - Examples

This walkthrough uses the Vitest setup commonly generated for a new Angular CLI project. If the existing workspace uses Karma/Jasmine, keep its configured runner or follow the migration guide deliberately.

## 1. Inspect and run the configured test target

Check the test target in angular.json and the testing packages in package.json. Then run the Angular CLI target:

~~~sh
ng test
~~~

For a single CI-style run, use the non-watch options supported by the installed CLI:

~~~sh
ng test --no-watch --no-progress
~~~

Check the local command help when an option differs by version:

~~~sh
ng test --help
~~~

The CLI configures the runner for typical projects. Avoid adding a custom Vitest configuration until the default behavior needs to change.

## 2. Test a pure function directly

A pure function does not need Angular dependency injection or a component fixture.

~~~ts
// src/app/cart/cart-total.ts
export interface CartLine {
  unitPriceCents: number;
  quantity: number;
}

export function totalCents(lines: CartLine[]): number {
  return lines.reduce(
    (total, line) => total + line.unitPriceCents * line.quantity,
    0
  );
}
~~~

~~~ts
// src/app/cart/cart-total.spec.ts
import { describe, expect, it } from 'vitest';
import { totalCents } from './cart-total';

describe('totalCents', () => {
  it('adds each line price multiplied by its quantity', () => {
    const result = totalCents([
      { unitPriceCents: 250, quantity: 2 },
      { unitPriceCents: 125, quantity: 3 }
    ]);

    expect(result).toBe(875);
  });

  it('returns zero for an empty cart', () => {
    expect(totalCents([])).toBe(0);
  });
});
~~~

The first test names a meaningful rule and checks the returned value. It does not test private implementation steps such as which loop or array method the function uses.

## 3. Test a service through Angular dependency injection

Use TestBed when the service's Angular registration or dependencies are part of the behavior:

~~~ts
// src/app/cart/cart-summary.service.ts
import { Injectable } from '@angular/core';
import { CartLine, totalCents } from './cart-total';

@Injectable({ providedIn: 'root' })
export class CartSummaryService {
  total(lines: CartLine[]): number {
    return totalCents(lines);
  }
}
~~~

~~~ts
// src/app/cart/cart-summary.service.spec.ts
import { TestBed } from '@angular/core/testing';
import { beforeEach, describe, expect, it } from 'vitest';
import { CartSummaryService } from './cart-summary.service';

describe('CartSummaryService', () => {
  let service: CartSummaryService;

  beforeEach(() => {
    TestBed.configureTestingModule({});
    service = TestBed.inject(CartSummaryService);
  });

  it('provides the cart total through Angular DI', () => {
    expect(
      service.total([
        { unitPriceCents: 500, quantity: 1 },
        { unitPriceCents: 200, quantity: 2 }
      ])
    ).toBe(900);
  });
});
~~~

A service test can use providers, fakes, or HTTP testing tools when those dependencies matter. Keep the TestBed setup focused on the behavior being checked.

## 4. Read the test in Arrange, Act, Assert order

In the totalCents example:

- Arrange: create cart lines with known prices and quantities.
- Act: call totalCents with the lines.
- Assert: check that the total is 875 cents.

When a test has many setup steps, extract only the reusable setup that clarifies the behavior. Avoid shared mutable fixtures that make a test depend on execution order.

## 5. Run tests in watch and CI modes

During development, watch mode reruns matching tests after a change. In CI, the runner should exit after the suite and return a failing status when an assertion fails. Confirm the command in the current CLI and CI environment, then use it consistently.

## 6. Optional runner configuration

Only if the defaults are insufficient, inspect the Angular CLI's Vitest configuration generator and current testing guide:

~~~sh
ng generate config vitest
~~~

Custom runner configuration can add maintenance and compatibility work. Keep it small, document why it exists, and review it when updating Angular CLI or Vitest.

## 7. What this example does not prove

A pure function test and a service test do not prove that the UI is usable or that the real browser renders the application correctly. Add component tests for template and interaction behavior, HTTP tests for request construction and response handling, and a small number of end-to-end tests for important user journeys in the following chapters.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="103-semantic-html-keyboard-access-and-accessible-angular-ui.md">103 - Semantic HTML, Keyboard Access, and Accessible Angular UI</a></td><td><a href="../lessons/104-angular-testing-fundamentals-and-the-cli-test-runner.md">Lesson</a></td><td><a href="../qnas/104-angular-testing-fundamentals-and-the-cli-test-runner.md">Q&amp;A</a></td><td><a href="105-component-tests-fixtures-and-template-behavior.md">105 - Component Tests, Fixtures, and Template Behavior</a></td>
</tr>
  </tbody>
</table>
