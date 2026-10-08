# Lesson 23 — Closures, Stale Closures and Effects

A closure is a function remembering variables from the scope where it was created.

In React, the main rule is:

> Every render has its own props, state and functions. A callback captures the values from the render that created it.

---

## 1. State Is a Render Snapshot

```jsx
function handleAlert() {
  setTimeout(() => {
    console.log(count);
  }, 3000);
}
```

If `count` is 5 when you click, this callback logs 5—even if later renders have a count of 6 or 7.

New renders create new callbacks. They do not change the values captured by an old callback.

**A stale closure is an old callback reading an old value when the logic needs a newer value.**

An old snapshot is not always wrong. A delayed purchase confirmation may intentionally refer to the product selected when the user clicked Buy.

---

## 2. Missing Effect Dependencies

This Effect stays connected to the initial room:

```jsx
useEffect(() => {
  const connection = connect(roomId);
  return () => connection.disconnect();
}, []); // Wrong: roomId is missing
```

Correct:

```jsx
useEffect(() => {
  const connection = connect(roomId);
  return () => connection.disconnect();
}, [roomId]);
```

When `roomId` changes, React cleans up the old connection and connects to the new room.

**Include every reactive value the Effect reads:** props, state, and variables/functions declared inside the component. Here, `connect` is assumed to be imported or defined outside the component.

Dependencies describe the code; they are not a list you choose to control how often it runs. If a dependency causes unwanted reruns, restructure the Effect rather than suppressing the dependency linter.

The same rule applies to event listeners: a listener installed with `[]` can keep the initial state. Including `[count]` lets cleanup remove the old listener and setup install one that captures the new count.

---

## 3. Fix State Updates with a Functional Updater

Classic interval bug:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1);
  }, 1000);

  return () => clearInterval(id);
}, []); // Wrong: captures the initial count
```

If the initial count is 0, the callback keeps requesting `setCount(1)`.

Correct:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(current => current + 1);
  }, 1000);

  return () => clearInterval(id);
}, []);
```

React supplies the updater with the pending state. The Effect no longer reads `count`, so it does not need `count` as a dependency. The state setter has stable identity.

Use this when **next state depends on previous state**.

It does not refresh other captured values:

```jsx
setMessages(current => [...current, newMessage]);
```

This avoids stale `messages`, but `newMessage` could still be stale.

---

## 4. Read a Recent Value with a Ref

Sometimes a long-lived callback needs a recent value without restarting the external synchronization.

Inside a component:

```jsx
const latestCount = useRef(count);

useEffect(() => {
  latestCount.current = count;
}, [count]);

function handleAlert() {
  setTimeout(() => {
    console.log(latestCount.current);
  }, 3000);
}
```

The callback captures the stable ref object and reads `current` when it runs. This example keeps the ref synchronized after the Effect runs; it does not write to the ref during rendering.

Changing a ref does not trigger a render.

| Need | Use |
| --- | --- |
| Reconnect when the room changes | Effect dependency |
| Update state from its previous value | Functional updater |
| Read a recent value without restarting a callback/subscription | Ref synchronized in an Effect |
| Keep the value from when the user acted | Captured snapshot |

Do not hide `roomId` in a ref if changing rooms should reconnect the subscription.

---

## 5. Stale Closure vs Race Condition

- **Stale closure:** a callback reads an older captured value.
- **Race condition:** overlapping async operations finish in an unexpected order—for example, an older search response overwrites a newer one.

Correct dependencies prevent stale synchronization, but do not by themselves prevent request races. Those need separate handling, such as cleanup that ignores obsolete results or cancels requests.

---

## 6. Debugging Checklist

1. Which render created this callback?
2. Which props or state did it capture?
3. Does it need that snapshot or a recent value?
4. Should a change restart the Effect?
5. Does the next state depend on previous state?

---

## Common Mistakes

- **Expecting every callback to read current state:** callbacks retain their render's values.
- **Using `[]` to stop reruns:** it can leave an Effect using old props/state.
- **Ignoring the dependency linter:** fix the code or Effect design.
- **Using refs to hide real dependencies:** reconnect when synchronization inputs change.
- **Assuming updaters fix every stale value:** they only supply the state being updated.
- **Treating every old snapshot as a bug:** the intended behavior decides.

---

## Interview Questions

### What is a stale closure in React?

A callback using values from an older render when its logic needs newer values.

### Why can setTimeout log old state?

Its callback captured the state from the render where it was created.

### How does a functional updater help?

React provides the pending state, so the update does not rely on captured state.

### Why do Effect dependencies matter?

They let React clean up and re-synchronize when reactive inputs change.

### When can a ref help?

When a callback needs a recent mutable value without restarting synchronization. Updating the ref does not render the UI.

### Is a stale closure the same as a race condition?

No. One concerns captured values; the other concerns async completion order.

---

## Quick Revision

- **Render snapshot:** old callbacks keep old render values.
- **Dependency:** restart synchronization when a reactive input changes.
- **Updater:** calculate next state from pending state.
- **Ref:** read a mutable value without triggering a render.
- **Intentional snapshot:** keep the value from when the action happened.

> Ask: does this callback need the value from when it was created, or a recent value when it runs?

---

## Next Lesson

➡️ [Lesson 24 — useRef](./24-useref.md)
