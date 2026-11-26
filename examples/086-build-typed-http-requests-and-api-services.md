# Build Typed HTTP Requests and API Services — Examples

These examples build a small typed products API layer. They assume HttpClient has already been configured with provideHttpClient as shown in lesson 085.

## 1. Define request and response types

~~~ts
// product.model.ts
export interface Product {
  id: number;
  name: string;
  price: number;
}

export interface CreateProductRequest {
  name: string;
  price: number;
}

export interface UpdateProductRequest {
  name?: string;
  price?: number;
}
~~~

The API may use different fields or update rules. Keep these types aligned with the actual server contract.

## 2. Build a focused API service

~~~ts
// product-api.service.ts
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import {
  CreateProductRequest,
  Product,
  UpdateProductRequest
} from './product.model';

@Injectable({ providedIn: 'root' })
export class ProductApiService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = '/api/products';

  getAll(): Observable<Product[]> {
    return this.http.get<Product[]>(this.baseUrl);
  }

  getById(id: number): Observable<Product> {
    return this.http.get<Product>(this.baseUrl + '/' + id);
  }

  create(input: CreateProductRequest): Observable<Product> {
    return this.http.post<Product>(this.baseUrl, input);
  }

  replace(id: number, product: Product): Observable<Product> {
    return this.http.put<Product>(this.baseUrl + '/' + id, product);
  }

  update(id: number, changes: UpdateProductRequest): Observable<Product> {
    return this.http.patch<Product>(this.baseUrl + '/' + id, changes);
  }

  remove(id: number): Observable<void> {
    return this.http.delete<void>(this.baseUrl + '/' + id);
  }
}
~~~

The remove method assumes the server returns an empty response body. If the backend returns a confirmation object or another payload, change the generic to that response type.

## 3. Display the collection with the async pipe

Expose an Observable from the component and consume it in the template:

~~~ts
// product-list.component.ts
import { AsyncPipe } from '@angular/common';
import { Component, inject } from '@angular/core';
import { ProductApiService } from './product-api.service';

@Component({
  selector: 'app-product-list',
  standalone: true,
  imports: [AsyncPipe],
  templateUrl: './product-list.component.html'
})
export class ProductListComponent {
  private readonly productApi = inject(ProductApiService);

  readonly products$ = this.productApi.getAll();
}
~~~

~~~html
<!-- product-list.component.html -->
<ul>
  @for (product of products$ | async; track product.id) {
    <li>{{ product.name }} — {{ product.price }}</li>
  } @empty {
    <li>No products are available.</li>
  }
</ul>
~~~

The async pipe subscribes while the view needs the value and releases its subscription when the view is destroyed. For a production page, also represent loading and error states so an empty list is not confused with a request that has not completed or failed.

## 4. Trigger a create operation from a component

A user action can call the API service and handle the result at the feature boundary:

~~~ts
saveProduct(input: CreateProductRequest): void {
  this.productApi.create(input).subscribe({
    next: createdProduct => {
      this.message.set('Created product ' + createdProduct.name);
    },
    error: () => {
      this.message.set('Could not create the product.');
    }
  });
}
~~~

This assumes the component has a writable signal named message and an injected ProductApiService. Keep UI feedback here because this layer knows which action the user took. For long-running or more complex flows, compose the stream deliberately and handle the component lifecycle.

## 5. Avoid accidental duplicate requests

This Observable is cold:

~~~ts
const products$ = this.productApi.getAll();

products$.subscribe(); // Sends one request.
products$.subscribe(); // Sends another independent request.
~~~

If both the component and its template subscribe to the same cold Observable, they can also trigger separate requests. Prefer one intended subscription, or explicitly share/cache the value when multiple consumers need one request result.

## 6. Validate a response when runtime certainty is needed

A type guard can check a simple response shape:

~~~ts
import { Product } from './product.model';

function isProduct(value: unknown): value is Product {
  if (typeof value !== 'object' || value === null) {
    return false;
  }

  const candidate = value as Record<string, unknown>;

  return typeof candidate['id'] === 'number'
    && typeof candidate['name'] === 'string'
    && typeof candidate['price'] === 'number';
}
~~~

Request unknown data, then validate before returning it as Product:

~~~ts
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { map, Observable } from 'rxjs';
import { Product } from './product.model';

@Injectable({ providedIn: 'root' })
export class ValidatedProductApiService {
  private readonly http = inject(HttpClient);

  getById(id: number): Observable<Product> {
    return this.http.get<unknown>('/api/products/' + id).pipe(
      map(value => {
        if (!isProduct(value)) {
          throw new Error('The server returned an invalid product.');
        }

        return value;
      })
    );
  }
}
~~~

Place the type guard in an imported module in a real project. For nested or complex data, a schema validation library can be easier to maintain than handwritten checks. Validation here is about the payload contract; it is not a substitute for server authorization.

## 7. Choose the expected response type

The generic type describes a JSON body:

~~~ts
this.http.get<Product>('/api/products/42');
~~~

For a plain text response, use responseType:

~~~ts
this.http.get('/api/status.txt', {
  responseType: 'text'
});
~~~

For raw bytes:

~~~ts
this.http.get('/assets/manual.pdf', {
  responseType: 'arraybuffer'
});
~~~

When responseType options are moved into a variable, retain the literal type so TypeScript selects the correct overload:

~~~ts
const options = {
  responseType: 'text' as const
};

this.http.get('/api/status.txt', options);
~~~

Detailed response metadata, URL parameters, headers, and credentials are handled in the next lesson.

## 8. Convert one request to a Promise when needed

Most Angular UI code can stay with Observables. If a one-shot asynchronous function needs a Promise, use firstValueFrom:

~~~ts
import { firstValueFrom } from 'rxjs';

async refreshOneProduct(id: number): Promise<Product> {
  return firstValueFrom(this.productApi.getById(id));
}
~~~

HttpClient's body Observable normally emits the response and completes. Do not convert to a Promise just to avoid understanding the Observable lifecycle.

## 9. Quick CRUD map

| User intent | API service method | HttpClient call |
| --- | --- | --- |
| Show products | getAll | get<Product[]>() |
| Open one product | getById | get<Product>() |
| Create a product | create | post<Product>(url, request) |
| Replace a product | replace | put<Product>(url, full resource) |
| Edit selected fields | update | patch<Product>(url, partial request) |
| Delete a product | remove | delete<void>() if there is no response body |

The server contract takes precedence. In particular, match the response type to what the endpoint really returns.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="085-configure-httpclient-with-providehttpclient.md">085 - Configure HttpClient with provideHttpClient</a></td><td><a href="../lessons/086-build-typed-http-requests-and-api-services.md">Lesson</a></td><td><a href="../qnas/086-build-typed-http-requests-and-api-services.md">Q&amp;A</a></td><td><a href="087-url-parameters-headers-response-types-and-credentials.md">087 - URL Parameters, Headers, Response Types, and Credentials</a></td>
</tr>
  </tbody>
</table>
