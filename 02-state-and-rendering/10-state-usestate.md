# Lesson 10 — State and useState ⭐⭐⭐⭐⭐

## 1. What Is State?

**State** is data that a React component remembers between renders and that can change over time.

Examples:

- counter value
- selected tab
- whether a modal is open
- form input
- cart items
- current filter
- logged-in user information

Example:

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>{count}</p>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

Mental model:

```text
Component State
      ↓
Render UI
      ↓
User Interaction
      ↓
State Update
      ↓
React Renders Again
      ↓
Updated UI
```

---

## 2. Why Normal Variables Are Not Enough ⭐⭐⭐⭐⭐

Consider:

```jsx
function Counter() {
  let count = 0;

  function handleClick() {
    count = count + 1;
    console.log(count);
  }

  return (
    <>
      <p>{count}</p>
      <button onClick={handleClick}>
        Increment
      </button>
    </>
  );
}
```

The JavaScript variable changes, but React is not told that the UI needs another render.

Also, when the component renders again:

```js
let count = 0;
```

runs again and creates a fresh local variable.

A normal local variable therefore has two important limitations:

```text
1. Changing it does not request a React render.
2. It does not persist as component state between renders.
```

State solves both problems.

---

## 3. useState Syntax ⭐⭐⭐⭐⭐

Import:

```jsx
import { useState } from "react";
```

Declare state:

```jsx
const [count, setCount] = useState(0);
```

Breakdown:

```text
useState(0)
    │
    └── initial state

count
    │
    └── state value for this render

setCount
    │
    └── function used to request an update
```

The array destructuring names are your choice:

```jsx
const [isOpen, setIsOpen] = useState(false);
const [user, setUser] = useState(null);
const [products, setProducts] = useState([]);
```

Convention:

```text
value       → user
setter      → setUser
```

---

## 4. State Belongs to a Component Instance ⭐⭐⭐⭐⭐

Suppose:

```jsx
function App() {
  return (
    <>
      <Counter />
      <Counter />
    </>
  );
}
```

Each rendered `Counter` has independent state.

```text
App
├── Counter → count = 2
└── Counter → count = 7
```

State is not simply attached to the component function definition.

A useful mental model is that React associates state with a component's identity/position in the rendered tree.

We will explore this deeply in Lesson 18.

---

## 5. Updating State

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleIncrement() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleIncrement}>
      Count: {count}
    </button>
  );
}
```

Flow:

```text
count = 0
   ↓
render
   ↓
button shows 0
   ↓
click
   ↓
setCount(1)
   ↓
React schedules update
   ↓
new render
   ↓
count = 1
```

---

## 6. Setting State Does Not Change the Current Render's Value ⭐⭐⭐⭐⭐

This is extremely important.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    console.log(count); // 0

    setCount(count + 1);

    console.log(count); // still 0
  }

  return <button onClick={handleClick}>{count}</button>;
}
```

Why?

The `count` variable belongs to the render in which the handler was created.

Calling:

```js
setCount(1);
```

requests another render.

It does not rewrite the `count` variable inside the already-running handler.

Mental model:

```text
Current Render
count = 0
   │
   ├── setCount(1)
   │       ↓
   │   request next render
   │
   └── count is still 0 here

Next Render
count = 1
```

This leads directly to **state as a snapshot**, covered deeply in Lesson 13.

---

## 7. State Updates Trigger Rendering

When state changes:

```text
setState(...)
     ↓
React queues/schedules update
     ↓
component renders
     ↓
new JSX description
     ↓
React reconciles
     ↓
required DOM changes committed
```

Important:

> A state update causes React rendering work; it does not mean React blindly recreates the entire DOM.

---

## 8. Do Not Mutate State Directly ⭐⭐⭐⭐⭐

Wrong:

```jsx
const [count, setCount] = useState(0);

count = count + 1;
```

State values should be treated as read-only for a render.

Use the setter:

```jsx
setCount(count + 1);
```

For objects and arrays, avoid mutating the existing state object too.

Wrong:

```jsx
user.name = "Vikash";
```

Instead create a new value and pass it to the setter.

---

## 9. Functional State Updates ⭐⭐⭐⭐⭐

Suppose the next state depends on the previous state.

Instead of:

```jsx
setCount(count + 1);
```

you can write:

```jsx
setCount((current) => current + 1);
```

The updater receives the pending/current state value React uses for processing the update.

Example:

```jsx
function handleClick() {
  setCount((current) => current + 1);
}
```

