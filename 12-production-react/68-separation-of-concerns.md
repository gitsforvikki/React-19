# Lesson 68 — Separation of Concerns and Container/Presentation Patterns

## 1. What Is Separation of Concerns?

Separation of concerns means organizing code so different responsibilities are not unnecessarily tangled together.

A production React feature may involve:

```text
rendering
state
remote data
business rules
formatting
analytics
browser APIs
side effects
```

Putting all of them in one component makes change risky.

The goal is not:

```text
one concern = one file
```

The goal is:

> Keep responsibilities clear enough that each part can change without forcing unrelated parts to change.

---

# Part 1 — Identify the Concerns

## 2. Example of a Mixed Component

```jsx
function ApplicationCard({
  id,
}) {
  // fetch application
  // format salary
  // calculate score
  // manage modal
  // send analytics
  // update server
  // render 200 lines JSX
}
```

This component has many reasons to change.

That makes it harder to:

- understand
- test
- reuse
- debug

---

## 3. Possible Separation

```text
ApplicationCardFeature
│
├── ApplicationCardView
│     → rendering
│
├── useApplicationCard
│     → React behavior
│
├── applications.api
│     → server communication
│
└── calculateMatchScore
      → pure domain logic
```

This is one possible design, not a required template.

---

# Part 2 — Container and Presentational Components

## 4. Classic Pattern

Historically, React often separated:

### Container

Responsible for:

- data
- state
- behavior
- orchestration

### Presentational component

Responsible for:

- rendering
- props
- user-facing UI

Example:

```jsx
function UserContainer() {
  const user =
    useUser();

  return (
    <UserProfile
      user={user}
    />
  );
}
```

```jsx
function UserProfile({
  user,
}) {
  return (
    <section>
      <h2>{user.name}</h2>
    </section>
  );
}
```

---

## 5. Modern React Interpretation

Do not treat container/presentation as a mandatory two-file pattern.

Hooks and composition allow more flexible separation:

```text
component
   ↓
custom Hook for meaningful behavior
   ↓
pure helpers for domain calculations
   ↓
API/data layer for external communication
```

The useful principle remains:

> Separate concerns when doing so creates clearer ownership.

---

# Part 3 — UI Components

## 6. Presentational UI

A UI-focused component should usually receive what it needs through props:

```jsx
function ApplicationCard({
  title,
  company,
  status,
  onStatusChange,
}) {
  return (
    <article>
      <h3>{title}</h3>
      <p>{company}</p>

      <StatusSelect
        value={status}
        onChange={
          onStatusChange
        }
      />
    </article>
  );
}
```

It does not need to know:

- which database exists
- which endpoint is called
- how authentication works

That improves portability and testability.

---

# Part 4 — Behavior Hooks

## 7. Extract Meaningful React Behavior

```jsx
function useApplicationFilters(
  applications
) {
  const [
    query,
    setQuery,
  ] = useState("");

  const filtered =
    useMemo(
      () =>
        filterApplications(
          applications,
          query
        ),
      [applications, query]
    );

  return {
    query,
    setQuery,
    filtered,
  };
}
```

The Hook owns React-specific behavior.

Pure filtering logic can remain a plain function.

---

## 8. Do Not Turn Everything Into a Hook

Bad mindset:

```text
logic
→ must be custom Hook
```

If code does not need React Hooks:

```js
function calculateTotal(
  items
) {
  return items.reduce(
    (sum, item) =>
      sum + item.price,
    0
  );
}
```

keep it as a normal function.

Pure functions are easier to test and reuse.

---

# Part 5 — API/Data Layer

## 9. Keep Transport Details Out of UI Where Useful

Instead of:

```jsx
function ApplicationCard() {
  await fetch(
    "/api/applications/123",
    {
      method: "PATCH",
      ...
    }
  );
}
```

you may expose:

```js
updateApplicationStatus(
  id,
  status
)
```

The UI thinks in domain operations.

The data layer owns transport details.

---

## 10. Do Not Hide Everything Behind Services

An abstraction is useful when it creates a meaningful boundary.

Avoid:

```text
component
→ service
→ manager
→ repository
→ adapter
→ wrapper
→ fetch
```

unless those layers solve real requirements.

Separation of concerns is not layer multiplication.

---

# Part 6 — Business Logic

