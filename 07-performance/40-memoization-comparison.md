# Lesson 40 — React.memo vs useMemo vs useCallback ⭐⭐⭐⭐⭐

## 1. Why These Three Are Confusing

All three are related to **memoization**, but they cache different things.

```text
memo
→ component rendering

useMemo
→ calculated value

useCallback
→ function reference
```

This lesson connects Lessons 37–39 into one decision model.

---

## 2. The Core Comparison ⭐⭐⭐⭐⭐

| API | What it memoizes | Typical purpose |
|---|---|---|
| `memo` | Component render based on props | Skip expensive child renders |
| `useMemo` | Result/value of a calculation | Skip expensive recalculation or preserve value identity |
| `useCallback` | Function reference | Preserve callback identity |

Memorize this table for interviews.

---

## 3. React.memo

```jsx
const Child = memo(
  function Child({
    value,
  }) {
    return (
      <div>{value}</div>
    );
  }
);
```

Question it answers:

> Can React usually skip running this child component when its props have not changed?

---

## 4. useMemo

```jsx
const visibleItems =
  useMemo(
    () =>
      filterItems(
        items,
        filter
      ),
    [
      items,
      filter,
    ]
  );
```

Question it answers:

> Can React reuse this calculated value while its dependencies remain unchanged?

---

## 5. useCallback

```jsx
const handleSave =
  useCallback(() => {
    save(id);
  }, [id]);
```

Question it answers:

> Can React return the same function reference while its dependencies remain unchanged?

---

## 6. One Diagram for All Three ⭐⭐⭐⭐⭐

```text
                   Parent render
                        │
        ┌───────────────┼────────────────┐
        ↓               ↓                ↓
      memo           useMemo        useCallback
        │               │                │
        ↓               ↓                ↓
 component render   calculated       function
   optimization       result         reference
        │               │                │
        └───────────────┼────────────────┘
                        ↓
                  performance tools
                  not correctness tools
```

---

## 7. Why They Often Appear Together ⭐⭐⭐⭐⭐

Suppose:

```jsx
const List = memo(
  function List({
    items,
    onSelect,
  }) {
    // expensive render
  }
);
```

Parent:

```jsx
const visibleItems =
  items.filter(
    matchesFilter
  );

const handleSelect =
  (id) => {
    setSelectedId(id);
  };

return (
  <List
    items={visibleItems}
    onSelect={handleSelect}
  />
);
```

Problem:

```text
visibleItems
→ new array every render

handleSelect
→ new function every render

memo(List)
→ sees changed props
→ cannot skip
```

---

## 8. Combined Optimization

```jsx
const visibleItems =
  useMemo(
    () =>
      items.filter(
        matchesFilter
      ),
    [
      items,
      matchesFilter,
    ]
  );

const handleSelect =
  useCallback(
    (id) => {
      setSelectedId(id);
    },
    []
  );

return (
  <List
    items={visibleItems}
    onSelect={handleSelect}
  />
);
```

Now:

```text
useMemo
→ stable items reference

useCallback
→ stable callback reference

memo
→ can compare stable props
→ can skip expensive child render
```

But only do this when the skipped work is actually worth optimizing.

---

## 9. They Are Not Automatically a Package Deal ⭐⭐⭐⭐⭐

Do not think:

```text
if memo
then always useMemo + useCallback
```

Examples:

```jsx
<MemoChild
  name={name}
  age={age}
/>
```

Primitive props may already be stable.

No `useMemo` or `useCallback` is needed.

Each tool should solve a specific identity/performance problem.

---

## 10. useMemo vs useCallback Internally

Conceptually:

```jsx
useCallback(
  fn,
  deps
);
```

is similar to:

```jsx
useMemo(
  () => fn,
  deps
);
```

The difference is API intent:

```text
value result
→ useMemo

function itself
→ useCallback
```

Use the API that communicates your intention.

---

## 11. memo vs useMemo ⭐⭐⭐⭐⭐

`memo` wraps a component:

```jsx
const Card =
  memo(CardComponent);
```

`useMemo` runs inside a component/custom Hook:

