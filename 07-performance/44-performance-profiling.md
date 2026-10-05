# Lesson 44 — React Performance Profiling and Optimization

## 1. The Most Important Performance Rule ⭐⭐⭐⭐⭐

> **Measure before optimizing.**

A component can render frequently and still be fast.

Another component can render rarely but perform expensive work.

Use this process:

```text
observe a real problem
        ↓
reproduce it
        ↓
measure/profile
        ↓
find the bottleneck
        ↓
apply targeted optimization
        ↓
measure again
```

---

## 2. Where Performance Problems Come From

A React application can feel slow because of:

```text
React rendering
JavaScript calculations
too much DOM
layout / paint
network requests
large JS bundles
images/assets
third-party libraries
Effect loops
poor state architecture
```

The correct fix depends on the actual bottleneck.

---

## 3. Render Does Not Equal DOM Update ⭐⭐⭐⭐⭐

Recall:

```text
Trigger
  ↓
Render
  ↓
Reconciliation
  ↓
Commit
  ↓
Browser paint
```

A component function running does not mean its whole DOM subtree changed.

Therefore:

```text
console.log("render")
appeared many times

≠

application is necessarily slow
```

---

## 4. React DevTools Profiler ⭐⭐⭐⭐⭐

The React DevTools Profiler helps investigate React rendering.

It can help answer:

- which components rendered?
- which commits were expensive?
- how long did rendering take?
- what happened during the slow interaction?
- which subtree deserves investigation?

Basic workflow:

```text
start profiling
      ↓
perform slow interaction
      ↓
stop recording
      ↓
inspect commits/components
```

---

## 5. Profile a Specific Interaction

Bad target:

```text
"make my React app faster"
```

Good target:

```text
"typing becomes slow when
5,000 applications are visible"
```

or:

```text
"opening Analytics takes too long"
```

Performance optimization works best when the problem is reproducible.

---

## 6. Development vs Production ⭐⭐⭐⭐⭐

Development includes extra diagnostics and may include Strict Mode behavior.

Therefore:

- use development tooling to investigate,
- test with realistic data,
- validate important performance conclusions in a production-like build.

Do not remove Strict Mode just to make development render logs look smaller.

---

## 7. Check State Placement First ⭐⭐⭐⭐⭐

Suppose:

```text
App
 ├── searchText state
 ├── Header
 ├── Sidebar
 ├── Dashboard
 └── Search
```

Typing updates `App`, so a large subtree participates in rendering.

If only Search needs the state:

```text
App
 ├── Header
 ├── Sidebar
 ├── Dashboard
 └── Search
      └── searchText state
```

State colocation can reduce rendering scope without any memoization API.

---

## 8. Check for Unnecessary Effects ⭐⭐⭐⭐⭐

Poor derived-state pattern:

```jsx
const [filtered, setFiltered] =
  useState([]);

useEffect(() => {
  setFiltered(
    items.filter(matchesFilter)
  );
}, [items, matchesFilter]);
```

Flow:

```text
render
→ Effect
→ setState
→ another render
```

If `filtered` is purely derived, calculate it during rendering.

If the calculation is expensive, then consider `useMemo`.

---

## 9. Check Expensive Calculations

```jsx
const result =
  expensiveTransform(data);
```

If measurement confirms this is costly:

1. improve the algorithm,
2. reduce input data,
3. calculate less often,
4. use `useMemo` if appropriate,
5. consider a Web Worker for truly CPU-heavy non-React work.

Memoization should not replace algorithmic thinking.

---

## 10. Check Expensive Child Renders

A strong `memo` candidate:

```text
parent renders frequently
+
child render is expensive
+
child props are usually unchanged
```

Then:

```jsx
const Child =
  memo(ChildComponent);
```

Check that object, array, and function props are not needlessly changing identity.

---

## 11. Check Function Identity

If a memoized child gets:

```jsx
onSave={() => save(id)}
```

the function reference changes each render.

If profiling shows that this prevents a valuable memo skip:

```jsx
const handleSave =
  useCallback(
    () => save(id),
    [id]
  );
```

Do not use `useCallback` everywhere by default.

---

## 12. Check Large Lists ⭐⭐⭐⭐⭐

Thousands of DOM nodes can be expensive even with good React code.

Potential solutions:

- pagination
- infinite loading
- virtualization/windowing
- rendering fewer details

Virtualization:

```text
10,000 records
      ↓
viewport displays ~20
      ↓
render only visible
or nearby rows
```

Reducing work is often better than caching all of it.

---

## 13. Check Update Priority

