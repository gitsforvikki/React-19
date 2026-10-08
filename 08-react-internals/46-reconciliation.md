# Lesson 46 — Reconciliation Algorithm ⭐⭐⭐⭐⭐

## What Is Reconciliation?

**Reconciliation** is React's process of figuring out **what changed** between the previous and next UI trees.

```text
State / props change
       ↓
Render next UI
       ↓
Reconciliation: match old and new elements
       ↓
Commit necessary DOM changes
```

React does not blindly replace the whole DOM after every render.

## Example 1 — Same Element Type

```jsx
function Greeting({ name }) {
  return <h1>Hello {name}</h1>;
}

// Before: <Greeting name="Vikash" />
// After:  <Greeting name="Rahul" />
```

The `h1` remains an `h1`. React can **reuse the existing DOM element** and update its text.

## Example 2 — Different Element Type

```jsx
function Message({ important }) {
  return important
    ? <strong>Important</strong>
    : <span>Normal</span>;
}
```

When the rendered element changes from `span` to `strong`, React replaces that element's DOM subtree. A change in type also affects component state preservation when components occupy that position.

## Example 3 — Why Keys Matter ⭐⭐⭐⭐⭐

Suppose CareerLoop renders applications:

```jsx
function ApplicationList({ applications }) {
  return (
    <ul>
      {applications.map(app => (
        <li key={app.id}>
          {app.company}
        </li>
      ))}
    </ul>
  );
}
```

**Keys tell React which sibling item is which** when items are inserted, removed, or reordered.

```text
Before: [A(id:1), B(id:2), C(id:3)]
After:  [C(id:3), A(id:1), B(id:2)]

Stable keys → React can match A, B, C across the reorder.
```

Using an array index as the key can cause incorrect state association when items move, particularly for rows containing inputs or local component state.

**Key rule:** Keys should be **stable and unique among siblings**, usually a database ID.

## Example 4 — State Preservation and Reset

React associates state with a component's **position in the UI tree**, including its type and key.

```jsx
function ProfilePage({ userId }) {
  return <ProfileForm key={userId} userId={userId} />;
}
```

When `userId` changes, the key changes, so React treats it as a different component and **resets its local state**. Without a changed key, React generally preserves state when the same component type stays in the same position.

Use this intentionally for resetting a form when switching users.

## Reconciliation vs Rendering vs Commit

- **Rendering:** Calls components to calculate the next UI.
- **Reconciliation:** Determines which elements can be reused and what needs changing.
- **Commit:** Applies the required changes to the DOM (and runs the appropriate commit-phase work).

React can render and reconcile components without committing DOM updates.

## Common Mistakes

- Using `Math.random()` as a key (causes needless remounting).
- Using array indices as keys in reorderable/stateful lists.
- Assuming a child render automatically changes its DOM.
- Thinking keys are globally unique; they only need to be unique among siblings.
- Changing a component's key without realizing it resets state.

## Interview Quick Check

**What is reconciliation?** React's process of matching the old and new UI so it can determine updates.

**Why are keys important?** They preserve item identity across list changes.

**What happens when a key changes?** React treats the component as a new identity, resetting its local state.

**Does rendering mean DOM mutation?** No. DOM mutations occur in the commit phase only if needed.

---

➡️ [Lesson 47 — Fiber Architecture ⭐⭐⭐⭐⭐](./47-fiber.md)
