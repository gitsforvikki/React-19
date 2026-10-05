# Lesson 73 — Most Important React Interview Questions ⭐⭐⭐⭐⭐

## 1. Purpose of This Lesson

This lesson is designed for fast React interview revision.

The goal is not to memorize definitions word-for-word.

For each important topic, you should be able to explain:

```text
WHAT is it?
WHY is it needed?
HOW does it work?
WHEN would you use it?
WHAT mistakes should you avoid?
```

A strong interview answer usually combines:

```text
clear definition
+
correct mental model
+
small example
+
tradeoff / production insight
```

---

# Part 1 — React Fundamentals

## 2. What Is React? ⭐⭐⭐⭐⭐

React is a JavaScript library for building user interfaces using declarative, component-based composition.

Instead of manually changing DOM elements after every state change, developers describe the UI for the current state and React coordinates updating the rendered interface.

```jsx
function Greeting({ name }) {
  return <h1>Hello {name}</h1>;
}
```

Interview answer:

```text
React is a declarative, component-based JavaScript
library for building user interfaces.

Components describe UI from props and state. When those
inputs change, React renders again, reconciles the new
element tree with the previous one, and commits the
necessary host-environment updates.
```

---

## 3. Why Use React?

Important reasons include:

- reusable component composition
- declarative UI
- predictable data flow
- state-driven rendering
- strong ecosystem
- efficient update coordination
- reusable Hooks/logic
- support for modern concurrent rendering capabilities

React does not automatically guarantee good performance or architecture.

---

## 4. Library vs Framework?

React primarily focuses on UI composition and rendering.

A full framework may additionally define concerns such as:

- routing
- data fetching conventions
- server rendering
- build/deployment behavior
- caching
- filesystem conventions

Do not describe React itself as if every framework feature belongs to core React.

---

## 5. What Is Declarative UI? ⭐⭐⭐⭐⭐

Imperative thinking:

```text
find element
change text
hide button
add class
```

Declarative React:

```jsx
return isLoggedIn
  ? <Dashboard />
  : <Login />;
```

You describe what UI should exist for the current state.

React coordinates how the rendered output changes.

---

## 6. What Is JSX?

JSX is syntax that lets us describe React element trees using HTML-like notation inside JavaScript.

```jsx
const element = (
  <button>
    Save
  </button>
);
```

JSX is transformed into JavaScript representation understood by React tooling/runtime.

JSX is not HTML itself.

---

## 7. Why Must Component Names Start With a Capital Letter?

Lowercase JSX names are treated as built-in host elements:

```jsx
<div />
<button />
```

Capitalized names are treated as component references:

```jsx
<UserCard />
```

---

# Part 2 — Components and Props

## 8. What Is a React Component? ⭐⭐⭐⭐⭐

A component is a reusable unit that describes part of the UI.

A function component receives inputs such as props and returns React elements.

```jsx
function UserCard({ user }) {
  return (
    <article>
      {user.name}
    </article>
  );
}
```

Good components usually have a clear responsibility.

---

## 9. What Is Component Composition? ⭐⭐⭐⭐⭐

Composition means building larger interfaces from smaller components.

```jsx
<Card>
  <Avatar />
  <UserDetails />
  <Actions />
</Card>
```

React generally favors composition instead of deep inheritance hierarchies for UI reuse.

---

## 10. What Are Props? ⭐⭐⭐⭐⭐

Props are inputs passed from a parent to a child.

```jsx
<UserCard
  name="Vikash"
/>
```

They are read-only from the receiving component's perspective.

A child should not mutate props.

---

## 11. What Is One-Way Data Flow?

Data normally flows:

```text
Parent
  │
  │ props
  ↓
Child
```

When a child needs to communicate an event upward:

```text
Parent
  │ callback prop
  ↓
Child
  │ invokes callback
  ↑
Parent handles change
```

This keeps ownership explicit.

---

## 12. Props vs State? ⭐⭐⭐⭐⭐

### Props

- supplied externally
- read-only to receiver
- controlled by parent

### State

- owned by a component instance
- persists across renders
- changed through React state updates

Mental model:

```text
props = external inputs
state = component memory
```

---

# Part 3 — Lists and Keys

## 13. Why Does React Need Keys? ⭐⭐⭐⭐⭐

Keys identify siblings across renders.

```jsx
items.map(item => (
  <Item
    key={item.id}
    item={item}
  />
))
```

During reconciliation, keys help React match:

```text
previous child
↕
next child
```

