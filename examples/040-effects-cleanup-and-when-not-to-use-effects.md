# 40. Example: effects, cleanup, and imperative side effects

This standalone Angular page demonstrates three appropriate effect uses: syncing a signal to browser storage, scheduling work that is cancelled when its input changes, and starting/stopping an independently managed logging effect. It also shows a computed value for derived state so an effect is not used as a state-copying mechanism.

## What you will build

- A theme preference synchronized with `localStorage` in the browser.
- A delayed console message whose previous timer is cleared when the text changes.
- An audit effect that can be started and explicitly destroyed.
- A read-only derived character count implemented with `computed()`.
- An optional post-render DOM integration using `afterRenderEffect()`.

## Requirements and setup

`effect()` and `afterRenderEffect()` are stable in Angular 20 and later. Check the Angular version used by your project. To create a practice project:

~~~sh
ng new effect-cleanup-demo --standalone --style=css
cd effect-cleanup-demo
ng serve
~~~

Use the generated root component. The files below use the conventional `app.component.*` names; adapt them if your CLI generated shorter names such as `app.*`.

## File layout

~~~text
src/app/
  app.component.ts
  app.component.html
  app.component.css
~~~

## 1. Create the component and effects

Replace or adapt `src/app/app.component.ts`:

~~~ts
import { isPlatformBrowser } from '@angular/common';
import {
  Component,
  computed,
  effect,
  EffectRef,
  inject,
  Injector,
  PLATFORM_ID,
  signal,
} from '@angular/core';

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent {
  private readonly platformId = inject(PLATFORM_ID);
  private readonly injector = inject(Injector);
  private auditEffect: EffectRef | undefined;

  readonly theme = signal<'light' | 'dark'>('light');
  readonly announcement = signal('Try changing this text quickly.');
  readonly announcementLength = computed(() => this.announcement().length);
  readonly currentUser = signal('Ada');

  constructor() {
    // Appropriate: sync signal state to an imperative browser storage API.
    effect(() => {
      const theme = this.theme();

      if (!isPlatformBrowser(this.platformId)) {
        return;
      }

      try {
        window.localStorage.setItem('effect-demo-theme', theme);
      } catch (error) {
        // Storage can be unavailable in some browser privacy modes.
        console.warn('Could not save the theme preference.', error);
      }
    });

    // Each keystroke cancels the previous timer and schedules a new one.
    effect((onCleanup) => {
      const announcement = this.announcement().trim();

      if (!announcement || !isPlatformBrowser(this.platformId)) {
        return;
      }

      const timeoutId = window.setTimeout(() => {
        console.info('Announcement settled:', announcement);
      }, 700);

      onCleanup(() => window.clearTimeout(timeoutId));
    });
  }

  toggleTheme(): void {
    this.theme.update((theme) => (theme === 'light' ? 'dark' : 'light'));
  }

  setAnnouncement(value: string): void {
    this.announcement.set(value);
  }

  startAudit(): void {
    if (this.auditEffect) {
      return;
    }

    // This method runs later, so provide the injector captured in the field initializer.
    this.auditEffect = effect(
      () => {
        console.info('Audit: current user is', this.currentUser());
      },
      { injector: this.injector },
    );
  }

  stopAudit(): void {
    this.auditEffect?.destroy();
    this.auditEffect = undefined;
  }

  switchUser(): void {
    this.currentUser.update((user) => (user === 'Ada' ? 'Grace' : 'Ada'));
  }
}
~~~

The first effect reads `theme()` and persists it only in a browser. The second reads `announcement()` synchronously, then starts a timer. If the text changes before the timer completes, cleanup cancels the older timer. The timer callback logs the captured text; it does not read another signal after the effect has returned.

The audit effect is deliberately created from a button handler rather than a constructor. It receives the component's injector explicitly, and the component retains its `EffectRef` so the Stop button can end it. The guard in `startAudit()` prevents repeated clicks from creating duplicate effects.

## 2. Add the template

Create or replace `src/app/app.component.html`:

~~~html
<main class="page" [class.dark-mode]="theme() === 'dark'">
  <header class="page-header">
    <p class="eyebrow">Angular example 40</p>
    <h1>Effects and cleanup</h1>
    <p>
      Use effects to synchronize signals with imperative APIs, and clean up
      work when the inputs change.
    </p>
  </header>

  <section class="card" aria-labelledby="theme-heading">
    <h2 id="theme-heading">Browser storage</h2>
    <p>Current theme: <strong>{{ theme() }}</strong></p>
    <button type="button" (click)="toggleTheme()">Toggle theme</button>
    <p class="hint">
      The effect saves this preference as <code>effect-demo-theme</code> in
      localStorage when running in a browser. Inspect the browser's storage panel
      to see it update.
    </p>
  </section>

  <section class="card" aria-labelledby="cleanup-heading">
    <h2 id="cleanup-heading">Cleanup a replaced timer</h2>
    <label for="announcement">Announcement text</label>
    <input
      id="announcement"
      type="text"
      [value]="announcement()"
      (input)="setAnnouncement($any($event.target).value)"
    />
    <p>{{ announcementLength() }} characters</p>
    <p class="hint">
      Wait 700 ms after typing and check the browser console. If you type again
      before the timer finishes, cleanup cancels the previous timer.
    </p>
  </section>

  <section class="card" aria-labelledby="audit-heading">
    <h2 id="audit-heading">Explicit effect lifetime</h2>
    <p>Current user: <strong>{{ currentUser() }}</strong></p>
    <div class="button-row">
      <button type="button" (click)="startAudit()">Start audit</button>
      <button type="button" (click)="stopAudit()">Stop audit</button>
      <button type="button" (click)="switchUser()">Switch user</button>
    </div>
    <p class="hint">
      Start the audit, switch users, and inspect the console. Stop the audit,
      then switch users again: the stopped effect no longer logs changes.
    </p>
  </section>

  <footer class="page-footer">
    <p>
      The character count is derived with <code>computed()</code>; no effect is
      needed to copy the announcement into another signal.
    </p>
  </footer>
