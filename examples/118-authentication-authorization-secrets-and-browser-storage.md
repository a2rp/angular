# Examples: Authentication, Authorization, Secrets, and Browser Storage

These are architecture examples, not a complete authentication implementation. Use the identity provider, backend session framework, and OAuth/OIDC library selected for the application. The server remains authoritative for authentication and authorization.

## 1. Redirect a signed-out user with a route guard

Assume the application has already called its same-origin session endpoint during startup and `SessionViewService` holds only the current UI view of that response. It does not read an access token from browser storage.

~~~ts
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { SessionViewService } from './session-view.service';

export const signedInGuard: CanActivateFn = (_route, state) => {
  const session = inject(SessionViewService);
  const router = inject(Router);

  if (session.isSignedIn()) {
    return true;
  }

  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url },
  });
};
~~~

Before using `returnUrl` after authentication, accept only a safe same-application route. This guard controls navigation only; it does not protect the API.

## 2. Show an admin route only when the session view says so

A UI role check can improve the interface by routing a user to an access-denied page. Keep the backend check for the actual admin operation.

~~~ts
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { SessionViewService } from './session-view.service';

export const adminPageGuard: CanActivateFn = () => {
  const session = inject(SessionViewService);
  const router = inject(Router);

  if (session.hasDisplayPermission('admin:dashboard')) {
    return true;
  }

  return router.createUrlTree(['/forbidden']);
};
~~~

A modified browser can bypass this guard. The server endpoint that returns admin data must independently verify the current session and permission.

## 3. Keep the session cookie server-managed

A server may issue an opaque session cookie after a successful login:

~~~http
Set-Cookie: __Host-session=OPAQUE_RANDOM_SESSION_ID; Path=/; Secure; HttpOnly; SameSite=Lax
Cache-Control: no-store
~~~

The server stores session state and checks it on every protected request. `__Host-` cookies require `Secure`, `Path=/`, and no `Domain` attribute. Select `SameSite` for the product's sign-in and cross-site needs, and add a suitable CSRF defense to state-changing requests.

Do not copy the cookie value into Angular state, localStorage, a query parameter, or an analytics event. JavaScript cannot read an `HttpOnly` cookie; the browser attaches it to matching requests.

## 4. Send a CSRF token with a state-changing request

In a cookie-session design, the backend can provide a CSRF token through a trusted same-origin session/bootstrap response. Angular sends that token in a header, while the browser attaches the `HttpOnly` session cookie. The backend must validate the token and the authenticated session before applying the change.

~~~ts
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class TransferApi {
  private readonly http = inject(HttpClient);

  createTransfer(input: { recipientId: string; amount: number }, csrfToken: string) {
    return this.http.post(
      '/api/transfers',
      input,
      {
        headers: new HttpHeaders({ 'X-CSRF-Token': csrfToken }),
      },
    );
  }
}
~~~

This shows only the client request shape. CSRF token creation, binding, validation, and rejection belong to the server's reviewed CSRF implementation. Never put the token in the URL or attach it to requests sent to an untrusted origin.

## 5. Reject unauthorized object access on the server

The client should send the resource identifier it wants to access, but the backend must derive the principal from the validated session and verify access to that particular resource.

~~~text
GET /api/invoices/inv_123

Server-side decision:
  principal = validateSession(request.cookie)
  invoice = loadInvoice("inv_123")
  if invoice is missing: return not found
  if principal cannot read invoice for its account: return forbidden or not found
  return only the authorized invoice fields
~~~

Do not authorize by trusting a request such as `?userId=current-user` or a role string sent by Angular. The server determines the principal and checks account ownership or policy for each requested object.

## 6. Complete logout on the server

Angular should call the backend logout endpoint, then clear its display state. The backend invalidates the session and expires the cookie with matching scope and attributes.

~~~ts
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { finalize } from 'rxjs';
import { SessionViewService } from './session-view.service';

@Injectable({ providedIn: 'root' })
export class LogoutAction {
  private readonly http = inject(HttpClient);
  private readonly session = inject(SessionViewService);

  logout() {
    return this.http.post<void>('/api/logout', {}).pipe(
      finalize(() => this.session.clearView()),
    );
  }
}
~~~

A production flow should decide how to handle a failed logout request. Clearing local display state is not equivalent to invalidating a server session; do not report success if the server session remains active.

## 7. Identify a frontend “secret” before release

A value like this is not secret after it enters the browser build:

~~~ts
// Do not ship a real confidential credential in frontend configuration.
export const environment = {
  apiBaseUrl: 'https://api.example.test',
  privateApiKey: 'replace-me',
};
~~~

Build tools can inline configuration into JavaScript. Put confidential keys on a trusted server and expose only a limited public API that checks the user's session and authorization. A public identity-provider client ID may be intentionally visible; it must not be confused with a client secret.

## 8. Review the request paths

For a protected feature, walk through these cases:

1. Signed-out user opens a protected route: the guard returns a login `UrlTree`.
2. User edits `returnUrl`: the login completion flow rejects external destinations.
3. User skips Angular and calls the API directly: the server rejects the missing/invalid session.
4. Signed-in user changes a record ID: the server checks object-level permission.
5. Cross-site page triggers a state change: the server's CSRF defense rejects it.
6. User logs out: the server invalidates the session and expires the cookie.
7. An XSS payload executes: no credentials are available in Web Storage; server authorization still limits each action.
8. Frontend bundle is inspected: no private key, database password, client secret, or privileged API credential is present.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="117-security-xss-sanitization-csp-and-trusted-content.md">117 - Security: XSS, Sanitization, CSP, and Trusted Content</a></td><td><a href="../lessons/118-authentication-authorization-secrets-and-browser-storage.md">Lesson</a></td><td><a href="../qnas/118-authentication-authorization-secrets-and-browser-storage.md">Q&amp;A</a></td><td><a href="119-service-workers-offline-behavior-deployment-and-updates.md">119 - Service Workers, Offline Behavior, Deployment, and Updates</a></td>
</tr>
  </tbody>
</table>
