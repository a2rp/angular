# Authentication, Cookies, and XSRF Protection: Q&A

## 1. What is the difference between authentication and authorization?

Authentication establishes who the requester is. Authorization decides whether that identity may perform a specific action on a specific resource. The backend must enforce both for protected operations.

## 2. Does Angular provide a complete authentication system?

No. Angular provides an HTTP client and client-side XSRF support. The application and backend must define login, session creation, cookie settings, authorization, token validation, expiry, and logout behavior.

## 3. How does a cookie-based session usually work?

The backend verifies a login, creates a session, and sends a session cookie. The browser stores it and attaches it to matching requests. The backend validates the session and permissions on every protected request.

## 4. What should an authentication cookie contain?

Usually an opaque, unpredictable session identifier. Do not put a password or unnecessary sensitive account data in the cookie value.

## 5. What does HttpOnly do?

HttpOnly prevents browser JavaScript from reading a cookie through APIs such as document.cookie. The browser can still attach that cookie to eligible JavaScript-initiated HTTP requests.

## 6. Does HttpOnly prevent all XSS impact?

No. It reduces direct cookie theft, but malicious script running in the application origin may still make authenticated requests while it runs. XSS prevention remains important.

## 7. Why use Secure on a session cookie?

Secure restricts the browser's cookie transmission to HTTPS connections, with browser-specific localhost behavior. Production authentication should use HTTPS.

## 8. What does SameSite control?

SameSite controls whether a browser sends a cookie with cross-site requests. Lax or Strict can reduce some CSRF exposure, but the appropriate value depends on the application's navigation and deployment requirements.

## 9. When is SameSite=None used?

It permits a cookie to be sent with cross-site requests. Modern browsers require Secure with SameSite=None. Browser privacy policies may still restrict cross-site cookies.

## 10. Is SameSite a complete replacement for CSRF tokens?

Not universally. Treat it as one defense. Use the server's CSRF protection strategy for cookie-authenticated state changes and verify requests on the backend.

## 11. What is the difference between the session cookie and the XSRF token cookie?

The session cookie authenticates the browser session and should normally be HttpOnly. The XSRF token cookie carries a separate request-validation token that Angular needs to read so it can copy the value into a request header.

## 12. Why must Angular's XSRF token cookie be JavaScript-readable?

Angular's built-in mechanism reads the token cookie in the browser and adds its value to a configured request header. If that cookie is HttpOnly, JavaScript cannot read it for this mechanism.

## 13. What are Angular's default XSRF cookie and header names?

The default cookie name is XSRF-TOKEN. The default header name is X-XSRF-TOKEN.

## 14. What does Angular do with the XSRF token?

For eligible mutating requests to relative or same-origin URLs, Angular reads the configured token cookie and adds the token as a request header. It does not add that header to safe reads such as GET or HEAD.

## 15. Does Angular validate the XSRF token?

No. Angular provides the client-side cookie-to-header step. The backend must issue a valid token and verify the expected header for protected requests.

## 16. Does provideHttpClient() enable XSRF support by default?

Angular's HttpClient configuration includes built-in XSRF support by default. When the default cookie and header names match the backend, provideHttpClient() does not need a custom XSRF configuration.

## 17. How do I change the XSRF names?

Use withXsrfConfiguration in the provideHttpClient configuration and set cookieName and headerName to match the backend's contract.

## 18. Should I manually build the XSRF header for every request?

Usually no. Use Angular's built-in support for its intended same-origin use case. Avoid sending a token to arbitrary or untrusted destinations.

## 19. What changes when the API is on another origin?

Angular's default XSRF behavior does not add the token header to a cross-origin absolute URL. Cross-origin cookie requests also require a deliberate browser credentials policy, server CORS configuration, appropriate cookie attributes, and a server-supported CSRF strategy.

## 20. How can an Angular fetch-backend request include cross-origin credentials?

