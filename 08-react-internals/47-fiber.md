# Lesson 47 — Fiber Architecture ⭐⭐⭐⭐⭐

## What Is React Fiber?

**Fiber** is React's internal architecture for organizing component work, tracking state and updates, and scheduling rendering.

A **Fiber node** is an internal record associated with a unit of React's component/element tree. Fiber is **not the browser DOM**.

```text
React App
   │
   ▼
App Fiber
   ├── Header Fiber
   └── Dashboard Fiber
         ├── Filter Fiber
         └── List Fiber
```

React uses this internal structure to decide what work needs doing and how to process it.

## Why Did React Introduce Fiber?

A large UI can require significant rendering work. React needs a way to organize updates so **not all work must be treated as one uninterrupted task**.

Fiber enables React to:

- Split rendering work into units.
- Assign different priorities to different updates.
- Pause, resume, restart, or abandon **rendering work** when appropriate.
- Keep the **commit phase** separate from rendering.

This supports responsive interfaces and React's concurrent rendering features. **It does not mean React always pauses rendering**, and it does not make JavaScript run on another thread.

## Practical Example — Search and a Large List

Imagine CareerLoop filtering thousands of applications while typing.

```jsx
import { useState, useTransition } from "react";

function Applications({ allApplications }) {
  const [query, setQuery] = useState("");
  const [filter, setFilter] = useState("");
  const [isPending, startTransition] = useTransition();

  const visible = allApplications.filter(app =>
    app.company.toLowerCase().includes(filter.toLowerCase())
  );

  function handleChange(event) {
    const next = event.target.value;
    setQuery(next); // Urgent: keep input responsive

    startTransition(() => {
      setFilter(next); // Lower-priority list update
    });
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <p>Updating results...</p>}
      <ul>
        {visible.map(app => (
          <li key={app.id}>{app.company}</li>
        ))}
      </ul>
    </>
  );
}
```

**How it works:**

1. Typing updates the input immediately through `setQuery`.
2. `startTransition` marks the list-filter update as non-urgent.
3. React can prioritize urgent work and interrupt/restart transition rendering if another input arrives.
4. The DOM receives the finished committed result.

**Note:** Transitions do not make expensive filtering computation free or automatically move it to a background thread. For very large lists, consider virtualization or moving truly heavy computation off the main thread.

## Render Phase vs Commit Phase ⭐⭐⭐⭐⭐

```text
UPDATE
  │
  ▼
RENDER PHASE
- Calculate next UI
- Reconcile Fiber work
- May be interrupted/restarted
- No DOM mutations yet
  │
  ▼
COMMIT PHASE
- Apply DOM mutations
- Run relevant layout work
- Not interruptible midway in the same way
```

This is why **render should be pure**: React may call a component more than once or discard a render before commit. Side effects during rendering can produce incorrect behavior.

## Fiber and Virtual DOM — Difference

| Concept | What it means |
|---|---|
| Virtual DOM | Informal name for React's JavaScript UI description |
| Reconciliation | Process of matching the previous and next UI |
| Fiber | Internal structure and architecture that organizes/schedules this work |
| Browser DOM | Actual UI nodes modified during commit |

**Mental model:** Virtual DOM describes **what** the UI should be; reconciliation decides **what changed**; Fiber helps React organize **how and when** rendering work is processed.

## Common Mistakes

- Thinking Fiber is a replacement for the actual DOM.
- Assuming concurrent rendering means multithreaded JavaScript.
- Believing every render is interrupted.
- Putting network calls or DOM changes directly into render functions.
- Assuming `useTransition` speeds up all expensive JavaScript.

## Interview Quick Check

**What is Fiber?** React's internal tree/work architecture for updates, state tracking, reconciliation, and scheduling.

**Why is Fiber important?** It supports prioritized and interruptible rendering work.

**Can the commit phase be interrupted like rendering?** No; React commits the selected update as a consistent DOM change.

**How is Fiber connected to reconciliation?** React uses Fiber nodes to track current and work-in-progress rendering trees and reconcile changes.

---

## Section 8 — React Internals Complete

- **Lesson 45:** Virtual DOM describes the UI.
- **Lesson 46:** Reconciliation identifies changes and preserves identity.
- **Lesson 47:** Fiber organizes and schedules rendering work.

➡️ [Back to React 19 README](../README.md)