Common names:

```jsx
setCount((prev) => prev + 1);
setCount((current) => current + 1);
```

Both are conventions; choose a clear name.

---

## 10. Why Functional Updates Matter ⭐⭐⭐⭐⭐

Consider:

```jsx
function handleClick() {
  setCount(count + 1);
  setCount(count + 1);
  setCount(count + 1);
}
```

If `count` is `0`, all three expressions use the same render snapshot:

```text
setCount(0 + 1)
setCount(0 + 1)
setCount(0 + 1)
```

You should not expect this to mean `+3`.

When each update depends on the previous pending value:

```jsx
function handleClick() {
  setCount((c) => c + 1);
  setCount((c) => c + 1);
  setCount((c) => c + 1);
}
```

React can process the updater functions in sequence:

```text
0
↓ +1
1
↓ +1
2
↓ +1
3
```

This connects to React's update queue and batching, covered deeply in Lesson 14.

---

## 11. Direct Value vs Updater Function

### Direct value

```jsx
setCount(10);
```

Use when the next value does not need to be calculated from previous state.

Example:

```jsx
setIsOpen(false);
```

### Updater function

```jsx
setCount((count) => count + 1);
```

Use when the next state is calculated from previous state.

Useful rule:

```text
Next state depends on previous state?
        ↓
Use updater function
```

---

## 12. State Can Hold Different Types

### Number

```jsx
const [count, setCount] = useState(0);
```

### String

```jsx
const [query, setQuery] = useState("");
```

### Boolean

```jsx
const [isOpen, setIsOpen] = useState(false);
```

### Object

```jsx
const [user, setUser] = useState({
  name: "",
  role: "",
});
```

### Array

```jsx
const [items, setItems] = useState([]);
```

### null

```jsx
const [selectedUser, setSelectedUser] = useState(null);
```

State can hold JavaScript values.

---

## 13. Updating Object State ⭐⭐⭐⭐⭐

Suppose:

```jsx
const [user, setUser] = useState({
  name: "Vikash",
  role: "Frontend Developer",
});
```

Do not mutate:

```jsx
user.role = "Full Stack Developer"; // ❌
setUser(user);
```

Create a new object:

```jsx
setUser({
  ...user,
  role: "Full Stack Developer",
});
```

Better when based on previous state:

```jsx
setUser((current) => ({
  ...current,
  role: "Full Stack Developer",
}));
```

The spread keeps unchanged properties while creating a new object.

---

## 14. useState Does Not Automatically Merge Objects ⭐⭐⭐⭐⭐

This surprises developers familiar with older class-component `setState`.

Suppose:

```jsx
const [user, setUser] = useState({
  name: "Vikash",
  role: "Developer",
});
```

If you do:

```jsx
setUser({
  role: "Full Stack Developer",
});
```

the new state is effectively:

```js
{
  role: "Full Stack Developer"
}
```

React's `useState` setter replaces the state value you provide; it does not shallow-merge object fields for you.

To preserve fields:

```jsx
setUser((current) => ({
  ...current,
  role: "Full Stack Developer",
}));
```

---

## 15. Updating Array State ⭐⭐⭐⭐⭐

Suppose:

```jsx
const [skills, setSkills] = useState([
  "JavaScript",
  "React",
]);
```

### Add

```jsx
setSkills((current) => [
  ...current,
  "Node.js",
]);
```

### Remove

```jsx
setSkills((current) =>
  current.filter((skill) => skill !== "React")
);
```

### Replace/update item

```jsx
setUsers((current) =>
  current.map((user) =>
    user.id === updatedUser.id
      ? updatedUser
      : user
  )
);
```

Avoid:

```jsx
skills.push("Node.js");
setSkills(skills);
```

Treat arrays in state as immutable.

Lesson 15 will cover object/array updates deeply.

---

## 16. State Initialization

Basic:

```jsx
const [count, setCount] = useState(0);
```

The initial value is used when the component state is first initialized.

But consider expensive initialization:

```jsx
const [data, setData] = useState(
  createExpensiveInitialData()
);
```

The function call expression executes whenever the component function renders, even though React only needs its result for initialization.

For expensive initialization, pass an initializer function:

```jsx
const [data, setData] = useState(
  createExpensiveInitialData
);
```

or:

```jsx
const [data, setData] = useState(() => {
  return createExpensiveInitialData();
});
```

This is called **lazy initialization**.

---

