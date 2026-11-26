# Examples: Security, XSS, Sanitization, CSP, and Trusted Content

These examples show safe data flow and review techniques. Adapt CSP directives and resource origins to the real application; the headers below are teaching examples, not universal production policies.

## 1. Render user text as text

Use interpolation when the feature needs to display a comment, profile field, or search result. The string remains text instead of becoming executable markup.

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-comment',
  standalone: true,
  template: `
    <article>
      <h2>{{ author }}</h2>
      <p>{{ comment }}</p>
    </article>
  `,
})
export class CommentComponent {
  author = 'Asha';
  comment = '<img src=x onerror=alert(1)>';
}
~~~

The example deliberately uses a script-like string to illustrate the boundary: interpolation displays the characters instead of treating them as an image element. Do not replace the paragraph with `[innerHTML]` just to preserve angle brackets.

## 2. Treat rich HTML as a separate, reviewed feature

Angular sanitizes untrusted values bound to `[innerHTML]`, but applications should not use this as a reason to accept arbitrary markup. If the product requires rich text, define the allowed content, use a reviewed sanitization process, and keep the sanitized data path separate from ordinary text.

~~~ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-article-preview',
  standalone: true,
  template: `
    <h2>{{ title }}</h2>
    <div class="article-preview" [innerHTML]="sanitizedExcerpt"></div>
  `,
})
export class ArticlePreviewComponent {
  title = 'Release notes';

  // This value is produced by the application's reviewed rich-text pipeline.
  // It is still bound normally so Angular applies its HTML-context handling.
  sanitizedExcerpt = '<p>Improved keyboard navigation.</p>';
}
~~~

Do not add a `bypassSecurityTrustHtml` call to make unsanitized content render. Document how `sanitizedExcerpt` is produced and what tags/attributes the feature supports.

## 3. Build an embed URL from a validated identifier

An iframe source is a Resource URL. Avoid accepting an arbitrary iframe URL. This example accepts only an 11-character video identifier, constructs the URL from a fixed origin, and uses the narrow Angular bypass only after that validation. If the identifier is invalid, no frame is rendered.

~~~ts
import { Component } from '@angular/core';
import { DomSanitizer, SafeResourceUrl } from '@angular/platform-browser';

@Component({
  selector: 'app-video',
  standalone: true,
  template: `
    @if (embedUrl) {
      <iframe
        [src]="embedUrl"
        title="Course video"
        loading="lazy"
        referrerpolicy="strict-origin-when-cross-origin"
        allow="encrypted-media; picture-in-picture"
        allowfullscreen>
      </iframe>
    }
  `,
})
export class VideoComponent {
  readonly videoId = 'dQw4w9WgXcQ';
  readonly embedUrl: SafeResourceUrl | null;

  constructor(sanitizer: DomSanitizer) {
    if (!/^[A-Za-z0-9_-]{11}$/.test(this.videoId)) {
      this.embedUrl = null;
      return;
    }

    const fixedProviderUrl = 'https://www.youtube-nocookie.com/embed/' + this.videoId;
    this.embedUrl = sanitizer.bypassSecurityTrustResourceUrl(fixedProviderUrl);
  }
}
~~~

The bypass is confined to a URL assembled from a fixed trusted origin and a validated identifier. A real application should also review iframe permissions, its provider's URL requirements, and its CSP `frame-src` directive. Never pass a user-supplied full URL to the bypass method.

## 4. Prefer text APIs over HTML insertion

Unsafe pattern to remove:

~~~ts
// Do not do this with API or user-controlled data.
element.innerHTML = apiResponse.message;
~~~

If direct DOM work is truly necessary and the intended result is text, use a text API:

~~~ts
element.textContent = apiResponse.message;
~~~

In an Angular component, a template binding is usually simpler and keeps the UI declarative:

~~~html
<p>{{ apiResponse.message }}</p>
~~~

Review any remaining `innerHTML`, `insertAdjacentHTML`, script URL, iframe URL, and style sink assignments. A third-party library may have its own DOM sinks even when the surrounding Angular code uses safe bindings.

## 5. Connect a per-response CSP nonce to Angular

A server or trusted edge can generate a cryptographically unpredictable nonce for each HTML response. The value in the response header must match the value Angular receives. The following is a simplified illustration; the server must substitute a fresh value on every response and avoid shared-cache reuse of the same nonce.

~~~html
<!-- Server-rendered response: NONCE is generated for this response. -->
<app-root ngCspNonce="NONCE"></app-root>
~~~

~~~http
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-NONCE'; style-src 'self' 'nonce-NONCE'; object-src 'none'; base-uri 'self'
~~~

When the application architecture injects the nonce through Angular DI instead, use the `CSP_NONCE` token with a request-scoped value. Do not put a real nonce in a checked-in file or a shared cached HTML document. Test the actual response, dynamically inserted styles, scripts, and required third-party resources in the deployed build.

## 6. Start a Trusted Types rollout with observed sinks

For an Angular CLI application that uses lazy-loaded chunks, Angular documents the `angular` and `angular#bundler` policies as commonly relevant. A deliberately narrow example header is:

~~~http
Content-Security-Policy: trusted-types angular angular#bundler; require-trusted-types-for 'script'
~~~

Before enforcing it, exercise the app and inspect browser violations. Add other Angular policies only when the application has the corresponding reviewed feature, such as an audited sanitizer bypass, JIT compilation, or AngularJS hybrid mode. Trusted Types policy names are an allowlist, not a substitute for fixing unsafe sinks.

## 7. Audit security-sensitive code paths

Run a repository search and review each result manually:

~~~text
bypassSecurityTrust
innerHTML
insertAdjacentHTML
ElementRef
createElement('script')
[src]=
~~~

For every match, record:

1. Where the value originates: user input, URL, API, CMS, or constant.
2. How the value is validated or sanitized.
3. Which browser security context consumes it.
4. Whether the operation can be replaced with a normal Angular binding or text API.
5. What CSP or Trusted Types policy is required, and why.

A text match is a review lead, not automatic proof of a vulnerability. A missing search result is also not proof that a dependency has no DOM sinks.

## 8. Security review checklist for an embed feature

- Accept a constrained identifier, not an arbitrary executable-resource URL.
- Validate the identifier before constructing the fixed-origin URL.
- Keep any `bypassSecurityTrustResourceUrl` call next to that construction and explain why it is safe.
- Set a descriptive iframe title and only the permissions the feature needs.
- Restrict allowed frame origins in the deployed CSP.
- Test valid, malformed, empty, and attacker-controlled identifiers.
- Re-review the flow if the provider, URL format, or data source changes.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="116-server-compatible-code-metadata-and-search-visibility.md">116 - Server-Compatible Code, Metadata, and Search Visibility</a></td><td><a href="../lessons/117-security-xss-sanitization-csp-and-trusted-content.md">Lesson</a></td><td><a href="../qnas/117-security-xss-sanitization-csp-and-trusted-content.md">Q&amp;A</a></td><td><a href="118-authentication-authorization-secrets-and-browser-storage.md">118 - Authentication, Authorization, Secrets, and Browser Storage</a></td>
</tr>
  </tbody>
</table>
