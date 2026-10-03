# 16. Browser security, accessibility, and performance

[Back to notes index](../README.md)

| [Previous: Collections, iterators, and generators](15-collections-iterators-and-generators.md) | [Notes index](../README.md) | [Next: All code samples](98-all-code-samples.md) |
|:--|:--:|--:|

## Treat input as untrusted

Text from a form, URL, browser storage, or network response can be controlled by someone outside the application. Validate it for the action being performed and render plain text as text.

~~~js
const message = document.querySelector('#message');
const userText = new URLSearchParams(location.search).get('message');

if (message) {
  message.textContent = userText ?? '';
}
~~~

textContent does not parse its value as markup. Avoid assigning untrusted data to innerHTML. Do not use eval or Function to execute a string as code.

~~~js
const heading = document.createElement('h2');
heading.textContent = 'A safe note title';
document.querySelector('main')?.append(heading);
~~~

If rich user-authored HTML is a real product requirement, use a well-reviewed sanitization library and a strict allowlist. Escaping or sanitizing should happen at the correct output boundary.

## Validate URL schemes

A URL can use protocols beyond https. Before turning an untrusted string into a link, parse it and allow only the schemes the application supports.

~~~js
function getSafeWebUrl(value) {
  try {
    const url = new URL(value, location.origin);

    if (url.protocol !== 'https:' && url.protocol !== 'http:') {
      return null;
    }

    return url.href;
  } catch {
    return null;
  }
}

const safeUrl = getSafeWebUrl(userProvidedUrl);

if (safeUrl) {
  const link = document.createElement('a');
  link.href = safeUrl;
  link.textContent = 'Open reference';
  document.querySelector('main')?.append(link);
}
~~~

The allowed protocols depend on the feature. A link to an external site may require https only. Avoid javascript: URLs and do not trust a URL merely because it came from a field named url.

## Avoid unsafe dynamic code and data merging

Never use eval to parse JSON or run text from a user. Use JSON.parse for JSON and validate the parsed value.

When updating an object from external data, select the fields the application accepts instead of blindly copying arbitrary keys.

~~~js
function makePreferences(input) {
  return {
    theme: input.theme === 'light' ? 'light' : 'dark',
    pageSize:
      Number.isInteger(input.pageSize) && input.pageSize > 0
        ? input.pageSize
        : 20,
  };
}
~~~

Whitelisting fields avoids unexpected values and reduces the risk of unsafe object merges. For nested configuration, validate each level and reject special keys that the application does not need.

## Protect credentials and API access

- Do not place server secrets in frontend JavaScript.
- Do not store passwords or private credentials in localStorage.
- Use HTTPS for production traffic.
- Validate authorization on the server for every protected operation.
- Treat CORS as a browser policy, not an access-control system.
- Keep dependencies and lock files reviewed and up to date.
- Use appropriate security headers, including a Content Security Policy, on the server.

Security controls work together. A browser-side check improves the interface but cannot protect a server endpoint by itself.

## Build accessible interactions

Use semantic HTML elements that already provide the expected behavior. A button is for an action. An anchor is for navigation. A form submits related input.

~~~html
<button id="save-note" type="button">Save note</button>
<p id="save-status" aria-live="polite"></p>
~~~

~~~js
const saveButton = document.querySelector('#save-note');
const saveStatus = document.querySelector('#save-status');

saveButton?.addEventListener('click', () => {
  if (saveStatus) {
    saveStatus.textContent = 'Note saved.';
  }
});
~~~

A native button works with a keyboard and assistive technology. A clickable non-button element needs additional keyboard handling, focusability, and an accessible name, so use the native control whenever possible.

Use an accessible name that describes the control's purpose. Keep focus visible. When an action removes or replaces the focused element, move focus to a sensible next place rather than leaving it lost.

## Announce dynamic changes carefully

A live region can announce a message that changes after an action. Use a polite announcement for routine updates and an alert for urgent errors. Do not announce every minor visual change.

~~~html
<p id="save-status" aria-live="polite"></p>
<p id="error-message" role="alert"></p>
~~~

