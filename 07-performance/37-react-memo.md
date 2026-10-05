# Lesson 37 — React.memo ⭐⭐⭐⭐⭐

## 1. What Problem Does memo Solve?

By default, when a parent component renders, React normally renders its child components too.

```text
Parent renders
     ↓
Child renders
```

Even if the child's props are unchanged, its component function may run again.

Usually this is completely fine.

But if a child:

- renders very frequently,
- often receives the same props,
- and is expensive to render,

you may want React to skip that work.

That is the purpose of `memo`.

---

## 2. What Is memo? ⭐⭐⭐⭐⭐

`memo` is a React API that returns a memoized version of a component.

```jsx
import { memo } from "react";

const DeveloperCard = memo(
  function DeveloperCard({
    developer,
  }) {
    return (
      <article>
        <h2>
          {developer.name}
        </h2>
      </article>
    );
  }
);
```

Mental model:

```text
Parent re-renders
      ↓
compare previous props
with next props
      ↓
unchanged?
 ┌────┴────┐
yes       no
 ↓         ↓
skip      render
child     child
```

---

## 3. memo Is a Performance Optimization

This is essential:

> Your component must work correctly without `memo`.

Wrong mindset:

```text
Component behaves incorrectly
→ add memo
```

Correct mindset:

```text
Component is correct
→ profiling shows repeated expensive renders
→ memo may optimize them
```

Memoization should never be required for correctness.

---

## 4. Basic Example

```jsx
import {
  memo,
  useState,
} from "react";

const Greeting = memo(
  function Greeting({
    name,
  }) {
    console.log(
      "Greeting rendered"
    );

    return (
      <h2>
        Hello {name}
      </h2>
    );
  }
);

export default function App() {
  const [name, setName] =
    useState("Vikash");

  const [theme, setTheme] =
    useState("light");

  return (
    <>
      <button
        onClick={() =>
          setTheme(
            theme === "light"
              ? "dark"
              : "light"
          )
        }
      >
        Change theme
      </button>

      <Greeting
        name={name}
      />
    </>
  );
}
```

Changing `theme` renders `App`.

But `Greeting` still receives:

```js
name === "Vikash"
```

So the memoized child can skip rendering.

---

## 5. What Does memo Compare? ⭐⭐⭐⭐⭐

By default, React compares each prop with its previous value using `Object.is`.

Primitive example:

```jsx
<Greeting
  name="Vikash"
/>
```

If `name` remains the same, memoization can succeed.

But references matter for objects, arrays, and functions.

---

## 6. Object Props Can Break Memoization ⭐⭐⭐⭐⭐

Consider:

```jsx
<Profile
  user={{
    name: "Vikash",
  }}
/>
```

Every parent render creates a new object:

```text
render 1
{ name: "Vikash" } → reference A

render 2
{ name: "Vikash" } → reference B
```

Although contents look identical:

```js
Object.is(
  referenceA,
  referenceB
) === false
```

Therefore `memo` sees a changed prop.

---

## 7. Function Props Can Also Break Memoization

```jsx
<DeveloperCard
  onConnect={() =>
    connect(developer.id)
  }
/>
```

A new function is created during each render.

If the child is memoized, this function prop may make the props different every time.

Later:

```text
useCallback
→ can stabilize a function reference
when doing so is useful
```

But do not add `useCallback` automatically.

---

## 8. Arrays Have the Same Identity Issue

```jsx
<List
  items={items.filter(
    matchesSearch
  )}
/>
```

`filter()` returns a new array each render.

So:

```text
memo(List)
+
new array every render
=
memoization may be defeated
```

If filtering is expensive or stable identity matters for a memoized child, `useMemo` may help.

---

## 9. memo Does Not Block Own State Updates ⭐⭐⭐⭐⭐

```jsx
const Counter = memo(
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
);
```

Clicking the button changes `Counter`'s own state.

It must render.

`memo` only helps skip renders caused by parent rendering when relevant props remain unchanged.

---

## 10. Context Can Still Re-render a Memoized Component ⭐⭐⭐⭐⭐

```jsx
const Header = memo(
  function Header() {
    const theme =
      useContext(
        ThemeContext
      );

    return (
      <header>
        {theme}
      </header>
    );
  }
);
```

If the consumed Context value changes, `Header` needs the new value.

So it renders even though it is wrapped in `memo`.

Mental model:

```text
memo
does not freeze component

own state changes → render
consumed context changes → render
changed props → render
```

---

## 11. Parent Render vs Child DOM Update

Remember Lesson 12:

```text
component renders
≠
DOM necessarily changes
```

React can render a component, reconcile its result, and find no DOM changes.

Therefore a re-render is not automatically a performance problem.

`memo` skips component render work; it is not primarily about preventing DOM updates.

---

## 12. When memo Is Useful ⭐⭐⭐⭐⭐

A strong candidate usually has all three:

```text
1. renders frequently
2. often receives identical props
3. rendering is meaningfully expensive
```

