# Functional Interceptors and Request Context

Angular HttpClient interceptors are middleware functions that can inspect or transform requests and response event streams. They let an application apply shared behavior such as authentication headers, logging, or loading indicators without repeating that logic in every service.

HttpContext provides typed metadata for an individual request. An interceptor can use that metadata to opt a request in or out of shared behavior. Context values stay inside the client pipeline; they are not sent as HTTP headers or body fields.

## Learning outcomes

After this lesson, you should be able to:

- Define a functional HttpInterceptorFn.
- Register interceptors with provideHttpClient and withInterceptors.
- Clone an immutable request to change its headers or options.
- Read and update response event streams.
- Use inject in the interceptor's injection context.
- Create HttpContextToken values for request-specific interceptor behavior.
- Avoid leaking authentication data or mutating request bodies unsafely.

## 1. What an interceptor does

An interceptor receives an HttpRequest and a next function. It can forward the request, return a transformed response stream, or, in advanced cases, produce a response without forwarding to the backend.

A normal interceptor calls next:

~~~ts
import {
  HttpEvent,
  HttpInterceptorFn
} from '@angular/common/http';
import { Observable } from 'rxjs';

export const loggingInterceptor: HttpInterceptorFn = (
  req,
  next
): Observable<HttpEvent<unknown>> => {
  console.log('Request:', req.method, req.url);
  return next(req);
};
~~~

An interceptor must be registered with the HttpClient configuration to run. Merely defining the function does not add it to the request pipeline.

## 2. Register functional interceptors

Use withInterceptors inside provideHttpClient:

~~~ts
import { ApplicationConfig } from '@angular/core';
import {
  provideHttpClient,
  withInterceptors
} from '@angular/common/http';
import { loggingInterceptor } from './logging.interceptor';
import { authInterceptor } from './auth.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([
        authInterceptor,
        loggingInterceptor
      ])
    )
  ]
};
~~~

For outgoing requests, interceptors process requests in the configured order. Each interceptor can pass its request to the next interceptor. Response events return through the chain in the opposite direction as each returned Observable is transformed.

Functional interceptors are recommended for predictable ordering, particularly in complex provider and injector setups.

## 3. Requests are immutable: clone to change them

HttpRequest and HttpHeaders are immutable. To add or replace a header, clone the request:

~~~ts
export const clientVersionInterceptor: HttpInterceptorFn = (req, next) => {
  const updatedRequest = req.clone({
    setHeaders: {
      'X-Client-Version': '2'
    }
  });

  return next(updatedRequest);
};
~~~

The clone method creates a new request with the specified changes. Using setHeaders replaces a header of the same name rather than appending a duplicate value.

Request bodies are not deeply immutable. If an interceptor mutates an object body, retries or repeated processing can observe the mutation. Prefer creating a modified body value rather than changing an existing object in place.

## 4. Use dependency injection inside an interceptor

Functional interceptors run in the injection context of the injector that registered them. They can use inject to access a service:

~~~ts
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.accessToken();

  if (!token) {
    return next(req);
  }

  const authenticatedRequest = req.clone({
    setHeaders: {
      Authorization: 'Bearer ' + token
    }
  });

  return next(authenticatedRequest);
};
~~~

Call inject while the interceptor function is executing, then capture the injected service or value before asynchronous callbacks. Do not assume inject can be called later from an arbitrary timer or Observable callback.

### Send credentials only to the intended API

A global interceptor should not attach a bearer token to every URL. Restrict authentication headers to the intended API origin or path, and never let user-controlled URLs cause credentials to be sent to an untrusted destination. The example above illustrates cloning mechanics; production auth interceptors should include an explicit destination check.

## 5. Transform response event streams

The Observable returned by next carries HttpEvents. Use RxJS operators to observe or transform those events:

~~~ts
import {
  HttpEvent,
  HttpEventType,
  HttpInterceptorFn
} from '@angular/common/http';
import { tap } from 'rxjs';