If expensive non-urgent rendering blocks an urgent interaction:

```text
useTransition
or
useDeferredValue
```

may improve responsiveness.

These APIs primarily **schedule** work; they do not necessarily eliminate the work.

---

## 14. Check Bundle Size

If startup is slow because too much JavaScript is shipped:

```text
memo
useMemo
useCallback
```

do not solve the main problem.

Investigate:

- code splitting
- `lazy`
- heavy dependencies
- optional features loaded eagerly
- bundle analysis

Optimize the correct layer.

---

## 15. Check Network and Data

A slow API cannot be fixed with `memo`.

Potential data improvements include:

- caching
- deduplication
- pagination
- prefetching
- cancellation of obsolete requests
- avoiding request waterfalls
- server/database optimization

Keep network performance separate from React render performance.

---

## 16. Check Browser Work

Even efficient React code can trigger expensive:

- layout
- paint
- large DOM processing
- CSS effects
- animations
- image decoding

Use browser performance tooling when the bottleneck is outside React rendering.

---

## 17. Performance Optimization Hierarchy ⭐⭐⭐⭐⭐

A useful mental order:

```text
1. Correct architecture
2. State colocation
3. Remove unnecessary Effects
4. Avoid redundant state
5. Improve algorithms/data size
6. Reduce rendered DOM
7. Profile the interaction
8. Target memoization
9. Schedule non-urgent work
10. Split/load code appropriately
11. Measure again
```

The exact order depends on the problem, but architecture usually beats blanket memoization.

---

## 18. Referential Stability

Remember:

```js
Object.is({}, {}) // false
Object.is([], []) // false
Object.is(
  () => {},
  () => {}
) // false
```

New references can affect:

- `memo` prop comparison
- Hook dependencies
- Effects
- memoized calculations

But stabilize references only when identity matters at a real boundary.

---

## 19. React.memo Review

Use when:

```text
expensive component
+
frequent parent rendering
+
same props often
```

Avoid automatic use for cheap components or constantly changing props.

---

## 20. useMemo Review

Use when:

```text
expensive pure calculation
+
dependencies often unchanged
```

or when stable calculated identity enables a meaningful optimization.

Never use it for side effects or correctness.

---

## 21. useCallback Review

Use when stable function identity matters, often:

```text
callback
→ memoized child
```

or:

```text
function identity
→ meaningful Hook dependency
```

Do not wrap every event handler.

---

## 22. useTransition Review

Use when:

```text
non-urgent state update
→ expensive rendering
→ urgent interaction must remain responsive
```

Keep controlled input state urgent.

---

## 23. useDeferredValue Review

Use when:

```text
rapidly changing value
→ expensive consumer UI
→ consumer may temporarily lag
```

It is not debounce.

---

## 24. lazy Review

Use when:

```text
large/optional feature
→ not needed initially
→ load code on demand
```

Do not split every tiny component.

---

## 25. React Compiler Perspective ⭐⭐⭐⭐⭐

React Compiler can automatically memoize eligible components, values, and functions.

Modern performance mindset:

```text
write pure idiomatic React
        ↓
use good state architecture
        ↓
avoid unnecessary Effects
        ↓
compiler can optimize eligible code
        ↓
profile remaining bottlenecks
        ↓
manual optimization where justified
```

Manual memoization remains relevant for:

- existing codebases
- projects without the compiler
- precise control
- library interoperability
- understanding performance
- interviews

---

## 26. Compiler Does Not Fix Everything

Automatic memoization does not magically solve:

- huge images
- slow APIs
- enormous DOM trees
- poor algorithms
- request waterfalls
- expensive third-party code
- CPU-heavy non-React work
- bad loading architecture

React performance has multiple layers.

---

## 27. Measuring Focused JavaScript Work

For a focused calculation:

```js
console.time("rank developers");

const result =
  rankDevelopers(developers);

console.timeEnd("rank developers");
```

This can help investigation.

For complete UI behavior, use proper profiling rather than relying only on micro-timings.

---

## 28. Browser Performance Tools

Browser tooling can reveal:

- long main-thread tasks
- scripting cost
- layout
- paint
- network waterfalls
- memory behavior
- resource loading

React Profiler explains React work.

Browser performance tools explain the wider runtime.

---

## 29. Why console.log Can Mislead

```jsx
console.log("rendered");
```

tells you code executed.

It does not tell you:

- whether the render was expensive,
- whether DOM changed,
- whether paint was expensive,
- whether production is slow,
- whether memoization will help.

Logs are debugging signals, not complete performance measurements.

---

