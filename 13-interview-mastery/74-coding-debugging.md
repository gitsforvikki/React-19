# Lesson 74 — React Coding and Debugging Interview Problems ⭐⭐⭐⭐⭐

## 1. Why This Lesson Matters

React interviews often do not ask only definitions.

You may receive code and be asked:

```text
What is wrong?
Why does it happen?
How would you fix it?
Is the fix always necessary?
What React concept is being tested?
```

A strong debugging answer follows this process:

```text
Observe symptom
      ↓
Identify ownership / render / Effect / identity issue
      ↓
Explain WHY React behaves this way
      ↓
Apply smallest correct fix
      ↓
Mention tradeoffs
```

Do not immediately add `useEffect`, `useMemo`, `useCallback`, or refs without understanding the problem.

---

# Part 1 — State Snapshot Problems

## 2. Problem: Why Does This Increment Only Once? ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const [count, setCount] =
    useState(0);

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

### What Is Happening?

The current render has one `count` snapshot.

If `count === 0`, each call requests:

```text
setCount(1)
setCount(1)
setCount(1)
```

### Correct Approach

When each update depends on the previously queued value:

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
setCount(c => c + 1);
```

Result:

```text
0 → 1 → 2 → 3
```

### Interview Concept

- state as snapshot
- batching
- state update queue
- functional updates

---

## 3. Problem: Why Is console.log Showing the Old Value? ⭐⭐⭐⭐⭐

```jsx
function handleClick() {
  setCount(count + 1);
  console.log(count);
}
```

### Explanation

`setCount` schedules an update.

It does not mutate the `count` variable belonging to the currently executing render.

If you need the calculated next value immediately:

```jsx
const nextCount =
  count + 1;

setCount(nextCount);
console.log(nextCount);
```

Do not expect the current render's state variable to change in place.

---

# Part 2 — Mutation Problems

## 4. Problem: Object State Mutation ⭐⭐⭐⭐⭐

Bug:

```jsx
function updateName() {
  user.name = "Vikash";
  setUser(user);
}
```

### Why Is This Wrong?

You mutated the existing state object and passed the same reference back.

React state should be treated as immutable.

### Fix

```jsx
setUser(prev => ({
  ...prev,
  name: "Vikash",
}));
```

### Interview Concept

```text
previous object
      ↓
do not mutate
      ↓
create new object
      ↓
React receives new state value
```

---

## 5. Problem: Array push()

Bug:

```jsx
function addSkill(skill) {
  skills.push(skill);
  setSkills(skills);
}
```

Fix:

```jsx
setSkills(prev => [
  ...prev,
  skill,
]);
```

Useful immutable operations:

```text
add     → spread / concat
remove  → filter
update  → map
sort    → copy first
reverse → copy first
```

---

## 6. Problem: Shallow Copy Still Mutates Nested Object

Bug:

```jsx
const nextUsers = [...users];

nextUsers[0].name =
  "Vikash";

setUsers(nextUsers);
```

The array is new, but:

```text
nextUsers[0]
and
users[0]
```

still point to the same object.

Fix the changed level too:

```jsx
setUsers(prev =>
  prev.map(user =>
    user.id === id
      ? {
          ...user,
          name: "Vikash",
        }
      : user
  )
);
```

---

# Part 3 — Keys and Identity

## 7. Problem: Input Values Move to the Wrong Row ⭐⭐⭐⭐⭐

```jsx
{users.map(
  (user, index) => (
    <UserRow
      key={index}
      user={user}
    />
  )
)}
```

Then the list is reordered.

### Why Can This Break?

The key represents position instead of logical identity.

After reordering:

```text
key 0
previously → Alice
now        → Bob
```

React may preserve component state against the wrong logical item.

### Fix

```jsx
<UserRow
  key={user.id}
  user={user}
