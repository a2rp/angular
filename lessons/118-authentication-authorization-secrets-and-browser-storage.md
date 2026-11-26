# Authentication, Authorization, Secrets, and Browser Storage

An Angular application can present sign-in screens and adapt its interface to a user's permissions, but it runs in a browser controlled by that user. The browser must not be the authority that grants access to protected data or actions. A secure design uses a trusted identity provider and backend, keeps confidential credentials out of frontend builds, and checks authorization on every protected server request.

## Learning outcomes

- Distinguish authentication from authorization.
- Understand Angular route guards as navigation and user-experience controls.
- Keep authorization enforcement and sensitive data on trusted servers.
- Avoid storing credentials and authentication tokens in Web Storage.
- Understand cookie, CSRF, and Backend-for-Frontend (BFF) trade-offs.
- Handle sign-in, logout, redirects, and errors without exposing secrets.

## 1. Authentication and authorization are different

**Authentication** answers “Who is this user?” The application or identity provider verifies an authenticator, such as a password, passkey, or federated sign-in response.

**Authorization** answers “May this authenticated principal perform this action on this resource?” The backend evaluates the request, the identity, the requested operation, the resource, and applicable policy. Being signed in does not automatically mean a user can read every account, edit every record, or enter an administrator workflow.

A browser can show a user's name or hide a button based on permissions, but those interface choices are not authorization. A user can edit JavaScript, call an API directly, change request data, or replay a request. The server must independently authenticate and authorize every protected operation, including access to individual records and tenant-owned resources.

## 2. Use an identity provider and trusted backend

For applications with real accounts, prefer a maintained identity provider and server-side authentication/session framework over custom password, token, and recovery flows. The backend or an OAuth/OIDC client library should handle protocol details, callback validation, token exchange, session creation, expiry, and revocation according to the chosen architecture.

An Angular build is a **public client**. JavaScript, source maps, static configuration, and downloaded assets can be inspected by visitors. A client identifier intended to be public is not a secret; a client secret, private key, database password, signing key, or privileged API key cannot be protected in the frontend. Put confidential credentials in a server-side secret manager or deployment environment and have the frontend call a narrowly authorized backend API.

Do not implement authentication by decoding a token in Angular and trusting its contents. A decoded JWT is only data; it is not proof of a valid signature, issuer, audience, expiry, or authorization. The API or trusted authentication layer must validate the credential and apply its own access policy.

## 3. Route guards improve navigation, not security boundaries

Angular route guards are useful for redirecting signed-out users away from a protected screen and showing an access-denied page to a user without a UI-level permission. Angular explicitly warns that client-side guards must never be the sole access control because browser JavaScript can be modified.

A guard can inspect the application's current session view and return a redirect `UrlTree`:

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

Treat `returnUrl` as untrusted input. Before using it after login, allow only a safe same-application path; reject absolute external URLs and protocol-relative paths such as `//example.invalid`. A guard should return a `UrlTree` or `RedirectCommand` for redirects rather than returning `false` and navigating separately.

The backend must still reject an unauthenticated or unauthorized API request. A hidden route, disabled button, route guard, or client-side role check cannot protect data returned by an endpoint.

## 4. Check authorization on the server for every request

For each protected API operation, the server should:

1. Validate the session or access token using the trusted authentication configuration.
2. Identify the authenticated principal and its current account or tenant context.
3. Check the required permission for this action.
4. Verify the principal may access this particular object, not merely that the user has a broad role.
5. Return only data the caller is authorized to receive.

Avoid relying on a client-supplied user ID, role, tenant ID, hidden form field, or decoded token claim as the authority. Derive identity from the validated server-side credential and enforce object-level checks. Apply the same checks to background jobs, exports, file downloads, and alternate API routes.

## 5. Do not place secrets or session credentials in browser storage

`localStorage`, `sessionStorage`, IndexedDB, JavaScript bundles, and client-side environment files are accessible to code running in the origin or to a user with access to the browser profile. A cross-site scripting flaw can read Web Storage. A build-time environment variable is generally substituted into public JavaScript; it does not become secret because it came from an environment file.

Do not store passwords, client secrets, private keys, long-lived access tokens, refresh tokens, or session identifiers in `localStorage` or `sessionStorage`. Do not put bearer credentials in URLs, query parameters, browser history, logs, analytics events, or error reports. Browser storage may be appropriate for non-sensitive UI preferences, but it must not be treated as a trusted source for identity, permissions, or confidential data.

