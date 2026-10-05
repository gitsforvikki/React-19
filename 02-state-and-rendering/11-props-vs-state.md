# Lesson 11 — Props vs State ⭐⭐⭐⭐⭐

## 1. Why Props vs State Matters

Props and state are two of the most fundamental ways data appears inside a React component.

A component may:

- receive data through **props**
- remember its own changing data through **state**
- use both at the same time

Understanding the difference is essential for:

- component design
- deciding data ownership
- avoiding duplicated state
- lifting state correctly
- debugging re-renders
- React interviews

The most useful mental model is:

```text
Props  = data given to the component
State  = data owned/remembered by the component
```

---

## 2. What Are Props?

Props are inputs supplied by a component's caller.

```jsx
function UserCard({ name, role }) {
  return (
    <article>
      <h2>{name}</h2>
      <p>{role}</p>
    </article>
  );
}

function App() {
  return (
    <UserCard
      name="Vikash"
      role="Developer"
    />
  );
}
```

Ownership:

```text
App
 │
 │ owns/provides values
 ↓
UserCard
 │
 └── receives props
```

The child should treat those props as read-only.

---

## 3. What Is State?

State is data React remembers for a component instance.

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button
      onClick={() => setCount((c) => c + 1)}
    >
      {count}
    </button>
  );
}
```

Ownership:

```text
Counter
   │
   └── owns count state
```

The component requests changes using its setter.

---

## 4. Props vs State — Core Comparison ⭐⭐⭐⭐⭐

| Props | State |
|---|---|
| Received from caller | Owned by the component declaring it |
| Read-only to receiver | Updated through state setters |
| Configure a component | Stores changing component data |
| Parent/caller controls value | State owner controls value |
| Changes can cause rendering | Updates can cause rendering |
| Can contain data/functions/React nodes | Can contain JavaScript values |
| Useful for communication | Useful for memory |

Short interview answer:

> **Props are external inputs to a component, while state is component-owned data that React preserves between renders.**

---

## 5. A Component Can Use Both

```jsx
function ProductCard({ product }) {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <article>
      <h2>{product.name}</h2>

      <button
        onClick={() => setIsExpanded((value) => !value)}
      >
        {isExpanded ? "Hide" : "Show"} details
      </button>

      {isExpanded && (
        <p>{product.description}</p>
      )}
    </article>
  );
}
```

Here:

```text
product
   ↓
prop
   ↓
owned by caller

isExpanded
   ↓
state
   ↓
owned by ProductCard
```

This is a very common React design.

---

## 6. The Most Important Question: Who Owns the Data? ⭐⭐⭐⭐⭐

When deciding between props and state, ask:

> **Which component should be the source of truth for this value?**

Example:

```text
ProductPage
    │
    ├── FilterBar
    └── ProductList
```

Both children need the selected category.

A good design may be:

```text
             ProductPage
                  │
        owns selectedCategory
                  │
          ┌───────┴────────┐
          ↓                ↓
      FilterBar        ProductList
```

The parent owns the state.

The children receive it through props.

---

## 7. Single Source of Truth ⭐⭐⭐⭐⭐

Suppose both components store their own copy:

```text
FilterBar
selectedCategory = "React"

ProductList
selectedCategory = "Node"
```

Now the UI can disagree.

Instead:

```text
ProductPage
selectedCategory = "React"
       │
       ├──→ FilterBar
       └──→ ProductList
```

One owner means one authoritative value.

This is called a **single source of truth**.

Important:

> Single source of truth does not mean all application state must live in one global store.

It means each piece of state should have a clear authoritative owner.

---

## 8. Props Can Change

A common misconception:

> Props are immutable, therefore props never change.

That is incorrect.

Props are read-only **from the receiving component's perspective**.

The parent can render the child with a different value:

```jsx
function App() {
  const [name, setName] = useState("Vikash");

  return (
    <>
      <UserCard name={name} />

      <button onClick={() => setName("Aman")}>
        Change Name
      </button>
    </>
  );
}
```

Flow:

```text
Parent state changes
       ↓
Parent renders
       ↓
new prop value
       ↓
