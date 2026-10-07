# Lesson 13 — State as a Snapshot

## 1. What Does "State as a Snapshot" Mean?

In React, each render receives its own fixed state values.

Calling a state setter does **not** change the state variable inside the current render.

It schedules a new render with the updated state.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    console.log(count);

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

On the first click:

```text
0
0
```

not:

```text
0
1
```

Because the current handler belongs to the render where:

```text
count = 0
```

---

## 2. Correct Mental Model

Do not think:

```text
count = 0
setCount(1)
count immediately becomes 1
```

Think:

```text
Render 1
count = 0
   ↓
setCount(1)
   ↓
React schedules another render
   ↓
Render 2
count = 1
```

Important rule:

> A setter updates state for a future render, not the current render.

---

## 3. Each Render Has Its Own Values

Imagine:

```text
Render 1 → count = 0
Render 2 → count = 1
Render 3 → count = 2
```

React calls your component function again for each render.

Each render gets new local variables and new event handlers.

This is why handlers can remember values from a particular render.

---

## 4. Multiple State Updates

Consider:

```jsx
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
}
```

If:

```text
count = 0
```

all three calls use the same snapshot:

```text
setCount(1)
setCount(1)
setCount(1)
```

So this does **not** mean:

```text
0 → 1 → 2 → 3
```

If the next state depends on previous state, use a functional updater:

```jsx
setCount((count) => count + 1);
setCount((count) => count + 1);
setCount((count) => count + 1);
```

React can process them as:

```text
0 → 1 → 2 → 3
```

### Rule to Remember

Use:

```js
setCount(count + 1);
```

when you simply want to replace state using the current render value.

Use:

```js
setCount((current) => current + 1);
```

when the next state depends on the previous or pending state.

---

## 5. State Snapshots and Closures

Event handlers are JavaScript functions, so they follow normal closure behavior.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function showLater() {
    setTimeout(() => {
      console.log(count);
    }, 3000);
  }

  return (
    <>
      <button onClick={showLater}>
        Show Later
      </button>

      <button
        onClick={() =>
          setCount((c) => c + 1)
        }
      >
        Increment
      </button>
    </>
  );
}
```

Suppose:

```text
count = 5
```

You click **Show Later**, then increase the count to 6.

The timeout can still print:

```text
5
```

Why?

The callback was created during the render where:

```text
count = 5
```

This is the foundation of the **stale closure** concept in React.

---

## 6. Calculate the Next Value When You Need It Immediately

If you need the next value inside the same event handler:

```jsx
function handleClick() {
  const nextCount =
    count + 1;

  setCount(nextCount);

  console.log(nextCount);
}
```

Do not expect this:

```jsx
setCount(count + 1);

console.log(count);
```

to print the updated state.

---

## 7. State Objects Are Not Deep-Cloned

"Snapshot" is a mental model.

It does **not** mean React deep-copies every state object.

Example:

```jsx
const [user, setUser] =
  useState({
    name: "Vikash",
  });
```

Avoid mutating state:

```js
user.name = "Kumar";
```

Prefer creating a new object:

```js
setUser({
  ...user,
  name: "Kumar",
});
```

This keeps React state updates predictable.

---

## Common Mistakes

### Mistake 1 — Expecting state to update immediately

```jsx
setCount(count + 1);

console.log(count);
```

The log still sees the current render's snapshot.

### Mistake 2 — Using direct updates repeatedly

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

All can use the same snapshot.

Use functional updates when each update depends on the previous one.

### Mistake 3 — Forgetting closures

Timers, promises, effects, and event handlers can keep values from older renders.

### Mistake 4 — Mutating state objects

Always prefer immutable updates.

---

## Interview Questions

### What does "state is a snapshot" mean?

Each React render receives a fixed state value. Calling a setter schedules a future render but does not change the state variable inside the current render.

### Why does console.log show the old state after setState?

Because the running event handler uses the state snapshot from the render that created it.

### Why don't three `setCount(count + 1)` calls necessarily increment by three?

Because all three can calculate from the same `count` snapshot.

### When should you use a functional updater?

When the next state depends on previous or pending state.

```js
setCount(
  (current) =>
    current + 1
);
```

### Why can setTimeout see an old state value?

Because its callback is a closure created during an earlier render.

---

## Quick Revision

```text
setState
   ≠
change current variable immediately

setState
   =
request another render
```

Each render gets:

```text
state snapshot
+
props
+
new event handlers
```

Remember these four points:

1. State is fixed for a particular render.
2. A setter schedules a future render.
3. Use functional updates when new state depends on previous state.
4. Callbacks can retain older render values because of closures.

---

## Next Lesson

➡️ [Lesson 14 — Batching and State Update Queue](./14-batching-state-queue.md)