Stable identity helps preserve the correct component state and DOM association.

---

## 14. Why Is Array Index Often a Bad Key? ⭐⭐⭐⭐⭐

If items can:

- reorder
- insert
- delete
- filter

the same index may represent a different logical item after an update.

That can cause state or DOM identity to follow the wrong item.

Index keys are acceptable mainly when the list is truly static and identity cannot change.

---

## 15. Why Is Math.random() a Bad Key?

A new key on every render tells React that the previous component identity no longer matches.

This can cause unnecessary remounting and state loss.

Keys should be stable and meaningful among siblings.

---

# Part 4 — State

## 16. What Is State? ⭐⭐⭐⭐⭐

State is React-managed memory associated with a component instance.

```jsx
const [count, setCount] =
  useState(0);
```

Calling the setter requests a render with a new state value.

---

## 17. Why Doesn't a Normal Variable Work as State?

```jsx
let count = 0;
```

Changing it:

- does not tell React to render
- does not provide React-managed persistence between renders

State provides both persistence and rendering integration.

---

## 18. What Does "State Is a Snapshot" Mean? ⭐⭐⭐⭐⭐

Each render receives its own state values.

```jsx
function handleClick() {
  setCount(count + 1);

  console.log(count);
}
```

The current handler still sees the `count` captured by the render that created it.

The setter schedules a future render; it does not rewrite the current render's variable.

---

## 19. Why Does This Usually Increment Once?

```jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

All three lines use the same current-render snapshot.

Conceptually they request the same replacement value.

For updates based on previous queued state:

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

Use functional updates.

---

## 20. What Is Batching? ⭐⭐⭐⭐⭐

React can group multiple state updates before rendering.

```text
event
 ├── update A
 ├── update B
 └── update C
       ↓
React processes updates
       ↓
render
```

Batching avoids unnecessary intermediate renders and helps keep updates consistent.

---

## 21. When Should You Use a Functional State Update?

When the next value depends on the previous state:

```jsx
setCount(c => c + 1);
```

Especially important when multiple updates may be queued.

---

## 22. How Do You Update Objects in State?

Do not mutate the existing state object.

```jsx
setUser(prev => ({
  ...prev,
  name: "Vikash",
}));
```

Create a new object for changed state.

---

## 23. How Do You Update Arrays in State?

Prefer non-mutating operations:

```text
add    → spread / concat
remove → filter
update → map
```

Example:

```jsx
setItems(prev =>
  prev.filter(
    item => item.id !== id
  )
);
```

---

# Part 5 — Controlled and Uncontrolled Components

## 24. Controlled vs Uncontrolled Input? ⭐⭐⭐⭐⭐

Controlled:

```jsx
<input
  value={name}
  onChange={e =>
    setName(e.target.value)
  }
/>
```

React state is the current source of truth for the input value.

Uncontrolled:

```jsx
<input
  defaultValue="Vikash"
  ref={inputRef}
/>
```

The DOM maintains the current value and React can read it when needed.

---

## 25. When Would You Use Controlled Inputs?

Useful when UI behavior needs current input state for:

- validation
- conditional UI
- formatting
- dependent fields
- immediate application logic

Uncontrolled approaches can be simpler when you only need values at specific times.

---

# Part 6 — State Ownership

## 26. What Is Lifting State Up? ⭐⭐⭐⭐⭐

When multiple components need the same state, move ownership to their closest common parent.

```text
       Parent
       state
      /     \
 Child A   Child B
```

The parent passes values/callbacks downward.

---

## 27. What Is State Colocation?

Keep state as close as possible to the components that actually need it.

Do not move local state to global state without a real sharing requirement.

---

## 28. How Does React Preserve State? ⭐⭐⭐⭐⭐

State is associated with a component's position/identity in the rendered tree.

React considers factors including:

- component type
- position
- key

Changing identity can reset state.

---

## 29. How Can You Intentionally Reset State?

A meaningful key can create a new component identity:

```jsx
<Chat
  key={user.id}
  user={user}
/>
```

When the key changes, React treats it as a different component instance.

---

# Part 7 — Rendering

## 30. What Causes a React Component to Render? ⭐⭐⭐⭐⭐

Common triggers include:

- its state update
- parent rendering it again
- consumed Context changing
- external-store subscription updates

A render means React calls component logic to determine the next UI representation.

It does not mean every DOM node is recreated.

---

## 31. Render vs Commit? ⭐⭐⭐⭐⭐

### Render phase

React calculates what the UI should look like.

Render must remain pure.

### Commit phase

React applies necessary changes to the host environment/DOM and handles relevant ref/effect lifecycle work.

```text
trigger
  ↓
