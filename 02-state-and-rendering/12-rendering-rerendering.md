# Lesson 12 — How React Rendering and Re-rendering Work ⭐⭐⭐⭐⭐

## 1. Why Rendering Is One of the Most Important React Concepts

Many React problems come from an incorrect mental model of rendering.

Common questions include:

- What exactly happens when state changes?
- Does React recreate the whole DOM?
- Does a child render when its parent renders?
- Does changing a normal variable trigger rendering?
- Is a re-render bad?
- What is the difference between render and commit?

The first rule to remember is:

> **A React render is not the same thing as a DOM update.**

---

## 2. What Does "Render" Mean in React? ⭐⭐⭐⭐⭐

Rendering means React calls components to calculate what the UI should look like.

```jsx
function Greeting({ name }) {
  return <h1>Hello {name}</h1>;
}
```

Conceptually:

```text
Component function runs
        ↓
returns JSX
        ↓
React gets a UI description
```

JSX does not directly mutate the DOM.

---

## 3. The High-Level Rendering Flow ⭐⭐⭐⭐⭐

A useful mental model is:

```text
Trigger
   ↓
Render
   ↓
Reconciliation
   ↓
Commit
   ↓
Browser Paint
```

### Trigger

Something tells React rendering work may be needed.

### Render

React calls components and calculates the next UI.

### Reconciliation

React determines how the new output relates to the previous output.

### Commit

React applies required changes to the DOM.

### Browser paint

The browser eventually displays the updated result.

---

## 4. Initial Render

```jsx
import { createRoot } from "react-dom/client";
import App from "./App";

createRoot(
  document.getElementById("root")
).render(<App />);
```

Conceptually:

```text
root.render(<App />)
        ↓
React renders App
        ↓
renders descendants
        ↓
calculates UI
        ↓
commits DOM
        ↓
browser displays page
```

This is the initial render.

---

## 5. What Is a Re-render?

A **re-render** means React executes a component again to calculate its latest output.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  console.log("Counter rendered");

  return (
    <button onClick={() => setCount((c) => c + 1)}>
      {count}
    </button>
  );
}
```

After clicking:

```text
setCount(...)
    ↓
update scheduled
    ↓
Counter executes again
    ↓
new JSX calculated
```

---

## 6. Re-render Does Not Mean Full DOM Recreation ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <main>
      <h1>My App</h1>
      <p>Count: {count}</p>
      <footer>Copyright</footer>
    </main>
  );
}
```

When `count` changes, React may execute `Counter` again.

It does not mean React blindly deletes and recreates every DOM element.

Conceptually:

```text
Previous UI
    ↓
Next UI
    ↓
Reconciliation
    ↓
Only required DOM mutation
    ↓
<p> text changes
```

Remember:

```text
Component render ≠ DOM update
```

---

## 7. What Can Trigger Rendering? ⭐⭐⭐⭐⭐

Important triggers include:

### Initial render

```jsx
root.render(<App />);
```

### State update

```jsx
setCount((c) => c + 1);
```

### Parent rendering

When a parent renders, React normally continues rendering child components in the subtree returned by that parent.

### Context update

Components consuming a changed context value can render again.

### Other subscribed React updates

External stores and other React APIs can also schedule work.

---

## 8. Normal Variables Do Not Trigger Rendering

```jsx
function Counter() {
  let count = 0;

  function handleClick() {
    count++;
  }

  return (
    <button onClick={handleClick}>
      {count}
    </button>
  );
}
```

Changing `count` does not ask React to render again.

This is one reason state exists.

---

## 9. Parent and Child Rendering ⭐⭐⭐⭐⭐

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  console.log("Parent render");

  return (
    <>
      <button onClick={() => setCount((c) => c + 1)}>
        {count}
      </button>

      <Child />
    </>
  );
}

function Child() {
  console.log("Child render");

  return <p>I am child</p>;
}
```

When Parent state changes, by default you can observe:

```text
Parent render
Child render
```

even though `Child` has no changed props.

Useful default mental model:

> When a component renders, React normally renders the child components returned by it as part of that subtree.

Memoization can sometimes allow React to skip child work.

---

## 10. Important Interview Trap ⭐⭐⭐⭐⭐

Incorrect:

> A component re-renders only when its props or state change.

That is incomplete.

A better answer:

> Rendering can happen because of state updates, parent rendering, consumed context changes, and other subscribed React updates. Memoization can sometimes let React skip component work.

---

## 11. Child Render Does Not Mean Child DOM Update

Suppose:

```text
Parent renders
      ↓
Child renders
      ↓
Child returns same UI
      ↓
React reconciles
      ↓
No child DOM mutation needed
```

This distinction is critical:

```text
Render work ≠ DOM work
```

---

## 12. Render Phase ⭐⭐⭐⭐⭐

During rendering, React calculates the next UI.

```text
props + state + context
          ↓
