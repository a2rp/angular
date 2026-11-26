# Authentication, Cookies, and XSRF Protection

Browser authentication depends on the frontend, browser, and server working together. This lesson explains session cookies, Angular's built-in XSRF request support, and the server responsibilities that make the protection effective.

## Learning outcomes

After this lesson, you should be able to:

- Distinguish authentication from authorization.
- Describe how a browser sends a session cookie with requests.
- Choose secure attributes for an authentication cookie.
- Explain the cookie-and-header XSRF token pattern.
- Configure Angular's built-in XSRF cookie and header names.
- Identify what changes when the API is cross-origin.
- Recognize that client-side settings do not replace server-side validation.

## 1. Authentication and authorization

Authentication answers: who is making this request? Authorization answers: is that identity allowed to perform this action?

An Angular application can display or hide controls based on a user's state, but that is only a user-interface decision. The backend must authenticate every protected request and check authorization for the specific resource and action. A user can call an API without using the Angular interface.

Angular does not provide a complete login system or decide how the backend stores accounts and sessions. The application and backend must agree on the authentication flow.

## 2. Cookie-based session flow

A common browser flow works like this:

1. The user submits credentials over HTTPS.
2. The backend validates them and creates a session.
3. The backend returns a session cookie using Set-Cookie.
4. The browser stores the cookie and sends it to matching requests automatically.
5. The backend validates the session and checks authorization for each protected operation.
6. Logout invalidates the server session and expires the cookie.

The session cookie should contain an opaque, unpredictable session identifier, not the user's password or sensitive account data. Keep session creation, rotation, expiry, and invalidation under backend control.

## 3. Set session cookie attributes deliberately

A production session cookie commonly uses Secure, HttpOnly, and a SameSite policy appropriate to the application's navigation and deployment.

~~~http
Set-Cookie: SESSION=opaque-session-id; Path=/; Secure; HttpOnly; SameSite=Lax
~~~

- Secure limits transmission to HTTPS connections.
- HttpOnly prevents JavaScript from reading the cookie. The browser can still attach it to matching JavaScript-initiated requests.
- SameSite controls whether the browser sends the cookie with cross-site requests. Lax or Strict can reduce CSRF exposure, but cookie behavior must fit the login and navigation flow.
- Path and Domain affect where the browser sends the cookie. Use the narrowest scope that works for the application.
- SameSite=None is for cross-site cookie use and requires Secure in modern browsers.

These are server-set cookie attributes. Angular code cannot add HttpOnly to a cookie or read a Set-Cookie response header.

HttpOnly reduces the chance that script can steal a session cookie, but it does not stop malicious same-origin script from making authenticated requests. Preventing and responding to XSS remains essential.

## 4. Why cookie authentication needs CSRF protection

Browsers attach matching cookies automatically. A malicious site may be able to cause a browser to send a request to another site where the user is signed in. If that request changes state and the server does not verify its origin or a CSRF token, the browser can act with the user's cookie.

XSRF and CSRF refer to the same general class of attack. A common defense uses two values:

1. The server sends a CSRF token in a cookie that frontend JavaScript can read.
2. Angular copies that token into a custom request header on eligible requests.
3. The server compares the header token with the expected token and rejects missing or invalid values.

The authentication session cookie and the XSRF token cookie serve different roles. Keep the session cookie HttpOnly. The XSRF token cookie must be readable by the Angular client for Angular's built-in cookie-to-header mechanism.

The backend must create a token that is unpredictable and valid for the user's session, set the token cookie, and verify the submitted header. A client that sends a header the server never checks does not have effective CSRF protection.

## 5. Angular's built-in XSRF behavior

When using provideHttpClient(), Angular includes XSRF support by default. The default names are:

| Purpose | Default name |
| --- | --- |
| JavaScript-readable token cookie | XSRF-TOKEN |
| Request header copied by Angular | X-XSRF-TOKEN |

Angular adds the token header to mutating requests such as POST when the URL is relative or same-origin. It does not add the header to safe read requests such as GET or HEAD, or to cross-origin absolute URLs.

The backend must set the XSRF-TOKEN cookie and verify the X-XSRF-TOKEN header. Angular supplies only the client half of this scheme.

If the backend uses different names, configure them when setting up HttpClient:

~~~ts
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

Use provideHttpClient() without a custom XSRF feature when the defaults match the backend. Do not disable XSRF protection just to work around an API configuration mismatch; align the frontend and server instead.

## 6. Use the default behavior for same-origin API calls

Serving the Angular application and API under the same origin, often through a reverse proxy, keeps cookie and XSRF behavior simpler. The browser sends the session cookie according to its scope, and Angular can read the XSRF token cookie and attach the matching header.

~~~ts
import { HttpClient } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { Observable } from 'rxjs';

