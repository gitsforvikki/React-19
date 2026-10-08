# Lesson 45 — Virtual DOM ⭐⭐⭐⭐⭐

## What Is the Virtual DOM?

The **Virtual DOM** is a common name for React's lightweight, JavaScript-based description of the UI. React elements describe **what should appear**, while the browser DOM contains the **actual nodes** you see.

Think of it like this:

```text
React component → UI description (React elements)
                        ↓
               React decides changes
                        ↓
                  Browser DOM
```

A React element is **not** a real DOM node:

```jsx
const heading = <h1>Hello Vikash</h1>;
```

This JSX describes an `h1`. React later creates or updates the actual DOM as needed.

## Practical Example — A Counter

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  console.log("Counter rendered");

  return (
    <div>
      <h1>Count: {count}</h1>
      <p>Welcome to React</p>
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </div>
  );
}
```

When you click **Increment**:

1. State changes from `0` to `1`.
2. React calls `Counter` again to calculate its next UI.
3. React compares the previous and next rendered result.
4. React updates the count text in the DOM. The unchanged paragraph does not need a DOM text update.

```text
Before                 After
Count: 0               Count: 1  ← changed
Welcome to React       Welcome to React  ← unchanged
```

**Important:** A component can **render again without any DOM changes**. React rendering means calculating UI; committing DOM changes is a separate step.

## Why Is This Useful?

- You describe the UI declaratively instead of manually updating DOM nodes.
- React can determine the DOM operations required by the new UI.
- Updates are usually limited to the parts that actually changed.

**Common misconception:** The Virtual DOM is not automatically faster than every possible direct DOM manipulation. Its main value is React's **declarative programming model** and coordinated updates. React also maintains an internal Fiber tree, which is more than simply a copy of the DOM.

## Virtual DOM vs Real DOM

| Virtual DOM / React elements | Browser DOM |
|---|---|
| JavaScript description of UI | Actual browser nodes |
| Produced by rendering React components | Updated in React's commit phase |
| Can be recalculated without visible changes | Mutated only when needed |

## Common Mistakes

- Thinking every re-render rebuilds the entire browser DOM.
- Assuming Virtual DOM means React never touches the real DOM.
- Believing all re-renders are performance problems.
- Treating the Virtual DOM as a literal DOM clone.

## Interview Quick Check

**What is Virtual DOM?** A JavaScript representation of the UI used by React to describe what should be rendered.

**Does a React re-render always update the DOM?** No. React may find no changes to commit.

**Is Virtual DOM always faster than direct DOM updates?** No. It helps React coordinate declarative UI updates, but has its own computation cost.

---

➡️ [Lesson 46 — Reconciliation Algorithm ⭐⭐⭐⭐⭐](./46-reconciliation.md)