Examples might include:

- complex visualization
- expensive list row
- large editor subtree
- expensive derived JSX tree
- granular interactive UI

Measure before deciding.

---

## 13. When memo Is Usually Unnecessary

Example:

```jsx
function Label({
  text,
}) {
  return <span>{text}</span>;
}
```

If rendering is extremely cheap, prop comparison and extra complexity may provide no useful benefit.

Do not optimize every component just because `memo` exists.

---

## 14. memo with Primitive Props

This is memo-friendly:

```jsx
<ProfileSummary
  name={user.name}
  age={user.age}
  isOnline={
    user.isOnline
  }
/>
```

Instead of passing a large object when the child only needs a few stable values.

Do not mechanically split every object prop, but minimize unnecessary prop changes when optimizing a measured hotspot.

---

## 15. Pass the Minimum Necessary Information

Suppose child only needs:

```text
hasGroups
```

Instead of:

```jsx
<CallToAction
  person={person}
/>
```

you could pass:

```jsx
<CallToAction
  hasGroups={
    person.groups !== null
  }
/>
```

A smaller API can change less frequently and also improves component responsibility.

---

## 16. Custom Prop Comparison

`memo` accepts an optional comparison function:

```jsx
const Chart = memo(
  ChartComponent,
  arePropsEqual
);
```

Example:

```jsx
function arePropsEqual(
  oldProps,
  newProps
) {
  return (
    oldProps.width ===
      newProps.width &&
    oldProps.height ===
      newProps.height
  );
}
```

Return:

```text
true
→ props considered equal
→ skip render

false
→ props considered different
→ render
```

This return meaning is easy to reverse accidentally.

---

## 17. Custom Comparators Are an Advanced Escape Hatch ⭐⭐⭐⭐⭐

A custom comparator has a cost.

React must run it on updates.

If comparison is more expensive than rendering, performance gets worse.

Also, comparing only some props incorrectly can produce stale UI or stale closures.

Use custom comparison only when:

- profiling justifies it,
- comparison is cheaper than rendering,
- correctness is easy to prove.

---

## 18. Never Ignore Function Props Carelessly

Dangerous comparator:

```jsx
function arePropsEqual(
  oldProps,
  newProps
) {
  return (
    oldProps.data ===
    newProps.data
  );

  // ignores onClick ❌
}
```

A function can close over state/props from the render where it was created.

Ignoring a changed function prop may leave the child using stale behavior.

Compare every prop that affects output or behavior.

---

## 19. Deep Equality Is Usually a Warning Sign

Avoid casually doing:

```js
JSON.stringify(oldProps) ===
JSON.stringify(newProps)
```

or arbitrary deep recursive comparison.

Deep comparison can be more expensive than simply rendering.

Prefer:

- better state design
- stable references where justified
- smaller prop APIs
- profiling

before complex custom equality logic.

---

## 20. memo + useMemo

Suppose:

```jsx
const VisibleList =
  memo(List);
```

Parent:

```jsx
const visibleItems =
  items.filter(
    (item) =>
      item.status ===
      status
  );

return (
  <VisibleList
    items={visibleItems}
  />
);
```

A new array is produced each render.

Potential optimization:

```jsx
const visibleItems =
  useMemo(
    () =>
      items.filter(
        (item) =>
          item.status ===
          status
      ),
    [items, status]
  );
```

Now the array reference can remain stable while dependencies remain unchanged.

Lesson 38 covers this deeply.

---

## 21. memo + useCallback

Memoized child:

```jsx
const SaveButton =
  memo(function SaveButton({
    onSave,
  }) {
    // ...
  });
```

Parent:

```jsx
const handleSave =
  useCallback(() => {
    saveApplication(id);
  }, [id]);
```

Now `onSave` can keep the same function reference while `id` is unchanged.

Lesson 39 explains when this is actually valuable.

---

## 22. One Always-New Prop Can Defeat memo ⭐⭐⭐⭐⭐

```jsx
<MemoizedChild
  name="Vikash"
  options={{ compact: true }}
/>
```

Even if every other prop is stable:

```text
options
→ new object every render
→ props differ
→ child renders
```

This is why memoization is a system of identities, not just one wrapper.

---

## 23. children Can Affect memo

This:

```jsx
<Panel>
  <ExpensiveContent />
</Panel>
```

passes `children` as a prop.

If new JSX is created each parent render, its identity can differ.

Do not assume wrapping a component in `memo` guarantees skipping renders regardless of its children/props.

---

## 24. Better Architecture Before Memoization ⭐⭐⭐⭐⭐

Many unnecessary renders are better solved by:

### Colocating state

```text
state needed by Search
→ keep it in Search
→ don't render entire App
```

### Accepting children

A wrapper with local visual state can often accept already-created children rather than owning their data.

### Avoiding unnecessary Effects

Effect chains that repeatedly set state can create far more rendering work than ordinary parent-child renders.

