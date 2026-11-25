# URL Parameters, Headers, Response Types, and Credentials — Examples

These snippets focus on HttpClient request options. They assume the application already provides HttpClient through provideHttpClient. Replace endpoint names and response types with the API contract used by your application.

## 1. Add simple query parameters

For a small fixed set of parameters, use an object:

~~~ts
getProducts(category: string, page: number) {
  return this.http.get<Product[]>('/api/products', {
    params: {
      category,
      page
    }
  });
}
~~~

Angular serializes them into the URL query string. Keep credentials and private values out of query strings.

## 2. Build conditional and repeated parameters

HttpParams returns a new instance from every update:

~~~ts
import { HttpParams } from '@angular/common/http';

searchProducts(query: string, tags: string[], page?: number) {
  let params = new HttpParams()
    .set('q', query)
    .set('sort', 'name');

  if (page !== undefined) {
    params = params.set('page', page);
  }

  for (const tag of tags) {
    params = params.append('tag', tag);
  }

  return this.http.get<Product[]>('/api/products/search', { params });
}
~~~

Repeated calls to append create repeated values for tag. If the server expects a comma-separated list or a JSON value instead, serialize the value according to that API contract.

## 3. Build params with appendAll

When an input object already matches the simple parameter shape, appendAll can add several values at once:

~~~ts
const params = new HttpParams()
  .set('q', 'rain jacket')
  .appendAll({
    tag: ['outdoor', 'clothing'],
    available: true
  });

return this.http.get<Product[]>('/api/products/search', { params });
~~~

Use set for a single replacement value and append when another value for the same key is intended.

## 4. Use a custom parameter encoder only when needed

The default HttpParams codec handles normal URL encoding. A custom codec is appropriate only when a specific backend expects a different convention:

~~~ts
import {
  HttpParameterCodec,
  HttpParams
} from '@angular/common/http';

export class StrictParameterCodec implements HttpParameterCodec {
  encodeKey(key: string): string {
    return encodeURIComponent(key);
  }

  encodeValue(value: string): string {
    return encodeURIComponent(value);
  }

  decodeKey(key: string): string {
    return decodeURIComponent(key);
  }

  decodeValue(value: string): string {
    return decodeURIComponent(value);
  }
}

const params = new HttpParams({
  encoder: new StrictParameterCodec()
})
  .set('email', 'dev+alerts@example.com')
  .set('search', 'a & b?');
~~~

Before introducing a custom codec, confirm the server's actual decoding rules and cover special characters in an API test.

## 5. Set one or two request headers

A plain object is concise for a request-specific header:

~~~ts
return this.http.get<Product[]>('/api/products', {
  headers: {
    'X-Client-Version': '2',
    'Accept-Language': 'en'
  }
});
~~~

For reusable construction, use HttpHeaders:

~~~ts
import { HttpHeaders } from '@angular/common/http';

const sharedHeaders = new HttpHeaders()
  .set('X-Client-Version', '2');

const requestHeaders = sharedHeaders
  .set('Accept-Language', 'en')
  .append('X-Feature', 'catalog');

return this.http.get<Product[]>('/api/products', {
  headers: requestHeaders
});
~~~

HttpHeaders is immutable. Keep the returned value from set or append. Browser-controlled headers and cross-origin CORS policy can limit which headers are actually sent.

## 6. Read a plain-text response

For a text endpoint, set responseType to text:

~~~ts
getStatusText() {
  return this.http.get('/api/status.txt', {
    responseType: 'text'
  });
}
~~~

The returned Observable emits a string.

## 7. Download a Blob or ArrayBuffer

Use Blob for file-like browser data:

~~~ts
downloadReport() {
  return this.http.get('/api/reports/monthly.pdf', {
    responseType: 'blob'
  });
}
~~~

Use ArrayBuffer when code needs the raw bytes:

~~~ts
downloadArchiveBytes() {
  return this.http.get('/api/archive', {
    responseType: 'arraybuffer'
  });
}
~~~

Choose based on the consuming API. Do not use a JSON generic to describe binary data.

## 8. Keep an extracted responseType literal

If options are moved into a constant, preserve the literal type:

~~~ts
const textOptions = {
  responseType: 'text' as const
};

return this.http.get('/api/status.txt', textOptions);
~~~

Without as const, TypeScript can widen the value and lose the overload that returns Observable<string>.

## 9. Read status and headers from the full response

Use observe: 'response' when the caller needs metadata:

~~~ts
import { HttpResponse } from '@angular/common/http';

