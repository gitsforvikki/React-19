# Lesson 49 — Why Keys Matter During Reconciliation ⭐⭐⭐⭐⭐

## What Is a Key?

A **key** helps React identify an item among its siblings across renders, especially when list items are inserted, removed, or reordered.

```text
Before: A(id:1), B(id:2), C(id:3)
After:  C(id:3), A(id:1), B(id:2)

Stable IDs → React knows which item moved.
```

## Example — CareerLoop Applications

```jsx
function ApplicationList({ applications }) {
  return (
    <ul>
      {applications.map(app => (
        <li key={app.id}>
          {app.company} — {app.status}
        </li>
      ))}
    </ul>
  );
}
```

Use a stable database ID such as `app.id` for each item. Keys are **unique among siblings**, not necessarily globally unique.

## Why Index Keys Can Cause Bugs

Imagine three rows with local input state:

```text
Index key      Before                 After deleting first item
0              A: "note A"            B: "note A"  ❌
1              B: "note B"            C: "note B"  ❌
2              C: "note C"            removed
```

React may reuse the component state associated with position `0` for B because the key `0` did not change.

### Correct Stateful List

```jsx
import { useState } from "react";

function ApplicationRow({ app }) {
  const [note, setNote] = useState("");

  return (
    <li>
      {app.company}
      <input
        value={note}
        onChange={e => setNote(e.target.value)}
        placeholder="Private note"
      />
    </li>
  );
}

function ApplicationList({ applications }) {
  return (
    <ul>
      {applications.map(app => (
        <ApplicationRow key={app.id} app={app} />
      ))}
    </ul>
  );
}
```

With `app.id` as the key, the input state stays with the correct application when items reorder.

## Keys Can Also Reset State Intentionally

```jsx
function Editor({ selectedApplication }) {
  return (
    <ApplicationForm
      key={selectedApplication.id}
      application={selectedApplication}
    />
  );
}
```

Switching to a different application changes the key, so React mounts a fresh `ApplicationForm` and resets its local state.

**Mental model:** React preserves state for the same component **type + position/key identity**. Changing a key creates a new identity.

## Rules and Common Mistakes

- Prefer stable IDs from your data.
- Put `key` on the element returned directly from `map`, not on a nested element.
- Don't generate keys with `Math.random()` or `crypto.randomUUID()` during rendering; they change on every render.
- Array indices are acceptable for truly static lists that never reorder, insert, or delete items.
- `key` is a special React hint, **not a normal prop**; pass an ID separately if the child needs it.
- Keys do not prevent re-renders by themselves; they control identity and state preservation.

## Interview Quick Check

**Why do keys matter?** They let React match sibling elements across list updates.

**Why avoid index keys?** Reordering or deletion can attach local state to the wrong item.

**Are keys globally unique?** No, only among siblings.

**What happens when a key changes?** React treats it as a different identity and resets its local state.

---

➡️ [Lesson 50 — React Scheduler, Priority and Concurrent Rendering](./50-scheduler-priority.md)
