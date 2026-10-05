# Lesson 19 — useEffect Fundamentals ⭐⭐⭐⭐⭐

## 1. Why This Topic Matters

`useEffect` is one of the most commonly misunderstood React Hooks.

Many developers learn it as:

> "Run some code after render."

That is incomplete.

The better mental model is:

> **An Effect lets a React component synchronize with an external system.**

Examples of external systems include:

- network connections
- browser APIs
- timers
- DOM APIs not managed declaratively by React
- subscriptions
- third-party libraries
- analytics or other external services

The first question should therefore be:

> **Does this logic actually need to synchronize React with something outside React?**

If not, you may not need an Effect at all.

Lesson 22 covers that decision in depth.

---

## 2. Basic Syntax

```jsx
import { useEffect } from "react";

function Component() {
  useEffect(() => {
    // synchronization logic
  }, []);

  return <div>Hello</div>;
}
```

General shape:

```jsx
useEffect(() => {
  // setup

  return () => {
    // optional cleanup
  };
}, [dependencies]);
```

There are three important pieces:

```text
Effect
├── setup function
├── optional cleanup function
└── dependency list
```

---

## 3. Rendering vs Effects ⭐⭐⭐⭐⭐

React components should calculate UI during rendering.

Example:

```jsx
function Greeting({ name }) {
  return <h1>Hello {name}</h1>;
}
```

Rendering should remain pure:

```text
props + state
     ↓
render
     ↓
JSX
```

An Effect is different.

It runs because the rendered component needs to synchronize with something external.

```text
Render
  ↓
React updates/commits UI
  ↓
Effect setup runs
  ↓
external system synchronized
```

---

## 4. A Simple Effect Example

Suppose you want the browser document title to reflect a count:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  return (
    <button
      onClick={() =>
        setCount((count) => count + 1)
      }
    >
      {count}
    </button>
  );
}
```

Here:

```text
React state
count
  ↓
render
  ↓
commit
  ↓
Effect
  ↓
browser document.title
```

The browser document API is outside React's normal JSX rendering.

---

## 5. Effects Run After React Commits ⭐⭐⭐⭐⭐

A simplified lifecycle is:

```text
Trigger update
     ↓
Render phase
     ↓
React calculates next UI
     ↓
Commit phase
     ↓
DOM updated
     ↓
Effect runs
```

This is why Effects are appropriate for synchronization that should happen after React has committed the UI.

Do not perform side effects directly during rendering.

---

## 6. Why Side Effects Should Not Run During Render ⭐⭐⭐⭐⭐

Bad:

```jsx
function ChatRoom({ roomId }) {
  connectToRoom(roomId);

  return <h1>{roomId}</h1>;
}
```

Rendering should be pure.

React may render components multiple times, pause work, restart work, or discard render work.

External actions should not happen merely because React evaluated a component function.

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

## 7. Event Handler vs Effect ⭐⭐⭐⭐⭐

This distinction is extremely important.

### Event handler

Use an event handler when logic happens because the user performed a specific interaction.

```jsx
function handleSubmit() {
  sendMessage(message);
}
```

### Effect

Use an Effect when logic happens because the component is currently rendered with particular state/props and must stay synchronized with an external system.

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

Mental model:

```text
User did something
      ↓
event handler

Component is present with these values
      ↓
Effect synchronization
```

---

## 8. Effects Are Not General-Purpose "After Render" Code ⭐⭐⭐⭐⭐

Avoid thinking:

```text
Need to run code?
     ↓
put it in useEffect
```

Instead:

```text
Can this be calculated during render?
     ↓ yes
calculate it during render

Is it caused by a user interaction?
     ↓ yes
use an event handler

Does React need to synchronize with an external system?
     ↓ yes
use an Effect
```

This decision model prevents many unnecessary Effects.

---

## 9. Effect Without a Dependency Array

```jsx
useEffect(() => {
  console.log("Effect");
});
```

There is no dependency array.

The Effect is scheduled after every committed render where this component participates.

Conceptually:

```text
render
  ↓
Effect

render
  ↓
Effect

render
  ↓