```jsx
const data =
  useMemo(
    calculate,
    deps
  );
```

Interview answer:

```text
memo
→ skips component render based on props

useMemo
→ skips recalculating a value based on dependencies
```

---

## 12. memo vs useCallback

`memo`:

```text
optimizes child component rendering
```

`useCallback`:

```text
stabilizes function reference
```

They often work together when a memoized child receives a callback prop.

But `useCallback` does not memoize the child.

---

## 13. Dependency Comparison vs Prop Comparison

`useMemo` and `useCallback` compare dependencies with `Object.is`.

`memo` by default compares each prop with `Object.is`.

So all three are strongly affected by reference identity.

```text
primitive
→ often naturally stable

object/array/function
→ identity can change every render
```

---

## 14. Reference Equality Example ⭐⭐⭐⭐⭐

```js
Object.is(
  10,
  10
); // true

Object.is(
  "React",
  "React"
); // true

Object.is(
  {},
  {}
); // false

Object.is(
  [],
  []
); // false

Object.is(
  () => {},
  () => {}
); // false
```

This explains a huge amount of React memoization behavior.

---

## 15. Memoization Does Not Mean "Never Recalculate/Render"

All three should be understood as optimizations.

```text
memo
→ React may still render

useMemo
→ cache may be discarded

useCallback
→ cache may be discarded
```

Never make correctness depend on memoization.

---

## 16. First Optimize State Placement ⭐⭐⭐⭐⭐

Suppose:

```text
App
 └── mouse position state
      ↓
huge application tree renders
```

You might be tempted to add:

```text
memo everywhere
```

Better:

```text
MouseTracker
 └── mouse position state
```

Keep rapidly changing local state close to its consumers.

State colocation can eliminate the need for broad memoization.

---

## 17. Avoid Unnecessary Effect Chains

Performance problem:

```text
render
→ Effect
→ setState
→ render
→ Effect
→ setState
→ render
```

Adding `memo` to children may treat symptoms.

First ask whether the Effect and derived state are necessary.

Lessons 19–23 established this principle.

---

## 18. Keep Rendering Pure

If a re-render causes visible bugs:

```text
"we need memo so component doesn't run"
```

is the wrong fix.

A pure component should be safe to render again.

Memoization optimizes valid rendering; it should not prevent broken render logic from executing.

---

## 19. Decision Tree ⭐⭐⭐⭐⭐

```text
Performance issue observed?
        │
       no
        ↓
Do not optimize blindly

       yes
        │
        ↓
Profile the interaction
        │
        ↓
What is expensive?
 ┌──────┼───────────┐
 ↓      ↓           ↓
child  calculation function
render              identity
 ↓      ↓           ↓
memo   useMemo   useCallback
```

Then ask:

```text
Can architecture/state placement
remove the problem more simply?
```

---

## 20. When to Use memo

Consider it when:

```text
component renders frequently
+
same props often
+
render is expensive
```

Avoid automatic use when rendering is cheap or props always change.

---

## 21. When to Use useMemo

Consider it when:

```text
expensive calculation
+
dependencies often unchanged
```

or:

```text
stable object/array/value identity
is necessary for another
measured optimization/dependency
```

Do not use it for side effects or semantic storage.

---

## 22. When to Use useCallback

Consider it when:

```text
function passed to memoized child
+
stable function prop enables skip
```

or:

```text
function is Hook dependency
+
stable identity is meaningful
```

First consider whether the function can be moved inside the Effect or outside the component.

---

## 23. When to Use None ⭐⭐⭐⭐⭐

Very often:

```jsx
function Counter() {
  const [
    count,
    setCount,
  ] = useState(0);

  return (
    <button
      onClick={() =>
        setCount(
          (c) => c + 1
        )
      }
    >
      {count}
    </button>
  );
}
```

needs:

```text
no memo
no useMemo
no useCallback
```

Simple code is often the best optimized code until evidence says otherwise.

---

## 24. Performance Cost Model

Memoization itself has work:

```text
store cached value/reference
+
compare dependencies/props
+
maintain more complex code
```

Benefit:

```text
skip expensive work
```