## 17. Initializer Function vs Storing a Function ⭐⭐⭐⭐⭐

Important distinction.

This:

```jsx
useState(createInitialValue)
```

treats the function as an initializer.

If you actually want the state value itself to be a function, wrap it:

```jsx
const [fn, setFn] = useState(() => myFunction);
```

Similarly, when setting state to a function value, remember that React interprets a function passed directly to a state setter as an updater.

To store a function:

```jsx
setFn(() => myFunction);
```

This is less common, but a useful interview edge case.

---

## 18. Multiple State Variables

A component can have multiple state variables:

```jsx
function ProfileEditor() {
  const [name, setName] = useState("");
  const [role, setRole] = useState("");
  const [isSaving, setIsSaving] = useState(false);

  // ...
}
```

This is perfectly valid.

State does not need to be one giant object.

A good rule is to group values when they naturally belong together and are commonly updated together, but do not combine unrelated state just because you can.

---

## 19. Independent vs Related State

Independent:

```jsx
const [query, setQuery] = useState("");
const [isSidebarOpen, setIsSidebarOpen] = useState(false);
```

These represent different concerns.

Related object:

```jsx
const [profile, setProfile] = useState({
  firstName: "",
  lastName: "",
});
```

There is no universal rule.

Ask:

```text
Do these values represent one logical thing?
Do they usually change together?
Will grouping make updates clearer?
```

For more complex state transitions, `useReducer` may eventually become useful.

---

## 20. Avoid Redundant State ⭐⭐⭐⭐⭐

Suppose:

```jsx
const [firstName, setFirstName] = useState("Vikash");
const [lastName, setLastName] = useState("Kumar");
const [fullName, setFullName] = useState("Vikash Kumar");
```

`fullName` can be derived:

```jsx
const fullName = firstName + " " + lastName;
```

Better:

```jsx
const [firstName, setFirstName] = useState("Vikash");
const [lastName, setLastName] = useState("Kumar");

const fullName = `${firstName} ${lastName}`;
```

Mental model:

```text
firstName + lastName
        ↓
     fullName
```

Do not store data that can be calculated cheaply from existing state/props unless there is a real reason.

This avoids synchronization bugs.

---

## 21. Avoid Contradictory State

Potentially problematic:

```jsx
const [isLoading, setIsLoading] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```

This allows impossible combinations:

```text
isLoading = true
isSuccess = true
isError   = true
```

Sometimes one state variable is clearer:

```jsx
const [status, setStatus] = useState("idle");
```

Possible values:

```text
idle
loading
success
error
```

Good state design tries to make invalid states difficult to represent.

---

## 22. Avoid Duplicated State

Suppose you have:

```jsx
const [items, setItems] = useState(initialItems);
const [selectedItem, setSelectedItem] = useState(items[0]);
```

Now the same logical item may exist in two state locations.

If `items` changes, `selectedItem` can become stale.

Often store the ID:

```jsx
const [items, setItems] = useState(initialItems);
const [selectedId, setSelectedId] = useState(initialItems[0].id);

const selectedItem = items.find(
  (item) => item.id === selectedId
);
```

Now there is one authoritative collection.

---

## 23. State Should Be Minimal ⭐⭐⭐⭐⭐

A strong React principle:

> Store the minimum information needed, then derive everything else.

Example:

Instead of:

```text
cartItems
cartItemCount
cartSubtotal
hasItems
```

you may only need:

```text
cartItems
```

Then derive:

```jsx
const cartItemCount = cartItems.length;

const cartSubtotal = cartItems.reduce(
  (sum, item) => sum + item.price * item.quantity,
  0
);

const hasItems = cartItems.length > 0;
```

This reduces synchronization problems.

---

## 24. State Ownership ⭐⭐⭐⭐⭐

State should live in the component that needs to own/control it.

Example:

```text
ProductPage
├── FilterBar
└── ProductList
```

If both `FilterBar` and `ProductList` need the selected filter, a common design is:

```text
ProductPage owns filter
      │
      ├── filter + callback → FilterBar
      │
      └── filter → ProductList
```

This is related to **lifting state up**, covered in Lesson 17.

---

## 25. Local State Is Often Best

Do not automatically put every value into global state.

Examples of good local state:

- modal open/closed
- currently selected tab
- local form draft
- dropdown open/closed

If only one component or small subtree needs a value, keeping it local often makes the application easier to understand.

Mental model:

```text
Keep state as close as practical
to where it is actually needed.
```

---

## 26. State and Props Together

