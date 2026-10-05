# Lesson 23 — Closures, Stale Closures and Effects ⭐⭐⭐⭐⭐

## 1. Why This Topic Matters

Many confusing React bugs are actually JavaScript closure problems.

Examples:

- a timer logs an old state value
- an Effect keeps using an old prop
- an event listener sees stale state
- an async callback uses data from an earlier render
- removing a dependency appears to "fix" repeated Effects but creates stale behavior

To understand these problems, connect three ideas:

```text
JavaScript closures
        +
React render snapshots
        +
Effect dependencies
```

The central rule is:

> **Every render creates its own values and functions. Functions created during that render close over that render's values.**

---

# 2. JavaScript Closure Refresher ⭐⭐⭐⭐⭐

A closure means a function remembers variables from the lexical scope where the function was created.

Example:

```js
function createCounter() {
  let count = 0;

  return function increment() {
    count += 1;
    console.log(count);
  };
}

const increment =
  createCounter();

increment(); // 1
increment(); // 2
```

The returned function still has access to:

```text
count
```

even after `createCounter()` has finished.

That is closure behavior.

---

# 3. Another Closure Example

```js
function createGreeting(name) {
  return function greet() {
    console.log(
      `Hello ${name}`
    );
  };
}

const greetVikash =
  createGreeting("Vikash");

greetVikash();
```

`greet` remembers the `name` from the call that created it.

Conceptually:

```text
createGreeting("Vikash")
        ↓
name = "Vikash"
        ↓
create greet function
        ↓
greet closes over name
```

React uses ordinary JavaScript functions, so the same closure rules apply.

---

# 4. React Creates a New Render Snapshot ⭐⭐⭐⭐⭐

Consider:

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

Each render creates a new `count` value and a new `handleClick` function.

Conceptually:

```text
Render #1
count = 0
handleClick #1 → closes over 0

Render #2
count = 1
handleClick #2 → closes over 1

Render #3
count = 2
handleClick #3 → closes over 2
```

Functions do not magically change the values they captured.

---

# 5. Closure + State as a Snapshot ⭐⭐⭐⭐⭐

Lesson 13 established:

> State behaves like a snapshot for a particular render.

Closures explain how callbacks retain that snapshot.

Example:

```jsx
function Counter() {
  const [count, setCount] =
    useState(0);

  function handleClick() {
    setTimeout(() => {
      console.log(count);
    }, 3000);
  }

  return (
    <>
      <p>{count}</p>

      <button onClick={handleClick}>
        Log later
      </button>

      <button
        onClick={() =>
          setCount(
            (count) => count + 1
          )
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
count = 0
      ↓
click "Log later"
      ↓
timeout callback created
and captures count = 0
      ↓
increment count to 1
      ↓
new render has count = 1
      ↓
old timeout fires
      ↓
logs 0
```

This is expected closure behavior.

---

# 6. Old Value Does Not Automatically Mean a Bug ⭐⭐⭐⭐⭐

If a callback represents an action from an earlier render, using that render's value may be exactly correct.

Example:

```jsx
function OrderButton({
  product,
}) {
  function handleBuy() {
    setTimeout(() => {
      alert(
        `Purchased ${product.name}`
      );
    }, 1000);
  }

  return (
    <button onClick={handleBuy}>
      Buy
    </button>
  );
}
```

If the selected product changes after the click, you may still want the callback to refer to the product that was actually purchased.

So:

```text
captured old value
≠
automatically stale bug
```

A value is problematic only when the logic requires the latest value but receives an older render's value.

---

# 7. What Is a Stale Closure? ⭐⭐⭐⭐⭐

A stale closure occurs when a function keeps using values captured from an older render even though the intended logic requires newer values.

Conceptually:

```text
Render #1
count = 0
   ↓
callback created
captures 0

Render #2
count = 1

Render #3
count = 2

old callback runs
   ↓
uses 0
   ↓
but logic needed latest value 2
```

