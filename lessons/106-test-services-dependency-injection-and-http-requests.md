# Test Services, Dependency Injection, and HTTP Requests

Angular services often coordinate business rules, injected dependencies, and remote data. Tests can verify their behavior without contacting a real server. TestBed creates an injection environment; HttpTestingController lets a test inspect requests and supply controlled responses.

## Learning outcomes

After this lesson, you should be able to:

- Test a service through Angular dependency injection.
- Provide a fake dependency when a service collaborates with another service.
- Configure HttpClient's testing backend.
- Assert request method, URL, query parameters, and headers.
- Flush successful and failed responses.
- Verify that no unexpected requests remain.
- Test interceptors separately from feature behavior.

## 1. Test the service through TestBed

TestBed provides the injector used in the test. Register the service and its dependencies in the test configuration, then retrieve the same instance the injector would supply.

~~~ts
TestBed.configureTestingModule({
  providers: [OrderSummaryService]
});

const service = TestBed.inject(OrderSummaryService);
~~~

Use a fake provider when the service depends on an external API or a collaborator whose behavior is not part of this test. Keep the fake contract aligned with what the service actually uses.

## 2. Replace the real HTTP backend

Do not let unit tests call production or development APIs. Provide HttpClient's testing backend:

~~~ts
providers: [
  provideHttpClient(),
  provideHttpClientTesting()
]
~~~

The order matters: configure HttpClient first, then install the testing backend. Inject HttpTestingController to capture outgoing requests and complete them with controlled responses.

## 3. Assert the request contract

A request contract may include method, URL, query parameters, body, and headers. Match the behavior the service promises rather than relying only on an exact string if query ordering can differ.

~~~ts
const response = firstValueFrom(service.listOrders());
const request = httpTesting.expectOne({
  method: 'GET',
  url: '/api/orders'
});

request.flush(testOrders);
expect(await response).toEqual(testOrders);
~~~

Call flush to deliver a success response. If the observable is not subscribed, the request might not be sent. HttpClient observables are lazy.

## 4. Test error responses

Use the testing request's error or status response APIs to simulate server failures. Then assert that the service propagates, maps, or recovers from the error according to its documented contract.

A service should not silently convert every failure into empty data unless that is the intended behavior. The UI often needs to distinguish an empty result from a failed request.

## 5. Verify each request and clean up

At the end of a test, call HttpTestingController.verify() to detect requests that were never expected or completed. This catches forgotten flushes and accidental extra requests.

Use expectOne for a request expected exactly once. Use match when the behavior intentionally makes multiple requests. Avoid broad matches that let an incorrect method or URL pass unnoticed.

## 6. Test dependency injection and overrides

Use providers to replace collaborators in a focused unit test:

~~~ts
providers: [
  OrderSummaryService,
  { provide: OrdersApi, useValue: fakeOrdersApi }
]
~~~

Verify behavior through the service's public API. Do not inspect TestBed's internal provider registry. If provider scope matters, configure the same scope or component context that the application uses.

## 7. Test interceptors separately where possible

An interceptor can add headers, transform a request, or handle a response. Test that contract using a focused HTTP testing setup. For functional interceptors, register them with provideHttpClient(withInterceptors([...])) and then provide the testing backend. Avoid testing every interceptor detail through unrelated feature tests.

## 8. Keep asynchronous tests clear

HTTP test requests do not resolve until the test flushes a response or error. Create and subscribe to the request, capture it, flush it, then await the result. This makes the sequence explicit and avoids arbitrary timer delays.

For promise-returning code, await the promise. For observables, use firstValueFrom or the project pattern. Ensure the test does not leave subscriptions, requests, or fake timers behind.

## 9. Know what HTTP unit tests do not prove

HttpTestingController verifies how Angular's client code constructs and processes requests. It does not prove that the real server accepts the request, that production credentials work, or that the endpoint contract matches the backend. Use integration or end-to-end coverage for those boundaries.

See Angular's [HTTP testing guide](https://angular.dev/guide/http/testing), [testing services guide](https://angular.dev/guide/testing/services), and [HttpClient setup guide](https://angular.dev/guide/http/setup).