export const responseLogInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    tap((event: HttpEvent<unknown>) => {
      if (event.type === HttpEventType.Response) {
        console.log('Response status:', event.status);
      }
    })
  );
};
~~~

An event stream can include Sent, progress events, response headers, the final Response, and custom User events. Check the event type before using response-specific fields.

An interceptor can also transform or replace the response stream. Short-circuiting the chain or synthesizing a response is an advanced pattern; if the interceptor does not call next, later interceptors and the backend will not process that request.

## 6. HttpContext and HttpContextToken

HttpContext is a typed map attached to an outgoing request. It carries metadata for interceptors and other client-side request logic. It is not part of the HTTP wire request.

Create a token with a default-value factory:

~~~ts
import { HttpContextToken } from '@angular/common/http';

export const SKIP_GLOBAL_LOADING =
  new HttpContextToken<boolean>(() => false);
~~~

The factory supplies the default for requests that have no explicit value. A factory is important for object or array defaults because each request should receive its own default instance.

An interceptor reads the typed value from the request context:

~~~ts
export const loadingInterceptor: HttpInterceptorFn = (req, next) => {
  if (req.context.get(SKIP_GLOBAL_LOADING)) {
    return next(req);
  }

  const loading = inject(LoadingIndicatorService);
  loading.begin();

  return next(req).pipe(
    finalize(() => loading.end())
  );
};
~~~

The caller sets the token when constructing a request:

~~~ts
const context = new HttpContext()
  .set(SKIP_GLOBAL_LOADING, true);

return this.http.get<BackgroundJob>('/api/jobs/status', {
  context
});
~~~

This request still goes to the server normally. The context value only tells interceptors how to treat it.

## 7. Context is mutable and shared across clones

Unlike most request properties, HttpContext is mutable. Cloned requests share the same context unless a different context is supplied. If an interceptor changes context and the request is retried, a later pass can observe that change.

Use context deliberately:

- Prefer booleans or small values that express a clear policy.
- Do not store secrets or request body data in context.
- Remember context values are client-side metadata, not server authorization.
- Avoid mutating context as a hidden way to coordinate unrelated interceptors.

## 8. Interceptor ordering

Order affects shared behavior. For example:

~~~ts
withInterceptors([
  authInterceptor,
  loggingInterceptor
])
~~~

The outgoing request reaches authInterceptor first, then loggingInterceptor, then the backend. The response event stream returns through the chain in reverse order.

Choose and document an order when interceptors depend on one another. Keep each interceptor focused so its effects remain understandable.

## 9. Common mistakes

### Defining but not registering an interceptor

Use withInterceptors([...]) in provideHttpClient. A standalone function is not active until configured.

### Mutating a request directly

Clone it and set changed values on the clone. Request and header objects are immutable.

### Attaching an auth token to every URL

Check the destination before adding credentials. Global interceptors must not leak secrets to third-party or user-controlled endpoints.

### Calling inject after an asynchronous boundary

Resolve dependencies synchronously during interceptor execution and capture them for later callbacks.

### Treating context as an HTTP header

HttpContext is not serialized or sent to the server. Use a real header only when the server needs that information.

### Mutating the request body in place

The body can be a mutable object. Create a modified body rather than changing shared data, especially when retry behavior may re-run the interceptor.

### Assuming the interceptor runs once per Observable

HttpClient Observables are cold. Each subscription sends a request and runs the configured interceptor chain again.

## Key idea

Functional interceptors centralize cross-cutting HTTP behavior. Register them explicitly, clone immutable requests to make changes, and use typed HttpContext metadata for per-request client-side policies.

## Related references

- Q&A: qnas/089-functional-interceptors-and-request-context.md
- Examples: examples/089-functional-interceptors-and-request-context.md
- Angular interceptors: https://angular.dev/guide/http/interceptors
- withInterceptors API: https://angular.dev/api/common/http/withInterceptors
- HttpContext API: https://angular.dev/api/common/http/HttpContext
- HttpContextToken API: https://angular.dev/api/common/http/HttpContextToken
