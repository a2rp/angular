# Test Routing, Forms, Accessibility, and User Flows — Examples

This example tests one short user journey: navigate to a profile edit page, submit an invalid name, correct it, and confirm the success state. It uses a test route and a real rendered form, while keeping the result deterministic and local.

## 1. Build a routed form component

~~~ts
// profile-edit-page.ts
import { Component, signal } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';

@Component({
  selector: 'app-profile-edit-page',
  imports: [ReactiveFormsModule],
  templateUrl: './profile-edit-page.html'
})
export class ProfileEditPage {
  private readonly formBuilder = new FormBuilder().nonNullable;

  readonly form = this.formBuilder.group({
    displayName: ['', [Validators.required, Validators.minLength(2)]]
  });

  readonly saved = signal(false);

  save(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    this.saved.set(true);
  }
}
~~~

~~~html
<!-- profile-edit-page.html -->
<main>
  <h1>Profile settings</h1>

  <form (ngSubmit)="save()">
    <label for="display-name">Display name</label>
    <input
      id="display-name"
      formControlName="displayName"
      [attr.aria-invalid]="form.controls.displayName.invalid && form.controls.displayName.touched ? 'true' : null"
      [attr.aria-describedby]="form.controls.displayName.invalid && form.controls.displayName.touched ? 'name-error' : null"
    />

    @if (form.controls.displayName.invalid && form.controls.displayName.touched) {
      <p id="name-error" role="alert">
        Enter a display name with at least two characters.
      </p>
    }

    <button type="submit">
      Save profile
    </button>
  </form>

  @if (saved()) {
    <p role="status">Profile saved.</p>
  }
</main>
~~~

The form has a visible label. Its error is text, the control exposes invalid state, and the error is associated with the input. This example keeps submit enabled so the user can submit and receive validation feedback.

## 2. Configure the router and create the harness

~~~ts
// profile-edit-page.spec.ts
import { ReactiveFormsModule } from '@angular/forms';
import { TestBed } from '@angular/core/testing';
import { RouterTestingHarness } from '@angular/router/testing';
import { provideRouter } from '@angular/router';
import { beforeEach, describe, expect, it } from 'vitest';
import { ProfileEditPage } from './profile-edit-page';

describe('profile edit flow', () => {
  let harness: RouterTestingHarness;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [ProfileEditPage],
      providers: [
        provideRouter([
          { path: 'profile/edit', component: ProfileEditPage }
        ])
      ]
    }).compileComponents();

    harness = await RouterTestingHarness.create();
  });
~~~

ReactiveFormsModule is already imported by the component itself; the test does not need to add it separately to TestBed. The routed component declares its template dependencies.

## 3. Submit an invalid value through the form

~~~ts
  it('shows an associated validation message for a short name', async () => {
    await harness.navigateByUrl('/profile/edit', ProfileEditPage);
    const host = harness.routeNativeElement;
    if (!host) {
      throw new Error('Profile page did not render.');
    }

    const input = host.querySelector<HTMLInputElement>('#display-name');
    const form = host.querySelector<HTMLFormElement>('form');
    if (!input || !form) {
      throw new Error('Expected form controls were not rendered.');
    }

    input.value = 'A';
    input.dispatchEvent(new Event('input', { bubbles: true }));
    input.dispatchEvent(new Event('blur'));
    form.dispatchEvent(new Event('submit', { bubbles: true, cancelable: true }));
    harness.detectChanges();

    expect(input.getAttribute('aria-invalid')).toBe('true');
    expect(input.getAttribute('aria-describedby')).toBe('name-error');
    expect(host.textContent).toContain('at least two characters');
  });
~~~

This test uses the rendered input and submit event rather than calling a private validator. The accessible relationship is asserted along with the visible message.

## 4. Correct the value and complete the flow

~~~ts
  it('saves after the user enters a valid name', async () => {
    const page = await harness.navigateByUrl(
      '/profile/edit',
      ProfileEditPage
    );
    const host = harness.routeNativeElement;
    if (!host) {
      throw new Error('Profile page did not render.');
    }

    const input = host.querySelector<HTMLInputElement>('#display-name');
    if (!input) {
      throw new Error('Display name input was not rendered.');
    }

    input.value = 'Grace';
    input.dispatchEvent(new Event('input', { bubbles: true }));
    harness.detectChanges();

    const button = host.querySelector<HTMLButtonElement>('button[type="submit"]');
    if (!button) {
      throw new Error('Save button was not rendered.');
    }

    button.click();
    await harness.fixture.whenStable();

    expect(page.saved()).toBe(true);
    expect(host.textContent).toContain('Profile saved.');
    expect(host.querySelector('[role="alert"]')).toBeNull();
  });
});
~~~

The route harness creates the page using application routing, while the test controls form input and checks the result. This demonstrates a short integration flow; it does not contact a backend.

## 5. Add broader checks when required

For a production form, add focused tests for any server failure, disabled or pending state, and retry behavior. Review the flow with keyboard navigation and a screen reader. Automated DOM assertions can check roles and relationships but cannot fully confirm spoken feedback.

## Further reading

- [Angular routing testing](https://angular.dev/guide/routing/testing)
- [RouterTestingHarness API](https://angular.dev/api/router/testing/RouterTestingHarness)
- [Angular form validation](https://angular.dev/guide/forms/form-validation)
- [Angular accessibility best practices](https://angular.dev/best-practices/a11y)