component function
          ↓
React elements
          ↓
next UI description
```

Rendering should be **pure**.

---

## 13. Why Rendering Must Be Pure ⭐⭐⭐⭐⭐

Bad:

```jsx
function Product({ product }) {
  localStorage.setItem("lastProduct", product.id);

  return <h2>{product.name}</h2>;
}
```

This performs an external side effect while rendering.

A useful separation is:

```text
Render
  ↓
calculate UI

Event Handler
  ↓
interaction-driven work

Effect
  ↓
synchronize with an external system
```

React may execute rendering logic multiple times or perform work that does not immediately commit, so render code must not depend on causing external effects.

---

## 14. Commit Phase ⭐⭐⭐⭐⭐

After React determines what must change, it performs commit work.

```text
Render
   ↓
calculate next UI
   ↓
Reconciliation
   ↓
required changes
   ↓
Commit
   ↓
DOM mutations
```

Commit work can include:

- inserting DOM nodes
- removing DOM nodes
- changing text
- changing DOM properties
- updating refs at the appropriate time

Lesson 48 covers render vs commit in greater depth.

---

## 15. Reconciliation ⭐⭐⭐⭐⭐

Reconciliation is React's process for determining how the previous rendered tree corresponds to the next one.

Previous:

```jsx
<h1>Hello Vikash</h1>
```

Next:

```jsx
<h1>Hello Aman</h1>
```

Conceptually:

```text
same <h1> type
      ↓
text changed
      ↓
reuse element
      ↓
update text
```

React does not need to replace everything.

---

## 16. Type and Identity Matter

Previous:

```jsx
<div>
  <Counter />
</div>
```

Next:

```jsx
<section>
  <Counter />
</section>
```

Changes to the tree structure can affect identity and state preservation.

React reasons using concepts such as:

- element/component type
- position in the tree
- keys among sibling lists

We will study this deeply later.

---

## 17. Keys and Rendering ⭐⭐⭐⭐⭐

```jsx
users.map((user) => (
  <UserCard
    key={user.id}
    user={user}
  />
))
```

Keys help React match sibling children.

```text
Previous:
A
B
C

Next:
B
C
D

Using stable keys:
A → removed
B → preserved/moved
C → preserved/moved
D → created
```

This is why:

> **key = logical identity among siblings.**

---

## 18. Rendering and State Preservation ⭐⭐⭐⭐⭐

```jsx
{showCounter && <Counter />}
```

Conceptually:

```text
showCounter = true
Counter exists
count = 5

        ↓

showCounter = false
Counter removed
state destroyed

        ↓

showCounter = true
new Counter
initial state
```

State preservation depends on component identity in the rendered tree.

Lesson 18 covers this deeply.

---

## 19. State Does Not Change Inside the Current Render ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);

    console.log(count);
  }

  return (
    <button onClick={handleClick}>
      {count}
    </button>
  );
}
```

If `count` is 0, the log remains 0 in that handler.

Mental model:

```text
Current render:
count = 0
    ↓
setCount(1)
    ↓
request future render
    ↓
current count still 0

Next render:
count = 1
```

This is the state snapshot model covered in Lesson 13.

---

## 20. React Can Batch Updates ⭐⭐⭐⭐⭐

```jsx
function handleClick() {
  setName("Vikash");
  setRole("Developer");
}
```

React can process multiple updates together.

```text
Event
 ↓
setName(...)
setRole(...)
 ↓
updates queued
 ↓
React processes them
 ↓
render
 ↓
commit
```

Batching is covered in Lesson 14.

---

## 21. Setting State to an Equivalent Value

Suppose:

```jsx
const [count, setCount] = useState(5);
```

Then:

```jsx
setCount(5);
```

React can avoid unnecessary work when the requested state is equivalent to the current state.

React uses `Object.is` semantics for this comparison.

This is also relevant to object references.

---

## 22. Why Object Mutation Causes Problems ⭐⭐⭐⭐⭐

```jsx
const [user, setUser] = useState({
  name: "Vikash",
});
```

Wrong:

```jsx
user.name = "Aman";
setUser(user);
```

Reference:

```text
Before:
user ─────→ Object A

After mutation:
user ─────→ Object A
```

Better:

```jsx
setUser((current) => ({
  ...current,
  name: "Aman",
}));
```

Now:

```text
Old state ───→ Object A
New state ───→ Object B
```

This follows React's immutable state model.

---

## 23. Does Every State Update Re-render the Entire App?

No.

Suppose:

```text
App
├── Header
├── Main
│   └── Counter
└── Footer
```

If local state inside `Counter` changes, React can begin rendering work from `Counter` and its relevant subtree.

Conceptually:

```text
Counter state update
        ↓
Counter rendering work
        ↓
relevant descendants
```

