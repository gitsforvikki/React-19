# Lesson 13 — State as a Snapshot ⭐⭐⭐⭐⭐

## 1. What Does "State as a Snapshot" Mean?

A very important React rule is:

> **State behaves like a snapshot for each render.**

When React renders a component, it gives that render a particular state value.

That value does not change inside the already-running render or its event handlers just because you call a state setter.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    console.log(count);

    setCount(count + 1);

    console.log(count);
  }

  return (
    <button onClick={handleClick}>
      {count}
    </button>
  );
}
```

On the first click:

```text
0
0
```

not:

```text
0
1
```

The handler belongs to the render where `count = 0`.

Calling `setCount` requests another render. It does not rewrite the `count` variable in the current render.

---

## 2. The Correct Mental Model ⭐⭐⭐⭐⭐

Do not imagine:

```text
count = 0
setCount(1)
count immediately becomes 1
```

Instead:

```text
Render #1
count = 0
   │
   ├── setCount(1)
   │
   └── current count remains 0
             ↓
       React schedules update
             ↓
Render #2
count = 1
```

Each render receives its own state snapshot.

---

## 3. Every Render Has Its Own Values ⭐⭐⭐⭐⭐

Imagine:

```text
Render #1 → count = 0
Render #2 → count = 1
Render #3 → count = 2
```

Conceptually:

```js
// Mental model only

function CounterRender1() {
  const count = 0;
}

function CounterRender2() {
  const count = 1;
}

function CounterRender3() {
  const count = 2;
}
```

React executes the component again rather than mutating the previous render's local variables.

---

## 4. Rendering Creates a UI Snapshot ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

One render can be imagined as:

```text
count = 0

UI:
Count: 0

handler:
setCount(0 + 1)
```

After the update:

```text
count = 1

UI:
Count: 1

handler:
setCount(1 + 1)
```

Each render creates UI and handlers using that render's values.

---

## 5. State Lives in React ⭐⭐⭐⭐⭐

When you write:

```jsx
const [count, setCount] = useState(0);
```

a useful conceptual model is:

```text
React
  │
  └── stores component state
          │
          ↓
Component render
          │
          └── receives count snapshot
```

The local `count` variable is the value React provides for that render.

The setter asks React to update the stored state and render again.

---

## 6. Why the Current Variable Does Not Change

```jsx
function handleClick() {
  setCount(10);

  console.log(count);
}
```

If this handler was created when:

```text
count = 5
```

then it still sees 5.

The setter requests:

```text
future render → count = 10
```

It does not mutate:

```text
current render → count = 5
```

---

## 7. Multiple Direct Updates ⭐⭐⭐⭐⭐

```jsx
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
}
```

If the current snapshot is:

```text
count = 0
```

all three expressions calculate:

```text
setCount(0 + 1)
setCount(0 + 1)
setCount(0 + 1)
```

or conceptually:

```text
setCount(1)
setCount(1)
setCount(1)
```

Do not expect this to mean +3.

---

## 8. Functional Updaters ⭐⭐⭐⭐⭐

When each update depends on previous pending state:

```jsx
function handleClick() {
  setCount((count) => count + 1);
  setCount((count) => count + 1);
  setCount((count) => count + 1);
}
```

React can process:

```text
0
↓ +1
1
↓ +1
2
↓ +1
3
```

The final state can become 3.

This connects directly to the update queue in Lesson 14.

---

## 9. Event Handlers Capture Render Values ⭐⭐⭐⭐⭐

Every render creates handlers using that render's values.

Conceptually:

```text
Render #1
count = 0
handler sees 0

Render #2
count = 1
handler sees 1

Render #3
count = 2
handler sees 2
```

This behavior comes from JavaScript **closures**.

---

## 10. Snapshot + setTimeout ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleShowLater() {
    setTimeout(() => {
      alert(count);
    }, 3000);
  }

  return (
    <>
      <p>{count}</p>

      <button onClick={handleShowLater}>
        Show count in 3 seconds
      </button>

      <button onClick={() => setCount((c) => c + 1)}>
        Increment
      </button>
    </>
  );
}
```

Suppose count is 5.

You click **Show count in 3 seconds**, then immediately increment to 6.

The old timer callback can still alert:

```text
5
```

because it closed over the render where count was 5.

---

## 11. Snapshot Timeline ⭐⭐⭐⭐⭐

```text
Time 0
Render A
count = 5
   │
   └── timer callback created
       and sees count = 5

Time 1
increment
   ↓
setCount(...)
   ↓
Render B
count = 6

Time 3
old timer callback runs
   ↓
still sees count = 5
```

This is standard JavaScript closure behavior combined with React render snapshots.

---

## 12. Closures and React State ⭐⭐⭐⭐⭐

A closure remembers variables from the scope where a function was created.

Plain JavaScript:

```js
function createLogger(value) {
  return function log() {
    console.log(value);
  };
}

const logFive = createLogger(5);

logFive(); // 5
```

React uses the same JavaScript behavior.

```text
Render
  │
  ├── props snapshot
  ├── state snapshot
  └── handlers close over those values
```

Understanding closures is essential for mastering React.

---

