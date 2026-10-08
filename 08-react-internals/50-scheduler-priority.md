# Lesson 50 — React Scheduler, Priority and Concurrent Rendering

## Why Does React Need Scheduling?

Not all UI updates are equally urgent. Typing should feel immediate, while rendering a large filtered list can wait briefly.

React's Fiber architecture lets it organize rendering work and prioritize some updates over others.

```text
User types
   ├── Urgent: update input text
   └── Non-urgent: update filtered results
           ↓
     React may interrupt/restart rendering
           ↓
     Commit completed UI
```

**Concurrent rendering** means React can prepare work in an interruptible way. It does **not** mean JavaScript runs on multiple threads or that every render is interrupted.

## Practical Example — useTransition

Suppose CareerLoop filters a large list of applications.

```jsx
import { useState, useTransition } from "react";

function ApplicationSearch({ applications }) {
  const [input, setInput] = useState("");
  const [filter, setFilter] = useState("");
  const [isPending, startTransition] = useTransition();

  const visible = applications.filter(app =>
    app.company.toLowerCase().includes(filter.toLowerCase())
  );

  function handleChange(e) {
    const next = e.target.value;
    setInput(next); // Urgent: controlled input

    startTransition(() => {
      setFilter(next); // Non-urgent: results
    });
  }

  return (
    <>
      <input value={input} onChange={handleChange} />
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

### How It Works

1. `setInput` updates the controlled input promptly.
2. `startTransition` marks `setFilter` as non-urgent.
3. React may pause or restart rendering the list if newer urgent input arrives.
4. React commits the completed update.

**Important:** Never use a transition to control the text input itself. Transitions don't move filtering computation to another thread or automatically fix slow synchronous work.

## useDeferredValue — Another Option

When a component receives an urgent value but can render a slower result later:

```jsx
const deferredQuery = useDeferredValue(query);
const visible = applications.filter(app =>
  app.company.toLowerCase().includes(deferredQuery.toLowerCase())
);
```

`useDeferredValue` lets the result lag behind the latest input while React prioritizes responsiveness. It is **not** a debounce timer and does not guarantee a fixed delay.

## What About React's Internal Scheduler?

React assigns work different priorities internally and uses scheduling to decide when rendering should proceed. As application developers, we generally use **public APIs** like `useTransition` and `useDeferredValue`, rather than depending on unstable internal priority details.

## Common Mistakes

- Assuming concurrent rendering is multithreading.
- Expecting `startTransition` to speed up CPU-heavy calculations.
- Wrapping the controlled input's own state update in a transition.
- Confusing `useDeferredValue` with debouncing.
- Assuming render work always commits; interrupted work can be abandoned.

For very large lists, consider **virtualization**, server-side filtering, or a Web Worker for genuinely heavy computation.

## Interview Quick Check

**What is scheduling in React?** Organizing update work so urgent interactions can be prioritized.

**What does concurrent rendering mean?** React may interrupt, restart, or abandon render work before committing.

**useTransition vs useDeferredValue?** `useTransition` marks a state update as non-urgent; `useDeferredValue` allows a derived value to lag behind.

**Can React interrupt a commit midway?** No; interruption applies to rendering work, not a partially applied DOM commit.

---

## Section 8 — React Internals Complete

- Lesson 45: Virtual DOM
- Lesson 46: Reconciliation
- Lesson 47: Fiber
- Lesson 48: Render vs Commit
- Lesson 49: Keys and identity
- Lesson 50: Scheduling and concurrency

➡️ [Lesson 51 — Portals](../09-advanced-react/51-portals.md)
