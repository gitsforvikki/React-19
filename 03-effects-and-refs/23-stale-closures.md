# Lesson 23 — Closures, Stale Closures and Effects

Many confusing React bugs come from normal JavaScript closure behavior.

The main rule is:

> Every render creates its own values and functions. Functions created during that render remember that render's values.

---

## 1. Quick Closure Refresher

A closure means a function remembers variables from the scope where it was created.

```js
function createGreeting(name) {
  return function greet() {
    console.log(name);
  };
}

const greet =
  createGreeting("Vikash");

greet(); // Vikash
```

React uses normal JavaScript functions, so the same rule applies.

---

## 2. Each React Render Has Its Own Values

```jsx
function Counter() {
  const [count, setCount] =
    useState(0);

  function handleClick() {
    console.log(count);
  }

  return (
    <button onClick={handleClick}>
      {count}
    </button>
  );
}
```

Think like this:

```text
Render 1
count = 0
handler sees 0

Render 2
count = 1
new handler sees 1

Render 3
count = 2
new handler sees 2
```

Old functions do not automatically change to use newer render values.

---

## 3. What Is a Stale Closure?

A stale closure happens when an old callback uses an old value, but the logic actually needs the latest value.

Example:

```jsx
function Counter() {
  const [count, setCount] =
    useState(0);

  function handleAlert() {
    setTimeout(() => {
      console.log(count);
    }, 3000);
  }

  // ...
}
```

If `count` is 5 when the timeout is created, that callback remembers 5.

Even if count later becomes 6 or 7, the old callback can still log:

```text
5
```

This is normal closure behavior.

---

## 4. Old Value Is Not Always a Bug

Sometimes you want the value from the moment the action happened.

Example:

```jsx
function handleBuy() {
  setTimeout(() => {
    console.log(
      product.name
    );
  }, 1000);
}
```

If the user clicked Buy for Product A, keeping Product A in that callback may be correct.

So remember:

```text
old captured value
≠
always wrong
```

It is a stale bug only when the logic needs the latest value.

---

## 5. Missing Effect Dependency Causes Stale Values

Bad:

```jsx
useEffect(() => {
  const connection =
    connect(roomId);

  return () => {
    connection.disconnect();
  };
}, []);
```

The Effect uses:

```text
roomId
```

but the dependency array says:

```text
no changing dependency
```

If `roomId` changes, the Effect can stay connected to the old room.

Correct:

```jsx
useEffect(() => {
  const connection =
    connect(roomId);

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

Rule:

> If changing a value should restart the synchronization, that value belongs in the dependency array.

---

## 6. Classic Interval Bug

Bad:

```jsx
useEffect(() => {
  const id =
    setInterval(() => {
      setCount(count + 1);
    }, 1000);

  return () => {
    clearInterval(id);
  };
}, []);
```

If the Effect was created when:

```text
count = 0
```

the callback keeps doing:

```text
setCount(1)
```

So the counter can get stuck at 1.

---

## 7. Functional Updater Fix

Better:

```jsx
useEffect(() => {
  const id =
    setInterval(() => {
      setCount(
        (count) =>
          count + 1
      );
    }, 1000);

  return () => {
    clearInterval(id);
  };
}, []);
```

Why does this work?

Because React gives the updater the latest pending state.

Compare:

```text
setCount(count + 1)
→ uses captured count

