# Feature-Based Application Architecture — Examples

This walkthrough organizes a small order workflow by feature. It shows the route entry point, feature-owned data access, a page, and the point where code becomes reusable. Adapt names and paths to the Angular workspace you created.

## Goal

Build an order list at the URL /orders. Keep order-specific code under one feature folder, then identify what belongs in application-wide infrastructure and what might be shared.

## 1. Start with the feature boundary

Use a structure like this:

~~~text
src/app/
  app.config.ts
  app.routes.ts
  features/
    orders/
      data-access/
        order.model.ts
        orders-api.service.ts
      pages/
        order-list-page/
          order-list-page.ts
          order-list-page.html
      orders.routes.ts
    inventory/
      pages/
      data-access/
~~~

The orders route, page, API service, and model stay together. Do not add empty folders for every possible file type. Create a subfolder when it gives a real concept a clear home.

## 2. Configure application-wide routing and HTTP

The root route table delegates the orders URL to the feature's route table.

~~~ts
// src/app/app.routes.ts
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

~~~ts
// src/app/app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient()
  ]
};
~~~

The feature has its own route definition:

~~~ts
// src/app/features/orders/orders.routes.ts
import { Routes } from '@angular/router';

export const ORDER_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () =>
      import('./pages/order-list-page/order-list-page')
        .then((module) => module.OrderListPage)
  }
];
~~~

The app route does not import the order page implementation directly. The feature route is a small entry point that owns navigation inside this feature.

## 3. Define the feature's data contract

~~~ts
// src/app/features/orders/data-access/order.model.ts
export interface Order {
  id: string;
  customerName: string;
  totalCents: number;
  status: 'pending' | 'paid' | 'shipped';
}
~~~

Keep a model near its feature if other independent areas do not need it. If inventory later needs a stable order summary, expose a small shared contract instead of importing private implementation details from the orders page folder.

## 4. Keep HTTP details in feature data access

~~~ts
// src/app/features/orders/data-access/orders-api.service.ts
import { HttpClient } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { Observable } from 'rxjs';
import { Order } from './order.model';

@Injectable({ providedIn: 'root' })
export class OrdersApiService {
  private readonly http = inject(HttpClient);

  list(): Observable<Order[]> {
    return this.http.get<Order[]>('/api/orders');
  }
}
~~~

This service owns the endpoint and response type. The page can request orders without building the URL or using HttpClient directly. If the API service becomes feature-scoped state, provide it at the route or component instead of root; choose lifetime based on intended sharing.

## 5. Create a focused page

~~~ts
// src/app/features/orders/pages/order-list-page/order-list-page.ts
import { DatePipe } from '@angular/common';
import { Component, inject } from '@angular/core';
import { RouterLink } from '@angular/router';
import { catchError, of } from 'rxjs';
import { toSignal } from '@angular/core/rxjs-interop';
import { Order } from '../../data-access/order.model';
import { OrdersApiService } from '../../data-access/orders-api.service';

@Component({
  selector: 'app-order-list-page',
  imports: [DatePipe, RouterLink],
  templateUrl: './order-list-page.html'
})
export class OrderListPage {
  private readonly ordersApi = inject(OrdersApiService);

  readonly orders = toSignal(
    this.ordersApi.list().pipe(
      catchError(() => of([] as Order[]))
    ),
    { initialValue: [] as Order[] }
  );
}
~~~

~~~html
<!-- src/app/features/orders/pages/order-list-page/order-list-page.html -->
<main>
  <h1>Orders</h1>

  @if (orders().length === 0) {
    <p>No orders are available.</p>
  } @else {
    <ul>
      @for (order of orders(); track order.id) {
        <li>
          <a [routerLink]="['/orders', order.id]">
            {{ order.customerName }} — {{ order.status }}
          </a>
          <span>{{ order.totalCents / 100 | currency }}</span>
        </li>
      }
    </ul>
  }
</main>
~~~

In a production page, model loading, empty, and failure states separately so an HTTP error is not presented as a confirmed empty result. The shortened code focuses on file ownership and route boundaries.

## 6. Decide whether a component is shared

Suppose the order list uses an order status badge, and the order detail page will use the same badge. Keep it in orders/ui because both uses are still order-specific.

If an independent inventory feature also needs a generic visual badge with the same stable behavior, consider moving a presentation-only component to shared/ui. Keep the shared API small. Do not move order-specific rules into a global badge just to avoid a second small component.

## 7. Inspect dependency direction

A reasonable dependency direction is:

~~~text
app.routes.ts
    |
    v
orders.routes.ts
    |
    v
order-list-page.ts
    |
    v
orders-api.service.ts -> order.model.ts
~~~

Avoid having OrdersApiService import the page component, or having two feature folders import one another's internal page files. If two features need to participate in a workflow, let a parent route compose them or define a narrow shared contract.

## 8. Refactor a type-based structure incrementally

If the existing source looks like this:

~~~text
src/app/
  components/
  services/
  models/
~~~

Move one cohesive workflow at a time:

1. Identify the routes and files that belong to one user capability.
2. Create that feature folder.
3. Move its page, specific UI, data access, and model together.
4. Update imports and route entry points.
5. Move only genuinely cross-feature code into shared or core.
6. Remove old directories only after all imports point to the new owners.

A gradual move is easier to review than a large folder shuffle. Keep each change independently understandable and verify it with the project's usual build and behavior checks.

## 9. Practice: add order details

Extend the orders feature with a detail route. Keep the detail page in the feature, load it by its order ID, and decide which information is part of the stable order model. If another feature needs a value, expose a narrow type or API instead of importing the details page.

## Further reading

- [Angular style guide](https://angular.dev/style-guide)
- [Workspace and project file structure](https://angular.dev/reference/configs/file-structure)
- [Angular routing guide](https://angular.dev/guide/routing)
