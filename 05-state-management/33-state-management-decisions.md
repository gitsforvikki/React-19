# Lesson 33 — When to Use Local State, Context or External State ⭐⭐⭐⭐⭐

## 1. The Real State-Management Question

Developers often ask:

> Which state-management library should I use?

A better first question is:

> Who needs this state, who owns it, and how does it change?

Most state-management problems become easier when you first classify:

```text
scope
ownership
update complexity
data source
subscription needs
```

Do not globalize state before understanding these dimensions.

---

## 2. Start with Local State ⭐⭐⭐⭐⭐

Default choice:

```jsx
const [open, setOpen] =
  useState(false);
```

If only one component or a small nearby subtree needs a value, keep it local.

Examples:

- modal open state
- dropdown state
- input value
- hover/selection UI
- accordion panel
- temporary draft
- local loading state

Rule:

> Keep state as close as possible to where it is used.

This is **state colocation**.

---

## 3. Lift State When Components Must Coordinate

Suppose siblings need the same query:

```text
SearchPage
   │ query
   ├── SearchInput
   └── SearchResults
```

The closest common ancestor owns it.

This is **lifting state up**.

Do not jump directly from:

```text
local state
→ global store
```

Often the correct solution is simply:

```text
local
→ nearest common ancestor
```

---

## 4. Use Props for Nearby Sharing

```jsx
<SearchInput
  value={query}
  onChange={setQuery}
/>

<SearchResults
  query={query}
/>
```

Props make dependencies explicit.

Prop drilling over a short, meaningful path is not a problem that must be eliminated.

---

## 5. Use Context for Deep Subtree Distribution ⭐⭐⭐⭐⭐

Use Context when:

- many descendants need the same dependency
- data would otherwise pass through unrelated intermediates
- the value belongs conceptually to a subtree/environment

Examples:

```text
theme
current user information
locale
feature state
state + reducer dispatch
```

Context solves **distribution**, not every form of state management.

---

## 6. Use useReducer for Complex Transition Logic ⭐⭐⭐⭐⭐

Choose `useReducer` when:

- many handlers update related state
- transitions are complex
- action semantics improve readability
- state bugs come from scattered update logic

You can use it locally:

```text
Component
 └── useReducer
```

or with Context:

```text
FeatureProvider
 ├── useReducer
 └── Context
```

Reducer complexity and state sharing are separate decisions.

---

## 7. Four Different Problems Often Called "State"

Before choosing a tool, distinguish:

### Local UI state

```text
modal open
selected tab
draft text
```

### Shared client UI/domain state

```text
feature filters
shopping cart UI model
multi-step editor state
```

### Server/remote state

```text
products from API
user records
applications from database
```

### URL/navigation state

```text
search query
page number
selected filter encoded in URL
```

These have different ownership and synchronization needs.

Do not put all of them into one generic "global state" bucket.

---

## 8. Server State Is Different ⭐⭐⭐⭐⭐

Suppose applications come from a database.

That data has an authoritative source outside the component:

```text
database / API
      ↓
client representation
```

Concerns may include:

- fetching
- caching
- stale data
- refetching
- invalidation
- mutation synchronization
- request deduplication

A plain Context is not automatically a server-cache solution.

Keep the distinction:

```text
client UI state
≠
remote/server state
```

Framework/data-library implementation belongs in its appropriate learning track.

---

## 9. URL State Is Also Different

If a value should:

- survive refresh
- be shareable by link
- support browser back/forward
- represent navigation

it may belong in the URL rather than hidden component state.

Example:

```text
/jobs?status=interview&page=2
```

The exact routing implementation depends on your routing environment, but the architectural question matters in React applications.

---

## 10. Derived State Usually Should Not Be Stored ⭐⭐⭐⭐⭐

Suppose:

```jsx
const [applications, setApplications] =
  useState(...);

const [selectedId, setSelectedId] =
  useState(null);
```

Do not also store:

```jsx
const [
  selectedApplication,
  setSelectedApplication,
] = useState(...);
```

if it can be derived:

