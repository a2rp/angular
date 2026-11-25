# Feature-Based Application Architecture — Q&A

## 1. What does feature-based architecture mean in Angular?

It means grouping code by the product capability or workflow it supports, such as orders, inventory, or account management. Components, routes, data access, and models that belong to one capability can live together. Angular does not require this folder convention; it is an organizational choice that should make code easier to find and change.

## 2. Should every application use the same folder structure?

No. A small application may only need a few folders. A larger application may split a feature into pages, data access, and UI areas. Add structure when it solves a real discovery, ownership, or dependency problem. Avoid creating many empty layers just to follow a diagram.

## 3. Why can global components, services, and models folders become difficult to maintain?

They group files by technical type rather than by the workflow they serve. Over time, each directory can contain unrelated code from many features. A developer changing one capability must search across several folders, and files that change together are separated.

## 4. When should code go in a shared folder?

Put code in shared when multiple independent features use it and it has a stable, broadly understood purpose. A reusable date formatter or a presentation-only status badge may qualify. Code should not be moved to shared merely because two components currently import it; feature-specific behavior should remain near its owner.

## 5. What belongs in a feature folder?

Usually pages and routes for that capability, feature-specific UI, models, data access, and logic used only by the feature. Exact subfolders depend on size. Keep the structure shallow until the feature becomes hard to navigate.

## 6. What is the difference between core and shared code?

Core commonly refers to application-wide infrastructure with a broad lifetime, such as authentication setup, global HTTP configuration, or error handling. Shared commonly refers to small reusable code or UI consumed by multiple features. These labels are conventions, so document the meaning your team assigns to them.

## 7. Does lazy loading automatically make an application well-architected?

No. Lazy loading changes when route code is loaded. It does not establish ownership, prevent circular dependencies, or make a feature API clear by itself. A lazy feature still needs sensible boundaries and loading, empty, and error states.

## 8. How should two features coordinate without importing each other's internals?

Define a narrow contract or let a higher-level page or route compose both features. Avoid direct mutual imports, which create cycles and make changes ripple across boundaries. If the behavior is genuinely shared, extract a small stable API with a clear owner.

## 9. Where should a service be provided?

Choose the injector scope based on lifetime and ownership. Root scope is appropriate for application-wide infrastructure or state that should be shared throughout the app. Route or component scope can be better for state that should exist only while a feature instance is active. Folder placement alone does not determine provider scope.

## 10. Should every component, service, model, and helper have its own file?

One main concept per file is a useful default because it improves discoverability and makes files easier to review. Small closely related types can sometimes live together. Split a file when its responsibilities or size make it difficult to understand, not just to satisfy a rigid file-count rule.

## 11. Should tests be collected in one top-level tests directory?

Keep focused tests near the code they exercise when that makes the relationship easy to see. A large end-to-end suite may have its own location because it tests full user journeys rather than one feature unit. The goal is discoverability and an intentional test boundary.

## 12. How can a team tell whether its architecture is helping?

A developer should be able to answer where a feature begins, who owns its state, how it talks to other features, and where to find its focused tests. If changes require searching many unrelated folders or cross-feature imports keep forming cycles, revisit the boundaries. Avoid judging architecture by folder count alone.

## Further reading

- [Angular style guide](https://angular.dev/style-guide)
- [Angular workspace and file structure](https://angular.dev/reference/configs/file-structure)
- [Angular routing guide](https://angular.dev/guide/routing)
