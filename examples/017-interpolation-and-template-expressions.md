# Examples: Interpolation and Template Expressions

This walkthrough displays order information using interpolation and small Angular expressions. It includes ordinary component properties, a signal read, arithmetic, a conditional expression, nullish fallbacks, and a string that looks like HTML.

Use the Angular workspace from earlier lessons. Adapt the root component filenames to your generated project structure.

## 1. Create the order summary component

Create src/app/order-summary.ts:

~~~ts
import { Component, signal } from '@angular/core';

interface Customer {
  name: string;
  deliveryNote: string | null;
}

@Component({
  selector: 'app-order-summary',
  standalone: true,
  templateUrl: './order-summary.html',
})
export class OrderSummary {
  readonly customer: Customer | null = {
    name: 'Mina Patel',
    deliveryNote: null,
  };

  readonly itemName = 'Notebook';
  readonly unitPrice = 12.5;
  readonly quantity = signal(3);
  readonly isPriority = true;
  readonly missingValue: string | null = null;
  readonly markupLikeText = '<strong>Sale</strong>';
}
~~~

The component exposes values for the template to read. quantity is a signal, so the template calls quantity() to read its current value.

## 2. Add the template

Create src/app/order-summary.html:

~~~html
<section aria-labelledby="order-title">
  <h2 id="order-title">Order summary</h2>

  <p>Customer: {{ customer?.name ?? 'Guest' }}</p>
  <p>Item: {{ itemName }}</p>
  <p>Quantity: {{ quantity() }}</p>
  <p>Subtotal: {{ unitPrice * quantity() }}</p>

  <p>Delivery note: {{ customer?.deliveryNote ?? 'No delivery note' }}</p>
  <p>Priority: {{ isPriority ? 'Priority order' : 'Standard order' }}</p>
  <p>Optional field: [{{ missingValue }}]</p>
  <p>Text value: {{ markupLikeText }}</p>
</section>
~~~

The expressions demonstrate different jobs:

- customer?.name avoids reading name when customer is null.
- ?? provides a fallback only when a value is null or undefined.
- quantity() reads the signal value.
- unitPrice * quantity() performs a small calculation.
- The ternary expression chooses the priority label.
- missingValue is null, so its interpolation renders empty text and the brackets remain.

The markupLikeText value displays its angle-bracket characters as text. Interpolation does not turn that string into a strong HTML element.

## 3. Render it from the root component

Update the root component, commonly src/app/app.ts:

~~~ts
import { Component } from '@angular/core';
import { OrderSummary } from './order-summary';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [OrderSummary],
  templateUrl: './app.html',
})
export class App {}
~~~

Preserve any other dependencies the root template already uses.

Update src/app/app.html:

~~~html
<main>
  <h1>Template expression practice</h1>
  <app-order-summary />
</main>
~~~

The root component imports OrderSummary because its template uses the app-order-summary element. The expressions inside order-summary.html are evaluated in the OrderSummary context.

## 4. Run and inspect

From the Angular workspace root, start the app:

~~~sh
ng serve
~~~

The page should show Mina Patel, Notebook, quantity 3, subtotal 37.5, “No delivery note,” and “Priority order.” The optional field should be empty between the brackets. The final text should display the literal characters <strong>Sale</strong>, not bold markup.

## 5. Experiment with the expressions

Make one change at a time and observe the rendered text:

1. Change itemName and unitPrice in the component.
2. Change quantity's initial signal value and confirm the quantity and subtotal expressions both use the updated value.
3. Set customer to null. The customer name should use Guest, and the delivery note should use its fallback.
4. Set deliveryNote to an empty string. The ?? fallback should not replace the empty string because it is not null or undefined.
5. Change isPriority to false and observe the other ternary result.
6. Change missingValue to a non-empty string and observe the interpolation between the brackets.
7. Change markupLikeText to another harmless string containing angle brackets and confirm Angular displays it as text.

Restore or keep deliberate changes after the exercise. Avoid adding assignments or state changes to the interpolation expressions themselves.

## 6. Keep expressions maintainable

A short calculation such as unitPrice * quantity() is readable here. If an expression grows into several decisions or calculations, move that work into the component or an appropriate computed value and interpolate the result.

Methods called from templates should be inexpensive and free of side effects. Angular can evaluate expressions repeatedly while checking the view.

## Troubleshooting

- If the template reports that quantity is not callable, confirm it is a signal in the class and read it with parentheses.
- If an expression reports an unknown name, check spelling and confirm that the property belongs to OrderSummary.
- If the page shows a blank value, check whether the expression evaluates to null or undefined.
- If HTML-looking text appears literally, that is expected for text interpolation.
- If Angular reports a missing component, check that OrderSummary is imported by the root component.
- If a value does not update after an edit, inspect the terminal for compilation errors and the browser Console for runtime errors.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="016-angular-templates-and-html-structure.md">016 - 16. Angular templates and HTML structure</a></td><td><a href="../lessons/017-interpolation-and-template-expressions.md">Lesson</a></td><td><a href="../qnas/017-interpolation-and-template-expressions.md">Q&amp;A</a></td><td><a href="018-property-attribute-class-and-style-bindings.md">018 - 18. Property, attribute, class, and style bindings</a></td>
</tr>
  </tbody>
</table>