~~~js
const status = document.querySelector('#save-status');

if (status) {
  status.textContent = 'Three notes loaded.';
}
~~~

Provide text for status and errors. Do not communicate meaning through color alone. Check the page using a keyboard and, when possible, a screen reader.

## Measure before optimizing

Use performance.now for a high-resolution time measurement. Measure the actual work with representative input rather than guessing where the slow part is.

~~~js
const start = performance.now();

const visibleNotes = notes.filter((note) =>
  note.title.toLowerCase().includes(query.toLowerCase()),
);

const elapsed = performance.now() - start;
console.log('Filter took', elapsed, 'milliseconds');
~~~

For a broader measure, use performance.mark and performance.measure or the browser Performance panel. Avoid optimizing a small operation unless the measurement shows that it matters.

## Reduce unnecessary repeated work

If a search runs on every keystroke, a debounce can wait briefly for input to stop before performing the work.

~~~js
let searchTimer;

function scheduleSearch(query) {
  clearTimeout(searchTimer);

  searchTimer = setTimeout(() => {
    performSearch(query);
  }, 250);
}
~~~

For very large lists, consider limiting rendered rows or using virtualization. For frequent DOM changes, build a fragment and append it once. Avoid reading layout properties between many style writes because that can force repeated layout calculations.

~~~js
const fragment = document.createDocumentFragment();

for (const note of notes) {
  const item = document.createElement('li');
  item.textContent = note.title;
  fragment.append(item);
}

document.querySelector('#note-list')?.replaceChildren(fragment);
~~~

The best strategy depends on the real bottleneck. Keep the implementation simple until there is evidence that it needs more work.

## Schedule visual updates with requestAnimationFrame

requestAnimationFrame schedules a callback before the browser's next repaint. It can coordinate visual updates, but it should not replace ordinary event or data flow.

~~~js
let frameId;

function scheduleVisualUpdate() {
  cancelAnimationFrame(frameId);

  frameId = requestAnimationFrame(() => {
    document.documentElement.classList.add('updated');
  });
}
~~~

Cancel an animation frame when the work is no longer needed. For animations, also respect the user's reduced-motion preference in CSS.

## Avoid leaks from listeners and retained data

Long-lived listeners, timers, and references can keep objects in memory after a view is no longer used. Remove listeners when they are no longer needed and clear timers during cleanup.

~~~js
const controller = new AbortController();

window.addEventListener('resize', handleResize, {
  signal: controller.signal,
});

function cleanupView() {
  controller.abort();
  clearTimeout(searchTimer);
}
~~~

For large data, avoid retaining duplicate copies without a reason. Use browser memory tools to investigate actual growth before changing data structures.

## Hands-on: review a small feature

Take the note list from the browser chapters and check it against these points:

1. User text is added with textContent.
2. Links accept only the URL schemes the feature needs.
3. API permissions are checked by the server.
4. Buttons and links use native elements.
5. Status changes are announced with useful text.
6. Keyboard focus remains visible and predictable.
7. Filtering is measured with a realistic list.
8. List rendering avoids unnecessary repeated DOM work.
9. Listeners and timers are cleaned up when the view is removed.

Fix the highest-impact issue first, then repeat the check.

## Notes to remember

- Treat user input, URLs, storage, and network data as untrusted.
- Use textContent for plain text and validate URL schemes before linking.
- Never execute input as code or expose private credentials in browser files.
- Use semantic controls, visible focus, and text for status and errors.
- Measure real work before optimizing.
- Clean up listeners and timers that outlive a view.
- Browser checks improve usability, while the server protects data and permissions.

## References

- [MDN web security](https://developer.mozilla.org/en-US/docs/Web/Security)
- [MDN Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [OWASP Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [MDN web accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility)
- [MDN performance](https://developer.mozilla.org/en-US/docs/Web/Performance)
- [MDN performance.now](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now)
- [MDN requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame)

---

| [Previous: Collections, iterators, and generators](15-collections-iterators-and-generators.md) | [Notes index](../README.md) | [Next: All code samples](98-all-code-samples.md) |
|:--|:--:|--:|