A component often receives props and owns state:

```jsx
function ProductCard({ product, onAddToCart }) {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <article>
      <h2>{product.name}</h2>

      <button
        onClick={() => setIsExpanded((open) => !open)}
      >
        {isExpanded ? "Hide details" : "Show details"}
      </button>

      {isExpanded && <p>{product.description}</p>}

      <button onClick={() => onAddToCart(product.id)}>
        Add to cart
      </button>
    </article>
  );
}
```

Ownership:

```text
product       → prop
onAddToCart   → prop
isExpanded    → local state
```

The parent owns product/cart behavior, while the card owns its local expanded UI state.

---

## 27. State Is Private to the Component Instance

A parent cannot directly access a child's local state variable.

```text
Parent
  ↓
Child
  └── local state
```

If the parent needs to control or share that value, the state may need to move upward.

This preserves explicit data ownership.

---

## 28. State Setter Identity

React guarantees that a `set` function returned by `useState` has a stable identity.

Example:

```jsx
const [count, setCount] = useState(0);
```

React does not create a conceptually unrelated setter every render.

This becomes useful later when reasoning about dependencies and memoization.

For now, remember that the setter is the supported mechanism for requesting state changes.

---

## 29. Setting the Same Value

If the new state is equivalent to the current state according to React's comparison behavior, React can skip unnecessary update work.

For example:

```jsx
setCount(5);
```

when state is already `5`.

React uses `Object.is` semantics when comparing the new state with the current state for this purpose.

This also helps explain why mutating an object and passing the same object reference is problematic.

---

## 30. Why Mutation Causes Problems ⭐⭐⭐⭐⭐

Suppose:

```jsx
const [user, setUser] = useState({
  name: "Vikash",
});
```

Wrong:

```jsx
user.name = "Aman";
setUser(user);
```

The reference is still the same:

```text
Before:
user ─────→ Object A

After mutation:
user ─────→ Object A
             name changed internally
```

React's state model works best when an update provides a new value/reference:

```jsx
setUser({
  ...user,
  name: "Aman",
});
```

Now:

```text
Old state ───→ Object A

New state ───→ Object B
```

Immutability also makes application behavior easier to reason about.

---

## 31. Batching Preview ⭐⭐⭐⭐⭐

React can group multiple state updates before rendering.

Example:

```jsx
function handleClick() {
  setName("Vikash");
  setRole("Full Stack Developer");
}
```

React does not necessarily render after every setter call individually.

It can batch updates and process them together.

Simplified:

```text
Event
 ↓
setName(...)
setRole(...)
 ↓
React processes queued updates
 ↓
render
```

Do not depend on state variables changing immediately after setters.

Batching gets a full lesson in **Lesson 14**.

---

## 32. useState Must Follow the Rules of Hooks ⭐⭐⭐⭐⭐

Correct:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  // ...
}
```

Wrong:

```jsx
function Counter({ enabled }) {
  if (enabled) {
    const [count, setCount] = useState(0); // ❌
  }
}
```

Also avoid calling Hooks inside ordinary loops or nested functions.

React relies on consistent Hook calling order.

The Rules of Hooks have a dedicated lesson later.

---

## 33. Real-World Example: Search Filter

```jsx
import { useState } from "react";

function DeveloperSearch({ developers }) {
  const [query, setQuery] = useState("");

  const filteredDevelopers = developers.filter((developer) =>
    developer.name
      .toLowerCase()
      .includes(query.toLowerCase())
  );

  return (
    <section>
      <input
        value={query}
        onChange={(event) => setQuery(event.target.value)}
        placeholder="Search developers"
      />

      {filteredDevelopers.map((developer) => (
        <p key={developer.id}>
          {developer.name}
        </p>
      ))}
    </section>
  );
}
```

Notice the state design:

```text
Stored state:
query