## 30. Avoid Premature Optimization ⭐⭐⭐⭐⭐

Over-memoized code often becomes:

```jsx
const x = useMemo(...);
const y = useCallback(...);
const Z = memo(...);
```

everywhere.

Costs include:

- harder code review
- dependency mistakes
- stale closures
- less obvious data flow
- memoization overhead
- little actual improvement

Optimize from evidence.

---

## 31. Never Break Correctness for Performance

Wrong:

```jsx
useMemo(
  () => calculate(a, b),
  [a] // ❌ b intentionally omitted
);
```

If `b` is reactive and used by the calculation, omitting it can produce stale results.

If correct recalculation is too expensive, redesign the computation.

Do not lie to React.

---

## 32. Accessibility Still Matters

Do not make UI "faster" by breaking:

- keyboard interaction
- focus behavior
- semantic controls
- screen-reader feedback

Performance is one quality dimension, not permission to remove accessibility.

---

## 33. Perceived Performance

Sometimes the best improvement is not reducing total milliseconds but improving continuity.

Examples:

- preserve existing content
- show skeletons
- provide pending feedback
- avoid layout shifts
- prioritize typing
- progressively reveal optional content

Transitions and Suspense can help create this experience.

---

## 34. CareerLoop Walkthrough ⭐⭐⭐⭐⭐

Scenario:

> Searching 5,000 job applications feels slow.

### Step 1 — Reproduce

Use realistic data.

### Step 2 — Profile

Determine whether time is spent in:

- filtering
- sorting
- rendering cards
- DOM/layout

### Step 3 — State placement

Keep search state close to the search/list feature.

### Step 4 — Derived data

Do not store the filtered list using Effect + state.

### Step 5 — Calculation

If filtering/sorting is expensive:

```text
improve algorithm
then maybe useMemo
```

### Step 6 — DOM volume

If thousands of cards are mounted:

```text
pagination / virtualization
```

This may outperform any memoization trick.

### Step 7 — Responsiveness

If typing still competes with list rendering:

```text
useDeferredValue
or
useTransition
```

### Step 8 — Card rendering

If individual cards are expensive and frequently receive unchanged props:

```text
memo
```

### Step 9 — Measure again

Verify the result.

---

## 35. CodeBuddy Walkthrough

Scenario:

> Discovery feed stutters during filtering.

Decision flow:

```text
ranking calculation expensive?
→ algorithm / maybe useMemo

too many cards mounted?
→ pagination / virtualization

unchanged expensive cards render?
→ maybe memo

callback identity breaks memo?
→ maybe useCallback

typing waits for result rendering?
→ Transition / deferred value

initial JavaScript too large?
→ lazy / code splitting

API slow?
→ data/network optimization
```

Different bottlenecks need different solutions.

---

## 36. Performance Decision Matrix ⭐⭐⭐⭐⭐

| Problem | First direction |
|---|---|
| State update renders huge unrelated tree | Colocate/restructure state |
| Effect repeatedly sets derived state | Remove the Effect |
| Expensive pure calculation | Algorithm / maybe `useMemo` |
| Expensive child with same props | Maybe `memo` |
| Function prop breaks useful memo | Maybe `useCallback` |
| Typing blocked by expensive UI | Transition/deferred value |
| Thousands of DOM rows | Pagination/virtualization |
| Huge initial JavaScript | Code splitting/`lazy` |
| Slow API | Network/data strategy |
| Layout/paint is slow | Browser/CSS/DOM optimization |
| No measured problem | Keep code simple |

---

## 37. Common Mistakes ⭐⭐⭐⭐⭐

1. Optimizing before measuring.
2. Treating every re-render as a bug.
3. Using console logs as the only profiler.
4. Adding `memo` everywhere.
5. Adding `useMemo` to cheap calculations.
6. Adding `useCallback` to every handler.
7. Keeping rapidly changing state too high.
8. Using Effects for derived state.
9. Rendering thousands of rows unnecessarily.
10. Using render optimization for network problems.
11. Ignoring layout/paint costs.
12. Removing dependencies for "performance".
13. Testing only development behavior.
14. Assuming React Compiler fixes every performance layer.
15. Failing to measure again.

---

## 38. Interview Questions ⭐⭐⭐⭐⭐

### How do you optimize a React application?

Reproduce and profile the slow interaction, identify the actual bottleneck, apply a targeted fix at the correct layer, then measure again.

### Is re-rendering always bad?

No. Rendering is normal React behavior and can be very cheap.

### How can you reduce unnecessary rendering?

Start with state colocation and removing unnecessary state/Effects, then use targeted memoization where profiling justifies it.

