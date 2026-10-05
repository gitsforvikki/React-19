# Lesson 43 — Lazy Loading, React.lazy and Code Splitting

## 1. The Problem: Shipping Too Much JavaScript

Imagine an application containing:

```text
Dashboard
Analytics
Settings
Admin
Rich Editor
Charts
```

If users must download and evaluate code for every feature before they need it, initial loading can become unnecessarily expensive.

**Code splitting** divides application code into chunks that can be loaded when required.

---

## 2. What Is Lazy Loading?

Lazy loading means delaying loading of a resource until it is needed.

```text
initial screen
→ load critical code

later user opens Analytics
→ load Analytics code
→ render Analytics
```

React provides `lazy` for lazy component loading.

---

## 3. React.lazy ⭐⭐⭐⭐⭐

```jsx
import {
  lazy,
  Suspense,
} from "react";

const Analytics =
  lazy(() =>
    import("./Analytics.jsx")
  );
```

Then:

```jsx
<Suspense
  fallback={<p>Loading…</p>}
>
  <Analytics />
</Suspense>
```

---

## 4. Dynamic import()

```js
import("./Analytics.jsx")
```

returns a Promise for a module.

Bundlers can use dynamic imports as code-splitting boundaries.

```text
main application chunk
       │
       └── analytics chunk
               ↑
          loaded later
```

---

## 5. What lazy Expects ⭐⭐⭐⭐⭐

The function passed to `lazy` returns a Promise/module.

The conventional form resolves to a module whose component is its default export:

```jsx
export default function Analytics() {
  return <div>Analytics</div>;
}
```

Then:

```jsx
const Analytics =
  lazy(() =>
    import("./Analytics.jsx")
  );
```

React caches the Promise and resolved component, so it does not repeatedly invoke the load function just because the component renders again.

---

## 6. Why Suspense Is Needed

While code is unavailable, the lazy component suspends.

```text
render lazy component
        │
code ready?
 ┌──────┴──────┐
yes            no
 ↓              ↓
render        suspend
component       │
                ↓
          nearest Suspense
                │
                ↓
             fallback
```

---

## 7. Suspense Boundary Placement ⭐⭐⭐⭐⭐

Too broad:

```jsx
<Suspense fallback={<Spinner />}>
  <EntireApplication />
</Suspense>
```

A small lazy feature could replace too much useful UI.

More focused:

```jsx
<Dashboard>
  <Sidebar />

  <Suspense
    fallback={<ChartSkeleton />}
  >
    <AnalyticsChart />
  </Suspense>
</Dashboard>
```

Choose boundaries around meaningful loading regions.

---

## 8. Interaction-Based Lazy Loading

```jsx
const SettingsPanel =
  lazy(() =>
    import("./SettingsPanel.jsx")
  );

function Page() {
  const [open, setOpen] =
    useState(false);

  return (
    <>
      <button
        onClick={() =>
          setOpen(true)
        }
      >
        Settings
      </button>

      {open && (
        <Suspense
          fallback={
            <p>Loading settings…</p>
          }
        >
          <SettingsPanel />
        </Suspense>
      )}
    </>
  );
}
```

The optional feature's code need not be part of the initial execution path.

---

## 9. Route-Level Splitting Concept

Large pages are natural splitting boundaries:

```text
Home chunk
Dashboard chunk
Analytics chunk
Admin chunk
```

Exact router/framework integration differs.

Framework-specific Next.js implementation belongs in the separate Next.js repository.

---

## 10. Good Candidates

Potential candidates include:

- large charting features
- rich text editors
- admin-only screens
- rarely opened complex dialogs
- analytics dashboards
- large route/page features

The benefit should justify another loading boundary.

---

## 11. Do Not Lazy Load Everything ⭐⭐⭐⭐⭐

Too many tiny chunks can create:

- network overhead
- waterfalls
- fallback flashes
- more complexity
- worse perceived performance

```text
every icon/button/card
→ separate lazy chunk
```

is not a good general strategy.

Split meaningful features.

---

## 12. Network Waterfalls

Poor dependency loading can become:

```text
Page chunk
   ↓
loads component A
   ↓
A reveals dependency B
   ↓
B loads
```

Sequential loading increases latency.

Code splitting is a tradeoff:

```text
less initial code
vs
later loading cost
```

---

## 13. Preloading Concept

Sometimes you know a lazy resource is likely to be needed soon.

Examples:

```text
user hovers Analytics
→ likely next action

important next screen known
→ start loading earlier
```

Modern React includes advanced resource preloading/preinitialization APIs.

Those resource APIs are covered later in the React 19 section.

---

## 14. Conditional Rendering Is Not Code Splitting ⭐⭐⭐⭐⭐

This:

```jsx
import SettingsPanel
  from "./SettingsPanel";

{open && <SettingsPanel />}
```

conditionally **renders** the component.

Its module was still statically imported.

Lazy loading uses:

```jsx
const SettingsPanel =
  lazy(() =>
    import("./SettingsPanel")
  );
```

This distinction is important for interviews.

---

## 15. Lazy Loading vs Render Performance

Code splitting primarily affects:

- downloading JavaScript
- parsing/evaluating JavaScript
- initial bundle work

It does not directly solve:

```text
"this mounted component
re-renders too often"
```

For render performance, investigate state architecture, algorithms, memoization, scheduling, and DOM size.

---

## 16. lazy vs memo

```text
lazy
→ WHEN component code loads

memo
→ whether component rendering
  can be skipped
```

Different optimization layers.