Child receives new props
```

So:

```text
Props are read-only
≠
Props never change
```

---

## 9. State Is Also Read-Only for a Render ⭐⭐⭐⭐⭐

Another misconception:

> State is mutable because we can update it.

React state should also be treated as immutable.

You do not mutate:

```jsx
user.name = "Aman"; // ❌
```

You request a new state value:

```jsx
setUser((current) => ({
  ...current,
  name: "Aman",
}));
```

Better distinction:

```text
Props
  → receiver cannot control/update ownership directly

State
  → owner can request updates through React
```

Both should be treated as read-only snapshots during rendering.

---

## 10. Parent State Becomes Child Props ⭐⭐⭐⭐⭐

This relationship is fundamental:

```jsx
function App() {
  const [count, setCount] = useState(0);

  return (
    <CounterDisplay count={count} />
  );
}

function CounterDisplay({ count }) {
  return <p>{count}</p>;
}
```

The same value is:

```text
State from App's perspective
          ↓
      count = 0
          ↓
passed to CounterDisplay
          ↓
Prop from CounterDisplay's perspective
```

Therefore:

> A value is not inherently "prop data" or "state data" everywhere. Its role depends on the component boundary.

---

## 11. State + Callback Props ⭐⭐⭐⭐⭐

Suppose a parent owns state but a child needs to request a change.

```jsx
function App() {
  const [count, setCount] = useState(0);

  function handleIncrement() {
    setCount((current) => current + 1);
  }

  return (
    <Counter
      count={count}
      onIncrement={handleIncrement}
    />
  );
}

function Counter({ count, onIncrement }) {
  return (
    <>
      <p>{count}</p>
      <button onClick={onIncrement}>
        Increment
      </button>
    </>
  );
}
```

Flow:

```text
Parent owns state
      │
      │ count prop
      ↓
    Child
      │
      │ calls callback prop
      ↓
Parent updates state
      │
      ↓
new prop flows to child
```

This preserves one-way data flow.

---

## 12. Controlled Component Mental Model ⭐⭐⭐⭐⭐

When a parent controls an important value through props:

```jsx
function App() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <Modal
      isOpen={isOpen}
      onClose={() => setIsOpen(false)}
    />
  );
}
```

`Modal` receives:

```text
isOpen
  ↓
current value

onClose
  ↓
way to request a change
```

The parent remains the source of truth.

This pattern appears frequently in controlled components.

Lesson 16 covers controlled vs uncontrolled components deeply.

---

## 13. Local State Mental Model

Sometimes the parent does not care about a UI detail.

```jsx
function FAQItem({ question, answer }) {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <article>
      <button
        onClick={() => setIsExpanded((value) => !value)}
      >
        {question}
      </button>

      {isExpanded && <p>{answer}</p>}
    </article>
  );
}
```

Ownership:

```text
question / answer
      ↓
props

isExpanded
      ↓
local state
```

If nobody outside `FAQItem` needs to control `isExpanded`, local state is often ideal.

---

## 14. Do Not Copy Props into State Automatically ⭐⭐⭐⭐⭐

A very common mistake:

```jsx
function Profile({ name }) {
  const [localName, setLocalName] = useState(name);

  return <h2>{localName}</h2>;
}
```

Suppose:

```text
First render:
name = "Vikash"
localName initialized to "Vikash"

Later parent sends:
name = "Aman"
```

The existing state is not automatically reinitialized:

```text
prop name = "Aman"

localName may still = "Vikash"
```

Now you have two sources of truth.

If you only need to display the prop:

```jsx
function Profile({ name }) {
  return <h2>{name}</h2>;
}
```

---

## 15. When Initializing State from a Prop Can Be Valid

Sometimes the prop is intentionally only an **initial value**.

Example:

```jsx
function EditableName({ initialName }) {
  const [name, setName] = useState(initialName);

  return (
    <input
      value={name}
      onChange={(event) => setName(event.target.value)}
    />
  );
}
```

The prop name `initialName` communicates:

> Use this value when the local state is initialized; afterward the local draft has its own lifecycle.

That is different from pretending the local state should always mirror a changing `name` prop.

---

## 16. Props Are Not a Replacement for State

Suppose a component needs to remember whether its dropdown is open:

```jsx
function Dropdown() {
  let isOpen = false;

  // ...
}
```

A plain local variable cannot provide React state behavior.

If the component owns the interaction:

```jsx
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);

  // ...
}
```

Use state when React must remember a changing value across renders.

---

## 17. State Is Not a Replacement for Props

Another bad design is making a child independently store data the parent already owns.

Parent:

```jsx
const [selectedCategory, setSelectedCategory] =
  useState("All");
