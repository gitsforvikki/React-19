# Lesson 76 — Complete React Revision Cheat Sheet ⭐⭐⭐⭐⭐

## 1. Final React Revision

This is the final lesson of the React learning roadmap.

Use it for:

- interview revision
- quick concept recall
- pre-project review
- debugging reminders
- React 19 refresh

Do not memorize React as isolated Hooks.

Remember the core model:

```text
UI = function of current inputs

props
state
context
external subscriptions
        ↓
      render
        ↓
 reconciliation
        ↓
      commit
        ↓
       DOM
```

---

# 2. React in One Definition ⭐⭐⭐⭐⭐

React is a declarative, component-based JavaScript library for building user interfaces.

Components describe UI from current inputs.

When inputs change, React renders again, reconciles identities/trees, and commits necessary host updates.

---

# 3. Declarative vs Imperative

Imperative:

```text
find DOM
change text
add class
hide element
```

Declarative:

```jsx
return isLoggedIn
  ? <Dashboard />
  : <Login />;
```

Describe the desired UI for the current state.

---

# 4. JSX

JSX:

- describes React elements
- allows JavaScript expressions with `{}`
- is not HTML
- uses `className`
- requires component names to be capitalized

```jsx
function Greeting({ name }) {
  return (
    <h1>Hello {name}</h1>
  );
}
```

---

# 5. Components ⭐⭐⭐⭐⭐

Component:

```text
input
(props/state/context)
       ↓
component render
       ↓
React elements
```

Good components:

- have clear responsibility
- compose with other components
- keep rendering pure
- expose intentional APIs

Prefer composition over inheritance for React UI reuse.

---

# 6. Props ⭐⭐⭐⭐⭐

Props are external read-only inputs.

```jsx
<UserCard
  user={user}
  onConnect={handleConnect}
/>
```

Rules:

- parent passes down
- child does not mutate
- callbacks communicate events upward

```text
Parent
  ↓ props
Child
  ↑ callback event
Parent
```

---

# 7. State ⭐⭐⭐⭐⭐

```jsx
const [count, setCount] =
  useState(0);
```

State:

- is component memory
- persists across renders
- updates request rendering
- must be treated as immutable

---

# 8. State Is a Snapshot ⭐⭐⭐⭐⭐

Each render receives its own state values.

```jsx
setCount(count + 1);
console.log(count);
```

The log still sees the current render's snapshot.

Setter:

```text
does not mutate current variable
→ schedules/queues future state
```

---

# 9. Functional Updates ⭐⭐⭐⭐⭐

When next state depends on previous queued state:

```jsx
setCount(c => c + 1);
```

Three updates:

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

```text
0 → 1 → 2 → 3
```

---

# 10. Batching ⭐⭐⭐⭐⭐

React can group multiple updates before rendering.

```text
event
├── state update
├── state update
└── state update
      ↓
process queue
      ↓
render
```

Batching avoids unnecessary intermediate renders.

---

# 11. Immutable State ⭐⭐⭐⭐⭐

Object:

```jsx
setUser(prev => ({
  ...prev,
  name: "Vikash",
}));
```

Array add:

```jsx
setItems(prev => [
  ...prev,
  item,
]);
```

Remove:

```jsx
setItems(prev =>
  prev.filter(
    item => item.id !== id
  )
);
```

Update:

```jsx
setItems(prev =>
  prev.map(item =>
    item.id === id
      ? {
          ...item,
          done: true,
        }
      : item
  )
);
```

Never mutate React state directly.

---

# 12. Minimal State ⭐⭐⭐⭐⭐

Avoid storing what can be calculated.

Bad:

```text
firstName state
lastName state
fullName state
```

Better:

```jsx
const fullName =
  `${firstName} ${lastName}`;
```

Rule:

> Keep the smallest authoritative state possible and derive the rest.

---

# 13. Props vs State ⭐⭐⭐⭐⭐

```text
PROPS
external input
read-only to child

STATE
owned memory
updated by component logic
```

Both participate in rendering.

---

# 14. Controlled vs Uncontrolled ⭐⭐⭐⭐⭐

Controlled:

```jsx
<input
  value={name}
  onChange={e =>
    setName(e.target.value)
  }
/>
```

React state controls value.