## 11. Keep Important Rules Authoritative

Suppose ShopHub calculates order totals.

Client UI may calculate a preview:

```text
subtotal
tax
shipping
discount
```

But authoritative pricing should be validated/calculated in the trusted server/domain layer before payment/order creation.

React separation:

```text
UI
→ displays estimate/result

server/domain
→ authoritative business rule
```

Client separation improves code quality, but it does not replace server-side security.

---

# Part 7 — Derived Data

## 12. Do Not Synchronize Pure Calculations With Effects

Bad:

```jsx
useEffect(() => {
  setFiltered(
    filterApplications(
      applications,
      query
    )
  );
}, [applications, query]);
```

Better:

```jsx
const filtered =
  filterApplications(
    applications,
    query
  );
```

Or memoize only if the calculation genuinely warrants it.

Derived data belongs in rendering/pure computation, not synchronization Effects.

---

# Part 8 — Event Logic vs Effects

## 13. Keep Cause and Responsibility Together

User clicks delete:

```text
click
→ delete operation
```

Put the logic in the event/Action flow.

External synchronization:

```text
chat room ID changes
→ socket connection must change
```

belongs in an Effect.

Separation of concerns also means separating **event-caused work** from **synchronization work**.

---

# Part 9 — Composition

## 14. Composition Reduces Coupling

Instead of a layout importing every possible feature:

```jsx
<Panel>
  {content}
</Panel>
```

or:

```jsx
<Panel>
  <ApplicationList />
</Panel>
```

lets the parent decide what content belongs inside.

This keeps generic components from learning feature-specific behavior.

---

# Part 10 — Prop Interfaces

## 15. Expose Intent, Not Implementation

Weak API:

```jsx
<Modal
  setOpen={setOpen}
  setSelectedId={setSelectedId}
  setMode={setMode}
/>
```

More intentional:

```jsx
<Modal
  onClose={handleClose}
/>
```

A child should usually receive the capability it needs, not unrestricted access to unrelated parent implementation details.

---

# Part 11 — Smart vs Dumb Is an Imperfect Label

## 16. Prefer Responsibility-Based Language

Terms such as:

```text
smart component
dumb component
```

are common historically, but can oversimplify modern React.

Prefer thinking:

```text
orchestration component
UI component
feature component
custom Hook
domain helper
data layer
```

A UI component may still have perfectly valid local UI state.

---

# Part 12 — Local State in Presentational Components

## 17. Presentation Does Not Mean Stateless

A reusable dropdown may own:

```text
isOpen
highlightedIndex
```

while still being UI-focused.

The important question is ownership.

If state exists only to implement the component's UI behavior, local ownership is appropriate.

---

# Part 13 — Error Boundaries as Separation

## 18. Separate Failure Domains

A production dashboard can isolate feature failures:

```text
Dashboard
├── Profile ErrorBoundary
├── Analytics ErrorBoundary
└── Applications ErrorBoundary
```

This prevents one rendering failure from necessarily destroying unrelated UI.

Failure boundaries are architectural boundaries too.

---

# Part 14 — Context as Dependency Injection

## 19. Context Can Supply Cross-Cutting Dependencies

Examples:

```text
theme
auth/session interface
localization
feature-level shared state
```

Context can reduce plumbing through components that do not care about the value.

But Context is not automatically the right solution for every shared concern.

---

# Part 15 — Testing Benefits

## 20. Pure Logic

```js
expect(
  calculateMatchScore(
    job,
    profile
  )
).toBe(85);
```

No React rendering required.

---

## 21. UI

```text
render ApplicationCard
with props
→ verify visible behavior
```

No real backend required if transport concerns are outside the component.

Clear concerns usually produce easier tests.

---

# Part 16 — CareerLoop Example

## 22. Before

```text
ApplicationsPage
├── fetch
├── filters
├── sort
├── analytics
├── modal
├── mutations
├── formatting
└── huge JSX
```

After:

```text
ApplicationsFeature
├── ApplicationsView
│   ├── Filters
│   ├── ApplicationList
│   └── EditDialog
│
├── useApplicationFilters
├── application API/data layer
└── pure application helpers
```

Do not split further unless complexity requires it.

---

# Part 17 — CodeBuddy Example

## 23. Chat Feature

