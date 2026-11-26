# Configure HttpClient with provideHttpClient - Examples

These examples show provider setup and small, practical configurations for Angular HttpClient. They use standalone APIs and assume the application's Angular version supports the shown APIs. Check the documentation matching your installed version when maintaining an older project.

## 1. Register HttpClient in a standalone application

Create an application configuration with provideHttpClient:

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

Pass that configuration to bootstrapApplication:

~~~ts
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig)
  .catch(error => console.error(error));
~~~

HttpClient can now be injected by root-provided services and other services in the application's injector hierarchy.

## 2. Inject HttpClient into an API service

Keep HTTP calls in an injectable service instead of making a component responsible for transport details:

~~~ts
// catalog.service.ts
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { Observable } from 'rxjs';

export interface Product {
  id: number;
  name: string;
  price: number;
}

@Injectable({ providedIn: 'root' })
export class CatalogService {
  private readonly http = inject(HttpClient);

  getProducts(): Observable<Product[]> {
    return this.http.get<Product[]>('/api/products');
  }

  getProduct(id: number): Observable<Product> {
    return this.http.get<Product>('/api/products/' + id);
  }
}
~~~

The generic type describes the expected response shape for TypeScript. It does not validate untrusted JSON at runtime.

## 3. Add a functional interceptor

A functional interceptor can inspect or clone requests before forwarding them. Register it through withInterceptors:

~~~ts
// request-id.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';

export const requestIdInterceptor: HttpInterceptorFn = (request, next) => {
  const updatedRequest = request.clone({
    setHeaders: {
      'X-Request-Source': 'angular-app'
    }
  });

  return next(updatedRequest);
};
~~~

~~~ts
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import {
  provideHttpClient,
  withInterceptors
} from '@angular/common/http';
import { requestIdInterceptor } from './request-id.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([requestIdInterceptor])
    )
  ]
};
~~~

An interceptor is part of the client chain only when it is included in this configuration. If multiple interceptors are listed, their order matters.

## 4. Customize XSRF cookie and header names

Most applications should keep the defaults. If the server uses different names, configure those names deliberately:

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

The server must set and validate the matching cookie and request header. Changing client names alone does not implement server-side protection.

## 5. Choose XHR for a browser upload flow

HttpClient uses Fetch by default in current Angular. Fetch does not report upload progress. If a browser upload flow needs progress events, configure XHR for that application:

~~~ts
import {
  provideHttpClient,
  withXhr
} from '@angular/common/http';

provideHttpClient(withXhr())
~~~

Use this only when the application needs XHR behavior. Angular recommends the default Fetch backend for server-side rendering; do not use XHR as the server-rendering transport.

## 6. Forward a feature client's requests through the parent

A child environment injector may need a local interceptor while retaining the parent HTTP pipeline:

~~~ts
import {
  provideHttpClient,
  withInterceptors,
  withRequestsMadeViaParent
} from '@angular/common/http';

const featureHttpProviders = provideHttpClient(
  withInterceptors([featureAuditInterceptor]),
  withRequestsMadeViaParent()
);
~~~

Register these providers only at the intended child environment-injector boundary. The parent injector must already have an HttpClient configuration. For a normal feature that does not need a local HTTP pipeline, rely on the root client instead of providing another one.

## 7. Configure an NgModule-based application

An NgModule application can register the modern provider API in its root module:

~~~ts
import { NgModule } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';

@NgModule({
  providers: [
    provideHttpClient()
  ]
})
export class AppModule {}
~~~

Existing applications using class-based interceptors can explicitly include them:

~~~ts
import {
  HTTP_INTERCEPTORS,
  provideHttpClient,
  withInterceptorsFromDi
} from '@angular/common/http';

@NgModule({
  providers: [
    provideHttpClient(withInterceptorsFromDi()),
    {
      provide: HTTP_INTERCEPTORS,
      useClass: LegacyLoggingInterceptor,
      multi: true
    }
  ]
})
export class AppModule {}
~~~

For new interceptors, Angular recommends functional interceptors configured with withInterceptors.

## 8. Test a service with a mock HTTP backend