Uncontrolled:

```jsx
<input
  defaultValue="Vikash"
  ref={inputRef}
/>
```

DOM maintains current value.

---

# 15. Lifting State Up ⭐⭐⭐⭐⭐

When siblings need coordinated state:

```text
       Parent
       state
      /     \
     ↓       ↓
 Child A   Child B
```

Lift state to the closest common owner.

---

# 16. State Colocation ⭐⭐⭐⭐⭐

Keep state close to where it is used.

```text
local need
→ local state

shared sibling need
→ lift

wide subtree distribution
→ Context may fit

specialized broader state requirements
→ external store may fit
```

Do not globalize by default.

---

# 17. Component Identity ⭐⭐⭐⭐⭐

React state is associated with component identity in the tree.

Important signals:

- type
- position
- key

Change identity:

```text
old component removed
new component created
→ state resets
```

---

# 18. Keys ⭐⭐⭐⭐⭐

Correct:

```jsx
items.map(item => (
  <Item
    key={item.id}
    item={item}
  />
))
```

Keys must be:

- stable
- unique among siblings
- based on logical identity

Avoid changing/random keys.

Avoid index keys when items reorder/insert/delete.

---

# 19. Intentional Reset With Key

```jsx
<Chat
  key={user.id}
  user={user}
/>
```

Changing key intentionally creates a new component identity and resets its state.

---

# 20. Conditional Rendering

Patterns:

```jsx
if (loading) {
  return <Spinner />;
}
```

```jsx
{isAdmin
  ? <Admin />
  : <User />}
```

```jsx
{items.length > 0 && (
  <List />
)}
```

Remember:

```jsx
{items.length && <List />}
```

can render `0`.

---

# 21. Events

Pass function:

```jsx
<button
  onClick={handleClick}
>
  Save
</button>
```

Do not invoke during render:

```jsx
onClick={handleClick()}
```

Arguments:

```jsx
onClick={() =>
  deleteUser(id)
}
```

---

# 22. preventDefault vs stopPropagation

```text
preventDefault
→ stop browser default behavior

stopPropagation
→ stop event propagation
```

Different purposes.

---

# 23. Render and Re-render ⭐⭐⭐⭐⭐

Common triggers:

- state update
- parent render
- consumed Context change
- external store subscription

Render means React executes component logic to calculate next UI.

Render does **not** mean the whole DOM is rebuilt.

---

# 24. Render → Reconcile → Commit ⭐⭐⭐⭐⭐

```text
trigger
   ↓
RENDER
calculate next UI
pure
   ↓
RECONCILIATION
match identities
   ↓
COMMIT
apply host/DOM changes
refs/lifecycle work
   ↓
browser paint
```

Render can occur without meaningful DOM changes.

---

# 25. Render Purity ⭐⭐⭐⭐⭐

Do not perform side effects during render.

Bad:

```jsx
function Component() {
  localStorage.setItem(
    "x",
    "y"
  );

  return ...;
}
```

Render should calculate UI.

Side effects belong in event handlers or synchronization mechanisms as appropriate.

---

# 26. useEffect ⭐⭐⭐⭐⭐

Correct mental model:

> `useEffect` synchronizes React with an external system.

Examples:

- subscription
- connection
- browser API
- timer
- third-party widget
- network synchronization when appropriate

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

# 27. You Might Not Need an Effect ⭐⭐⭐⭐⭐

Do not use Effect for:

### Derived data

```jsx
const total =
  price * quantity;
```

### User action

Do work in event handler.

### Reset by identity

Use meaningful `key` when appropriate.

Question:

> What external system am I synchronizing with?

If none, reconsider the Effect.

---

# 28. Effect Dependencies ⭐⭐⭐⭐⭐

Dependencies are reactive values used by synchronization.

```jsx
useEffect(() => {
  connect(roomId);
}, [roomId]);
```

Do not omit dependencies just to control execution frequency.

Change the code/architecture instead.

---

# 29. Effect Cleanup ⭐⭐⭐⭐⭐

```jsx
useEffect(() => {
  subscribe();

  return () => {
    unsubscribe();
  };
}, []);
```

Cleanup happens:

- before re-synchronization
- on unmount
- in development Strict Mode checks

Setup and cleanup should be symmetrical.

