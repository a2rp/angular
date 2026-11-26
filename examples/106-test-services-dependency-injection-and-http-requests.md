# Test Services, Dependency Injection, and HTTP Requests - Examples

This example tests an Angular service without a real backend. The test captures the request, checks its method, parameters, and interceptor-added header, then supplies controlled responses.

## 1. Define the service and request model

~~~ts
// orders-api.ts
import { HttpClient, HttpParams } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { Observable } from 'rxjs';

export interface Order {
  id: string;
  status: 'open' | 'closed';
}

@Injectable({ providedIn: 'root' })
export class OrdersApi {
  private readonly http = inject(HttpClient);

  list(status: Order['status']): Observable<Order[]> {
    const params = new HttpParams().set('status', status);
    return this.http.get<Order[]>('/api/orders', { params });
  }
}
~~~

## 2. Define a small functional interceptor

~~~ts
// app-version.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';

export const appVersionInterceptor: HttpInterceptorFn = (request, next) =>
  next(
    request.clone({
      setHeaders: { 'X-App-Version': 'demo' }
    })
  );
~~~

The interceptor adds one header to demonstrate how the test can inspect the request that reaches the testing backend. A real application may use a different value source.

## 3. Configure TestBed with the HTTP testing backend

~~~ts
// orders-api.spec.ts
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import {
  HttpTestingController,
  provideHttpClientTesting
} from '@angular/common/http/testing';
import { TestBed } from '@angular/core/testing';
import { afterEach, beforeEach, describe, expect, it } from 'vitest';
import { appVersionInterceptor } from './app-version.interceptor';
import { Order, OrdersApi } from './orders-api';

describe('OrdersApi', () => {
  let api: OrdersApi;
  let httpTesting: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        provideHttpClient(withInterceptors([appVersionInterceptor])),
        provideHttpClientTesting()
      ]
    });

    api = TestBed.inject(OrdersApi);
    httpTesting = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpTesting.verify();
  });
~~~

Provide HttpClient before provideHttpClientTesting so the test backend replaces the real backend. No network call leaves the test.

## 4. Verify the request and flush a successful response

~~~ts
  it('requests open orders and returns the response body', async () => {
    const expected: Order[] = [{ id: 'order-17', status: 'open' }];
    const resultPromise = firstValueFrom(api.list('open'));

    const request = httpTesting.expectOne(
      (candidate) =>
        candidate.method === 'GET' &&
        candidate.url === '/api/orders' &&
        candidate.params.get('status') === 'open'
    );

    expect(request.request.headers.get('X-App-Version')).toBe('demo');

    request.flush(expected);

    await expect(resultPromise).resolves.toEqual(expected);
  });
~~~

Import firstValueFrom for this test:

~~~ts
import { firstValueFrom } from 'rxjs';
~~~

Matching the URL, method, and query parameter checks the request contract without relying on query-string ordering.

## 5. Verify server failures

~~~ts
  it('propagates an HTTP error response', async () => {
    const resultPromise = firstValueFrom(api.list('closed'));

    const request = httpTesting.expectOne(
      (candidate) =>
        candidate.method === 'GET' &&
        candidate.url === '/api/orders' &&
        candidate.params.get('status') === 'closed'
    );

    request.flush(
      { message: 'Unavailable' },
      { status: 503, statusText: 'Service Unavailable' }
    );

    await expect(resultPromise).rejects.toMatchObject({
      status: 503
    });
  });
});
~~~

Together, the snippets form one spec file: keep the imports with the first block, then place both test cases inside the describe block before its closing brace.

## 6. Test a service dependency with a fake

For a non-HTTP collaborator, provide a value whose API matches the real service contract:

~~~ts
const fakeClock = {
  now: () => new Date('2026-10-18T12:00:00Z')
};

TestBed.configureTestingModule({
  providers: [
    ReminderService,
    { provide: Clock, useValue: fakeClock }
  ]
});
~~~

Use fakes only for collaborators outside the behavior being tested. A mock that reproduces the implementation instead of the contract can make a broken service appear correct.

## 7. Clean up and run the suite

HttpTestingController.verify in afterEach detects unexpected or unflushed requests. Each test should subscribe to its own HTTP observable and flush the request it creates.

Run the project's configured test command with ng test. This document shows test code for learning; adapt imports and runner setup to the Angular and test-runner versions in your workspace.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="105-component-tests-fixtures-and-template-behavior.md">105 - Component Tests, Fixtures, and Template Behavior</a></td><td><a href="../lessons/106-test-services-dependency-injection-and-http-requests.md">Lesson</a></td><td><a href="../qnas/106-test-services-dependency-injection-and-http-requests.md">Q&amp;A</a></td><td><a href="107-test-routing-forms-accessibility-and-user-flows.md">107 - Test Routing, Forms, Accessibility, and User Flows</a></td>
</tr>
  </tbody>
</table>
