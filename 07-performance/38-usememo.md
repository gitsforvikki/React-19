# Lesson 38 — useMemo ⭐⭐⭐⭐⭐

## 1. What Is useMemo?

`useMemo` is a React Hook that caches the **result of a calculation** between renders.

```jsx
const cachedValue =
  useMemo(
    calculateValue,
    dependencies
  );
```

Mental model:

```text
render
  ↓
dependencies changed?
 ┌──────┴──────┐
no            yes
↓              ↓
reuse        calculate
cached       new value
value          ↓
               cache it
```

---

## 2. The Main Purpose ⭐⭐⭐⭐⭐

Suppose:

```jsx
const visibleApplications =
  filterApplications(
    applications,
    query,
    status
  );
```

This calculation runs every time the component renders.

Usually that is fine.

If the calculation is meaningfully expensive and its inputs often remain unchanged:

```jsx
const visibleApplications =
  useMemo(
    () =>
      filterApplications(
        applications,
        query,
        status
      ),
    [
      applications,
      query,
      status,
    ]
  );
```

React can reuse the previous result while those dependencies remain unchanged.

---

## 3. useMemo Caches a Result ⭐⭐⭐⭐⭐

This distinction is critical.

```jsx
const value =
  useMemo(
    () =>
      expensiveCalculation(
        data
      ),
    [data]
  );
```

React caches:

```text
result of expensiveCalculation(data)
```

not the calculation function itself.

Compare with Lesson 39:

```text
useMemo
→ caches result/value

useCallback
→ caches function definition/reference
```

---

## 4. Basic Example

```jsx
import {
  useMemo,
  useState,
} from "react";

function ApplicationList({
  applications,
}) {
  const [query, setQuery] =
    useState("");

  const [theme, setTheme] =
    useState("light");

  const visibleApplications =
    useMemo(() => {
      return applications.filter(
        (application) =>
          application.company
            .toLowerCase()
            .includes(
              query.toLowerCase()
            )
      );
    }, [
      applications,
      query,
    ]);

  // ...
}
```

Changing `theme` alone does not require recalculating the filtered list if the memo dependencies are unchanged.

---

## 5. Dependencies ⭐⭐⭐⭐⭐

Every reactive value used inside the calculation belongs in the dependency list.

```jsx
const result =
  useMemo(() => {
    return calculate(
      data,
      filter,
      sortOrder
    );
  }, [
    data,
    filter,
    sortOrder,
  ]);
```

Reactive values include:

- props
- state
- variables/functions declared in the component that are used by the calculation

Keep React Hook linting enabled.

---

## 6. How Dependencies Are Compared

React compares each dependency to its previous value using `Object.is`.

Primitive:

```js
Object.is(
  "active",
  "active"
) // true
```

New object:

```js
Object.is(
  {},
  {}
) // false
```

So reference identity matters.

---

## 7. useMemo Is Not for Every Calculation ⭐⭐⭐⭐⭐

This does not need memoization:

```jsx
const fullName =
  firstName +
  " " +
  lastName;
```

Wrapping cheap calculations:

```jsx
const fullName =
  useMemo(
    () =>
      firstName +
      " " +
      lastName,
    [
      firstName,
      lastName,
    ]
  );
```

usually adds more conceptual overhead than value.

Use `useMemo` when there is a measured or structurally meaningful reason.

---

## 8. useMemo Is Only a Performance Optimization ⭐⭐⭐⭐⭐

Your code must remain correct if React recalculates the value.

Do not use `useMemo` as semantic storage.

Wrong mental model:

```text
I need this value to exist forever
→ useMemo
```

Use state or a ref when semantics require persistent mutable/stateful storage.

`useMemo` is a cache React may discard for supported reasons.

---

## 9. Expensive Calculation Use Case

```jsx
const statistics =
  useMemo(() => {
    return calculateStatistics(
      applications
    );
  }, [applications]);
```

Good candidate if:

- `calculateStatistics` is actually expensive,
- component renders often,
- `applications` often stays the same.