Provide the client features first and provideHttpClientTesting afterward. The testing provider replaces the real backend and supplies HttpTestingController.

~~~ts
// config.service.ts
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';

export interface AppConfig {
  apiUrl: string;
}

@Injectable({ providedIn: 'root' })
export class ConfigService {
  private readonly http = inject(HttpClient);

  load() {
    return this.http.get<AppConfig>('/assets/config.json');
  }
}
~~~

~~~ts
// config.service.spec.ts
import { TestBed } from '@angular/core/testing';
import { provideHttpClient } from '@angular/common/http';
import {
  HttpTestingController,
  provideHttpClientTesting
} from '@angular/common/http/testing';
import { firstValueFrom } from 'rxjs';
import { ConfigService } from './config.service';

describe('ConfigService', () => {
  let service: ConfigService;
  let httpTesting: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        ConfigService,
        provideHttpClient(),
        provideHttpClientTesting()
      ]
    });

    service = TestBed.inject(ConfigService);
    httpTesting = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpTesting.verify();
  });

  it('requests the configuration and returns the response', async () => {
    const responsePromise = firstValueFrom(service.load());

    const request = httpTesting.expectOne('/assets/config.json');
    expect(request.request.method).toBe('GET');

    request.flush({ apiUrl: '/api' });

    await expectAsync(responsePromise).toBeResolvedTo({
      apiUrl: '/api'
    });
  });
});
~~~

The test uses Jasmine-style matchers. If the project uses Vitest or Jest, adapt the assertion syntax to the configured runner. The provider order is important: provideHttpClientTesting must come after provideHttpClient when both are present.

## 9. Test a configured interceptor without a real network

The HTTP testing backend can inspect the request after configured interceptors have run:

~~~ts
TestBed.configureTestingModule({
  providers: [
    provideHttpClient(
      withInterceptors([requestIdInterceptor])
    ),
    provideHttpClientTesting()
  ]
});

const http = TestBed.inject(HttpClient);
const httpTesting = TestBed.inject(HttpTestingController);

http.get('/api/status').subscribe();

const request = httpTesting.expectOne('/api/status');
expect(request.request.headers.get('X-Request-Source')).toBe('angular-app');

request.flush({ ok: true });
httpTesting.verify();
~~~

Import HttpClient, withInterceptors, requestIdInterceptor, and the test APIs in the actual spec file. The test backend captures requests locally; it does not contact the API server.

## 10. Setup decision table

| Requirement | Configuration |
| --- | --- |
| Basic standalone application | provideHttpClient() in root ApplicationConfig |
| Functional interceptors | provideHttpClient(withInterceptors([...])) |
| Existing class-based interceptors | withInterceptorsFromDi() plus HTTP_INTERCEPTORS providers |
| Custom XSRF cookie or header names | withXsrfConfiguration(...) |
| Upload progress through XHR | withXhr(), for the browser use case |
| Child interceptor plus parent HTTP chain | withRequestsMadeViaParent() in the child injector |
| Mock requests in tests | provideHttpClientTesting() after provideHttpClient(...) |

## Troubleshooting checklist

- Injection fails: verify the right application or test injector has HttpClient providers.
- Interceptor does not run: verify it is passed to withInterceptors, or enable DI-based interceptors explicitly.
- A child feature skips root configuration: check for a local HttpClient provider and decide whether parent forwarding is intended.
- Tests use a real backend: provideHttpClientTesting and assert that all expected requests were handled.
- Upload progress is unavailable: Fetch does not report upload progress; evaluate the XHR backend for browser-only work.
- Cross-origin requests fail: configure server CORS and cookie behavior; do not disable XSRF protection as a generic fix.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="084-preloading-scroll-restoration-and-router-testing.md">084 - 84. Preloading, scroll restoration, and router testing</a></td><td><a href="../lessons/085-configure-httpclient-with-providehttpclient.md">Lesson</a></td><td><a href="../qnas/085-configure-httpclient-with-providehttpclient.md">Q&amp;A</a></td><td><a href="086-build-typed-http-requests-and-api-services.md">086 - Build Typed HTTP Requests and API Services</a></td>
</tr>
  </tbody>
</table>