interface Profile {
  id: string;
  displayName: string;
}

@Injectable({ providedIn: 'root' })
export class ProfileService {
  private readonly http = inject(HttpClient);

  getProfile(): Observable<Profile> {
    return this.http.get<Profile>('/api/profile');
  }

  updateProfile(displayName: string): Observable<Profile> {
    return this.http.patch<Profile>('/api/profile', {
      displayName
    });
  }
}
~~~

The service does not manually read document.cookie or construct the XSRF header. Angular's configured HttpClient handles its client-side token behavior; the server still validates the token and session.

## 7. Cross-origin APIs need deliberate configuration

A frontend at one origin calling an API at another origin is a different deployment. For cross-origin cookie requests, the browser must be allowed to send credentials, the server must opt into credentialed CORS for the exact frontend origin, and the cookie's SameSite and Secure attributes must permit the request.

With Angular's default fetch backend, a request can opt in to cross-origin credentials as follows:

~~~ts
this.http.get<Profile>('https://api.example.test/profile', {
  credentials: 'include'
});
~~~

The server must return an appropriate Access-Control-Allow-Origin value for the requesting application and Access-Control-Allow-Credentials: true. A wildcard origin cannot be used for credentialed CORS. Custom headers can cause a CORS preflight, which the server must handle. CORS controls whether the browser exposes a cross-origin response to JavaScript; it is not a substitute for CSRF validation.

Angular's built-in XSRF behavior does not automatically add its header to a cross-origin absolute URL. Do not solve this by attaching a secret token to every request. Prefer a same-origin API or a backend-designed CSRF mechanism with a narrowly scoped, explicit origin policy.

Cross-site cookie use can also be restricted by browser privacy settings. Test the actual deployed domains and browser behavior rather than assuming that a local development setup proves production cookie behavior.

## 8. Avoid sensitive credential storage in JavaScript-readable places

A JavaScript-readable storage location exposes its contents to any script running in the application origin. XSS can therefore steal bearer tokens stored there. An HttpOnly cookie prevents direct JavaScript reads of the cookie, though an XSS payload may still issue requests as the user while it runs.

Choose an authentication design that fits the backend and threat model. Avoid storing passwords or long-lived session secrets in localStorage or sessionStorage as a shortcut. Use HTTPS, short and controlled session lifetimes, server-side authorization, and strong XSS defenses.

## 9. Logout and session expiry

Logout should invalidate the server-side session and expire the session cookie using matching cookie scope attributes. The frontend can then clear its in-memory user state and navigate to the signed-out view.

~~~ts
logout(): Observable<void> {
  return this.http.post<void>('/api/logout', {});
}
~~~

A cookie-based logout request changes state, so the backend should apply its CSRF policy to it. Do not rely only on deleting a client-side value when the server session remains valid.

## Common mistakes

### Assuming Angular alone enables CSRF protection

Angular can copy a token into a header. The server must issue the token and verify it against the authenticated request.

### Making the session cookie JavaScript-readable unnecessarily

The session cookie is normally an authentication credential. Use HttpOnly when the application does not need script access to that cookie. The separate XSRF token cookie has a different role.

### Setting HttpOnly on the Angular-readable XSRF token cookie

Angular's built-in mechanism reads the token cookie in the browser. If JavaScript cannot read it, the client cannot copy it into the configured header. Keep the authentication cookie HttpOnly and design the CSRF token cookie separately.

### Treating SameSite as the only defense

SameSite can reduce cross-site cookie sending, but it is not a universal replacement for server-validated CSRF tokens or other appropriate controls.

### Assuming CORS prevents a forged request from being sent

CORS governs browser access to cross-origin responses. It does not guarantee that a browser will never send a cross-site request. Use server-side CSRF defenses for cookie-authenticated state changes.

### Manually adding a token header to every URL

Sending secrets to untrusted or cross-origin destinations can leak credentials. Keep request destinations constrained and use the built-in behavior or an explicit, reviewed policy.

## Key idea

For cookie-based authentication, the backend issues and validates the session and CSRF tokens. Angular can copy a JavaScript-readable XSRF cookie into a request header for eligible same-origin mutations. Secure cookie attributes, correct CORS configuration, server-side authorization, and XSS defenses complete the design.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="091-cancel-requests-and-prevent-stale-response-races.md">091 - Cancel Requests and Prevent Stale Response Races</a></td><td><a href="../qnas/092-authentication-cookies-and-xsrf-protection.md">Q&amp;A</a></td><td><a href="../examples/092-authentication-cookies-and-xsrf-protection.md">Example</a></td><td><a href="093-caching-revalidation-and-transfer-cache.md">093 - Caching, Revalidation, and Transfer Cache</a></td>
</tr>
  </tbody>
</table>
