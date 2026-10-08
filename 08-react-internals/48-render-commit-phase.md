# Lesson 48 — Render Phase vs Commit Phase ⭐⭐⭐⭐⭐

## The Core Difference

A **render** calculates what the UI should look like. A **commit** applies necessary changes to the browser DOM.

```text
State/props update
      ↓
RENDER: run components + reconcile next UI
      ↓
COMMIT: apply necessary DOM changes
      ↓
Browser paints (when needed)
```

A component can render without changing any DOM node.

## Example — Parent and Child

```jsx
import { useState } from "react";

function Child() {
  console.log("Child rendered");
  return <p>Hello Vikash</p>;
}

export default function Parent() {
  const [count, setCount] = useState(0);
  console.log("Parent rendered");

  return (
    <>
      <h2>Count: {count}</h2>
      <Child />
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </>
  );
}
```

Click **Increment**:

1. `setCount` requests an update.
2. React runs `Parent` and normally `Child` again (render phase).
3. React finds that the Child's `<p>` output is unchanged.
4. React updates only the changed count text in the DOM (commit phase).

**Render ≠ DOM update.** `React.memo` can skip some child render work when props stay the same; it is not primarily about avoiding DOM changes that React already knows are unnecessary.

## Render vs Commit

| Render phase | Commit phase |
|---|---|
| Calculates next UI | Applies required DOM changes |
| Runs component functions | Updates DOM and refs |
| Must be pure | Performs commit-related work |
| May be restarted or discarded | A selected commit is applied consistently |

React can render and reconcile without needing any DOM mutation.

## Where Do Effects Fit?

- **`useLayoutEffect`** runs after DOM mutations but before the browser paints; use sparingly for layout measurement.
- **`useEffect`** runs after commit, generally after paint for non-interaction updates, but its timing relative to paint can vary.
- Neither Effect runs during render.

## Why Must Render Be Pure?

React may render more than once or abandon a render. Do **not** mutate the DOM, send network requests, or change external state directly in the component body.

```jsx
// ❌ Side effect during render
function Bad() {
  document.title = "Dashboard";
  return <h1>Dashboard</h1>;
}

// ✅ Synchronize an external system after render
function Good() {
  useEffect(() => {
    document.title = "Dashboard";
  }, []);
  return <h1>Dashboard</h1>;
}
```

In development, Strict Mode can re-run render logic to help expose impurities.

## Common Mistakes

- Thinking every re-render changes the DOM.
- Doing side effects directly during render.
- Assuming `useEffect` always runs strictly after paint.
- Adding `memo` to every small component without measuring.

## Interview Quick Check

**What is render?** React calculates the next UI and reconciles changes.

**What is commit?** React applies the selected updates to the DOM.

**Can render happen without a DOM update?** Yes.

**Why should render be pure?** React may retry or discard rendering work.

---

➡️ [Lesson 49 — Why Keys Matter During Reconciliation ⭐⭐⭐⭐⭐](./49-keys-reconciliation.md)