render
  ↓
reconcile
  ↓
commit
  ↓
browser paint
```

---

## 32. Why Must Rendering Be Pure? ⭐⭐⭐⭐⭐

Given the same props/state/context, rendering should calculate JSX without causing external side effects.

React may render work more than once, pause it, or discard work under modern rendering behavior.

Side effects during render make this unsafe.

---

# Part 8 — Effects

## 33. What Is useEffect For? ⭐⭐⭐⭐⭐

`useEffect` synchronizes a component with an external system after rendering.

Examples:

- network synchronization when appropriate
- subscriptions
- browser APIs
- timers
- third-party widgets
- connections

Mental model:

```text
render
  ↓
commit
  ↓
Effect synchronizes external system
```

---

## 34. What Is useEffect NOT For? ⭐⭐⭐⭐⭐

Usually not for:

- deriving values from props/state
- handling direct user actions
- copying props into state
- chaining internal state transformations

If no external system is involved, ask whether an Effect is actually necessary.

---

## 35. What Does the Dependency Array Mean? ⭐⭐⭐⭐⭐

Dependencies describe reactive values used by the Effect.

```jsx
useEffect(() => {
  connect(roomId);
}, [roomId]);
```

When a dependency changes according to React's comparison semantics, React re-synchronizes the Effect.

Do not think of the array as an arbitrary scheduling configuration.

---

## 36. What Happens With No Dependency Array?

```jsx
useEffect(() => {
  ...
});
```

The Effect is eligible to run after every committed render.

---

## 37. What Does [] Mean?

```jsx
useEffect(() => {
  ...
}, []);
```

The Effect does not depend on changing reactive values from the component.

In development Strict Mode, React may intentionally perform an extra setup/cleanup cycle to expose missing cleanup bugs.

Do not describe `[]` simply as "runs exactly once" in all development behavior.

---

## 38. What Is Effect Cleanup? ⭐⭐⭐⭐⭐

An Effect can return cleanup:

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

Cleanup runs when synchronization must stop, including before re-synchronizing due to changed dependencies and when the component unmounts.

---

## 39. What Is a Stale Closure? ⭐⭐⭐⭐⭐

A callback captures values from the render that created it.

If it later executes while you expected newer values, it may appear stale.

Example:

```jsx
useEffect(() => {
  const id = setInterval(
    () => {
      console.log(count);
    },
    1000
  );

  return () =>
    clearInterval(id);
}, []);
```

The callback captured the initial `count`.

Fix depends on intended semantics:

- correct dependencies
- functional updater
- ref for latest non-rendering value
- Effect Event for appropriate non-reactive Effect logic

Do not simply suppress dependency warnings.

---

## 40. Race Condition vs Stale Closure?

Stale closure:

```text
callback uses an older render's captured value
```

Async race:

```text
multiple async operations finish out of order
```

They are related async problems but not identical.

---

# Part 9 — Refs

## 41. What Is useRef? ⭐⭐⭐⭐⭐

```jsx
const ref = useRef(null);
```

It returns a stable mutable object:

```js
{
  current: ...
}
```

Changing `ref.current` does not request a render.

Use refs for values that must persist but do not directly determine rendered output.

---

## 42. useRef vs useState? ⭐⭐⭐⭐⭐

```text
State:
persists
+
updates trigger rendering

