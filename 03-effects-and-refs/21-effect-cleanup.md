# Lesson 21 — Effect Cleanup and Memory Leaks ⭐⭐⭐⭐⭐

## 1. Why This Topic Matters

Whenever an Effect starts something outside React, you must ask:

> **What should stop, undo, disconnect, unsubscribe, or cancel this synchronization?**

Examples:

- WebSocket connections
- event listeners
- timers
- subscriptions
- observers
- third-party widgets
- asynchronous requests

A well-designed Effect usually has symmetry:

```text
setup
  ↓
external synchronization active
  ↓
cleanup
  ↓
external synchronization stopped/undone
```

Cleanup is not merely an "unmount callback."

It is part of the lifecycle of the Effect itself.

---

## 2. Basic Cleanup Syntax ⭐⭐⭐⭐⭐

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
    createConnection(roomId);

  connection.connect();

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

The returned function is the Effect's cleanup function.

---

## 3. When Does Cleanup Run? ⭐⭐⭐⭐⭐

Cleanup commonly runs in two important situations:

1. before an Effect re-runs because dependencies changed
2. when the component is removed from the UI

In development Strict Mode, React may also perform an extra setup → cleanup → setup cycle to help detect incorrect Effects.

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
component unmounts
   ↓
cleanup B
```

---

## 4. Cleanup Happens Before the Next Setup ⭐⭐⭐⭐⭐

Suppose:

```jsx
useEffect(() => {
  const connection =
    connect(roomId);

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

Initially:

```text
roomId = "general"
```

React performs:

```text
connect("general")
```

Then:

```text
roomId = "react"
```

The correct sequence is:

```text
disconnect("general")
        ↓
connect("react")
```

React cleans up the previous synchronization before starting the new one.

---

## 5. Cleanup Belongs to the Effect That Created the Resource ⭐⭐⭐⭐⭐

Good:

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

The same Effect:

```text
creates resource
      +
destroys resource
```

This makes the lifecycle easy to understand.

Avoid scattering setup and cleanup across unrelated Effects.

---

# Event Listeners

## 6. Cleaning Up Event Listeners ⭐⭐⭐⭐⭐

Example:

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

Symmetry:

```text
addEventListener
       ↓
removeEventListener
```

Without cleanup, old listeners can remain registered after the component no longer needs them.

---

## 7. The Handler Reference Must Match

This is problematic:

```jsx
useEffect(() => {
  window.addEventListener(
    "resize",
    () => {
      console.log(window.innerWidth);
    }
  );

  return () => {
    window.removeEventListener(
      "resize",
      () => {
        console.log(
          window.innerWidth
        );
      }
    );
  };
}, []);
```

The two arrow functions are different function objects.

```text
setup handler
→ Function A

cleanup handler
→ Function B
```

The browser cannot remove Function A by receiving Function B.

Better:

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

---

# Timers

## 8. Cleaning Up setInterval ⭐⭐⭐⭐⭐

```jsx
useEffect(() => {
  const intervalId =
    setInterval(() => {
      console.log("tick");
    }, 1000);

  return () => {
    clearInterval(intervalId);
  };
}, []);
```

Symmetry:

```text
setInterval
    ↓
clearInterval
```

If the component disappears, its interval should normally stop.

---

## 9. Cleaning Up setTimeout

```jsx
useEffect(() => {
  const timeoutId =
    setTimeout(() => {
      console.log("Finished");
    }, 3000);

  return () => {
    clearTimeout(timeoutId);
  };
}, []);
```

If the component unmounts before the timer fires, cleanup cancels it.

---

## 10. Timer Bug from Missing Cleanup

Bad:

```jsx
useEffect(() => {
  setInterval(() => {
    setCount(
      (count) => count + 1
    );
  }, 1000);
}, []);
```

If the component is mounted/unmounted repeatedly, old timers may continue unnecessarily.

Better:

```jsx
useEffect(() => {
  const id =
    setInterval(() => {
      setCount(
        (count) => count + 1
      );
    }, 1000);

  return () => {
    clearInterval(id);
  };
}, []);
```

---

# Subscriptions and Connections

## 11. Cleaning Up Subscriptions ⭐⭐⭐⭐⭐

Suppose an external service exposes:

```js
subscribe(callback)
unsubscribe(callback)
```

Effect:

```jsx
useEffect(() => {
  function handleMessage(message) {
    setMessage(message);
  }

  service.subscribe(handleMessage);

  return () => {
    service.unsubscribe(
      handleMessage
    );
  };
}, []);
```

Symmetry:

```text
subscribe
   ↓
unsubscribe
```

---

## 12. WebSocket Cleanup ⭐⭐⭐⭐⭐

```jsx
useEffect(() => {
  const socket =
    new WebSocket(url);

  socket.addEventListener(
    "message",
    handleMessage
  );

  return () => {
    socket.removeEventListener(
      "message",
      handleMessage
    );

    socket.close();
  };
}, [url]);
```

The cleanup may need to undo multiple setup operations.

---

## 13. Real-World Chat Example ⭐⭐⭐⭐⭐

```jsx
function ChatRoom({
  roomId,
}) {
  useEffect(() => {
    const socket =
      createChatSocket(roomId);

    socket.connect();

    function handleMessage(message) {
      console.log(message);
    }

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

  return <ChatUI />;
}
```

When the room changes:

```text
Room A socket
     ↓
unsubscribe listener
     ↓
disconnect A
     ↓
connect B
     ↓
subscribe B listener
```

This prevents duplicate connections and duplicate message handlers.

---

# Observers and Third-Party APIs

## 14. Observer Cleanup

Example:

```jsx
useEffect(() => {
  const observer =
    new IntersectionObserver(
      (entries) => {
        console.log(entries);
      }
    );

  const element =
    targetRef.current;

  if (element) {
    observer.observe(element);
  }

  return () => {
    observer.disconnect();
  };
}, []);
```

Other browser APIs can have similar cleanup requirements:

- ResizeObserver
- MutationObserver
- media queries
- geolocation watchers

Always understand the external API's lifecycle.

---

## 15. Third-Party Widget Cleanup

Suppose:

```jsx
useEffect(() => {
  const map =
    createMap(mapRef.current);

  return () => {
    map.destroy();
  };
}, []);
```

If an external library allocates resources or attaches listeners, cleanup should undo them when required.

---

# What Is a Memory Leak?

## 16. Memory Leak Mental Model ⭐⭐⭐⭐⭐

A memory leak generally means resources remain reachable or active even though the application no longer needs them.

Conceptually:

```text
Component needed resource
        ↓
resource created
        ↓
component no longer needs it
        ↓
resource remains active/referenced
        ↓
wasted resources / unwanted behavior
```

Examples can include:

- forgotten event listeners
- forgotten subscriptions
- unclosed sockets
- forgotten intervals
- observers that continue observing

---

## 17. Not Every Late State Update Is a Memory Leak ⭐⭐⭐⭐⭐

Older React versions commonly warned about state updates after unmount.

Modern React removed that broad warning because a late state update does not automatically prove a memory leak.

Important distinction:

```text
state update after unmount
        ≠
automatic proof of memory leak
```

React can ignore updates to an unmounted component.

But asynchronous work may still need cancellation or stale-result protection for correctness, bandwidth, or resource usage.

So do not define "memory leak" simply as:

> "setState happened after unmount."

---

# Async Effects and Race Conditions

## 18. The Async Race Condition Problem ⭐⭐⭐⭐⭐

Suppose:

```jsx
useEffect(() => {
  fetchUser(userId)
    .then((user) => {
      setUser(user);
    });
}, [userId]);
```

Imagine:

```text
userId = 1
request A starts
       ↓
userId = 2
request B starts
       ↓
request B finishes
setUser(User 2)
       ↓
request A finishes later
setUser(User 1)
```

Now the UI shows stale data.

This is a **race condition**.

The problem is not only memory.

It is correctness.

---

## 19. Ignore Stale Async Results ⭐⭐⭐⭐⭐

One pattern:

```jsx
useEffect(() => {
  let ignore = false;

  async function loadUser() {
    const user =
      await fetchUser(userId);

    if (!ignore) {
      setUser(user);
    }
  }

  loadUser();

  return () => {
    ignore = true;
  };
}, [userId]);
```

Flow:

```text
Effect A starts
ignore = false

dependency changes
      ↓
cleanup A
ignore = true

old request finishes
      ↓
if (!ignore)
      ↓
false
      ↓
discard stale result
```

This prevents an obsolete request result from updating the current UI.

---

## 20. AbortController ⭐⭐⭐⭐⭐

For APIs that support cancellation, such as `fetch`, use `AbortController`.

```jsx
useEffect(() => {
  const controller =
    new AbortController();

  async function loadUsers() {
    try {
      const response =
        await fetch(
          "/api/users",
          {
            signal:
              controller.signal,
          }
        );

      const data =
        await response.json();

      setUsers(data);
    } catch (error) {
      if (
        error.name ===
        "AbortError"
      ) {
        return;
      }

      throw error;
    }
  }

  loadUsers();

  return () => {
    controller.abort();
  };
}, []);
```

Cleanup:

```text
Effect no longer relevant
        ↓
controller.abort()
        ↓
fetch receives abort signal
```

---

## 21. Abort vs Ignore ⭐⭐⭐⭐⭐

These solve related but different concerns.

### Abort

Attempts to stop supported external work:

```text
request
   ↓
abort
   ↓
network operation cancelled
when supported
```

### Ignore

Protects React from applying an obsolete result:

```text
old request finishes
       ↓
result ignored
```

Depending on the API, you may use cancellation, stale-result protection, or an abstraction that handles both.

---

## 22. Promise Cancellation Is Not Universal

You cannot assume every Promise can be cancelled.

This:

```js
const promise = doSomethingAsync();
```

does not automatically provide:

```js
promise.cancel();
```

Cancellation depends on the external API.

For `fetch`:

```text
AbortController
```

For other libraries:

```text
use their supported cancellation/unsubscribe API
```

---

## 23. Do Not Make the Effect Callback async

Wrong:

```jsx
useEffect(async () => {
  // ...
}, []);
```

An async function returns a Promise.

React expects:

```text
nothing
OR
cleanup function
```

Correct:

```jsx
useEffect(() => {
  async function load() {
    // ...
  }

  load();

  return () => {
    // cleanup if needed
  };
}, []);
```

---

# Strict Mode

## 24. Strict Mode Exposes Missing Cleanup ⭐⭐⭐⭐⭐

Suppose:

```jsx
useEffect(() => {
  connection.connect();
}, []);
```

In development Strict Mode you may notice repeated setup.

The temptation is to blame React.

But the real question is:

> If this Effect starts a connection, where is the disconnect?

Correct:

```jsx
useEffect(() => {
  connection.connect();

  return () => {
    connection.disconnect();
  };
}, []);
```

Strict Mode helps reveal missing lifecycle symmetry.

---

## 25. Development Setup → Cleanup → Setup ⭐⭐⭐⭐⭐

Think:

```text
Development check

setup
  ↓
cleanup
  ↓
setup
```

Your user should not be able to distinguish this from one correct active synchronization.

For example:

```text
connect
disconnect
connect
```

is fine if only one connection remains active.

But:

```text
connect
connect
```

without disconnect indicates a problem.

---

## 26. Do Not Use a Ref to Hide Missing Cleanup

Bad workaround:

```jsx
const connected =
  useRef(false);

useEffect(() => {
  if (connected.current) {
    return;
  }

  connected.current = true;

  connect();
}, []);
```

This prevents the development check from exposing the actual resource lifecycle.

If `connect()` requires cleanup:

```jsx
useEffect(() => {
  const connection =
    connect();

  return () => {
    connection.disconnect();
  };
}, []);
```

Fix the synchronization process.

---

# Cleanup and Dependencies

## 27. Cleanup Uses Values from Its Own Render ⭐⭐⭐⭐⭐

Consider:

```jsx
useEffect(() => {
  const connection =
    connect(roomId);

  return () => {
    console.log(
      "Disconnecting",
      roomId
    );

    connection.disconnect();
  };
}, [roomId]);
```

If this Effect was created when:

```text
roomId = "general"
```

its cleanup belongs to that synchronization.

When `roomId` becomes `react`:

```text
cleanup old Effect
roomId = general
       ↓
setup new Effect
roomId = react
```

This is closures working correctly.

---

## 28. Cleanup Should Undo Setup ⭐⭐⭐⭐⭐

Good symmetry examples:

```text
connect      ↔ disconnect
subscribe    ↔ unsubscribe
add listener ↔ remove listener
start timer  ↔ clear timer
observe      ↔ unobserve/disconnect
start request↔ abort/ignore when appropriate
```

If cleanup exists without corresponding setup, or setup has no matching cleanup despite needing one, inspect the design.

---

## 29. Cleanup Is Not Always Required

Not every Effect requires cleanup.

Example:

```jsx
useEffect(() => {
  document.title =
    `Profile: ${name}`;
}, [name]);
```

If there is nothing meaningful to undo, no cleanup is required.

Do not add empty cleanup functions mechanically.

---

## 30. Example Where Browser Cleanup May Be Automatic

Suppose an Effect calls a method on a DOM node and no persistent subscription/resource is created.

There may be nothing to clean up.

The rule is not:

> Every Effect must return cleanup.

The rule is:

> If setup creates an ongoing external synchronization or resource that must be undone, provide the corresponding cleanup.

---

# Common Bugs

## 31. Duplicate Event Listeners ⭐⭐⭐⭐⭐

Bad:

```jsx
useEffect(() => {
  window.addEventListener(
    "scroll",
    handleScroll
  );
});
```

No dependency array means the Effect runs after every render.

No cleanup means listeners accumulate.

Potential result:

```text
Render 1 → listener 1
Render 2 → listener 2
Render 3 → listener 3
...
```

One scroll can invoke multiple stale handlers.

Correct dependency and cleanup design prevents this.

---

## 32. Duplicate Socket Listeners

Bad:

```jsx
useEffect(() => {
  socket.on(
    "message",
    handleMessage
  );
}, [messages]);
```

If `messages` changes for every incoming message, the Effect may subscribe repeatedly.

If cleanup is missing:

```text
message
  ↓
state update
  ↓
render
  ↓
new subscription
  ↓
future message handled multiple times
```

Always align dependencies and cleanup with the actual subscription lifecycle.

---

## 33. Stale Timers

Suppose:

```jsx
useEffect(() => {
  const id =
    setInterval(() => {
      console.log(count);
    }, 1000);

  return () => {
    clearInterval(id);
  };
}, []);
```

The cleanup is correct, but the callback may still read the initial `count` because of its closure.

This is not a cleanup problem.

It is a stale closure/dependency problem.

Lesson 23 covers this deeply.

Important distinction:

```text
resource lifecycle bug
        ≠
stale closure bug
```

---

## 34. Cleanup Should Not Cause Unrelated State Changes

Be careful with:

```jsx
return () => {
  setSomething(...);
};
```

Cleanup's main job is usually to stop or undo the synchronization created by the Effect.

If cleanup is driving unrelated application state, reconsider the design.

Especially on unmount, updating the state of the component being removed is usually meaningless.

---

# Real-World Patterns

## 35. CodeBuddy Socket Room Example ⭐⭐⭐⭐⭐

```jsx
function Conversation({
  conversationId,
}) {
  useEffect(() => {
    const socket =
      getSocket();

    socket.emit(
      "join-room",
      conversationId
    );

    function handleMessage(message) {
      console.log(message);
    }

    socket.on(
      "new-message",
      handleMessage
    );

    return () => {
      socket.off(
        "new-message",
        handleMessage
      );

      socket.emit(
        "leave-room",
        conversationId
      );
    };
  }, [conversationId]);

  return <ChatMessages />;
}
```

Lifecycle:

```text
Conversation A
      ↓
join room A
subscribe
      ↓
switch to B
      ↓
unsubscribe A
leave A
      ↓
join B
subscribe B
```

This prevents stale room listeners from accumulating.

---

## 36. Search Request Race Example ⭐⭐⭐⭐⭐

```jsx
function SearchResults({
  query,
}) {
  const [results, setResults] =
    useState([]);

  useEffect(() => {
    const controller =
      new AbortController();

    async function search() {
      try {
        const response =
          await fetch(
            `/api/search?q=${encodeURIComponent(
              query
            )}`,
            {
              signal:
                controller.signal,
            }
          );

        const data =
          await response.json();

        setResults(data);
      } catch (error) {
        if (
          error.name !==
          "AbortError"
        ) {
          console.error(error);
        }
      }
    }

    search();

    return () => {
      controller.abort();
    };
  }, [query]);

  // ...
}
```

When query changes quickly:

```text
query = react
request A
     ↓
query = next
cleanup A → abort
request B
     ↓
only relevant request should drive UI
```

In production applications, dedicated data-fetching libraries or framework APIs may handle this more comprehensively.

---

## 37. Cleanup Checklist ⭐⭐⭐⭐⭐

Whenever you write an Effect, ask:

```text
Did I...
├── add an event listener?
├── start a timer?
├── subscribe?
├── connect?
├── observe?
├── create a third-party instance?
├── start async work?
└── allocate an external resource?
```

Then ask:

```text
What is the corresponding:
├── remove?
├── clear?
├── unsubscribe?
├── disconnect?
├── unobserve?
├── destroy?
├── abort?
└── stale-result protection?
```

---

## 38. Common Mistakes ⭐⭐⭐⭐⭐

### Mistake 1 — Thinking cleanup only runs on unmount

Cleanup also runs before re-synchronization when dependencies change.

### Mistake 2 — Forgetting to remove event listeners

Persistent listeners can cause unwanted callbacks and retain references.

### Mistake 3 — Forgetting to clear timers

Intervals or timeouts can continue after they are no longer useful.

### Mistake 4 — Forgetting to unsubscribe

Subscriptions can continue delivering events.

### Mistake 5 — Opening new sockets without closing old ones

This can cause duplicate connections and duplicate messages.

### Mistake 6 — Treating every late state update as a memory leak

A late update alone does not prove leaked memory.

### Mistake 7 — Ignoring async race conditions

An older request can overwrite newer data.

### Mistake 8 — Assuming every Promise is cancellable

Cancellation depends on the API.

### Mistake 9 — Using Strict Mode suppression instead of proper cleanup

Fix lifecycle symmetry.

### Mistake 10 — Returning cleanup when there is nothing to clean

Cleanup should have a purpose.

---

## 39. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is useEffect cleanup?

**Answer:** A function returned from an Effect that stops or undoes the synchronization/resource created by that Effect.

### Q2. When does cleanup run?

**Answer:** Before the Effect re-runs with changed dependencies and when the component unmounts. Development Strict Mode may also run an extra setup/cleanup cycle.

### Q3. Why should event listeners be removed?

**Answer:** To prevent obsolete handlers from remaining registered, firing unexpectedly, or retaining unnecessary references.

### Q4. Why should intervals be cleared?

**Answer:** To stop work that is no longer needed and prevent duplicate timers across synchronization lifecycles.

### Q5. Is a state update after unmount always a memory leak?

**Answer:** No. It does not automatically prove leaked memory. However, stale async work can still cause correctness or resource-usage problems.

### Q6. What is a race condition in data fetching?

**Answer:** Multiple requests can finish in a different order than they started, allowing an older result to overwrite newer state.

### Q7. How can you handle stale fetch requests?

**Answer:** Abort obsolete requests with `AbortController` when supported and/or ignore results from synchronization that is no longer current.

### Q8. Why must removeEventListener receive the same function reference?

**Answer:** The browser identifies the registered listener by its function reference; a newly created function is a different listener.

### Q9. Does every Effect need cleanup?

**Answer:** No. Cleanup is needed when the Effect creates something ongoing that must be stopped or undone.

### Q10. What does Strict Mode teach about cleanup?

**Answer:** An Effect should remain correct through setup → cleanup → setup. The user should not observe broken behavior from that lifecycle.

### Q11. Should cleanup be separated into another Effect?

**Answer:** Usually no. Setup and its corresponding cleanup should live together so the synchronization lifecycle is clear.

### Q12. What is the best cleanup mental model?

**Answer:** Cleanup should mirror setup: whatever the Effect starts, registers, subscribes to, or connects should be appropriately stopped, removed, unsubscribed, or disconnected.

---

## 40. Interview Scenario ⭐⭐⭐⭐⭐

What is wrong here?

```jsx
useEffect(() => {
  socket.on(
    "message",
    handleMessage
  );
}, [roomId]);
```

Every time `roomId` changes, another listener can be registered.

Correct:

```jsx
useEffect(() => {
  socket.on(
    "message",
    handleMessage
  );

  return () => {
    socket.off(
      "message",
      handleMessage
    );
  };
}, [roomId]);
```

Now:

```text
old listener removed
       ↓
new listener registered
```

---

## 41. Another Interview Scenario ⭐⭐⭐⭐⭐

Requests:

```text
Request A → user 1
Request B → user 2
```

B finishes first:

```text
UI = user 2
```

Then A finishes:

```text
UI = user 1
```

This is not primarily a rendering problem.

It is an asynchronous race.

Solution:

```text
obsolete Effect
     ↓
abort request
and/or
ignore stale result
```

---

## 42. Complete Mental Model ⭐⭐⭐⭐⭐

```text
EFFECT SETUP
     │
     ├── listener
     ├── timer
     ├── subscription
     ├── connection
     ├── observer
     └── async work
     │
     ↓
synchronization active
     │
     ├───────────────┐
     │               │
dependency changes   component unmounts
     │               │
     └───────┬───────┘
             ↓
          CLEANUP
             │
     ┌───────┼────────┐
     ↓       ↓        ↓
   remove   clear   disconnect
     ↓       ↓        ↓
unsubscribe / abort / ignore
             │
             ↓
   old synchronization gone
```

---

## 43. Quick Revision

Remember the pairs:

```text
addEventListener → removeEventListener

setInterval      → clearInterval

setTimeout       → clearTimeout

subscribe        → unsubscribe

connect          → disconnect

observe          → disconnect/unobserve

fetch            → AbortController
                   and/or stale-result protection
```

And remember:

```text
cleanup
≠
only unmount

cleanup
=
stop previous synchronization
before it is no longer valid
```

---

## 44. Key Takeaways

- Effect cleanup stops or undoes synchronization created by an Effect.
- Cleanup runs before re-synchronization when dependencies change.
- Cleanup also runs when the component is removed.
- Development Strict Mode may intentionally test setup → cleanup → setup.
- Setup and cleanup should usually be symmetrical.
- Keep a resource's setup and cleanup in the same Effect.
- Remove event listeners using the same handler reference.
- Clear timers when they are no longer needed.
- Unsubscribe from subscriptions and disconnect sockets.
- Clean up observers and third-party resources according to their APIs.
- Not every Effect requires cleanup.
- A late state update does not automatically prove a memory leak.
- Async requests can create race conditions even when memory is not leaking.
- `AbortController` can cancel supported requests such as `fetch`.
- Ignoring stale results is another important correctness technique.
- Not every Promise supports cancellation.
- Do not hide Strict Mode checks with refs instead of implementing proper cleanup.
- Cleanup problems and stale-closure problems are different.
- The central question is: **What did this Effect start, and how do I correctly stop or undo it?**

---

## Next Lesson

➡️ [Lesson 22 — You Might Not Need an Effect ⭐⭐⭐⭐⭐](./22-you-might-not-need-effect.md)