### How do you optimize huge lists?

Reduce mounted DOM with pagination or virtualization.

### How do you keep input responsive during expensive rendering?

Consider `useTransition` or `useDeferredValue` after confirming rendering is the bottleneck.

### How do you reduce initial JavaScript?

Use meaningful code-splitting/lazy-loading boundaries and remove unnecessary heavy dependencies.

### React Profiler vs browser performance tools?

React Profiler focuses on React rendering; browser tools expose broader scripting, layout, paint, network, and main-thread behavior.

### What does React Compiler change?

It can automate much memoization, reducing manual `memo`, `useMemo`, and `useCallback` needs, but it does not solve every performance problem.

---

## 39. Interview-Ready Answer ⭐⭐⭐⭐⭐

```text
I optimize React from measurements rather than
adding memoization by default.

First I reproduce and profile the slow interaction.
I check whether state is too high, whether Effects
are creating extra updates, whether derived state is
duplicated, and whether an algorithm or large DOM is
the real problem.

For an expensive pure calculation I may use useMemo.
For an expensive child that often receives unchanged
props I may use memo, and I use useCallback only when
stable function identity enables a useful optimization.

For large lists I prefer pagination or virtualization.
For responsiveness I consider transitions or deferred
values. For startup problems I inspect bundle size and
code splitting.

I also separate React rendering cost from network,
layout, paint, and other browser work. Finally I
measure again to verify the improvement. In modern
compiler-enabled React, I also account for automatic
memoization instead of manually memoizing everything.
```

---

## 40. Section 7 Complete Mental Model ⭐⭐⭐⭐⭐

```text
                 REACT PERFORMANCE
                        │
       ┌────────────────┼─────────────────┐
       ↓                ↓                 ↓
   reduce work      schedule work      load less code
       │                │                 │
       ↓                ↓                 ↓
state colocation   useTransition          lazy
remove Effects     deferredValue      code splitting
algorithms
virtualization
       │
       ↓
targeted caching
 ├── memo
 ├── useMemo
 └── useCallback

                 EVERY PATH
                     ↓
                  PROFILE
                     ↓
                  MEASURE
                     ↓
                   VERIFY
```

---

## 41. Section 7 Revision

### Lesson 37 — React.memo

```text
skip expensive component rendering
when props are unchanged
```

### Lesson 38 — useMemo

```text
cache a calculated value
```

### Lesson 39 — useCallback

```text
preserve function reference
```

### Lesson 40 — Memoization Comparison

```text
choose memoization based on
the actual bottleneck
```

### Lesson 41 — useTransition

```text
mark non-urgent updates
as non-blocking
```

### Lesson 42 — useDeferredValue

```text
let expensive UI lag behind
a rapidly changing value
```

### Lesson 43 — lazy / Code Splitting

```text
load optional component code
when needed
```

### Lesson 44 — Profiling

```text
measure
→ optimize the real bottleneck
→ measure again
```

---

## 42. Key Takeaways ⭐⭐⭐⭐⭐

- Performance starts with measurement, not memoization.
- Re-renders are normal and not automatically expensive.
- Keep state as local as practical.
- Avoid Effects for purely derived state.
- Reduce work before caching work.
- Improve algorithms and reduce DOM size where possible.
- Use `memo`, `useMemo`, and `useCallback` at meaningful boundaries.
- Use Transitions/deferred values for responsiveness, not correctness.
- Use pagination/virtualization for very large lists.
- Use lazy loading/code splitting for bundle-delivery problems.
- Separate React, network, and browser performance.
- Validate important conclusions with realistic production-like behavior.
- React Compiler can reduce manual memoization needs but cannot fix every bottleneck.
- Always measure after optimization.

---

# Section 7 — Performance Completed ✅

Completed lessons:

- Lesson 37 — React.memo ⭐⭐⭐⭐⭐
- Lesson 38 — useMemo ⭐⭐⭐⭐⭐
- Lesson 39 — useCallback ⭐⭐⭐⭐⭐
- Lesson 40 — React.memo vs useMemo vs useCallback ⭐⭐⭐⭐⭐
- Lesson 41 — useTransition and Concurrent Rendering ⭐⭐⭐⭐⭐
- Lesson 42 — useDeferredValue
- Lesson 43 — Lazy Loading, React.lazy and Code Splitting
- Lesson 44 — React Performance Profiling and Optimization

---

## Next Section — React Internals

➡️ [Lesson 45 — Virtual DOM ⭐⭐⭐⭐⭐](../08-react-internals/45-virtual-dom.md)
