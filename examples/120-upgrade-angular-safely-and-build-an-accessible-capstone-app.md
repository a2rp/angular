# Examples: Angular Upgrades and the Accessible Capstone

This chapter closes the roadmap with a safe upgrade workflow and an accessible service-request tracker. Replace illustrative API and design tokens with the project's real services and tested design system.

## 1. Capture a reproducible upgrade baseline

Start from a clean branch and record the versions and available updates:

~~~bash
git status --short
ng version
ng update
~~~

Before applying an update, check the Angular version table and generate the version-specific steps in the Angular Update Guide. Apply the exact command it recommends; package versions and compatible Node.js/TypeScript ranges vary by release.

After the CLI migrations finish, inspect the changes and verify the working tree:

~~~bash
git diff -- package.json package-lock.json angular.json tsconfig.json
git diff --check
~~~

Then run the checks configured by the project, build the production and SSR targets if they exist, and exercise critical routes and forms. Do not use `--force` to hide peer-dependency problems; identify and resolve the incompatibility first.

## 2. Use semantic structure for a request list

Represent a collection as a heading and list. Make the request title a link to the detail route and express status with text so color is not the only signal.

~~~html
<main id="main-content">
  <h1>Service requests</h1>
  <p>Search requests or filter them by status.</p>

  <label for="status-filter">Filter by status</label>
  <select id="status-filter" name="status">
    <option value="all">All statuses</option>
    <option value="open">Open</option>
    <option value="in-progress">In progress</option>
    <option value="closed">Closed</option>
  </select>

  <p aria-live="polite">Showing 2 requests.</p>
  <ul>
    <li>
      <h2><a routerLink="/requests/req-101">Broken street light</a></h2>
      <p>Status: Open</p>
      <p>Submitted 24 November 2026</p>
    </li>
    <li>
      <h2><a routerLink="/requests/req-102">Damaged sidewalk</a></h2>
      <p>Status: In progress</p>
      <p>Submitted 22 November 2026</p>
    </li>
  </ul>
</main>
~~~

In the actual component, bind the list to application data, provide an empty state when no requests match, and verify that live result announcements are helpful rather than repeated excessively. Include a skip link near the start of the page that targets the main content.

## 3. Build a labeled form with associated validation messages

This standalone component demonstrates a title and description form. It assumes an injected `RequestApiService.create()` method that returns an Observable from the backend. The server remains responsible for validation, authorization, and persistence.

~~~ts
import { Component, inject } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';
import { RequestApiService } from './request-api.service';