That mismatch is the problem.

---

# 8. Classic Stale Interval Bug ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const [count, setCount] =
    useState(0);

  useEffect(() => {
    const id =
      setInterval(() => {
        console.log(count);
      }, 1000);

    return () => {
      clearInterval(id);
    };
  }, []);

  return (
    <button
      onClick={() =>
        setCount(
          (count) => count + 1
        )
      }
    >
      {count}
    </button>
  );
}
```

The Effect has:

```text
dependencies = []
```

So its setup uses the values from the render in which that Effect was created.

If initial:

```text
count = 0
```

the interval callback can keep seeing:

```text
count = 0
```

even after later renders.

---

# 9. Why the Interval Is Stale ⭐⭐⭐⭐⭐

Timeline:

```text
Render #1
count = 0
   ↓
Effect setup
   ↓
interval callback
captures count = 0

Render #2
count = 1
   ↓
Effect does not re-run
because dependencies = []

Render #3
count = 2
   ↓
same old interval remains

interval fires
   ↓
logs 0
```

The interval is not reading a magical global state variable.

It is executing a function created by Render #1.

---

# 10. Fix 1 — Include the Reactive Dependency ⭐⭐⭐⭐⭐

If the synchronization should restart whenever `count` changes:

```jsx
useEffect(() => {
  const id =
    setInterval(() => {
      console.log(count);
    }, 1000);

  return () => {
    clearInterval(id);
  };
}, [count]);
```

Now:

```text
count changes
    ↓
cleanup old interval
    ↓
create interval with new count
```

This is correct when the external synchronization genuinely depends on `count`.

But restarting an interval for every count change may not always be the desired design.

---

# 11. Fix 2 — Functional State Update ⭐⭐⭐⭐⭐

Suppose the interval's purpose is to increment state:

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

The callback captured:

```text
count = 0
```

so it repeatedly requests:

```text
setCount(1)
```

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

The updater receives the latest pending state from React.

---

# 12. Why Functional Updates Help ⭐⭐⭐⭐⭐

Compare:

```jsx
setCount(count + 1);
```

This calculates using:

```text
count captured by this closure
```

But:

```jsx
setCount(
  (count) => count + 1
);
```

asks React:

```text
Give this updater
the latest pending state
        ↓
calculate next state
```

This can remove the need to read `count` inside the Effect.

Then `count` no longer needs to be a dependency for that specific update logic.

---

# 13. Functional Updates Do Not Solve Every Stale Closure

Suppose:

```jsx
setMessages((messages) => [
  ...messages,
  newMessage,
]);
```

This solves stale access to:

```text
messages
```

But if `newMessage` itself is captured from an old render and needs to be current, that remains a separate issue.

Functional updaters solve:

```text
next state depends on previous state
```

They are not a universal stale-closure fix.

---

# 14. Stale Closures and Missing Effect Dependencies ⭐⭐⭐⭐⭐

Bad:

```jsx
function ChatRoom({
  roomId,
}) {
  useEffect(() => {
    const connection =
      connect(roomId);

    return () => {
      connection.disconnect();
    };
  }, []);
}
```

The Effect reads:

```text
roomId
```

but the dependency array says:

```text
nothing reactive is used
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

---

# 15. The Dependency Linter Protects You from Stale Closures ⭐⭐⭐⭐⭐

When React's Hooks lint rules say:

```text
missing dependency: roomId
```

they are identifying a possible mismatch:

```text
Effect closure uses roomId
        ↓
dependency array does not track roomId
```

Do not silence the warning simply to stop the Effect from running.

Fix the synchronization design.

---