---

# 30. Strict Mode Effect Behavior ⭐⭐⭐⭐⭐

Development Strict Mode may intentionally exercise:

```text
setup
↓
cleanup
↓
setup
```

This helps reveal unsafe Effects.

Do not suppress the behavior with a ref.

Fix the Effect.

---

# 31. Stale Closure ⭐⭐⭐⭐⭐

Every render creates its own values/functions.

A callback remembers the render that created it.

```text
render 1
count = 0
creates callback
      ↓
callback runs later
      ↓
still sees render 1 count
```

Possible solutions depend on intended semantics:

- correct dependencies
- functional updater
- ref for latest mutable value
- Effect Event for appropriate non-reactive Effect logic

---

# 32. Async Race Condition ⭐⭐⭐⭐⭐

```text
request A starts
request B starts

B finishes
→ show B

A finishes later
→ must not overwrite B
```

Use appropriate:

- cancellation
- stale-result ignore
- request identity
- data-layer coordination

Race condition is not the same as stale closure.

---

# 33. useRef ⭐⭐⭐⭐⭐

```jsx
const ref =
  useRef(initialValue);
```

Returns stable object:

```js
{
  current: value
}
```

Mutation does not trigger render.

Use for:

- DOM nodes
- timer IDs
- instance-like mutable values
- latest-value patterns when appropriate

---

# 34. State vs Ref ⭐⭐⭐⭐⭐

```text
STATE
persists
triggers render

REF
persists
does not trigger render
```

Visible UI data normally belongs in state.

---

# 35. DOM Refs

```jsx
const inputRef =
  useRef(null);

<input ref={inputRef} />

<button
  onClick={() =>
    inputRef.current?.focus()
  }
>
  Focus
</button>
```

DOM ref is assigned during commit.

Do not assume it exists during initial render.

---

# 36. React 19 Ref Prop

Modern React 19 function components can receive `ref` as a prop:

```jsx
function MyInput({
  ref,
  ...props
}) {
  return (
    <input
      ref={ref}
      {...props}
    />
  );
}
```

Older React code commonly uses `forwardRef`.

---

# 37. useImperativeHandle

Expose a restricted imperative API:

```jsx
useImperativeHandle(
  ref,
  () => ({
    focus() {
      inputRef.current?.focus();
    },
  }),
  []
);
```

Use sparingly.

Prefer declarative props/state when possible.

---

# 38. Context ⭐⭐⭐⭐⭐

Context distributes a value through a provider subtree.

Useful for:

- theme
- auth/session interface
- locale
- feature-shared state

Context:

```text
solves distribution
≠ automatically solves performance
≠ automatically means global state
```

---

# 39. useReducer ⭐⭐⭐⭐⭐

```jsx
const [state, dispatch] =
  useReducer(
    reducer,
    initialState
  );
```

Reducer:

```jsx
function reducer(
  state,
  action
) {
  switch (action.type) {
    case "added":
      return ...;

    default:
      return state;
  }
}
```

Useful for complex/related transitions.

Reducer must not mutate existing state.

---

# 40. State Management Decision ⭐⭐⭐⭐⭐

```text
local UI state
→ useState/useReducer

shared subtree
→ lift / Context

external server state
→ data layer/framework/store suited to server state

complex broad client state
→ external store if justified
```

Use the simplest tool that matches ownership.

---

# 41. Custom Hooks ⭐⭐⭐⭐⭐

```jsx
function useOnlineStatus() {
  ...
}
```

Custom Hooks reuse React logic.

They do not automatically share the same state instance between callers.

---

# 42. Rules of Hooks ⭐⭐⭐⭐⭐

Call ordinary Hooks:

1. at top level
2. from React components/custom Hooks

Do not conditionally alter Hook call order.

React relies on consistent ordering.

---

# 43. React.memo ⭐⭐⭐⭐⭐

```jsx
const Card =
  memo(function Card({
    user,
  }) {
    ...
  });
```

Can skip component rendering when props compare unchanged.

It is an optimization.

Not a correctness tool.

---

# 44. useMemo ⭐⭐⭐⭐⭐

Caches calculated value:

```jsx
const filtered =
  useMemo(
    () =>
      filter(items, query),
    [items, query]
  );
```

