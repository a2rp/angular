# Upgrade Angular Safely and Build an Accessible Capstone App

A sustainable Angular project stays current through deliberate dependency upgrades, migration review, and regression checks. A complete learning project should also bring its framework skills together in a usable application that works for people using keyboards, screen readers, zoom, and different input methods.

This final chapter covers two practical workflows: upgrading an existing Angular workspace with the official update tools, and planning an accessible capstone whose quality can be reviewed against clear requirements.

## Learning outcomes

- Plan an Angular upgrade using the official compatibility and update guidance.
- Apply migrations in a clean, reviewable source-control change.
- Diagnose compatibility issues without bypassing them blindly.
- Design an Angular capstone around semantic HTML and accessible interaction.
- Define functional, accessibility, security, and deployment acceptance criteria.
- Review a finished project with automated checks and manual assistive-technology use.

## 1. Prepare before changing versions

Start from a clean, committed working tree. Record the current Angular, CLI, Node.js, TypeScript, and RxJS versions with `ng version` and inspect `package.json` and the lockfile. Check Angular's version-compatibility table for the Node.js and TypeScript ranges required by both the current and target versions.

Use the interactive Angular Update Guide with the exact current and target versions. Read the target release notes and migration notes for Angular and important ecosystem packages. Confirm that build tools, test runners, UI libraries, and third-party Angular integrations support the target version.

Plan a restore point before applying migrations. A source-control commit or branch gives the team a reviewable rollback path; a lockfile and reproducible install help restore the dependency graph.

## 2. Apply official updates and migrations

The Angular CLI can report available updates:

~~~bash
ng update
~~~

A standard update of the framework and CLI uses:

~~~bash
ng update @angular/cli @angular/core
~~~

For a specific target major, follow the exact command and step sequence supplied by the Angular Update Guide. The CLI runs package updates and available code migrations. Angular recommends taking the latest patch release in a chosen major because it contains fixes released after the initial major version.

Do not use `--force` as a routine way to silence peer-dependency conflicts. Determine which packages are incompatible, update or replace them deliberately, and document any package that must wait. Do not edit generated migration results mechanically without understanding the change.

For a large migration, use small source-control commits: framework/CLI changes, third-party library changes, application refactors, and follow-up cleanup. Review the full diff, especially config files, generated code, router behavior, forms, SSR, and tests.

## 3. Verify the upgraded application

Use the checks appropriate for the project:

1. Install from the lockfile in a clean environment.
2. Run the type checker, lint rules, unit tests, and integration or end-to-end suites configured by the project.
3. Build all relevant configurations, including production, SSR, or prerender targets if present.
4. Exercise critical user flows and compare the output with the previous release.
5. Review browser console and server logs for deprecations, hydration issues, and runtime failures.
6. Check generated bundle size and deployment artifacts when the upgrade changes the build system.
7. Deploy to a representative staging environment before production.

A passing compilation is not proof of behavior. Migrations can change runtime timing, styles, routing, package compatibility, or server rendering. Keep focused regression coverage for the product's most important journeys.

## 4. Capstone: Accessible Service Request Tracker

Build a small application where a person can browse service requests, filter the list, open a request, submit a new request, and view its status. Use a domain that lets the project demonstrate routing, reusable components, forms, HTTP services, error/loading states, and responsive layouts without hiding the fundamentals behind a template.

A possible route map:

| Route | Purpose |
| --- | --- |
| `/requests` | Searchable, filterable request list |
| `/requests/:id` | Request details and status history |
| `/requests/new` | Submit a new request |
| `/requests/:id/edit` | Edit an authorized request |
| `/help` | Explain the service and how to get support |

Keep responsibilities clear: route pages compose features; focused components render controls; services communicate with APIs; typed models describe the data. Use a backend for persistence and authorization. The Angular app should not claim that a request was saved until the server confirms it.

## 5. Build accessibility into the structure

Use semantic HTML first: headings in a meaningful hierarchy, landmarks, lists for collections, buttons for actions, and links for navigation. A native button already supports keyboard activation and focus behavior. Avoid clickable `div` elements when a button or link expresses the intended action.