# 16. Stale Event Listener Example ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const [count, setCount] =
    useState(0);

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

  // ...
}
```

The listener created by the initial Effect can keep seeing the initial `count`.

If the listener's behavior must synchronize with `count`, include it:

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

This removes and re-adds the listener when `count` changes.

---

# 17. But Re-subscribing Is Not Always Ideal

Sometimes you want:

```text
subscription identity
to remain stable
```

while still reading some latest value.

Possible tools include:

- functional state updates
- refs
- restructuring the Effect
- React Effect Events in appropriate modern React code

Do not solve this by lying to the dependency array.

---

# 18. Stale Async Callback ⭐⭐⭐⭐⭐

```jsx
function Search({
  query,
}) {
  useEffect(() => {
    async function load() {
      const result =
        await search(query);

      console.log(query);
    }

    load();
  }, []);
}
```

The Effect captures the initial `query`.

If query changes later, this synchronization does not follow it.

Correct dependency:

```jsx
useEffect(() => {
  async function load() {
    const result =
      await search(query);

    // ...
  }

  load();
}, [query]);
```

Then also handle race conditions as covered in Lesson 21.

---

# 19. Closure Problem vs Race Condition ⭐⭐⭐⭐⭐

These are related but different.

### Stale closure

A callback uses a value from an older render.

```text
old closure
   ↓
old value
```

### Race condition

Multiple async operations complete in an unexpected order.

```text
Request A starts
Request B starts
B finishes
A finishes
```

You can have:

- a stale closure without a race condition
- a race condition without a missing dependency
- both at the same time

Debug them separately.

---

# 20. Stale setTimeout Example ⭐⭐⭐⭐⭐

```jsx
function AlertCounter() {
  const [count, setCount] =
    useState(0);

  function handleAlert() {
    setTimeout(() => {
      alert(count);
    }, 3000);
  }

  // ...
}
```

If:

```text
count = 5
click Alert
increment to 6
increment to 7
timeout fires
```

the alert may show:

```text
5
```

because that callback belongs to the render where count was 5.

This may be correct if you mean:

> Show the value when the user clicked.

If you mean:

> Show the latest value when the timeout fires.

you need a different design.

---

# 21. Refs Can Hold a Latest Mutable Value ⭐⭐⭐⭐⭐

A ref can provide a stable object whose `current` value is mutable.

```jsx
const latestCount =
  useRef(count);

latestCount.current =
  count;
```

Then a delayed callback can read:

```jsx
setTimeout(() => {
  console.log(
    latestCount.current
  );
}, 3000);
```

Conceptually:

```text
render snapshots
0 → 1 → 2

stable ref object
current:
0 → 1 → 2

old callback
    ↓
reads ref.current
    ↓
latest 2
```

Refs are covered deeply in Lesson 24.

---

# 22. Snapshot vs Ref ⭐⭐⭐⭐⭐

State:

```text
render-specific snapshot
```

Ref:

```text
stable mutable container
across renders
```

Example:

```text
Render #1 state count = 0
Render #2 state count = 1
Render #3 state count = 2

old callback from Render #1
state count → 0

same callback reading latestRef.current
→ potentially 2
```

Use refs intentionally.

Do not replace normal state with refs just to avoid understanding dependencies.

---

# 23. Updating a Latest-Value Ref

One common pattern is:

```jsx
const latestValue =
  useRef(value);

useEffect(() => {
  latestValue.current =
    value;
}, [value]);
```

Or, for values where assigning during render is appropriate for the pattern:

```jsx
latestValue.current =
  value;