Use memoization when:

```text
cost skipped
>
cost of memoization + complexity
```

This is the real performance equation.

---

## 25. "Everything in useMemo/useCallback" Is Not Free

Over-memoization can cause:

- harder-to-read code
- dependency bugs
- stale closure mistakes
- ineffective caches
- unnecessary comparisons
- harder debugging

A performance API used everywhere becomes noise.

---

## 26. Profiling Before Optimization ⭐⭐⭐⭐⭐

Use React DevTools Profiler to identify:

- which components render
- how often
- which renders are expensive
- what interaction causes the work

Lesson 44 covers profiling deeply.

For now:

```text
feel slow
→ measure
→ locate bottleneck
→ optimize
→ measure again
```

---

## 27. Development vs Production Performance

Development can include extra checks such as Strict Mode behavior.

Do not judge final performance only from development logs.

For serious performance analysis:

- use realistic data,
- test representative interactions,
- consider production-like builds,
- use profiling tools.

---

## 28. React Compiler Changes the Default Strategy ⭐⭐⭐⭐⭐

Modern React Compiler can automatically memoize:

- components
- values
- functions

This reduces manual `memo`, `useMemo`, and `useCallback` needs in compiled applications.

Modern strategy:

```text
1. write pure idiomatic React
2. colocate state
3. avoid unnecessary Effects
4. enable/use compiler where appropriate
5. profile remaining bottlenecks
6. use manual memoization for precise control
   or non-compiled/existing code when justified
```

Do not delete existing manual memoization blindly when adopting the compiler; test behavior/performance carefully.

---

## 29. Why Learn These APIs If Compiler Can Optimize?

Because they teach fundamental concepts:

- render propagation
- reference identity
- closures
- dependency arrays
- component purity
- performance tradeoffs

Also:

- many production codebases are not compiler-enabled,
- existing code uses manual memoization,
- libraries may expose identity-sensitive APIs,
- interviews still ask them,
- manual control remains available.

---

## 30. Example — CareerLoop ⭐⭐⭐⭐⭐

Suppose:

```text
ApplicationsPage
 ├── query input
 └── ApplicationList
      └── 5,000 cards
```

Potential flow:

### Step 1

Profile.

### Step 2

Check state location.

Can query state live closer to the list instead of causing unrelated page regions to render?

### Step 3

Check expensive calculation.

```jsx
const visible =
  useMemo(
    () =>
      filterAndSort(
        applications,
        filters
      ),
    [
      applications,
      filters,
    ]
  );
```

### Step 4

If card renders are expensive and props are stable:

```jsx
const ApplicationCard =
  memo(...);
```

### Step 5

If callback prop defeats card memoization:

```jsx
const handleDelete =
  useCallback(...);
```

This is evidence-driven optimization, not ritual.

---

## 31. Example — CodeBuddy Discovery Feed

```text
DiscoveryFeed
 ├── preferences
 ├── ranked developers
 └── DeveloperCard × N
```

Possible measured optimization:

```text
rankDevelopers expensive
→ useMemo

DeveloperCard expensive
and often same props
→ memo

onConnect recreated
and breaks memo
→ useCallback
```

But if pagination only renders a few cheap cards, manual memoization may provide little benefit.

---

## 32. Broken Memoization Example

```jsx
const Card =
  memo(CardComponent);

const handleClick =
  useCallback(
    () => doSomething(),
    []
  );

return (
  <Card
    onClick={handleClick}
    style={{
      padding: 12,
    }}
  />
);
```

`style` is a new object every render.

So:

```text
useCallback works
but
style changes identity
→ memo can still be defeated
```

One unstable prop is enough.

---

## 33. Don't Memoize to Silence exhaustive-deps

Bad process:

```text
Effect warning
→ wrap everything in useCallback/useMemo
→ suppress warning
```

Better:

1. understand why the Effect needs the dependency,
2. remove unnecessary Effect logic,
3. move values/functions inside the Effect where possible,
4. memoize only if stable identity is truly required.

---

## 34. Interview Comparison ⭐⭐⭐⭐⭐

### React.memo

