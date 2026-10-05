# Lesson 24 — useRef ⭐⭐⭐⭐⭐

## 1. What Is useRef?

`useRef` is a React Hook that gives you a **stable mutable object** that survives across renders.

Basic syntax:

```jsx
import { useRef } from "react";

function Example() {
  const ref = useRef(initialValue);

  console.log(ref.current);
}
```

React returns an object conceptually like:

```js
{
  current: initialValue
}
```

The important characteristics are:

```text
useRef
  │
  ├── persists across renders
  ├── changing ref.current does NOT trigger a render
  └── returns the same ref object on later renders
```

---

# 2. Why Does React Need Refs? ⭐⭐⭐⭐⭐

State is designed for information that affects rendering.

```jsx
const [count, setCount] =
  useState(0);
```

When state changes:

```text
setCount(...)
    ↓
React schedules render
    ↓
UI can update
```

But sometimes a component needs to remember information that **does not belong in the rendered output**.

Examples:

- timer IDs
- DOM elements
- previous values
- mutable integration objects
- latest values needed by long-lived callbacks
- counters used only internally

For these cases, a ref may be appropriate.

---

# 3. State vs Ref ⭐⭐⭐⭐⭐

This distinction is extremely important.

| State | Ref |
|---|---|
| Persists across renders | Persists across renders |
| Updating schedules a render | Updating does not schedule a render |
| Read as a render snapshot | `current` is mutable |
| Used for rendered data | Used for non-rendering mutable data |
| Updated with setter | Updated through `.current` |

Mental model:

```text
STATE
change
  ↓
React needs to know
  ↓
render again

REF
change
  ↓
React does not need to know
  ↓
no render
```

---

# 4. A Ref Persists Across Renders ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const renderCount =
    useRef(0);

  // same ref object survives
  // across renders

  return <div>...</div>;
}
```

Conceptually:

```text
Render #1
ref ───────────────┐
{ current: 0 }     │
                   │
Render #2          │
ref ───────────────┤
same object        │
                   │
Render #3          │
ref ───────────────┘
same object
```

React preserves the ref object for the lifetime of that component instance.

---

# 5. Updating a Ref Does Not Re-render ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const countRef =
    useRef(0);

  function handleClick() {
    countRef.current += 1;

    console.log(
      countRef.current
    );
  }

  return (
    <button onClick={handleClick}>
      Increment ref
    </button>
  );
}
```

Each click changes:

```text
countRef.current
```

but React does not re-render because of that change.

This is intentional.

---

# 6. Why You Should Not Store Visible UI State in a Ref ⭐⭐⭐⭐⭐

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

The value changes internally, but React does not know it should render again.

So the displayed number may remain unchanged.

If a value affects what the user sees:

```jsx
const [count, setCount] =
  useState(0);
```

is normally the correct choice.

Rule:

> **If changing a value should update the UI, use state rather than a ref.**

---

# 7. First Major useRef Use Case: Non-Rendering Mutable Values ⭐⭐⭐⭐⭐

Suppose you need to store a timeout ID:

```jsx
function SearchBox() {
  const timeoutRef =
    useRef(null);

  function handleSearch() {
    clearTimeout(
      timeoutRef.current
    );

    timeoutRef.current =
      setTimeout(() => {
        console.log("Searching...");
      }, 500);
  }

  return (
    <button onClick={handleSearch}>
      Search
    </button>
  );
}
```

The timer ID must survive between calls.

But changing the timer ID does not affect the rendered UI.

That makes a ref suitable.

---

# 8. Why a Normal Variable Is Not Enough ⭐⭐⭐⭐⭐

Consider:

```jsx
function SearchBox() {
  let timeoutId;

  function handleSearch() {
    clearTimeout(timeoutId);

    timeoutId =
      setTimeout(() => {
        console.log("Searching");
      }, 500);
  }

  // ...
}
```

Every render creates a new:

```text
timeoutId
```

So it does not reliably persist across renders.

A ref does:

```jsx
const timeoutRef =
  useRef(null);
```

Conceptually:

```text
normal local variable
→ recreated per render

ref
→ preserved by React
  across renders
```

---

# 9. Ref vs Normal Variable vs State ⭐⭐⭐⭐⭐

```text
NORMAL VARIABLE
├── recreated during each render
├── mutation does not render
└── not suitable for persistent component data

STATE
├── preserved across renders
├── update schedules render
└── use when UI depends on value

REF
├── preserved across renders
├── mutation does not schedule render
└── use for mutable non-rendering information
```

This is one of the most important `useRef` interview comparisons.

---

# 10. Second Major Use Case: Accessing DOM Elements ⭐⭐⭐⭐⭐

React normally manages the DOM declaratively.

But sometimes you need direct access to a DOM element.

Examples:

- focus an input
- scroll an element
- measure an element
- interact with browser APIs
- integrate a third-party library

Example:

```jsx
function SearchInput() {
  const inputRef =
    useRef(null);

  function handleFocus() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />

      <button onClick={handleFocus}>
        Focus input
      </button>
    </>
  );
}
```

After React creates the input DOM node:

```text
inputRef.current
       ↓
actual <input> DOM element
```

---

# 11. DOM Ref Lifecycle ⭐⭐⭐⭐⭐

Initially:

```jsx
const inputRef =
  useRef(null);
```

Before the DOM node is attached:

```text
inputRef.current = null
```

After React commits:

```text
inputRef.current
=
HTMLInputElement
```

When the node is removed:

```text
inputRef.current
=
null
```

High-level flow:

```text
Render
  ↓
React creates/updates DOM
  ↓
Commit
  ↓
ref.current assigned
```

---

# 12. Why DOM Refs Are Escape Hatches

Normally prefer:

```jsx
<input
  value={name}
  onChange={...}
/>
```

rather than manually changing:

```js
inputRef.current.value = ...
```

React's declarative model should control UI whenever possible.

Refs are useful when you need an imperative DOM operation such as:

```js
inputRef.current.focus();
inputRef.current.scrollIntoView();
```

Think:

```text
normal UI state
→ declarative React

imperative DOM operation
→ ref
```

---

# 13. Example: Focus an Input After Clicking Edit

```jsx
function ProfileName() {
  const inputRef =
    useRef(null);

  const [editing, setEditing] =
    useState(false);

  function handleEdit() {
    setEditing(true);
  }

  useEffect(() => {
    if (editing) {
      inputRef.current?.focus();
    }
  }, [editing]);

  return (
    <>
      {editing && (
        <input ref={inputRef} />
      )}

      <button onClick={handleEdit}>
        Edit
      </button>
    </>
  );
}
```

Why is the Effect useful here?

Because the input must first be committed to the DOM.

```text
setEditing(true)
      ↓
render
      ↓
input created
      ↓
commit
      ↓
Effect
      ↓
focus DOM node
```

---

# 14. Optional Chaining with DOM Refs

Because a DOM ref can temporarily be `null`:

```jsx
inputRef.current?.focus();
```

is often safer than:

```jsx
inputRef.current.focus();
```

especially when the node may conditionally exist.

---

# 15. Refs and Event Handlers ⭐⭐⭐⭐⭐

Reading or writing refs from event handlers is normal.

```jsx
function VideoPlayer() {
  const videoRef =
    useRef(null);

  function handlePlay() {
    videoRef.current?.play();
  }

  function handlePause() {
    videoRef.current?.pause();
  }

  return (
    <>
      <video ref={videoRef} />

      <button onClick={handlePlay}>
        Play
      </button>

      <button onClick={handlePause}>
        Pause
      </button>
    </>
  );
}
```

These are imperative browser operations.

A ref is appropriate.

---

