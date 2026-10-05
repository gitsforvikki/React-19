# Lesson 20 — Dependency Arrays and Reactive Dependencies ⭐⭐⭐⭐⭐

## 1. Why This Topic Matters

Writing an Effect is often easy.

Writing its dependency list correctly is where many React bugs begin.

Common problems include:

- stale values
- Effects running too often
- infinite loops
- unnecessary reconnections
- ignored linter warnings
- object/function dependencies changing every render

The most important rule is:

> **You do not freely choose an Effect's dependencies. The code inside the Effect determines them.**

If an Effect reads a reactive value, that value generally belongs in the dependency list.

---

## 2. What Is a Reactive Value? ⭐⭐⭐⭐⭐

Reactive values are values that can change because of rendering.

Common examples:

- props
- state
- variables calculated inside the component
- functions declared inside the component
- objects/arrays created inside the component

Example:

```jsx
function ChatRoom({
  roomId,
}) {
  const [serverUrl, setServerUrl] =
    useState(
      "https://localhost:1234"
    );

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
}
```

The Effect reads:

```text
serverUrl
roomId
```

Both are reactive.

Therefore both belong in the dependency list.

---

## 3. Dependencies Describe the Effect ⭐⭐⭐⭐⭐

Do not think:

> "How often do I want this Effect to run?"

Think:

> "Which reactive values does this synchronization process use?"

Example:

```jsx
useEffect(() => {
  document.title =
    `Room: ${roomId}`;
}, [roomId]);
```

The dependency list is a description:

```text
This synchronization depends on roomId.
```

When `roomId` changes, the external document title must be synchronized again.

---

## 4. Three Dependency Forms

### No dependency array

```jsx
useEffect(() => {
  // ...
});
```

Effect is scheduled after every relevant committed render.

### Empty dependency array

```jsx
useEffect(() => {
  // ...
}, []);
```

The Effect has no changing reactive dependencies.

### Dependency list

```jsx
useEffect(() => {
  // ...
}, [roomId, serverUrl]);
```

React re-synchronizes when a dependency changes.

---

## 5. How React Compares Dependencies ⭐⭐⭐⭐⭐

React compares each dependency with its previous value using `Object.is`.

Conceptually:

```text
previous dependency
        ↓
     Object.is
        ↑
current dependency
```

If all dependencies compare equal:

```text
Effect does not need to re-synchronize
```

If one differs:

```text
cleanup old synchronization
        ↓
run setup with new values
```

---

## 6. Primitive Dependencies

Primitives are usually straightforward:

```jsx
useEffect(() => {
  console.log(query);
}, [query]);
```

If:

```text
previous query = "react"
current query  = "react"
```

then it is equal.

If:

```text
previous query = "react"
current query  = "next"
```

the Effect re-synchronizes.

---

## 7. Missing Dependencies Cause Stale Synchronization ⭐⭐⭐⭐⭐

Bad:

```jsx
function ChatRoom({
  roomId,
}) {
  useEffect(() => {
    const connection =
      createConnection(roomId);

    connection.connect();

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

but claims:

```text
dependencies = []
```

If `roomId` changes, the Effect may remain synchronized with the old room.

The dependency list is lying about the Effect's inputs.

Correct:

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

---

## 8. Stale Closure Connection ⭐⭐⭐⭐⭐

Suppose the first render has:

```text
roomId = "general"
```

The Effect callback from that render closes over:

```text
roomId = "general"
```

If the dependency is incorrectly omitted, a later render may have:

```text
roomId = "react"
```

while the existing synchronization still uses the old value.

```text
Render #1
roomId = general
     ↓
Effect closure
general

Render #2
roomId = react
     ↓
Effect not re-run
     ↓
external connection still general
```

This is one reason dependency correctness matters.

Closures are covered deeply in Lesson 23.

---

## 9. The Exhaustive Dependencies Linter ⭐⭐⭐⭐⭐

React's Hooks lint rules can detect many missing dependencies.

Example warning conceptually:

```text
React Hook useEffect has a missing dependency: 'roomId'
```

Do not treat this as an annoying warning to silence.

Treat it as:

> React is telling you that your Effect's code and dependency declaration disagree.

Often the right solution is to restructure the Effect, not disable the linter.

---

## 10. Do Not Suppress the Linter to Control Timing ⭐⭐⭐⭐⭐

Bad:

```jsx
useEffect(() => {
  connect(roomId);

  // eslint-disable-next-line ...
}, []);
```

This says:

```text
Effect uses roomId
but pretend roomId never changes
```

That can produce bugs.

Better:

1. include the dependency, or
2. change the code so the Effect genuinely no longer depends on it.

---

## 11. How to Remove a Dependency Correctly ⭐⭐⭐⭐⭐

You cannot safely remove a dependency merely by deleting it from the array.

You remove a dependency by changing the code so it is no longer reactive.

Example:

```jsx
const serverUrl =
  "https://example.com";