Derived value:
filteredDevelopers
```

We do not need another state variable for the filtered list.

---

## 34. Common Mistakes

### Mistake 1 — Using a normal variable for reactive UI data

Normal variables do not request React renders.

### Mistake 2 — Mutating state directly

Use the state setter and create new object/array values.

### Mistake 3 — Expecting state to change immediately

The state variable belongs to the current render snapshot.

### Mistake 4 — Multiple updates using stale snapshot values

When the next value depends on previous state, use an updater function.

### Mistake 5 — Assuming useState merges objects

It replaces the value you provide.

### Mistake 6 — Storing derived data unnecessarily

Calculate values from existing props/state when practical.

### Mistake 7 — Creating contradictory state

Design state so invalid combinations are difficult.

### Mistake 8 — Duplicating the same information in multiple state variables

Prefer a single source of truth.

### Mistake 9 — Making all state global

Keep state local when it is only needed locally.

### Mistake 10 — Calling useState conditionally

Hooks must follow stable calling rules.

---

## 35. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is state in React?

**Answer:** State is component-owned data that React preserves between renders and whose updates can trigger rendering.

### Q2. Why can't we use a normal variable instead of state?

**Answer:** Changing a normal local variable does not tell React to render again, and local variables are recreated when the component function executes again.

### Q3. What does useState return?

**Answer:** It returns an array containing the current state value for that render and a setter function used to request updates.

### Q4. Does calling a state setter immediately change the current state variable?

**Answer:** No. The current render's state value remains the same. The setter requests an update that React processes for a future render.

### Q5. When should you use a functional updater?

**Answer:** When the next state depends on the previous/pending state, for example `setCount(c => c + 1)`.

### Q6. Does useState merge objects?

**Answer:** No. The setter replaces the state value. If you need to retain other object properties, create a new object containing them.

### Q7. Why shouldn't state be mutated directly?

**Answer:** React expects state to be treated as immutable snapshots. Mutation can break predictable updates and reference-based comparisons.

### Q8. What is lazy state initialization?

**Answer:** Passing a function to `useState` so expensive initial-state computation is performed when state is initialized rather than evaluating that computation on every render.

### Q9. What is redundant state?

**Answer:** State that can be derived from existing props or other state, such as storing `fullName` when it can be calculated from `firstName` and `lastName`.

### Q10. Where should state live?

**Answer:** As close as practical to the components that need it, while moving it to a common owner when multiple components need to coordinate around the same value.

### Q11. What happens when state updates?

**Answer:** React queues/schedules the update, renders affected components, reconciles the resulting UI, and commits required DOM changes.

### Q12. Why can three setCount(count + 1) calls fail to increment by three?

**Answer:** Each expression can read the same state snapshot from the current render. Updater functions allow React to process sequential changes based on pending state.

---

## 36. Interview Scenario ⭐⭐⭐⭐⭐

### Question

What is the result of:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      {count}
    </button>
  );
}
```

A common incorrect answer is:

```text
3
```

All three calls use the same `count` value from that render.

For sequential increments:

```jsx
function handleClick() {
  setCount((count) => count + 1);
  setCount((count) => count + 1);
  setCount((count) => count + 1);
}
```

Updater queue:

```text
0
↓
1
↓
2
↓
3
```

This will become even clearer in Lessons 13 and 14.

---

## 37. State Design Checklist

Before creating state, ask:

```text
Does this value change over time?
        ↓
Does changing it need to affect rendering?
        ↓
Can it be derived from existing props/state?
        ↓
Who needs to own it?
        ↓
Can contradictory/duplicate state be avoided?
```

A useful rule:

> Do not add state merely because a value exists. Add state when the component needs React to remember changing information that affects its behavior or UI.

---

## 38. Quick Revision

```text
useState
   ↓
[value, setter]
```

```text
State Update Flow

Event
 ↓
setState
 ↓
update queued
 ↓
render
 ↓
reconciliation
 ↓
DOM changes if required
```

Important:

```text
State is remembered by React.
State belongs to a component instance.
State values are snapshots for a render.
Do not mutate state.
Use updater functions for previous-state calculations.
Keep state minimal.
Derive what you can.
Keep state close to where it is needed.
```

---

## 39. Key Takeaways

- State lets components remember changing information between renders.
- `useState` returns a state value and setter.
- Normal local variables are not a substitute for reactive component state.
- Calling a setter requests an update; it does not mutate the current render's state variable.
- State should be treated as immutable.
- Use functional updater syntax when the next state depends on previous state.
- `useState` does not automatically merge objects.
- Create new object/array values when updating state.
- Lazy initialization helps avoid repeating expensive initialization work.
- Avoid redundant, duplicated, and contradictory state.
- Store the minimum state needed and derive other values.
- Keep state local unless multiple components genuinely need shared ownership.
- React can batch multiple updates.
- `useState` must follow the Rules of Hooks.
- Understanding state snapshots and update queues is essential for mastering React rendering.

---

## Next Lesson

➡️ [Lesson 11 — Props vs State ⭐⭐⭐⭐⭐](./11-props-vs-state.md)
