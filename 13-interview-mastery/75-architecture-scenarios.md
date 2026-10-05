# Lesson 75 — React Architecture and Scenario-Based Questions ⭐⭐⭐⭐⭐

## 1. Why Scenario Questions Matter

Architecture interviews rarely have one perfect answer.

The interviewer wants to see whether you can:

```text
understand requirements
        ↓
identify ownership
        ↓
separate responsibilities
        ↓
choose the simplest suitable design
        ↓
explain tradeoffs
        ↓
handle failure / scale / security
```

A strong answer is not:

> "Use Redux."

A strong answer explains **why a particular state, component boundary, data flow, or optimization belongs where it does**.

---

# Part 1 — Architecture Answer Framework

## 2. Use This Framework ⭐⭐⭐⭐⭐

When given a React architecture problem, answer in this order:

### Step 1 — Clarify the requirements

Ask or establish:

- Who uses the feature?
- What data changes?
- Which components need the data?
- Is the data local, shared, server-owned, or external?
- Does it need persistence?
- Is real-time behavior required?
- What are loading/error/empty states?
- Are there permissions?
- What scale/performance constraints exist?

### Step 2 — Identify sources of truth

```text
UI-only state
→ React component/feature

shared subtree state
→ lifted state / Context / reducer

server data
→ server/API remains authoritative

external store
→ subscription/store abstraction
```

### Step 3 — Define boundaries

Separate:

```text
presentation
React interaction/state
domain logic
data access
server authority
```

### Step 4 — Design failure behavior

Consider:

- loading
- empty
- error
- retry
- stale data
- optimistic failure
- partial failure

### Step 5 — Discuss tradeoffs

Do not present every choice as universally correct.

---

# Part 2 — Scenario: Large React Project Structure

## 3. How Would You Structure a Growing React Application? ⭐⭐⭐⭐⭐

Avoid a project that becomes:

```text
components/
  hundreds of unrelated files

hooks/
  every hook in application

utils/
  every helper

services/
  every request
```

Prefer clear feature ownership when the product is domain-heavy:

```text
src/
├── app/
├── features/
│   ├── applications/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── api/
│   │   └── utils/
│   └── auth/
├── shared/
│   ├── ui/
│   └── utils/
└── ...
```

### Interview Answer

```text
I prefer feature-oriented boundaries for domain code so
related UI, Hooks, data integration and helpers are
colocated.

I move code into shared layers only when it represents
genuine reusable infrastructure or UI. I also try to
keep dependencies directional so shared code does not
depend back on feature-specific code.
```

The exact folders are less important than clear ownership.

---

# Part 3 — Scenario: Where Should State Live?

## 4. Local vs Lifted vs Context vs External Store ⭐⭐⭐⭐⭐

Use the narrowest ownership that satisfies consumers.

```text
one component
     ↓
local state

siblings need it
     ↓
closest common parent

many descendants need it
without prop plumbing
     ↓
Context may fit

complex app-wide external/shared state
with stronger store requirements
     ↓
external state solution may fit
```

Do not start by globalizing everything.

---

## 5. Scenario: Modal Open State

Question:

> Where should modal open/close state live?

Answer depends on who controls it.

If only one feature owns it:

```jsx
const [isOpen, setIsOpen] =
  useState(false);
```

Keep it local.

If distant components genuinely coordinate the same modal, lift or introduce an appropriate shared boundary.

Do not create global state merely because something is a modal.

---

# Part 4 — Scenario: Form Architecture

## 6. Large Form With Many Fields

First determine:

- validation requirements
- cross-field dependencies
- submission model
- server errors
- performance requirements
- reusable field behavior

Possible architecture:

```text
Form feature
├── field components
├── validation/domain rules
├── submission Action/handler
└── server/API boundary
```

Do not automatically create one `useState` per field or automatically install a form library.

Choose based on complexity.

---

## 7. Client vs Server Validation ⭐⭐⭐⭐⭐