If `applications` changes every render, caching may provide little benefit.

---

## 10. Stabilizing a Prop for memo ⭐⭐⭐⭐⭐

Memoized child:

```jsx
const ApplicationList =
  memo(function ApplicationList({
    applications,
  }) {
    // ...
  });
```

Without `useMemo`:

```jsx
const visible =
  applications.filter(
    matchesFilter
  );

return (
  <ApplicationList
    applications={visible}
  />
);
```

`visible` is a new array every render.

With `useMemo`:

```jsx
const visible =
  useMemo(
    () =>
      applications.filter(
        matchesFilter
      ),
    [
      applications,
      matchesFilter,
    ]
  );
```

Now the child can receive the same array reference when dependencies are unchanged.

---

## 11. Important: Memoizing a Cheap Value May Still Matter for Identity

Sometimes the calculation itself is cheap, but its **reference identity** matters because:

- it is passed to a memoized child,
- another Hook depends on it.

Example:

```jsx
const options =
  useMemo(
    () => ({
      roomId,
      serverUrl,
    }),
    [
      roomId,
      serverUrl,
    ]
  );
```

But before doing this, ask whether the object can simply be created inside the Effect/consumer that needs it.

Often architecture can remove the need for memoization.

---

## 12. Object Dependency Trap ⭐⭐⭐⭐⭐

Problem:

```jsx
const searchOptions = {
  text,
  matchMode: "whole-word",
};

const visibleItems =
  useMemo(
    () =>
      searchItems(
        items,
        searchOptions
      ),
    [
      items,
      searchOptions,
    ]
  );
```

`searchOptions` is new every render.

Therefore:

```text
dependency always changes
→ useMemo recalculates every render
```

The memoization is defeated.

---

## 13. Better: Move Object Creation Inside the Calculation

```jsx
const visibleItems =
  useMemo(() => {
    const searchOptions = {
      text,
      matchMode:
        "whole-word",
    };

    return searchItems(
      items,
      searchOptions
    );
  }, [
    items,
    text,
  ]);
```

Now dependencies are simpler.

This is often better than memoizing one value only so another memoization can depend on it.

---

## 14. Or Memoize the Dependency When It Has Independent Meaning

```jsx
const searchOptions =
  useMemo(
    () => ({
      text,
      matchMode:
        "whole-word",
    }),
    [text]
  );
```

Then:

```jsx
const visibleItems =
  useMemo(
    () =>
      searchItems(
        items,
        searchOptions
      ),
    [
      items,
      searchOptions,
    ]
  );
```

Valid, but use only when the separate memoized value makes the design clearer or identity itself matters.

---

## 15. useMemo Must Return a Value

Correct:

```jsx
const processed =
  useMemo(() => {
    return data.map(
      transform
    );
  }, [data]);
```

Wrong:

```jsx
useMemo(() => {
  data.forEach(
    sendAnalytics
  );
}, [data]); // ❌ side effect
```

`useMemo` is not an Effect.

Do not use it for:

- API requests
- analytics
- DOM changes
- subscriptions
- localStorage synchronization

---

## 16. The Calculation Must Be Pure ⭐⭐⭐⭐⭐

Bad:

```jsx
const visible =
  useMemo(() => {
    applications.push(
      newApplication
    ); // ❌ mutation

    return applications;
  }, [applications]);
```

A memo calculation runs during rendering and must be pure.

Correct:

```jsx
const visible =
  useMemo(() => {
    return applications.filter(
      matchesFilter
    );
  }, [applications]);
```

---

## 17. Strict Mode and useMemo

In development Strict Mode, React may call the calculation function extra times to help reveal accidental impurities.

If the calculation is pure:

```text
extra development call
→ no correctness problem
```

If duplicate calculation mutates data:

```text
bug becomes visible
```

Do not write code that depends on the memo calculation running exactly once.

---

## 18. useMemo vs State ⭐⭐⭐⭐⭐

Suppose value is derived:

```jsx
const filtered =
  applications.filter(
    matchesFilter
  );
```

Do not store it in state just because calculation is expensive:

```jsx
const [
  filtered,
  setFiltered,
] = useState([]); // usually unnecessary
```

Instead:

```jsx
const filtered =
  useMemo(
    () =>
      applications.filter(
        matchesFilter
      ),
    [
      applications,
      matchesFilter,
    ]
  );
```

Conceptually:

```text
derived data
→ calculate during render

expensive derived data
→ maybe memoize calculation
```

---

## 19. useMemo vs useEffect

Wrong pattern:

```jsx
const [
  filtered,
  setFiltered,
] = useState([]);

useEffect(() => {
  setFiltered(
    applications.filter(
      matchesFilter
    )
  );
}, [
  applications,
  matchesFilter,
]);
```

This creates:

```text
render
→ Effect
→ setState
→ extra render
```

If the value is purely derived, calculate it during render; use `useMemo` only if calculation cost justifies caching.

---

## 20. useMemo and JSX

JSX is an object value.

Technically:

```jsx
const children =
  useMemo(
    () => (
      <List
        items={visibleItems}
      />
    ),
    [visibleItems]
  );
```

can memoize JSX identity.

But in normal code, `memo(List)` is usually clearer when component-level memoization is actually needed.

Understand the concept; prefer readable architecture.

---

## 21. useMemo Can Cache Any Value

Examples:

### Number

```jsx
const total =
  useMemo(
    () =>
      calculateTotal(items),
    [items]
  );
```

### Array

```jsx
const sorted =
  useMemo(
    () =>
      [...items].sort(
        compareItems
      ),
    [items]
  );
```

### Object

```jsx
const config =
  useMemo(
    () => ({
      theme,
      density,
    }),
    [
      theme,
      density,
    ]
  );
```

The important question is not type—it is whether caching the calculation/result is useful.

---

## 22. Sorting Example — Keep It Immutable

Wrong:

```jsx
const sorted =
  useMemo(
    () =>
      applications.sort(
        compare
      ),
    [applications]
  );
```

`sort()` mutates the original array.

Correct:

```jsx
const sorted =
  useMemo(
    () =>
      [...applications].sort(
        compare
      ),
    [applications]
  );
```

or use a non-mutating sorting method where appropriate.

Memoization does not remove immutability requirements.

---

## 23. Measuring Expensive Calculations

You can estimate calculation cost during development:

```js
console.time(
  "filter applications"
);

const result =
  filterApplications(
    applications
  );

console.timeEnd(
  "filter applications"
);
```

Better still, profile realistic interactions and production-like builds.

Development Strict Mode and tooling can affect timings.

---

## 24. Don't Depend on useMemo for Correctness ⭐⭐⭐⭐⭐

If removing `useMemo` breaks behavior:

```text
something is architecturally wrong
```

Correct code:

```text
without useMemo
→ still correct

with useMemo
→ potentially faster
```

This is a favorite interview principle.

---

## 25. useMemo Cache Is Local to a Component

If two component instances run:

```jsx
<Report data={data} />
<Report data={data} />
```

and each has its own `useMemo`, they do not automatically share one cache.

```text
Report instance A
└── own memo cache

Report instance B
└── own memo cache
```

`useMemo` caches across renders of that component instance.

---

## 26. React Compiler Perspective ⭐⭐⭐⭐⭐

React Compiler can automatically memoize values and calculations in compiled code, reducing the need for manual `useMemo`.

Modern perspective:

```text
write pure clear code
      ↓
compiler can optimize automatically
      ↓
manual useMemo when:
- precise identity control is needed
- existing code requires it
- compiler is not used
- profiling justifies it
```

Still learn `useMemo` deeply because:

- existing applications use it,
- it explains reference identity,
- it remains available for precise control,
- interviews test it.

---

## 27. CareerLoop Example

Suppose 5,000 applications need filtering and sorting:

```jsx
const visibleApplications =
  useMemo(() => {
    return applications
      .filter(
        (application) =>
          matchesFilters(
            application,
            filters
          )
      )
      .toSorted(
        compareApplications
      );
  }, [
    applications,
    filters,
  ]);
```

If the page renders because an unrelated UI state changes, the expensive list transformation can be reused while dependencies remain stable.

But first verify it is actually expensive.

---

## 28. CodeBuddy Example

Discovery ranking:

```jsx
const rankedDevelopers =
  useMemo(
    () =>
      rankDevelopers(
        developers,
        preferences
      ),
    [
      developers,
      preferences,
    ]
  );
```

Good candidate if ranking is expensive and inputs often remain stable.

If `preferences` is reconstructed every render, fix/stabilize that dependency architecture or the memo will recalculate.

---

## 29. Common Mistakes ⭐⭐⭐⭐⭐

1. Wrapping every calculation in `useMemo`.
2. Using `useMemo` for side effects.
3. Forgetting to return a value.
4. Omitting reactive dependencies.
5. Passing an always-new object as a dependency.
6. Mutating arrays/objects inside the calculation.
7. Assuming cached value can never be discarded.
8. Using `useMemo` for correctness.
9. Memoizing extremely cheap calculations without a reason.
10. Using Effect + state for purely derived data.
11. Forgetting each component instance has its own cache.
12. Ignoring React Compiler's automatic memoization capabilities.

---

## 30. Interview Questions ⭐⭐⭐⭐⭐

### What is useMemo?

A Hook that caches the result of a calculation between renders while its dependencies remain unchanged.

### What does useMemo cache?

The calculated return value.

### How are dependencies compared?

Using `Object.is`.

### When should useMemo be used?

For expensive calculations or when stable result identity is useful for another optimization/dependency and profiling/design justifies it.

### Is useMemo required for correctness?

No.

### useMemo vs useEffect?

`useMemo` calculates/caches a value during rendering; `useEffect` synchronizes with external systems after rendering.

### useMemo vs state?

State represents data React must remember and whose updates drive rendering. `useMemo` caches a derived calculation as a performance optimization.

### Can useMemo contain side effects?

No. Its calculation should be pure.

### Why can an object dependency defeat useMemo?

If recreated each render, its reference changes each render.

### Does useMemo share its cache between component instances?

No.

### What happens when a dependency changes?

React recalculates the value and caches the new result.

### How does React Compiler affect useMemo?

The compiler can automatically memoize values/calculations, reducing the need for manual `useMemo` in compiled code.

---

## 31. Complete Mental Model

```text
              component render
                    │
                    ↓
                useMemo
                    │
         compare dependencies
             with Object.is
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
      unchanged             changed
          │                   │
          ↓                   ↓
     cached value        run pure
       returned          calculation
                              │
                              ↓
                         cache result
                              │
                              ↓
                         return value
```

---

## 32. Quick Revision

```jsx
const value =
  useMemo(
    () =>
      expensiveCalculation(
        input
      ),
    [input]
  );
```

Remember:

```text
useMemo
→ memoizes a VALUE/result

not:
→ side effects
→ semantic storage
→ automatic correctness
```

---

## 33. Key Takeaways

- `useMemo` caches calculation results between renders.
- Dependencies are compared with `Object.is`.
- The calculation must be pure.
- Include all reactive dependencies.
- It is mainly useful for expensive calculations or meaningful stable identity.
- Cheap calculations usually do not need it.
- New object/array dependencies can defeat memoization.
- Move temporary objects inside the calculation when that simplifies dependencies.
- Do not use `useMemo` for side effects.
- Derived data usually should not be copied into state.
- `useMemo` is local to a component instance.
- Code must remain correct without it.
- Memoization does not replace immutable updates.
- React Compiler can automatically memoize values in compiled code.
- Manual `useMemo` remains useful for precise control and existing code.

---

## Next Lesson

➡️ [Lesson 39 — useCallback ⭐⭐⭐⭐⭐](./39-usecallback.md)