An `HttpOnly` cookie prevents JavaScript from reading the cookie value, but an XSS flaw may still let attacker-controlled code issue requests from the signed-in page. Continue to prevent XSS and apply authorization checks to each backend operation.

## 6. Choose a session architecture deliberately

A common server-backed session cookie has attributes similar to:

~~~http
Set-Cookie: __Host-session=OPAQUE_SESSION_ID; Path=/; Secure; HttpOnly; SameSite=Lax
~~~

The server keeps the session meaning and permissions. The cookie carries an opaque identifier, not personal details or authorization rules. The `__Host-` prefix requires a secure, host-only cookie with `Path=/`; do not add a `Domain` attribute. Select `SameSite` based on legitimate cross-site flows and the threat model.

For a Single-Page Application, a **Backend-for-Frontend (BFF)** keeps OAuth tokens on the server and gives the browser an application session. The Angular app calls the BFF; the BFF calls protected APIs. This reduces exposure of OAuth tokens to browser JavaScript, while adding a backend service and operational complexity. Other browser OAuth architectures have different trade-offs. Choose a maintained, standards-based approach for the sensitivity and deployment needs of the product rather than copying a token-storage snippet.

Cookie-based authentication changes the CSRF threat model because browsers attach cookies automatically. Use a suitable server-side CSRF defense for state-changing requests, such as a synchronizer token or a correctly implemented signed double-submit pattern. `SameSite` is a useful defense-in-depth control, not a universal substitute. Do not make state changes through `GET` endpoints.

## 7. Handle sign-in and logout safely

- Send credentials only over HTTPS and avoid logging them.
- Use the identity provider's supported redirect and callback flow; validate the callback and anti-forgery parameters through the maintained library or server flow.
- After authentication, create or rotate the server session identifier to prevent session fixation.
- Use server-enforced idle and absolute session lifetimes appropriate to the product.
- On logout, invalidate the session server-side and expire the browser cookie. Clearing Angular memory or navigating to a login route alone does not revoke a server session or a previously issued token.
- Avoid revealing whether an account exists in password-reset and sign-in errors when that detail would aid account enumeration.
- Require stronger verification for account recovery and sensitive actions, based on the application's risk.

## 8. Keep authentication state minimal in Angular

The Angular layer usually needs a small view of session state: whether a user appears signed in, display-safe profile details, and perhaps permissions for deciding which controls to render. Load this view through a trusted session endpoint. Treat it as a presentation aid: the backend remains the source of truth for each protected action.

Do not put credentials into component state for longer than a maintained auth flow requires, print credentials or tokens to the console, or attach an authorization header to arbitrary external URLs. If using an interceptor, scope it to trusted API origins and follow the selected OAuth or BFF library's guidance.

## 9. Review the design

Ask these questions before shipping:

- Which trusted system authenticates the user?
- Where are OAuth tokens, session records, and confidential keys stored?
- Can a browser visitor inspect a supposed secret in the downloaded build?
- Does every protected endpoint verify authorization, including per-record and per-tenant access?
- Can a route guard be bypassed without exposing protected server data?
- If cookies authenticate API calls, which CSRF defenses protect state-changing requests?
- Do logout and expiry invalidate the server-side session?
- Are credentials excluded from URLs, logs, telemetry, and error messages?
- Are redirects restricted to safe same-application destinations?

## Key takeaways

- Authentication establishes identity; authorization decides access to a specific operation and resource.
- Angular guards are useful UI routing controls, never the sole security boundary.
- A frontend build cannot keep a secret from its users.
- Do not store authentication credentials or session tokens in Web Storage.
- Prefer a reviewed server-backed session or OAuth/BFF architecture that fits the application.
- Cookie sessions need secure attributes and a deliberate CSRF defense.
- Enforce authentication, authorization, expiry, and logout on the server.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="117-security-xss-sanitization-csp-and-trusted-content.md">117 - Security: XSS, Sanitization, CSP, and Trusted Content</a></td><td><a href="../qnas/118-authentication-authorization-secrets-and-browser-storage.md">Q&amp;A</a></td><td><a href="../examples/118-authentication-authorization-secrets-and-browser-storage.md">Example</a></td><td><a href="119-service-workers-offline-behavior-deployment-and-updates.md">119 - Service Workers, Offline Behavior, Deployment, and Updates</a></td>
</tr>
  </tbody>
</table>