Effect
```

This is rarely what you want for synchronization unless the Effect genuinely depends on every render.

---

## 10. Effect with an Empty Dependency Array

```jsx
useEffect(() => {
  console.log("setup");

  return () => {
    console.log("cleanup");
  };
}, []);
```

The empty list means the Effect has no reactive values from the component that require re-synchronization.

In production, this generally corresponds to setup while mounted and cleanup when unmounted.

However, development behavior under Strict Mode needs special attention, which we cover below.

---

## 11. Effect with Dependencies ⭐⭐⭐⭐⭐

```jsx
useEffect(() => {
  document.title =
    `Search: ${query}`;
}, [query]);
```

The Effect depends on:

```text
query
```

When `query` changes, React can re-run the Effect so the external system stays synchronized.

Simplified:

```text
query changes
    ↓
render
    ↓
commit
    ↓
Effect re-synchronizes
```

Lesson 20 covers dependency arrays deeply.

---

## 12. Setup and Cleanup ⭐⭐⭐⭐⭐

Many external systems need both:

```text
setup
+
cleanup
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

The returned function is the cleanup function.

Mental model:

```text
setup
  ↓
external connection active
  ↓
dependency changes / unmount
  ↓
cleanup
```

Cleanup is covered deeply in Lesson 21.

---

## 13. Cleanup Happens Before Re-synchronization ⭐⭐⭐⭐⭐

Suppose:

```text
roomId = "general"
```

Effect connects to `general`.

Then:

```text
roomId = "react"
```

React should not simply create another connection and leave the old one alive.

Conceptually:

```text
Effect setup
connect general

roomId changes
      ↓
cleanup old Effect
disconnect general
      ↓
setup new Effect
connect react
```

This setup → cleanup → setup model is essential.

---

## 14. Component Unmounting

When a component leaves the tree:

```text
component mounted
      ↓
Effect setup active
      ↓
component removed
      ↓
Effect cleanup
```

Examples requiring cleanup:

- WebSocket connection
- event listener
- timer
- subscription
- observer
- some asynchronous synchronization workflows

---

## 15. Example: Timer

```jsx
function Clock() {
  const [time, setTime] =
    useState(() => new Date());

  useEffect(() => {
    const timerId = setInterval(() => {
      setTime(new Date());
    }, 1000);

    return () => {
      clearInterval(timerId);
    };
  }, []);

  return (
    <time>
      {time.toLocaleTimeString()}
    </time>
  );
}
```

Flow:

```text
Clock mounts
    ↓
create interval
    ↓
interval updates state
    ↓
Clock renders new time
    ↓
Clock unmounts
    ↓
clear interval
```

Without cleanup, the external timer can outlive the component.

---

## 16. Example: Window Event Listener

```jsx
function WindowWidth() {
  const [width, setWidth] =
    useState(window.innerWidth);

  useEffect(() => {
    function handleResize() {
      setWidth(window.innerWidth);
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

  return <p>{width}px</p>;
}
```

React synchronizes component state with a browser event source.

---

## 17. Example: External Connection ⭐⭐⭐⭐⭐

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

    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);

  return <h1>Room: {roomId}</h1>;
}
```

Dependencies represent values used by the Effect that can change across renders.

---

## 18. Effect Lifecycle Is Different from Component Lifecycle Thinking ⭐⭐⭐⭐⭐

A legacy mental model asks:

```text
Did component mount?
Did component update?
Will component unmount?
```

A better Hooks mental model is:

> Think about the lifecycle of each Effect independently.

For a chat connection:

```text
Start synchronization
        ↓
Stop synchronization
        ↓
Start again with new values
```

This scales better than trying to imitate class lifecycle methods.

---

## 19. Each Effect Should Represent a Synchronization Process ⭐⭐⭐⭐⭐

Suppose a component:

1. connects to chat
2. tracks page analytics

Avoid mixing unrelated processes unnecessarily:

```jsx
useEffect(() => {
  const connection =
    createConnection(roomId);

  connection.connect();

  logPageView(page);

  return () => {
    connection.disconnect();
  };
}, [roomId, page]);
```

Now changing `page` reconnects chat even if `roomId` did not change.

Better:

```jsx
useEffect(() => {
  const connection =
    createConnection(roomId);

  connection.connect();

  return () => {
    connection.disconnect();
  };
}, [roomId]);