```text
client validation
→ fast UX feedback

server validation
→ trust boundary
```

The same business constraint may be represented on both sides, but the server remains authoritative.

---

# Part 5 — Scenario: Search Page

## 8. Search With 10,000 Local Items

Potential approach:

1. Keep query state near the search UI.
2. Derive filtered results during render.
3. Profile.
4. If calculation is expensive, consider `useMemo`.
5. If rendering blocks typing, consider `useDeferredValue`.
6. If list rendering itself is huge, consider virtualization.

```text
query
  ↓ urgent
input updates immediately
  ↓
deferred query
  ↓
expensive result rendering
```

Do not assume memoization alone solves large DOM rendering.

---

## 9. Search With Millions of Server Records

Do not download the whole dataset into React.

Better conceptual architecture:

```text
input
 ↓
debounce if appropriate
 ↓
query API
 ↓
server/database search
 ↓
pagination/cursor
 ↓
render manageable result set
```

Also handle:

- cancellation/stale requests
- loading
- empty
- error
- pagination
- caching where architecture supports it

---

# Part 6 — Scenario: Data Fetching

## 10. Should Data Fetching Live in useEffect? ⭐⭐⭐⭐⭐

Sometimes an Effect can synchronize a client component with a network resource.

But architecture questions require more nuance.

Ask:

- Is a framework providing server data loading?
- Do we need caching?
- request deduplication?
- retries?
- mutations?
- optimistic updates?
- server rendering?
- prefetching?

For complex server state, a framework or dedicated data layer may provide better coordination than manually building everything around Effects.

React core and framework-specific data architecture are separate concerns.

---

## 11. Avoid Fetching Waterfalls

Potential poor flow:

```text
Parent renders
→ fetch parent data
→ child appears
→ child fetches
→ grandchild appears
→ grandchild fetches
```

This can serialize requests unnecessarily.

Where architecture allows, identify independent data dependencies and start work earlier/in parallel.

---

# Part 7 — Scenario: Real-Time Chat

## 12. CodeBuddy Chat Architecture ⭐⭐⭐⭐⭐

Requirements:

- conversation selection
- historical messages
- real-time messages
- reconnect behavior
- cleanup
- optimistic sending
- authorization

Conceptual architecture:

```text
ChatPage
│
├── ConversationList
│
└── ChatRoom
     ├── MessageList
     └── Composer

selected conversation
        ↓
history data
        +
real-time subscription
        ↓
normalized/current messages
        ↓
render
```

Connection synchronization:

```jsx
useEffect(() => {
  const socket =
    connect(roomId);

  return () => {
    socket.disconnect();
  };
}, [roomId]);
```

Real production logic may use a shared socket manager rather than opening an entirely new transport per component.

---

## 13. Who Authorizes the Chat Room? ⭐⭐⭐⭐⭐

Not React.

The server must verify:

```text
authenticated user
       +
membership/permission
       +
requested conversation
```

A user should not gain room access merely because the client knows a room ID.

---

# Part 8 — Scenario: E-Commerce Cart

## 14. ShopHub Cart Architecture

Potential sources:

```text
guest cart
authenticated cart
server inventory
server pricing
payment provider
```

Important distinction:

```text
React cart display
≠ authoritative inventory
≠ authoritative price
≠ confirmed order
```

The UI may optimistically update quantity, but checkout must be validated by the server.

---

## 15. Preventing Overselling ⭐⭐⭐⭐⭐

Disabling the Buy button is not enough.

```text
React
→ pending UX

Server/database
→ concurrency invariant
```

Inventory must be protected at the authoritative persistence layer using an appropriate atomic/transactional design.

This is a backend/system concern, not something React state can guarantee.

---

# Part 9 — Scenario: Authentication

## 16. Where Should Auth State Live?

The UI may need session information such as:

```text
current user
role/permissions for presentation
loading state
sign-out action
```

A Context or external auth integration may distribute this information.

But:

```text
client auth state
≠ authorization boundary
```