Use when calculation/reference stability has meaningful value.

Not for every calculation.

---

# 45. useCallback ⭐⭐⭐⭐⭐

Caches function reference:

```jsx
const handleSave =
  useCallback(() => {
    save(user);
  }, [user]);
```

Useful when function identity matters.

It does not make the function body inherently faster.

---

# 46. Memo Comparison ⭐⭐⭐⭐⭐

```text
React.memo
→ component render optimization

useMemo
→ calculated value identity

useCallback
→ function identity
```

Always ask:

> Is the avoided work worth the complexity?

---

# 47. Performance Workflow ⭐⭐⭐⭐⭐

```text
measure/profile
      ↓
find actual bottleneck
      ↓
fix architecture
      ↓
target optimization
      ↓
measure again
```

Potential tools:

- state colocation
- remove redundant state/Effects
- memoization
- code splitting
- transitions
- deferred values
- virtualization
- pagination

---

# 48. useTransition ⭐⭐⭐⭐⭐

```jsx
const [
  isPending,
  startTransition,
] = useTransition();

startTransition(() => {
  setTab(nextTab);
});
```

Marks suitable updates as non-urgent.

Does not make computation itself faster.

---

# 49. useDeferredValue

```jsx
const deferredQuery =
  useDeferredValue(query);
```

Allows expensive dependent UI to lag behind urgent input.

Useful for responsiveness.

---

# 50. React.lazy

```jsx
const Analytics =
  lazy(() =>
    import("./Analytics")
  );
```

Use with Suspense:

```jsx
<Suspense
  fallback={<Skeleton />}
>
  <Analytics />
</Suspense>
```

Supports code splitting.

---

# 51. Suspense ⭐⭐⭐⭐⭐

Suspense provides a boundary for fallback UI while supported child work is not ready.

```jsx
<Suspense
  fallback={<Skeleton />}
>
  <Content />
</Suspense>
```

Place boundaries around meaningful UX regions.

---

# 52. Virtual DOM ⭐⭐⭐⭐⭐

Useful simplified model:

```text
props/state change
      ↓
new React element representation
      ↓
reconciliation
      ↓
necessary host changes
```

Do not say:

> React rebuilds the entire real DOM.

---

# 53. Reconciliation ⭐⭐⭐⭐⭐

React determines how previous and next trees correspond.

Important identity signals:

- type
- key
- position

Correct keys are essential for list identity.

---

# 54. Fiber ⭐⭐⭐⭐⭐

Fiber is React's internal work-unit/tree architecture.

It enables React to organize rendering work for capabilities such as:

- prioritization
- interruption
- resumption
- concurrent rendering

Do not depend on private Fiber internals in application code.

---

# 55. Render vs Commit ⭐⭐⭐⭐⭐

```text
RENDER
calculate
pure
may be restarted/interrupted

COMMIT
apply finalized host changes
refs/lifecycle-related work
```

This distinction explains many React behaviors.

---

# 56. Portals

Portal renders children into another DOM location while keeping them in the same React tree.

Useful for:

- modals
- overlays
- tooltips

React event/context relationships follow the React tree.

---

# 57. Error Boundaries ⭐⭐⭐⭐⭐

Error Boundaries isolate certain descendant rendering/lifecycle failures and show fallback UI.

Use around meaningful failure regions.

They do not automatically catch every async/event-handler error.

---

# 58. Compound Components

Pattern:

```jsx
<Tabs>
  <Tabs.List />
  <Tabs.Panel />
</Tabs>
```

Useful for flexible component APIs with coordinated internal behavior.

Avoid unnecessary complexity for simple components.

---

# 59. HOC and Render Props

Older/common reuse patterns:

```text
Higher-Order Component
component → enhanced component

Render Prop
component receives function describing UI
```

Custom Hooks often provide simpler logic reuse for modern function components, but HOCs/render props remain relevant in existing code/libraries.

---

# 60. forwardRef

Historically used to forward refs through function components.

React 19 supports `ref` as a prop for modern function components, reducing the need for `forwardRef` in new code.

Know it for older codebases and compatibility.

---

# 61. React 19 — Actions ⭐⭐⭐⭐⭐

Actions support asynchronous mutation workflows with React-integrated pending/error/form behavior in supported APIs.