```jsx
const selectedApplication =
  applications.find(
    (application) =>
      application.id ===
      selectedId
  );
```

Before asking where state should live, ask:

> Should this be state at all?

---

## 11. Avoid Duplicate Sources of Truth

Bad:

```text
FilterBar owns status
ApplicationList owns status
Context owns status
URL owns status
```

unless these are intentionally different concepts.

Better:

```text
one authoritative source
      ↓
other UI derives/reads it
```

Many synchronization bugs are ownership bugs.

---

## 12. State Decision Ladder ⭐⭐⭐⭐⭐

Use this order:

```text
1. Can it be derived?
   │
   ├── yes → derive it
   ↓ no

2. Does one component need it?
   │
   ├── yes → local state
   ↓ no

3. Do nearby components need it?
   │
   ├── yes → lift to common ancestor + props
   ↓ no

4. Do many deep descendants need it?
   │
   ├── yes → Context
   ↓

5. Are update rules complex?
   │
   ├── yes → consider useReducer
   ↓

6. Are external-store capabilities actually required?
   │
   ├── yes → evaluate external state tool
   ↓

7. Is it remote/server data or URL state?
      → use the appropriate data/navigation model
```

These choices can combine.

---

## 13. When External Client State Can Be Useful ⭐⭐⭐⭐⭐

An external store can become useful when requirements include:

- state shared across distant unrelated application areas
- fine-grained subscriptions/selectors
- updates outside React component ownership
- specialized persistence
- devtools/history requirements
- middleware/event architecture
- large client-side domain store with frequent independent updates

Examples of external state ecosystems include Redux Toolkit and Zustand, but this lesson is about **when**, not library-specific APIs.

Do not adopt one simply because a project is "production".

---

## 14. Context vs External Store ⭐⭐⭐⭐⭐

### Context

Good for:

- subtree dependencies
- relatively cohesive shared values
- theme/auth-like environment values
- reducer state for a feature

Core model:

```text
provider value changes
→ Context consumers update
```

### External store

Can provide:

- independent store outside component tree
- selector-based subscriptions
- specialized tooling
- domain-specific state architecture

Core model:

```text
store changes
→ subscribed slices notify consumers
```

The decision is requirements-driven.

---

## 15. Context Is Not "Bad for Performance"

Oversimplification:

```text
Context is slow
```

Better:

```text
a frequently changing broad Context
with many consumers
can create a large render scope
```

Fix architecture first:

- colocate frequently changing state
- split unrelated contexts
- keep provider scope focused
- separate state and dispatch when useful
- profile before optimizing

Do not reject Context categorically.

---

## 16. Local State Is Not "Beginner State"

Production applications should contain lots of local state.

Example:

```text
Global store
  ✗ every dropdown
  ✗ every modal
  ✗ every input
  ✗ every hover
```

Globalizing local UI state increases coupling and makes components harder to reuse.

Local state is an architectural strength when the state is local.

---

## 17. State Colocation and Performance

Suppose:

```text
App
 └── query state
      ↓
entire app renders around query changes
```

but only Search needs it.

Move it lower:

```text
App
 └── SearchFeature
      └── query state
```

Benefits:

- smaller ownership scope
- clearer responsibility
- potentially smaller render scope

Do not lift state higher than coordination requires.

---

## 18. Example — Modal

Only one component controls/uses it:

```text
ProfileCard
 └── isModalOpen
```

Use local state.

If trigger and modal are siblings:

```text
ProfileSection
 ├── EditButton
 └── EditModal
```

lift `isOpen` to `ProfileSection`.

No global store required.

---

## 19. Example — Theme

Many distant components need it:

```text
App
 ├── Header
 ├── Sidebar
 └── Content
      └── Buttons
```

Theme Context is natural.

If theme persistence is required, persistence is an additional concern; it does not change the basic Context distribution decision.

---

## 20. Example — CareerLoop Filters ⭐⭐⭐⭐⭐

Suppose:

```text
ApplicationsPage
 ├── FilterBar
 ├── ApplicationList
 └── Summary
```