Server/API operations must verify identity and permission.

---

## 17. Protected Route UI

Client routing can improve UX by redirecting unauthenticated users.

It does not secure server data.

```text
client route guard
→ navigation UX

server auth check
→ data/action security
```

---

# Part 10 — Scenario: Permissions

## 18. Role-Based UI

A component may hide unavailable actions:

```jsx
{canDelete && (
  <DeleteButton />
)}
```

This is correct UX.

But server authorization still decides whether deletion is allowed.

For complex products, consider permission capabilities instead of scattering raw role checks everywhere:

```text
canEditApplication
canDeleteUser
canManageBilling
```

This can make UI intent clearer.

---

# Part 11 — Scenario: Context Performance

## 19. One Giant Context Is Causing Re-renders

Suppose:

```text
AppContext
├── user
├── theme
├── notifications
├── cart
├── filters
└── modal
```

Every concern has different update frequency and consumers.

Potential improvements:

- colocate local state
- split contexts by responsibility
- stabilize provider values when useful
- use external-store selectors if requirements justify them

Do not assume one global Context is simple architecture.

---

# Part 12 — Scenario: Prop Drilling

## 20. Is Prop Drilling Always Bad?

No.

Passing props through one or two levels can be explicit and simple.

Context is useful when many intermediate components pass values they do not use.

Avoid:

```text
prop drilling exists
→ automatically add global store
```

Evaluate actual complexity.

---

# Part 13 — Scenario: Reusable Component Library

## 21. Designing a Reusable Button

A good reusable primitive should preserve native behavior.

```jsx
function Button({
  children,
  ...props
}) {
  return (
    <button {...props}>
      {children}
    </button>
  );
}
```

Consider:

- semantic element
- disabled state
- focus
- accessible name
- variants
- loading behavior
- prop forwarding
- ref requirements

Do not turn every visual difference into a separate component.

---

## 22. Designing a Reusable Modal

Prefer declarative API:

```jsx
<Modal
  open={isOpen}
  onClose={handleClose}
/>
```

rather than primarily:

```text
modalRef.current.open()
```

Imperative handles are appropriate for genuinely imperative behavior, but normal state/props should remain the default.

---

# Part 14 — Scenario: Component API Design

## 23. Too Many Boolean Props

Problem:

```jsx
<Card
  compact
  elevated
  horizontal
  selectable
  interactive
  showFooter
  ...
/>
```

This can create invalid combinations.

Alternatives may include:

- composition
- explicit variants
- child slots
- smaller focused components

Example:

```jsx
<Card>
  <Card.Header />
  <Card.Body />
  <Card.Footer />
</Card>
```

Choose APIs that make valid usage easy.

---

# Part 15 — Scenario: Error Boundaries

## 24. Where Should Error Boundaries Go? ⭐⭐⭐⭐⭐

Too high:

```text
entire app
→ one widget crashes
→ whole app replaced
```

Too low:

```text
boundary around every tiny element
→ excessive complexity
```

Prefer meaningful failure regions:

```text
Dashboard
├── Header
├── ErrorBoundary
│   └── Analytics
├── ErrorBoundary
│   └── ActivityFeed
└── Navigation
```

Ask:

> What part of the interface should remain usable if this region fails?

---

# Part 16 — Scenario: Loading Boundaries

## 25. One Spinner for the Entire Page?

Often poor UX when independent regions load separately.

Better architecture can provide progressive UI:

```text
Page shell
├── Profile → skeleton
├── Feed → skeleton
└── Recommendations → skeleton
```

Suspense boundaries or explicit async-state boundaries should follow meaningful UX regions.

Avoid excessive fallback flicker from boundaries that are too granular.

---

# Part 17 — Scenario: Optimistic UI

## 26. When Is Optimistic UI Appropriate? ⭐⭐⭐⭐⭐

Good candidate:

```text
like
follow
connect
status change
simple message send
```

when success is likely and rollback is understandable.

Be more cautious with:

- destructive irreversible operations
- money
- inventory
- permission-sensitive changes

Architecture:

```text
user action
   ↓
optimistic projection
   ↓
authoritative request
  / \
success failure
  │      │
confirm reconcile + feedback
```

---

# Part 18 — Scenario: State Machine Thinking

## 27. Too Many Booleans

Problem:

```jsx
isLoading
isSuccess
isError
isEmpty
```

Can produce impossible combinations.

For mutually exclusive states:

```jsx
const status =
  "idle"
  | "loading"
  | "success"
  | "error";
```

Then derive:

```text
empty =
status === success
&& data.length === 0
```

This improves reasoning.

---

# Part 19 — Scenario: Complex State Transitions

## 28. When Would You Choose useReducer? ⭐⭐⭐⭐⭐

Suppose an editor has:

- selected item
- draft values
- validation
- undo-like transitions
- reset
- save result

If updates are highly related, a reducer can express domain transitions:

```text
FIELD_CHANGED
ITEM_SELECTED
RESET
SAVE_SUCCEEDED
SAVE_FAILED
```

Use reducers for clearer transition logic, not merely because there are several state variables.

---

# Part 20 — Scenario: Custom Hook

## 29. What Belongs in a Custom Hook?

A custom Hook is useful when multiple components share React-specific behavior.

Example:

```text
useOnlineStatus
useConversation
useDebouncedValue
```

Do not hide every helper inside a Hook.

Pure transformation:

```js
calculateTotal(items)
```

can remain a normal function.

---

# Part 21 — Scenario: Performance

## 30. A Page Is Slow. What Do You Do? ⭐⭐⭐⭐⭐

Strong answer:

```text
1. Reproduce the slowdown.
2. Profile.
3. Identify whether cost is rendering, computation,
   network, bundle, DOM size, or external library.
4. Fix architecture/state ownership first.
5. Apply targeted optimization.
6. Measure again.
```

Possible React improvements:

- colocate state
- remove unnecessary Effects
- avoid redundant state
- memoize expensive calculations
- memoize expensive children when useful
- stabilize identities where necessary
- defer non-urgent rendering
- virtualize large lists
- split code

Do not optimize from intuition alone.

---

# Part 22 — Scenario: Huge List

## 31. 50,000 Rows

Even if calculation is fast, rendering 50,000 DOM rows can be expensive.

Consider:

```text
pagination
virtualization/windowing
server-side filtering
incremental loading
```

`React.memo` does not solve the cost of initially creating an enormous DOM tree.

---

# Part 23 — Scenario: React.memo Everywhere

## 32. Is Blanket Memoization Good Architecture?

No.

Costs include:

- comparison work
- dependency complexity
- harder debugging
- false confidence
- unstable props defeating memoization

Memoize where skipped work has meaningful value.

---

# Part 24 — Scenario: Microfrontend Boundary

## 33. How Would You Choose a Microfrontend Boundary?

A React component is not automatically a good microfrontend boundary.

Prefer business/domain ownership such as:

```text
checkout
catalog
account
analytics
```

Consider:

- independent team ownership
- deployment needs
- shared dependencies
- routing
- design system
- authentication
- cross-app communication
- failure isolation
- versioning

Microfrontends introduce operational complexity.

Do not choose them simply because the application is large.

---

# Part 25 — Scenario: Cross-Feature Communication

## 34. How Should Features Communicate?

Prefer explicit contracts.

Possible approaches:

- props/composition
- shared state boundary
- URL/router state
- external store
- event/message contract for independently deployed systems

Avoid hidden mutable globals.

Choose based on coupling and ownership.

---

# Part 26 — Scenario: URL State

## 35. Should Filters Be React State or URL State?

If users should be able to:

- bookmark
- share
- refresh
- use browser navigation

then URL state may be the better source of truth for filters/search/pagination.

This is often framework/router-specific in implementation.

Architecture principle:

> State should live in the system whose semantics it represents.

---

