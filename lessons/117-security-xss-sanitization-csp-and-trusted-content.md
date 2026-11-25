# Security: XSS, Sanitization, CSP, and Trusted Content

Cross-site scripting (XSS) occurs when attacker-controlled content becomes executable in a user's page. Angular protects common template bindings by treating values as untrusted and applying context-aware sanitization. Those protections do not automatically cover direct DOM APIs, unsafe third-party code, server-side template construction, or values that application code explicitly marks as trusted.

Use Angular's defaults, keep untrusted data separate from executable templates, and add Content Security Policy (CSP) and Trusted Types as defense in depth.

## Learning outcomes

- Explain Angular's default handling of untrusted template values.
- Distinguish HTML, Style, URL, and Resource URL security contexts.
- Recognize why bypassSecurityTrust methods are security-sensitive.
- Avoid unsafe DOM manipulation and template construction.
- Configure CSP nonces without reusing predictable values.
- Enable Trusted Types with only the Angular policies the app needs.

## 1. Treat external content as untrusted

User input, query parameters, API responses, CMS content, and third-party data should be treated as untrusted unless there is a reviewed reason to trust a specific value.

Interpolation displays text as text:

~~~html
<p>{{ userProvidedText }}</p>
~~~

Angular escapes interpolated content, so text such as a script tag is not executed as markup.

Binding a value to an HTML sink such as innerHTML is different:

~~~html
<div [innerHTML]="articleExcerpt"></div>
~~~

Angular sanitizes untrusted HTML in this binding context, removing unsafe parts while allowing safe markup. The content should still be treated carefully: rich text should be produced by a trusted editor or a reviewed sanitization pipeline with an explicit product requirement.

## 2. Understand security contexts

Angular uses a context based on how the value will be interpreted:

| Context | Example | Angular behavior |
| --- | --- | --- |
| HTML | innerHTML | Sanitizes untrusted HTML |
| Style | A style property binding | Applies style-context checks |
| URL | A link href | Sanitizes unsafe URL schemes |
| Resource URL | iframe src or script src | Cannot sanitize arbitrary executable resources |

A URL used for a link and a Resource URL that loads executable content are not interchangeable. A value safe in one context may be unsafe in another.

## 3. Do not bypass sanitization for untrusted values

DomSanitizer exposes methods such as bypassSecurityTrustHtml, bypassSecurityTrustUrl, and bypassSecurityTrustResourceUrl. They tell Angular to skip normal sanitization for the matching context.

Never pass user-controlled data directly to these methods. They are not sanitizers or validators. A bypass can convert a malicious value into executable content.

If a real feature needs a trusted value, validate it at its source against a narrow, documented policy. For an embedded video, accept a video identifier, validate the identifier format, and construct a fixed provider URL. Do not accept an arbitrary iframe URL from a user and mark it trusted.

## 4. Avoid unsafe direct DOM manipulation

Angular template bindings provide protections that native DOM APIs do not automatically provide. Avoid writing attacker-controlled strings to innerHTML, insertAdjacentHTML, script elements, or unsafe URL-bearing properties.

Prefer Angular templates and normal data binding. When direct DOM work is unavoidable, use Angular's sanitization APIs with the correct SecurityContext and understand how the consuming API interprets the result. Do not assume all third-party libraries sanitize DOM input.

## 5. Keep templates executable and trusted

Angular templates are executable application code. Do not build a template string by concatenating user input and compile it at runtime. Avoid JIT compilation for production and use the Angular CLI's Ahead-of-Time compiler.

Server-side template generation has the same risk: never interpolate unescaped user content into a generated Angular template. Use a server templating system that safely escapes output, and keep user data out of executable template source.

## 6. Content Security Policy

CSP is a browser-enforced defense-in-depth policy that limits where scripts, styles, images, frames, and other resources may load from. Configure it at the server or edge as an HTTP Content-Security-Policy header. A policy must be tested against the app's actual routes, build output, and third-party integrations.

