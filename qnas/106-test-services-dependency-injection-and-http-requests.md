# Test Services, Dependency Injection, and HTTP Requests - Q&A

## 1. Why use TestBed for a service test?

TestBed creates an Angular injection environment. It is useful when the service depends on Angular DI, providers, or other framework services. A pure function or class without Angular dependencies can usually be tested directly.

## 2. How do I test an HttpClient service without a real network request?

Configure the test with provideHttpClient() and provideHttpClientTesting(), then use HttpTestingController to capture and complete requests. The testing backend prevents the test from calling a real endpoint.

## 3. In which order should the HTTP test providers be registered?

Register provideHttpClient() before provideHttpClientTesting(). The latter replaces the backend configured by the former with the testing backend.

## 4. Why did my request not appear in HttpTestingController?

HttpClient observables are lazy. A request is sent when the observable is subscribed to. Subscribe or convert it to a promise before expecting the request.

## 5. What does request.flush do?

It delivers a controlled response to the captured request. Use it to test success payload handling or to provide an error response with a status and status text.

## 6. How do I test a network-level error?

Use the testing request's error method with a ProgressEvent or the pattern supported by the Angular version. Then assert how the observable or service handles that failure.

## 7. What does HttpTestingController.verify check?

It verifies that no unmatched or outstanding requests remain. Call it during cleanup so forgotten flushes or accidental extra requests fail the test.

## 8. When should I use expectOne and when match?

Use expectOne when exactly one request should match a URL or request predicate. Use match when multiple requests are expected and the test needs to inspect them. Avoid loose matches that ignore the method or important request details.

## 9. Should I compare the entire URL string?

Only if the exact URL is part of the contract. For query parameters, matching by method and URL or inspecting HttpParams may be more robust than depending on query order.

## 10. How should I test a service with another service dependency?

Provide a fake or spy implementation for that dependency when its behavior is outside the current test. Keep the fake's methods and return types aligned with the real contract and assert on the service's public behavior.

## 11. Should every interceptor be tested inside every service test?

No. Test important interceptor contracts in focused tests. A service test can verify its own request and response behavior without duplicating all global interceptor assertions.

## 12. Does an HttpTestingController test prove that the backend API works?

No. It verifies the client-side request construction and response handling. It does not contact the server or validate the deployed endpoint. Use integration or end-to-end checks for that boundary.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="105-component-tests-fixtures-and-template-behavior.md">105 - Component Tests, Fixtures, and Template Behavior</a></td><td><a href="../lessons/106-test-services-dependency-injection-and-http-requests.md">Lesson</a></td><td><a href="../examples/106-test-services-dependency-injection-and-http-requests.md">Example</a></td><td><a href="107-test-routing-forms-accessibility-and-user-flows.md">107 - Test Routing, Forms, Accessibility, and User Flows</a></td>
</tr>
  </tbody>
</table>