function ChatRoom({
  roomId,
}) {
  useEffect(() => {
    const connection =
      createConnection(
        serverUrl,
        roomId
      );

    // ...
  }, [roomId]);
}
```

Because `serverUrl` is now outside the component and does not depend on rendering, it is not a reactive component value.

You have **proved** it does not need to be a dependency.

---

## 12. Constants Outside the Component

```jsx
const API_URL =
  "https://api.example.com";

function Users() {
  useEffect(() => {
    fetch(API_URL);
  }, []);
}
```

`API_URL` is module-level constant data.

It does not change because the component rendered again.

Therefore it is not a reactive dependency.

---

# Object Dependencies

## 13. Objects Created During Render ⭐⭐⭐⭐⭐

Consider:

```jsx
function ChatRoom({
  roomId,
}) {
  const options = {
    serverUrl:
      "https://localhost:1234",
    roomId,
  };

  useEffect(() => {
    const connection =
      createConnection(options);

    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [options]);
}
```

Problem:

Every render creates a new object:

```text
Render #1
options → Object A

Render #2
options → Object B

Object.is(A, B)
→ false
```

Even if the contents look identical, the references differ.

The Effect can re-run unnecessarily.

---

## 14. Best Fix: Create the Object Inside the Effect ⭐⭐⭐⭐⭐

Instead:

```jsx
function ChatRoom({
  roomId,
}) {
  useEffect(() => {
    const options = {
      serverUrl:
        "https://localhost:1234",
      roomId,
    };

    const connection =
      createConnection(options);

    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [roomId]);
}
```

Now the Effect depends directly on:

```text
roomId
```

instead of an object recreated during rendering.

This is often better than adding memoization just to satisfy an Effect.

---

## 15. Do Not Reach for useMemo First ⭐⭐⭐⭐⭐

You could write:

```jsx
const options = useMemo(
  () => ({
    serverUrl:
      "https://localhost:1234",
    roomId,
  }),
  [roomId]
);
```

But if the object is only needed by the Effect, moving it inside the Effect is simpler.

Prefer structural simplification before memoization.

`useMemo` is covered later.

---

# Function Dependencies

## 16. Functions Declared During Render ⭐⭐⭐⭐⭐

Consider:

```jsx
function ChatRoom({
  roomId,
}) {
  function createOptions() {
    return {
      serverUrl:
        "https://localhost:1234",
      roomId,
    };
  }

  useEffect(() => {
    const options =
      createOptions();

    const connection =
      createConnection(options);

    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [createOptions]);
}
```

A new `createOptions` function is created during each render.

Conceptually:

```text
Render #1
createOptions → Function A

Render #2
createOptions → Function B
```

So the dependency changes.

---

## 17. Best Fix: Move the Function Inside the Effect

```jsx
function ChatRoom({
  roomId,
}) {
  useEffect(() => {
    function createOptions() {
      return {
        serverUrl:
          "https://localhost:1234",
        roomId,
      };
    }

    const connection =
      createConnection(
        createOptions()
      );

    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [roomId]);
}
```

Now the dependency is the actual reactive value:

```text
roomId
```

---

## 18. Or Move Non-Reactive Functions Outside

If a helper does not use component props/state:

```jsx
function createOptions(roomId) {
  return {
    serverUrl:
      "https://localhost:1234",
    roomId,
  };
}

function ChatRoom({
  roomId,
}) {
  useEffect(() => {
    const connection =
      createConnection(
        createOptions(roomId)
      );

    // ...
  }, [roomId]);
}
```

The helper itself is stable because it is defined outside the component.

---

## 19. useCallback Is Not the First Fix

You may later learn:

```jsx
const createOptions =
  useCallback(() => {
    // ...
  }, [roomId]);
```

That can stabilize a function reference.

But first ask:

> Does this function need to exist outside the Effect?

If no, move it inside.

Simpler dependency graphs are usually easier to maintain.

---

# Dependency Design

## 20. Every Reactive Value Used by the Effect Matters ⭐⭐⭐⭐⭐

Example:

```jsx
function Search({
  query,
  page,
}) {
  useEffect(() => {
    fetchResults(
      query,
      page
    );
  }, [query, page]);
}
```

The Effect reads:

```text
query
page
```

Therefore:

```jsx
[query, page]
```

describes its reactive inputs.

---

## 21. Values Used Only During Render Are Not Effect Dependencies

```jsx
function Search({
  query,
  users,
}) {
  const filteredUsers =
    users.filter((user) =>
      user.name.includes(query)
    );

  useEffect(() => {
    document.title =
      `Search: ${query}`;
  }, [query]);

  // ...
}
```

The Effect does not use:

```text
filteredUsers
users
```

so they are not dependencies of this Effect.

Dependencies are based on what the Effect actually reads.

---

## 22. State Setters Are Stable

React state setter functions have stable identity.

Example:

```jsx
const [count, setCount] =
  useState(0);
```

React guarantees that `setCount` has stable identity across renders.

You generally do not need to include it manually when the linter permits omission.

Focus on the reactive values whose changes alter synchronization.

---

## 23. Refs and Dependencies

A ref object returned by:

```jsx
const ref = useRef(null);
```

has stable identity.

Also, changing:

```js
ref.current
```

does not trigger a render.

Therefore `ref.current` is not a normal reactive dependency.

This is important:

> Dependencies track reactive render values, not arbitrary mutable values.

Refs are covered deeply in Lesson 24.

---

## 24. Dependency Array Must Have Stable Structure

Write dependencies inline:

```jsx
useEffect(() => {
  // ...
}, [roomId, serverUrl]);
```

The dependency list should have a constant number of items.

Do not dynamically construct changing dependency-array shapes.

---

## 25. Dependencies and Cleanup ⭐⭐⭐⭐⭐

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

Transition:

```text
roomId = A
  ↓
connect A

roomId changes to B
  ↓
render
  ↓
cleanup Effect from A
  ↓
disconnect A
  ↓
setup Effect for B
  ↓
connect B
```

Dependencies do not merely decide whether code runs.

They define when a synchronization process must be restarted with new values.

---

## 26. Multiple Dependencies

```jsx
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
```

React compares both.

```text
serverUrl same?
roomId same?
     ↓
both same
→ preserve current synchronization

either changed
→ cleanup + setup
```

---

## 27. Dependency Arrays and Infinite Loops ⭐⭐⭐⭐⭐

Example:

```jsx
const options = {
  page: 1,
};

useEffect(() => {
  setResults(
    calculateResults(options)
  );
}, [options]);
```

Every render:

```text
new options object
     ↓
dependency changed
     ↓
Effect
     ↓
setResults
     ↓
render
     ↓
new options object
     ↓
...
```

This can create repeated Effect execution or loops.

But the deeper question is:

> Do you need an Effect to calculate `results`?

If `calculateResults` is pure, calculate during render instead.

---

## 28. The Correct Fix Is Often Removing the Effect ⭐⭐⭐⭐⭐

Bad:

```jsx
const [filtered, setFiltered] =
  useState([]);

useEffect(() => {
  setFiltered(
    products.filter(
      (product) =>
        product.name.includes(query)
    )
  );
}, [products, query]);
```

No external system exists.

Better:

```jsx
const filtered =
  products.filter(
    (product) =>
      product.name.includes(query)
  );
```

Now there is:

- no Effect
- no dependency problem
- no extra state
- no extra render

This is an important advanced React habit.

---

## 29. Separate Effects by Dependency Purpose ⭐⭐⭐⭐⭐

Bad:

```jsx
useEffect(() => {
  connectToRoom(roomId);
  logPage(page);
}, [roomId, page]);
```

If `page` changes:

```text
chat connection logic runs again
```

even if chat synchronization did not need to change.

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

Each Effect has a focused dependency model.

---

## 30. Dependency Changes Are About Equality, Not Meaning ⭐⭐⭐⭐⭐

React does not deeply inspect:

```js
{
  roomId: "react"
}
```

to decide whether it "means the same thing."

It compares references with `Object.is`.

Therefore:

```js
const a = { roomId: "react" };
const b = { roomId: "react" };

Object.is(a, b); // false
```

This explains many object dependency issues.

---

## 31. Array Dependencies Have the Same Reference Issue

```jsx
const selectedIds =
  users.map((user) => user.id);

useEffect(() => {
  syncSelection(selectedIds);
}, [selectedIds]);
```

A new array can be created on every render.

Even if values are identical:

```text
Array A !== Array B
```

Before memoizing, ask whether the Effect can depend on simpler primitive inputs or whether the calculation belongs elsewhere.

---

## 32. Do Not Stringify Dependencies as a Shortcut

Avoid patterns such as:

```jsx
useEffect(() => {
  // ...
}, [JSON.stringify(options)]);
```

This is usually a workaround rather than good dependency design.

Problems include:

- unnecessary serialization
- hidden dependency semantics
- performance cost
- brittle behavior

Design the Effect around the actual reactive values.

---

## 33. Reading Latest Values Without Re-synchronizing

Sometimes you need an Effect to synchronize based on one value but read another latest value without that value restarting the synchronization.

This is an advanced problem.

Do not solve it by simply omitting a dependency.

Depending on the use case, React patterns such as refs or Effect Events may be appropriate.

We will first build the foundations of:

- closures
- refs
- Effect design

before using advanced solutions.

---

## 34. Real-World Example: Chat Connection ⭐⭐⭐⭐⭐

```jsx
function ChatRoom({
  serverUrl,
  roomId,
}) {
  useEffect(() => {
    const socket =
      createSocket({
        serverUrl,
        roomId,
      });

    socket.connect();

    return () => {
      socket.disconnect();
    };
  }, [serverUrl, roomId]);

  return <ChatUI />;
}
```

Correct dependency reasoning:

```text
Effect reads serverUrl
       +
Effect reads roomId
       ↓
dependencies
[serverUrl, roomId]
```

If either changes, the connection should be recreated.

---

## 35. Real-World Example: Search Query

Suppose an Effect synchronizes a browser API:

```jsx
function SearchPage({
  query,
}) {
  useEffect(() => {
    const params =
      new URLSearchParams(
        window.location.search
      );

    params.set("q", query);

    window.history.replaceState(
      null,
      "",
      `?${params}`
    );
  }, [query]);

  // ...
}
```

The external browser URL is synchronized with:

```text
query
```

so `query` is the dependency.

---

## 36. Real-World Example: Reconnection Bug

Bad:

```jsx
function ChatRoom({
  roomId,
}) {
  const options = {
    roomId,
  };

  useEffect(() => {
    const socket =
      connect(options);

    return () => {
      socket.disconnect();
    };
  }, [options]);
}
```

Typing into unrelated state in the component can trigger a render.

Then:

```text
new render
  ↓
new options object
  ↓
dependency changed
  ↓
disconnect
  ↓
reconnect
```

Better:

```jsx
useEffect(() => {
  const options = {
    roomId,
  };

  const socket =
    connect(options);

  return () => {
    socket.disconnect();
  };
}, [roomId]);
```

Now unrelated renders do not recreate the connection.

---

## 37. Common Mistakes ⭐⭐⭐⭐⭐

### Mistake 1 — Choosing dependencies based on desired frequency

Dependencies come from reactive values used by the Effect.

### Mistake 2 — Omitting a dependency to prevent reruns

This can create stale synchronization.

### Mistake 3 — Disabling the exhaustive-deps lint rule

Usually fix the Effect instead.

### Mistake 4 — Adding an object created during render as a dependency

Its reference may change every render.

### Mistake 5 — Adding a function created during render without considering its identity

The function may change every render.

### Mistake 6 — Using useMemo/useCallback before simplifying the Effect

Move Effect-only objects/functions inside the Effect when possible.

### Mistake 7 — Using an Effect for derived state

Often remove the Effect entirely.

### Mistake 8 — Assuming React deep-compares dependencies

React uses `Object.is`, not deep equality.

### Mistake 9 — Treating ref.current as normal reactive state

Ref changes do not cause rendering.

### Mistake 10 — Combining unrelated processes into one Effect

Separate them according to synchronization purpose.

---

## 38. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What belongs in a useEffect dependency array?

**Answer:** Reactive values read by the Effect, including relevant props, state, and component-scoped values that can change between renders.

### Q2. Can you choose dependencies manually based on how often you want the Effect to run?

**Answer:** No. Dependencies should reflect the Effect's reactive inputs.

### Q3. How does React compare dependencies?

**Answer:** React compares each dependency with its previous value using `Object.is`.

### Q4. Why are object dependencies dangerous?

**Answer:** Objects created during render have new references on each render, so React can see them as changed even when their contents are equivalent.

### Q5. How can you fix an unnecessary object dependency?

**Answer:** Often create the object inside the Effect or move static data outside the component so the Effect depends on simpler reactive values.

### Q6. Why can function dependencies cause repeated Effects?

**Answer:** Functions declared during rendering are new function objects on each render unless their identity is otherwise stabilized.

### Q7. Should you disable exhaustive-deps?

**Answer:** Generally no. A warning often reveals a mismatch between the Effect's code and declared dependencies.

### Q8. How do you legitimately remove a dependency?

**Answer:** Change the code so the Effect no longer reads that reactive value, or prove it is non-reactive by moving appropriate logic/data outside the component.

### Q9. Does an empty dependency array mean "run once"?

**Answer:** That is an oversimplification. It means the Effect has no changing reactive dependencies. Development Strict Mode can also perform an extra setup/cleanup cycle.

### Q10. Are refs reactive dependencies?

**Answer:** A ref object has stable identity, and changes to `ref.current` do not trigger renders, so ref contents do not behave like normal reactive state dependencies.

### Q11. Why can missing dependencies cause stale closures?

**Answer:** The Effect can continue using values captured from an older render instead of re-synchronizing when those values change.

### Q12. What is often the best fix for difficult dependency problems?

**Answer:** Simplify the Effect, split unrelated Effects, move Effect-specific objects/functions inside it, or remove the Effect if no external synchronization is needed.

---

## 39. Interview Prediction Problem ⭐⭐⭐⭐⭐

Consider:

```jsx
function Chat({
  roomId,
}) {
  const [message, setMessage] =
    useState("");

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

  return (
    <input
      value={message}
      onChange={(event) =>
        setMessage(
          event.target.value
        )
      }
    />
  );
}
```

What happens when the user types?

```text
setMessage
   ↓
render
   ↓
new options object
   ↓
Object.is(oldOptions, newOptions)
   ↓
false
   ↓
cleanup old connection
   ↓
create new connection
```

This is an unnecessary reconnection.

Better:

```jsx
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
```

---

## 40. Dependency Debugging Checklist ⭐⭐⭐⭐⭐

When an Effect runs too often, ask:

```text
1. Does this Effect need to exist?
2. What external system is it synchronizing?
3. Which reactive values does it actually read?
4. Are any dependencies objects created during render?
5. Are any dependencies functions created during render?
6. Can those objects/functions move inside the Effect?
7. Can static values move outside the component?
8. Are unrelated synchronization processes mixed together?
9. Am I trying to fight the linter instead of fixing the design?
```

This checklist solves many real-world Effect problems.

---

## 41. Complete Mental Model ⭐⭐⭐⭐⭐

```text
Effect code
    │
    ↓
Which reactive values does it read?
    │
    ↓
DEPENDENCIES
    │
    ↓
React compares old vs new
using Object.is
    │
 ┌──┴─────────────┐
 │                │
same            changed
 │                │
 ↓                ↓
keep current    cleanup
sync process       ↓
                setup
                with new
                values
```

For dependency problems:

```text
Do not ask:
"How do I stop this Effect running?"

Ask:
"Why does this Effect depend on this value,
and is this synchronization designed correctly?"
```

---

## 42. Quick Revision

```text
Props used by Effect
        ↓
dependency

State used by Effect
        ↓
dependency

Component-scoped calculated value
used by Effect
        ↓
dependency if reactive

Module-level constant
        ↓
not reactive

Object/function created every render
        ↓
reference may change every render
```

Best first fixes:

```text
object only needed by Effect
→ create inside Effect

function only needed by Effect
→ define inside Effect

static value
→ move outside component

derived calculation
→ maybe remove Effect entirely
```

---

## 43. Key Takeaways

- Effect dependencies are determined by reactive values used by the Effect.
- Props and state are reactive values.
- Component-scoped values can also be reactive.
- Dependencies should describe synchronization, not desired execution frequency.
- React compares dependencies using `Object.is`.
- Missing dependencies can create stale synchronization and stale closures.
- The exhaustive-deps linter is a correctness tool, not an inconvenience.
- Do not suppress dependency warnings simply to make an Effect run less often.
- To remove a dependency, change the code so the dependency is genuinely unnecessary.
- Objects and functions created during rendering have new identities and can trigger Effects repeatedly.
- If an object or function is only needed by an Effect, moving it inside the Effect is often the simplest solution.
- Move truly static values outside the component when appropriate.
- Do not use memoization automatically just to satisfy dependency arrays.
- Refs do not behave like normal reactive state.
- Split unrelated synchronization processes into separate Effects.
- Many difficult dependency problems disappear when an unnecessary Effect is removed.
- The best question is: **Which reactive values does this synchronization process actually depend on?**

---

## Next Lesson

➡️ [Lesson 21 — Effect Cleanup and Memory Leaks ⭐⭐⭐⭐⭐](./21-effect-cleanup.md)