Ref:
persists
+
mutation does not trigger rendering
```

If the value should affect visible UI, state is usually appropriate.

---

## 43. Common Ref Use Cases?

- focus
- scrolling
- media control
- DOM measurement
- timer IDs
- mutable instance-like values
- latest-value escape hatches when appropriate

Refs should not replace normal state/data flow.

---

## 44. What Is useImperativeHandle?

It lets a component customize the imperative API exposed through a ref.

Instead of exposing an entire DOM node, a component may expose:

```js
{
  focus(),
  select()
}
```

Use it sparingly when an imperative API is genuinely appropriate.

---

# Part 10 — Context and Reducers

## 45. What Problem Does Context Solve? ⭐⭐⭐⭐⭐

Context lets a component receive a value from a distant provider without manually passing it through every intermediate component.

Good examples:

- theme
- auth/session interface
- locale
- feature-level shared state

Context is not automatically a replacement for all state management.

---

## 46. Does Context Prevent Re-renders?

No.

When a consumed Context value changes, consumers can render again.

Context solves value distribution, not automatic performance optimization.

---

## 47. What Is useReducer? ⭐⭐⭐⭐⭐

`useReducer` manages state transitions using a reducer:

```jsx
function reducer(
  state,
  action
) {
  switch (action.type) {
    case "increment":
      return {
        ...state,
        count:
          state.count + 1,
      };

    default:
      return state;
  }
}
```

It is useful when:

- transitions are complex
- multiple values change together
- actions provide clearer semantics
- state logic benefits from centralization

---

## 48. useState vs useReducer?

Use `useState` for straightforward independent state.

Consider `useReducer` when state transitions become complex or action-oriented.

Neither is universally better.

---

## 49. useReducer + Context?

Context can distribute state and dispatch from a reducer to a subtree.

This can be useful for feature-level state.

Do not use it automatically for all application data.

---

# Part 11 — Custom Hooks

## 50. What Is a Custom Hook? ⭐⭐⭐⭐⭐

A custom Hook is a function whose name begins with `use` and which composes React Hooks to reuse stateful React logic.

```jsx
function useOnlineStatus() {
  ...
}
```

Custom Hooks share logic, not component state instances.

Each call gets its own Hook state unless the Hook connects to shared external state.

---

## 51. Rules of Hooks? ⭐⭐⭐⭐⭐

Hooks should be called:

1. at the top level of React function components/custom Hooks
2. only from React components or custom Hooks

Do not conditionally change Hook call order.

React relies on consistent Hook ordering to associate Hook state correctly.

---

# Part 12 — Memoization and Performance

## 52. What Does React.memo Do? ⭐⭐⭐⭐⭐

`memo` can skip rendering a component when its props are unchanged according to its comparison behavior.

```jsx
const UserCard =
  memo(function UserCard({
    user,
  }) {
    ...
  });
```

It is a performance optimization, not a correctness tool.

---

## 53. What Does useMemo Do? ⭐⭐⭐⭐⭐

`useMemo` caches a calculated value between renders until dependencies change.

```jsx
const filtered =
  useMemo(
    () =>
      expensiveFilter(
        items,
        query
      ),
    [items, query]
  );
```

Use it when memoization has measurable/useful value.

Do not wrap every calculation.

---

## 54. What Does useCallback Do? ⭐⭐⭐⭐⭐

`useCallback` caches a function reference until dependencies change.

```jsx
const handleSelect =
  useCallback(
    id => {
      ...
    },
    [dependency]
  );
```

It is useful when function identity matters, such as with memoized children or dependency-sensitive APIs.

It does not prevent function creation work from ever occurring conceptually; it gives React a cached function identity to return when dependencies are unchanged.

---

## 55. React.memo vs useMemo vs useCallback? ⭐⭐⭐⭐⭐

```text
React.memo
→ memoizes component rendering based on props

useMemo
→ memoizes a calculated value

useCallback
→ memoizes a function reference
```

All are optimization tools.

---

## 56. Why Can Memoization Fail?

Example:

```jsx
<MemoChild
  options={{
    sort: "asc",
  }}
/>
```

A new object is created each render.

Referential identity changed, so shallow prop comparison cannot consider it unchanged.

But do not stabilize every object blindly—optimize based on real need.

---

# Part 13 — Concurrent UI

## 57. What Is useTransition? ⭐⭐⭐⭐⭐

`useTransition` lets you mark state updates as non-urgent transitions.

```jsx
const [
  isPending,
  startTransition,
] = useTransition();

startTransition(() => {
  setTab(nextTab);
});
```

Urgent interactions can remain responsive while React works on lower-priority rendering.

A transition does not make JavaScript computation magically faster.

---

## 58. What Is useDeferredValue?

It lets a part of the UI lag behind an urgently changing value.

Example:

```jsx
const deferredQuery =
  useDeferredValue(query);
```

The input can update immediately while expensive result rendering uses the deferred value.

---

# Part 14 — Lazy Loading and Suspense

## 59. What Is React.lazy?

It lets a component's code load dynamically.

```jsx
const Settings =
  lazy(
    () =>
      import("./Settings")
  );
```

Usually rendered under Suspense:

```jsx
<Suspense
  fallback={<Spinner />}
>
  <Settings />
</Suspense>
```

This supports code splitting.

---

## 60. What Is Suspense? ⭐⭐⭐⭐⭐

Suspense defines a boundary for displaying fallback UI while supported child work/resources are not ready.

```jsx
<Suspense
  fallback={<Skeleton />}
>
  <Content />