# Part 27 — Scenario: Server State vs Client State

## 36. Important Distinction ⭐⭐⭐⭐⭐

Client/UI state:

```text
modal open
selected tab
draft input
hover/interaction
```

Server state:

```text
applications
products
user profile
orders
messages
```

Server state usually needs concerns such as:

- fetching
- caching
- stale data
- refetching
- mutation reconciliation

Do not treat a server response as if React local state automatically becomes the authoritative database.

---

# Part 28 — Scenario: Offline / Retry

## 37. Network Failure

Production UI should answer:

```text
What does user see?
Can they retry?
Are draft values preserved?
Can duplicate submission occur?
Was the operation actually accepted?
```

A retry button should retry the intended operation safely.

For mutations, server idempotency may be necessary depending on the operation.

---

# Part 29 — Scenario: Accessibility Architecture

## 38. Accessibility Should Be Built Into Primitives

If every feature creates its own modal, button and select behavior, accessibility bugs multiply.

A reusable accessible design system can centralize:

- semantics
- focus behavior
- keyboard interaction
- accessible names
- disabled behavior

Accessibility is architecture, not final QA polish.

---

# Part 30 — Scenario: Security Architecture

## 39. What Must Never Depend Only on React? ⭐⭐⭐⭐⭐

Never trust React alone for:

- authentication proof
- authorization
- ownership
- validation
- price
- inventory
- payment verification
- secret protection
- file validation

```text
React
→ UX

Server
→ trust decisions
```

---

# Part 31 — Scenario: Testing Strategy

## 40. How Would You Test a Feature? ⭐⭐⭐⭐⭐

Example CareerLoop application creation:

### Unit

```text
pure validation
salary formatting
domain transformations
```

### Component / Integration

```text
fill form
submit
pending state
success result
server error
validation
```

### E2E

```text
sign in
create application
see it in tracker
change status
verify persistence
```

Test based on risk, not arbitrary coverage numbers.

---

# Part 32 — Scenario: Refactoring

## 41. Component Is 1,000 Lines

Do not split purely by line count.

Identify responsibilities:

```text
data orchestration
form state
table rendering
filters
modal
domain calculations
```

Extract where boundaries become clearer.

Bad extraction:

```text
ComponentPart1
ComponentPart2
ComponentPart3
```

Good extraction expresses domain/UI responsibility.

---

# Part 33 — Scenario: Duplicate Logic

## 42. Two Components Share Similar Logic

Do not immediately abstract.

Ask:

```text
Is behavior truly the same?
Will it evolve together?
Is abstraction simpler than duplication?
```

If yes, possible extraction:

- pure function
- custom Hook
- reusable component
- shared domain module

Premature abstraction can couple unrelated features.

---

# Part 34 — Scenario: API Layer

## 43. Should Components Call fetch Everywhere?

For small apps, direct calls may be acceptable.

As complexity grows, a data boundary can centralize:

- request construction
- response parsing
- error normalization
- authentication integration
- domain mapping

Avoid creating a giant generic service layer that hides every request behind meaningless wrappers.

Keep domain intent visible.

---

# Part 35 — Scenario: Server Components

## 44. Server vs Client Component Boundary ⭐⭐⭐⭐⭐

Conceptually place non-interactive work on the server when the framework supports React Server Components and doing so benefits the application.

Interactive regions need client capabilities such as:

- state
- event handlers
- Effects
- browser APIs

Do not make an entire tree client-side merely because one small child is interactive.

Framework-specific directives and caching rules belong in the framework repository.

---

# Part 36 — Scenario: Suspense

## 45. Where Should Suspense Boundaries Go?

Choose UX boundaries:

```text
Can the rest of this page be useful
while this region waits?
```

Example:

```text
Profile page
├── Header
├── Suspense
│   └── Activity
└── Suspense
    └── Recommendations
```

Avoid one global spinner if independent content can progressively appear.

---

# Part 37 — Scenario: Code Review

