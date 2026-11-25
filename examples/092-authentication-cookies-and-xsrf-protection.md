# Authentication, Cookies, and XSRF Protection: Examples

These examples show a same-origin browser session flow. The backend sets and validates the cookies and CSRF token; Angular sends requests through HttpClient and uses its built-in XSRF support.

## 1. Cookie contract for the browser

A same-origin application might receive two separate Set-Cookie headers:

~~~http
Set-Cookie: SESSION=opaque-session-id; Path=/; Secure; HttpOnly; SameSite=Lax
Set-Cookie: XSRF-TOKEN=server-generated-token; Path=/; Secure; SameSite=Lax
~~~

The SESSION cookie is an authentication credential and is HttpOnly. The XSRF-TOKEN cookie is readable by the Angular application so HttpClient can copy it into the configured request header.

These headers are illustrative. The backend must generate a token valid for the user's session, select appropriate expiry and cookie scope, and verify the token on protected state-changing requests. Avoid setting an unnecessarily broad Domain attribute. Use HTTPS in production.

## 2. Configure Angular's default XSRF names

When the backend uses the Angular defaults, provideHttpClient() enables the built-in XSRF support. The default token cookie is XSRF-TOKEN and the default request header is X-XSRF-TOKEN.

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

When the backend uses custom names, configure both sides to match:

~~~ts
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import {
  provideHttpClient,
  withXsrfConfiguration
} from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withXsrfConfiguration({
        cookieName: 'APP-CSRF',
        headerName: 'X-APP-CSRF'
      })
    )
  ]
};
~~~

The backend must set APP-CSRF and verify X-APP-CSRF. Changing the Angular names alone does not enable server-side protection.

## 3. Model the session API

A login response can include non-sensitive user information in its body while the browser processes the Set-Cookie response header. Frontend JavaScript cannot read the Set-Cookie response header directly.

~~~ts
// auth.service.ts
import { HttpClient } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { Observable } from 'rxjs';

export interface LoginCredentials {
  email: string;
  password: string;
}

export interface SessionUser {
  id: string;
  displayName: string;
}

@Injectable({ providedIn: 'root' })
export class AuthService {
  private readonly http = inject(HttpClient);

  loadCsrfCookie(): Observable<void> {
    // The backend returns an empty response and sets XSRF-TOKEN.
    return this.http.get<void>('/api/csrf');
  }

  login(credentials: LoginCredentials): Observable<SessionUser> {
    // The backend validates credentials and sets SESSION as HttpOnly.
    return this.http.post<SessionUser>('/api/session', credentials);
  }

  logout(): Observable<void> {
    // The backend invalidates the session and expires SESSION.
    return this.http.post<void>('/api/logout', {});
  }
}
~~~

The backend may issue the initial CSRF token during the page response or from an endpoint such as GET /api/csrf. If the backend requires CSRF validation on login, obtain the token before submitting login. The backend should rotate or refresh session-bound tokens when its authentication design requires it.

## 4. Obtain the token, then log in

This flow assumes the backend sets the JavaScript-readable XSRF-TOKEN cookie from GET /api/csrf. Angular reads it for the subsequent POST and adds X-XSRF-TOKEN automatically for the same-origin request.

~~~ts
import { switchMap } from 'rxjs';
import { AuthService, LoginCredentials } from './auth.service';

function startSession(
  auth: AuthService,
  credentials: LoginCredentials
) {
  return auth.loadCsrfCookie().pipe(
    switchMap(() => auth.login(credentials))
  );
}
~~~

Do not manually read the SESSION cookie, put a password in browser storage, or construct the XSRF header in application code for this same-origin flow.

## 5. Send protected same-origin requests

The service uses relative URLs. With the defaults, Angular attaches the XSRF header to eligible mutating requests when the token cookie is present.

~~~ts
// profile.service.ts
import { HttpClient } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { Observable } from 'rxjs';

export interface Profile {
  id: string;
  displayName: string;
}

@Injectable({ providedIn: 'root' })
export class ProfileService {
  private readonly http = inject(HttpClient);

  getProfile(): Observable<Profile> {
    return this.http.get<Profile>('/api/profile');
  }

  updateDisplayName(displayName: string): Observable<Profile> {
    return this.http.patch<Profile>('/api/profile', {
      displayName
    });
  }
}
~~~