# 16. Avoid Reading/Writing Refs During Rendering ⭐⭐⭐⭐⭐

In general, do not do this:

```jsx
function Example() {
  const ref =
    useRef(0);

  ref.current += 1;

  return (
    <p>{ref.current}</p>
  );
}
```

Rendering should remain pure.

Mutating arbitrary refs during render can make component behavior unpredictable and conflict with React's rendering model.

Prefer reading/writing refs:

- in event handlers
- in Effects
- in callbacks outside render

when appropriate.

---

# 17. Important Initialization Exception ⭐⭐⭐⭐⭐

Sometimes a ref needs an expensive object initialized once.

This:

```jsx
const playerRef =
  useRef(
    new VideoPlayer()
  );
```

evaluates `new VideoPlayer()` on every render even though React only uses the initial ref value.

A common predictable initialization pattern is:

```jsx
const playerRef =
  useRef(null);

if (playerRef.current === null) {
  playerRef.current =
    new VideoPlayer();
}
```

This render-time write is a special case because the result is stable and initialization happens predictably once for that component instance.

Do not generalize this into arbitrary ref mutation during render.

---

# 18. Ref Mutation Is Immediate ⭐⭐⭐⭐⭐

State:

```jsx
setCount(count + 1);

console.log(count);
```

still reads the current render's snapshot.

But:

```jsx
ref.current += 1;

console.log(ref.current);
```

reads the newly mutated ref value immediately.

Why?

```text
state
→ React-managed snapshot

ref.current
→ ordinary mutable property
```

This is a major behavioral difference.

---

# 19. Ref Does Not Have a Setter

State:

```jsx
const [count, setCount] =
  useState(0);

setCount(1);
```

Ref:

```jsx
const countRef =
  useRef(0);

countRef.current = 1;
```

No setter is required.

And therefore:

```text
no React state update
→ no render scheduled
```

---

# 20. Ref Identity Is Stable ⭐⭐⭐⭐⭐

React returns the same ref object across renders.

Conceptually:

```text
Render #1
ref object A

Render #2
ref object A

Render #3
ref object A
```

Only:

```text
ref.current
```

may change.

This stable identity is useful for DOM references and mutable integration state.

---

# 21. useRef and Closures ⭐⭐⭐⭐⭐

Lesson 23 showed:

```text
old callback
→ old state snapshot
```

A ref behaves differently because old callbacks can access the same stable ref object.

Example:

```jsx
function Counter() {
  const [count, setCount] =
    useState(0);

  const latestCount =
    useRef(count);

  useEffect(() => {
    latestCount.current =
      count;
  }, [count]);

  function handleAlert() {
    setTimeout(() => {
      alert(
        latestCount.current
      );
    }, 3000);
  }

  // ...
}
```

Even though the timeout callback was created earlier:

```text
callback
   ↓
stable ref object
   ↓
latest ref.current
```

---

# 22. State Snapshot vs Latest Ref ⭐⭐⭐⭐⭐

Suppose:

```text
count state:
Render 1 → 0
Render 2 → 1
Render 3 → 2

latestCount.current:
0 → 1 → 2
```

An old callback from Render 1:

```text
count
→ 0

latestCount.current
→ 2
```

This is why refs can help long-lived callbacks access latest mutable information.

---

# 23. Do Not Automatically Replace Dependencies with Refs ⭐⭐⭐⭐⭐

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

Do not change this to:

```jsx
const roomRef =
  useRef(roomId);
```

just to avoid re-running the Effect.

If `roomId` changes, the external connection must change.

So `roomId` belongs in the dependency array.

Rule:

> Use a ref when you need mutable non-reactive information—not to hide genuine reactive dependencies.

---

# 24. useRef for Timer IDs ⭐⭐⭐⭐⭐

A practical pattern:

```jsx
function Notification() {
  const timeoutRef =
    useRef(null);

  function showNotification() {
    clearTimeout(
      timeoutRef.current
    );

    timeoutRef.current =
      setTimeout(() => {
        console.log("Hide");
      }, 3000);
  }

  useEffect(() => {
    return () => {
      clearTimeout(
        timeoutRef.current
      );
    };
  }, []);

  // ...
}
```

Why ref?

```text
timer ID must persist
+
timer ID does not affect UI
```

Perfect ref use case.

---

# 25. useRef for Previous Values

Sometimes you need to remember a previous committed value.

Example:

```jsx
function Price({
  price,
}) {
  const previousPrice =
    useRef(price);

  useEffect(() => {
    previousPrice.current =
      price;
  }, [price]);

  return (
    <p>
      Current: {price}
      Previous:
      {previousPrice.current}
    </p>
  );
}
```

Conceptually:

```text
render with new price
       ↓
ref still contains
previous committed price
       ↓
Effect runs
       ↓
ref updated for future render
```

Be careful: whether this exact pattern gives the semantics you need depends on when the component renders and when you update the ref.

---

# 26. Custom usePrevious Pattern

A common abstraction:

```jsx
function usePrevious(value) {
  const ref =
    useRef();

  useEffect(() => {
    ref.current =
      value;
  }, [value]);

  return ref.current;
}
```

Usage:

```jsx
const previousName =
  usePrevious(name);
```

The Hook remembers a value without requiring that remembered value itself to trigger a render.

Custom Hooks are covered later.

---

# 27. useRef for Instance-Like Values

Function components do not have a class instance such as:

```js
this.someValue
```

A ref can sometimes hold instance-like mutable information:

```jsx
const connectionRef =
  useRef(null);
```

However, if a resource has a lifecycle tied to dependencies, an Effect should usually create and clean it up correctly.

Example:

```jsx
useEffect(() => {
  const connection =
    createConnection(roomId);

  connectionRef.current =
    connection;

  connection.connect();

  return () => {
    connection.disconnect();

    connectionRef.current =
      null;
  };
}, [roomId]);
```

---

# 28. Ref vs Module-Level Variable ⭐⭐⭐⭐⭐

Bad for component-specific mutable data:

```js
let timeoutId;
```

outside the component.

Why?

Multiple component instances may share the same variable.

```text
Component A ─┐
             ├→ same global variable
Component B ─┘
```

A ref is instance-specific:

```text
Component A
→ ref A

Component B
→ ref B
```

Each component instance gets its own ref.

---

# 29. Ref vs State: Decision Rule ⭐⭐⭐⭐⭐

Ask:

> If this value changes, should React render the component again?

If:

```text
YES
↓
state
```

If:

```text
NO
↓
ref may be appropriate
```

Then ask:

> Does this value need to persist across renders?

If no, a normal local variable may be enough.

Decision tree:

```text
Need value to persist?
      │
  ┌───┴───┐
 no      yes
 │         │
 ↓         ↓
local   Should changing it
variable update the UI?
          │
      ┌───┴───┐
     yes      no
      │        │
      ↓        ↓
    state     ref
```

---

# 30. Ref Is Not Reactive ⭐⭐⭐⭐⭐

Suppose:

```jsx
const countRef =
  useRef(0);

countRef.current = 10;
```

React does not track this as a reactive update.

Therefore an Effect dependency like:

```jsx
useEffect(() => {
  console.log(
    countRef.current
  );
}, [countRef.current]);
```

does not make ref mutations reactive in the same way as state.

Changing `ref.current` does not trigger a render, so React has no new render from which to detect the changed dependency.

If the UI or synchronization must react to changes, state is usually the correct mechanism.

---

# 31. Why Ref Objects Usually Aren't Effect Dependencies

A ref created by:

```jsx
const ref =
  useRef(null);
```

has stable object identity across renders.

Therefore:

```text
ref object
does not change identity
```

Adding the ref object itself to a dependency array usually does not create useful reactivity.