setCount(current => current + 1)
→ React provides current pending state
```

Use a functional updater when the next state depends on previous state.

---

## 8. Functional Updaters Do Not Fix Everything

They only help when the stale value is the previous state being updated.

Example:

```jsx
setMessages((messages) => [
  ...messages,
  newMessage,
]);
```

This fixes stale `messages`.

But if `newMessage` itself is stale, that is a different problem.

---

## 9. Stale Event Listener

Bad:

```jsx
useEffect(() => {
  function handleKeyDown() {
    console.log(count);
  }

  window.addEventListener(
    "keydown",
    handleKeyDown
  );

  return () => {
    window.removeEventListener(
      "keydown",
      handleKeyDown
    );
  };
}, []);
```

The listener can keep reading the initial `count`.

If the listener should react to the latest count, one option is:

```jsx
useEffect(() => {
  function handleKeyDown() {
    console.log(count);
  }

  window.addEventListener(
    "keydown",
    handleKeyDown
  );

  return () => {
    window.removeEventListener(
      "keydown",
      handleKeyDown
    );
  };
}, [count]);
```

This removes the old listener and creates a new one when count changes.

---

## 10. Refs Can Hold the Latest Value

Sometimes you want a long-lived callback to stay stable but still read the latest value.

A ref can help:

```jsx
const latestCount =
  useRef(count);

latestCount.current =
  count;
```

Later:

```jsx
setTimeout(() => {
  console.log(
    latestCount.current
  );
}, 3000);
```

Mental model:

```text
state
→ render snapshot

ref.current
→ mutable latest value
```

Do not use refs just to avoid correct Effect dependencies.

---

## 11. Dependency or Ref?

Ask:

### Should changing this value restart the external synchronization?

If yes:

```text
use dependency
```

Example:

```text
roomId changes
→ disconnect old room
→ connect new room
```

### Should synchronization stay the same, but callback needs the latest value?

Then a ref or another latest-value pattern may be useful.

---

## 12. Stale Closure vs Race Condition

They are different.

### Stale closure

A callback uses an older captured render value.

```text
old callback
→ old value
```

### Race condition

Two async operations finish in the wrong order.

```text
Request A starts
Request B starts
B finishes
A finishes later
```

Do not confuse them.

---

## 13. Dependency Linter Is Helpful

If React's lint rule says:

```text
missing dependency: roomId
```

do not remove or ignore it just to stop the Effect from running.

The warning often means:

```text
Effect uses roomId
but dependencies do not track it
```

Fix the Effect design instead.

---

## 14. Simple Debugging Questions

When you suspect a stale closure, ask:

```text
1. Which render created this function?

2. Which props/state did it capture?

3. When does this function run?

4. Should it use the old value
   or the latest value?

5. Should the Effect re-run
   when this value changes?

6. Does next state depend
   on previous state?
```

These questions solve most stale-closure bugs.

---

## Common Mistakes

### Mistake 1 — Assuming callbacks always see latest state

They see values from the render that created them.

### Mistake 2 — Removing dependencies to stop reruns

This can create stale values.

### Mistake 3 — Using `[]` even though the Effect uses changing props/state

The Effect may stay stuck on old values.

### Mistake 4 — Using refs to hide real dependencies

If synchronization should restart, use the dependency.

### Mistake 5 — Thinking every old value is wrong

Sometimes the old snapshot is exactly what you want.

---

## Interview Questions

### What is a closure?

A function remembering variables from the scope where it was created.

### What is a stale closure in React?

A callback using values from an older render when the logic needs newer values.

### Why can setTimeout log an old state value?

Because the timeout callback was created during an earlier render and captured that render's state.

### How does a functional updater help?

It lets React provide the latest pending state instead of using a captured state value.

### Why are missing Effect dependencies dangerous?

Because the Effect can keep using old captured values instead of re-synchronizing.

### When can a ref help?

When a long-lived callback should stay active but needs to read the latest mutable value.

---

## Quick Revision

```text
Every render
→ new values
→ new functions
```

A function remembers:

```text
the render where
it was created
```

Common fixes:

```text
Effect should restart?
→ correct dependency

Next state depends on previous state?
→ functional updater

Long-lived callback needs latest value?
→ ref

Old value is actually intentional?
→ keep the snapshot
```

Main question:

> Does this callback need the value from when it was created, or the latest value now?

---

## Next Lesson

➡️ [Lesson 24 — useRef](./24-useref.md)