React does not need to execute unrelated ancestors merely because a descendant updated its own state.

---

## 24. State Location Affects Rendering Scope

Compare:

### State high in the tree

```text
App owns query
    ↓
App renders
    ↓
larger subtree participates
```

### State close to where it is needed

```text
SearchBox owns local UI state
        ↓
SearchBox renders
        ↓
smaller subtree participates
```

Good state colocation can improve both architecture and performance.

Do not move state only for performance without a real design reason or measurement.

---

## 25. Re-renders Are Normal ⭐⭐⭐⭐⭐

A common misconception:

> Re-render = performance problem.

React is designed to render components.

The better question is:

> Is the rendering work expensive enough to matter?

A healthy optimization process is:

```text
Correct code
    ↓
Good state ownership
    ↓
Measure performance
    ↓
Find real bottleneck
    ↓
Optimize
```

Do not add memoization everywhere automatically.

---

## 26. Render Frequency vs Render Cost

A useful mental model:

```text
Performance impact
       ≈
render frequency
       ×
render cost
```

This is not an exact formula.

A frequently rendered tiny component may be cheap.

A rarely rendered component may still perform an expensive calculation.

Measure real bottlenecks.

---

## 27. React.memo Preview

By default:

```text
Parent renders
      ↓
Child normally renders
```

A memoized child can sometimes be skipped:

```jsx
const Child = memo(function Child({ name }) {
  return <p>{name}</p>;
});
```

If its props compare equal, React may skip rendering that component.

Memoization has costs and tradeoffs.

Lesson 37 covers `React.memo`.

---

## 28. useMemo and useCallback Do Not Prevent Rendering by Themselves ⭐⭐⭐⭐⭐

A common interview misconception:

> useMemo or useCallback prevents my component from re-rendering.

No.

`useMemo` caches a calculated value.

`useCallback` caches a function reference.

The component containing them can still render.

They are useful in specific optimization and dependency scenarios covered later.

---

## 29. Context and Rendering Preview

A component can consume context:

```jsx
const theme = useContext(ThemeContext);
```

When the relevant context value changes:

```text
Provider value changes
        ↓
context consumers
        ↓
rendering work
```

Context gets a dedicated lesson later.

---

## 30. StrictMode and Development Rendering ⭐⭐⭐⭐⭐

With:

```jsx
<StrictMode>
  <App />
</StrictMode>
```

React may intentionally invoke rendering logic extra times in development to help detect impure logic and other problems.

Therefore you may see a render log more often than expected.

Important:

> Development Strict Mode checks are not the same as normal production behavior.

This is another reason render logic must be pure.

---

## 31. Do Not Perform External Actions During Render

Avoid using render execution itself for:

- analytics events
- purchases
- API mutations
- storage writes
- manual DOM mutation

Rendering is for calculating UI.

Use event handlers for interaction-driven actions and Effects for appropriate external synchronization.

---

## 32. React Render vs Browser Paint ⭐⭐⭐⭐⭐

These are different concepts.

```text
React Render
    ↓
calculate UI

React Commit
    ↓
update DOM

Browser
    ↓
layout / paint
    ↓
pixels
```

Do not use "React render" and "browser paint" as synonyms.

---

## 33. Complete React Update Flow ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button
      onClick={() => setCount((c) => c + 1)}
    >
      Count: {count}
    </button>
  );
}
```

Conceptual flow:

```text
User clicks
    ↓
event handler
    ↓
setCount updater queued
    ↓
React schedules work
    ↓
Counter renders
    ↓
new UI description
    ↓
reconciliation
    ↓
text difference found
    ↓
commit
    ↓
DOM text updated
    ↓
browser paints
```

This is one of the most useful React interview diagrams.

---

## 34. Real-World Example

```jsx
function SearchPage({ products }) {
  const [query, setQuery] = useState("");

  const filteredProducts = products.filter((product) =>
    product.name
      .toLowerCase()
      .includes(query.toLowerCase())
  );

  return (
    <>
      <input
        value={query}
        onChange={(event) =>
          setQuery(event.target.value)
        }
      />

      <ProductList products={filteredProducts} />
    </>
  );
}
```

Typing causes:

```text
input event
    ↓
setQuery
    ↓
SearchPage renders
    ↓
filteredProducts recalculated
    ↓
ProductList rendered with new props
    ↓
reconciliation
    ↓