```text
Input:
component

Comparison:
props

Output:
memoized component

Goal:
skip component rendering
```

### useMemo

```text
Input:
calculation + dependencies

Comparison:
dependencies

Output:
cached calculated value

Goal:
skip recalculation /
stabilize value identity
```

### useCallback

```text
Input:
function + dependencies

Comparison:
dependencies

Output:
cached function reference

Goal:
stabilize function identity
```

---

## 35. Interview Questions ⭐⭐⭐⭐⭐

### What is the difference between memo, useMemo and useCallback?

`memo` memoizes component rendering based on props, `useMemo` caches a calculated value, and `useCallback` caches a function reference.

### Does useCallback improve performance by itself?

Not necessarily. It is useful when stable function identity enables another optimization or is required by a dependency relationship.

### Does useMemo stop a component from rendering?

No. It only caches a calculation result within the component.

### Does memo stop state updates?

No. Own state updates still render the component.

### Why can one object prop break memo?

A newly created object has a new reference, so the prop is considered changed.

### Should you memoize everything?

No. Memoization has overhead and increases code complexity.

### What should you do before memoizing?

Keep state local, keep rendering pure, remove unnecessary Effects, profile the actual bottleneck.

### Are these APIs required for correctness?

No.

### How does React compare dependencies?

With `Object.is`.

### How does React Compiler affect these APIs?

It can automatically memoize components, values, and functions, reducing the need for manual memoization while manual APIs remain available for precise control.

---

## 36. Five-Star Interview Answer ⭐⭐⭐⭐⭐

If asked:

> When do you use memo, useMemo, and useCallback?

A strong concise answer:

```text
I don't use them by default.

I first keep state close to where it is used,
keep render logic pure, and profile the slow
interaction.

I use React.memo when an expensive child
re-renders frequently with unchanged props.

I use useMemo to cache an expensive calculated
value or preserve a value's identity when that
identity enables another optimization.

I use useCallback when stable function identity
matters, commonly for a callback passed to a
memoized child or used as a Hook dependency.

All three are performance optimizations, not
correctness mechanisms. In React Compiler-enabled
code, much of this memoization may be automatic.
```

---

## 37. Complete Mental Model ⭐⭐⭐⭐⭐

```text
                PERFORMANCE ISSUE
                       │
                       ↓
                    PROFILE
                       │
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
 expensive child   expensive value   unstable
    rendering       calculation      callback
       │               │                │
       ↓               ↓                ↓
      memo           useMemo        useCallback
       │               │                │
       └───────────────┼────────────────┘
                       ↓
               measure again

Before all three:
- colocate state
- keep render pure
- remove unnecessary Effects

With React Compiler:
automatic memoization may make
manual calls unnecessary.
```

---

## 38. Quick Revision Table ⭐⭐⭐⭐⭐

| Question | Use |
|---|---|
| Expensive child renders with same props? | `memo` |
| Expensive calculation repeats unnecessarily? | `useMemo` |
| Function identity breaks useful memoization? | `useCallback` |
| Cheap component/calculation with no measured issue? | None |
| Correctness depends on memoization? | Fix architecture |
| React Compiler handles it sufficiently? | Prefer simpler code |

---

## 39. Key Takeaways

- `memo`, `useMemo`, and `useCallback` memoize different things.
- `memo` targets component rendering.
- `useMemo` targets calculated values.
- `useCallback` targets function identity.
- All three rely heavily on identity comparisons.
- One always-new prop can defeat component memoization.
- These APIs often work together but are not automatically used together.
- Memoization has runtime and maintenance costs.
- Re-renders are normal and not automatically performance bugs.
- State colocation often removes the need for memoization.
- Unnecessary Effect chains can be a bigger performance problem.
- Keep rendering pure before optimizing.
- Profile before and after optimization.
- Code must remain correct without memoization.
- React Compiler can automatically memoize components, values, and functions.
- Manual memoization remains important for existing code, non-compiled code, precise control, and interviews.

---

## Next Lesson

➡️ [Lesson 41 — useTransition and Concurrent Rendering ⭐⭐⭐⭐⭐](./41-usetransition.md)