What matters is the lifecycle and semantics of what the Effect is doing.

---

# 32. DOM Measurement with Refs

Sometimes you need to measure an element:

```jsx
const boxRef =
  useRef(null);

useEffect(() => {
  const rect =
    boxRef.current
      ?.getBoundingClientRect();

  console.log(rect);
}, []);
```

For visual measurements that must happen before browser paint to avoid flicker, `useLayoutEffect` may be appropriate.

That is an advanced timing distinction.

Do not use `useLayoutEffect` by default.

---

# 33. Scrolling with a Ref

```jsx
function Messages() {
  const endRef =
    useRef(null);

  function scrollToBottom() {
    endRef.current
      ?.scrollIntoView({
        behavior: "smooth",
      });
  }

  return (
    <>
      <div>
        {/* messages */}
        <div ref={endRef} />
      </div>

      <button
        onClick={scrollToBottom}
      >
        Latest message
      </button>
    </>
  );
}
```

This is an appropriate imperative DOM operation.

---

# 34. Managing Multiple DOM Nodes

One `useRef` is convenient for a single node.

For dynamic lists, you may need a collection of nodes.

Conceptually:

```jsx
const itemRefs =
  useRef(new Map());
```

Then callback refs can register and unregister individual elements.

This is more advanced and should be used only when direct node access is genuinely required.

---

# 35. Callback Refs Preview

Instead of passing a ref object:

```jsx
<div ref={myRef} />
```

React can also receive a ref callback:

```jsx
<div
  ref={(node) => {
    // node when attached
    // null when detached
  }}
/>
```

Modern React also supports cleanup behavior for ref callbacks.

This becomes useful for dynamic DOM collections and more advanced imperative integrations.

Ref handling is covered further in Lesson 25 and the React 19 ref lesson.

---

# 36. Refs and Controlled Inputs ⭐⭐⭐⭐⭐

Controlled:

```jsx
const [name, setName] =
  useState("");

<input
  value={name}
  onChange={(event) =>
    setName(event.target.value)
  }
/>
```

React state is the source of truth.

A ref can read an uncontrolled input:

```jsx
const inputRef =
  useRef(null);

function handleSubmit() {
  console.log(
    inputRef.current.value
  );
}

<input ref={inputRef} />
```

But do not choose refs merely to avoid state.

Choose controlled or uncontrolled patterns according to the component requirements.

---

# 37. Ref Mutation Does Not Cause Reconciliation

State update:

```text
setState
   ↓
render
   ↓
reconciliation
   ↓
possible commit
```

Ref mutation:

```text
ref.current = value
   ↓
no render caused
   ↓
no reconciliation caused
```

This explains both the power and limitation of refs.

---

# 38. Refs and Strict Mode ⭐⭐⭐⭐⭐

In development Strict Mode, React may call a component function more than once to help detect impurities.

Each development-only render may create ref objects internally, but one version is discarded.

Your component should not depend on impure render-time ref mutation.

This reinforces:

> Keep rendering pure.

Use refs mainly outside render except for safe predictable initialization patterns.

---

# 39. Real-World CodeBuddy Example: Auto-Focus Chat Input

```jsx
function ChatComposer({
  conversationId,
}) {
  const inputRef =
    useRef(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, [conversationId]);

  return (
    <input
      ref={inputRef}
      placeholder="Write a message..."
    />
  );
}
```

When the conversation changes:

```text
conversationId changes
        ↓
render/commit
        ↓
Effect
        ↓
focus message input
```

The ref gives imperative access to the actual input element.

---

# 40. Real-World Example: Prevent Duplicate Timer Creation

```jsx
function SearchBox() {
  const timeoutRef =
    useRef(null);

  function handleChange(event) {
    const query =
      event.target.value;

    clearTimeout(
      timeoutRef.current
    );

    timeoutRef.current =
      setTimeout(() => {
        search(query);
      }, 400);
  }

  return (
    <input
      onChange={handleChange}
    />
  );
}
```