```

Then asynchronous callbacks can read the ref.

However:

> If changing the value should cause the external synchronization itself to restart, use the dependency instead of hiding the change behind a ref.

---

# 24. Do Not Use Refs to Bypass Correct Dependencies ⭐⭐⭐⭐⭐

Bad reasoning:

```text
The linter says roomId is required.
I'll put roomId in a ref so
the Effect never re-runs.
```

If changing rooms should:

```text
disconnect old room
+
connect new room
```

then `roomId` must drive synchronization.

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

A ref is not a loophole around Effect semantics.

---

# 25. Effect Dependencies and Function Closures ⭐⭐⭐⭐⭐

Suppose:

```jsx
function Search({
  query,
}) {
  function createRequest() {
    return {
      query,
    };
  }

  useEffect(() => {
    const request =
      createRequest();

    send(request);
  }, [createRequest]);
}
```

Every render creates a new `createRequest` function.

That function also closes over that render's `query`.

This combines:

```text
function identity
+
closure
+
Effect dependency
```

Often simpler:

```jsx
useEffect(() => {
  const request = {
    query,
  };

  send(request);
}, [query]);
```

---

# 26. Moving Functions Inside Effects Can Simplify Closures

If a function is used only by an Effect:

```jsx
useEffect(() => {
  function createRequest() {
    return {
      query,
    };
  }

  send(createRequest());
}, [query]);
```

Now dependency reasoning is clearer:

```text
Effect uses query
      ↓
[query]
```

This is usually easier than stabilizing an unnecessary external function.

---

# 27. Functions Defined Outside the Component

If a function does not need component values:

```jsx
function formatRequest(query) {
  return {
    query:
      query.trim(),
  };
}

function Search({
  query,
}) {
  useEffect(() => {
    send(
      formatRequest(query)
    );
  }, [query]);
}
```

`formatRequest` itself is not recreated on each component render.

The reactive input remains:

```text
query
```

---

# 28. Stale Closure in a Custom Hook

Custom Hooks can have the same issue.

```jsx
function useWindowKey(
  onKey
) {
  useEffect(() => {
    function handleKey(event) {
      onKey(event.key);
    }

    window.addEventListener(
      "keydown",
      handleKey
    );

    return () => {
      window.removeEventListener(
        "keydown",
        handleKey
      );
    };
  }, []);
}
```

The Effect captures the first `onKey`.

If the caller provides a new callback later, the listener may still use the old one.

At minimum, dependency reasoning must acknowledge:

```jsx
[onKey]
```

or the Hook must intentionally use another modern pattern for reading the latest non-reactive callback.

Custom Hooks are covered later.

---

# 29. React 19: Effect Events for Non-Reactive Effect Logic ⭐⭐⭐⭐⭐

Modern React includes `useEffectEvent` for a specific problem:

> An Effect should react to some values, while logic called from that Effect needs to read the latest values without making those values dependencies of the Effect itself.

Conceptual example:

```jsx
import {
  useEffect,
  useEffectEvent,
} from "react";