getProductsWithMetadata() {
  return this.http.get<Product[]>('/api/products', {
    observe: 'response'
  });
}

this.getProductsWithMetadata().subscribe((response: HttpResponse<Product[]>) => {
  console.log('Status:', response.status);
  console.log('ETag:', response.headers.get('ETag'));

  if (response.body !== null) {
    console.log('Products:', response.body);
  }
});
~~~

HttpResponse.body can be null. Handle that possibility when consuming it.

## 10. Store observe options without losing the literal

~~~ts
const fullResponseOptions = {
  observe: 'response' as const
};

return this.http.get<Product[]>('/api/products', fullResponseOptions);
~~~

The result is an Observable of HttpResponse<Product[]> rather than an Observable of Product[].

## 11. Inspect the final response from an event stream

observe: 'events' emits multiple event types. Filter for the final HttpResponse before reading its body:

~~~ts
import {
  HttpEvent,
  HttpEventType,
  HttpResponse
} from '@angular/common/http';
import { filter } from 'rxjs';

const events$ = this.http.get<Product[]>('/api/products', {
  observe: 'events'
});

const finalResponse$ = events$.pipe(
  filter(
    (event): event is HttpResponse<Product[]> =>
      event.type === HttpEventType.Response
  )
);

finalResponse$.subscribe(response => {
  console.log(response.body);
});
~~~

Progress event support depends on the backend and request options. The next lesson covers upload and download progress in detail.

## 12. Use same-origin credentials

For same-origin requests, Fetch's credentials default is same-origin. It can also be set explicitly:

~~~ts
getCurrentUser() {
  return this.http.get<UserProfile>('/api/profile', {
    credentials: 'same-origin'
  });
}
~~~

This allows browser credentials for same-origin requests according to browser cookie policy.

## 13. Include credentials in a cross-origin request

Use include only when the external API is trusted and intentionally uses cookie credentials:

~~~ts
getExternalProfile() {
  return this.http.get<UserProfile>(
    'https://api.example.com/profile',
    { credentials: 'include' }
  );
}
~~~

The server must explicitly allow the requesting origin and credentialed CORS access. Browser cookie policy and attributes must also permit the cookie. Never derive a credentialed destination from arbitrary user-controlled input.

## 14. Omit credentials for a public cross-origin request

~~~ts
getPublicCatalog() {
  return this.http.get<Product[]>(
    'https://public.example.com/catalog',
    { credentials: 'omit' }
  );
}
~~~

This makes the intended no-credentials policy explicit.

## 15. Legacy withCredentials usage

Older code may use withCredentials:

~~~ts
getLegacyProfile() {
  return this.http.get<UserProfile>(
    'https://api.example.com/profile',
    { withCredentials: true }
  );
}
~~~

Do not set both credentials and withCredentials in one request. Current Angular documentation states that withCredentials takes precedence.

## 16. Combine request options

A request can combine query params, headers, a full response, and an explicit credentials policy:

~~~ts
const params = new HttpParams()
  .set('page', 1)
  .set('pageSize', 25);

const headers = new HttpHeaders()
  .set('X-Client-Version', '2');

const options = {
  params,
  headers,
  observe: 'response' as const,
  credentials: 'same-origin' as const
};

return this.http.get<Product[]>('/api/products', options);
~~~

The Observable emits HttpResponse<Product[]>, allowing the caller to inspect both the body and response metadata.

## Quick selection guide

| Need | Use |
| --- | --- |
| A few URL query values | params object |
| Repeated or conditionally built query values | HttpParams |
| A small request-specific header set | headers object |
| Headers built across multiple steps | HttpHeaders |
| JSON response | Generic type with the default responseType |
| Plain text | responseType: 'text' |
| File-like binary | responseType: 'blob' |
| Raw bytes | responseType: 'arraybuffer' |
| Status and response headers | observe: 'response' |
| Request lifecycle events | observe: 'events' |
| Same-origin cookie behavior | credentials: 'same-origin' |
| Cross-origin cookies when explicitly required | credentials: 'include' |
| Do not send browser credentials | credentials: 'omit' |

## Related files and official documentation

- Lesson: lessons/087-url-parameters-headers-response-types-and-credentials.md
- Q&A: qnas/087-url-parameters-headers-response-types-and-credentials.md
- Angular making HTTP requests: https://angular.dev/guide/http/making-requests
- HttpParams API: https://angular.dev/api/common/http/HttpParams
- HttpHeaders API: https://angular.dev/api/common/http/HttpHeaders
- HttpClient API: https://angular.dev/api/common/http/HttpClient
