# Lesson 42 — useDeferredValue

## 1. What Is useDeferredValue?

`useDeferredValue` gives you a version of a value that may **lag behind** its latest value so urgent UI can update first.

```jsx
const deferredValue =
  useDeferredValue(value);
```

Mental model:

```text
latest value
    │
    ├── urgent UI uses latest value
    │
    └── expensive UI uses deferred value
             │
             ↓
        catches up later
```

---

## 2. Basic Search Example

```jsx
function SearchPage() {
  const [query, setQuery] = useState("");

  const deferredQuery =
    useDeferredValue(query);

  return (
    <>
      <input
        value={query}
        onChange={(event) =>
          setQuery(event.target.value)
        }
      />

      <SearchResults
        query={deferredQuery}
      />
    </>
  );
}
```

The input receives the latest `query` immediately.

The expensive results are allowed to render using `deferredQuery`.

---

## 3. Why It Helps ⭐⭐⭐⭐⭐

Without deferral:

```text
type key
  ↓
query changes
  ↓
input + expensive results
render together
```

With deferral:

```text
type key
  ↓
latest query updates
  ↓
input updates quickly
  ↓
results catch up later
```

This can improve responsiveness when downstream rendering is expensive.

---

## 4. Initial and Subsequent Renders

Without a separate initial value, the initial deferred value matches the current value.

On later updates React can:

1. render with the new source value and old deferred value,
2. attempt a background render with the new deferred value.

That background work can be interrupted.

---

## 5. Interruptible Background Rendering ⭐⭐⭐⭐⭐

Suppose:

```text
query = "r"
background render begins

before completion:
query = "re"
```

React can restart the background work with the newer value rather than requiring obsolete rendering to finish.

---

## 6. Showing Stale Content

You can indicate that visible results are behind:

```jsx
const isStale =
  query !== deferredQuery;

return (
  <div
    style={{
      opacity: isStale ? 0.5 : 1,
    }}
  >
    <SearchResults
      query={deferredQuery}
    />
  </div>
);
```

This communicates that the old results remain visible while new results prepare.

---

## 7. useDeferredValue Is Not Debounce ⭐⭐⭐⭐⭐

Debounce:

```text
wait for a time/inactivity period
before performing work
```

Deferred rendering:

```text
React prioritizes urgent rendering
and lets another value catch up
```

There is no inherent fixed `300ms` delay.

---

## 8. It Does Not Automatically Reduce Requests

If changing the deferred value eventually triggers data loading, `useDeferredValue` is still not a request-rate limiting mechanism.

If your requirement is:

```text
"send search request only after
the user stops typing for 300ms"
```

use an appropriate debounce/request strategy.

Rendering priority and request frequency are different concerns.

---

## 9. useDeferredValue vs useTransition ⭐⭐⭐⭐⭐

### useTransition

You control the state update:

```jsx
startTransition(() => {
  setQuery(value);
});
```

### useDeferredValue

You already have a value:

```jsx
const deferredQuery =
  useDeferredValue(query);
```

This is especially useful when a value arrives as a prop and you cannot control where its state update occurred.

---

## 10. Prop Example

```jsx
function ResultsPage({ query }) {
  const deferredQuery =
    useDeferredValue(query);

  return (
    <SlowResults
      query={deferredQuery}
    />
  );
}
```

The component can defer expensive consumption without owning `query`.

---

## 11. Pairing with memo ⭐⭐⭐⭐⭐

```jsx
const SlowList = memo(
  function SlowList({ query }) {
    // expensive rendering
  }
);
```

Then:

```jsx
const deferredQuery =
  useDeferredValue(query);

<SlowList query={deferredQuery} />
```

During the urgent render, `deferredQuery` may still have its old value. A memoized expensive subtree can therefore skip work while the deferred value is unchanged.

This is why deferral and a useful memoization boundary often complement each other.

---

## 12. Deferred Does Not Mean Permanently Stale

```text
source:
"r" → "re" → "rea"

deferred:
"r" --------→ "rea"
```

Intermediate background work may be interrupted, but React attempts to catch up to the latest value.

---

## 13. React 19 initialValue

Modern React supports:

```jsx
const deferredValue =
  useDeferredValue(
    value,
    initialValue
  );
```

On initial render, `initialValue` can be returned while React schedules a background render using the actual value.

