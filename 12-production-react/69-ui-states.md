# Lesson 69 — Error, Loading and Empty-State Design

## 1. Production UI Is More Than the Happy Path

A feature is not complete when only this works:

```text
data exists
→ render data
```

Real UI must handle:

```text
initial
loading
success
empty
error
refreshing
submitting
partial failure
offline/retry
```

A production React developer designs these states intentionally.

---

# Part 1 — Model States Clearly

## 2. The Boolean Explosion Problem

A component may start with:

```jsx
const [loading, setLoading] =
  useState(false);

const [error, setError] =
  useState(false);

const [empty, setEmpty] =
  useState(false);
```

This permits impossible combinations:

```text
loading = true
error = true
empty = true
```

What should the UI show?

---

## 3. Prefer Mutually Exclusive Status Where Appropriate

```jsx
const [
  status,
  setStatus,
] = useState("idle");
```

Possible values:

```text
idle
loading
success
error
```

Empty can often be derived from successful data:

```js
const isEmpty =
  status === "success" &&
  items.length === 0;
```

This reduces contradictory state.

---

# Part 2 — State Machine Mental Model

## 4. Async UI as State Transitions

```text
          ┌─────────┐
          │  idle   │
          └────┬────┘
               │ load
               ↓
          ┌─────────┐
          │ loading │
          └──┬───┬──┘
             │   │
       success   failure
             │   │
             ↓   ↓
       ┌────────┐ ┌───────┐
       │success │ │ error │
       └───┬────┘ └───┬───┘
           │          │ retry
           │ refresh  └──────┐
           └──────────────┐  │
                          ↓  ↓
                       loading
```

You do not need a state-machine library to benefit from state-machine thinking.

---

# Part 3 — Loading States

## 5. Initial Loading

When no useful content exists yet:

```jsx
if (
  status === "loading" &&
  !data
) {
  return <PageSkeleton />;
}
```

A skeleton may preserve layout better than a full-page spinner.

---

## 6. Refreshing Is Different ⭐⭐⭐⭐⭐

Suppose useful data is already visible.

A refresh begins.

Bad UX:

```text
content disappears
→ giant spinner
→ content returns
```

Better:

```text
keep existing content
+
show subtle refresh state
```

For example:

```jsx
<>
  {isRefreshing && (
    <RefreshIndicator />
  )}

  <ApplicationList
    applications={data}
  />
</>
```

Initial loading and background refreshing are different states.

---

## 7. Loading State Should Match Scope

Button action:

```jsx
<button
  disabled={isSubmitting}
>
  {isSubmitting
    ? "Saving..."
    : "Save"}
</button>
```

Do not block the entire page for a small button mutation unless the whole page truly depends on it.

Principle:

> Loading UI should usually be scoped to the operation it represents.

---

# Part 4 — Prevent Duplicate Actions

## 8. Pending Mutations

During submission:

- disable or guard duplicate action where appropriate
- communicate progress
- preserve context
- avoid accidental duplicate requests

Example:

```jsx
<button
  disabled={isPending}
  onClick={handleSubmit}
>
  {isPending
    ? "Submitting..."
    : "Submit"}
</button>
```

Do not assume disabling is the only protection; authoritative backend operations should also be designed safely.

---

# Part 5 — Empty States

## 9. Empty Is Not an Error ⭐⭐⭐⭐⭐

```text
request succeeded
+
zero items
=
empty state
```

Do not display:

```text
Something went wrong
```

when the real result is simply empty.

---

## 10. Different Types of Empty State

### First-use empty

```text
You have not created
any applications yet.
```

Useful action:

```text
Add your first application
```

### Filtered empty

```text
No applications match
"Frontend + Remote".
```

Useful action:

```text
Clear filters
```

### Search empty

```text
No developers found
for "xyz".
```

These states have different causes and should often have different messages/actions.

---

## 11. Empty State Should Explain the Next Step

Weak:

```text
No data.
```

Better:

```text
No applications yet.

Track your first application
to start building your pipeline.

[Add application]
```

A useful empty state answers:

```text
What happened?
Why?
What can I do next?
```

---

# Part 6 — Error States

