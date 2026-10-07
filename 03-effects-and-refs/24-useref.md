# Lesson 24 — useRef

`useRef` gives you a value that:

- stays available across renders
- can be changed with `.current`
- does **not** cause a re-render when changed

Basic syntax:

```jsx
const ref = useRef(initialValue);
```

React gives you an object like:

```js
{
  current: initialValue
}
```

---

## 1. State vs Ref

This is the most important difference.

### State

```jsx
const [count, setCount] =
  useState(0);
```

Changing state:

```text
setState
↓
React renders again
```

Use state when the value affects the UI.

### Ref

```jsx
const countRef =
  useRef(0);
```

Changing a ref:

```jsx
countRef.current += 1;
```

does not cause a re-render.

Use a ref when the value must persist but does not directly control the UI.

---

## 2. Ref vs Normal Variable

Normal variables are recreated on every render.

```jsx
function Component() {
  let value = 0;
}
```

A ref persists:

```jsx
function Component() {
  const valueRef =
    useRef(0);
}
```

Mental model:

```text
normal variable
→ recreated every render

state
→ persists + causes render

ref
→ persists + no render
```

---

## 3. Do Not Use Refs for Visible UI State

Bad:

```jsx
function Counter() {
  const count =
    useRef(0);

  function handleClick() {
    count.current += 1;
  }

  return (
    <button onClick={handleClick}>
      {count.current}
    </button>
  );
}
```

The value changes, but the UI may not update because ref changes do not trigger rendering.

Use state instead:

```jsx
const [count, setCount] =
  useState(0);
```

Rule:

> If changing the value should update the screen, use state.

---

## 4. Store Non-UI Values

A common use case is storing a timer ID.

```jsx
const timeoutRef =
  useRef(null);

function handleSearch() {
  clearTimeout(
    timeoutRef.current
  );

  timeoutRef.current =
    setTimeout(() => {
      console.log("Search");
    }, 500);
}
```

Why a ref?

```text
timer ID must persist
+
changing it should not render UI
```

---

## 5. Accessing DOM Elements

Refs are also used to access DOM nodes.

```jsx
function SearchInput() {
  const inputRef =
    useRef(null);

  function handleFocus() {
    inputRef.current
      ?.focus();
  }

  return (
    <>
      <input ref={inputRef} />

      <button
        onClick={handleFocus}
      >
        Focus
      </button>
    </>
  );
}
```

After React creates the input:

```text
inputRef.current
→ actual DOM input element
```

Common DOM-ref use cases:

- focus
- scroll
- media control
- measurement
- third-party DOM libraries

---

## 6. DOM Ref Can Be null

Before the element exists:

```text
inputRef.current = null
```

After React commits the element:

```text
inputRef.current = DOM node
```

So this is often safer:

```jsx
inputRef.current?.focus();
```

---

## 7. Focus After Conditional Rendering

This can fail:

```jsx
function handleEdit() {
  setEditing(true);
  inputRef.current?.focus();
}
```

Why?

The new input may not be in the DOM yet.

A common solution:

```jsx
useEffect(() => {
  if (editing) {
    inputRef.current
      ?.focus();
  }
}, [editing]);
```

Flow:

```text
state changes
↓
render
↓
DOM committed
↓
Effect
↓
focus input
```

---

## 8. Ref Mutation Is Immediate

State:

```jsx
setCount(count + 1);

console.log(count);
```

still sees the current render snapshot.

Ref:

```jsx
countRef.current += 1;

console.log(
  countRef.current
);
```

sees the new value immediately.

Why?

```text
state
→ React-managed snapshot

ref.current
→ normal mutable property
```

---

## 9. Refs and Stale Closures

A ref can help a long-lived callback read the latest value.

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

The callback may be old, but it reads the current ref value.

Use this only when you truly need the latest mutable value.

---

## 10. Do Not Use Refs to Hide Effect Dependencies

Suppose changing `roomId` should reconnect a socket.

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

Do not move `roomId` into a ref just to stop the Effect from re-running.

Rule:

> If changing a value should restart synchronization, keep it as a dependency.

---

## 11. State vs Ref Decision

Ask:

> If this value changes, should the UI re-render?

If yes:

```text
use state
```

If no, but the value must persist across renders:

```text
use ref
```

If it does not need to persist:

```text
normal variable
```

---

## Common Mistakes

### Mistake 1 — Using refs instead of state

If the UI depends on the value, use state.

### Mistake 2 — Expecting ref changes to re-render

They do not.

### Mistake 3 — Reading a DOM ref before the element exists

The ref may still be `null`.

### Mistake 4 — Using refs to avoid correct Effect dependencies

Refs should not hide real reactive behavior.

### Mistake 5 — Mutating refs during render without a clear reason

Prefer using refs from event handlers and Effects.

---

## Interview Questions

### What is useRef?

A Hook that stores a persistent mutable value without causing re-renders when `.current` changes.

### What is the main difference between state and ref?

State changes trigger rendering. Ref changes do not.

### When should you use useRef?

For persistent non-UI values or imperative DOM access.

### Why can useRef help with stale callbacks?

Because old callbacks can read the latest value from the same stable ref object.

### Should refs replace state?

No. Use state when changes should affect the rendered UI.

---

## Quick Revision

```text
UI should update?
→ state

Need persistent value
without rendering?
→ ref

Need temporary value only
during this render?
→ normal variable
```

Common ref use cases:

```text
timer ID
DOM node
latest mutable value
imperative browser API
```

---

## Next Lesson

➡️ [Lesson 25 — DOM Refs and useImperativeHandle](./25-dom-refs-imperative-handle.md)
