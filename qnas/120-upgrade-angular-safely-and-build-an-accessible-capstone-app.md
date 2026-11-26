# Q&A: Angular Upgrades and the Accessible Capstone

## 1. What should I check before an Angular upgrade?

Record the current Angular, CLI, Node.js, TypeScript, and RxJS versions. Check the official compatibility table for the target version and review the target release notes, update guide, and third-party package compatibility.

## 2. Why use the Angular Update Guide?

It provides version-specific steps for the current and target Angular releases. Upgrade instructions can change across major versions, so avoid applying a remembered command sequence without checking the guide for the versions in the project.

## 3. What does `ng update` do?

It reports or applies package updates and runs available migrations. Migrations can transform project source and configuration. Review the resulting diff and the package changes instead of assuming every change is complete or correct for the application's custom behavior.

## 4. Should Angular CLI and Angular core versions match?

Angular documents that core and CLI major versions are aligned. Keep the framework and CLI on a supported compatible release path and follow the Update Guide for the exact target.

## 5. Should I update directly across several major versions?

Follow the supported path recommended by the Angular Update Guide for the installed and target versions. For a large application, smaller reviewed steps are easier to diagnose and roll back than a single opaque change.

## 6. Why start from a clean Git working tree?

A clean baseline makes it clear which changes came from the upgrade, makes migration output reviewable, and provides a reliable rollback point. Do not casually allow a migration to run over unrelated uncommitted work.

## 7. Is a successful production build enough to approve an upgrade?

No. Compilation does not prove that routing, forms, hydration, browser behavior, API integration, styles, or user journeys still work. Run the project's configured tests and exercise critical flows in a representative environment.

## 8. Should I use `--force` when peer dependencies conflict?

Not as a routine fix. It can silence a compatibility warning without resolving it. Find which dependency has an incompatible peer range, update or replace it deliberately, or plan the Angular upgrade around its supported release.

## 9. Why should I review Angular migrations instead of reverting generated code?

A migration often makes a coordinated source or configuration change to preserve behavior or prepare the project for a new API. Understand the change and verify it against the app. Reverting it blindly can leave an incomplete upgrade.

## 10. What should an upgrade checklist include?

Compatibility review, a clean source-control baseline, the version-specific update guide, dependency and migration changes, type checking, linting, configured unit/integration/end-to-end tests, production and SSR builds if used, and a staging deployment.

## 11. What is the difference between Angular and AngularJS?

AngularJS refers to the legacy 1.x framework. Angular's current `ng update` and version guide concern modern Angular projects; AngularJS migration has its own process and should not be confused with a normal major-version update.

## 12. What does “semantic HTML first” mean for accessibility?

Choose elements whose built-in meaning and behavior match the control: buttons for actions, links for navigation, headings for section structure, and labels for inputs. Native elements provide keyboard and assistive-technology behavior that a generic clickable container does not provide automatically.

## 13. Does adding ARIA make a component accessible?

No. ARIA can expose roles, names, and states where native HTML does not provide them, but incorrect ARIA can make the experience worse. Use native HTML first and add ARIA only when needed to describe the interaction accurately.

## 14. How should a form show validation errors?

Give each input a programmatic label, associate help and error text with it, identify invalid fields in text, and explain how to correct them. On submission, provide a clear overall result and a sensible route to the first error. Avoid relying on color alone.

## 15. How do I make an Angular dialog keyboard accessible?

Use a tested dialog pattern that moves focus into the dialog, keeps focus within it when modal, supports keyboard dismissal where appropriate, and returns focus to the invoking control when it closes. Angular CDK focus tools can help, but the dialog still needs correct semantics and user-tested behavior.

## 16. Should the application remove the browser's focus outline?

No. Keyboard users need a visible indication of focus. If the design replaces the native outline, the replacement must be equally visible and remain clear across the site's backgrounds and states.

## 17. How should a single-page app communicate route changes?

Give each route a meaningful page title and heading, and choose a focus strategy that makes the new context apparent without disorienting users. Test with keyboard navigation and screen readers; a URL change alone may not announce the new page context.

## 18. Do accessibility scanners prove WCAG conformance?

No. Automated tools catch some issues, but they cannot fully judge meaningful labels, understandable instructions, logical focus, or whether a workflow works with assistive technology. Combine automated checks with manual review and user feedback.

## 19. Is Angular Material or CDK automatically accessible?

Angular Material aims to provide accessible reusable components, and CDK includes accessibility utilities. Correct configuration and use still matter, and the surrounding page, labels, content, focus flow, and product workflow must be reviewed.

## 20. What accessibility target should a capstone use?

State the product's intended standard and scope, such as the relevant WCAG 2.2 level, then evaluate the actual success criteria that apply. Do not claim conformance because the interface uses Angular, a component library, or a scanner.

## 21. How can I test keyboard accessibility?

Complete the full workflow without a mouse. Check that every action is reachable, focus order is logical, focus is visible and not obscured, dialogs can be entered and exited, and no keyboard trap occurs.

## 22. How should offline and error states be designed accessibly?

Use plain text to explain what happened and what the user can do next. Announce important asynchronous results when needed, identify stale data as stale, and do not use color or an icon as the only status signal.

## 23. When is a capstone complete?

When its stated core journeys work, server permissions are enforced, errors and empty states are understandable, accessibility has been reviewed through manual and automated methods, and another developer can install, run, check, and build the project from its documentation. It can still need future maintenance.

## 24. Does an accessible component guarantee an accessible application?

No. Accessibility depends on the complete page and journey, including content, labels, focus transitions, validation, responsive behavior, third-party widgets, and backend error handling. Review shared components and end-to-end workflows.

## Chapter navigation

<table width="100%">
  <thead>
    <tr>
<th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th>
</tr>
  </thead>
  <tbody>
    <tr>
<td><a href="119-service-workers-offline-behavior-deployment-and-updates.md">119 - Service Workers, Offline Behavior, Deployment, and Updates</a></td><td><a href="../lessons/120-upgrade-angular-safely-and-build-an-accessible-capstone-app.md">Lesson</a></td><td><a href="../examples/120-upgrade-angular-safely-and-build-an-accessible-capstone-app.md">Example</a></td><td>End of course</td>
</tr>
  </tbody>
</table>