The timer ID:

- must persist between events
- does not need to render

Therefore:

```text
useRef
```

is appropriate.

Note that this is a manual debounce-style example; production search may use more complete abstractions.

---

# 41. Common Mistakes ⭐⭐⭐⭐⭐

## Mistake 1 — Using a ref for visible state

If changing the value should update UI, use state.

---

## Mistake 2 — Expecting ref mutation to re-render

```jsx
ref.current++;
```

does not schedule a React render.

---

## Mistake 3 — Using a normal variable for persistent mutable data

Normal variables are recreated on each render.

---

## Mistake 4 — Mutating refs arbitrarily during render

Rendering should remain pure.

---

## Mistake 5 — Using refs to bypass Effect dependencies

If a value should cause re-synchronization, keep it reactive.

---

## Mistake 6 — Manipulating DOM that React should control declaratively

Use refs for necessary imperative operations, not as a replacement for React rendering.

---

## Mistake 7 — Assuming refs are global

Each component instance receives its own ref.

---

## Mistake 8 — Expecting ref.current in a dependency array to behave like state

Ref mutation does not trigger rendering.

---

## Mistake 9 — Using refs everywhere because they seem faster

Refs bypass React's rendering mechanism. Use them only when reactivity is not required.

---

## Mistake 10 — Forgetting null DOM refs

A DOM ref may be `null` before attachment or after removal.

---

# 42. Interview Questions ⭐⭐⭐⭐⭐

## Q1. What is useRef?

**Answer:** `useRef` is a Hook that returns a stable mutable object with a `current` property that persists across renders.

---

## Q2. Does changing ref.current cause a re-render?

**Answer:** No. Ref mutation does not schedule a React render.

---

## Q3. What are the two major uses of useRef?

**Answer:** Storing mutable non-rendering values across renders and accessing DOM nodes or other imperative handles.

---

## Q4. What is the difference between useState and useRef?

**Answer:** Both persist across renders, but state updates schedule rendering while ref mutations do not. State represents reactive UI data; refs hold non-reactive mutable data.

---

## Q5. Why not use a normal variable instead of useRef?

**Answer:** A normal local variable is recreated during each render, while React preserves the ref object across renders.

---

## Q6. When should a value be state instead of a ref?

**Answer:** When changing that value should cause the rendered UI to update.

---

## Q7. Can refs help with stale closures?

**Answer:** Yes. A long-lived callback can read a stable ref's latest `current` value, but refs should not be used to hide dependencies that should genuinely trigger synchronization.

---

## Q8. Is ref.current reactive?

**Answer:** No. React does not schedule rendering when `ref.current` changes.

---

## Q9. When is a DOM ref populated?

**Answer:** React assigns the DOM node to the ref during the commit process after the element is created/attached.

---

## Q10. Should you read or write refs during render?

**Answer:** Generally no, because rendering should be pure. Predictable one-time lazy initialization is a special exception.

---

## Q11. Can each component instance have a separate ref?

**Answer:** Yes. React preserves refs per component instance.

---

## Q12. What happens to a DOM ref when the node is removed?

**Answer:** React clears it, typically setting `current` back to `null`.

---

# 43. Interview Scenario 1 ⭐⭐⭐⭐⭐

Why does this UI not update?

```jsx
function Counter() {
  const count =
    useRef(0);

  return (
    <button
      onClick={() => {
        count.current++;
      }}
    >
      {count.current}
    </button>
  );
}
```

Because:

```text
count.current changes
        ↓
no render scheduled
        ↓
JSX is not recalculated
```

Use state if the number should appear updated on screen.

---

# 44. Interview Scenario 2 ⭐⭐⭐⭐⭐

Which should store a timeout ID?

```text
state
or
ref?
```

Usually:

```text
ref
```

because the timer ID:

- must persist
- does not affect rendering