## 13. Async Functions Also Capture Render Values ⭐⭐⭐⭐⭐

```jsx
function Search() {
  const [query, setQuery] = useState("");

  async function handleSearch() {
    const submittedQuery = query;

    await someAsyncOperation();

    console.log(query);
    console.log(submittedQuery);
  }

  // ...
}
```

The handler belongs to a particular render.

Even if newer renders occur while `await` is pending, the old function closure still references values from the render that created it.

This becomes important for:

- stale closures
- race conditions
- Effects
- request cancellation

---

## 14. Props Are Snapshots Too ⭐⭐⭐⭐⭐

```jsx
function UserCard({ user }) {
  function handleClick() {
    setTimeout(() => {
      console.log(user.name);
    }, 2000);
  }

  return (
    <button onClick={handleClick}>
      Log User Later
    </button>
  );
}
```

The callback closes over the `user` prop from that render.

Useful mental model:

```text
Each render receives:

props snapshot
+
state snapshot
+
context values
```

Functions created during the render can close over them.

---

## 15. Snapshot Does Not Mean Deep Clone

The word **snapshot** is a mental model.

React does not deep-clone every state object.

```jsx
const [user, setUser] = useState({
  name: "Vikash",
});
```

The state value contains an object reference.

This is one reason mutation is dangerous.

Treat props and state as immutable.

---

## 16. Immutability Supports Snapshot Semantics ⭐⭐⭐⭐⭐

Imagine:

```text
Render A
user ───→ Object A
```

If you later mutate Object A, old code holding that reference can observe the mutation.

Instead:

```jsx
setUser((current) => ({
  ...current,
  name: "Changed",
}));
```

Conceptually:

```text
Render A
user ───→ Object A

Render B
user ───→ Object B
```

Previous values remain easier to reason about.

---

## 17. Predict the Next Value Locally

Sometimes you need the next value immediately:

```jsx
function handleClick() {
  const nextCount = count + 1;

  setCount(nextCount);

  console.log(nextCount);
}
```

Here:

```text
count
  ↓
current snapshot

nextCount
  ↓
locally calculated next value
```

Do not expect `count` itself to change inside that handler.

---

## 18. Replacement and Updater Functions ⭐⭐⭐⭐⭐

Consider:

```jsx
setCount(count + 5);
setCount((n) => n + 1);
```

If:

```text
count = 0
```

conceptually:

```text
replace with 5
      ↓
updater receives 5
      ↓
5 + 1
      ↓
6
```

Now reverse them:

```jsx
setCount((n) => n + 1);
setCount(count + 5);
```

The second expression was calculated from the render snapshot:

```text
count + 5
= 0 + 5
= 5
```

Conceptually:

```text
updater: 0 → 1
then
replace with 5
```

The final state can therefore be 5.

Lesson 14 explains this queue behavior deeply.

---

## 19. Setter vs Variable Assignment ⭐⭐⭐⭐⭐

Normal JavaScript:

```js
let count = 0;

count = 1;
```

The variable changes immediately.

React:

```jsx
const [count, setCount] = useState(0);

setCount(1);
```

Interpret this as:

```text
React, please use 1 as state
for an upcoming render.
```

Not:

```text
Change this current count variable to 1.
```

---

## 20. Snapshots Across Separate User Actions

The snapshot concept does not mean state never progresses.

Example:

```text
Render 1
count = 0
click → request 1

Render 2
count = 1
click → request 2

Render 3
count = 2
```

Each completed render gives later interactions handlers based on the newer state.

The important rule is:

> Values are fixed within a particular render.

---

## 21. Snapshots and Effects

Effects also close over values from the render that created them.

Later you will see code such as:

```jsx
useEffect(() => {
  // closes over values from a render
}, []);
```

Incorrect dependency handling can cause an Effect to keep using older values.

This is the foundation of the **stale closure** problem covered in Lesson 23.

---

## 22. An Older Captured Value Is Not Always Wrong ⭐⭐⭐⭐⭐

Suppose a user submits a message.

You may want an async request to use exactly the message that existed at submission time.

```jsx
function handleSubmit() {
  const submittedMessage = message;

  sendMessage(submittedMessage);
}
```

The correct question is:

```text
Should this work use:

the value at interaction time?
        OR
the latest value at execution time?
```

Those are different requirements.

---

## 23. When You Need the Latest Mutable Value

Sometimes asynchronous code genuinely needs the latest value rather than a render snapshot.

A ref can be useful in specific cases.

Conceptually:

```jsx
const latestCountRef = useRef(count);
latestCountRef.current = count;
```

Later code can read:

```js
latestCountRef.current
```

But do not replace ordinary UI state with refs just to avoid snapshot behavior.

State and refs solve different problems.

Lesson 24 covers `useRef`.

---

## 24. Snapshot Semantics Improve UI Consistency ⭐⭐⭐⭐⭐

During one render, React can reason from a consistent set of inputs:

```text
props
+
state
+
context
      ↓
render
      ↓
UI
```

A simplified React idea is:

```text
UI = function(props, state, context)
```

Stable render snapshots support predictable declarative UI.

---

## 25. Real-World Example: Delayed Connection Request