## 12. Error UI Should Match Scope ⭐⭐⭐⭐⭐

Page-level failure:

```text
Unable to load applications
[Retry]
```

Widget failure:

```text
Analytics unavailable
[Retry]
```

Field error:

```text
Email
[invalid value]
Enter a valid email address.
```

Do not replace an entire application because one optional widget failed.

---

## 13. User Errors vs System Errors

User-correctable:

```text
Password must contain...
Coupon is expired.
Email is invalid.
```

System/remote:

```text
Network unavailable.
Server failed.
Unexpected rendering error.
```

Messages should reflect what the user can actually do.

---

## 14. Do Not Expose Internal Error Details

Avoid showing users:

```text
SQLSTATE...
stack trace...
database host...
internal exception...
```

User UI should be safe and actionable.

Detailed diagnostics belong in controlled logging/observability systems.

---

# Part 7 — Retry Design

## 15. Retry Only When Retry Makes Sense

Useful retry:

```text
temporary network request failed
→ Retry
```

Not useful:

```text
user lacks permission
→ Retry forever
```

Retry behavior should understand failure type.

---

## 16. Preserve User Work

If form submission fails:

Bad:

```text
error
→ clear entire form
```

Better:

```text
error
→ preserve entered values
→ show actionable feedback
→ allow retry/correction
```

Production error design protects user effort.

---

# Part 8 — Error Boundaries

## 17. Rendering Errors ⭐⭐⭐⭐⭐

Error Boundaries handle rendering-tree failures:

```text
component throws during render
      ↓
nearest Error Boundary
      ↓
fallback UI
```

They do not automatically replace all ordinary request/event error handling.

---

## 18. Place Boundaries Around Failure Domains

```text
Dashboard
├── Header
├── ErrorBoundary
│    └── Analytics
└── ErrorBoundary
     └── ApplicationList
```

If Analytics crashes:

```text
Header remains
ApplicationList remains
Analytics fallback appears
```

This creates resilient UI.

---

# Part 9 — Suspense Boundaries

## 19. Loading Boundaries

From Lesson 66:

```jsx
<Suspense
  fallback={
    <ApplicationSkeleton />
  }
>
  <ApplicationList />
</Suspense>
```

Suspense lets loading behavior align with component-tree boundaries for supported resources.

Error Boundary:

```text
failure boundary
```

Suspense:

```text
not-ready boundary
```

---

# Part 10 — Combined Boundary Design

## 20. Example

```jsx
<ErrorBoundary
  fallback={
    <ApplicationsError />
  }
>
  <Suspense
    fallback={
      <ApplicationsSkeleton />
    }
  >
    <ApplicationList />
  </Suspense>
</ErrorBoundary>
```

Mental model:

```text
pending
→ Suspense fallback

fulfilled with zero
→ EmptyState

fulfilled with items
→ ApplicationList

rejected/render failure
→ ErrorBoundary fallback
```

---

# Part 11 — Partial Success

## 21. Production Pages Often Have Independent Data

Dashboard:

```text
Profile      ✅
Applications ✅
Analytics    ❌
Notifications ⏳
```

The page can still be useful.

Do not always reduce the entire page to:

```text
ERROR
```

Design sections with independent loading/error boundaries when appropriate.

---

# Part 12 — Stale Data and Refresh Failure

## 22. Refresh Failure Is Different From Initial Failure ⭐⭐⭐⭐⭐

Initial failure:

```text
no data available
+
request failed
→ error state
```

Refresh failure:

```text
old useful data exists
+
refresh failed
→ keep old data
+ show refresh warning
```

Do not throw away useful information unnecessarily.

---

# Part 13 — Optimistic UI

## 23. Optimistic State Adds Another Layer

```text
authoritative state
+
optimistic state
+
pending mutation
+
possible rollback
```

Example:

```text
Connect clicked
→ immediately show Connected
→ request pending
→ success: keep
→ failure: reconcile/rollback
```

The UI should communicate failures without leaving impossible state behind.

See Lesson 60 for `useOptimistic`.

---

# Part 14 — Form States

## 24. A Form Has Multiple UI States

```text
idle
editing
validating
submitting
success
field error
submission error
```

Avoid one generic:

```js
loading
```

for every form operation.

Use names that express the actual state:

```text
isSubmitting
isValidating
isPending
```

when booleans truly represent independent dimensions.

---

# Part 15 — Accessibility

## 25. Loading Accessibility

Consider:

- clear text, not only animation
- `aria-busy` on regions being updated where appropriate
- avoid focus loss
- avoid disabling unrelated controls

Example:

```jsx
<section
  aria-busy={isLoading}
>
  ...
</section>
```

---

## 26. Error Accessibility

Errors should be:

- associated with relevant controls
- understandable without color alone
- discoverable by assistive technology
- focus-managed when necessary

Detailed accessibility is covered in Lesson 70.

---

# Part 16 — Layout Stability

## 27. Avoid Unnecessary Layout Shift

Loading UI should approximately preserve final dimensions when practical.

```text
final card
████████████

skeleton
████████████
```

is often smoother than:

```text
spinner
  ●

then suddenly
████████████
```

---

# Part 17 — Avoid Spinner Hell

## 28. Too Many Independent Indicators

Bad:

```text
spinner spinner spinner
spinner spinner spinner
```

A dashboard with dozens of tiny spinners feels unstable.

Group loading states around meaningful UX regions.

This connects to Suspense reveal-group design.

---

# Part 18 — Error Logging vs Error UI

## 29. Different Responsibilities

User-facing:

```text
"We couldn't load analytics.
Try again."
```

Developer-facing observability:

```text
error type
stack
request ID
feature
environment
context
```

Do not make users your logging system.

---

# Part 19 — State Priority

## 30. Rendering Decision Order

For a simple request:

```jsx
if (isLoading) {
  return <Skeleton />;
}

if (error) {
  return (
    <ErrorState />
  );
}

if (items.length === 0) {
  return (
    <EmptyState />
  );
}

return (
  <ItemList
    items={items}
  />
);
```

This is fine when those states are genuinely exclusive.

For richer UIs, refreshing/error-with-stale-data may coexist, so use a model that represents those dimensions intentionally.

---

# Part 20 — Do Not Store Derived Empty State

## 31. Avoid Redundant State

Bad:

```jsx
const [
  isEmpty,
  setIsEmpty,
] = useState(false);
```

when emptiness can be derived:

```jsx
const isEmpty =
  status === "success" &&
  items.length === 0;
```

Redundant state can become inconsistent.

---

# Part 21 — CareerLoop Example

## 32. Application List

```text
initial load
→ ApplicationListSkeleton

success + applications
→ ApplicationList

success + no applications
→ "Track your first application"

success + filters + no match
→ "No applications match these filters"
  [Clear filters]

refreshing
→ keep list + subtle progress

initial request failure
→ error panel + Retry

refresh failure
→ keep old list + warning
```

This is production UI thinking.

---

# Part 22 — CodeBuddy Example

## 33. Discovery Feed

Possible states:

```text
loading first cards
→ card skeletons

developers available
→ discovery cards

no more candidates
→ end-of-feed empty state

filters too strict
→ filtered empty state

network failed
→ retry

connection request pending
→ disable/mark relevant action

connection failed
→ restore authoritative state
  + show actionable feedback
```

One generic `loading` boolean cannot communicate all of this well.

---

# Part 23 — Testing UI States

## 34. Test More Than Success

Important cases:

```text
loading
success
empty
filtered empty
error
retry
refreshing with old data
mutation pending
mutation failure
partial section failure
```

A feature that tests only the happy path is not fully production-tested.

---

# Part 24 — Common Mistakes

## 35. Common Mistakes

1. Designing only the success state.
2. Using contradictory booleans for mutually exclusive states.
3. Treating empty results as errors.
4. Using the same empty message for first-use and filtered-empty states.
5. Replacing useful stale data with a full spinner during refresh.
6. Blocking the whole page for a local operation.
7. Clearing user input after submission failure.
8. Showing technical/internal errors directly to users.
9. Offering retry for permanent authorization/validation failures.
10. Letting one optional widget failure destroy an entire dashboard.
11. Using one generic `loading` flag for unrelated operations.
12. Storing derived empty state separately.
13. Ignoring layout shift.
14. Showing many tiny spinners without meaningful grouping.
15. Forgetting accessibility in loading/error feedback.
16. Testing only successful data.

