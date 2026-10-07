# Lesson 18 — Preserving and Resetting State; Component Identity

React preserves or resets state based on **component identity in the rendered tree**.

A useful practical mental model is:

```text
component identity
=
type
+
position
+
key when relevant
```

---

## 1. Re-rendering Does Not Reset State

```jsx
function Counter() {
  const [count, setCount] =
    useState(0);

  return (
    <button
      onClick={() =>
        setCount(
          (count) =>
            count + 1
        )
      }
    >
      {count}
    </button>
  );
}
```

When `Counter` re-renders at the same place with the same identity, React preserves its state.

```text
Render 1
count = 0

Render 2
count = 1

same identity
→ state preserved
```

---

## 2. Removing a Component Resets State

```jsx
{show && <Counter />}
```

If:

```text
show = false
```

the component is removed.

Its local state is discarded.

When it appears again, React creates a new instance with fresh state.

---

## 3. Different Component Type Resets State

```jsx
{loggedIn
  ? <Dashboard />
  : <LoginForm />
}
```

At that position, the component type changes.

```text
Dashboard
↓
LoginForm
```

React removes the old subtree and creates a new one.

State resets.

---

## 4. Same Type + New Props Usually Preserves State

```jsx
<Profile
  user={selectedUser}
/>
```

Changing `selectedUser` does not automatically reset local state.

If `Profile` remains the same type, position, and key, its state is usually preserved.

This is important for forms.

---

## 5. Reset State Intentionally with key

Suppose a chat has local draft state.

```jsx
<Chat
  key={contact.id}
  contact={contact}
/>
```

When `contact.id` changes:

```text
Chat key=1
↓
Chat key=2
```

React treats it as a different component identity.

The old local state is reset.

Use this when the whole component should start fresh for a new logical entity.

---

## 6. Key Is Also Important in Lists

Good:

```jsx
users.map((user) => (
  <UserRow
    key={user.id}
    user={user}
  />
))
```

Stable keys help React preserve the correct state with the correct item.

---

## 7. Why Index Keys Can Cause Bugs

Bad for dynamic lists:

```jsx
items.map((item, index) => (
  <Row
    key={index}
    item={item}
  />
))
```

If items are inserted, removed, or reordered, the same index may now represent a different item.

State can appear to move to the wrong row.

Prefer stable IDs.

---

## 8. Never Use Random Keys

Bad:

```jsx
<Component
  key={Math.random()}
/>
```

Every render gets a new key.

React treats it as a new component every time.

Possible effects:

- state resets
- DOM recreated
- focus lost
- unnecessary work

---

## 9. Do Not Define Components Inside Components

Bad:

```jsx
function Parent() {
  function Child() {
    const [count, setCount] =
      useState(0);

    return (
      <button>
        {count}
      </button>
    );
  }

  return <Child />;
}
```

Every parent render creates a new `Child` function object.

That can make React treat it as a new component type and reset state.

Better:

```jsx
function Child() {
  const [count, setCount] =
    useState(0);

  return (
    <button>
      {count}
    </button>
  );
}

function Parent() {
  return <Child />;
}
```

Define normal components at module level.

---

## 10. Preserve vs Reset Decision

Preserve state when the component represents the same logical thing.

Examples:

- theme changed
- parent re-rendered
- props changed
- unrelated sibling changed

Reset state when the component represents a new logical entity.

Examples:

- new chat contact
- different user form
- different product
- new workflow session

A meaningful key can express that new identity.

---

## Common Mistakes

### Mistake 1 — Expecting every re-render to reset state

It does not.

### Mistake 2 — Expecting new props to reset local state

Usually they do not.

### Mistake 3 — Using index keys for dynamic stateful lists

Use stable IDs.

### Mistake 4 — Using random keys

This remounts unnecessarily.

### Mistake 5 — Defining components inside components

This can break component identity.

### Mistake 6 — Adding keys just to fix a bug

Use keys intentionally when identity should change.

---

## Interview Questions

### When does React preserve state?

When React can match the component to the same compatible identity in the next render.

### Does a re-render reset state?

No.

### How can you intentionally reset state?

Change the component identity, commonly with a meaningful `key`.

### Why are random keys bad?

Because they cause remounting and state loss on every render.

### Why can index keys be dangerous?

Because state may stay attached to positions instead of the correct logical items.

### Why should components usually be defined at module level?

To keep the component type stable across renders.

---

## Quick Revision

```text
same type
+
same position
+
same key
→ preserve state
```

```text
removed component
OR different type
OR different meaningful key
→ reset state
```

Remember:

> State belongs to a component's identity in the rendered tree.

---

## Section 2 Complete

You have now covered the most important React state concepts:

```text
useState
↓
state snapshots
↓
batching
↓
immutable updates
↓
controlled inputs
↓
state ownership
↓
component identity
```

---

## Next Lesson

➡️ [Lesson 19 — useEffect Fundamentals](../03-effects-and-refs/19-useeffect.md)
