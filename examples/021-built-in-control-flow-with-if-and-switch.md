# Examples: Built-in Control Flow with if and switch

This standalone Angular component demonstrates conditional branches with @if and @else if, a local alias, and a typed @switch. Buttons change the component state so you can observe which block Angular renders.

The example uses current built-in control flow syntax and does not import NgIf or NgSwitch.

## 1. Create the component class

Create src/app/control-flow-demo.ts:

~~~ts
import { Component } from '@angular/core';

type LoadState = 'loading' | 'ready' | 'error';

interface UserProfile {
  name: string;
  email: string;
}

@Component({
  selector: 'app-control-flow-demo',
  standalone: true,
  templateUrl: './control-flow-demo.html',
  styleUrl: './control-flow-demo.css',
})
export class ControlFlowDemo {
  user: UserProfile | null = {
    name: 'Mina Patel',
    email: 'mina@example.test',
  };

  readonly score = 82;
  readonly itemCount = 0;
  loadState: LoadState = 'loading';

  toggleUser(): void {
    this.user = this.user
      ? null
      : {
          name: 'Mina Patel',
          email: 'mina@example.test',
        };
  }

  showLoading(): void {
    this.loadState = 'loading';
  }

  showReady(): void {
    this.loadState = 'ready';
  }

  showError(): void {
    this.loadState = 'error';
  }
}
~~~

The LoadState union lists every expected status. Keeping the state typed as a finite set lets the template checker help find missing switch cases.

## 2. Add the template

Create src/app/control-flow-demo.html:

~~~html
<section class="demo-card" aria-labelledby="control-flow-title">
  <h2 id="control-flow-title">Account and data status</h2>

  <section aria-labelledby="profile-title">
    <h3 id="profile-title">Profile</h3>

    @if (user; as currentUser) {
      <p>Name: {{ currentUser.name }}</p>
      <p>Email: {{ currentUser.email }}</p>
    } @else {
      <p>No profile is currently selected.</p>
    }

    <button type="button" (click)="toggleUser()">
      Toggle profile
    </button>
  </section>

  <section aria-labelledby="score-title">
    <h3 id="score-title">Score: {{ score }}</h3>

    @if (score >= 90) {
      <p>Excellent result.</p>
    } @else if (score >= 70) {
      <p>Passing result.</p>
    } @else {
      <p>Keep practicing.</p>
    }
  </section>

  <section aria-labelledby="count-title">
    <h3 id="count-title">Items</h3>

    @if (itemCount >= 0) {
      <p>There are {{ itemCount }} items.</p>
    } @else {
      <p>The item count is unavailable.</p>
    }
  </section>

  <section aria-labelledby="load-state-title">
    <h3 id="load-state-title">Load state</h3>

    @switch (loadState) {
      @case ('loading') {
        <p role="status">Loading data...</p>
      }
      @case ('ready') {
        <p role="status">Data is ready.</p>
      }
      @case ('error') {
        <p role="alert">The data could not be loaded.</p>
      }
      @default never;
    }

    <div class="actions" role="group" aria-label="Choose a load state">
      <button type="button" (click)="showLoading()">Loading</button>
      <button type="button" (click)="showReady()">Ready</button>
      <button type="button" (click)="showError()">Error</button>
    </div>
  </section>
</section>
~~~

The profile block uses user as its condition and names the truthy value currentUser inside the block. The else block handles the null state.

The score uses ordered @if and @else if branches. The item count example deliberately checks itemCount >= 0 so the valid value zero is displayed.

The switch handles every LoadState member. The @default never; line asks the current Angular template checker to report a future union member if no case is added for it.

## 3. Add component styles

Create src/app/control-flow-demo.css:

~~~css
:host {
  display: block;
  max-width: 42rem;
  color: #1f2937;
}

.demo-card {
  display: grid;
  gap: 1rem;
  padding: 1.25rem;
  border: 1px solid #cbd5e1;
  border-radius: 0.75rem;
  background: #fff;
}

.demo-card section {
  padding-block: 0.5rem;
  border-block-start: 1px solid #e2e8f0;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

button {
  padding: 0.5rem 0.75rem;
  font: inherit;
  cursor: pointer;
}
~~~

## 4. Render the component

Import ControlFlowDemo into the root component and place its selector in the root template. For example, if the root component is src/app/app.ts:

~~~ts
import { Component } from '@angular/core';
import { ControlFlowDemo } from './control-flow-demo';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ControlFlowDemo],
  template: '<app-control-flow-demo />',
})
export class App {}
~~~

If the root component already has imports, providers, or a separate template, preserve them and add ControlFlowDemo and app-control-flow-demo.

Start the app:

~~~sh
ng serve
~~~

## 5. Try the branches

1. Select Toggle profile. The profile details are removed; select it again to render them.
2. Observe that the score of 82 selects the passing branch.
3. Observe that a count of zero is still displayed because the condition checks a range instead of truthiness.
4. Select Loading, Ready, and Error. Each value selects its matching switch case.
5. Add another member to LoadState without adding a corresponding @case. With a current Angular compiler and template type checking, @default never; should report the unhandled state.

The profile view is recreated when it returns. If the profile contained a child component with state that must survive removal, keep that state outside the conditional view.

## 6. Try ordinary defaults instead of exhaustive checking

For a switch whose value is not a TypeScript union, or when the project compiler does not support exhaustive checking, use a normal @default block:

~~~html
@switch (statusText) {
  @case ('ready') {
    <p>Ready.</p>
  }
  @default {
    <p>Status is not recognized.</p>
  }
}
~~~

Use @default never; only when the switch is a finite typed union and every allowed value has an explicit case.

## 7. Practice variations

1. Change score to 95, then 60, and confirm the ordered branches change.
2. Change itemCount to 1 and then 0. Confirm both values render.
3. Change itemCount to null by updating its TypeScript type, then create an explicit null check before displaying a numeric count.
4. Add a new status such as 'empty' to LoadState, handle it with an @case, and observe the exhaustive check pass again.
5. Change one switch case from the string '1' to the number 1 and compare it with a value of the other type.
6. Replace @default never; with a normal @default block and decide which form communicates the desired behavior.
7. Temporarily move a template local inside a conditional block and try to read it afterward. Angular template scoping should reject the out-of-scope reference.

## Compatibility note

Built-in @if and @switch blocks are available in modern Angular and require no NgIf or NgSwitch import. The @default never; exhaustive form depends on the Angular compiler and its template type-checking support. If it is not available in the project's version, use a normal @default block and verify all states through the project's checks.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="020-two-way-binding-and-ngmodel.md">020 - 20. Two-way binding and ngModel</a></td><td><a href="../lessons/021-built-in-control-flow-with-if-and-switch.md">Lesson</a></td><td><a href="../qnas/021-built-in-control-flow-with-if-and-switch.md">Q&amp;A</a></td><td><a href="022-render-collections-with-for-track-and-empty.md">022 - 22. Render collections with for, track, and empty</a></td>
</tr>
  </tbody>
</table>