For every form control, provide a programmatic label. Associate help and error text with the relevant input, set an invalid state when appropriate, and explain how to correct the problem in text. Do not communicate status only with color or icon shape.

Ensure all workflows are keyboard operable, focus is visible, focus is not trapped, and focused items are not hidden by sticky headers or dialogs. For dialogs, manage focus deliberately and return focus to the invoking control when the dialog closes. Angular CDK accessibility tools can help with patterns such as focus trapping, but the component still needs correct semantics and behavior.

Use accessible names for icon-only controls. Announce important asynchronous results, such as successful submission or a filter result count, with an appropriate live region when a screen reader would otherwise miss the change. After route navigation, provide a meaningful page title and a sensible focus strategy so users understand the new context.

Support text resizing and narrow viewports. Check text and non-text contrast, visible focus, target spacing, and reflow. Keep focus indicators visible; do not remove the browser outline without an equally clear replacement.

## 6. Capstone acceptance criteria

Write acceptance criteria before implementation. For example:

- A visitor can find a request by text and status without losing keyboard position unexpectedly.
- A request detail page has a unique heading, status, readable history, and useful not-found state.
- The create form has labels, help text, field-level errors, and a clear submission result.
- Errors are identified in text and associated with the fields that need correction.
- Loading, empty, offline, and server-error states are distinguishable and do not expose internal details.
- All interactive actions work without a mouse, and focus remains visible and logical.
- The layout remains usable at increased zoom and on a narrow viewport.
- The backend checks the current user and resource permissions on every protected action.
- Private or user-specific API responses are not cached by the service worker by accident.
- The app provides a clear not-found route and does not leak credentials or sensitive data into URLs or logs.

The target accessibility standard should be explicit for the product. WCAG 2.2 provides testable success criteria, but no framework or automated scanner makes an application conformant by itself.

## 7. Review with tools and people

Automated accessibility checks can catch some missing names, invalid attributes, and contrast problems. They cannot fully assess whether instructions make sense, a workflow is understandable, focus moves appropriately, or a screen-reader announcement is useful.

Combine automated checks with manual review:

1. Complete every route and form using only the keyboard.
2. Check visible focus, logical tab order, and escape from dialogs or menus.
3. Use a screen reader to inspect headings, landmarks, labels, errors, and dynamic status updates.
4. Test zoom, text resizing, reflow, contrast, and non-color status cues.
5. Ask people with varied access needs to evaluate important workflows when possible.
6. Fix issues by user impact, and repeat checks after changing shared components.

Maintain an issue list with the affected workflow, observed barrier, expected behavior, and verified fix. This makes accessibility an ongoing engineering responsibility rather than a final visual pass.

## 8. Finish and maintain the project

Document the supported Angular version, how to install and run the project, how to run the configured checks, and how to build and deploy it. Keep dependencies current, review deprecations, and schedule upgrades before support windows end. When a new Angular major becomes available, repeat the same compatibility, migration, and regression process.

A capstone is complete when the core workflows work, error states are understandable, access is enforced by the backend, accessibility has been checked with people and tools, and the build can be reproduced and deployed. It can still evolve as product needs and standards change.

## Key takeaways

- Check Angular compatibility and follow the version-specific Update Guide.
- Apply migrations in source control and review the generated diff.
- Verify runtime behavior, not only compilation.
- Build semantic, keyboard-operable, screen-reader-friendly interfaces from the start.
- Use Angular and CDK accessibility features as support, not as a substitute for review.
- Define measurable capstone acceptance criteria and test with people and tools.
- Keep upgrades, accessibility, security, and deployment as ongoing project work.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="119-service-workers-offline-behavior-deployment-and-updates.md">119 - Service Workers, Offline Behavior, Deployment, and Updates</a></td><td><a href="../qnas/120-upgrade-angular-safely-and-build-an-accessible-capstone-app.md">Q&amp;A</a></td><td><a href="../examples/120-upgrade-angular-safely-and-build-an-accessible-capstone-app.md">Example</a></td><td>End of course</td>
</tr>
  </tbody>
</table>
