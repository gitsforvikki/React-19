# Lesson 19 — useEffect Fundamentals

`useEffect` is used to **synchronize a React component with something outside React**.

Good examples:

- browser APIs
- timers
- event listeners
- subscriptions
- sockets
- third-party libraries

Before using an Effect, ask:

> Is this logic synchronizing with an external system?

If not, you may not need `useEffect`.

---

## 1. Basic Syntax

```jsx
useEffect(() => {
  // setup

  return () => {
    // optional cleanup
  };
}, [dependencies]);
```

An Effect has:

```text
setup
+
optional cleanup
+
dependencies
```

---

## 2. Effects Run After React Commits

Simplified flow:

```text
render
↓
React updates the DOM
↓
Effect runs
```

Do not perform side effects directly during render.

Bad:

```jsx
function ChatRoom({ roomId }) {
  connectToRoom(roomId);

  return <h1>{roomId}</h1>;
}
```

Better:

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection =
      connectToRoom(roomId);

    return () => {
      connection.disconnect();
    };
  }, [roomId]);

  return <h1>{roomId}</h1>;
}
```

---

## 3. Event Handler vs Effect

Use an **event handler** when something happens because the user did something.

```jsx
function handleSubmit() {
  sendMessage(message);
}
```

Use an **Effect** when the component must stay synchronized with an external system.

```jsx
useEffect(() => {
  const connection =
    connect(roomId);

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

Mental model:

```text
user action
→ event handler

component state/props require external sync
→ useEffect
```

---

## 4. Cleanup

Many Effects need cleanup.

Example:

```jsx
useEffect(() => {
  const timerId =
    setInterval(() => {
      console.log("tick");
    }, 1000);

  return () => {
    clearInterval(timerId);
  };
}, []);
```

Cleanup runs when:

- the component unmounts
- dependencies change before the Effect runs again

---

## 5. Setup → Cleanup → Setup

Suppose:

```text
roomId = "general"
```

Then it changes to:

```text
roomId = "react"
```

React conceptually does:

```text
connect general
↓
roomId changes
↓
disconnect general
↓
connect react
```

This is the correct mental model for Effects.

---

## 6. Common External-System Examples

### Window event listener

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

### Socket connection

```jsx
useEffect(() => {
  const socket =
    createSocket(roomId);

  socket.connect();

  return () => {
    socket.disconnect();
  };
}, [roomId]);
```

---

## 7. Do Not Use Effects for Derived Values

Bad:

```jsx
const [fullName, setFullName] =
  useState("");

useEffect(() => {
  setFullName(
    firstName + " " + lastName
  );
}, [firstName, lastName]);
```

Better:

```jsx
const fullName =
  firstName + " " + lastName;
```

If you can calculate something during render, do that instead.

---

## 8. Do Not Use Effects for User Actions

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

The button click is the real cause, so keep the logic in the event handler.

---

## 9. Data Fetching

Client-side fetching in an Effect can be valid:

```jsx
useEffect(() => {
  async function loadUsers() {
    const response =
      await fetch("/api/users");

    const data =
      await response.json();

    setUsers(data);
  }

  loadUsers();
}, []);
```

Do not make the Effect callback itself `async`.

Avoid:

```jsx
useEffect(async () => {
  // ...
}, []);
```

because an Effect callback must return:

```text
nothing
OR
cleanup function
```

not a Promise.

---

## 10. Strict Mode in Development

In development, Strict Mode may run:

```text
setup
↓
cleanup
↓
setup
```

This is intentional.

Do not try to hide it with a ref.

Instead make sure your Effect has correct cleanup.

---

## 11. Infinite Loop Mistake

Bad:

```jsx
useEffect(() => {
  setCount(count + 1);
}, [count]);
```

Flow:

```text
count changes
↓
Effect runs
↓
setCount
↓
count changes
↓
Effect runs again
↓
...
```

If an Effect updates one of its own dependencies, inspect the design carefully.

Often the Effect is unnecessary.

---

## Common Mistakes

### Mistake 1 — Using useEffect for every calculation

Derived values usually belong in render.

### Mistake 2 — Using an Effect for button-click logic

Use the event handler.

### Mistake 3 — Forgetting cleanup

Timers, listeners, subscriptions, and connections often need cleanup.

### Mistake 4 — Making the Effect callback async

Create an async function inside the Effect instead.

### Mistake 5 — Treating `[]` as “run once no matter what”

Dependencies should reflect the reactive values used by the Effect.

---

## Interview Questions

### What is useEffect?

A Hook for synchronizing a component with external systems after React commits the UI.

### When should you use it?

When something outside React needs to stay synchronized with props or state.

### When should you not use it?

For pure calculations or logic caused directly by a user event.

### What is cleanup?

A function returned from the Effect that stops the previous synchronization.

### Why should the Effect callback not be async?

Because an async function returns a Promise, but React expects either nothing or a cleanup function.

---

## Quick Revision

```text
Need calculation?
→ do it during render

Need response to user action?
→ event handler

Need external synchronization?
→ useEffect
```

Remember:

1. Effects run after commit
2. Effects can return cleanup
3. Dependencies control re-synchronization
4. Do not use Effects when ordinary render logic is enough

---

## Next Lesson

➡️ [Lesson 20 — Dependency Arrays and Reactive Dependencies](./20-effect-dependencies.md)