</Suspense>
```

Suspense is a coordination mechanism.

It is not simply a replacement for every manually fetched `useEffect` request.

---

# Part 15 — React Internals

## 61. What Is the Virtual DOM? ⭐⭐⭐⭐⭐

"Virtual DOM" commonly describes React's in-memory representation of UI elements used while determining updates.

A useful flow:

```text
state/props change
      ↓
new React element tree
      ↓
reconciliation
      ↓
commit necessary host changes
```

Avoid saying React always rebuilds the entire real DOM.

---

## 62. What Is Reconciliation? ⭐⭐⭐⭐⭐

Reconciliation is React's process of comparing/rendering element trees and determining how component/host identities correspond across renders.

Important identity signals include:

- element/component type
- position
- key

---

## 63. What Is Fiber? ⭐⭐⭐⭐⭐

Fiber is React's internal architecture for representing units of rendering work.

It enables React to organize work in a way that supports capabilities such as:

- prioritization
- interruption
- resumption
- concurrent rendering

Interview-level answer:

```text
Fiber is React's internal work-unit/tree architecture.
It allows rendering work to be broken into units rather
than requiring all render work to be treated as one
indivisible synchronous operation.
```

Do not depend on undocumented Fiber internals in application code.

---

## 64. Render Phase vs Commit Phase? ⭐⭐⭐⭐⭐

Render:

- calculates next tree
- should be pure
- can potentially be interrupted/restarted

Commit:

- applies finalized host changes
- updates refs/lifecycle-related work
- is where the chosen result becomes visible to the host environment

This distinction is fundamental to modern React.

---

# Part 16 — Error Boundaries

## 65. What Is an Error Boundary? ⭐⭐⭐⭐⭐

An Error Boundary catches certain rendering/lifecycle errors in its descendant component tree and displays fallback UI.

It helps isolate failures:

```text
Dashboard
├── Header
├── ErrorBoundary
│    └── Analytics
└── Applications
```

It does not automatically catch every asynchronous/event-handler error.

---

# Part 17 — React 19

## 66. What Are Actions in React 19? ⭐⭐⭐⭐⭐

Actions are a React model for handling asynchronous mutations/transitions with integrated pending/error/form behavior in supported APIs.

They reduce manual orchestration around common async mutation flows.

The exact integration depends on the React API/environment being used.

---

## 67. What Is useActionState? ⭐⭐⭐⭐⭐

`useActionState` manages state associated with an Action.

Conceptually:

```jsx
const [
  state,
  action,
  isPending,
] = useActionState(
  updateProfile,
  initialState
);
```

It is especially useful for form/action workflows where the action result becomes UI state.

---

## 68. What Is useFormStatus?

It exposes status information for a parent form submission from a component rendered within that form.

A submit button can react to pending state without manually threading that state through props.

---

## 69. What Is useOptimistic? ⭐⭐⭐⭐⭐

It lets UI temporarily show an optimistic result while an Action/request is pending.

```text
user action
   ↓
optimistic UI immediately
   ↓
server operation
  / \
success failure
  │      │
keep   reconcile
```

The server remains authoritative.

---

## 70. What Is the use() API? ⭐⭐⭐⭐⭐

`use` can read supported resources such as a Promise or Context in React-supported rendering environments.

When reading a pending Promise, rendering can suspend to the nearest Suspense boundary.

It has special React semantics and should not be treated as a normal custom Hook.

---

## 71. What Changed With refs in React 19?

In React 19, function components can receive `ref` as a prop in modern code.

This reduces the need for `forwardRef` in new React 19 component APIs.

Legacy code and older React versions commonly use `forwardRef`.

---

# Part 18 — Server Component Concepts

## 72. Client vs Server Components? ⭐⭐⭐⭐⭐

At a conceptual level:

### Server Components

Can render in a server environment and do not ship their component code as ordinary interactive client component JavaScript.

### Client Components

Support client-side interactivity, state, effects and browser APIs.

Framework-specific implementation rules belong to the framework.

---

## 73. What Is the Main Benefit of Server Components?

They can move suitable rendering/data work to the server and reduce client-side JavaScript for non-interactive portions.

They are not simply traditional SSR under a different name.

---

# Part 19 — Data Fetching

## 74. What Is a Request Race Condition? ⭐⭐⭐⭐⭐

```text
request A starts
request B starts

B finishes first
→ UI shows B