</main>
~~~

The character count is read-only derived state. `computed()` recalculates it when the announcement changes, and Angular updates the template without an effect that writes to another signal.

## 3. Add simple styles

Create or replace `src/app/app.component.css`:

~~~css
:host {
  display: block;
  min-height: 100vh;
  color: #172033;
  background: #f3f6fb;
  font-family: system-ui, sans-serif;
}

.page {
  box-sizing: border-box;
  width: min(100% - 2rem, 48rem);
  margin-inline: auto;
  padding-block: 3rem;
}

.page-header,
.card,
.page-footer {
  margin-block-end: 1.25rem;
}

.eyebrow {
  color: #3156a6;
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.card {
  padding: 1.25rem;
  border: 1px solid #d6deeb;
  border-radius: 0.75rem;
  background: #fff;
}

h1,
h2 {
  line-height: 1.2;
}

label,
input {
  display: block;
}

input {
  box-sizing: border-box;
  width: 100%;
  margin-block: 0.5rem;
  padding: 0.65rem;
  border: 1px solid #9aa8bd;
  border-radius: 0.4rem;
  font: inherit;
}

button {
  margin: 0.25rem 0.5rem 0.25rem 0;
  padding: 0.6rem 0.8rem;
  border: 0;
  border-radius: 0.4rem;
  color: #fff;
  background: #244b9a;
  font: inherit;
  cursor: pointer;
}

button:focus-visible,
input:focus-visible {
  outline: 3px solid #f2a900;
  outline-offset: 2px;
}

.hint {
  color: #526078;
  line-height: 1.55;
}

.dark-mode {
  color: #eef3ff;
  background: #172033;
}

.dark-mode .card {
  border-color: #43516a;
  background: #222e44;
}

.dark-mode .hint {
  color: #c0cbe0;
}

.page-footer {
  padding-block: 0.5rem;
}
~~~

## 4. Run and explore

1. Start the app with `ng serve` and open the local page.
2. Toggle the theme and inspect the browser's local storage. The value should change between `light` and `dark`.
3. Type several characters quickly, then pause. The console should show only the final settled announcement because each previous timer was cleared.
4. Start the audit and switch users. The console should log each user change while the effect exists.
5. Stop the audit and switch users again. No further audit message should appear.
6. Compare the character count with the input. It updates through `computed()`, not through an effect that copies state.

## 5. Optional: update a DOM integration after rendering

A regular `effect()` is not a post-render DOM hook. For a chart or other imperative integration that must write to an element after Angular renders, use `afterRenderEffect()` and an explicit phase. For example, inside a component that owns a signal named `chartScale` and a canvas query:

~~~ts
import {
  afterRenderEffect,
  ElementRef,
  viewChild,
} from '@angular/core';

readonly canvas = viewChild.required<ElementRef<HTMLCanvasElement>>('canvas');
readonly chartScale = signal(1);

constructor() {
  afterRenderEffect({
    write: () => {
      this.canvas().nativeElement.style.setProperty(
        '--chart-scale',
        String(this.chartScale()),
      );
    },
  });
}
~~~

Use template bindings when possible. `afterRenderEffect()` runs only in the browser, and Angular does not guarantee that hydration has completed before its callback. For DOM measurements, separate reads and writes into the appropriate phases; use browser observers such as `ResizeObserver` when they better fit the task.

## 6. Compare a bad state-copying effect with the right derivation

Avoid this pattern:

~~~ts
readonly announcement = signal('Hello');
readonly announcementLength = signal(5);

effect(() => {
  this.announcementLength.set(this.announcement().length);
});
~~~

It stores the same fact twice and needs an effect-driven update. Use a computed signal instead:

~~~ts
readonly announcement = signal('Hello');
readonly announcementLength = computed(() => this.announcement().length);
~~~

## Expected behavior and concepts to notice

- Effects run at least once and track synchronous signal reads.
- The storage effect has one relevant dependency: `theme()`.
- The delayed-log effect tracks the latest announcement and cancels the previous timeout before it reruns.
- The audit effect is created later with an explicit injector and manually stopped through its `EffectRef`.
- The computed character count is derived state, so it does not require an effect.
- Browser-only APIs are guarded so the storage and timer effects do not access `window` during server rendering.

## Official references

- [Angular effects guide](https://angular.dev/guide/signals/effect)
- [Angular `effect()` API](https://angular.dev/api/core/effect)
- [Angular `afterRenderEffect()` API](https://angular.dev/api/core/afterRenderEffect)
- [Angular signals overview](https://angular.dev/guide/signals)