### Keeping render pure

Memoization should not hide bugs caused by impure rendering.

---

## 25. React Compiler Perspective ⭐⭐⭐⭐⭐

Modern React Compiler can automatically apply memoization equivalent to `memo` in compiled code.

This changes the modern recommendation:

```text
older/manual approach
→ proactively add memo/useMemo/useCallback

compiler-aware modern approach
→ write clear pure React code
→ let compiler optimize when enabled
→ use manual memoization for precise control
  or existing/non-compiled code when justified
```

You still need to understand `memo` because:

- many codebases use it,
- not every project uses React Compiler,
- interviews ask it,
- reference identity remains fundamental.

---

## 26. memo Is Not a Guarantee

Think:

```text
memo
= optimization hint/strategy
≠ semantic guarantee
```

React may still render a memoized component.

Never write logic that depends on the component being skipped.

---

## 27. CodeBuddy Example

Imagine a discovery feed:

```text
DiscoveryPage
 ├── search/filter controls
 └── DeveloperCard × many
```

If a parent state update causes many expensive cards to repeatedly render with unchanged props, profiling may show `DeveloperCard` as a hotspot.

Then:

```jsx
const DeveloperCard =
  memo(function DeveloperCard({
    developer,
    onConnect,
  }) {
    // expensive card UI
  });
```

But if `onConnect` is recreated every render or `developer` is reconstructed every time, memoization may not help.

---

## 28. CareerLoop Example

```text
ApplicationsPage
 ├── FilterBar
 └── ApplicationCard × 200
```

If typing somewhere unrelated causes all expensive cards to render, investigate:

1. Is state too high?
2. Are card props actually unchanged?
3. Is card rendering expensive?
4. Does profiling show a real bottleneck?

Only then consider memoization.

---

## 29. Common Mistakes ⭐⭐⭐⭐⭐

1. Wrapping every component in `memo`.
2. Assuming every re-render is bad.
3. Expecting `memo` to block own state updates.
4. Expecting `memo` to block Context updates.
5. Passing new object/array/function props every render.
6. Using expensive deep equality comparators.
7. Ignoring function props in a custom comparator.
8. Using memoization to fix correctness bugs.
9. Optimizing without profiling.
10. Forgetting that React Compiler can reduce manual memoization needs.
11. Treating `memo` as a guarantee rather than an optimization.

---

## 30. Interview Questions ⭐⭐⭐⭐⭐

### What is React.memo?

`memo` memoizes a component so React can usually skip rendering it when its props are unchanged.

### How are props compared by default?

Each prop is compared with its previous value using `Object.is`.

### Does memo prevent every re-render?

No.

### Will a memoized component render when its own state changes?

Yes.

### Will it render when consumed Context changes?

Yes.

### Why can an object prop break memoization?

A newly created object has a different reference even when its contents look equal.

### Why is useCallback often used with memo?

It can preserve a function prop's reference while dependencies remain unchanged.

### Why is useMemo often used with memo?

It can preserve a calculated object/array/value reference while dependencies remain unchanged.

### When is memo useful?

When a component renders frequently with the same props and its rendering work is meaningfully expensive.

### Should every component use memo?

No.

### What does a custom comparator return?

`true` means props are considered equal and the render can be skipped; `false` means they differ.

### How does React Compiler affect memo?

The compiler can automatically provide equivalent component memoization, reducing the need for manual `memo` in compiled code.

---

## 31. Complete Mental Model

```text
              Parent render
                   │
                   ↓
            memoized child
                   │
          compare all props
            using Object.is
                   │
          ┌────────┴────────┐
          ↓                 ↓
      unchanged          changed
          │                 │
          ↓                 ↓
    usually skip          render
       render              child

But:
own state change ───────→ render
context change ─────────→ render
```

---

## 32. Quick Revision

```jsx
import { memo } from "react";

const Child = memo(
  function Child({
    value,
  }) {
    return <div>{value}</div>;
  }
);
```

Remember:

```text
memo
→ component render optimization

best candidate:
frequent render
+ same props
+ expensive child
```

---

## 33. Key Takeaways

- `memo` can skip component rendering when props are unchanged.
- It is a performance optimization, not a correctness tool.
- Default prop comparison uses `Object.is`.
- Reference identity matters for objects, arrays, functions, and JSX.
- One always-new prop can defeat component memoization.
- Own state and consumed Context can still re-render a memoized component.
- Re-render does not automatically mean DOM update.
- Custom comparators should be rare and carefully measured.
- State colocation and good architecture often matter more than memoization.
- `useMemo` can stabilize calculated values.
- `useCallback` can stabilize function references.
- Profile before optimizing.
- React Compiler can automatically provide component memoization in compiled applications.
- Understanding manual memoization remains important for existing code and interviews.

---

## Next Lesson

➡️ [Lesson 38 — useMemo ⭐⭐⭐⭐⭐](./38-usememo.md)