Mental model:

```text
user mutation
    ↓
Action
    ↓
pending / result / error
    ↓
UI coordination
```

---

# 62. useActionState ⭐⭐⭐⭐⭐

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

Useful when Action result should become UI state.

---

# 63. useFormStatus

A descendant of a form can observe submission status.

Useful for submit controls:

```text
form pending
→ disable/show pending UI
```

without manually threading status through every component.

---

# 64. useOptimistic ⭐⭐⭐⭐⭐

Optimistically project UI while authoritative operation is pending.

```text
action
 ↓
optimistic UI
 ↓
server
 ├── success → confirm
 └── failure → reconcile + feedback
```

Optimistic state is not authoritative state.

---

# 65. use() ⭐⭐⭐⭐⭐

`use` reads supported resources such as a Promise or Context in supported React rendering environments.

Pending Promise:

```text
use(promise)
   ↓
suspend
   ↓
nearest Suspense fallback
```

It has special React semantics.

---

# 66. React 19 Metadata / Resources

React 19 adds capabilities around document metadata and resource handling/preloading.

Know that React can coordinate certain metadata/resource declarations, while framework behavior may add its own higher-level conventions.

Do not confuse framework APIs with React core APIs.

---

# 67. Server Components ⭐⭐⭐⭐⭐

Conceptual model:

```text
SERVER COMPONENT
server environment
non-interactive rendering/data work
can reduce client JS

CLIENT COMPONENT
interactivity
state
Effects
browser APIs
```

Server Components are not simply traditional SSR.

Framework-specific implementation belongs to the framework.

---

# 68. Data Fetching ⭐⭐⭐⭐⭐

For client-side Effect fetching, handle:

- loading
- error
- cleanup
- stale results
- race conditions

But complex server data often benefits from framework/data-layer features:

- caching
- deduplication
- prefetching
- retries
- mutation coordination

Do not make `useEffect + fetch` the universal answer.

---

# 69. Architecture ⭐⭐⭐⭐⭐

Prefer clear ownership.

```text
feature/
├── components
├── hooks
├── data/api
└── domain helpers
```

Shared code should be genuinely shared.

Avoid:

- giant global folders
- circular dependencies
- premature abstractions
- giant components with mixed responsibilities

---

# 70. Separation of Concerns

Separate when useful:

```text
UI
React behavior
domain calculation
data access
server authority
```

Do not create layers merely for architecture appearance.

---

# 71. Async UI States ⭐⭐⭐⭐⭐

Think beyond:

```text
isLoading
```

Consider:

```text
idle
initial loading
success
empty
error
retry
refreshing
mutation pending
mutation error
partial failure
```

Initial loading and background refreshing are different UX states.

---

# 72. Accessibility ⭐⭐⭐⭐⭐

Start with semantic HTML.

```text
button → action
link → navigation
label → input name
headings → structure
list → list semantics
```

Also:

- keyboard support
- visible focus
- accessible names
- error relationships
- focus management
- reduced motion
- meaningful dynamic announcements

ARIA supplements native semantics; it does not replace behavior.

---

# 73. Security ⭐⭐⭐⭐⭐

Core rule:

> Never trust the client.

```text
React checks
→ UX

server checks
→ security
```

Server must enforce:

- authentication
- authorization
- validation
- ownership
- prices
- inventory
- payment verification
- file rules
- business invariants

---

# 74. XSS ⭐⭐⭐⭐⭐

React escapes normal text interpolation by default.

Danger:

```jsx
dangerouslySetInnerHTML={{
  __html: content,
}}
```

If untrusted HTML must be rendered, use a trusted sanitization strategy.

Do not build HTML sanitization with a few regex replacements.

---

# 75. Frontend Secrets

Anything delivered to browser JavaScript should be treated as public.

Never ship:

- database passwords
- secret payment keys
- signing secrets
- webhook secrets
- private service credentials

Environment variable does not automatically mean secret.

---

# 76. Authentication vs Authorization ⭐⭐⭐⭐⭐

```text
authentication
→ Who are you?

authorization
→ Are you allowed?
```

Hidden admin button is not authorization.

The server must reject unauthorized direct requests.

---

# 77. CORS vs Security

```text
CORS
≠ authentication
≠ authorization
```