## 46. What Do You Check in React Code Review? ⭐⭐⭐⭐⭐

### Correctness

- source of truth
- state transitions
- async races
- cleanup
- keys

### Architecture

- ownership
- boundaries
- unnecessary global state
- unnecessary Effects
- duplicated logic

### Performance

- measured expensive work
- huge DOM/list
- unnecessary high-level state
- accidental remounting

### Production

- loading/error/empty
- accessibility
- security assumptions
- tests

---

# Part 38 — Scenario: CareerLoop Architecture

## 47. Job Application Tracker

Possible feature boundaries:

```text
features/
├── applications/
├── dashboard/
├── auth/
├── follow-ups/
└── settings/
```

Application feature might contain:

```text
applications/
├── components/
├── hooks/
├── api/
├── domain/
└── types/
```

Potential state ownership:

```text
search/filter reflected in URL if shareable
form draft → local/form state
modal → local feature state
application records → server state
auth session → shared auth boundary
```

Server remains authoritative for application ownership and permissions.

---

# Part 39 — Scenario: CodeBuddy Architecture

## 48. Developer Networking Platform

Possible feature boundaries:

```text
discovery
connections
chat
profile
subscriptions
auth
```

Discovery:

```text
server paginated feed
+
local interaction state
+
optimistic connection request
```

Chat:

```text
history
+
real-time subscription
+
message composer
+
reconnection/error states
```

Subscriptions:

```text
React displays plan/payment status
server verifies payment
server updates authoritative subscription
```

This is a good interview example because it demonstrates frontend + backend boundary awareness.

---

# Part 40 — Senior-Style Tradeoff Questions

## 49. Context or Redux/External Store?

Answer:

```text
I don't choose based only on app size.

I look at update frequency, number of consumers, selector
needs, debugging requirements, persistence, async/server
state responsibilities and team conventions.

Context is useful for distributing values through a
subtree. An external store becomes useful when the state
model needs stronger subscription/selective update or
state-management capabilities.
```

---

## 50. Controlled or Uncontrolled Form?

Controlled when current value drives React behavior.

Uncontrolled when DOM-owned input state simplifies the requirement.

Large form architecture may also use specialized form tooling.

Choose based on behavior, validation and performance—not ideology.

---

## 51. Composition or Configuration?

Composition gives callers flexibility:

```jsx
<Card>
  <Card.Header />
  <Card.Body />
</Card>
```

Configuration can simplify standardized cases:

```jsx
<Alert
  variant="error"
  message="Failed"
/>
```

A good API balances flexibility and constraints.

---

# Part 41 — Common Architecture Mistakes

## 52. Avoid These ⭐⭐⭐⭐⭐

1. Globalizing all state.
2. Putting all server data into Context by default.
3. Using Effects for derived values.
4. Treating every prop chain as a reason for global state.
5. Creating giant Context providers.
6. Memoizing everything.
7. Ignoring component identity and keys.
8. Mixing API, UI and domain logic until components become unmaintainable.
9. Abstracting before reuse is understood.
10. Building generic components with dozens of boolean props.
11. Ignoring loading/error/empty states.
12. Ignoring accessibility until the end.
13. Trusting frontend permission checks.
14. Treating framework behavior as core React behavior.
15. Choosing microfrontends without organizational/deployment need.
16. Optimizing without profiling.
17. Making tests depend on implementation details.
18. Ignoring failure/retry behavior.
19. Sending data to the browser and assuming hidden UI protects it.
20. Designing architecture around folder names rather than ownership.

---

# Part 42 — Interview Questions

## 53. How Do You Decide Component Boundaries?

I separate components when they represent meaningful UI/domain responsibilities, have independent state/lifecycle concerns, improve reuse, or make a large component easier to reason about.

I do not split components solely because they exceed a certain line count.

---

## 54. Where Should State Live? ⭐⭐⭐⭐⭐

At the lowest common owner that needs to coordinate it.

Start local, lift when sharing is required, use Context for subtree distribution, and choose external state when broader state-management requirements justify it.

