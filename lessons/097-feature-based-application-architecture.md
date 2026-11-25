# Feature-Based Application Architecture

As an Angular application grows, file placement becomes an architectural decision. A good structure helps people find the code that implements a user workflow, understand which parts belong together, and change a feature without searching through unrelated folders.

This chapter uses **feature-based architecture**: organize application code around the product capabilities and workflows it supports. It is a practical convention, not an Angular runtime requirement. Angular works with many folder structures; choose one the team can follow consistently.

## Learning outcomes

After this lesson, you should be able to:

- Organize a growing Angular application by feature and workflow.
- Distinguish feature code from truly cross-feature shared code.
- Keep components, templates, styles, and focused tests discoverable.
- Load a feature route without importing its implementation everywhere.
- Set boundaries that reduce accidental coupling and circular imports.
- Avoid both a single crowded folder and unnecessary abstraction layers.

## 1. Organize around user-facing capabilities

Imagine an application for managing orders. The product has workflows for browsing orders, viewing an order, editing an order, and tracking fulfillment. A feature-oriented layout groups the code that changes together:

~~~text
src/app/
  app.config.ts
  app.routes.ts
  core/
    auth/
    http/
  shared/
    date-format/
    status-badge/
  features/
    orders/
      data-access/
        orders-api.service.ts
        order.model.ts
      pages/
        order-list-page/
          order-list-page.ts
          order-list-page.html
          order-list-page.spec.ts
        order-detail-page/
          order-detail-page.ts
          order-detail-page.html
      ui/
        order-summary/
          order-summary.ts
          order-summary.html
      orders.routes.ts
    inventory/
      data-access/
      pages/
      ui/
~~~

A feature folder gives related concepts a home. A developer changing order details can find the page, data access, and feature UI without opening a global components folder containing every page in the application.

The example is a starting point, not a universal directory template. A small app may only need a few feature folders. Split a feature into subfolders when its size makes navigation difficult; avoid creating empty layers just to match a diagram.

## 2. Prefer feature folders over global type buckets

A type-based structure often starts like this:

~~~text
components/
services/
models/
pipes/
directives/
~~~

As the application grows, a single folder can contain code from many unrelated domains. A change to one workflow then requires jumping between distant locations. Feature folders keep related files together:

~~~text
features/
  orders/
  inventory/
  accounts/
~~~

A feature may still contain focused subfolders such as pages, data-access, or ui when that improves clarity. The important boundary is the product capability, not the exact set of folder names.

Angular's style guide recommends grouping closely related files and organizing code by feature areas. It also recommends one main concept per file. See the [Angular style guide](https://angular.dev/style-guide).

## 3. Separate feature-owned and application-wide code

Use a feature folder for code that has a clear owner:

- A page used only by the orders workflow belongs in orders.
- An order-specific API service and order model normally belong in orders.
- A reusable component that only makes sense for orders belongs in orders/ui.
- A small presentation component used by several independent features may belong in shared.
- Authentication setup, global HTTP configuration, and application-wide error handling may belong in core or app configuration.

Do not move code into shared just because two files import it today. Ask whether it has a stable meaning across features, a clear public API, and a reason to change independently. If it is specific to one workflow, keep it with that workflow.

Avoid making shared a dumping ground. Shared code with broad dependencies can become the most tightly coupled part of the application.

## 4. Keep feature boundaries understandable

A page should coordinate its view and user actions. Domain rules and remote data access should live in focused services or functions when that separation makes behavior clearer or reusable. A component should not know every detail of the HTTP transport if a feature API service can own that concern.

Keep dependencies flowing in a direction that is easy to explain:

~~~text
Application shell
      |
      v
Feature page -> Feature UI and feature data access
      |
      v
Small shared presentation utilities
~~~

The diagram is a guide, not a rule that every import must follow exactly. Avoid cycles such as orders importing inventory internals while inventory imports orders internals. If two features need to coordinate, define a narrow contract or let a higher-level route/container compose them.

Prefer importing a feature's intended public entry points over reaching into deeply nested implementation files. This becomes especially valuable if a feature is moved, split, or tested independently.

## 5. Use routing to establish feature entry points

Angular routes can load a page when a user visits its feature URL. This keeps route configuration focused on navigation and gives each feature a clear entry point:

~~~ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: 'orders',
    loadChildren: () =>
      import('./features/orders/orders.routes')
        .then((module) => module.ORDER_ROUTES)
  },
  {
    path: '',
    pathMatch: 'full',
    redirectTo: 'orders'
  }
];
~~~

The feature can define its own routes:

~~~ts
import { Routes } from '@angular/router';

export const ORDER_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./pages/order-list-page/order-list-page')
        .then((module) => module.OrderListPage)
  },
  {
    path: ':orderId',
    loadComponent: () =>
      import('./pages/order-detail-page/order-detail-page')
        .then((module) => module.OrderDetailPage)
  }
];
~~~

Lazy loading affects when code is loaded; it does not automatically create good architecture. A route should still have clear ownership, sensible dependencies, and appropriate loading and error states. Follow the current Angular [routing guide](https://angular.dev/guide/routing).

## 6. Choose provider scope deliberately

A service provided at the application root is shared by the application injector. That is appropriate for long-lived application-wide state or shared infrastructure. Feature-specific state may belong to a route or component provider when it should be created and destroyed with that feature instance.

Provider scope affects lifetime and sharing. Avoid placing user-specific or feature-specific mutable state in a root singleton unless that lifetime is intended. Choose route, component, or application scope based on ownership, not folder name alone.

## 7. Keep files and tests discoverable

Group a component with the template, styles, and focused tests that explain its behavior. Use file names that make the concept and feature obvious. Avoid giant files that mix route definitions, unrelated models, multiple pages, and network code.

As a feature grows, split it when there is a concrete navigation or ownership benefit. Do not create a new abstraction only because a folder looks more symmetrical.

## 8. A practical architecture review

Before adding or moving code, ask:

1. Which user capability owns this behavior?
2. Does this code belong to one feature or is it genuinely cross-feature?
3. Who owns the state and how long should it live?
4. Can the feature's public entry point stay small and clear?
5. Are imports moving in a consistent direction?
6. Can someone find the implementation and its tests without knowing the author's history?
7. Is the new layer solving a real maintenance problem?

Feature-based architecture is successful when it reduces the cost of understanding and changing the application. A small, consistent structure is better than a large directory taxonomy that no one follows.
