# Q&A: Authentication, Authorization, Secrets, and Browser Storage

## 1. What is authentication?

Authentication verifies an identity claim, such as whether a person controls an account. It is commonly handled by an identity provider or a trusted server-side authentication system.

## 2. What is authorization?

Authorization decides whether an authenticated principal may perform a particular action on a particular resource. It must account for the requested operation and object, not only whether the user is signed in.

## 3. Does a successful login mean a user can access every API endpoint?

No. Each protected server operation must authenticate the request and apply the relevant authorization policy. A signed-in user may still lack permission to access a record, account, or tenant.

## 4. Are Angular route guards security controls?

They control client-side navigation and improve the user experience. They are not a security boundary because the browser user can modify JavaScript or call the API without using the Angular router. Angular's routing guide says never to rely on client-side guards as the sole access control.

## 5. Why should a guard return a `UrlTree` for login redirects?

Returning a `UrlTree` lets the router perform the redirect as part of navigation. Angular recommends this instead of returning `false` and separately triggering navigation. The resulting `returnUrl` must still be validated before it is used after login.

## 6. What is an open redirect, and how can `returnUrl` cause one?

An open redirect sends a user to an attacker-controlled site using an unvalidated redirect parameter. Accept only safe same-application paths, and reject absolute URLs and values beginning with `//` or another scheme. The server should also validate redirects used in authentication callbacks.

## 7. Can I store an API key in Angular's environment file?

A frontend build generally includes its configuration in downloadable JavaScript. Treat every value shipped to the browser as public. A public client ID is not a confidential credential; database passwords, private keys, client secrets, and privileged API keys belong on a trusted server.

## 8. Is a JWT secure because it is encoded or signed?

Encoding only changes representation, and a signature does not make the claims confidential. A token must be validated by the trusted server for signature, issuer, audience, expiry, and applicable policy. Angular decoding a JWT does not authenticate the user or authorize an API operation.

## 9. Should access tokens be stored in `localStorage`?

Avoid storing authentication credentials, session identifiers, access tokens, or refresh tokens in Web Storage. Any JavaScript executing in the origin can read them, so XSS can expose them. Use a reviewed session or OAuth architecture instead.

## 10. Is `sessionStorage` safe for tokens because it clears when a tab closes?

No. It is still accessible to JavaScript executing in that origin and remains exposed to XSS during the tab's lifetime. Shorter persistence does not make the token inaccessible to malicious same-origin script.

## 11. Can browser storage hold anything sensitive?

Do not assume localStorage, sessionStorage, or IndexedDB provides confidentiality. A user with access to the browser profile and same-origin scripts may inspect or alter stored data. Store ordinary UI preferences if appropriate, but do not treat browser storage as a trusted permissions database or a secret vault.

## 12. What does `HttpOnly` do for a session cookie?

It prevents JavaScript from reading the cookie value. The browser can still attach the cookie to matching requests. It reduces direct token theft through script access, but it does not stop an XSS payload from issuing authenticated actions from the compromised page.

## 13. What do `Secure` and `SameSite` do?

`Secure` restricts cookie transmission to secure connections. `SameSite` controls when browsers attach cookies to cross-site requests. Use explicit cookie attributes and choose `SameSite` based on legitimate login and cross-site flows; it is useful defense in depth, not a universal CSRF solution.

## 14. What does the `__Host-` cookie prefix require?

A `__Host-` cookie must be set with `Secure`, `Path=/`, and no `Domain` attribute. This constrains it to the host that set it and helps prevent some subdomain cookie-injection problems. Confirm the app's domain and deployment model before selecting the prefix.

## 15. Why does a cookie-authenticated API need CSRF protection?

Browsers attach matching cookies automatically. A malicious site may cause a signed-in browser to send an unwanted state-changing request. Use a suitable CSRF defense on the server, keep state changes off safe methods such as `GET`, and treat `SameSite` as an additional layer unless the full deployment meets a narrowly reviewed set of conditions.

## 16. What is a Backend-for-Frontend (BFF)?

A BFF is a server component that handles OAuth responsibilities and proxies the browser application's API requests. It can keep OAuth tokens on the server and expose an application session to Angular. It adds server deployment and proxy complexity, so architecture selection should consider sensitivity, operational cost, and resource-server access patterns.

## 17. Is a BFF the only valid OAuth architecture for a browser app?

No. Browser-based OAuth architectures make different security and complexity trade-offs. Use the current OAuth guidance and a maintained implementation; do not copy token storage or callback code from an unrelated sample. For sensitive or business applications, review the stronger server-backed patterns carefully.

## 18. Does clearing Angular state log the user out?

Not by itself. Logout should invalidate the server-side session or use the identity provider's supported revocation/logout flow, and expire the session cookie. Clearing in-memory profile state is useful for the interface but cannot revoke an active server credential.

## 19. Where should session expiry be enforced?

On the server. The server should enforce session expiration and revocation. A client timer may warn the user or refresh the interface, but a client-provided timestamp is not authoritative.

## 20. Why should authentication credentials never appear in a URL?

URLs can be retained in browser history, server logs, analytics, copied links, and referrer information. Use the supported authentication protocol and secure request channels; keep credentials out of query strings and fragments unless a specific standardized flow requires a parameter and its library handles it safely.

## 21. Should an HTTP interceptor attach an access token to every request?

No. If the selected architecture requires bearer tokens, attach them only to explicitly trusted API origins using a maintained library or a narrowly scoped interceptor. Sending credentials to arbitrary URLs can leak them to third parties.

## 22. Does a role check in Angular protect an admin endpoint?

No. A role check can hide admin controls or improve navigation, but the server must validate the authenticated principal and enforce the admin permission for every admin operation. The server must not trust a role value supplied by the browser.

## 23. What should logout do in a cookie-session design?

The server should invalidate the current session and send an expired cookie with matching scope and security attributes. The frontend should clear display state and navigate appropriately. If identity-provider logout is part of the design, follow that provider's supported flow as well.

## 24. How should a team protect credentials in logs and telemetry?

Do not record passwords, authorization headers, cookies, raw session IDs, access tokens, refresh tokens, or client secrets. Configure backend, proxy, analytics, and error reporting systems to filter sensitive headers and fields. Use a non-secret correlation identifier if session-related troubleshooting is required.

## 25. What is the simplest safe mental model?

The browser can display session information and request actions. The trusted backend authenticates credentials, decides whether each action is permitted, protects secrets, and controls session lifetime. Treat every client-supplied value as untrusted.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="117-security-xss-sanitization-csp-and-trusted-content.md">117 - Security: XSS, Sanitization, CSP, and Trusted Content</a></td><td><a href="../lessons/118-authentication-authorization-secrets-and-browser-storage.md">Lesson</a></td><td><a href="../examples/118-authentication-authorization-secrets-and-browser-storage.md">Example</a></td><td><a href="119-service-workers-offline-behavior-deployment-and-updates.md">119 - Service Workers, Offline Behavior, Deployment, and Updates</a></td>
</tr>
  </tbody>
</table>
