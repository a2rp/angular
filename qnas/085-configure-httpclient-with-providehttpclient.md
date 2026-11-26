# Configure HttpClient with provideHttpClient - Q&A

## Setup and dependency injection

### 1. What does provideHttpClient do?

provideHttpClient registers Angular's HttpClient and its supporting providers in the current environment injector. It can also accept feature functions that configure behavior such as interceptors, XSRF settings, or the transport backend.

### 2. Where should provideHttpClient go in a standalone application?

Usually in the root ApplicationConfig providers, which is passed to bootstrapApplication. This makes the configured HttpClient available throughout the application unless a child injector defines another configuration.

### 3. Do I need to call provideHttpClient in every service?

No. Register the client at an application or appropriate environment-injector boundary. Services that inject HttpClient use the configuration inherited from that injector.

### 4. Why do I get “No provider for HttpClient”?

The injector that creates the requesting service cannot find HttpClient. Add provideHttpClient to the relevant application providers, or check the version-specific Angular setup. For a test, provide the HTTP testing configuration.

### 5. Is provideHttpClient required in every Angular version?

The current Angular setup guide says HttpClient is available for injection by default in Angular v21 and later. provideHttpClient is still the API for configuring client features and is the explicit setup used in this course. Earlier versions require explicit registration. Check the documentation matching the application's installed Angular version.

### 6. Can an NgModule application use provideHttpClient?

Yes. Register it in the NgModule providers for the application injector. New code should prefer provideHttpClient over the older HttpClientModule setup.

### 7. Does provideHttpClient return a regular Provider?

It returns EnvironmentProviders. That makes it suitable for application and environment injector configuration. Put it in the providers collection accepted by the corresponding application configuration API.

### 8. What is the difference between HttpClient and fetch?

HttpClient is Angular's injectable HTTP abstraction. It integrates with dependency injection, interceptors, typed request methods, and Angular's test utilities. Fetch is one possible transport backend used under HttpClient. Application code can keep using HttpClient while Angular chooses the underlying transport.

## Optional features

### 9. How do I register functional interceptors?

Pass them to withInterceptors inside provideHttpClient:

~~~ts
provideHttpClient(
  withInterceptors([authInterceptor, loggingInterceptor])
)
~~~

The functions run in the configured client pipeline. The order in the array matters. An interceptor that is defined but not passed to withInterceptors is not registered in that chain.

### 10. How do class-based interceptors work with provideHttpClient?

Class-based interceptors registered through HTTP_INTERCEPTORS require withInterceptorsFromDi in the provideHttpClient feature list. Current Angular documentation recommends functional interceptors for new code because their ordering is more predictable.

### 11. Can I configure more than one feature?

Yes. Pass multiple feature functions to provideHttpClient. Keep features at the injector that should own them and avoid duplicating the shared configuration across unrelated injectors.

### 12. What backend does HttpClient use by default?

Current Angular documentation says Fetch is the default backend. withFetch is deprecated because it is no longer needed to select Fetch.

### 13. When would I use withXhr?

Use withXhr when a browser feature requires XMLHttpRequest behavior, such as upload progress events. Fetch does not report upload progress through HttpClient. Angular warns against XHR for server-side rendering; prefer the default Fetch backend there.

### 14. Should I add withFetch to every app?

No. Fetch is already the default in current Angular, and withFetch is deprecated in current API documentation. Verify version-specific guidance for an older application before changing its setup.

### 15. What XSRF protection does the default client provide?

The default configuration includes Angular's built-in XSRF behavior. Applications can customize the cookie and header names with withXsrfConfiguration. Do not disable it merely to work around an API or cookie issue; first understand the client/server configuration.

### 16. What does withNoXsrfProtection do?

It disables Angular's built-in XSRF protection for that client configuration. This is security-sensitive. Use it only when the security design explicitly calls for it and the server has an appropriate alternative.

### 17. Is JSONP a good solution for cross-origin requests?

Prefer regular HTTP requests with correct CORS configuration. Current Angular documentation marks JSONP support as deprecated because JSONP can create cross-site scripting risks.

## Injector scope and existing projects

### 18. What happens if a child injector provides its own HttpClient?

A child-level provideHttpClient configuration is distinct from the parent configuration and, by default, overrides the parent's HttpClient configuration for that injector. This can mean root interceptors do not automatically process the child's requests.

### 19. How can child-level interceptors still use the parent HTTP pipeline?

Configure the child client with its local features and withRequestsMadeViaParent. This forwards requests to an HttpClient configured in an ancestor injector after the child interceptors run. Without a parent client, it is invalid and can cause a runtime error.

### 20. Is HttpClientModule still supported?

Existing applications may contain it, but the current setup guide deprecates HttpClientModule and recommends provideHttpClient with feature functions. Avoid importing the old module in multiple injectors because behavior can be difficult to reason about.

### 21. How do I migrate class-based interceptors from HttpClientModule?

Register provideHttpClient(withInterceptorsFromDi()) and keep the class interceptors registered through HTTP_INTERCEPTORS. For new interceptors, consider converting to functional interceptors and using withInterceptors.

### 22. Can I configure HttpClient in a component's providers?

A component provider creates a local injector boundary, which is usually unnecessary for shared HTTP setup and can produce a separate client configuration. Put application-wide configuration in the root ApplicationConfig. Use local configuration only when the injector boundary is intentional and understood.

## Testing

### 23. How should a service test configure HTTP?

Use provideHttpClientTesting from @angular/common/http/testing. It replaces the real backend with a controllable testing backend and provides HttpTestingController for asserting requests and flushing responses.

### 24. What order should HTTP test providers use?

If the test needs HttpClient features, register provideHttpClient with those features first and provideHttpClientTesting after it. The testing provider replaces parts of the client configuration, so reversing the order can break the intended setup.

### 25. Should an HTTP test call a real server?

Usually no. Mock the backend with provideHttpClientTesting so the test controls requests and responses. Use a separate integration or end-to-end check if the goal is to verify real deployment, CORS, cookies, or server behavior.

## Troubleshooting summary

| Symptom | First check |
| --- | --- |
| No provider for HttpClient | Is HttpClient registered in the requesting service's injector? |
| Interceptor does not run | Is it passed to withInterceptors or explicitly enabled as a DI interceptor? |
| Child feature skips root interceptors | Did the child provide another client? Should it use withRequestsMadeViaParent? |
| Upload progress is missing | Is the default Fetch backend insufficient for this browser upload? |
| Test attempts real network access | Is provideHttpClientTesting configured? |
| XSRF or CORS errors | Verify cookie/header settings and server CORS behavior before disabling protections. |

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="084-preloading-scroll-restoration-and-router-testing.md">084 - 84. Preloading, scroll restoration, and router testing</a></td><td><a href="../lessons/085-configure-httpclient-with-providehttpclient.md">Lesson</a></td><td><a href="../examples/085-configure-httpclient-with-providehttpclient.md">Example</a></td><td><a href="086-build-typed-http-requests-and-api-services.md">086 - Build Typed HTTP Requests and API Services</a></td>
</tr>
  </tbody>
</table>