Use this only when that initial behavior makes sense.

---

## 14. Do Not Use It for Correctness

Wrong mental model:

```text
business state B must update
after state A
→ useDeferredValue
```

No.

Deferral is a rendering performance strategy.

Business sequencing must be modeled explicitly.

---

## 15. useDeferredValue vs useMemo

```text
useDeferredValue
→ changes rendering priority/
  allows stale value temporarily

useMemo
→ caches calculation result
```

They may be combined:

```jsx
const deferredQuery =
  useDeferredValue(query);

const filtered =
  useMemo(
    () =>
      filterItems(
        items,
        deferredQuery
      ),
    [items, deferredQuery]
  );
```

Only do this if the calculation is actually expensive enough to justify memoization.

---

## 16. useDeferredValue vs setTimeout

```text
setTimeout
→ clock-based delay

useDeferredValue
→ React scheduling
```

They are not equivalent.

---

## 17. Suspense Connection

Deferred values work well with Suspense-aware UI.

Conceptually:

```text
new deferred value
       │
new content suspends
       │
old useful content can remain
       │
new content ready
       ↓
UI catches up
```

Suspense is covered deeply in Lesson 53.

---

## 18. CareerLoop Example

```jsx
const [searchText, setSearchText] =
  useState("");

const deferredSearch =
  useDeferredValue(searchText);
```

Then:

```jsx
<input
  value={searchText}
  onChange={(event) =>
    setSearchText(
      event.target.value
    )
  }
/>

<ApplicationResults
  query={deferredSearch}
/>
```

For thousands of complex rows, typing can stay responsive while results lag slightly.

If the list is already fast, deferral is unnecessary.

---

## 19. CodeBuddy Example

```jsx
const deferredSkillQuery =
  useDeferredValue(skillQuery);

<DeveloperDiscovery
  skillQuery={deferredSkillQuery}
/>
```

This can help expensive client-side discovery rendering.

It does not automatically debounce APIs or Socket.IO traffic.

---

## 20. When to Use It

Consider it when:

- a rapidly changing value drives expensive UI,
- urgent UI needs the latest value immediately,
- downstream UI is allowed to lag,
- you do not control or do not want to transition the source update,
- profiling shows rendering pressure.

---

## 21. Common Mistakes ⭐⭐⭐⭐⭐

1. Calling it debounce.
2. Expecting a fixed delay.
3. Expecting it to reduce API requests automatically.
4. Deferring UI that must update immediately.
5. Using it for correctness.
6. Deferring cheap UI unnecessarily.
7. Forgetting that a useful memo boundary may be needed for the expensive subtree.
8. Confusing it with `useMemo`.
9. Assuming background rendering uses another JS thread.

---

## 22. Interview Questions ⭐⭐⭐⭐⭐

### What is useDeferredValue?

A Hook that lets non-urgent UI temporarily use an older value so urgent rendering remains responsive.

### Does it debounce?

No.

### Is there a fixed delay?

No.

### useDeferredValue vs useTransition?

`useTransition` marks an update you control as non-blocking; `useDeferredValue` defers consumption of an existing value.

### Does it prevent network requests?

No.

### Why can memo help?

A memoized expensive subtree can skip the urgent render while its deferred prop remains unchanged.

### Can you indicate stale content?

Yes. Compare the latest value with its deferred value.

---

## 23. Complete Mental Model

```text
              value changes
                   │
         ┌─────────┴─────────┐
         ↓                   ↓
   latest value        deferred value
         │                   │
         ↓                   ↓
    urgent UI          slow subtree
    updates now        may stay stale
                             │
                             ↓
                     background render
                     can be interrupted
                             │
                             ↓
                        catches up
```

---

## 24. Key Takeaways

- `useDeferredValue` lets expensive UI lag behind a changing value.
- Urgent UI still uses the latest value immediately.
- Background rendering can be interrupted.
- There is no fixed delay.
- It is not debounce.
- It does not automatically reduce network requests.
- It is useful when deferring consumption is easier than controlling the original update.
- Memoization can make an expensive deferred subtree more effective.
- React 19 supports an optional `initialValue`.
- Never depend on deferred timing for application correctness.

---

## Next Lesson

➡️ [Lesson 43 — Lazy Loading, React.lazy and Code Splitting](./43-lazy-code-splitting.md)
