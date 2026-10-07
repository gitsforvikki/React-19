# Lesson 21 — Effect Cleanup and Memory Leaks

Whenever an Effect starts something outside React, ask:

> What should stop, undo, disconnect, unsubscribe, or cancel it?

Typical examples:

- event listeners
- timers
- subscriptions
- sockets
- observers
- async requests

---

## 1. Basic Cleanup Pattern

```jsx
useEffect(() => {
  // setup

  return () => {
    // cleanup
  };
}, [dependencies]);
```

Example:

```jsx
useEffect(() => {
  const connection =
    connect(roomId);

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

---

## 2. When Cleanup Runs

Cleanup runs:

1. before the Effect re-runs because a dependency changed
2. when the component unmounts

Mental model:

```text
setup A
↓
dependency changes
↓
cleanup A
↓
setup B
↓
unmount
↓
cleanup B
```

---

## 3. Event Listener Cleanup

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

Important:

The same function reference used in `addEventListener` must be used in `removeEventListener`.

---

## 4. Timer Cleanup

```jsx
useEffect(() => {
  const id =
    setInterval(() => {
      console.log("tick");
    }, 1000);

  return () => {
    clearInterval(id);
  };
}, []);
```

Common pairs:

```text
setInterval   ↔ clearInterval
setTimeout    ↔ clearTimeout
subscribe     ↔ unsubscribe
connect       ↔ disconnect
add listener  ↔ remove listener
```

---

## 5. Socket / Subscription Cleanup

```jsx
useEffect(() => {
  const socket =
    createSocket(roomId);

  function handleMessage(
    message
  ) {
    console.log(message);
  }

  socket.connect();
  socket.on(
    "message",
    handleMessage
  );

  return () => {
    socket.off(
      "message",
      handleMessage
    );

    socket.disconnect();
  };
}, [roomId]);
```

If the room changes:

```text
disconnect old room
↓
remove old listener
↓
connect new room
```

This prevents duplicate listeners and stale connections.

---

## 6. What Is a Memory Leak?

A memory/resource leak means something remains active or reachable even though the UI no longer needs it.

Examples:

- forgotten listeners
- forgotten intervals
- unclosed sockets
- forgotten subscriptions
- observers that keep running

Important:

> A late state update by itself does not automatically prove a memory leak.

The bigger question is whether unnecessary external work is still active.

---

## 7. Async Race Conditions

Suppose:

```jsx
useEffect(() => {
  fetchUser(userId)
    .then((user) => {
      setUser(user);
    });
}, [userId]);
```

Possible problem:

```text
request A → user 1

request B → user 2

B finishes first
→ show user 2

A finishes later
→ incorrectly show user 1
```

This is a race condition.

---

## 8. Abort Fetch Requests

For `fetch`, use `AbortController`:

```jsx
useEffect(() => {
  const controller =
    new AbortController();

  async function loadUser() {
    try {
      const response =
        await fetch(
          `/api/users/${userId}`,
          {
            signal:
              controller.signal,
          }
        );

      const data =
        await response.json();

      setUser(data);
    } catch (error) {
      if (
        error.name !==
        "AbortError"
      ) {
        console.error(error);
      }
    }
  }

  loadUser();

  return () => {
    controller.abort();
  };
}, [userId]);
```

When `userId` changes, the old request can be aborted.

---

## 9. Ignore Stale Results When Cancellation Is Not Available

```jsx
useEffect(() => {
  let ignore = false;

  async function load() {
    const data =
      await fetchUser(userId);

    if (!ignore) {
      setUser(data);
    }
  }

  load();

  return () => {
    ignore = true;
  };
}, [userId]);
```

This does not stop the request, but it prevents an old result from updating the current UI.

---

## 10. Strict Mode Helps Find Missing Cleanup

In development, React may run:

```text
setup
↓
cleanup
↓
setup
```

This is intentional.

Do not hide it with a ref.

Fix the Effect so setup and cleanup are symmetrical.

---

## 11. Not Every Effect Needs Cleanup

Example:

```jsx
useEffect(() => {
  document.title =
    `Profile: ${name}`;
}, [name]);
```

There is nothing important to undo here.

Rule:

> Add cleanup only when setup creates ongoing work or a resource that must be stopped.

---

## Common Mistakes

### Mistake 1 — Forgetting cleanup

Listeners, timers, subscriptions, and sockets can remain active.

### Mistake 2 — Removing an event listener with a different function reference

The browser cannot remove the original listener.

### Mistake 3 — Ignoring async race conditions

Old requests can overwrite newer data.

### Mistake 4 — Treating every late state update as a memory leak

It may be stale or unnecessary work, but not every case is a memory leak.

### Mistake 5 — Using a ref to hide Strict Mode behavior

Fix the cleanup instead.

---

## Interview Questions

### When does Effect cleanup run?

Before the Effect re-runs because dependencies changed and when the component unmounts.

### Why is cleanup important?

It stops external resources that are no longer needed.

### How do you cancel a fetch request?

With `AbortController`.

### What is a race condition in an Effect?

When an older async operation finishes after a newer one and incorrectly updates the UI.

### Does every Effect need cleanup?

No. Only Effects that create something that should later be stopped or undone.

---

## Quick Revision

```text
setup resource
↓
Effect active
↓
dependency changes / unmount
↓
cleanup resource
```

Remember:

1. cleanup should undo setup
2. clean up timers, listeners, sockets, and subscriptions
3. protect async Effects from stale results
4. use AbortController when the API supports cancellation

---

## Next Lesson

➡️ [Lesson 22 — You Might Not Need an Effect](./22-you-might-not-need-effect.md)
