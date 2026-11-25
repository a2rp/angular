# State ownership and avoiding duplicated state — Q&A

## 1. What does state ownership mean?

State ownership means one clearly identified part of the application is responsible for the authoritative value and the rules for changing it. Other parts can observe that value or request a change from its owner.

## 2. Why should a fact usually have one owner?

If separate parts of the UI can independently change copies of the same fact, those copies can disagree. A single owner reduces synchronization work and makes it clear where validation and invariants belong.

## 3. Does “one owner” mean there can only be one copy of a value in memory?

No. Frameworks and the browser may create internal snapshots, caches, or rendered values. The design goal is to avoid multiple independently writable application-level copies that are supposed to mean the same thing.

## 4. What is canonical state?

Canonical state is the authoritative representation of a fact the application needs to remember. For a task board, a task collection and the selected task ID may be canonical state. Counts and filtered views can be derived from them.

## 5. What is derived state?

Derived state is a value calculated from other state, such as a total, a filtered list, a full name, or the selected record looked up by ID. If the value has no independent meaning, calculate it instead of storing a separately writable copy.

## 6. Should I store a filtered list in a signal?

Usually not if it can be calculated from a source list and a filter. Use a `computed()` signal so changes to either dependency update the result automatically. Store a separate list only when it represents a distinct fact, such as a server-maintained snapshot with its own lifecycle.

## 7. Should I store a count when I already have the collection?

If the count is simply the length of the collection or a filter over it, derive it with `computed()`. A separately writable count can drift out of sync. A cached or server-provided aggregate can be stored when it has its own semantics and invalidation strategy.

## 8. Why store a selected ID instead of the selected object?

An ID is stable when an object is replaced during an immutable update. Store the ID and derive the current object from the latest collection. A separately stored object may point to an old snapshot after the collection changes.

## 9. Is storing a selected object always wrong?

No. It can be fine for short-lived local state when the object is not also maintained elsewhere or when snapshot semantics are intentional. Prefer an ID when the object is updated elsewhere and the UI should always show the latest version.

## 10. What if the selected ID no longer exists?

Define the behavior explicitly. The derived lookup can return `null`; the owner can clear the selection when removing the record; or a dependent selection can choose a valid fallback. The template should handle the empty state.

## 11. When should I use `computed()`?

Use `computed()` for a read-only value derived from signals. It is lazy and memoized, and Angular tracks the signals it reads. Do not use it for a value that must be independently editable.

## 12. When should I use `linkedSignal()`?

Use `linkedSignal()` when a value is based on another signal but users or application code must also be able to set it. It can recalculate when its source changes while retaining writable behavior between source changes.

## 13. How is `linkedSignal()` different from copying state in an effect?

`linkedSignal()` explicitly describes a writable value that depends on a source. An effect that copies one signal into another adds an asynchronous synchronization step and a second writable value, which can get out of sync or form a cycle.

## 14. When should a linked value reset?

That is a product rule. Reset when the old choice should no longer carry meaning, preserve it when it remains valid, or clear it when no valid choice exists. Put the rule in the linked computation or the state owner's transition method.

## 15. How can a `linkedSignal()` keep a current selection if it still exists?

Use its source-and-computation form. The computation receives the new source and the previous source/value, so it can keep the prior ID if the new options still contain it; otherwise it can choose a fallback or `null`.

## 16. Are effects a way to derive values?

No. Prefer `computed()` for read-only derivation and `linkedSignal()` for writable dependent state. Effects are meant for appropriate side effects that interact with non-signal APIs, such as logging, storage, or a third-party imperative widget.

## 17. Can I use an effect to copy an input into a local signal?

Avoid that as a default. First decide whether the child should use the input directly, derive a view from it, or maintain a genuinely separate editable draft. If the child needs a writable value linked to the input, `linkedSignal()` may fit. Do not create a mirror only to make the value writable.

## 18. How should a parent and child share an editable value?

Usually the parent owns the value, passes it through an input, and handles a child output that requests a change. A `model()` input is appropriate when the child is intentionally designed as a two-way editable control. Choose one clear contract and avoid an unrelated local mirror.

## 19. Is a form draft duplicated state?