required DOM changes committed
```

This is ordinary React behavior and is not automatically a performance problem.

---

## 35. Common Mistakes

### Mistake 1 — Render means DOM recreation

No. Render calculates UI; commit applies required DOM changes.

### Mistake 2 — Children render only when props change

Parent rendering can normally cause child rendering too.

### Mistake 3 — Every re-render is bad

Re-renders are normal.

### Mistake 4 — Normal variable changes update React UI

They do not request React rendering.

### Mistake 5 — Side effects during render

Rendering should remain pure.

### Mistake 6 — Memoizing everything

Optimize only where it is useful.

### Mistake 7 — React render equals browser paint

They are separate stages.

### Mistake 8 — State setter changes the current variable immediately

State values are snapshots for each render.

### Mistake 9 — Extra StrictMode development renders mean React is broken

Strict Mode intentionally performs development checks.

---

## 36. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What does rendering mean in React?

**Answer:** React calls components to calculate the next UI description from their props, state, and context.

### Q2. Does a re-render recreate the DOM?

**Answer:** No. React can re-execute components, reconcile the output with the previous tree, and commit only required DOM changes.

### Q3. What are the high-level stages of a React update?

**Answer:** A useful model is trigger → render → reconciliation → commit, followed by the browser's own painting work.

### Q4. What can cause rendering?

**Answer:** Initial rendering, state updates, parent rendering, context changes, and other subscribed React updates can cause rendering work.

### Q5. Does a child render only when its props change?

**Answer:** No. Parent rendering normally causes rendering work through its child subtree unless React can skip it through mechanisms such as memoization.

### Q6. What is the render phase?

**Answer:** React executes components and calculates the next UI. Render logic should remain pure.

### Q7. What is the commit phase?

**Answer:** React applies the required changes to the DOM and performs other commit-related work.

### Q8. What is reconciliation?

**Answer:** It is React's process for determining how the previous and next rendered trees correspond so it can preserve, update, create, or remove the correct UI.

### Q9. Is a re-render always a performance issue?

**Answer:** No. Re-rendering is normal. Optimization should target measured expensive work.

### Q10. Do useMemo and useCallback prevent the containing component from rendering?

**Answer:** No. They cache a value or function reference but do not themselves stop the component from rendering.

### Q11. Why should rendering be pure?

**Answer:** React may execute render logic multiple times or discard unfinished work, so rendering should calculate UI without causing external side effects.

### Q12. Why can components appear to render extra times in development?

**Answer:** React Strict Mode can intentionally invoke rendering logic extra times to reveal impure code and other issues.

---

## 37. Interview Scenario ⭐⭐⭐⭐⭐

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  console.log("Parent");

  return (
    <>
      <button onClick={() => setCount((c) => c + 1)}>
        {count}
      </button>

      <Child name="React" />
    </>
  );
}

function Child({ name }) {
  console.log("Child");

  return <p>{name}</p>;
}
```

### Question

When `count` changes, does `Child` render even though `name` remains `"React"`?

### Answer

By default, yes.

`Parent` renders again, and React normally renders the child component returned by it.

But:

```text
Child component render
        ≠
Child DOM mutation
```

If the resulting paragraph has not changed, React may determine that no DOM update is needed for it.

---

## 38. Rendering Mental Model ⭐⭐⭐⭐⭐

```text
            UPDATE TRIGGER
                  │
                  ↓
          ┌──────────────┐
          │    RENDER    │
          └──────────────┘
                  │
          call components
                  │
                  ↓
          next UI description
                  │
                  ↓
           reconciliation
                  │
                  ↓
          required changes
                  │
                  ↓
          ┌──────────────┐
          │    COMMIT    │
          └──────────────┘
                  │
             DOM updates
                  │
                  ↓
            browser paint
```

Remember:

```text
React render
    ≠
DOM mutation
    ≠
browser paint
```

---

## 39. Quick Revision

```text
Render
  = execute components
  = calculate UI

Reconciliation
  = match old and new UI

Commit
  = apply required DOM changes
```

Common triggers:

```text
Initial render
State update
Parent render
Context update
Other subscribed updates
```

Important:

```text
Parent renders
      ↓
children normally render
      ↓
DOM changes only where required
```

Performance:

```text
Re-render
   ≠
performance bug

Measure first
Optimize second
```

---

## 40. Key Takeaways

- Rendering means React executes components to calculate UI.
- A re-render is not the same as recreating or changing the DOM.
- Use **trigger → render → reconciliation → commit** as the core mental model.
- Reconciliation determines how previous and next UI correspond.
- State updates can trigger rendering.
- Parent rendering normally causes child component rendering unless work is skipped.
- Child rendering does not mean child DOM nodes must change.
- A local state update does not require unrelated ancestors above that component to execute.
- Render logic should remain pure.
- State values belong to specific render snapshots.
- React can batch updates.
- Keys help React preserve sibling identity.
- State location can affect rendering scope.
- Re-renders are normal and should not be optimized blindly.
- `useMemo` and `useCallback` do not themselves prevent component renders.
- Strict Mode can intentionally perform extra rendering checks in development.
- React rendering, DOM committing, and browser painting are different concepts.

---

## Next Lesson

➡️ [Lesson 13 — State as a Snapshot ⭐⭐⭐⭐⭐](./13-state-as-snapshot.md)
