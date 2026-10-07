# Lesson 14 — Batching and State Update Queue

## 1. What Is Batching?

React can group multiple state updates before rendering.

```jsx
function handleClick() {
  setName("Vikash");
  setRole("Developer");
  setOnline(true);
}
```

Conceptually:

```text
event handler
   ↓
multiple state updates
   ↓
React processes them together
   ↓
next render
```

Batching reduces unnecessary renders and keeps UI updates consistent.

---

## 2. Why Three Direct Updates Do Not Mean +3

Suppose:

```text
count = 0
```

and you write:

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

All three lines read the same render snapshot:

```text
count = 0
```

So React effectively receives:

```text
setCount(1)
setCount(1)
setCount(1)
```

This does not produce 3.

---

## 3. Functional Updaters

When the next state depends on previous state, use an updater function:

```jsx
setCount((count) => count + 1);
setCount((count) => count + 1);
setCount((count) => count + 1);
```

React can process them in order:

```text
0 → 1 → 2 → 3
```

### Rule to Remember

Use:

```js
setCount(count + 1);
```

when you are replacing state from the current render value.

Use:

```js
setCount((current) => current + 1);
```

when the next value depends on previous or pending state.

---

## 4. Replacement vs Updater

Example:

```jsx
setCount(5);
setCount((count) => count + 1);
```

Result:

```text
5 → 6
```

But:

```jsx
setCount((count) => count + 1);
setCount(5);
```

the later replacement wins:

```text
final = 5
```

Queue order matters.

---

## 5. Updater Functions Must Be Pure

Good:

```jsx
setCount((count) => count + 1);
```

Avoid side effects inside an updater:

```jsx
setCount((count) => {
  sendAnalytics();

  return count + 1;
});
```

Updater functions should only calculate the next state.

---

## 6. Object and Array Updates

Object:

```jsx
setUser((user) => ({
  ...user,
  points: user.points + 1,
}));
```

Array add:

```jsx
setItems((items) => [
  ...items,
  newItem,
]);
```

Array remove:

```jsx
setItems((items) =>
  items.filter(
    (item) => item.id !== id
  )
);
```

Array update:

```jsx
setItems((items) =>
  items.map((item) =>
    item.id === id
      ? { ...item, status: "Done" }
      : item
  )
);
```

---

## 7. Modern React Automatic Batching

Modern React can batch updates from more than just click handlers.

Example:

```jsx
setTimeout(() => {
  setCount((count) => count + 1);
  setOpen((open) => !open);
}, 1000);
```

The important idea is:

> React often groups related updates before the next render.

Do not overthink the internal scheduling unless you are debugging advanced behavior.

---

## 8. Better Interview Explanation

Avoid saying only:

> setState is asynchronous.

A better answer is:

> React state setters queue updates for a future render. The current render keeps its existing state snapshot, and React may batch multiple queued updates before rendering.

---

## Common Mistakes

### Mistake 1 — Expecting repeated direct updates to accumulate

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

They can all use the same snapshot.

### Mistake 2 — Using updater functions without understanding why

Use them when the next state depends on previous state.

### Mistake 3 — Putting side effects inside updater functions

Updater functions should stay pure.

### Mistake 4 — Using setTimeout to "wait for state"

A timeout callback can still close over an old render value.

---

## Interview Questions

### What is batching?

React grouping multiple state updates before rendering.

### Why do three `setCount(count + 1)` calls not add three?

Because all three can read the same state snapshot.

### How do you correctly increment three times?

```jsx
setCount((c) => c + 1);
setCount((c) => c + 1);
setCount((c) => c + 1);
```

### When should you use a functional updater?

When the next state depends on previous or pending state.

### Why should updater functions be pure?

Because React expects them to calculate state without side effects.

---

## Quick Revision

```text
State snapshot
   ↓
setter queues update
   ↓
updates can be batched
   ↓
React processes queue
   ↓
next render
```

Remember:

1. Direct updates can read the same snapshot.
2. Functional updaters can build on pending state.
3. Queue order matters.
4. Updaters should be pure.

---

## Next Lesson

➡️ [Lesson 15 — Updating Objects and Arrays in State](./15-objects-arrays-state.md)