---

## 17. lazy vs useMemo

```text
lazy
→ component code loading

useMemo
→ calculation result caching
```

They solve unrelated problems.

---

## 18. Loading Errors ⭐⭐⭐⭐⭐

Lazy imports can fail due to:

- network failure
- unavailable deployment chunks
- server/CDN problems

Suspense handles the waiting state.

An **Error Boundary** should handle suitable failures and recovery UI.

```text
ErrorBoundary
  └── Suspense
       └── LazyComponent
```

Error Boundaries are covered in Lesson 52.

---

## 19. Declare lazy at Module Scope ⭐⭐⭐⭐⭐

Do not do this:

```jsx
function Page() {
  const Analytics =
    lazy(() =>
      import("./Analytics")
    );

  return <Analytics />;
}
```

Prefer:

```jsx
const Analytics =
  lazy(() =>
    import("./Analytics")
  );

function Page() {
  return <Analytics />;
}
```

Declaring component types during rendering creates unstable identity and can reset state.

---

## 20. Component Identity Connection

From Lesson 18:

```text
component type
+ tree position/key
→ component identity
```

Creating a new component definition during each parent render breaks stable type identity.

Keep lazy declarations outside component functions.

---

## 21. Named Export Adapter

If a module exposes a named export:

```jsx
export function Analytics() {
  return <div>...</div>;
}
```

you can adapt it:

```jsx
const Analytics =
  lazy(() =>
    import("./Analytics.jsx")
      .then((module) => ({
        default: module.Analytics,
      }))
  );
```

A dedicated module with a default component export may sometimes be simpler.

---

## 22. Loading UX Matters ⭐⭐⭐⭐⭐

A generic spinner is not always the best fallback.

Consider:

- skeletons matching content shape
- preserving existing content
- avoiding layout shifts
- meaningful pending feedback
- appropriately sized Suspense boundaries

Performance includes perceived continuity, not only bundle size.

---

## 23. Avoid Fallback Flicker

If code loads quickly, a rapidly flashing fallback may make the experience feel worse.

Transitions and thoughtful Suspense placement can help preserve useful content during navigation-like changes.

Do not mechanically add spinners around everything.

---

## 24. CareerLoop Example

A large analytics feature is a reasonable candidate:

```jsx
const AnalyticsDashboard =
  lazy(() =>
    import(
      "./AnalyticsDashboard"
    )
  );
```

Conceptually:

```text
job tracking core
→ initial code

analytics/charts
→ load when requested
```

---

## 25. CodeBuddy Example

Potential split decisions:

```text
core discovery/feed
→ likely eager

heavy profile editor
→ maybe lazy

subscription management
→ maybe lazy

admin analytics
→ strong candidate
```

Do not lazy-load critical first-screen interaction simply because you can.

---

## 26. Measuring Code Splitting

Check:

- initial JS size
- chunks loaded on startup
- when lazy chunks are requested
- loading waterfalls
- fallback frequency
- first interaction
- later navigation cost

Use browser network/performance tools and bundler analysis where appropriate.

---

## 27. Common Mistakes ⭐⭐⭐⭐⭐

1. Thinking conditional rendering automatically splits code.
2. Lazy-loading every tiny component.
3. Creating network waterfalls.
4. Using one giant Suspense boundary.
5. Declaring `lazy()` inside a component.
6. Ignoring import failures.
7. Using poor loading fallbacks.
8. Lazy-loading immediately needed critical UI.
9. Confusing bundle optimization with render optimization.
10. Assuming smaller initial JS always means better total UX.

---

## 28. Interview Questions ⭐⭐⭐⭐⭐

### What is code splitting?

Breaking JavaScript into separate chunks so all application code does not need to load initially.

### What is React.lazy?

An API for lazily loading a component's code.

### Why use Suspense with lazy?

To render fallback UI while the lazy component's code is loading.

### Does conditional rendering code-split automatically?

No.

### Where should lazy declarations normally be placed?

At module scope.

### Why not lazy-load every component?

Excessive chunks can create network overhead, waterfalls, and poor loading UX.

### lazy vs memo?

`lazy` optimizes code loading; `memo` can optimize repeated rendering.

### What if a lazy import fails?

Use an Error Boundary for appropriate error handling; Suspense handles waiting, not error recovery.

---

## 29. Complete Mental Model

```text
          Application JavaScript
                  │
        ┌─────────┴──────────┐
        ↓                    ↓
 critical code          optional code
        │                    │
        ↓                    ↓
 initial bundle         separate chunk
                             │
                       feature needed
                             │
                             ↓
                         lazy import
                             │
                      ┌──────┴──────┐
                      ↓             ↓
                   waiting         ready
                      │             │
                      ↓             ↓
                  Suspense       component
                  fallback        renders
```

---

## 30. Key Takeaways

- Code splitting reduces unnecessary initial JavaScript.
- `lazy` loads component code on demand.
- Dynamic `import()` gives bundlers an asynchronous module boundary.
- Lazy components should use an appropriate Suspense boundary.
- Choose meaningful split points.
- Conditional rendering alone is not code splitting.
- Keep lazy component declarations at module scope.
- Loading UX and Error Boundaries matter.
- Avoid excessive tiny chunks and waterfalls.
- Lazy loading optimizes code delivery, not component re-rendering.
- Measure both initial load and later interaction.
- Framework-specific implementations belong in their respective framework repository.

---

## Next Lesson

➡️ [Lesson 44 — React Performance Profiling and Optimization](./44-performance-profiling.md)
