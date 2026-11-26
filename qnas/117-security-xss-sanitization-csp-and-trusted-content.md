# Q&A: Security, XSS, Sanitization, CSP, and Trusted Content

## 1. What is cross-site scripting (XSS)?

XSS happens when untrusted data is interpreted as executable markup or script in a page. An attacker may then act with the privileges of the affected site in a victim's browser. The root cause is usually an unsafe path from data input to an executable browser sink.

## 2. Does Angular automatically prevent every XSS vulnerability?

No. Angular sanitizes values in supported template binding contexts, but it cannot protect every way application code can interact with the browser. Direct DOM APIs, unsafe third-party libraries, server-generated templates, insecure URLs, and values explicitly marked trusted require careful review.

## 3. What is the difference between interpolation and `[innerHTML]`?

Interpolation such as `{{ comment }}` renders a value as text. `[innerHTML]="comment"` asks the browser to parse the value as HTML; Angular sanitizes untrusted HTML for this binding, but the application should still minimize and review rich-content features.

## 4. What are Angular security contexts?

A security context describes how a bound value will be interpreted. Angular distinguishes HTML, Style, URL, and Resource URL contexts. A URL loaded into an iframe or script can execute or load active content, so it cannot be treated like an ordinary link URL.

## 5. Why can’t Angular sanitize a Resource URL?

A Resource URL identifies content the browser may execute or embed, such as a script or iframe document. There is no general transformation that makes every arbitrary resource safe. Applications must construct these URLs from trusted, validated parts and restrict allowed origins.

## 6. What do `bypassSecurityTrust...` methods do?

They tell Angular to trust a value in a specific context and skip the normal sanitizer for it. They do not validate or clean the input. Passing user-controlled or otherwise unreviewed data to one can create an XSS vulnerability.

## 7. How should an application safely support video embeds?

Accept a narrowly defined identifier rather than an arbitrary iframe URL. Validate the identifier format, construct the embed URL from a fixed provider origin, and review the iframe permissions and CSP `frame-src` rule. Do not trust an arbitrary user-supplied URL with `bypassSecurityTrustResourceUrl`.

## 8. Is a sanitizer the same as input validation?

No. Sanitization adapts a value for a particular output context. Validation checks whether input meets an application's expected rules. Neither replaces the other; validation should happen near input boundaries, and output handling must still match the sink.

## 9. Are direct DOM APIs always unsafe?

No, but they do not automatically receive all protections Angular applies to template bindings. Avoid sending untrusted values to sinks such as `innerHTML` or `insertAdjacentHTML`. Prefer text APIs or Angular templates, and review any unavoidable DOM manipulation and third-party library behavior.

## 10. Why should templates never contain user-controlled strings?

Angular templates are executable application source. Building a template from user input and compiling it can turn data into code. Keep content in component data, use AOT builds for production, and never concatenate untrusted data into template source.

## 11. What does CSP add if Angular already sanitizes bindings?

Content Security Policy is browser-enforced defense in depth. It restricts which sources may provide scripts, styles, frames, and other resources, reducing the impact of mistakes. It does not make unsafe code safe, and a policy must be tested against the application's real deployment and integrations.

## 12. What is a CSP nonce, and how should Angular receive it?

A nonce is a cryptographically unpredictable value attached to permitted inline resources and named in the response's CSP header. For Angular-inserted styles, provide the same per-response nonce using the root `ngCspNonce` attribute or Angular's `CSP_NONCE` token. Generate it for each response and ensure HTML caching does not reuse it across responses.

## 13. Is the example CSP in the lesson ready to copy into production?

No. It is an illustration of directives and nonce wiring. A real policy depends on the app's scripts, styles, APIs, fonts, images, frames, deployment, and third-party services. Start narrowly, test all routes, and collect violation reports before enforcing changes broadly.

## 14. What does Angular CLI `security.autoCsp` do?

For supported static builds, it can generate a hash-based script policy from the build output. The Angular workspace documentation notes important limitations, including lack of SSR support and that it addresses scripts rather than every style requirement. Check the current builder documentation and deployment output before enabling it.

## 15. What is Trusted Types?

Trusted Types is a browser feature that can require values passed to certain DOM injection sinks to come from approved policies. It helps prevent DOM-based XSS caused by string assignment, but it complements rather than replaces Angular sanitization and safe application design.

## 16. Which Angular Trusted Types policies might an application need?

Angular documents policies including `angular`, `angular#bundler`, `angular#unsafe-bypass`, `angular#unsafe-jit`, and `angular#unsafe-upgrade`. The required set depends on the app: lazy chunks may need the bundler policy; bypass APIs, JIT, or AngularJS hybrid mode have additional security-sensitive requirements. Enable only policies that match actual usage.

## 17. Should CSP be configured with a meta tag or an HTTP header?

Use the HTTP `Content-Security-Policy` response header for security-sensitive deployment policy. It offers the full policy behavior and is managed at the server or trusted edge. Verify the actual production response, not only a local development page.

## 18. How do CDNs and caches affect nonce-based CSP?

If cached HTML reuses the same nonce, the nonce is no longer unique per response and loses an important security property. Generate or replace the nonce and matching HTML attribute at a trusted edge for every response, or use an appropriate hash-based approach for the deployment.

## 19. What should a team audit before adding Trusted Types enforcement?

Search for sanitizer bypass methods, direct HTML insertion, script or iframe assignments, runtime template compilation, and dependencies that mutate the DOM. For each result, trace the data source, sink context, validation, and required policy; fix unsafe paths before enforcing a restrictive policy.

## 20. How can a team roll out CSP and Trusted Types safely?

Inventory resources and sinks, test in a representative environment, review browser violations, exercise routes and lazy-loaded features, and enforce a narrowly scoped policy after resolving breakage. Keep policy changes under review and recheck them when dependencies or deployment architecture change.

## 21. Do CSP and Trusted Types replace server-side output escaping?

No. They are defense-in-depth controls. Servers still need context-appropriate output escaping, safe handling of user content, secure dependencies, and normal authorization and input validation.

## 22. What is the practical rule for trusted values?

Trust should be explicit, narrow, and tied to a documented source and context. Never mark data trusted merely to make a binding render. Prefer a safe representation that avoids executable markup altogether.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="116-server-compatible-code-metadata-and-search-visibility.md">116 - Server-Compatible Code, Metadata, and Search Visibility</a></td><td><a href="../lessons/117-security-xss-sanitization-csp-and-trusted-content.md">Lesson</a></td><td><a href="../examples/117-security-xss-sanitization-csp-and-trusted-content.md">Example</a></td><td><a href="118-authentication-authorization-secrets-and-browser-storage.md">118 - Authentication, Authorization, Secrets, and Browser Storage</a></td>
</tr>
  </tbody>
</table>