For Angular's dynamically inserted styles, use a secure per-response nonce where the hosting model supports it. The same nonce must be passed to Angular through the root ngCspNonce attribute or the CSP_NONCE injection token.

A nonce must be cryptographically unpredictable and unique per response. Never hard-code one into a static page or reuse a value across users. A CDN that caches HTML containing the same nonce defeats the uniqueness requirement; generate or replace it at a trusted edge per response.

A simplified policy fragment can illustrate the relationship:

~~~http
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-RESPONSE_NONCE'; style-src 'self' 'nonce-RESPONSE_NONCE'; object-src 'none'; base-uri 'self'
~~~

This is not a drop-in policy for every application. Add only the source directives required by the app, and verify the real emitted scripts, styles, images, APIs, fonts, and frames. Keep security-sensitive directives in the HTTP header.

Angular CLI's security.autoCsp option can generate a hash-based script policy for supported static builds. Current Angular documentation notes that autoCsp does not support SSR and covers scripts rather than all style requirements. Check the current builder docs before using it.

## 7. Trusted Types

Trusted Types enforcement helps prevent DOM-based XSS by requiring dangerous DOM sinks to receive values created through an approved policy. Angular's security guide documents policies such as:

- angular: used by Angular's security-reviewed code.
- angular#bundler: used by Angular CLI bundling for lazy chunks.
- angular#unsafe-bypass: required if the app calls DomSanitizer bypassSecurityTrust methods.
- angular#unsafe-jit: required for JIT compilation.
- angular#unsafe-upgrade: required for AngularJS hybrid applications.

Enable only policies that the application actually needs. For an Angular CLI app with lazy chunks, a starting header may include angular and angular#bundler:

~~~http
Content-Security-Policy: trusted-types angular angular#bundler; require-trusted-types-for 'script'
~~~

If the app uses a bypass or JIT policy, that is a deliberate security-sensitive design that needs review. Trusted Types support depends on the browser; Angular's sanitizer still provides its framework protections where enforcement is unavailable.

## 8. Audit trusted content paths

Search the codebase for:

- bypassSecurityTrustHtml
- bypassSecurityTrustUrl
- bypassSecurityTrustResourceUrl
- innerHTML and insertAdjacentHTML
- direct assignments to script, iframe, or style sinks
- runtime template compilation
- third-party packages that mutate DOM

For every occurrence, identify the data source, target security context, validation or sanitization step, and reviewer. A single safe-looking value can become unsafe when its source or URL construction changes.

## 9. Verify policy behavior

1. Inspect the built application and list required resource origins.
2. Start with the narrowest practical CSP and Trusted Types configuration.
3. Test every route, lazy-loaded chunk, image, font, API, and embedded resource.
4. Check browser console violations and server or edge reports.
5. Confirm the nonce is different per response when using nonces.
6. Inspect direct DOM sinks and audit any trusted-value bypass.
7. Keep the policy in the actual HTTP response rather than relying only on a local development setup.

CSP and Trusted Types reduce risk; they do not replace safe coding, server-side output escaping, dependency review, or security testing.

## Key takeaways

- Angular sanitizes many untrusted template bindings according to context.
- Resource URLs cannot be made safe through general sanitization.
- Bypass APIs disable protections and must never receive untrusted values.
- Avoid direct DOM sinks and runtime template construction.
- Use CSP and Trusted Types as defense in depth with only required policies.
- CSP nonces must be unique per response and integrated with caching correctly.

## Further reading

- [Angular security best practices](https://angular.dev/best-practices/security)
- [Angular DomSanitizer API](https://angular.dev/api/platform-browser/DomSanitizer)
- [Angular SecurityContext API](https://angular.dev/api/core/SecurityContext)
- [Angular CSP_NONCE API](https://angular.dev/api/core/CSP_NONCE)
- [Angular workspace security configuration](https://angular.dev/reference/configs/workspace-config)