---

# 45. Interview Scenario 3 ⭐⭐⭐⭐⭐

You need to focus an input after it appears.

Use:

```text
useRef
+
Effect when focus depends
on the committed render
```

Example:

```jsx
const inputRef =
  useRef(null);

useEffect(() => {
  inputRef.current?.focus();
}, []);

return (
  <input ref={inputRef} />
);
```

---

# 46. Interview Scenario 4 ⭐⭐⭐⭐⭐

An old timeout callback must read the latest count.

Possible design:

```jsx
const latestCount =
  useRef(count);

useEffect(() => {
  latestCount.current =
    count;
}, [count]);

function handleClick() {
  setTimeout(() => {
    console.log(
      latestCount.current
    );
  }, 3000);
}
```

But first ask whether you actually need:

```text
latest value
```

or the:

```text
value captured when
the event occurred
```

Both are valid requirements.

---

# 47. Complete Mental Model ⭐⭐⭐⭐⭐

```text
                  Need to remember a value
                          │
                          ↓
              Must it survive renders?
                    │             │
                   no            yes
                    │             │
                    ↓             ↓
             normal variable   Should changing
                                it update UI?
                                  │       │
                                 yes      no
                                  │       │
                                  ↓       ↓
                                state    ref
                                         │
                              ┌──────────┴──────────┐
                              ↓                     ↓
                     mutable internal          imperative
                         value                DOM/resource
                              │                     │
                              ↓                     ↓
                         ref.current          element ref
```

---

# 48. State, Ref and Closure Together ⭐⭐⭐⭐⭐

```text
STATE
─────
Render #1 → count = 0
Render #2 → count = 1
Render #3 → count = 2

Old callback from Render #1
captures count = 0


REF
───
stable ref object
       │
       ├── current = 0
       ├── current = 1
       └── current = 2

Old callback from Render #1
can access same ref object
       ↓
reads current latest value
```

This diagram connects Lessons 13, 23 and 24.

---

# 49. Quick Revision ⭐⭐⭐⭐⭐

Remember:

```jsx
const ref =
  useRef(initialValue);
```

Read:

```jsx
ref.current
```

Write:

```jsx
ref.current = value;
```

Changing it:

```text
does NOT trigger render
```

Use state when:

```text
value affects UI
```

Use ref when:

```text
value must persist
+
changing it should not render
```

Typical ref uses:

```text
DOM nodes
timer IDs
latest mutable values
previous values
imperative resources
instance-like data
```

---

# 50. Key Takeaways

- `useRef` returns a stable object with a mutable `current` property.
- React preserves the ref object across renders of the same component instance.
- Changing `ref.current` does not schedule a re-render.
- State and refs both persist, but only state is designed to drive rendering.
- Use state when a value affects what the user sees.
- Use refs for mutable information that must persist without triggering rendering.
- A normal local variable does not reliably persist across renders.
- Refs are commonly used for timer IDs and imperative external handles.
- Passing a ref to a DOM element gives access to the actual node after commit.
- DOM refs are useful for focus, scrolling, measurement, media control, and integrations.
- Prefer React's declarative model over unnecessary manual DOM manipulation.
- Read and write refs mainly outside rendering; keep rendering pure.
- Predictable one-time lazy ref initialization is a special render-time exception.
- Ref mutation is immediately visible through `ref.current`, unlike state snapshots.
- Refs can help long-lived callbacks access a latest mutable value.
- Do not use refs to hide genuine Effect dependencies.
- `ref.current` is not reactive and changing it does not make an Effect re-run.
- Each component instance gets its own ref.
- DOM refs can temporarily be `null`.
- The key decision is: **Should changing this value cause React to render? If yes, use state; if no but it must persist, consider a ref.**

---

## Next Lesson

➡️ [Lesson 25 — DOM Refs and useImperativeHandle](./25-dom-refs-imperative-handle.md)