useEffect(() => {
  logPageView(page);
}, [page]);
```

Separate unrelated synchronization processes.

---

## 20. Effects and Data Fetching

You may see:

```jsx
useEffect(() => {
  async function loadUsers() {
    const response =
      await fetch("/api/users");

    const users =
      await response.json();

    setUsers(users);
  }

  loadUsers();
}, []);
```

This can work in client-side React.

However, data fetching introduces concerns such as:

- race conditions
- cancellation/ignoring stale responses
- loading states
- error states
- caching
- duplicate requests
- framework-specific data fetching

These topics are covered later.

Important:

> `useEffect` is not automatically the best data-fetching architecture for every React application.

---

## 21. Why the Effect Callback Itself Should Not Be async ⭐⭐⭐⭐⭐

Avoid:

```jsx
useEffect(async () => {
  const response =
    await fetch("/api/users");
}, []);
```

An Effect callback may return:

```text
nothing
OR
a cleanup function
```

An `async` function returns a Promise.

Instead:

```jsx
useEffect(() => {
  async function loadUsers() {
    const response =
      await fetch("/api/users");

    // ...
  }

  loadUsers();
}, []);
```

Or use an appropriate data-fetching abstraction.

---

## 22. Strict Mode and Effects in Development ⭐⭐⭐⭐⭐

In development, React Strict Mode may perform an extra setup → cleanup → setup cycle for Effects.

You may observe:

```text
connect
disconnect
connect
```

This is intentional development behavior.

It helps reveal Effects that do not clean up correctly.

Your Effect should be designed so:

```text
setup
cleanup
setup
```

produces behavior equivalent to one correct active synchronization.

---

## 23. Do Not "Fix" Strict Mode by Blocking the Effect ⭐⭐⭐⭐⭐

A tempting workaround:

```jsx
const didRun = useRef(false);

useEffect(() => {
  if (didRun.current) {
    return;
  }

  didRun.current = true;

  connect();
}, []);
```

This can hide the real bug.

If the external connection requires cleanup, implement cleanup:

```jsx
useEffect(() => {
  const connection = connect();

  return () => {
    connection.disconnect();
  };
}, []);
```

The goal is not:

> Make the Effect execute once in development.

The goal is:

> Make the Effect correct when setup and cleanup happen as required.

---

## 24. Effects Run Only on the Client

Effects do not run during server rendering.

They run in the browser after the component is committed on the client.

Therefore an Effect should not be required to calculate the initial server-rendered JSX.

This is another reason derived UI values belong in rendering rather than Effects.

---

## 25. Effect Dependencies Are Not a Scheduling Preference ⭐⭐⭐⭐⭐

A common misconception:

```jsx
useEffect(() => {
  // ...
}, []);
```

means:

> "I want this to run once."

A better interpretation:

> "This Effect does not use changing reactive values that require re-synchronization."

Dependencies should describe the Effect's actual reactive inputs.

Do not omit dependencies merely to control execution frequency.

Lesson 20 focuses entirely on this.

---

## 26. Do Not Use Effects for Derived State ⭐⭐⭐⭐⭐

Bad:

```jsx
function FullName({
  firstName,
  lastName,
}) {
  const [fullName, setFullName] =
    useState("");

  useEffect(() => {
    setFullName(
      `${firstName} ${lastName}`
    );
  }, [firstName, lastName]);

  return <p>{fullName}</p>;
}
```

There is no external system.

Better:

```jsx
function FullName({
  firstName,
  lastName,
}) {
  const fullName =
    `${firstName} ${lastName}`;

  return <p>{fullName}</p>;
}
```

Flow:

```text
props
  ↓
render calculation
  ↓
UI
```

No Effect needed.

---

## 27. Do Not Use Effects for User Actions ⭐⭐⭐⭐⭐

Bad:

```jsx
useEffect(() => {
  if (submitted) {
    sendOrder();
  }
}, [submitted]);
```

if the actual reason to send the order is a button click.

Better:

```jsx
function handleSubmit() {
  sendOrder();
}
```

Reason:

```text
user clicked Submit
       ↓