A finishes later
→ incorrectly overwrites B
```

Solutions depend on architecture and may include:

- ignoring stale results
- cancellation with AbortController
- request identity
- data libraries/framework coordination

---

## 75. AbortController vs Ignore Flag?

AbortController attempts to cancel supported underlying work.

An ignore/request-identity strategy prevents stale results from being committed to state.

Even cancellation strategies should consider race semantics carefully.

---

# Part 20 — Architecture

## 76. How Do You Structure a Large React App? ⭐⭐⭐⭐⭐

Strong answer:

```text
I organize domain-heavy code around features and clear
ownership rather than putting every component or Hook
into one global folder.

Within a feature I colocate related UI, Hooks, data
integration and pure helpers when needed. I keep state
close to consumers, expose small feature boundaries,
avoid circular dependencies and move code into shared
layers only when genuine reuse appears.
```

---

## 77. What Is Separation of Concerns?

Keep distinct responsibilities understandable:

```text
UI rendering
React behavior
external communication
pure domain logic
authoritative server logic
```

Do not turn separation into unnecessary layers.

---

## 78. Local State vs Context vs External Store? ⭐⭐⭐⭐⭐

Start with local state.

Lift when nearby components genuinely share it.

Use Context when a subtree needs shared values without repetitive prop passing.

Use an external store when application-wide/state-management requirements justify it.

Do not globalize state by default.

---

# Part 21 — Production UI

## 79. How Do You Model Async UI State?

For mutually exclusive states, a status model can avoid contradictory booleans:

```text
idle
loading
success
error
```

Then derive empty state from successful data.

Also distinguish:

```text
initial loading
vs
background refreshing
```

---

## 80. Empty State vs Error State?

Empty:

```text
request succeeded
but zero relevant results
```

Error:

```text
operation failed
```

They should not share the same message or recovery action.

---

# Part 22 — Accessibility

## 81. How Do You Make React UI Accessible?

Start with:

- semantic HTML
- native controls
- labels
- keyboard support
- visible focus
- accessible names
- focus management
- meaningful error/status feedback

Use ARIA only when native semantics are insufficient.

---

## 82. Button vs Clickable div?

Prefer:

```jsx
<button>
  Save
</button>
```

A native button already provides semantics, focusability and expected keyboard behavior.

---

# Part 23 — Security

## 83. Is Hiding a Button Authorization? ⭐⭐⭐⭐⭐

No.

```jsx
{isAdmin && (
  <DeleteButton />
)}
```

is UI behavior only.

The server must independently verify authorization when the request arrives.

---

## 84. Is React Safe From XSS? ⭐⭐⭐⭐⭐

React escapes ordinary interpolated text by default.

However, XSS risk still exists through unsafe patterns such as:

- unsanitized raw HTML
- unsafe URLs/integrations
- vulnerable third-party code
- other browser injection paths

Treat `dangerouslySetInnerHTML` carefully.

---

## 85. Can Frontend Environment Variables Hold Secrets?

Not if they are shipped to browser JavaScript.

Anything delivered to the browser should be considered visible to the user.

True secrets remain server-side.

---

## 86. Why Must Payment Totals Be Calculated Server-Side?

The browser is attacker-controlled.

A user can modify:

```text
price
discount
quantity
total
```

The server must calculate/validate authoritative financial values.

---

# Part 24 — Testing

## 87. How Should React Components Be Tested?

Prefer tests that resemble user behavior:

```text
render UI
→ interact
→ observe visible result
```

Avoid coupling tests to private state or implementation details.

---

## 88. Unit vs Integration vs E2E?

```text
Unit
→ isolated logic

Component/Integration
→ UI + related behavior