function ChatRoom({
  roomId,
  theme,
}) {
  const onConnected =
    useEffectEvent(() => {
      showNotification(
        "Connected!",
        theme
      );
    });

  useEffect(() => {
    const connection =
      createConnection(roomId);

    connection.on(
      "connected",
      () => {
        onConnected();
      }
    );

    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [roomId]);

  // ...
}
```

Here:

```text
roomId
→ determines connection synchronization
→ reactive Effect dependency

theme
→ should be latest when notification runs
→ but should not reconnect the room
```

`useEffectEvent` separates those concerns.

---

# 30. Effect Events Are Not a Dependency Escape Hatch ⭐⭐⭐⭐⭐

Do not move every dependency into an Effect Event.

Wrong mental model:

```text
I don't want my Effect to re-run
      ↓
hide dependencies in useEffectEvent
```

Correct mental model:

```text
Does this value determine whether
the external synchronization
must restart?
      ↓
yes
→ dependency

Do I only need the latest value
inside non-reactive logic
triggered by the Effect?
      ↓
possibly Effect Event
```

Use the tool according to semantics.

---

# 31. Effect Events Have Special Semantics

Effect Events are intended to be called from Effect-related logic, not used as ordinary event handlers everywhere.

They allow code to:

```text
read latest props/state
without making that particular
logic reactive
```

They are useful when separating:

```text
reactive synchronization
from
non-reactive Effect logic
```

For normal button clicks, continue using normal event handlers.

---

# 32. Example: Connection + Theme ⭐⭐⭐⭐⭐

Without separating concerns:

```jsx
useEffect(() => {
  const connection =
    createConnection(roomId);

  connection.on(
    "connected",
    () => {
      showNotification(
        "Connected",
        theme
      );
    }
  );

  connection.connect();

  return () => {
    connection.disconnect();
  };
}, [roomId, theme]);
```

Problem:

```text
theme changes
    ↓
Effect dependency changes
    ↓
disconnect room
    ↓
reconnect room
```

But theme does not determine which room should be connected.

With an Effect Event:

```text
roomId change
→ reconnect

theme change
→ notification uses latest theme
→ no room reconnection solely for theme
```

This is the exact type of problem Effect Events address.

---

# 33. Event Handlers Naturally Capture Render Values ⭐⭐⭐⭐⭐

Normal event handlers also use closures.

```jsx
function Product({
  product,
}) {
  function handleBuy() {
    buy(product.id);
  }

  return (
    <button onClick={handleBuy}>
      Buy
    </button>
  );
}
```

The handler created by a render captures that render's `product`.

Normally this is desirable because the rendered button corresponds to that product.

Closures are fundamental to React, not something to avoid.

---

# 34. Stale Closure from Memoization

Later you will learn `useCallback`.

A callback can become stale if its dependencies are wrong:

```jsx
const handleSave =
  useCallback(() => {
    save(name);
  }, []);
```

If `name` changes, this callback can still use the initial name.

Correct:

```jsx
const handleSave =
  useCallback(() => {
    save(name);
  }, [name]);
```

Memoization does not eliminate closure rules.

It makes correct dependency reasoning even more important.

---

# 35. React.memo Does Not Fix Stale Closures

A stale closure is not fundamentally a rendering-performance issue.

Tools like:

- `React.memo`
- `useMemo`
- `useCallback`

do not automatically solve stale data.

You must understand:

```text
which render created the function
+
which values it captured
+
when that function executes
```

---

# 36. Stale Closures and Batching

Batching is different.

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

All three expressions use the same render snapshot.

This is related to snapshot semantics, but the state update queue solves it with:

```jsx
setCount((count) => count + 1);
setCount((count) => count + 1);
setCount((count) => count + 1);
```

Do not confuse:

```text
state update queue
with
long-lived stale callbacks
```

They share snapshot concepts but are different problems.

---

# 37. Debugging a Suspected Stale Closure ⭐⭐⭐⭐⭐

Ask:

```text
1. Which render created this function?

2. Which props/state did that function capture?

3. When does the function actually execute?

4. Should it use the captured value
   or the latest value?

5. If this is an Effect:
   are all reactive dependencies declared?

6. If next state depends on previous state:
   can I use a functional updater?

7. If the external synchronization should
   restart when a value changes:
   make it a dependency.

8. If synchronization should stay stable
   but callback logic needs latest data:
   is a ref or Effect Event appropriate?
```

This is a strong debugging checklist.

---

# 38. Common Mistakes ⭐⭐⭐⭐⭐

## Mistake 1 — Assuming callbacks always see latest state

They see values captured from the render that created them.

---

## Mistake 2 — Removing Effect dependencies to stop reruns

This can create stale closures.

---

## Mistake 3 — Adding [] because you want an Effect to run once

The Effect may still depend on reactive values.

---

## Mistake 4 — Treating every old captured value as a bug

Sometimes the historical snapshot is exactly what the callback should use.

---

## Mistake 5 — Using refs to hide dependencies

If synchronization should restart, declare the dependency.

---

## Mistake 6 — Recreating subscriptions unnecessarily

Sometimes the Effect can be restructured, use a functional update, or separate reactive/non-reactive logic.

---

## Mistake 7 — Expecting useCallback to solve stale values automatically

Incorrect dependencies can make a memoized callback stale.

---

## Mistake 8 — Confusing race conditions with stale closures

They are different problems and can require different solutions.

---

## Mistake 9 — Putting side effects in state updater functions

Updater functions should remain pure.

---

## Mistake 10 — Using Effect Events as a general dependency workaround

Use them only when logic genuinely needs latest values without making that logic reactive.

---

# 39. Real-World CodeBuddy Example ⭐⭐⭐⭐⭐

Imagine a conversation connection:

```jsx
function Conversation({
  conversationId,
  currentUser,
}) {
  useEffect(() => {
    const socket =
      connectToConversation(
        conversationId
      );

    function handleMessage(
      message
    ) {
      console.log(
        currentUser.name,
        message
      );
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
  }, [
    conversationId,
    currentUser,
  ]);

  // ...
}
```

Ask two separate questions:

```text
Does conversationId determine
which socket synchronization exists?
→ yes

Should currentUser changing
recreate the socket connection?
→ maybe not
```

If only the message callback needs the latest user information, separating reactive connection logic from latest-value callback logic may produce a better design.

The key is semantic reasoning—not blindly adding or removing dependencies.

---

# 40. Interview Questions ⭐⭐⭐⭐⭐

## Q1. What is a JavaScript closure?

**Answer:** A closure is when a function retains access to variables from the lexical scope in which it was created.

---

## Q2. How do closures relate to React state?

**Answer:** Functions created during a render close over the props and state values from that render's snapshot.

---

## Q3. What is a stale closure?

**Answer:** A stale closure occurs when a callback uses values captured from an older render while the intended logic requires newer values.

---

## Q4. Why can setTimeout log an old state value?

**Answer:** The timeout callback was created during an earlier render and captured that render's state value.

---

## Q5. Is logging an old value always a bug?

**Answer:** No. Sometimes the callback intentionally represents the state at the time an event occurred.

---

## Q6. Why can [] cause stale Effect values?

**Answer:** If the Effect reads reactive values but declares no dependencies, it may continue using values captured by its initial setup instead of re-synchronizing.

---

## Q7. How does a functional state update help?

**Answer:** It lets React provide the latest pending state to the updater, avoiding reliance on a captured state value when calculating the next state.

---

## Q8. Can refs solve stale closures?

**Answer:** Refs can expose a latest mutable value to long-lived callbacks, but they should not be used to hide values that should genuinely trigger Effect re-synchronization.

---

## Q9. Why does the exhaustive-deps rule matter?

**Answer:** It helps ensure an Effect re-synchronizes when reactive values used by its closure change.

---

## Q10. What is the difference between a stale closure and a race condition?

**Answer:** A stale closure reads an older captured render value. A race condition occurs when asynchronous operations complete in an unexpected order.

---

## Q11. Can useCallback have stale closures?

**Answer:** Yes. A memoized callback still follows JavaScript closure rules, and incorrect dependencies can make it use old values.

---

## Q12. What problem does useEffectEvent address?

**Answer:** It can separate reactive Effect synchronization from non-reactive Effect logic that needs to read the latest props/state without restarting the synchronization solely because those values changed.

---

# 41. Interview Scenario 1 ⭐⭐⭐⭐⭐

What will this log?

```jsx
function Counter() {
  const [count, setCount] =
    useState(0);

  function handleClick() {
    setTimeout(() => {
      console.log(count);
    }, 3000);

    setCount(
      (count) => count + 1
    );
  }

  // ...
}
```

If `count` was 0 when clicked:

```text
timeout callback
captures count = 0
```

The later state update creates a new render but does not mutate the old callback's captured value.

Result:

```text
0
```

---

# 42. Interview Scenario 2 ⭐⭐⭐⭐⭐

Why does this stop at 1?

```jsx
const [count, setCount] =
  useState(0);

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

The Effect captures:

```text
count = 0
```

Every interval tick requests:

```text
setCount(1)
```

Fix:

```jsx
setCount(
  (count) => count + 1
);
```

Now React provides the latest pending count.

---

# 43. Interview Scenario 3 ⭐⭐⭐⭐⭐

What is wrong?

```jsx
useEffect(() => {
  fetchUser(userId)
    .then(setUser);
}, []);
```

The Effect uses:

```text
userId
```

but does not declare it.

If `userId` changes, the Effect does not re-synchronize.

Correct dependency:

```jsx
[userId]
```

Then also protect against request races where appropriate.

---

# 44. Interview Scenario 4 ⭐⭐⭐⭐⭐

Suppose a chat connection depends on `roomId`, while a connected notification should use the latest `theme`.

Should changing theme reconnect the room?

Usually:

```text
No
```

Conceptually:

```text
roomId
→ reactive connection dependency

theme
→ latest value needed by notification
→ does not define connection identity
```

This is a case where separating reactive Effect logic from non-reactive latest-value logic, such as with `useEffectEvent`, can be appropriate.

---

# 45. Closure Timeline Diagram ⭐⭐⭐⭐⭐

```text
RENDER #1
count = 0
    │
    ├── handler A
    │     closes over 0
    │
    └── Effect callback A
          closes over 0

        state update
             ↓

RENDER #2
count = 1
    │
    ├── handler B
    │     closes over 1
    │
    └── Effect callback B
          closes over 1

Old handler A still exists?
        │
       yes
        ↓
If it executes,
it still sees its
Render #1 snapshot.
```

---

# 46. Choosing the Correct Solution ⭐⭐⭐⭐⭐

```text
Callback sees an older value
          │
          ↓
Does it actually need latest?
     │              │
    no             yes
     │              │
     ↓              ↓
keep snapshot   Why is callback long-lived?
                    │
       ┌────────────┼─────────────┐
       ↓            ↓             ↓
Effect should   next state     synchronization
re-sync when    depends on      should stay stable
value changes   previous state  but callback needs
       │            │            latest data
       ↓            ↓             ↓
dependency      functional     ref / appropriate
                updater        Effect Event pattern
```

---

# 47. Quick Revision ⭐⭐⭐⭐⭐

Remember:

```text
Every render
creates new values
and new functions.
```

A function remembers:

```text
the render in which
it was created
```

Stale closure:

```text
old callback
+
old captured value
+
logic needed latest value
```

Common solutions:

```text
Correct Effect dependency
→ re-synchronize

Functional updater
→ latest pending state

Ref
→ latest mutable value
  without render

useEffectEvent
→ latest non-reactive values
  in Effect-related logic

Restructure code
→ often best solution
```

---

# 48. Key Takeaways

- Closures are normal JavaScript behavior and are fundamental to React.
- Every React render creates its own snapshot of props and state.
- Functions created during a render close over that render's values.
- Old callbacks do not automatically receive values from newer renders.
- A stale closure exists when an older captured value is used but current logic requires the latest value.
- An older captured value is not always wrong; sometimes historical event-time state is exactly what you want.
- Missing Effect dependencies are a common cause of stale synchronization.
- The exhaustive-deps linter helps prevent stale closure bugs.
- Do not remove dependencies merely to reduce Effect execution.
- Functional state updaters avoid captured state when next state depends on previous state.
- Functional updaters do not solve stale props or every stale variable.
- Refs can expose latest mutable values to long-lived callbacks without causing renders.
- Do not use refs to hide dependencies that should restart synchronization.
- Stale closures and race conditions are different concepts.
- Functions and objects created during rendering also participate in dependency and closure behavior.
- Memoization does not remove JavaScript closure semantics.
- `useEffectEvent` can separate reactive Effect synchronization from non-reactive Effect logic that needs latest values.
- Effect Events are not a general dependency escape hatch.
- The best debugging question is: **Which render created this function, what did it capture, and should it use that snapshot or the latest value?**

---

## Next Lesson

➡️ [Lesson 24 — useRef ⭐⭐⭐⭐⭐](./24-useref.md)