event handler
```

The event contains the causal information.

---

## 28. Effects Can Update State, But Be Careful

An Effect may need to update state when synchronizing with an external source.

Example:

```jsx
useEffect(() => {
  function handleOnline() {
    setOnline(true);
  }

  function handleOffline() {
    setOnline(false);
  }

  window.addEventListener(
    "online",
    handleOnline
  );

  window.addEventListener(
    "offline",
    handleOffline
  );

  return () => {
    window.removeEventListener(
      "online",
      handleOnline
    );

    window.removeEventListener(
      "offline",
      handleOffline
    );
  };
}, []);
```

But unnecessary state updates inside Effects can create extra render cycles.

Always ask whether the value could be calculated directly instead.

---

## 29. Infinite Effect Loop ⭐⭐⭐⭐⭐

Example:

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  setCount(count + 1);
}, [count]);
```

Flow:

```text
count changes
   ↓
render
   ↓
Effect
   ↓
setCount
   ↓
count changes
   ↓
render
   ↓
Effect
   ↓
...
```

An Effect that updates one of its own dependencies can create a loop.

The correct fix is not automatically removing the dependency.

Ask:

> Why does this Effect exist?

Often the Effect itself is unnecessary.

---

## 30. Real-World Example: Online Status

```jsx
function OnlineStatus() {
  const [isOnline, setIsOnline] =
    useState(navigator.onLine);

  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }

    function handleOffline() {
      setIsOnline(false);
    }

    window.addEventListener(
      "online",
      handleOnline
    );

    window.addEventListener(
      "offline",
      handleOffline
    );

    return () => {
      window.removeEventListener(
        "online",
        handleOnline
      );

      window.removeEventListener(
        "offline",
        handleOffline
      );
    };
  }, []);

  return (
    <p>
      {isOnline ? "Online" : "Offline"}
    </p>
  );
}
```

This is genuine synchronization with a browser API.

---

## 31. Real-World Example: CodeBuddy Chat Room

Conceptually:

```jsx
function ChatRoom({
  userId,
  connectionId,
}) {
  useEffect(() => {
    const socket =
      createChatConnection({
        userId,
        connectionId,
      });

    socket.connect();

    return () => {
      socket.disconnect();
    };
  }, [userId, connectionId]);

  return <ChatUI />;
}
```

The Effect's job is:

```text
Rendered chat identity
        ↓
synchronize socket connection
        ↓
cleanup when identity changes
```

This is an excellent use case for an Effect.

---

## 32. Rules of Hooks Still Apply

`useEffect` is a Hook.

Do not call it conditionally:

```jsx
if (loggedIn) {
  useEffect(() => {
    // ...
  }, []);
}
```

Instead call Hooks at the top level and put conditions inside when appropriate:

```jsx
useEffect(() => {
  if (!loggedIn) {
    return;
  }

  // synchronization
}, [loggedIn]);
```

Rules of Hooks are covered deeply later.

---

## 33. useEffect vs useLayoutEffect

For most Effects:

```text
useEffect
```

is the correct default.

`useLayoutEffect` is a specialized Hook for cases where code must run around layout measurement before the browser paints the updated screen.

Do not replace `useEffect` with `useLayoutEffect` casually.

Use `useEffect` unless you have a specific visual/layout requirement.

---

## 34. Common Mistakes ⭐⭐⭐⭐⭐

### Mistake 1 — Using Effects for every calculation

Derived values usually belong in render.

### Mistake 2 — Using an Effect for a button action

User-caused logic usually belongs in the event handler.

### Mistake 3 — Performing side effects during render

Render must remain pure.

### Mistake 4 — Forgetting cleanup

Subscriptions, timers, listeners, and connections often require cleanup.

### Mistake 5 — Treating [] as "run once" regardless of used values

Dependencies describe reactive inputs.

### Mistake 6 — Making the Effect callback async

An Effect must return either nothing or cleanup, not a Promise.

### Mistake 7 — Hiding Strict Mode behavior with a ref

Fix cleanup rather than suppressing development checks.

### Mistake 8 — Creating a state-update loop

Effects that update their own dependencies require careful design.

### Mistake 9 — Combining unrelated synchronization logic

Separate Effects by synchronization purpose.

---

## 35. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is useEffect?

