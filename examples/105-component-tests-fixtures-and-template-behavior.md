# Component Tests, Fixtures, and Template Behavior - Examples

This example renders a name editor in Angular's test environment and verifies visible behavior. It sets up a ComponentFixture, dispatches a real input event, checks the updated template, and observes an output.

## 1. Create a focused component

~~~ts
// name-editor.ts
import { Component, output, signal } from '@angular/core';

@Component({
  selector: 'app-name-editor',
  template: '<section><label for="name-input">Display name</label><input id="name-input" [value]="name()" (input)="updateName($event)" /><p aria-live="polite">Preview: {{ name() }}</p><button type="button" (click)="clearName()">Clear name</button></section>'
})
export class NameEditor {
  readonly name = signal('Ada');
  readonly nameChanged = output<string>();

  updateName(event: Event): void {
    const input = event.target;

    if (!(input instanceof HTMLInputElement)) {
      return;
    }

    this.name.set(input.value);
    this.nameChanged.emit(input.value);
  }

  clearName(): void {
    this.name.set('');
    this.nameChanged.emit('');
  }
}
~~~

## 2. Configure the fixture and check the initial view

~~~ts
// name-editor.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { beforeEach, describe, expect, it, vi } from 'vitest';
import { NameEditor } from './name-editor';

describe('NameEditor', () => {
  let fixture: ComponentFixture<NameEditor>;
  let component: NameEditor;
  let host: HTMLElement;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [NameEditor]
    }).compileComponents();

    fixture = TestBed.createComponent(NameEditor);
    component = fixture.componentInstance;
    host = fixture.nativeElement;
    fixture.detectChanges();
  });

  it('shows the initial name in the preview', () => {
    const preview = host.querySelector('p');

    expect(preview?.textContent).toContain('Preview: Ada');
  });
~~~

The component uses an inline template, so compileComponents is often not necessary, but keeping asynchronous setup is useful when a test uses external resources or configuration that must compile first.

## 3. Dispatch an input event and verify the rendered result

~~~ts
  it('updates the preview when the user types', () => {
    const input = host.querySelector<HTMLInputElement>('#name-input');

    if (!input) {
      throw new Error('Name input was not rendered.');
    }

    input.value = 'Grace';
    input.dispatchEvent(new Event('input'));
    fixture.detectChanges();

    const preview = host.querySelector('p');
    expect(preview?.textContent).toContain('Preview: Grace');
  });
~~~

Setting input.value alone is not enough. Dispatching the input event lets the component's template listener receive the changed value.

## 4. Observe the public output and use the rendered button

~~~ts
  it('emits the changed name', () => {
    const changed = vi.fn();
    component.nameChanged.subscribe(changed);

    const input = host.querySelector<HTMLInputElement>('#name-input');

    if (!input) {
      throw new Error('Name input was not rendered.');
    }

    input.value = 'Grace';
    input.dispatchEvent(new Event('input'));
    fixture.detectChanges();

    expect(changed).toHaveBeenCalledWith('Grace');
  });

  it('clears the name when the user activates the button', () => {
    const button = host.querySelector<HTMLButtonElement>('button');

    if (!button) {
      throw new Error('Clear button was not rendered.');
    }

    button.click();
    fixture.detectChanges();

    expect(host.querySelector('p')?.textContent).toContain('Preview:');
    expect(component.name()).toBe('');
  });
});
~~~

These tests exercise the component through its template and public output. They do not call a private handler directly. For a larger shared component, consider an Angular component harness so tests do not depend on internal selectors.

## 5. Understand the fixture lifecycle

Each test gets a fixture created by TestBed. Use fixture.detectChanges when the test needs an explicit render pass. Use await fixture.whenStable when the behavior includes asynchronous work or pending updates. The test framework normally destroys fixtures between tests, but manually created timers, subscriptions, or external resources still need clear cleanup.

## 6. Run the component tests

~~~sh
ng test
~~~

This is illustrative test content; adapt the runner imports to the project's configured test environment. Vitest is the default in new Angular CLI projects, while older workspaces may use Karma and Jasmine.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="104-angular-testing-fundamentals-and-the-cli-test-runner.md">104 - Angular Testing Fundamentals and the CLI Test Runner</a></td><td><a href="../lessons/105-component-tests-fixtures-and-template-behavior.md">Lesson</a></td><td><a href="../qnas/105-component-tests-fixtures-and-template-behavior.md">Q&amp;A</a></td><td><a href="106-test-services-dependency-injection-and-http-requests.md">106 - Test Services, Dependency Injection, and HTTP Requests</a></td>
</tr>
  </tbody>
</table>