```

Child should often receive:

```jsx
<FilterBar
  selectedCategory={selectedCategory}
  onCategoryChange={setSelectedCategory}
/>
```

rather than creating another independent selected category state unless there is a specific reason.

---

## 18. Derived Data Is Usually Neither New Props nor New State ⭐⭐⭐⭐⭐

Suppose:

```jsx
function Cart({ items }) {
  const total = items.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );

  return <p>Total: ₹{total}</p>;
}
```

`items` is a prop.

`total` is a derived value.

There is no need for:

```jsx
const [total, setTotal] = useState(...);
```

Mental model:

```text
Props + State
     ↓
derive values
     ↓
render UI
```

Do not turn every calculated value into state.

---

## 19. State Should Be as Local as Possible ⭐⭐⭐⭐⭐

Suppose only `SearchBox` needs its temporary focus UI:

```text
App
└── SearchPage
    ├── SearchBox
    └── Results
```

If `isFocused` matters only to `SearchBox`, keep it there.

Do not automatically move it to:

```text
App
```

General principle:

> Keep state close to where it is used, but high enough to cover every component that genuinely needs the same source of truth.

This is called **state colocation**.

Lesson 17 covers it deeply.

---

## 20. When State Should Move Up ⭐⭐⭐⭐⭐

Suppose two sibling components need the same state:

```text
TemperatureCalculator
├── CelsiusInput
└── FahrenheitInput
```

If each independently owns unrelated temperature state, synchronization becomes difficult.

Instead:

```text
       TemperatureCalculator
           owns temperature
                 │
        ┌────────┴────────┐
        ↓                 ↓
 CelsiusInput      FahrenheitInput
```

This is **lifting state up**.

The parent owns the shared state and passes values/callbacks as props.

---

## 21. State Ownership Decision Tree ⭐⭐⭐⭐⭐

Ask:

```text
Does this value need to change?
        │
        ├── no → probably prop, constant, or derived value
        │
        └── yes
             ↓
Does React need to remember it between renders?
             │
             ├── no → normal variable/derived value may be enough
             │
             └── yes
                  ↓
Who needs the value?
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
One component          Multiple components
        ↓                   ↓
local state        nearest common owner
                            ↓
                      pass via props
```

This is a better way to think than simply asking:

> Props or state?

First determine **ownership**.

---

## 22. Props Can Pass Behavior Too

Props are not only data.

```jsx
<Button
  label="Delete"
  onClick={handleDelete}
/>
```

Here:

```text
label
  ↓
data prop

onClick
  ↓
function/behavior prop
```

This lets parents define what should happen while children provide reusable UI.

---

## 23. State Can Affect Child Props

```jsx
function App() {
  const [theme, setTheme] = useState("light");

  return (
    <Page theme={theme} />
  );
}
```

When state changes:

```text
App state
theme = light
     ↓
Page prop
theme = light

state update
     ↓

App state
theme = dark
     ↓
Page receives new prop
theme = dark
```

State changes in one component often become prop changes for descendants.

---

## 24. Props, State and Re-rendering ⭐⭐⭐⭐⭐

A component can render again because of several reasons, including:

- its state was updated
- its parent rendered it again
- consumed context changed
- other React mechanisms triggered relevant work

Do not memorize the incorrect rule:

> "A component only re-renders when its props or state change."

For example, a parent render can normally cause its child components to be evaluated/rendered as part of the subtree even if the visible prop values appear unchanged.

Memoization can sometimes skip work.

Rendering is covered deeply in Lesson 12.

---

## 25. Props and State Are Snapshots ⭐⭐⭐⭐⭐

During a particular render, React gives the component a snapshot of its inputs:

```text
Render
├── props snapshot
└── state snapshot
```

Example:

```jsx
function Counter({ step }) {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + step);
  }

  return <button onClick={handleClick}>{count}</button>;
}
```

The handler closes over the `count` and `step` values belonging to the render that created it.

This idea becomes critical for:

- batching
- closures
- event handlers
- Effects
- asynchronous callbacks

Lesson 13 focuses specifically on state snapshots.

---

## 26. Do Not Mutate Props or State ⭐⭐⭐⭐⭐

Wrong prop mutation:

```jsx
function UserCard({ user }) {
  user.name = "Changed"; // ❌

  return <h2>{user.name}</h2>;
}
```

Wrong state mutation:

```jsx
const [user, setUser] = useState({
  name: "Vikash",
});

