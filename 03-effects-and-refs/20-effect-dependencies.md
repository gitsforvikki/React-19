# Lesson 20 — Dependency Arrays and Reactive Dependencies

The dependency array tells React **which reactive values an Effect depends on**.

Example:

```jsx
useEffect(() => {
  document.title =
    `Room: ${roomId}`;
}, [roomId]);
```

The Effect reads `roomId`, so `roomId` belongs in the dependency array.

---

## 1. What Counts as a Reactive Dependency?

Usually:

- props
- state
- values created inside the component that can change between renders

Example:

```jsx
function ChatRoom({
  serverUrl,
  roomId,
}) {
  useEffect(() => {
    const connection =
      createConnection(
        serverUrl,
        roomId
      );

    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
}
```

The Effect reads:

```text
serverUrl
roomId
```

so both are dependencies.

---

## 2. Dependencies Are Not a Scheduling Preference

Do not think:

> I want this Effect to run once, so I will use [].

Instead ask:

> Which reactive values does this Effect actually read?

If it reads a changing prop or state value, include it.

---

## 3. Missing Dependencies Cause Stale Values

Bad:

```jsx
useEffect(() => {
  console.log(count);
}, []);
```

If `count` changes later, this Effect still uses the value captured by the earlier render.

That is a stale closure problem.

Do not remove dependencies just to stop reruns.

---

## 4. How React Compares Dependencies

React compares dependencies using:

```js
Object.is(previous, next)
```

For primitives:

```text
1 → 1
same

"react" → "react"
same
```

For objects:

```js
const a = { roomId: 1 };
const b = { roomId: 1 };

Object.is(a, b);
// false
```

Different references mean different dependency values.

---

## 5. Object Dependency Problem

Bad:

```jsx
function ChatRoom({
  roomId,
}) {
  const options = {
    roomId,
  };

  useEffect(() => {
    const connection =
      connect(options);

    return () => {
      connection.disconnect();
    };
  }, [options]);
}
```

A new `options` object is created every render.

That can cause unnecessary Effect reruns.

Better:

```jsx
function ChatRoom({
  roomId,
}) {
  useEffect(() => {
    const options = {
      roomId,
    };

    const connection =
      connect(options);

    return () => {
      connection.disconnect();
    };
  }, [roomId]);
}
```

Now the Effect depends on the actual reactive value.

---

## 6. Function Dependency Problem

Functions created inside a component are new function objects on each render.

Instead of:

```jsx
function createOptions() {
  return {
    roomId,
  };
}

useEffect(() => {
  const options =
    createOptions();

  connect(options);
}, [createOptions]);
```

move the helper inside the Effect if it is only used there:

```jsx
useEffect(() => {
  function createOptions() {
    return {
      roomId,
    };
  }

  connect(
    createOptions()
  );
}, [roomId]);
```

Simplify before reaching for `useCallback`.

---

## 7. Static Values Can Stay Outside

If a value never depends on props or state, move it outside the component.

```jsx
const SERVER_URL =
  "https://example.com";

function ChatRoom({
  roomId,
}) {
  useEffect(() => {
    connect(
      SERVER_URL,
      roomId
    );
  }, [roomId]);
}
```

`SERVER_URL` is not reactive.

---

## 8. Cleanup and Dependency Changes

```jsx
useEffect(() => {
  const connection =
    connect(roomId);

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

When `roomId` changes:

```text
cleanup old room
↓
setup new room
```

Dependencies tell React when synchronization must restart.

---

## 9. Ref Values Are Different

A ref object is stable:

```jsx
const ref =
  useRef(null);
```

Changing:

```js
ref.current
```

does not trigger a render.

So `ref.current` is not a normal reactive dependency like state or props.

---

## 10. Often the Best Fix Is Removing the Effect

Bad:

```jsx
const [filtered, setFiltered] =
  useState([]);

useEffect(() => {
  setFiltered(
    products.filter(
      (product) =>
        product.name.includes(
          query
        )
    )
  );
}, [products, query]);
```

Better:

```jsx
const filtered =
  products.filter(
    (product) =>
      product.name.includes(
        query
      )
  );
```

No Effect. No dependency problem. No extra state.

---

## 11. Separate Unrelated Effects

Bad:

```jsx
useEffect(() => {
  connectToRoom(roomId);
  logPage(page);
}, [roomId, page]);
```

Now changing `page` may also reconnect the room.

Better:

```jsx
useEffect(() => {
  const connection =
    connectToRoom(roomId);

  return () => {
    connection.disconnect();
  };
}, [roomId]);

useEffect(() => {
  logPage(page);
}, [page]);
```

Each Effect should represent one synchronization process.

---

## Common Mistakes

### Mistake 1 — Choosing dependencies based on desired frequency

Dependencies come from values used by the Effect.

### Mistake 2 — Omitting dependencies to stop reruns

This can create stale behavior.

### Mistake 3 — Using objects created during render as dependencies

Their references can change every render.

### Mistake 4 — Using functions created during render as dependencies unnecessarily

Move them inside the Effect if possible.

### Mistake 5 — Suppressing the dependency lint rule instead of fixing the design

Usually the linter is showing a real problem.

---

## Interview Questions

### What belongs in a dependency array?

Reactive values read by the Effect.

### How does React compare dependencies?

Using `Object.is`.

### Why can object dependencies rerun Effects often?

Because new object references are different even if their contents look the same.

### Why can missing dependencies cause stale closures?

Because the Effect can keep using values from an older render.

### What is often the best fix for difficult dependency problems?

Simplify the Effect, move Effect-only values inside it, split unrelated Effects, or remove the Effect if it is unnecessary.

---

## Quick Revision

```text
Effect reads prop/state?
→ dependency

Object/function created every render?
→ may change by reference

Only needed inside Effect?
→ move it inside

Pure derived calculation?
→ remove Effect
```

Remember:

> Dependencies should describe the Effect's real reactive inputs, not how often you want it to run.

---

## Next Lesson

➡️ [Lesson 21 — Effect Cleanup and Memory Leaks](./21-effect-cleanup.md)