For a cross-origin request that should include cookies, Angular's fetch options support credentials: 'include'. The backend must permit credentialed CORS for the exact frontend origin, and the browser must allow the cookie under its SameSite and privacy rules.

## 21. Can Access-Control-Allow-Origin be a wildcard for credentialed CORS?

No. A credentialed CORS response must allow the specific requesting origin rather than using a wildcard, and it must allow credentials. The API also needs to handle preflight requests when required.

## 22. Does CORS alone stop CSRF?

No. CORS controls whether a browser exposes a cross-origin response to frontend JavaScript. It does not guarantee that a cross-site request cannot be sent. The backend needs appropriate CSRF checks for cookie-authenticated state changes.

## 23. Should GET endpoints change server state?

No. Safe read methods such as GET should not perform state-changing actions. If a GET changes state, it can undermine assumptions used by browsers and CSRF protections.

## 24. If the session cookie is HttpOnly, how does Angular send it?

The browser attaches matching cookies according to their domain, path, security, and SameSite rules. JavaScript does not need to read the session cookie for the browser to send it.

## 25. Is a bearer-token API affected by CSRF in the same way as cookie authentication?

CSRF risk is highest when credentials are attached automatically by the browser, as cookies are. A bearer token explicitly added by application code has different CSRF properties, but it creates token-storage and XSS risks. The backend must still authenticate and authorize requests.

## 26. Is localStorage a good place for a long-lived session secret?

It exposes the value to JavaScript running in the origin, including script injected by an XSS flaw. Choose storage based on a deliberate threat model; avoid storing passwords or long-lived secrets there as a shortcut.

## 27. What should logout do?

The backend should invalidate the session and expire the cookie with matching scope attributes. The frontend can clear in-memory user state. A cookie-authenticated logout is a state-changing request and should follow the backend's CSRF policy.

## 28. How do I troubleshoot a missing XSRF header?

Check that the server sets the configured token cookie, that it is readable by the application origin, and that the request uses a mutating method and a relative or same-origin URL. Then check that Angular's cookie and header names match the backend configuration.

## 29. Why does Angular not add the XSRF header to an external API URL?

The built-in mechanism is constrained to relative and same-origin URLs to avoid sending a readable token to another origin. Cross-origin APIs need an explicit, backend-designed approach rather than a broad token-forwarding interceptor.

## 30. Is same-site the same as same-origin?

No. Origin includes scheme, host, and port. Site uses a broader browser concept. Two subdomains can be same-site but different origins, so Angular's same-origin XSRF behavior and the browser's SameSite cookie rules are not interchangeable.

## 31. Should the session cookie use a broad Domain attribute?

Use the narrowest cookie scope that serves the application. A broad Domain can expose the cookie to more subdomains than necessary. Cookie scope should be designed by the backend team.

## 32. Does adding X-XSRF-TOKEN make a request safe by itself?

No. The server must compare the received token with the valid token associated with the session or use another well-designed CSRF validation scheme. A header that the server ignores provides no protection.

## 33. What if the frontend and API are served from one origin through a reverse proxy?

This often simplifies browser cookie and XSRF behavior. The browser sees same-origin URLs, Angular can apply its built-in XSRF behavior, and the proxy routes API paths to the backend. The backend still validates sessions, CSRF tokens, and permissions.

## 34. What is the main security responsibility split?

Angular can attach the XSRF header for eligible requests. The browser enforces cookie rules and same-origin restrictions. The backend must issue and validate sessions and CSRF tokens, apply authorization, and configure CORS and cookie attributes correctly.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="091-cancel-requests-and-prevent-stale-response-races.md">091 - Cancel Requests and Prevent Stale Response Races</a></td><td><a href="../lessons/092-authentication-cookies-and-xsrf-protection.md">Lesson</a></td><td><a href="../examples/092-authentication-cookies-and-xsrf-protection.md">Example</a></td><td><a href="093-caching-revalidation-and-transfer-cache.md">093 - Caching, Revalidation, and Transfer Cache</a></td>
</tr>
  </tbody>
</table>