E2E
→ complete application journey
```

Use each level where it gives useful confidence.

---

## 89. Why Prefer getByRole in UI Tests?

It queries elements through semantic roles and accessible names, which is closer to how users and assistive technologies understand the interface.

---

# Part 25 — Scenario Questions

## 90. Parent Re-renders. Does Child Re-render? ⭐⭐⭐⭐⭐

Normally React will render child components as part of rendering the parent's returned tree.

Optimization mechanisms such as `memo` may allow React to skip some child rendering when their relevant inputs are unchanged.

A child render does not automatically mean DOM changes.

---

## 91. Does setState Immediately Change the Current Variable?

No.

Current render state is a snapshot.

The setter schedules an update for a future render.

---

## 92. Should Every Function Be Wrapped in useCallback?

No.

`useCallback` adds memoization complexity and is useful when function identity actually matters.

Use performance optimizations deliberately.

---

## 93. Should Every Calculation Use useMemo?

No.

Cheap calculations usually do not need memoization.

Memoize when avoiding recalculation/reference changes provides actual value.

---

## 94. Should Every API Call Be in useEffect?

No.

It depends on the architecture.

Effects can synchronize client components with external systems, but modern frameworks/data libraries may provide better data-loading, caching, deduplication and server-rendering patterns.

---

## 95. Can You Mutate State and Then Call the Setter With the Same Object?

This is incorrect state management.

Mutation breaks snapshot/identity assumptions, and React may see the same reference.

Create a new state value instead.

---

## 96. Why Does an Effect Run Twice in Development?

With Strict Mode, React can intentionally perform an extra development-only setup/cleanup cycle to reveal Effects that are not resilient or lack cleanup.

Do not "fix" this by disabling Strict Mode or using a ref to suppress legitimate lifecycle behavior.

Fix the Effect.

---

## 97. Why Does an Input Lose Focus When Typing?

Possible causes include:

- component unexpectedly remounting
- changing key on every render
- defining component types in unstable places/patterns that recreate identity
- conditional tree identity changes
- manual focus behavior

Debug component identity first.

---

## 98. Why Does React.memo Not Stop Re-rendering?

Possible reasons:

- props changed
- new object/array/function references
- component consumes changed Context
- internal state changed
- custom comparison behavior is unsuitable

Memoization is not a universal render blocker.

---

## 99. How Would You Optimize a Slow React Page? ⭐⭐⭐⭐⭐

Do not start by adding `useMemo` everywhere.

Process:

```text
1. Measure/profile.
2. Find expensive render/work.
3. Reduce unnecessary state/effects.
4. Colocate state.
5. Avoid unnecessary parent updates.
6. Split expensive UI/code when useful.
7. Stabilize identities only where they matter.
8. Memoize expensive calculations/components where beneficial.
9. Virtualize very large lists when appropriate.
10. Measure again.
```

Optimization should be evidence-driven.

---

# Part 26 — Rapid-Fire Questions

## 100. Does Updating a Ref Re-render?

No.

---

## 101. Does Updating State Re-render?

It requests/schedules a React update. React may bail out in cases such as setting state to an equivalent value.

---

## 102. Are Props Mutable?

No. Treat them as read-only inputs.

---

## 103. Can a Component Return null?

Yes.

It means that component renders no host UI for that render.

---

## 104. Can Hooks Be Called Conditionally?

Ordinary Hooks should not have their call order conditionally changed.

React depends on stable Hook ordering.

---

## 105. Is Context Global State?

Not necessarily.

Context is a value-distribution mechanism scoped to its provider tree.

---

## 106. Does useMemo Guarantee a Semantic Contract?

It should be treated as a performance optimization rather than something your application's correctness fundamentally depends upon.

---

## 107. Does useCallback Make a Function Faster?

No.

Its main purpose is preserving function identity between renders when dependencies remain unchanged.

---

## 108. Does React.memo Deep Compare Props?

By default it compares individual props using React's shallow-style equality behavior, not recursive deep comparison.

---

## 109. Are Keys Passed as Normal Props?

No.

If the component needs the identifier, pass it separately:

```jsx
<Item
  key={item.id}
  id={item.id}
/>
```

---

## 110. Does useEffect Run During Render?

No.

Effects run after React commits the relevant render.

---

## 111. Can Cleanup Run Without Final Unmount?

Yes.

Cleanup also runs before an Effect re-synchronizes because dependencies changed.

---

## 112. Is useRef Only for DOM Nodes?

No.

It can hold any persistent mutable value that does not need to trigger rendering.

---

## 113. Is useReducer Always Better for Complex Apps?

No.

Choose based on state-transition complexity and ownership, not application size alone.

---

## 114. Is Redux Required for React?

No.

React provides local state, reducers and Context. External state libraries solve additional state-management requirements when needed.

---

# Part 27 — Interview Traps

## 115. Avoid These Weak Answers ⭐⭐⭐⭐⭐

### Weak

```text
Virtual DOM makes React fast.
```

Better:

```text
React uses an in-memory representation and reconciliation
to determine updates, but performance depends on component
design, rendering cost, update frequency and many other
factors.
```

### Weak

```text
useEffect is for API calls.
```

Better:

```text
useEffect synchronizes with external systems. Network
requests are one possible synchronization case, but an
Effect is not automatically the best data-fetching
architecture.
```

### Weak

```text
useCallback improves performance.
```

Better:

```text
useCallback stabilizes function identity. It helps only
when that identity stability prevents meaningful work or
is required by another dependency-sensitive API.
```

### Weak

```text
React.memo prevents re-render.
```

Better:

```text
memo can let React skip rendering when props are
unchanged, but state, Context or changed prop references
can still cause rendering.
```

### Weak

```text
[] means useEffect runs once.
```

Better:

```text
[] means the Effect has no changing reactive
dependencies. Development Strict Mode may intentionally
exercise setup and cleanup more than once.
```

---

# Part 28 — How to Answer React Interview Questions

## 116. Four-Step Answer Formula ⭐⭐⭐⭐⭐

For an important concept:

```text
1. Definition
2. Why / use case
3. Small example or internal mental model
4. Tradeoff / common mistake
```

Example — `useMemo`:

```text
Definition:
useMemo caches a calculated value between renders.

