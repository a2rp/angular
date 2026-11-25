# Configure HttpClient with provideHttpClient

Angular's HttpClient is the framework service for making HTTP requests. Before an application service can inject and use it, the relevant dependency-injection environment must provide it. In standalone applications, the standard provider API is provideHttpClient from @angular/common/http.

This lesson focuses on registering and configuring HttpClient. Request methods, typed API services, interceptors in depth, error handling, and HTTP testing are covered in later lessons.

## Learning outcomes

After this lesson, you should be able to:

- Register HttpClient in a standalone application's provider configuration.
- Explain what provideHttpClient returns and where it belongs.
- Recognize common optional HTTP features.
- Choose the browser transport intentionally.
- Explain why provider location matters in applications with child injectors.
- Recognize the older HttpClientModule setup and when to migrate it.

## 1. HttpClient and dependency injection

HttpClient is an injectable service. A service that depends on it can request it through Angular's dependency injection:

~~~ts
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class CatalogService {
  private readonly http = inject(HttpClient);
}
~~~

The injection only succeeds when HttpClient is available in the injector hierarchy where CatalogService is created. An error such as “No provider for HttpClient” usually means the application or test environment has not registered the required HTTP providers.

## 2. Register HttpClient in a standalone application

A standalone application commonly stores application-wide providers in app.config.ts:

~~~ts
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient()
  ]
};
~~~

The application bootstrap uses that configuration:

~~~ts
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch(error => console.error(error));
~~~

provideHttpClient returns environment providers intended for an application or environment injector. It can be called without arguments to register the default HttpClient configuration, or with feature functions to add supported behavior.

### Version note

Angular's current setup guide says HttpClient is available for injection by default in Angular v21 and later. Explicitly using provideHttpClient remains the API for configuring its features and is the clear setup to use when following this course. For earlier Angular versions, register it explicitly as shown above. When changing an existing project, follow the setup guidance for that project's Angular version.

## 3. Configure HTTP features

Features are passed to provideHttpClient as arguments. For example, functional interceptors can be registered as part of the client configuration:

~~~ts
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './auth.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor])
    )
  ]
};
~~~

The interceptor is a separate function configured into the client. Registering or importing a function by itself does not add it to the HTTP chain. A later lesson covers how interceptors transform requests and responses.

Other feature functions configure concerns such as XSRF behavior, JSONP support, XHR transport, and forwarding requests through a parent injector. Add only features the application needs.

## 4. Default transport and XHR

The current HttpClient setup uses the Fetch API by default. This is generally recommended, including for server-side rendering. Fetch does not provide upload progress events.

If browser upload progress is required, the application may choose the XHR backend:

~~~ts
import { provideHttpClient, withXhr } from '@angular/common/http';

provideHttpClient(withXhr())
~~~

Use withXhr only when its capability is needed. Angular's setup guide warns against XHR in server-side rendering environments and recommends Fetch there. In current Angular, withFetch is deprecated because Fetch is already the default; do not add it just to select the default transport.

## 5. XSRF protection

The default HttpClient configuration includes Angular's built-in XSRF protection behavior for supported same-origin requests. It uses the standard XSRF cookie and header names unless configured otherwise.

When an application needs different names, configure them explicitly:

~~~ts
import {
  provideHttpClient,
  withXsrfConfiguration
} from '@angular/common/http';

provideHttpClient(
  withXsrfConfiguration({
    cookieName: 'APP-XSRF',
    headerName: 'X-APP-XSRF'
  })
)
~~~

Disabling XSRF protection is a security-sensitive decision. Do not use withNoXsrfProtection as a generic way to silence a request or server configuration problem. Understand the backend's protections and the application's threat model before changing this behavior.

## 6. Provider scope and child injectors

A root-level provideHttpClient configuration is appropriate for most applications. A feature that creates a child environment injector can also register its own HTTP configuration, but that local client configuration does not automatically inherit the parent's features.

To configure child-level interceptors and then pass requests through the parent HttpClient, Angular provides withRequestsMadeViaParent:

~~~ts
provideHttpClient(
  withInterceptors([featureLoggingInterceptor]),
  withRequestsMadeViaParent()
)
~~~

This feature requires an HttpClient configured in an ancestor injector. It is useful for deliberate hierarchical configuration, not for ordinary root setup. Accidental duplicate configurations can lead to confusing interceptor behavior, so decide which injector owns the shared HTTP pipeline.

## 7. NgModule-based applications

An application bootstrapped with NgModules can register the provider in its application module:

~~~ts
@NgModule({
  providers: [
    provideHttpClient()
  ]
})
export class AppModule {}
~~~

Older projects may instead import HttpClientModule. For new configuration, prefer provideHttpClient and feature functions. Avoid importing the legacy module in multiple injectors; doing so can make interceptor behavior difficult to reason about. When migrating, account for class-based interceptors explicitly with withInterceptorsFromDi if the application still uses them.

## 8. JSONP and legacy configuration

JSONP support is deprecated in current Angular documentation because JSONP can introduce cross-site scripting risks. Prefer normal HTTP requests with correct server-side CORS configuration. Do not add withJsonpSupport unless maintaining a legacy integration that specifically requires it and the risk is understood.

The older HttpClientModule, HttpClientJsonpModule, and HttpClientXsrfModule APIs are deprecated in the current setup guide. Use provideHttpClient and the matching feature function for new configurations.

## 9. Common setup mistakes

### HttpClient cannot be injected

Confirm provideHttpClient is registered in the application's environment providers, or that the current Angular version provides it by default. In tests, configure the test HTTP providers.

### A configured interceptor never runs

Confirm the interceptor is passed to withInterceptors inside provideHttpClient. A functional interceptor that is merely declared or provided elsewhere will not enter the HttpClient chain by itself.

### A child feature does not use the root interceptors

Check whether the feature created its own HttpClient configuration. If requests should traverse the parent pipeline, configure the child with withRequestsMadeViaParent and make sure a parent HttpClient exists.

### Upload progress is missing

Fetch does not support upload progress reporting through HttpClient. Consider withXhr for a browser-only upload flow, and keep server-side rendering on the recommended Fetch backend.

### Tests accidentally use a real backend

Use provideHttpClientTesting in the test environment. If test interceptors are needed, register provideHttpClient with its features before provideHttpClientTesting so the testing provider can replace the backend correctly.

## 10. A practical configuration checklist

Before moving on, confirm:

1. The project Angular version and bootstrap style are known.
2. HttpClient is available in the relevant injector.
3. Application-wide features are registered once at the intended scope.
4. Interceptors are explicitly added to the HTTP client.
5. The selected transport supports the application's requirements.
6. The default XSRF behavior has not been disabled casually.
7. Tests provide the HTTP testing backend instead of relying on a real server.

## Key idea

provideHttpClient configures HttpClient through Angular dependency injection. Start with the default configuration, then add feature functions for specific requirements. Keep the provider at the correct injector level and make transport or security changes deliberately.

## Related references

- Q&A: qnas/085-configure-httpclient-with-providehttpclient.md
- Examples: examples/085-configure-httpclient-with-providehttpclient.md
- Angular HttpClient setup: https://angular.dev/guide/http/setup
- provideHttpClient API: https://angular.dev/api/common/http/provideHttpClient
- Angular interceptors: https://angular.dev/guide/http/interceptors
- Angular HTTP testing: https://angular.dev/guide/http/testing