---

## 55. How Do You Avoid Unnecessary Re-renders?

First determine whether they are actually expensive.

Then consider:

- state colocation
- smaller ownership boundaries
- stable component identity
- removing unnecessary Effects
- memoization where beneficial
- selective external-store subscriptions
- avoiding unnecessary provider updates

A render is not automatically a performance bug.

---

## 56. How Do You Design for Failure?

Every asynchronous feature should intentionally consider:

```text
initial loading
success
empty
error
retry
refreshing
mutation pending
mutation failure
partial failure
```

The exact states depend on the feature.

---

## 57. How Do You Design a Secure React App?

I assume the browser is untrusted.

React handles presentation and client UX, while the server authenticates, authorizes, validates input, verifies ownership and enforces business invariants.

I avoid shipping secrets, avoid unsafe raw HTML, and treat client totals/roles/IDs as untrusted input.

---

# Part 43 — Interview Answer Template

## 58. Scenario Answer Formula ⭐⭐⭐⭐⭐

Use:

```text
1. Requirements
2. Source of truth
3. Ownership
4. Component/data boundaries
5. Data flow
6. Async/failure states
7. Performance only where needed
8. Accessibility
9. Security/server authority
10. Testing
11. Tradeoffs
```

Example:

> "For a job application dashboard, I would keep application records as server-owned data, keep short-lived modal/form interaction state local, and represent shareable filters in the URL where appropriate. I would organize the feature around applications rather than generic component folders, explicitly model loading/empty/error states, protect mutations with server authorization, and test the main create/update flows at integration level. If profiling showed the table was expensive, I would then consider pagination, virtualization, or targeted memoization."

---

# Part 44 — Architecture Mental Model

## 59. Complete Map ⭐⭐⭐⭐⭐

```text
                       REQUIREMENT
                            │
                            ↓
                    SOURCE OF TRUTH
                            │
       ┌────────────────────┼────────────────────┐
       ↓                    ↓                    ↓
    UI STATE            SERVER STATE         URL STATE
       │                    │                    │
 local/lifted/          data layer /          shareable
context/store           framework             navigation
       │                    │                    │
       └────────────────────┼────────────────────┘
                            ↓
                     FEATURE BOUNDARY
                            │
          ┌─────────────────┼─────────────────┐
          ↓                 ↓                 ↓
          UI             React logic       domain logic
                            │
                            ↓
                    external systems
                            │
                            ↓
                       SERVER/API
                 authoritative security
                 and business rules


Then evaluate:

loading / error / empty
accessibility
performance
testing
tradeoffs
```

---

## 60. Key Takeaways

- Architecture starts with requirements and sources of truth.
- Keep state at the narrowest useful ownership boundary.
- Separate UI state from server-owned state.
- Context distributes values; it is not automatically the best global store.
- Do not use Effects for values that can be derived during render.
- Data-fetching architecture may belong to a framework/data layer rather than manual Effects.
- Real-time subscriptions require cleanup and server authorization.
- React cannot guarantee inventory, payment, permissions, or other server invariants.
- Design reusable components around semantics and valid APIs.
- Place Error Boundaries and Suspense around meaningful UX regions.
- Optimistic UI must reconcile with authoritative results.
- State-machine thinking reduces contradictory async booleans.
- Use reducers when transitions benefit from explicit actions.
- Profile before optimizing.
- Large lists may require pagination or virtualization.
- Microfrontends should follow domain/team/deployment boundaries, not arbitrary component boundaries.
- URL state is useful for shareable/navigation-relevant state.
- Accessibility belongs in reusable primitives and architecture.
- Security assumes the browser is untrusted.
- Test features according to risk and behavior.
- Good architecture answers explain tradeoffs instead of naming tools.

---

## Next Lesson

➡️ [Lesson 76 — Complete React Revision Cheat Sheet ⭐⭐⭐⭐⭐](./76-revision-cheat-sheet.md)