Use:
I use it when an expensive calculation or reference
stability has meaningful performance value.

Example:
A large filtered list based on items and query.

Caution:
I don't use it everywhere because memoization itself
adds complexity and should not be required for
correctness.
```

---

# Part 29 — Top 20 Must-Master Questions

## 117. Before a React Interview, Be Able to Explain These Without Notes ⭐⭐⭐⭐⭐

1. What is React and declarative UI?
2. Props vs state.
3. State as a snapshot.
4. Batching and functional updates.
5. Why state must not be mutated.
6. Controlled vs uncontrolled components.
7. Lifting state and state colocation.
8. How React preserves/resets component state.
9. What causes rendering.
10. Render vs commit.
11. What `useEffect` is actually for.
12. Effect dependencies and cleanup.
13. Stale closures.
14. `useRef` vs state.
15. Context vs reducer vs external state.
16. `memo` vs `useMemo` vs `useCallback`.
17. Reconciliation, keys and Fiber.
18. Suspense and concurrent UI concepts.
19. React 19 Actions / `useActionState` / `useOptimistic` / `use`.
20. Production architecture, security, accessibility and testing.

---

# Part 30 — Complete Revision Map

## 118. React Mental Model ⭐⭐⭐⭐⭐

```text
                         REACT
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
      INPUTS            RENDERING          ESCAPE HATCHES
        │                  │                  │
   props/state        pure component       Effects / refs
   context/store           │                  │
        │                  ↓                  ↓
        └────────────→ element tree     external systems
                           │
                           ↓
                     reconciliation
                           │
                    type / key / position
                           │
                           ↓
                         commit
                           │
                           ↓
                           DOM


STATE
snapshot
   ↓
queued updates
   ↓
batching
   ↓
next render


PERFORMANCE
measure
   ↓
find expensive work
   ↓
improve architecture
   ↓
memoization / transition /
deferred rendering when useful


PRODUCTION
architecture
+ async UI states
+ accessibility
+ security
+ testing


SECURITY
browser = untrusted
        ↓
server validates + authorizes


INTERVIEW RULE
Don't only say WHAT.
Explain WHY + HOW + TRADEOFF.
```

---

## 119. Key Takeaways

- Understand React's mental model instead of memorizing APIs.
- Props are external inputs; state is component memory.
- State belongs to a render snapshot.
- Functional updates solve previous-state queueing cases.
- Never mutate React state.
- Keys define sibling identity during reconciliation.
- Rendering and DOM mutation are different phases.
- Rendering must remain pure.
- Effects synchronize external systems.
- Dependency arrays describe reactive dependencies.
- Cleanup must mirror setup.
- Closures capture values from the render that created them.
- Refs persist without triggering renders.
- Context distributes values; it is not automatic global-state optimization.
- Reducers clarify complex transitions.
- Custom Hooks reuse React logic, not shared state instances.
- Memoization is a performance optimization, not a correctness requirement.
- Understand `memo`, `useMemo`, and `useCallback` separately.
- Concurrent APIs help prioritize user experience; they do not magically make computation faster.
- Reconciliation depends heavily on identity, type, position and keys.
- Fiber is React's internal work-unit architecture.
- Suspense coordinates fallback UI for supported not-ready work.
- Know the important React 19 APIs and their purpose.
- Distinguish React concepts from framework-specific implementations.
- Treat the browser as untrusted for security.
- Build accessibility into semantics and interaction.
- Test user-visible behavior rather than private implementation.
- Strong interview answers include definition, use case, mental model and tradeoff.

---

## Next Lesson

➡️ [Lesson 74 — React Coding and Debugging Interview Problems ⭐⭐⭐⭐⭐](./74-coding-debugging.md)