```text
ChatFeature
├── ChatWindow
│   → UI composition
│
├── MessageList
│   → message rendering
│
├── MessageComposer
│   → draft UI
│
├── useSocketChat
│   → socket lifecycle
│
└── chat helpers
    → pure transformations
```

Socket connection lifecycle should not be duplicated throughout visual components.

---

# Part 18 — Common Mistakes

## 24. Common Mistakes

1. Treating container/presentation as a mandatory file pattern.
2. Keeping all logic in page-level components.
3. Turning every helper into a custom Hook.
4. Creating unnecessary service layers.
5. Putting pure derived calculations in Effects.
6. Mixing transport details throughout UI components.
7. Giving children unrestricted setters instead of intentional callbacks.
8. Assuming presentational components cannot own local UI state.
9. Putting authoritative security/business validation only in client React.
10. Creating generic components that know feature-specific business rules.
11. Extracting abstractions that make simple flows harder to trace.

---

# Part 19 — Decision Guide

## 25. Where Should Logic Go?

```text
Does it render UI?
→ component

Does it coordinate React Hooks/state/lifecycle?
→ component or custom Hook

Is it pure calculation/transformation?
→ plain function

Does it communicate with external API/service?
→ data/API boundary where useful

Is it authoritative business/security logic?
→ trusted server/domain layer

Was it caused directly by user interaction?
→ event/Action flow

Does it synchronize with an external system?
→ Effect
```

---

# Part 20 — Interview Questions

## 26. What Is Separation of Concerns in React?

Organizing rendering, React behavior, external communication, and pure/domain logic so responsibilities remain understandable and can evolve with limited unnecessary coupling.

---

## 27. Container vs Presentational Components?

Traditionally, containers handled data/behavior while presentational components rendered props.

Modern React still benefits from that separation when useful, but custom Hooks and composition mean it is no longer necessary to enforce a rigid container/presentation pair for every feature.

---

## 28. Are Presentational Components Always Stateless?

No.

They may own local UI state when that state belongs to their own behavior.

---

## 29. When Should Logic Become a Custom Hook?

When it represents meaningful reusable or independently complex React behavior that uses Hooks.

Pure logic should usually remain a plain function.

---

## 30. Why Separate API Logic From UI?

It can reduce transport coupling, improve reuse/testing, and let UI express domain intentions rather than HTTP details.

Do it when the abstraction provides real value.

---

# Part 21 — Interview-Ready Summary

## 31. Final Answer

```text
I separate concerns in React by responsibility rather
than blindly creating layers.

UI components focus on rendering and interaction,
custom Hooks encapsulate meaningful React-specific
behavior, pure business transformations remain plain
functions, and external communication can sit behind a
small data/API boundary when that improves clarity.

I still use the container/presentation idea when it
helps, but I do not treat it as a mandatory pattern in
modern React because Hooks and composition provide more
flexible boundaries.

I also keep authoritative business and security rules
on the trusted server side rather than relying on React
client separation for security.
```

---

# Part 22 — Mental Model

## 32. Separation of Concerns

```text
                  FEATURE
                     │
      ┌──────────────┼──────────────┐
      ↓              ↓              ↓
     UI          React behavior    Data/API
      │              │              │
 components      custom Hooks   external system
      │
      ↓
user interaction

        PURE DOMAIN HELPERS
                 │
          calculations /
          transformations


Goal:
each concern has
clear ownership
without unnecessary layers
```

---

## 33. Key Takeaways

- Separation of concerns is about clear responsibilities, not maximum file count.
- Container/presentation remains a useful concept, not a mandatory architecture.
- Hooks provide modern behavior extraction.
- Pure logic should remain plain JavaScript when possible.
- UI should not need unnecessary transport details.
- Avoid service-layer abstraction for its own sake.
- Derived values usually do not need Effects.
- Separate event-caused logic from external synchronization.
- Composition reduces coupling.
- Prefer intentional callbacks over exposing unrelated setters.
- UI-focused components may own local UI state.
- Error boundaries can separate failure domains.
- Context can provide cross-cutting dependencies when appropriate.
- Clear separation improves testing.
- Trusted server/domain code remains responsible for authoritative validation and security.

---

## Next Lesson

➡️ [Lesson 69 — Error, Loading and Empty-State Design](./69-ui-states.md)
