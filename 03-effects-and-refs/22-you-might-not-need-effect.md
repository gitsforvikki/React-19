# Lesson 22 — You Might Not Need an Effect

A very important React rule:

> Do not use `useEffect` just because one React value depends on another.

Effects are mainly for synchronizing with systems outside React.

Before writing an Effect, ask:

> What external system am I synchronizing with?

If you cannot name one, the Effect may be unnecessary.

---

## 1. Derived Values Should Usually Be Calculated During Render

Bad:

```jsx
const [fullName, setFullName] =
  useState("");

useEffect(() => {
  setFullName(
    `${firstName} ${lastName}`
  );
}, [firstName, lastName]);
```

Better:

```jsx
const fullName =
  `${firstName} ${lastName}`;
```

No extra state. No extra render. No dependency problem.

---

## 2. Filtering and Counts Usually Do Not Need Effects

Bad:

```jsx
const [filtered, setFiltered] =
  useState([]);

useEffect(() => {
  setFiltered(
    users.filter((user) =>
      user.name
        .toLowerCase()
        .includes(
          query.toLowerCase()
        )
    )
  );
}, [users, query]);
```

Better:

```jsx
const filtered =
  users.filter((user) =>
    user.name
      .toLowerCase()
      .includes(
        query.toLowerCase()
      )
  );
```

Same rule for totals and counts:

```jsx
const total =
  items.reduce(
    (sum, item) =>
      sum + item.price,
    0
  );
```

---

## 3. User Actions Belong in Event Handlers

Bad:

```jsx
useEffect(() => {
  if (submitted) {
    sendOrder();
  }
}, [submitted]);
```

Better:

```jsx
function handleSubmit() {
  sendOrder();
}
```

If the user action is the reason something should happen, keep the logic in the event handler.

---

## 4. Do Not Synchronize Duplicate React State

Bad:

```jsx
function Child({ value }) {
  const [
    localValue,
    setLocalValue,
  ] = useState(value);

  useEffect(() => {
    setLocalValue(value);
  }, [value]);
}
```

If the child only needs to display the value:

```jsx
function Child({ value }) {
  return <p>{value}</p>;
}
```

Use props directly.

Only create local state if it intentionally has different meaning, such as an editable draft.

---

## 5. Resetting State May Be an Identity Problem

Suppose a form should reset when a different user is selected.

Instead of:

```jsx
useEffect(() => {
  setName(user.name);
  setEmail(user.email);
}, [user]);
```

you may model each form as a different component identity:

```jsx
<UserForm
  key={user.id}
  user={user}
/>
```

Use this when the whole subtree should reset for a different logical entity.

---

## 6. Share State by Lifting It Up

If two siblings need the same state, do not keep duplicate copies and synchronize them with Effects.

Use one owner:

```text
Parent state
   /      \
  ↓        ↓
Child A  Child B
```

Pass values and callbacks through props.

---

## 7. Expensive Calculations Are Not Effects

A slow calculation still does not automatically need `useEffect`.

Start with:

```jsx
const filtered =
  expensiveFilter(
    products,
    query
  );
```

If performance is actually a problem, `useMemo` may be appropriate:

```jsx
const filtered =
  useMemo(
    () =>
      expensiveFilter(
        products,
        query
      ),
    [products, query]
  );
```

`useMemo` is a performance optimization, not synchronization.

---

## 8. When an Effect Is Correct

Effects are useful for genuine external synchronization.

### Socket connection

```jsx
useEffect(() => {
  const socket =
    connect(roomId);

  return () => {
    socket.disconnect();
  };
}, [roomId]);
```

### Browser listener

```jsx
useEffect(() => {
  function handleResize() {
    console.log(
      window.innerWidth
    );
  }

  window.addEventListener(
    "resize",
    handleResize
  );

  return () => {
    window.removeEventListener(
      "resize",
      handleResize
    );
  };
}, []);
```

### Client-side fetching

```jsx
useEffect(() => {
  // fetch external data
}, [query]);
```

Client-side fetching can be a valid Effect use case, though frameworks or data libraries may provide better abstractions.

---

## 9. A Simple Decision Guide

Ask these questions in order:

```text
Can I calculate it from props/state?
→ calculate during render

Is it caused by a user action?
→ event handler

Is state duplicated or owned incorrectly?
→ fix state ownership

Should a new entity start fresh?
→ consider a meaningful key

Is there an external system?
→ useEffect
```

---

## Common Mistakes

### Mistake 1 — Storing derived values in state

Calculate them directly.

### Mistake 2 — Using Effects for button actions

Use event handlers.

### Mistake 3 — Mirroring props into state

Use the prop directly unless local state has a separate purpose.

### Mistake 4 — Chaining Effects that only transform React state

Keep source state minimal and derive the rest.

### Mistake 5 — Using useEffect for expensive calculations

Use normal rendering first, then `useMemo` only if needed.

---

## Interview Questions

### What does “You Might Not Need an Effect” mean?

Many things developers put in Effects can be handled more directly through rendering, event handlers, state ownership, or component identity.

### Should derived state use useEffect?

Usually no. Calculate it during render.

### Where should user-triggered logic go?

In the event handler.

### When is useEffect appropriate?

When React needs to synchronize with an external system.

### How can you reset an entire component for a different entity?

Use a meaningful `key` when changing component identity is the correct model.

---

## Quick Revision

```text
Derived value
→ calculate

User action
→ event handler

Shared state
→ lift state

Local-only state
→ colocate state

New logical entity
→ key/reset identity

External system
→ useEffect
```

The best question before writing an Effect is:

> What external system am I synchronizing with?

---

## Next Lesson

➡️ [Lesson 23 — Closures, Stale Closures and Effects](./23-stale-closures.md)