user.name = "Changed"; // ❌
```

Correct state update:

```jsx
setUser((current) => ({
  ...current,
  name: "Changed",
}));
```

React works best when props and state are treated as immutable render inputs.

---

## 27. Real-World Example: Job Application Card

Suppose a page owns application data:

```jsx
function ApplicationsPage() {
  const [applications, setApplications] =
    useState(initialApplications);

  function handleStatusChange(id, status) {
    setApplications((current) =>
      current.map((application) =>
        application.id === id
          ? { ...application, status }
          : application
      )
    );
  }

  return applications.map((application) => (
    <ApplicationCard
      key={application.id}
      application={application}
      onStatusChange={handleStatusChange}
    />
  ));
}
```

Child:

```jsx
function ApplicationCard({
  application,
  onStatusChange,
}) {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <article>
      <h2>{application.company}</h2>
      <p>{application.status}</p>

      <button
        onClick={() =>
          onStatusChange(application.id, "Interview")
        }
      >
        Move to Interview
      </button>

      <button
        onClick={() => setIsExpanded((value) => !value)}
      >
        {isExpanded ? "Hide" : "Show"} details
      </button>
    </article>
  );
}
```

Ownership:

```text
ApplicationsPage
├── applications       → state
└── handleStatusChange → behavior

ApplicationCard
├── application        → prop
├── onStatusChange     → prop
└── isExpanded         → local state
```

This is a healthy division of responsibility.

---

## 28. Common Mistakes

### Mistake 1 — Saying props cannot change

Props can change when the parent supplies new values. The child simply must not mutate them.

### Mistake 2 — Copying every prop into state

This creates unnecessary duplicated sources of truth.

### Mistake 3 — Putting all state in the top-level component

State should normally remain close to where it is needed.

### Mistake 4 — Giving siblings independent copies of shared state

Lift shared state to their nearest common owner.

### Mistake 5 — Mutating props

Props are read-only inputs.

### Mistake 6 — Mutating state

Use React state setters and immutable updates.

### Mistake 7 — Storing derived values as state

Calculate them from existing props/state when practical.

### Mistake 8 — Thinking state is globally shared

State declared by a component belongs to that component instance unless explicitly shared through architecture.

### Mistake 9 — Thinking props are only data strings

Props can carry objects, arrays, functions, React nodes, and other JavaScript values.

---

## 29. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is the difference between props and state?

**Answer:** Props are read-only inputs supplied by a component's caller, while state is data owned by a component that React preserves between renders and can update through state APIs.

### Q2. Can props change?

**Answer:** Yes. A parent can render a child with new prop values. Props are immutable only from the receiving component's perspective.

### Q3. Can a child modify a prop?

**Answer:** No. It should request changes through a callback or another explicit API provided by the owner.

### Q4. Can state be mutated directly?

**Answer:** No. State should be treated as immutable and updated through its setter.

### Q5. Can parent state become child props?

**Answer:** Yes. This is extremely common. A value can be state in the owner and a prop in a descendant.

### Q6. What is a single source of truth?

**Answer:** It means a particular piece of data has one authoritative owner rather than multiple independent copies that can become inconsistent.

### Q7. When should state be lifted up?

**Answer:** When multiple components need to coordinate around the same changing value, move that state to their nearest appropriate common owner.

### Q8. Why is copying props into state often a problem?

**Answer:** It duplicates data and can cause the local copy to become out of sync when the prop changes.

### Q9. Is initializing state from props always wrong?

**Answer:** No. It can be appropriate when the prop intentionally provides only an initial value and the local state then has an independent lifecycle.

### Q10. Where should state live?

**Answer:** As close as practical to the components that need it, while being high enough to serve every component that requires the same source of truth.

### Q11. Should derived data be state?

**Answer:** Usually not. If it can be calculated from current props/state during rendering, deriving it avoids synchronization problems.

### Q12. Does a child render only when its props change?

**Answer:** No. Parent rendering can also cause child rendering. React's rendering and memoization behavior is more nuanced.

---

## 30. Interview Scenario ⭐⭐⭐⭐⭐

### Requirement

A search page contains:

```text
SearchPage
├── SearchInput
└── SearchResults
```

The text entered into `SearchInput` determines what `SearchResults` displays.

Where should the query state live?

### Bad approach

```text
SearchInput
└── owns query