All need filter state.

Option 1:

```text
ApplicationsPage owns filters
→ props
```

This is likely sufficient.

If the tree becomes deep:

```text
ApplicationsProvider
→ Filters Context
```

Do not put filters in an app-wide store unless unrelated areas genuinely need them or other requirements justify it.

---

## 21. Example — CodeBuddy Selected Developer

Suppose:

```text
DeveloperWorkspace
 ├── DeveloperList
 ├── ProfilePanel
 └── ChatPanel
```

All need the selected developer ID.

Best starting point:

```text
DeveloperWorkspace
  owns selectedDeveloperId
```

Pass it to nearby children.

If the feature becomes deeply nested, feature Context may help.

Again:

```text
shared
≠
automatically global
```

---

## 22. Example — Authentication

Current-user/session information may be required by many distant UI components:

```text
Header
AccountMenu
ProtectedArea
Profile
```

Context can be a useful way to distribute already-known client-visible auth/session information.

But Context itself does **not** authenticate users or enforce authorization.

Trusted security checks belong at trusted boundaries.

---

## 23. Example — Server Applications List

If a list comes from a server and needs:

- caching
- refetch
- invalidation
- synchronization after mutations

do not assume:

```text
fetch once
→ put everything in Context
```

is automatically the best architecture.

First identify it as remote state.

Then choose an appropriate data strategy for the environment.

---

## 24. State Ownership Questions ⭐⭐⭐⭐⭐

For every state value ask:

1. Who reads it?
2. Who changes it?
3. What is the closest common owner?
4. Is it derived?
5. Is it temporary UI state?
6. Is an external system authoritative?
7. Should it survive refresh?
8. Should it be shareable in a URL?
9. How frequently does it change?
10. Do consumers need independent slices?

These questions are more useful than choosing a library by popularity.

---

## 25. State Structure Still Matters

Regardless of tool:

- avoid contradictions
- avoid redundant state
- avoid duplication
- avoid deeply nested structures when possible
- normalize relationships when helpful
- store IDs instead of duplicate objects when appropriate

A poor state model stays poor inside Redux, Zustand, Context, or `useReducer`.

---

## 26. State Management Is More Than Storage ⭐⭐⭐⭐⭐

Good state management includes:

```text
ownership
+
structure
+
transitions
+
distribution
+
synchronization
+
lifetime
```

A library only solves part of this.

Architecture begins before library selection.

---

## 27. Decision Table ⭐⭐⭐⭐⭐

| Situation | Start With |
|---|---|
| One component needs state | `useState` |
| Complex local transitions | `useReducer` |
| Nearby siblings share state | Lift state + props |
| Deep subtree shares a value | Context |
| Deep subtree + complex transitions | Context + `useReducer` |
| Value can be calculated | Derive it; don't store it |
| Data is remote/server-owned | Data-fetch/cache strategy |
| State belongs in navigation | URL/router state |
| Broad client state needs specialized subscriptions/tooling | Evaluate external store |

This is a starting framework, not a rigid law.

---

## 28. Common Mistakes ⭐⭐⭐⭐⭐

1. Putting all state into a global store.
2. Using Context for every prop.
3. Choosing Redux/Zustand before identifying requirements.
4. Storing derived data.
5. Duplicating the same state in multiple places.
6. Treating remote server data exactly like local UI state.
7. Lifting state to `App` when only one feature needs it.
8. Assuming Context is inherently slow.
9. Assuming local state is not production-grade.
10. Using Effects to synchronize duplicated state.
11. Confusing state distribution with state ownership.
12. Ignoring URL state for shareable navigation state.

---

## 29. Interview Questions ⭐⭐⭐⭐⭐

### Where should React state live?

As close as possible to the components that need it, while being high enough to coordinate all consumers that share it.

### When should you lift state?

When multiple components need to coordinate around the same source of truth.

### When should you use Context?

When a value must be available to many or deeply nested descendants and explicit prop passing becomes inconvenient.

### When should you use useReducer?

When state transitions are complex or update logic is scattered across many handlers.