@Component({
  selector: 'app-request-form',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <main>
      <h1>Submit a service request</h1>
      <p id="form-help">Describe the issue. Do not include passwords or other private information.</p>

      @if (formError) {
        <p role="alert">{{ formError }}</p>
      }

      <form [formGroup]="form" (ngSubmit)="submit()" aria-describedby="form-help">
        <div>
          <label for="request-title">Title</label>
          <p id="title-help">Use a short description, up to 80 characters.</p>
          <input
            id="request-title"
            type="text"
            formControlName="title"
            maxlength="80"
            [attr.aria-invalid]="titleHasError ? 'true' : null"
            [attr.aria-describedby]="titleHasError ? 'title-help title-error' : 'title-help'">
          @if (titleHasError) {
            <p id="title-error">Enter a title for this request.</p>
          }
        </div>

        <div>
          <label for="request-description">Description</label>
          <p id="description-help">Explain where the issue is and what you observed.</p>
          <textarea
            id="request-description"
            formControlName="description"
            rows="5"
            maxlength="1000"
            [attr.aria-invalid]="descriptionHasError ? 'true' : null"
            [attr.aria-describedby]="descriptionHasError ? 'description-help description-error' : 'description-help'"></textarea>
          @if (descriptionHasError) {
            <p id="description-error">Enter a description for this request.</p>
          }
        </div>

        <button type="submit" [disabled]="saving">{{ saving ? 'Sending…' : 'Submit request' }}</button>
        <p role="status" aria-live="polite">{{ statusMessage }}</p>
      </form>
    </main>
  `,
})
export class RequestFormComponent {
  private readonly formBuilder = inject(FormBuilder);
  private readonly api = inject(RequestApiService);

  readonly form = this.formBuilder.nonNullable.group({
    title: ['', [Validators.required, Validators.maxLength(80)]],
    description: ['', [Validators.required, Validators.maxLength(1000)]],
  });

  submitted = false;
  saving = false;
  formError = '';
  statusMessage = '';

  get titleHasError(): boolean {
    const control = this.form.controls.title;
    return control.invalid && (control.touched || this.submitted);
  }

  get descriptionHasError(): boolean {
    const control = this.form.controls.description;
    return control.invalid && (control.touched || this.submitted);
  }

  submit(): void {
    this.submitted = true;
    this.formError = '';

    if (this.form.invalid) {
      this.form.markAllAsTouched();
      this.formError = 'Some information is missing. Review the fields marked below.';
      return;
    }

    this.saving = true;
    this.statusMessage = 'Sending request…';

    this.api.create(this.form.getRawValue()).subscribe({
      next: () => {
        this.form.reset();
        this.submitted = false;
        this.statusMessage = 'Your request was submitted.';
        this.saving = false;
      },
      error: () => {
        this.formError = 'The request could not be sent. Check your connection and try again.';
        this.statusMessage = '';
        this.saving = false;
      },
    });
  }
}
~~~

For a larger form, provide an error summary with links to invalid controls and move focus to that summary or the first invalid field after a failed submission. Do not clear entered values on an error. Confirm the API response before announcing that the request was submitted.

## 4. Keep focus visible in the design system

Use a design token that has been checked against the relevant backgrounds and states. Do not remove the focus indicator without an equally visible replacement.

~~~css
:where(a, button, input, select, textarea, [tabindex]):focus-visible {
  outline: 3px solid var(--focus-color);
  outline-offset: 3px;
}

@media (max-width: 40rem) {
  .request-grid {
    grid-template-columns: 1fr;
  }
}
~~~

Test the actual color token for contrast and ensure sticky navigation does not cover a focused control. A media query alone does not make the whole page responsive; verify reflow and readable controls at increased zoom.

## 5. Make the upgrade change reviewable

A team might divide a major upgrade into these commits on a dedicated branch:

1. Update Angular CLI and core using the official guide.
2. Update UI, testing, and build packages with compatible releases.
3. Apply any application refactors required by migrations.
4. Remove deprecated APIs only after behavior is covered.
5. Record follow-up work that cannot safely be completed in the same change.

For every commit, include the changed package range, migration summary, checks run, and any known issue. Keep the lockfile with the package update so another developer can reproduce the dependency graph.

## 6. Capstone verification checklist

### Functional behavior

- List, filter, detail, create, and edit routes behave as described.
- Empty, loading, offline, not-found, and server-error states are visible and understandable.
- A request is announced as saved only after the backend confirms it.
- The backend checks the current user's access to each protected record.

### Accessibility

- Every workflow works with a keyboard and has a visible, logical focus order.
- Labels, instructions, errors, status messages, and headings are meaningful to assistive technology.
- No information depends only on color, position, or icon shape.
- Dialogs and route changes have deliberate focus behavior.
- The layout remains usable when text is enlarged and on a narrow viewport.
- A screen reader review and automated accessibility checks have both been completed.

### Security and reliability

- Frontend assets contain no confidential API credentials.
- Server authorization is enforced independently of Angular guards.
- User-specific API data is not cached by service-worker rules by accident.
- Offline states distinguish cached information from current server data.
- The production build and deployment assets come from a reproducible install.

### Upgrade maintenance

- The upgrade path matched the official guide and compatibility table.
- Migrations and dependency changes were reviewed.
- Critical route, form, API, and rendering flows were checked after the upgrade.
- The release was exercised in a representative staging environment.

## Further reading

- [Angular Update Guide](https://angular.dev/update-guide)
- [Angular CLI `ng update`](https://angular.dev/cli/update)
- [Angular version compatibility](https://angular.dev/reference/versions)
- [Angular accessibility best practices](https://angular.dev/best-practices/a11y)
- [Angular CDK accessibility overview](https://material.angular.dev/cdk/a11y/overview)
- [W3C WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)