---

# Part 25 — UI State Checklist

## 36. Before Shipping a Feature, Ask

```text
What does the user see when:

1. data has not loaded yet?
2. data loads successfully?
3. successful data is empty?
4. filters produce zero matches?
5. the initial request fails?
6. refresh fails but old data exists?
7. a mutation is pending?
8. a mutation fails?
9. only one section fails?
10. the user retries?
11. the network is slow?
12. assistive technology encounters the update?
```

---

# Part 26 — Interview Questions

## 37. How Do You Model Loading/Error/Success State?

For mutually exclusive request states, I often use a status such as `idle | loading | success | error` rather than several contradictory booleans. I derive states such as empty from successful data where possible.

---

## 38. Empty State vs Error State?

Empty means the operation succeeded but returned no relevant content.

Error means the operation failed.

They require different messaging and recovery actions.

---

## 39. Initial Loading vs Refreshing?

Initial loading has no useful data yet.

Refreshing can keep existing data visible while newer data loads.

---

## 40. Why Use Error Boundaries?

To catch rendering-tree failures and provide scoped fallback UI so one failing area does not necessarily break unrelated parts of the application.

---

## 41. Why Use Suspense Boundaries?

For Suspense-aware resources, they define where fallback UI appears and how meaningful sections progressively reveal.

---

## 42. Why Avoid Multiple Boolean Flags?

Independent booleans can represent impossible combinations. A mutually exclusive status/state-machine model often makes valid transitions clearer.

---

# Part 27 — Interview-Ready Summary

## 43. Final Answer

```text
For production React UI I explicitly design loading,
success, empty, error, refreshing and mutation states
instead of implementing only the happy path.

For mutually exclusive async states I prefer a status
or state-machine style model rather than contradictory
booleans. I distinguish initial loading from refreshing
so useful stale data can remain visible, and I derive
empty state from successful data where possible.

I scope loading and error UI to the operation or feature
that failed. Suspense can define loading boundaries for
supported resources, while Error Boundaries isolate
rendering failures.

I also preserve user work after recoverable failures,
provide meaningful retry actions, avoid exposing
internal error details, and test every important UI
state rather than only success.
```

---

# Part 28 — Mental Model

## 44. Production UI State ⭐⭐⭐⭐⭐

```text
                    ASYNC FEATURE
                         │
                         ↓
                      loading
                    ┌────┴────┐
                    ↓         ↓
                 success     error
                    │          │
             ┌──────┴─────┐    └── retry
             ↓            ↓
           data          empty
             │
             └── refresh
                   │
          keep useful old data
          + show refresh state
                   │
              ┌────┴────┐
              ↓         ↓
           success    refresh error
              │         │
           new data   old data +
                      warning


Mutation:
idle → pending → success
              └→ failure / reconcile


Boundary design:
Suspense      → not ready
ErrorBoundary → rendering failure
EmptyState    → successful zero result
```

---

## 45. Key Takeaways

- Production UI includes more than success.
- Model async states intentionally.
- Avoid contradictory booleans for mutually exclusive states.
- State-machine thinking improves correctness even without a state-machine library.
- Initial loading and refreshing are different UX states.
- Scope loading indicators to the operation.
- Prevent or safely handle duplicate mutations.
- Empty is a successful state, not an error.
- First-use, search and filtered empty states may need different messages.
- Error UI should be scoped and actionable.
- Do not expose internal diagnostics to users.
- Retry only when retry can help.
- Preserve user work after recoverable failures.
- Error Boundaries isolate rendering failures.
- Suspense boundaries coordinate not-ready UI for supported resources.
- Partial success can keep a page useful.
- Refresh failure should not necessarily remove useful stale data.
- Optimistic UI needs failure reconciliation.
- Accessibility and layout stability are part of state design.
- Avoid spinner-heavy interfaces.
- Derive empty state instead of redundantly storing it.
- Test loading, empty, error, refreshing and mutation paths.

---

## Next Lesson

➡️ [Lesson 70 — Accessibility in React](./70-accessibility.md)