### When should you use reducer + Context?

When complex state is shared across distant descendants in a subtree.

### When should you consider an external store?

When requirements need broader client state coordination, fine-grained subscriptions/selectors, external updates, persistence, specialized tooling, or similar capabilities.

### Is Context a replacement for Redux?

Not directly. They solve overlapping but different architectural needs.

### Should server data always be put into global client state?

No. Server state has fetching, caching, invalidation, and synchronization concerns and often deserves a dedicated data strategy.

### What is state colocation?

Keeping state near the components that use it rather than lifting/globalizing it unnecessarily.

### What is the first question before choosing a state library?

Who needs the state, who owns it, and what capabilities are actually required?

---

## 30. Interview Scenario ⭐⭐⭐⭐⭐

A search input and results list are siblings.

Where should `query` live?

```text
SearchPage
 ├── SearchInput
 └── SearchResults
```

Answer:

```text
SearchPage
```

because it is the closest common ancestor.

Context/global state would be unnecessary unless the real component structure or requirements make the value broadly/deeply shared.

---

## 31. Interview Scenario — Context or External Store?

A dashboard has hundreds of components, but only one feature's deeply nested children need editor state.

Do not choose an external store merely because the application is large.

Start with a feature-scoped Provider, potentially with reducer + Context.

If measured requirements later demand fine-grained independent subscriptions or specialized store capabilities, evaluate an external store.

---

## 32. Complete Decision Mental Model ⭐⭐⭐⭐⭐

```text
                  VALUE
                    │
          Can it be derived?
             ┌──────┴──────┐
            yes            no
             │              │
          derive         Who needs it?
                         │
             ┌───────────┼────────────┐
             ↓           ↓            ↓
          one/local   nearby shared  deep shared
             │           │            │
          useState     lift state    Context
             │           │            │
       complex rules? complex rules? complex rules?
          │               │            │
          └── useReducer when justified ┘

Then separately ask:

Is the authoritative source external/server?
→ use appropriate data strategy

Should it be navigable/shareable?
→ consider URL state

Need specialized broad subscriptions/tooling?
→ evaluate external store
```

---

## 33. Section 5 Revision ⭐⭐⭐⭐⭐

```text
Lesson 30
Context + useContext
→ distribute values deeply

Lesson 31
useReducer
→ centralize complex transitions

Lesson 32
Reducer + Context
→ complex shared subtree state

Lesson 33
Decision framework
→ choose the smallest correct state architecture
```

The central principle:

> **Start local. Lift only when sharing requires it. Use Context for deep distribution. Use reducers for transition complexity. Add external state tooling only when concrete requirements justify it.**

---

## 34. Key Takeaways

- State location should follow ownership and consumers.
- Start with local state.
- Lift state to the nearest common ancestor when components need to coordinate.
- Props are excellent for nearby sharing.
- Context is useful for deep subtree distribution.
- `useReducer` addresses transition complexity, not sharing by itself.
- Reducer + Context is a strong built-in architecture for complex feature state.
- Do not store values that can be derived.
- Avoid duplicate sources of truth.
- Distinguish local UI state, shared client state, remote state, and URL state.
- Context is not inherently global or inherently slow.
- Local state is fully appropriate in production.
- Server state has different synchronization/caching concerns.
- External stores are useful when concrete capabilities justify them.
- Good state structure matters regardless of library.
- The best state-management architecture is usually the smallest one that correctly matches the state’s scope and behavior.

---

## Section 5 — Sharing and Managing State Completed ✅

You have completed:

- Lesson 30 — Context API and useContext ⭐⭐⭐⭐⭐
- Lesson 31 — useReducer ⭐⭐⭐⭐⭐
- Lesson 32 — useReducer + Context Architecture
- Lesson 33 — When to Use Local State, Context or External State ⭐⭐⭐⭐⭐

---

## Next Section — Reusable Logic

➡️ [Lesson 34 — Custom Hooks ⭐⭐⭐⭐⭐](../06-reusable-logic/34-custom-hooks.md)