```jsx
function DeveloperCard({ developer }) {
  const [message, setMessage] = useState("");

  function handleConnect() {
    const messageAtSubmit = message;

    setTimeout(() => {
      console.log(
        "Sending to:",
        developer.id,
        messageAtSubmit
      );
    }, 2000);
  }

  return (
    <>
      <input
        value={message}
        onChange={(event) =>
          setMessage(event.target.value)
        }
      />

      <button onClick={handleConnect}>
        Connect
      </button>
    </>
  );
}
```

The callback uses values associated with the interaction that created it.

For many request flows, preserving the submitted value is desirable.

---

## 26. Common Mistakes

### Mistake 1 — Expecting immediate state mutation

```jsx
setCount(count + 1);
console.log(count);
```

The log sees the current snapshot.

### Mistake 2 — Repeated direct updates for sequential calculations

All direct calculations can read the same snapshot.

### Mistake 3 — Forgetting JavaScript closures

Timers and async callbacks can retain values from older renders.

### Mistake 4 — Mutating state objects

Mutation makes render snapshots harder to reason about and conflicts with React's immutable state model.

### Mistake 5 — Thinking props behave differently

Props are also render-specific inputs.

### Mistake 6 — Using refs everywhere for latest values

Refs are for specific mutable-value use cases, not a replacement for state.

### Mistake 7 — Assuming an old captured value is always a bug

Sometimes preserving the original interaction value is correct.

---

## 27. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What does "state is a snapshot" mean?

**Answer:** Each render receives a fixed state value. Calling a setter requests a future render but does not change the state variable inside the current render.

### Q2. Why does console.log show old state after setState?

**Answer:** The running handler uses the state snapshot from the render that created it.

### Q3. Why do three setCount(count + 1) calls not necessarily increment by three?

**Answer:** They all calculate from the same `count` snapshot. Use updater functions for sequential previous-state calculations.

### Q4. What is a functional updater?

**Answer:** A function passed to a setter, such as `setCount(c => c + 1)`, which React applies to the pending state value.

### Q5. Why can setTimeout see an older state value?

**Answer:** The callback is a JavaScript closure created during an earlier render and retains that render's state values.

### Q6. Are props snapshots too?

**Answer:** Yes. Each render receives particular prop values and functions created by that render can close over them.

### Q7. Does snapshot mean React deep-copies state?

**Answer:** No. Snapshot is a mental model. Objects are still references, which is why immutable updates matter.

### Q8. How can you use the next calculated value immediately?

**Answer:** Calculate it in a local variable, use that variable immediately, and also pass it to the setter.

### Q9. When should you use a functional updater?

**Answer:** When the next state depends on previous or pending state.

### Q10. How are snapshots related to stale closures?

**Answer:** Functions close over render-specific values. If they execute later after newer renders exist, their captured values can be older than the latest state.

---

## 28. Interview Scenario ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);

    setTimeout(() => {
      console.log(count);
    }, 3000);
  }

  return (
    <button onClick={handleClick}>
      {count}
    </button>
  );
}
```

If the button initially displays 0 and the user clicks once, the timeout callback from that render can log:

```text
0
```

Timeline:

```text
Render A
count = 0
   ↓
click
   ↓
setCount(1)
   ↓
timer captures count = 0
   ↓
Render B
count = 1
   ↓
timer fires
   ↓
logs 0
```

---

## 29. Complete Mental Model ⭐⭐⭐⭐⭐

```text
React stores state
      │
      ↓
Render #1
state snapshot = A
      │
      ├── UI based on A
      ├── handlers capture A
      │
      └── setState(B)
              │
              ↓
       schedule update
              │
              ↓
Render #2
state snapshot = B
      │
      ├── UI based on B
      └── handlers capture B

Old callback → can still see A
New callback → sees B
```

---

## 30. Quick Revision

```text
State setter
     ≠
mutating current variable

State setter
     =
requesting future render
```

Each render has:

```text
props snapshot
state snapshot
derived values
handlers/closures
```

Repeated direct updates:

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

can all use the same snapshot.

Sequential updates:

```jsx
setCount((c) => c + 1);
setCount((c) => c + 1);
setCount((c) => c + 1);
```

use pending values through React's update queue.

---

## 31. Key Takeaways

- Every React render receives a particular state snapshot.
- Calling a setter does not change the current render's state variable.
- A setter requests an update for a future render.
- Each render creates UI and handlers based on its own values.
- JavaScript closures let event handlers, timers, and async callbacks retain render-specific values.
- Multiple direct updates can read the same snapshot.
- Use functional updater syntax when the next state depends on previous pending state.
- Props are also render-specific inputs.
- Snapshot does not mean React deep-clones objects.
- Immutability makes snapshot behavior predictable.
- Older captured values are not automatically bugs; sometimes they correctly represent the original interaction.
- Refs can provide access to latest mutable values in specific cases, but they do not replace normal state.
- Understanding snapshots is essential for batching, Effects, stale closures, and concurrent React.

---

## Next Lesson

➡️ [Lesson 14 — Batching and State Update Queue ⭐⭐⭐⭐⭐](./14-batching-state-queue.md)