CORS is a browser cross-origin policy.

Your API still requires real security.

---

# 78. Testing ⭐⭐⭐⭐⭐

Test user-visible behavior.

```text
render
→ interact
→ observe result
```

Prefer semantic queries such as role/name.

Avoid testing:

- private state
- Hook internals
- exact implementation calls
- React itself

---

# 79. Testing Levels

```text
UNIT
pure logic

COMPONENT
UI behavior

INTEGRATION
feature pieces together

E2E
critical full journey
```

Choose based on risk and confidence.

---

# 80. Test Async States

Test:

- loading
- success
- empty
- error
- retry
- optimistic success
- optimistic failure
- race conditions when relevant

Do not test only the happy path.

---

# 81. Common React Mistakes ⭐⭐⭐⭐⭐

Avoid:

1. Mutating state.
2. Copying props into state unnecessarily.
3. Storing derived values.
4. Using index/random keys for changing lists.
5. Defining component types in unstable places.
6. Using Effects for internal calculations.
7. Omitting Effect dependencies.
8. Missing Effect cleanup.
9. Confusing stale closures with races.
10. Using refs for visible state.
11. Memoizing everything.
12. Globalizing all state.
13. Giant Context providers.
14. Treating client checks as security.
15. Ignoring accessibility.
16. Testing implementation details.
17. Optimizing without profiling.
18. Confusing React core with framework-specific behavior.

---

# 82. Debugging Checklist ⭐⭐⭐⭐⭐

When React behaves unexpectedly:

```text
STATE
- mutation?
- snapshot?
- functional update?
- duplicated state?

IDENTITY
- key?
- type?
- position?
- remount?

EFFECT
- needed?
- dependencies?
- cleanup?
- stale closure?

ASYNC
- race?
- cancellation?
- stale result?

RENDER
- what triggered it?
- is it actually expensive?

PERFORMANCE
- measured?
- large list?
- high-level state?
- unstable props?

PRODUCTION
- loading/error?
- accessibility?
- security?
```

---

# 83. Performance Checklist ⭐⭐⭐⭐⭐

Do not ask first:

> Where can I add `useMemo`?

Ask:

```text
1. What is slow?
2. Is it network, JS, render, DOM, bundle, third party?
3. What does profiler show?
4. Is state too high?
5. Is work unnecessary?
6. Is list too large?
7. Would code splitting help?
8. Would transition/deferred rendering improve responsiveness?
9. Is memoization targeted and useful?
10. Did performance improve after the change?
```

---

# 84. State Decision Tree ⭐⭐⭐⭐⭐

```text
Does value affect rendered UI?
      │
   YES│
      ↓
Does it already exist in props/state?
      │
   YES│
      ↓
derive it
don't duplicate

If new state:
who needs it?
      │
 one component → local
 siblings       → lift
 subtree        → Context may fit
 broad complex  → external store may fit

Does it need to persist but NOT render?
      ↓
ref may fit
```

---

# 85. Effect Decision Tree ⭐⭐⭐⭐⭐

```text
Need to respond to user action?
      ↓
event handler

Can value be calculated from props/state?
      ↓
derive during render

Need to reset subtree for identity?
      ↓
key may fit

Need to synchronize external system?
      ↓
Effect
      │
      ├── dependencies
      └── cleanup
```

This is one of the most important React decision trees.

---

# 86. Memoization Decision Tree

```text
Is there a measured/reasonable expensive problem?
       │
      NO
       ↓
keep code simple

      YES
       ↓
What identity/work matters?

component rendering
→ memo

calculated value
→ useMemo

function identity
→ useCallback
```

Then measure again.

---

# 87. React Interview Answer Formula ⭐⭐⭐⭐⭐

For any major concept:

```text
1. WHAT
2. WHY
3. HOW
4. EXAMPLE
5. TRADEOFF / MISTAKE
```

Example:

```text
useRef is a Hook that returns a stable mutable object.
It is useful when a value must persist across renders
without triggering rendering, such as a DOM node or
timer ID.

Unlike state, changing ref.current does not schedule a
render, so I would not use it for data that must update
the visible UI.
```

---

# 88. Top Interview Questions ⭐⭐⭐⭐⭐

You should answer these confidently:

1. What is React?
2. Declarative vs imperative UI?
3. Props vs state?
4. What does state as a snapshot mean?
5. What is batching?
6. Why use functional state updates?
7. Why should state be immutable?
8. Controlled vs uncontrolled components?
9. What is lifting state up?
10. What is state colocation?
11. How does React preserve/reset state?
12. Why are keys important?
13. What triggers a render?
14. Render vs commit?
15. What is reconciliation?
16. What is Fiber?
17. What is `useEffect` actually for?
18. How do Effect dependencies work?
19. Why is cleanup required?
20. What is a stale closure?
21. Stale closure vs race condition?
22. `useRef` vs state?
23. Context vs external store?
24. `useState` vs `useReducer`?
25. What is a custom Hook?
26. Rules of Hooks?
27. `memo` vs `useMemo` vs `useCallback`?
28. `useTransition`?
29. `useDeferredValue`?
30. Suspense?
31. Error Boundaries?
32. React 19 Actions?
33. `useActionState`?
34. `useOptimistic`?
35. `use()`?
36. React 19 ref changes?
37. Server vs Client Component concepts?
38. How do you structure a large React app?
39. How do you optimize a slow React page?
40. How do you secure/test/accessibilize production React UI?

---

# 89. React 19 Quick Map ⭐⭐⭐⭐⭐

```text
React 19
│
├── Actions
│   └── async mutation model
│
├── useActionState
│   └── Action result + pending
│
├── useFormStatus
│   └── form submission status
│
├── useOptimistic
│   └── temporary optimistic projection
│
├── use()
│   └── read supported resource
│
├── ref as prop
│   └── less need for forwardRef
│
└── metadata/resources
    └── improved document/resource coordination
```

---

# 90. Production React Checklist ⭐⭐⭐⭐⭐

Before shipping a feature:

### Correctness

- state ownership clear?
- derived state avoided?
- immutable updates?
- stable keys?
- race conditions handled?
- Effect cleanup correct?

### UX

- loading?
- empty?
- error?
- retry?
- pending?
- optimistic rollback?

### Accessibility

- semantic controls?
- labels?
- keyboard?
- focus?
- accessible names?

### Security

- server authorization?
- server validation?
- no client secrets?
- safe HTML?
- server-authoritative money/ownership?

### Performance

- measured?
- unnecessary high-level state?
- huge list?
- targeted optimization?

### Testing

- main flow?
- failure flow?
- async states?
- behavior rather than internals?

---

# 91. React Architecture Map ⭐⭐⭐⭐⭐

```text
                       APPLICATION
                            │
                     feature boundaries
                            │
       ┌────────────────────┼────────────────────┐
       ↓                    ↓                    ↓
    UI STATE            SERVER DATA          URL STATE
       │                    │                    │
 useState/reducer      data layer /          navigation
 lift/context/store    framework             semantics
       │                    │                    │
       └────────────────────┼────────────────────┘
                            ↓
                         RENDER
                            ↓
                     RECONCILIATION
                            ↓
                          COMMIT
                            ↓
                           DOM

Escape hatches:
Effect → external synchronization
Ref    → persistent non-rendering mutable value

Production:
accessibility
security
performance
testing
```

---

# 92. React Rendering Map ⭐⭐⭐⭐⭐

```text
EVENT / STATE / CONTEXT / STORE
              │
              ↓
         update queued
              │
              ↓
            render
        (pure calculation)
              │
              ↓
       reconciliation
     type + key + position
              │
              ↓
            commit
      DOM / refs / lifecycle
              │
              ↓
        browser paints
              │
              ↓
      passive Effects run
```

This mental model explains a large percentage of React interview questions.

---

# 93. Effect + Closure Map ⭐⭐⭐⭐⭐

```text
Render #1
state = A
creates Effect callback
        │
        ↓
callback remembers A

Render #2
state = B
creates new callback
        │
        ↓
dependency changed?
  │
 YES
  ↓
cleanup old synchronization
setup new synchronization
```

Dependencies connect synchronization to the render values it uses.

---

# 94. Identity Map ⭐⭐⭐⭐⭐

```text
same type
same position
same key
     ↓
React can preserve identity/state

key/type changes
     ↓
new identity
     ↓
state resets
```