It can be intentional state with a different meaning. The saved record is the committed value; the draft is an in-progress edit that may be saved or discarded. Define how it is initialized, submitted, cancelled, and reconciled with background updates.

## 20. Should every keystroke update the canonical server-backed record?

Not necessarily. For immediate-edit interfaces, that can be appropriate if updates and failures are handled. For forms with Save and Cancel, keep a draft and commit it on save. The interaction determines the ownership boundary.

## 21. Where should a local filter live?

If only one component uses it, keep it in that component. If multiple parts of a feature must coordinate around the same filter, move ownership to their common parent or a feature-scoped state service. Derive the visible list from the filter and source collection.

## 22. When should state move into a service?

Move state into a service when it needs a non-component owner, is shared across several components, or has transitions and data access that belong to a feature. Do not move every local toggle or temporary display detail into a global service.

## 23. What does `providedIn: 'root'` mean for state?

It makes a service available from the application's root injector, so consumers resolving it from that injector normally receive the same instance. That is useful for intentionally app-wide state but gives that state a broad lifetime.

## 24. What happens when a component provides a service?

Angular creates or uses a service instance in that component's injector. Its descendants can resolve that instance, while another component with a separate provider can get a different one. This is useful for isolated feature instances and incorrect when unrelated component subtrees were expected to share state.

## 25. How can two sibling components share one feature state instance?

Provide the service at a common ancestor, route, or feature injector above both siblings, and do not provide another copy in either child. If the state should be shared application-wide, a root provider may be suitable.

## 26. Should all services be root-provided?

No. Select scope according to lifetime and sharing needs. Component or route scope can prevent state from leaking between separate feature instances and can make cleanup more natural.

## 27. How do I avoid duplicated entities across screens?

Keep one normalized collection or data-access owner for the feature and refer to records by stable IDs. Derive list rows and selected details from that collection. If the app uses a query cache, treat its cache/update policy as the owner rather than maintaining an unrelated second store.

## 28. What if a server response updates data while a form is being edited?

The server-backed record and local draft have different owners. Decide whether to preserve the draft, merge changes, or surface a conflict. Silently replacing a draft can destroy user work; blindly submitting stale values can overwrite newer data.

## 29. Should loading and error flags be separate state?

They can be, but make their relationships explicit. A single request-state union or resource may prevent impossible combinations such as “loading and succeeded” at once. Choose a representation that captures valid transitions clearly.

## 30. Can a child mutate an object it received from an input?

Avoid mutating parent-owned objects. Request a change through an output or a model API designed for editing. The owner should apply the update so invariants and other consumers remain consistent.

## 31. Is local component state less important than shared state?

No. Local state is often the best owner for state with a short lifetime and one consumer. Scope state as narrowly as possible while still serving all consumers that must coordinate.

## 32. What is an invariant, and where should it live?

An invariant is a rule that must remain true, such as “the selected ID is either null or refers to an existing task.” Put enforcement in the state owner or in a transition function shared by all writers, not in several templates independently.

## 33. Should components write directly to a service's signals?

For small examples, direct signal writes can be acceptable. For important domain transitions, expose methods such as `removeTask()` or `selectTask()` so the owner can validate input and update related state consistently. Keep internal writable signals private when that improves the API.

## 34. How does state ownership help testing?

It creates a clear seam. A component can be tested with a controlled provider or mock owner; the state service can be tested through its public transitions and derived values. Scoped providers also make it possible to test isolated feature instances.

## 35. What is a quick way to spot a redundant state value?

Ask whether it can always be recomputed from existing state and whether it has an independent lifecycle or user meaning. If it is only a mirror, count, selected object, or filtered projection, make it derived. If it is a meaningful draft, snapshot, cache, or user choice, document its owner and reset rules.

## 36. Which Angular documentation explains these APIs?

- [Signals and computed values](https://angular.dev/guide/signals)
- [Dependent state with `linkedSignal`](https://angular.dev/guide/signals/linked-signal)
- [Effects and when to use them](https://angular.dev/guide/signals/effect)
- [Dependency provider configuration](https://angular.dev/guide/di/dependency-injection-providers)
- [Hierarchical dependency injection](https://angular.dev/guide/di/hierarchical-dependency-injection)