/>
```

Use stable data identity.

---

## 8. Problem: Component State Resets on Every Parent Render

Bug:

```jsx
function App() {
  return (
    <Form
      key={Math.random()}
    />
  );
}
```

Every render creates a different key.

React sees:

```text
old Form identity removed
new Form identity created
```

So state resets.

### Fix

Remove the unstable key or use a meaningful stable identity.

---

## 9. Problem: Component Defined Inside Component

```jsx
function Parent() {
  function Child() {
    const [text, setText] =
      useState("");

    return (
      <input
        value={text}
        onChange={e =>
          setText(
            e.target.value
          )
        }
      />
    );
  }

  return <Child />;
}
```

### Problem

Each Parent render creates a new `Child` function identity.

This can cause React to treat it as a different component type and reset state.

### Better

```jsx
function Child() {
  const [text, setText] =
    useState("");

  return (
    <input
      value={text}
      onChange={e =>
        setText(e.target.value)
      }
    />
  );
}

function Parent() {
  return <Child />;
}
```

Define component types at module level unless you have a very specific reason not to.

---

# Part 4 — Derived State

## 10. Problem: Unnecessary Effect ⭐⭐⭐⭐⭐

```jsx
const [firstName, setFirstName] =
  useState("");

const [lastName, setLastName] =
  useState("");

const [fullName, setFullName] =
  useState("");

useEffect(() => {
  setFullName(
    firstName + " " + lastName
  );
}, [firstName, lastName]);
```

### Problem

`fullName` can be calculated directly.

The Effect creates:

```text
render
→ Effect
→ setState
→ extra render
```

### Better

```jsx
const fullName =
  `${firstName} ${lastName}`;
```

### Interview Rule

If a value can be derived from current props/state during render, usually derive it during render.

---

## 11. Problem: Filtered Data Stored in State

```jsx
const [
  filteredUsers,
  setFilteredUsers,
] = useState([]);

useEffect(() => {
  setFilteredUsers(
    users.filter(user =>
      user.name.includes(query)
    )
  );
}, [users, query]);
```

Better:

```jsx
const filteredUsers =
  users.filter(user =>
    user.name.includes(query)
  );
```

If the calculation is genuinely expensive and profiling justifies it:

```jsx
const filteredUsers =
  useMemo(
    () =>
      users.filter(user =>
        user.name.includes(
          query
        )
      ),
    [users, query]
  );
```

Do not use `useMemo` merely because a value is derived.

---

# Part 5 — Controlled Inputs

## 12. Problem: Input Cannot Be Edited

```jsx
<input value={name} />
```

A controlled input needs a way to update its controlling state.

Fix:

```jsx
<input
  value={name}
  onChange={e =>
    setName(e.target.value)
  }
/>
```

Or use an uncontrolled input when that is actually the intended design.

---

## 13. Problem: Controlled/Uncontrolled Warning

Potential problem:

```jsx
const [name, setName] =
  useState();

return (
  <input
    value={name}
    onChange={e =>
      setName(e.target.value)
    }
  />
);
```

Initially `name` is `undefined`, then becomes a string.

For a controlled text input, initialize consistently:

```jsx
const [name, setName] =
  useState("");
```

Avoid switching an input between controlled and uncontrolled modes unintentionally.

---

# Part 6 — Effect Dependencies

## 14. Problem: Missing Dependency ⭐⭐⭐⭐⭐

```jsx
useEffect(() => {
  const connection =
    createConnection(roomId);

  connection.connect();

  return () =>
    connection.disconnect();
}, []);
```

### Bug

The Effect uses `roomId`, but declares no changing dependency.

When room changes, synchronization remains attached to the old room.

### Fix

```jsx
useEffect(() => {
  const connection =
    createConnection(roomId);

  connection.connect();

  return () =>
    connection.disconnect();
}, [roomId]);
```

Do not silence dependency linting to force the behavior you want.

Change the code so dependencies correctly describe synchronization.

---

## 15. Problem: Object Dependency Causes Effect to Re-run

```jsx
const options = {
  roomId,
};

useEffect(() => {
  connect(options);
}, [options]);
```

`options` is a new object every render.

### Better

If the object is only needed inside the Effect:

```jsx
useEffect(() => {
  const options = {
    roomId,
  };

  connect(options);
}, [roomId]);
```

This removes the unnecessary object dependency.

---

## 16. Problem: Function Dependency Changes Every Render

```jsx
function createOptions() {
  return {
    roomId,
  };
}