This is why keys are about identity, not merely removing warnings.

---

# 95. Security Map ⭐⭐⭐⭐⭐

```text
              BROWSER
          attacker-controlled
                 │
        React validation/UI
                 │
          untrusted request
                 ↓
        ═ TRUST BOUNDARY ═
                 ↓
               SERVER
        ┌────────┼────────┐
        ↓        ↓        ↓
      auth    authorize validate
        │        │        │
        └────────┼────────┘
                 ↓
         business rules
                 ↓
              database
```

Never trust client:

- role
- resource ID
- price
- total
- ownership
- hidden button
- validation
- secret

---

# 96. Testing Map

```text
                     CONFIDENCE
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
      UNIT           COMPONENT        INTEGRATION
   pure logic       UI behavior       feature flow
        │                │                │
        └────────────────┼────────────────┘
                         ↓
                        E2E
                  critical journeys

Prefer:
user behavior
semantic queries
success + failure
deterministic tests

Avoid:
private state
implementation coupling
arbitrary sleeps
over-mocking
```

---

# 97. 10 Golden React Rules ⭐⭐⭐⭐⭐

1. **Keep rendering pure.**
2. **Treat state as immutable snapshots.**
3. **Keep the smallest possible source of truth.**
4. **Use stable identity and keys.**
5. **Use Effects only for external synchronization.**
6. **Use refs only for persistent values that should not drive rendering.**
7. **Keep state close to its consumers.**
8. **Optimize after understanding/measuring the problem.**
9. **Treat the browser as untrusted.**
10. **Design for accessibility, failure, and testing—not only the happy path.**

---

# 98. 30-Second React Interview Summary ⭐⭐⭐⭐⭐

```text
React is a declarative component-based UI library.

Components render from props, state and context. State is
a snapshot, updates are queued/batched, and state should
be treated immutably.

When inputs change React renders, reconciles component
identity using factors such as type, position and keys,
then commits necessary DOM changes.

Effects are for synchronizing with external systems,
while refs store persistent mutable values that do not
trigger rendering.

I keep state close to consumers, derive values instead
of duplicating state, and use Context/reducers/external
stores only when their ownership requirements justify
them.

For performance I profile first and then use targeted
techniques such as memoization, transitions, deferred
rendering, code splitting or virtualization.

In production I explicitly design loading/error/empty
states, accessibility, testing and security, and I never
treat the React client as the authorization or business
rule boundary.
```

---

# 99. Final Learning Map

```text
SECTION 1
React Foundations
      ↓
SECTION 2
State + Rendering
      ↓
SECTION 3
Effects + Refs
      ↓
SECTION 4
Forms
      ↓
SECTION 5
State Management
      ↓
SECTION 6
Reusable Logic
      ↓
SECTION 7
Performance
      ↓
SECTION 8
React Internals
      ↓
SECTION 9
Advanced React
      ↓
SECTION 10
React 19
      ↓
SECTION 11
Server + Data Concepts
      ↓
SECTION 12
Production React
      ↓
SECTION 13
Interview Mastery
      ↓
COMPLETE REACT MENTAL MODEL
```

---

# 100. What You Should Be Able to Do Now

After completing this roadmap, you should be able to:

- explain React from fundamentals to internals
- design state ownership intentionally
- reason about rendering and component identity
- use Effects and refs correctly
- build reusable Hooks/components
- choose state-management approaches based on requirements
- diagnose common React bugs
- optimize based on evidence
- explain reconciliation and Fiber at interview level
- use modern React 19 concepts
- reason about Server Component boundaries conceptually
- design production loading/error/empty states
- build accessible interfaces
- recognize frontend security boundaries
- test React behavior effectively
- answer coding, debugging and architecture interview questions

The next step is not collecting more APIs.

It is applying these mental models repeatedly in real applications.

---

# React 19 Learning Roadmap — Completed ✅

You have reached the end of the structured React roadmap:

**76 lessons — Foundations → Advanced React → React 19 → Production → Interview Mastery**

For revision, return to the topics you cannot explain without notes, rebuild small examples, debug intentionally broken code, and practice explaining tradeoffs aloud.

A React developer becomes stronger not by memorizing more Hooks, but by understanding **state, identity, rendering, synchronization, ownership, and boundaries**.