SearchResults
└── needs query but cannot access it directly
```

### Better approach

```jsx
function SearchPage({ products }) {
  const [query, setQuery] = useState("");

  const filteredProducts = products.filter((product) =>
    product.name
      .toLowerCase()
      .includes(query.toLowerCase())
  );

  return (
    <>
      <SearchInput
        query={query}
        onQueryChange={setQuery}
      />

      <SearchResults
        products={filteredProducts}
      />
    </>
  );
}
```

Flow:

```text
             SearchPage
             query state
                 │
        ┌────────┴─────────┐
        ↓                  ↓
  SearchInput        SearchResults
  query prop         derived products
        │
onQueryChange
        │
        └────────→ SearchPage updates state
```

Why?

Because `SearchPage` is the nearest common owner that needs to coordinate both children.

---

## 31. Props vs State Decision Table

| Question | Likely Choice |
|---|---|
| Is the value supplied by a parent/caller? | Prop |
| Does this component need to remember a changing value? | State |
| Do siblings need the same changing value? | Lift state to common owner, then pass props |
| Can the value be calculated from existing data? | Derive it |
| Is it only a local UI detail? | Local state |
| Does a child need to request an owner update? | Callback prop |
| Is the value only an initial seed for an independent draft? | State initialized from an explicitly named initial prop may be appropriate |

---

## 32. Complete Mental Model ⭐⭐⭐⭐⭐

```text
                    Owner Component
                          │
                    owns state
                          │
             ┌────────────┴────────────┐
             │                         │
        data props               callback props
             │                         │
             ↓                         ↓
                       Child
                         │
                 owns local state
                  when appropriate
                         │
                 user interaction
                         │
                 calls callback
                         │
                         ↓
                    Owner updates
                       state
                         │
                         ↓
                 new props flow down
```

And derived values:

```text
Props + State
     │
     ↓
Derived Values
     │
     ↓
Rendered UI
```

---

## 33. Quick Revision

```text
Props
├── external input
├── supplied by caller
├── read-only to receiver
├── can change across renders
└── data / functions / React content

State
├── component-owned memory
├── preserved by React
├── updated through state API
├── local to component instance
└── causes rendering when updated
```

Most important relationship:

```text
Parent State
     ↓
Child Props
     ↓
Child Callback
     ↓
Parent State Update
     ↓
New Child Props
```

Remember:

```text
Do not mutate props.
Do not mutate state.
Do not duplicate data without a reason.
Keep one clear source of truth.
Keep state as local as practical.
Derive values when possible.
```

---

## 34. Key Takeaways

- Props are inputs supplied by a component's caller.
- State is component-owned data React remembers between renders.
- Props are read-only to the receiving component.
- State should also be treated as an immutable snapshot.
- Props can change when the parent renders new values.
- Parent state commonly becomes child props.
- Callback props let children request changes from the state owner.
- Every piece of shared state should have a clear source of truth.
- Do not automatically copy props into state.
- Initializing local state from an explicitly initial prop can be valid when the local value intentionally becomes independent.
- Derived data usually should not become additional state.
- Keep state as close as practical to where it is needed.
- Lift state when multiple components need to coordinate around the same changing value.
- A component can simultaneously receive props and own local state.
- Understanding **ownership** is more important than memorizing a props-vs-state definition.

---

## Next Lesson

➡️ [Lesson 12 — How React Rendering and Re-rendering Work ⭐⭐⭐⭐⭐](./12-rendering-rerendering.md)