useEffect(() => {
  connect(
    createOptions()
  );
}, [createOptions]);
```

The function is recreated on each render.

Often the simplest fix is moving the logic inside the Effect:

```jsx
useEffect(() => {
  function createOptions() {
    return {
      roomId,
    };
  }

  connect(
    createOptions()
  );
}, [roomId]);
```

Do not automatically reach for `useCallback`.

---

# Part 7 — Cleanup

## 17. Problem: Duplicate Event Listeners ⭐⭐⭐⭐⭐

Bug:

```jsx
useEffect(() => {
  window.addEventListener(
    "resize",
    handleResize
  );
}, []);
```

No cleanup exists.

Fix:

```jsx
useEffect(() => {
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

Setup and cleanup should be symmetrical.

---

## 18. Problem: Interval Continues After Component Leaves

```jsx
useEffect(() => {
  setInterval(() => {
    refresh();
  }, 5000);
}, []);
```

Fix:

```jsx
useEffect(() => {
  const id = setInterval(
    refresh,
    5000
  );

  return () =>
    clearInterval(id);
}, [refresh]);
```

But now ask:

```text
Is refresh stable?
Should it be defined inside Effect?
Does it use reactive values?
```

The correct dependency design depends on the surrounding code.

---

# Part 8 — Stale Closures

## 19. Problem: Interval Always Sees 0 ⭐⭐⭐⭐⭐

```jsx
const [count, setCount] =
  useState(0);

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

The interval callback captures the initial render's `count`.

If synchronization should follow `count`:

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
}, [count]);
```

If you need different semantics, choose a solution based on that requirement rather than hiding the dependency.

---

## 20. Problem: Delayed Increment Uses Old State

```jsx
function handleClick() {
  setTimeout(() => {
    setCount(count + 1);
  }, 1000);
}
```

If multiple delayed updates should each increment the latest queued state:

```jsx
setTimeout(() => {
  setCount(c => c + 1);
}, 1000);
```

Functional updates are useful when the next state depends on previous state.

---

# Part 9 — Async Race Conditions

## 21. Problem: Search Shows Old Result ⭐⭐⭐⭐⭐

```jsx
useEffect(() => {
  fetchUsers(query)
    .then(result => {
      setUsers(result);
    });
}, [query]);
```

Sequence:

```text
query = "rea"
request A starts

query = "react"
request B starts

B finishes
→ React results

A finishes later
→ stale "rea" results overwrite UI
```

### Ignore Stale Result Pattern

```jsx
useEffect(() => {
  let ignore = false;

  async function load() {
    const result =
      await fetchUsers(query);

    if (!ignore) {
      setUsers(result);
    }
  }

  load();

  return () => {
    ignore = true;
  };
}, [query]);
```

### Cancellation

If the API supports `AbortSignal`, cancellation may also be appropriate.

Core interview point:

```text
stale closure
≠
out-of-order async race
```

---

# Part 10 — Infinite Effect Loops

## 22. Problem: Effect Keeps Rendering ⭐⭐⭐⭐⭐

```jsx
useEffect(() => {
  setUser({
    name: "Vikash",
  });
}, [user]);
```

Flow:

```text
Effect
→ new user object
→ state update
→ render
→ user changed
→ Effect
→ ...
```

The first question should be:

> Why is this Effect needed?

If the value is static initialization:

```jsx
const [user, setUser] =
  useState({
    name: "Vikash",
  });
```

Remove unnecessary synchronization.

---

# Part 11 — Event Handler Mistakes

## 23. Problem: Function Runs During Render

Bug:

```jsx
<button
  onClick={deleteUser(id)}
>
  Delete
</button>
```

This calls `deleteUser` while rendering.

Fix:

```jsx
<button
  onClick={() =>
    deleteUser(id)
  }
>
  Delete
</button>
```

Pass a function to the event prop.

---

## 24. Problem: preventDefault vs stopPropagation

```jsx
function handleSubmit(e) {
  e.preventDefault();
}
```

`preventDefault()` prevents the browser's default action.

```jsx
function handleClick(e) {
  e.stopPropagation();
}
```

`stopPropagation()` stops the event from continuing through propagation.

They solve different problems.

---

# Part 12 — Conditional Rendering Bugs

## 25. Problem: UI Displays 0

```jsx
{items.length && (
  <List items={items} />
)}
```

When length is `0`, React can render the numeric `0`.

Better:

```jsx
{items.length > 0 && (
  <List items={items} />
)}
```

Or use a clear conditional/ternary.

---

## 26. Problem: Hooks Inside Condition

Wrong:

```jsx
if (isLoggedIn) {
  const [user, setUser] =
    useState(null);
}
```

Hook order can change between renders.

Correct structure:

```jsx
const [user, setUser] =
  useState(null);

if (!isLoggedIn) {
  return <Login />;
}
```

Ordinary Hooks must preserve consistent call ordering.

---

# Part 13 — Refs

## 27. Problem: Using Ref for Visible State

```jsx
const countRef =
  useRef(0);

function increment() {
  countRef.current++;
}
```

If UI displays:

```jsx
<p>
  {countRef.current}
</p>
```

it will not automatically update when the ref changes.

Use state when changes should trigger rendering.

---

## 28. Problem: Reading DOM Ref During Initial Render

```jsx
const inputRef =
  useRef(null);

console.log(
  inputRef.current.value
);
```

During initial render, the DOM node does not exist yet.

Refs to DOM nodes are populated during commit.

Use them from appropriate event/effect timing.

---

# Part 14 — Memoization Bugs

## 29. Problem: React.memo Does Not Help ⭐⭐⭐⭐⭐

```jsx
const Child =
  memo(function Child({
    options,
  }) {
    return ...;
  });

function Parent() {
  const options = {
    sort: "asc",
  };

  return (
    <Child
      options={options}
    />
  );
}
```

Each Parent render creates a new object reference.

`memo` sees a changed prop.

Possible fix **if profiling shows this matters**:

```jsx
const options =
  useMemo(
    () => ({
      sort: "asc",
    }),
    []
  );
```

Or move a constant outside the component when it truly never depends on render inputs.

Do not memoize automatically.

---

## 30. Problem: useCallback With Missing Dependency

```jsx
const handleSave =
  useCallback(() => {
    saveUser(user);
  }, []);
```

The callback captures the initial `user`.

Correct:

```jsx
const handleSave =
  useCallback(() => {
    saveUser(user);
  }, [user]);
```

Memoization does not remove closure semantics.

---

## 31. Problem: Expensive Calculation Runs Every Render

```jsx
const result =
  expensiveCalculation(data);
```

First question:

```text
Is it actually expensive?
```

If profiling shows meaningful cost and inputs are stable enough:

```jsx
const result =
  useMemo(
    () =>
      expensiveCalculation(
        data
      ),
    [data]
  );
```

Performance optimization should follow measurement.

---

# Part 15 — Context

## 32. Problem: Context Value Recreated Every Render

```jsx
<AuthContext.Provider
  value={{
    user,
    logout,
  }}
>
  {children}
</AuthContext.Provider>
```

The object has a new identity every provider render.

This can matter for consumers.

Depending on architecture and profiling, you may stabilize the value:

```jsx
const value =
  useMemo(
    () => ({
      user,
      logout,
    }),
    [user, logout]
  );
```

But first consider:

- why provider renders
- whether consumers are expensive
- whether context should be split
- whether state should be colocated

Do not treat `useMemo` as the only architecture solution.

---

# Part 16 — Reducers

## 33. Problem: Reducer Mutates State

Wrong:

```jsx
function reducer(
  state,
  action
) {
  if (
    action.type === "add"
  ) {
    state.items.push(
      action.item
    );

    return state;
  }

  return state;
}
```

Better:

```jsx
function reducer(
  state,
  action
) {
  if (
    action.type === "add"
  ) {
    return {
      ...state,
      items: [
        ...state.items,
        action.item,
      ],
    };
  }

  return state;
}
```

Reducers should calculate the next state without mutating existing state.

---

# Part 17 — Performance Scenarios

## 34. Problem: Search Input Feels Slow ⭐⭐⭐⭐⭐

Suppose typing updates:

```text
query
+
huge expensive result list
```

Possible investigation:

```text
1. Profile.
2. Check expensive filtering/rendering.
3. Check unnecessary parent state.
4. Consider memoizing expensive pure calculation.
5. Consider useDeferredValue for expensive dependent UI.
6. Consider virtualization for huge lists.
7. Consider server-side/query architecture if dataset is large.
```

Example:

```jsx
const deferredQuery =
  useDeferredValue(query);

const results =
  useMemo(
    () =>
      filterUsers(
        users,
        deferredQuery
      ),
    [
      users,
      deferredQuery,
    ]
  );
```

`useDeferredValue` improves scheduling responsiveness; it does not make the expensive algorithm itself faster.

---

## 35. Problem: Expensive Tab Switch Blocks Urgent UI

A transition may help mark the expensive state update as non-urgent:

```jsx
const [
  isPending,
  startTransition,
] = useTransition();

function selectTab(tab) {
  startTransition(() => {
    setTab(tab);
  });
}
```

Use transitions for suitable non-urgent updates.

Do not wrap controlled text-input state itself in a transition when the input must update immediately.

---

# Part 18 — Lazy Loading

## 36. Problem: Huge Initial Bundle

Instead of importing every large route/widget eagerly:

```jsx
import Analytics
  from "./Analytics";
```

A suitable component may be lazy loaded:

```jsx
const Analytics =
  lazy(() =>
    import("./Analytics")
  );
```

Then:

```jsx
<Suspense
  fallback={
    <AnalyticsSkeleton />
  }
>
  <Analytics />
</Suspense>
```

Code splitting is useful when it meaningfully reduces initial work without damaging UX.

---

# Part 19 — Error Handling

## 37. Problem: One Widget Crashes Entire Screen

Use an appropriately scoped Error Boundary:

```text
Dashboard
├── Header
├── ErrorBoundary
│    └── AnalyticsWidget
└── ApplicationList
```

Now one failure can show local fallback instead of replacing unrelated UI.

Boundary placement is an architecture decision.

---

# Part 20 — React 19 Problems

## 38. Problem: Manually Managing Form Pending State Everywhere

Older/manual pattern:

```jsx
const [pending, setPending] =
  useState(false);

async function submit() {
  setPending(true);

  try {
    await save();
  } finally {
    setPending(false);
  }
}
```

In appropriate React 19 Action/form workflows, APIs such as:

- Actions
- `useActionState`
- `useFormStatus`

can integrate pending/result handling more naturally.

Do not mechanically rewrite every async button as a form Action. Use the model where it fits.

---

## 39. Problem: Optimistic UI Becomes Permanent After Failure

Conceptual bug:

```text
click Connect
→ immediately show Connected
→ server fails
→ UI remains Connected forever
```

Optimistic state must reconcile with authoritative outcome.

With React 19 `useOptimistic`, structure the optimistic state around the Action lifecycle and handle failure feedback appropriately.

Core principle:

```text
optimistic UI
≠ confirmed server state
```

---

# Part 21 — Accessibility Debugging

## 40. Problem: Clickable Card Cannot Be Used With Keyboard

```jsx
<div
  onClick={openProfile}
>
  Open profile
</div>
```

If it is an action:

```jsx
<button
  onClick={openProfile}
>
  Open profile
</button>
```

If it navigates:

```jsx
<a href={profileUrl}>
  Open profile
</a>
```

Use native semantics instead of rebuilding interaction behavior.

---

## 41. Problem: Icon Button Has No Accessible Name

```jsx
<button>
  <TrashIcon />
</button>
```

Better:

```jsx
<button
  aria-label={
    "Delete application"
  }
>
  <TrashIcon
    aria-hidden="true"
  />
</button>
```

---

# Part 22 — Security Debugging

## 42. Problem: Admin Security Only in React ⭐⭐⭐⭐⭐

```jsx
{user.role === "admin" && (
  <DeleteUserButton />
)}
```

Question:

> Is this secure?

No.

The attacker can bypass React and call the API directly.

Correct architecture:

```text
React role check
→ UX

Server authorization
→ security boundary
```

---

## 43. Problem: Trusting Client Price ⭐⭐⭐⭐⭐

Client:

```js
fetch("/checkout", {
  method: "POST",
  body: JSON.stringify({
    productId: 10,
    total: 1,
  }),
});
```

The server must not trust the supplied final total.

It should use authoritative product/pricing data and enforce business rules server-side.

---

# Part 23 — Testing Debugging

## 44. Problem: Fragile Test

```js
expect(
  wrapper.state("loading")
).toBe(false);
```

This is tightly coupled to implementation.

Prefer behavior:

```js
expect(
  screen.getByRole(
    "button",
    { name: /save/i }
  )
).toBeEnabled();
```

Test what the user can observe.

---

# Part 24 — Coding Challenges

## 45. Challenge 1 — Build a Search Filter ⭐⭐⭐⭐⭐

Requirements:

- controlled search input
- filter users by name
- case-insensitive
- show empty state
- do not use an Effect for derived results

One solution:

```jsx
function UserSearch({
  users,
}) {
  const [query, setQuery] =
    useState("");

  const normalizedQuery =
    query
      .trim()
      .toLowerCase();

  const filteredUsers =
    users.filter(user =>
      user.name
        .toLowerCase()
        .includes(
          normalizedQuery
        )
    );

  return (
    <>
      <label htmlFor="search">
        Search users
      </label>

      <input
        id="search"
        value={query}
        onChange={e =>
          setQuery(
            e.target.value
          )
        }
      />

      {filteredUsers.length
        === 0 ? (
        <p>No users found.</p>
      ) : (
        <ul>
          {filteredUsers.map(
            user => (
              <li key={user.id}>
                {user.name}
              </li>
            )
          )}
        </ul>
      )}
    </>
  );
}
```

Concepts tested:

- controlled input
- derived data
- keys
- empty state
- accessibility

---

## 46. Challenge 2 — Toggle Selected Item

Requirements:

```text
click item
→ select

click same item again
→ deselect
```

```jsx
function List({ items }) {
  const [
    selectedId,
    setSelectedId,
  ] = useState(null);

  function toggle(id) {
    setSelectedId(
      current =>
        current === id
          ? null
          : id
    );
  }

  return items.map(item => (
    <button
      key={item.id}
      onClick={() =>
        toggle(item.id)
      }
    >
      {item.name}

      {selectedId
        === item.id
        ? " ✓"
        : ""}
    </button>
  ));
}
```

Why store `selectedId` rather than the whole selected object?

Because identity can be derived from the authoritative `items` collection and avoids duplicated state.

---

## 47. Challenge 3 — Add and Remove Todo

```jsx
function TodoList() {
  const [todos, setTodos] =
    useState([]);

  function addTodo(text) {
    const todo = {
      id: crypto.randomUUID(),
      text,
    };

    setTodos(prev => [
      ...prev,
      todo,
    ]);
  }

  function removeTodo(id) {
    setTodos(prev =>
      prev.filter(
        todo =>
          todo.id !== id
      )
    );
  }

  ...
}
```

Concepts:

- immutable arrays
- functional updates
- stable keys

---

## 48. Challenge 4 — Debounced Search

Core design:

```jsx
function Search() {
  const [query, setQuery] =
    useState("");

  useEffect(() => {
    const id = setTimeout(
      () => {
        search(query);
      },
      400
    );

    return () =>
      clearTimeout(id);
  }, [query]);

  ...
}
```

Interview follow-up:

> Is this enough for asynchronous network requests?

Not necessarily.

Debouncing prevents some requests, but completed requests can still race. Consider cancellation or stale-result protection.

---

## 49. Challenge 5 — Previous Value

```jsx
function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}
```

Why does it return the previous value?

During render, `ref.current` still contains the value stored after the previous commit. The Effect updates it after the current commit.

---

# Part 25 — Debugging Strategy

## 50. React Debugging Decision Tree ⭐⭐⭐⭐⭐

```text
UI is wrong
   │
   ├── wrong displayed data?
   │      ↓
   │   check source of truth
   │   duplicated/derived state
   │
   ├── state not updating?
   │      ↓
   │   mutation?
   │   snapshot misunderstanding?
   │   same value?
   │
   ├── state resets?
   │      ↓
   │   key/type/position identity
   │
   ├── Effect wrong?
   │      ↓
   │   should Effect exist?
   │   dependencies?
   │   cleanup?
   │   stale closure?
   │
   ├── async result wrong?
   │      ↓
   │   race/cancellation/request identity
   │
   ├── slow?
   │      ↓
   │   profile first
   │   render scope?
   │   expensive calculation?
   │   huge list?
   │
   └── DOM interaction wrong?
          ↓
       ref timing?
       focus?
       accessibility?
```

---

# Part 26 — Interview Debugging Checklist

## 51. When Given React Code, Inspect in This Order ⭐⭐⭐⭐⭐

### 1. State

```text
Is state necessary?
Is it mutated?
Is it duplicated?
Does next state depend on previous state?
```

### 2. Identity

```text
Are keys stable?
Is component type changing?
Is state unexpectedly resetting?
```

### 3. Effects

```text
Does this need an Effect?
Are dependencies correct?
Is cleanup symmetrical?
```

### 4. Closures

```text
Which render created this callback?
Does it need snapshot or latest semantics?
```

### 5. Async

```text
Can requests resolve out of order?
Can work be cancelled?
Should stale results be ignored?
```

### 6. Rendering

```text
What triggered render?
Is expensive work repeated?
Does DOM actually change?
```

### 7. Performance

```text
Was the problem measured?
Would colocation help?
Does identity matter?
Is memoization justified?
```

### 8. Accessibility

```text
Is native semantic HTML used?
Can keyboard users operate it?
Does the control have a name?
```

### 9. Security

```text
Is client code being trusted for authorization,
validation, money, ownership, or secrets?
```

---

# Part 27 — Rapid Debugging Questions

## 52. Why Is My Component Rendering Again?

Check:

- state update
- parent render
- consumed Context update
- external store subscription
- development Strict Mode behavior

Then ask whether the render is actually expensive.

---

## 53. Why Did State Reset?

Check:

- changed key
- changed component type
- changed tree position/identity
- conditional rendering
- explicit remount

---

## 54. Why Is My Effect Looping?

Check whether:

```text
Effect
→ updates state
→ dependency changes
→ Effect
```

Also inspect unstable object/function dependencies.

Most importantly, ask whether the Effect is necessary.

---

## 55. Why Is My Effect Reading Old Data?

Likely inspect:

- missing dependency
- stale closure
- intentionally captured snapshot
- latest-value requirement

Do not automatically solve it with a ref.

---

## 56. Why Is memo Not Working?

Check:

- changing object props
- changing function props
- internal state
- Context
- whether memoization is useful at all

---

## 57. Why Is UI Slow?

```text
Measure first.
```

Then investigate:

- expensive render computation
- huge lists
- unnecessary high-level state
- excessive Effects/state chains
- repeated network/data work
- unnecessary remounting
- bundle size
- expensive third-party components

---

# Part 28 — Interview Traps

## 58. Trap: "Fix It With useEffect"

Not every synchronization-looking state problem needs an Effect.

Ask:

```text
Is there an external system?
```

If not, consider:

- derive during render
- event handler
- better state structure
- key-based reset

---

## 59. Trap: "Fix It With useMemo"

Memoization does not correct broken logic.

Wrong:

```text
buggy derived state
→ add useMemo everywhere
```

First make the code correct and simple.

Then optimize measured bottlenecks.

---

## 60. Trap: "Fix It With useRef"

Refs can hide reactivity bugs.

Do not move a reactive value into a ref just to remove an Effect dependency warning.

Use refs when non-rendering mutable/latest-value semantics are genuinely required.

---

## 61. Trap: "Disable Strict Mode"

Strict Mode often reveals fragile Effects or render impurities.

Fix the underlying code instead of disabling the diagnostic behavior.

---

# Part 29 — Practical Interview Exercise

## 62. Find Every Problem ⭐⭐⭐⭐⭐

```jsx
function Users({
  users,
  roomId,
}) {
  const [query, setQuery] =
    useState("");

  const [
    filtered,
    setFiltered,
  ] = useState([]);

  useEffect(() => {
    setFiltered(
      users.filter(user =>
        user.name.includes(
          query
        )
      )
    );
  }, [query]);

  useEffect(() => {
    const socket =
      connect(roomId);

    socket.on(
      "message",
      message => {
        console.log(message);
      }
    );
  }, []);

  return (
    <>
      <input
        value={query}
        onChange={e =>
          setQuery(
            e.target.value
          )
        }
      />

      {filtered.map(
        (user, index) => (
          <UserCard
            key={index}
            user={user}
          />
        )
      )}
    </>
  );
}
```

### Problems

1. `filtered` is derived state.
2. Filtering Effect is unnecessary.
3. Filtering Effect misses `users`.
4. Socket Effect misses `roomId`.
5. Socket has no cleanup.
6. Index key is unsafe if list changes.
7. Search input should have an accessible label/name.

### Better Version

```jsx
function Users({
  users,
  roomId,
}) {
  const [query, setQuery] =
    useState("");

  const filtered =
    users.filter(user =>
      user.name
        .toLowerCase()
        .includes(
          query.toLowerCase()
        )
    );

  useEffect(() => {
    const socket =
      connect(roomId);

    function handleMessage(
      message
    ) {
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

  return (
    <>
      <label htmlFor="search">
        Search users
      </label>

      <input
        id="search"
        value={query}
        onChange={e =>
          setQuery(
            e.target.value
          )
        }
      />

      {filtered.map(
        user => (
          <UserCard
            key={user.id}
            user={user}
          />
        )
      )}
    </>
  );
}
```

---

# Part 30 — Final Interview Practice

## 63. Explain Bugs, Don't Just Patch Them ⭐⭐⭐⭐⭐

Weak answer:

```text
Add roomId to dependencies.
```

Strong answer:

```text
The Effect synchronizes a connection with roomId.
Because roomId is reactive and is used by the Effect,
the synchronization must be re-established when roomId
changes. Otherwise the component remains connected to
the room captured by the old render.

I would include roomId as a dependency and make cleanup
disconnect the previous room before the Effect connects
to the new one.
```

That demonstrates React understanding rather than lint-rule memorization.

---

# Part 31 — Complete Debugging Mental Model

## 64. The React Bug Map ⭐⭐⭐⭐⭐

```text
                        REACT BUG
                            │
       ┌────────────────────┼────────────────────┐
       ↓                    ↓                    ↓
      STATE               EFFECTS             IDENTITY
       │                    │                    │
 snapshot?             necessary?             key?
 mutation?             dependency?            type?
 derived?              cleanup?               position?
 updater?              stale closure?         remount?
       │                    │                    │
       └──────────────┬─────┴─────┬──────────────┘
                      ↓           ↓
                    ASYNC      PERFORMANCE
                      │           │
                    race?      measure?
                    abort?     expensive?
                    stale?     memo useful?
                      │           │
                      └─────┬─────┘
                            ↓
                       PRODUCTION
                   accessibility
                   security
                   testing


Interview process:

SYMPTOM
   ↓
MENTAL MODEL
   ↓
ROOT CAUSE
   ↓
SMALLEST CORRECT FIX
   ↓
TRADEOFF
```

---

## 65. Key Takeaways

- Debug React from its mental model, not by randomly adding Hooks.
- State values belong to render snapshots.
- Use functional updates when next state depends on previous queued state.
- Never mutate state objects or arrays.
- Stable keys preserve logical identity.
- Unstable keys can reset or misassociate state.
- Do not define reusable component types inside another component when that changes identity across renders.
- Derived values usually do not need separate state or Effects.
- Keep controlled inputs consistently controlled.
- Effect dependencies must reflect reactive values used by synchronization.
- Move Effect-only objects/functions inside the Effect when appropriate.
- Cleanup should mirror setup.
- Understand stale closures separately from async races.
- Protect UI from out-of-order request results.
- Investigate infinite Effects by questioning the Effect itself first.
- Event props receive functions; do not accidentally invoke handlers during render.
- Hooks require stable call order.
- Ref mutations do not trigger rendering.
- DOM refs are populated during commit, not initial render.
- Memoization does not remove closure/dependency rules.
- Profile before optimizing.
- Transitions/deferred values change scheduling, not algorithmic complexity.
- Error Boundaries should be scoped around meaningful failure regions.
- Optimistic UI must reconcile with authoritative results.
- Native semantic controls prevent many accessibility bugs.
- Client-side UI checks never replace server security.
- Tests should assert observable behavior.
- In interviews, explain the root cause before giving the patch.

---

## Next Lesson

➡️ [Lesson 75 — React Architecture and Scenario-Based Questions ⭐⭐⭐⭐⭐](./75-architecture-scenarios.md)