**Answer:** `useEffect` lets a component synchronize with external systems after React commits its UI.

### Q2. When should you use an Effect?

**Answer:** When rendering a component requires synchronization with something outside React, such as a connection, subscription, browser API, timer, or third-party system.

### Q3. When should you not use useEffect?

**Answer:** Usually not for values that can be calculated during render or logic caused directly by a user event.

### Q4. When does useEffect run?

**Answer:** Effects run on the client after React commits the relevant render. Re-running depends on the Effect's dependencies.

### Q5. What does an Effect return?

**Answer:** It may return a cleanup function. Otherwise it returns nothing.

### Q6. When does cleanup run?

**Answer:** Before an Effect re-synchronizes with changed dependencies and when the component unmounts. Development Strict Mode may also perform an extra setup/cleanup cycle.

### Q7. Why should rendering be pure?

**Answer:** React may evaluate rendering multiple times or discard render work, so external side effects during render can create inconsistent behavior.

### Q8. Why shouldn't the Effect callback itself be async?

**Answer:** An async function returns a Promise, while React expects an Effect callback to return nothing or a cleanup function.

### Q9. What does [] mean?

**Answer:** It means the Effect has no changing reactive dependencies that require re-synchronization. It should not be treated merely as an execution-frequency trick.

### Q10. Why can an Effect create an infinite loop?

**Answer:** If it updates state that changes one of its dependencies on every run, each Effect can trigger another render and Effect.

### Q11. Does useEffect run during server rendering?

**Answer:** No. Effects run on the client.

### Q12. What is the best mental model for Effects?

**Answer:** Think in terms of starting and stopping synchronization with an external system, rather than imitating class component lifecycle methods.

---

## 36. Interview Decision Problem ⭐⭐⭐⭐⭐

Where should each piece of logic go?

### Calculate full name

```js
firstName + " " + lastName
```

**Answer:** Render calculation.

### Send form when user clicks Submit

**Answer:** Event handler.

### Connect to chat while a room is displayed

**Answer:** Effect.

### Subscribe to browser resize events

**Answer:** Effect.

### Filter a list from query + items

**Answer:** Usually calculate during render.

This decision-making skill is more important than memorizing Effect syntax.

---

## 37. Complete Mental Model ⭐⭐⭐⭐⭐

```text
COMPONENT RENDERS
       │
       ↓
React calculates JSX
       │
       ↓
COMMIT
       │
       ↓
DOM reflects UI
       │
       ↓
EFFECT SETUP
       │
       ↓
external system synchronized
       │
       ├───────────────┐
       │               │
dependency changes   unmount
       │               │
       ↓               ↓
    cleanup          cleanup
       │
       ↓
new setup
```

---

## 38. Quick Revision

```text
Render
→ calculate UI

Event handler
→ respond to a specific user interaction

Effect
→ synchronize with an external system
```

Effect structure:

```jsx
useEffect(() => {
  // setup

  return () => {
    // cleanup
  };
}, [dependencies]);
```

Remember:

```text
setup
  ↓
synchronized
  ↓
cleanup
  ↓
setup again when needed
```

---

## 39. Key Takeaways

- `useEffect` is primarily for synchronizing React with external systems.
- Effects run on the client after React commits the UI.
- Rendering must stay pure.
- User-triggered logic usually belongs in event handlers.
- Derived values usually belong in render rather than Effects.
- Effects may have setup and cleanup logic.
- Cleanup occurs before re-synchronization and on unmount.
- Think about each Effect as an independent synchronization process.
- Dependencies describe the Effect's reactive inputs.
- Do not omit dependencies merely to control execution frequency.
- Do not make the Effect callback itself `async`.
- Strict Mode may intentionally run an extra setup/cleanup cycle in development.
- Correct cleanup makes Effects resilient to remounting and re-synchronization.
- Avoid unnecessary state updates inside Effects.
- An Effect that updates its own dependency can create an infinite loop.
- `useEffect` is not automatically the right data-fetching architecture for every React app.
- The most important question is: **What external system am I synchronizing with?**

---

## Next Lesson

➡️ [Lesson 20 — Dependency Arrays and Reactive Dependencies ⭐⭐⭐⭐⭐](./20-effect-dependencies.md)