A request/response sequence might look like this:

| Step | Request or response | Expected behavior |
| --- | --- | --- |
| 1 | GET /api/csrf | Backend sets the readable XSRF-TOKEN cookie |
| 2 | POST /api/session | Browser sends the XSRF header; backend validates credentials and sets HttpOnly SESSION |
| 3 | GET /api/profile | Browser attaches SESSION; backend validates session and authorization |
| 4 | PATCH /api/profile | Browser attaches SESSION; Angular adds X-XSRF-TOKEN; backend verifies both |
| 5 | POST /api/logout | Backend applies CSRF policy, invalidates the session, and expires SESSION |

The backend contract determines whether login itself requires the CSRF token and how token rotation works. Confirm this with the server implementation rather than assuming every API uses the same flow.

## 6. Handle an expired or forbidden session

A 401 usually means the request is not authenticated or the session is no longer valid. A 403 means the server understood the request but refused it; the cause may be authorization or a CSRF policy, depending on the API contract.

~~~ts
import { HttpErrorResponse } from '@angular/common/http';

function classifySessionError(error: unknown): string {
  if (!(error instanceof HttpErrorResponse)) {
    return 'The request failed.';
  }

  if (error.status === 401) {
    return 'Your session has expired. Sign in again.';
  }

  if (error.status === 403) {
    return 'The server refused this request. Check access and CSRF requirements.';
  }

  return 'The request failed. Try again later.';
}
~~~

Do not assume every 403 means a bad XSRF token. Log the server's structured reason safely and follow the backend's documented response contract. Never display stack traces or raw internal error details to the user.

## 7. Cross-origin request with cookies

If the Angular application and API are on different origins, the request must opt into credentials. With Angular's default fetch backend, a request can use credentials: 'include':

~~~ts
this.http.get<Profile>('https://api.example.test/profile', {
  credentials: 'include'
});
~~~

The backend must return credentialed CORS headers for the exact frontend origin, not a wildcard, and allow credentials. It must handle preflight requests when required. Cross-site cookie use also requires compatible cookie attributes, commonly SameSite=None; Secure, and may be restricted by browser privacy settings.

Angular's built-in XSRF support does not automatically attach its token header to cross-origin absolute URLs. Do not attach a token to arbitrary origins. Prefer serving the API through a same-origin reverse proxy or follow a CSRF design explicitly supported by the backend for the cross-origin deployment.

## 8. Verify cookie properties in browser developer tools

Inspect the Application or Storage panel and the Network panel:

- SESSION should be marked HttpOnly and Secure in production.
- The XSRF token cookie should be visible to the Angular origin if Angular reads it.
- A protected mutating same-origin request should include X-XSRF-TOKEN.
- The response to a request with a missing or invalid token should be rejected by the backend.
- A protected request with an expired session should not return private data.
- Credentialed cross-origin requests should use the expected exact CORS origin and credentials response headers.

Do not paste live session cookies, CSRF tokens, passwords, or authorization headers into screenshots, logs, issue reports, or chat.

## Security checklist

- Keep authentication and authorization checks on the backend.
- Use HTTPS and a Secure, HttpOnly session cookie where appropriate.
- Keep the session cookie separate from the JavaScript-readable XSRF token cookie.
- Align Angular's token cookie/header names with the backend.
- Verify the XSRF token on protected state-changing requests.
- Keep GET and HEAD endpoints free of state-changing side effects.
- Do not treat CORS or SameSite as the only CSRF defense.
- Do not send tokens to arbitrary cross-origin destinations.
- Invalidate the server session during logout.
- Avoid exposing credentials and tokens in browser storage, URLs, or logs.

## Related references

- Lesson: lessons/092-authentication-cookies-and-xsrf-protection.md
- Q&A: qnas/092-authentication-cookies-and-xsrf-protection.md
- Angular security and XSRF: https://angular.dev/best-practices/security
- Angular HttpClient setup: https://angular.dev/guide/http/setup
- Angular XSRF configuration API: https://angular.dev/api/common/http/withXsrfConfiguration
- Angular request credentials: https://angular.dev/guide/http/making-requests
- MDN Set-Cookie header: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie
- OWASP CSRF Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
