# Hydration, Event Replay, and Incremental Hydration

When Angular renders a route on the server or prerenders it at build time, the browser receives HTML before the client application has attached its behavior. Hydration reuses the existing server-rendered DOM and connects Angular's client-side state and event listeners to it.

Event Replay can capture supported user actions made before listeners are ready and replay them after hydration. Incremental hydration lets an application leave selected server-rendered sections dehydrated until a trigger such as viewport entry or interaction.

## Learning outcomes

- Explain the difference between server rendering and hydration.
- Describe how Angular reuses server-rendered DOM.
- Explain how Event Replay handles early interactions.
- Use incremental hydration triggers with deferrable views.
- Diagnose common hydration mismatches and DOM mutation problems.
- Use ngSkipHydration only as a narrow temporary workaround.

## 1. Server HTML needs client behavior

SSR and prerendering produce HTML that the browser can display immediately. HTML alone does not attach Angular event handlers or initialize client-side state. Hydration starts the Angular client application and associates it with the DOM already on the page.

Without hydration, Angular may recreate content in the browser after the server response arrives. Recreating content can cause flicker, duplicate work, or layout shifts. Hydration aims to preserve the server-rendered DOM while making it interactive.

Hydration applies to applications configured for SSR or prerendering. A purely client-rendered route does not have server-rendered DOM to hydrate.

## 2. Enable hydration

When SSR is added through the Angular CLI, the generated setup usually configures hydration. A custom bootstrap can provide it explicitly:

~~~ts
import { bootstrapApplication, provideClientHydration } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [provideClientHydration()],
});
~~~

Do not add duplicate providers if the CLI already configured hydration. Check the generated application and server configuration for the installed Angular version.

Current Angular documentation describes incremental hydration as enabled by default with provideClientHydration. Older Angular releases may differ, so verify the exact behavior and APIs for the project version.

## 3. Event Replay

The server-rendered page can be visible before hydration completes. A visitor may click a button during that interval, before Angular's client-side listeners are attached. Event Replay captures supported events and dispatches them after the application hydrates.

In current Angular versions, incremental hydration enables Event Replay. If a project opts out of incremental hydration but still needs early-event capture, the hydration provider can include withEventReplay:

~~~ts
import {
  provideClientHydration,
  withEventReplay,
  withNoIncrementalHydration,
} from '@angular/platform-browser';

export const appConfig = {
  providers: [
    provideClientHydration(
      withNoIncrementalHydration(),
      withEventReplay(),
    ),
  ],
};
~~~

Use the provider options supported by the project's Angular version. Event Replay improves early interaction handling, but it does not replace clear loading states or guarantee that every arbitrary browser event is replayed. Test the specific interactions the page relies on.

## 4. Incremental hydration

Incremental hydration uses @defer boundaries to let the server render a section's main content while delaying client hydration for that section. The browser can display the server HTML immediately, while Angular loads and attaches the section's code only when a hydration trigger fires.

A trigger might hydrate the section when it enters the viewport, when the user interacts, when the browser is idle, or when a condition becomes true. Hydration triggers are separate from normal defer loading triggers. They primarily control initial SSR hydration; later client-side navigations use the regular defer behavior.

See Angular's [incremental hydration guide](https://angular.dev/guide/incremental-hydration).

## 5. Hydrate content on interaction

A comments component can be included in server HTML and hydrated when the reader interacts with its trigger:

~~~html
@defer (hydrate on interaction) {
  <app-comments />
} @placeholder {
  <section aria-label="Comments">
    <h2>Comments</h2>
    <p>Comments will load when opened.</p>
  </section>
}
~~~

The comments markup can be present in the server-rendered response while its client dependencies are delayed. Interaction can trigger loading and hydration; Event Replay can preserve supported early interactions while the client is preparing.

Keep the placeholder because a later client-side navigation may render the block as an ordinary defer block. Provide a placeholder that explains the next action and keeps the layout stable.

## 6. Choose a hydration trigger

Common hydration triggers include:

- **hydrate on idle:** hydrate when the browser is idle.
- **hydrate on viewport:** hydrate when the section enters the viewport.
- **hydrate on interaction:** hydrate after a user interacts with the section.
- **hydrate on hover:** hydrate after hover or focus-related activity.
- **hydrate on immediate:** hydrate after non-deferred content is rendered.
- **hydrate on timer:** hydrate after a delay.
- **hydrate when:** hydrate once a condition becomes truthy.
- **hydrate never:** keep the section static during the initial server-rendered load.

Choose based on when the user needs the section to work. Do not delay hydration for controls that users are likely to activate immediately. A section that is static at first can still need to hydrate on later client-side navigation.

## 7. Keep server and client DOM structure consistent

Hydration expects the browser DOM to match the structure Angular rendered on the server. Common mismatch sources include:

- Invalid or differently parsed HTML nesting.
- Direct DOM mutation before hydration.
- Third-party scripts that insert, move, or remove nodes.
- CDN transformations that alter HTML whitespace or comments.
- Browser extensions that modify the page.
- Server/client conditions that render different markup.

Prefer Angular templates and platform-safe APIs. Delay browser-only DOM work until after rendering when appropriate, and avoid using platform checks in a template to produce different server and client trees.

## 8. ngSkipHydration as a temporary escape hatch

Angular supports ngSkipHydration on a component host when a specific component cannot yet be made hydration-compatible. Angular skips hydration for that component subtree and recreates it in the browser.

Use it only on a component host, only for a known incompatible subtree, and as a temporary workaround. It can reintroduce browser work and layout shifts. It is not a fix for an entire app-wide mismatch.

## 9. Debug hydration problems

1. Reproduce on an SSR or prerendered route with a production-like build.
2. Inspect the browser console for hydration mismatch errors.
3. Check that the server and browser builds use consistent compiler settings.
4. Validate HTML nesting and remove direct pre-hydration DOM changes.
5. Inspect third-party scripts, CDN transformations, and affected browser extensions.
6. Use Angular DevTools to inspect which components hydrated.
7. Apply a narrow workaround only if the underlying incompatibility cannot yet be removed.

## 10. Measure the trade-offs

Hydration can reduce duplicate rendering and preserve the initial DOM, while incremental hydration can delay client code for selected sections. Measure first paint, interaction readiness, layout stability, and the first interaction with each deferred section.

Do not delay all hydration simply to reduce startup work. A visible but unresponsive control can frustrate users. Balance initial load with the time at which each part becomes interactive.

## Key takeaways

- SSR or prerendering creates HTML; hydration connects Angular behavior to that HTML.
- Event Replay captures supported early interactions and replays them after hydration.
- Incremental hydration uses defer boundaries and hydration-specific triggers.
- The server and browser must produce compatible DOM structure.
- Use ngSkipHydration only on a specific component host as a temporary measure.
- Verify both initial rendering and the first user interaction.

## Further reading

- [Angular hydration](https://angular.dev/guide/hydration)
- [Incremental hydration](https://angular.dev/guide/incremental-hydration)
- [Deferrable views](https://angular.dev/guide/templates/defer)
- [Angular SSR and hybrid rendering](https://angular.dev/guide/ssr)