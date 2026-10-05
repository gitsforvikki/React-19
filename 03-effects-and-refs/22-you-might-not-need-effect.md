# Lesson 22 — You Might Not Need an Effect ⭐⭐⭐⭐⭐

## 1. Why This Lesson Is Extremely Important

A common React mistake is using `useEffect` whenever one value needs to change because another value changed.

For example:

```jsx
useEffect(() => {
  setFullName(
    firstName + " " + lastName
  );
}, [firstName, lastName]);
```

This works, but it is usually unnecessary.

The most important principle is:

> **Effects are an escape hatch for synchronizing React with external systems.**

If there is no external system involved, first ask whether the logic can happen:

- during rendering,
- inside an event handler,
- through better state structure,
- through a `key`,
- or through another React mechanism.

Reducing unnecessary Effects usually makes React code:

- simpler,
- easier to understand,
- less error-prone,
- faster,
- easier to debug.

---

# 2. The Core Decision Rule ⭐⭐⭐⭐⭐

Before writing:

```jsx
useEffect(() => {
  // ...
}, [dependencies]);
```

ask:

```text
Why does this code need to run?
```

Then use this decision tree:

```text
Need to calculate something
from props/state?
        ↓
Do it during render

User performed an action?
        ↓
Use an event handler

Need to reset an entire component
for a new logical entity?
        ↓
Consider changing its key

Need to synchronize with
an external system?
        ↓
Use an Effect
```

This mental model prevents a large class of React bugs.

---

# 3. Effects Are an Escape Hatch ⭐⭐⭐⭐⭐

React already handles:

```text
props
state
rendering
events
component identity
```

You normally do not need an Effect to make React values synchronize with other React values.

Effects become useful when React must coordinate with something outside its normal declarative model.

Examples:

```text
React
  ↕
WebSocket

React
  ↕
browser event listener

React
  ↕
third-party map

React
  ↕
timer

React
  ↕
external subscription
```

---

# 4. Do Not Use an Effect for Derived Values ⭐⭐⭐⭐⭐

Suppose:

```jsx
function User({
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

  return <h1>{fullName}</h1>;
}
```

Problems:

```text
props change
   ↓
render with old fullName
   ↓
commit
   ↓
Effect
   ↓
setFullName
   ↓
second render
```

But `fullName` can be calculated immediately.

Better:

```jsx
function User({
  firstName,
  lastName,
}) {
  const fullName =
    `${firstName} ${lastName}`;

  return <h1>{fullName}</h1>;
}
```

Now:

```text
props
 ↓
render
 ↓
derived value
 ↓
UI
```

One render path. No synchronization problem.

---

# 5. Derived State Is Often Redundant State ⭐⭐⭐⭐⭐

Bad:

```jsx
const [products, setProducts] =
  useState([]);

const [filteredProducts, setFilteredProducts] =
  useState([]);

useEffect(() => {
  setFilteredProducts(
    products.filter(
      (product) =>
        product.name.includes(query)
    )
  );
}, [products, query]);
```

If `filteredProducts` is completely determined by:

```text
products + query
```

it usually does not need separate state.

Better:

```jsx
const filteredProducts =
  products.filter(
    (product) =>
      product.name.includes(query)
  );
```

Rule:

> If a value can be calculated from existing props/state during rendering, usually calculate it instead of storing and synchronizing another state variable.

---

# 6. Why Redundant State Is Dangerous

Suppose you store:

```text
products
query
filteredProducts
```

Now React must keep three pieces of information consistent.

But:

```text
filteredProducts
=
f(products, query)
```

So storing it introduces another source of truth.

Possible bug:

```text
products changed
      ↓
filteredProducts not synchronized correctly
      ↓
UI becomes inconsistent
```

Prefer minimal state.

---

# 7. Do Not Use an Effect for Simple Transformations

Bad:

```jsx
const [upperName, setUpperName] =
  useState("");

useEffect(() => {
  setUpperName(
    name.toUpperCase()
  );
}, [name]);
```

Better:

```jsx
const upperName =
  name.toUpperCase();
```

Other examples:

```jsx
const total =
  items.reduce(
    (sum, item) =>
      sum + item.price,
    0
  );

const completedCount =
  todos.filter(
    (todo) => todo.completed
  ).length;

const isValid =
  email.includes("@");
```

These are render calculations, not Effects.

---

# 8. Expensive Calculations Still Do Not Automatically Need Effects ⭐⭐⭐⭐⭐

Suppose filtering is expensive.

Do not convert it into Effect-driven state:

```jsx
useEffect(() => {
  setVisibleItems(
    expensiveFilter(items, query)
  );
}, [items, query]);
```

If necessary, memoize the calculation:

```jsx
const visibleItems =
  useMemo(
    () =>
      expensiveFilter(
        items,
        query
      ),
    [items, query]
  );
```

Important:

```text
useMemo
=
performance optimization

useEffect + setState
=
additional synchronization/render cycle
```

Do not use `useMemo` for every calculation either. Measure or have a concrete reason.

---

# 9. Do Not Use an Effect for User Events ⭐⭐⭐⭐⭐

Suppose the user clicks Buy.

Bad:

```jsx
const [shouldBuy, setShouldBuy] =
  useState(false);

useEffect(() => {
  if (shouldBuy) {
    buyProduct();
  }
}, [shouldBuy]);

function handleBuy() {
  setShouldBuy(true);
}
```

The real reason the action happened is:

```text
user clicked Buy
```

So put the logic in the event handler:

```jsx
function handleBuy() {
  buyProduct();
}
```

---

# 10. Event Logic vs Effect Logic ⭐⭐⭐⭐⭐

Ask:

> Why should this code run?

If the answer is:

> Because the user clicked Submit.

Use:

```text
event handler
```

If the answer is:

> Because this component is currently displayed and must remain connected to room 42.

Use:

```text
Effect
```

Diagram:

```text
Specific interaction
      ↓
event handler

Component presence/state
requires external synchronization
      ↓
Effect
```

---

# 11. Example: Form Submission

Bad:

```jsx
const [submitted, setSubmitted] =
  useState(false);

useEffect(() => {
  if (submitted) {
    sendForm();
  }
}, [submitted]);

function handleSubmit(event) {
  event.preventDefault();
  setSubmitted(true);
}
```

Better:

```jsx
function handleSubmit(event) {
  event.preventDefault();
  sendForm();
}
```

The event handler knows exactly what happened.

---

# 12. Effect for Synchronization, Event for Action ⭐⭐⭐⭐⭐

Consider analytics.

### User clicked a purchase button

```jsx
function handlePurchase() {
  purchaseProduct();
  logPurchase();
}
```

This belongs to the interaction.

### Page is currently displayed

If an external analytics system must be synchronized with page presence, an Effect may be appropriate:

```jsx
useEffect(() => {
  logPageView(pageId);
}, [pageId]);
```

The distinction is causal.

---

# 13. Do Not Chain Effects to Transform State ⭐⭐⭐⭐⭐

Bad design:

```jsx
const [score, setScore] =
  useState(0);

const [isWinner, setIsWinner] =
  useState(false);

const [message, setMessage] =
  useState("");

useEffect(() => {
  if (score >= 10) {
    setIsWinner(true);
  }
}, [score]);

useEffect(() => {
  if (isWinner) {
    setMessage("You won!");
  }
}, [isWinner]);
```

Flow:

```text
score update
   ↓
render
   ↓
Effect
   ↓
isWinner update
   ↓
render
   ↓
Effect
   ↓
message update
   ↓
render
```

This is unnecessarily complicated.

---

# 14. Calculate What You Can During Render

Better:

```jsx
const isWinner =
  score >= 10;

const message =
  isWinner
    ? "You won!"
    : "Keep playing";
```

Now:

```text
score
  ↓
render
  ↓
isWinner
  ↓
message
  ↓
UI
```

No Effect chain.

---

# 15. Update Related State in the Same Event ⭐⭐⭐⭐⭐

Suppose an event logically changes multiple pieces of state.

Instead of:

```jsx
setCount(count + 1);

useEffect(() => {
  if (count >= 10) {
    setGameOver(true);
  }
}, [count]);
```

you may be able to calculate the next state inside the event:

```jsx
function handleIncrement() {
  setCount((count) => {
    const nextCount =
      count + 1;

    return nextCount;
  });
}
```

Often the best design is even simpler:

```jsx
const gameOver =
  count >= 10;
```

Store only the minimal source state.

---

# 16. Avoid Effect Chains ⭐⭐⭐⭐⭐

A chain like:

```text
state A
  ↓
Effect
  ↓
state B
  ↓
Effect
  ↓
state C
```

is a warning sign.

It can cause:

- extra renders
- hard-to-follow data flow
- transient inconsistent state
- dependency complexity
- difficult debugging

Prefer:

```text
source state
    ↓
render calculations
    ↓
UI
```

or update related state together in the event that caused the change.

---

# 17. Resetting All State for a Different Entity ⭐⭐⭐⭐⭐

Suppose:

```jsx
function ProfilePage({
  userId,
}) {
  const [comment, setComment] =
    useState("");

  // ...
}
```

When `userId` changes, you want all local profile state to reset.

A common approach is:

```jsx
useEffect(() => {
  setComment("");
}, [userId]);
```

But this means:

```text
render new user
with old comment
      ↓
commit
      ↓
Effect
      ↓
clear comment
      ↓
second render
```

---

# 18. Better Reset with Component Identity ⭐⭐⭐⭐⭐

Use a key:

```jsx
function ProfilePage({
  userId,
}) {
  return (
    <Profile
      key={userId}
      userId={userId}
    />
  );
}

function Profile({
  userId,
}) {
  const [comment, setComment] =
    useState("");

  // ...
}
```

When `userId` changes:

```text
Profile key=A
      ↓
Profile key=B
      ↓
new component identity
      ↓
local state initialized fresh
```

This connects directly to Lesson 18.

---

# 19. Resetting an Entire Subtree with key

A key can reset:

- form state
- local selections
- local drafts
- nested child state

when the whole subtree represents a new logical entity.

Example:

```jsx
<EditUserForm
  key={user.id}
  user={user}
/>
```

Use this intentionally.

Do not use random keys.

---

# 20. Adjusting Some State When Props Change

This is more subtle.

Suppose:

```jsx
function List({
  items,
}) {
  const [selection, setSelection] =
    useState(null);

  useEffect(() => {
    setSelection(null);
  }, [items]);

  // ...
}
```

This causes:

```text
render with old selection
       ↓
commit
       ↓
Effect
       ↓
reset
       ↓
render again
```

First ask whether selection should instead be represented in a way that naturally becomes invalid when items change.

---

# 21. Prefer Storing an ID Over a Whole Derived Object ⭐⭐⭐⭐⭐

Instead of:

```jsx
const [selectedItem, setSelectedItem] =
  useState(null);
```

when `selectedItem` is derived from `items`, consider:

```jsx
const [selectedId, setSelectedId] =
  useState(null);

const selectedItem =
  items.find(
    (item) =>
      item.id === selectedId
  ) ?? null;
```

Now if the selected item disappears:

```text
selectedId
   ↓
find in current items
   ↓
not found
   ↓
selectedItem = null
```

No synchronization Effect is needed.

This is a powerful state-design pattern.

---

# 22. State Structure Can Remove Effects ⭐⭐⭐⭐⭐

Many unnecessary Effects are actually state-design problems.

Bad model:

```text
items
selectedItem copy
filteredItems copy
itemCount copy
```

Better:

```text
STATE:
items
selectedId
query

DERIVED:
selectedItem
filteredItems
itemCount
```

Minimal source state reduces synchronization requirements.

---

# 23. Notify Parent in the Event Handler ⭐⭐⭐⭐⭐

Suppose:

```jsx
function Toggle({
  onChange,
}) {
  const [isOn, setIsOn] =
    useState(false);

  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange]);

  function handleClick() {
    setIsOn(
      (isOn) => !isOn
    );
  }

  // ...
}
```

If the notification is caused by the click, you can often notify at the same time:

```jsx
function handleClick() {
  setIsOn((isOn) => {
    const nextIsOn =
      !isOn;

    onChange(nextIsOn);

    return nextIsOn;
  });
}
```

However, updater functions should remain pure, so calling external callbacks from an updater is not ideal.

A cleaner controlled design is often:

```jsx
function Toggle({
  isOn,
  onChange,
}) {
  function handleClick() {
    onChange(!isOn);
  }

  return (
    <button onClick={handleClick}>
      {isOn ? "ON" : "OFF"}
    </button>
  );
}
```

This is another example where better ownership can eliminate an Effect.

---

# 24. Important: Keep State Updaters Pure ⭐⭐⭐⭐⭐

Do not put side effects inside updater functions:

```jsx
setCount((count) => {
  sendAnalytics(); // avoid
  return count + 1;
});
```

React may call updater functions more than once in development to verify purity.

Calculate state in the updater.

Perform event-caused external actions in the event handler.

---

# 25. Passing Data Up Does Not Automatically Require an Effect

If a child needs to tell its parent about a user interaction:

```jsx
function SearchInput({
  value,
  onChange,
}) {
  return (
    <input
      value={value}
      onChange={(event) =>
        onChange(
          event.target.value
        )
      }
    />
  );
}
```

No Effect.

Flow:

```text
user types
   ↓
event handler
   ↓
callback prop
   ↓
parent updates state
```

This is normal React data flow.

---

# 26. Do Not Use Effects to Synchronize Duplicate React State ⭐⭐⭐⭐⭐

Bad:

```jsx
function Child({
  value,
}) {
  const [localValue, setLocalValue] =
    useState(value);

  useEffect(() => {
    setLocalValue(value);
  }, [value]);
}
```

Ask why two copies exist.

If the child should display the parent value:

```jsx
function Child({
  value,
}) {
  return <p>{value}</p>;
}
```

If the child needs an independent editable draft, model that intentionally.

Do not create duplicate state casually and then use Effects to keep it synchronized.

---

# 27. Initial State Is Different from Synchronized State

This can be valid:

```jsx
function Form({
  initialName,
}) {
  const [name, setName] =
    useState(initialName);

  // ...
}
```

The prop name communicates:

```text
initialName
      ↓
used to initialize local state
      ↓
local state becomes independent
```

If changing the logical form entity should reset it, consider component identity with a key.

---

# 28. Fetching Data Is a Genuine External Synchronization Case

Fetching data communicates with something outside React.

So an Effect can be appropriate in a client-side React application:

```jsx
useEffect(() => {
  const controller =
    new AbortController();

  async function load() {
    // fetch...
  }

  load();

  return () => {
    controller.abort();
  };
}, [query]);
```

However:

> Just because data fetching can use an Effect does not mean hand-written Effect fetching is always the best architecture.

Frameworks and data libraries may provide:

- caching
- request deduplication
- preloading
- server fetching
- race handling
- loading/error management

Data fetching is covered later.

---

# 29. Effects for External Subscriptions ⭐⭐⭐⭐⭐

This is a legitimate Effect:

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

Why?

Because React is synchronizing with an external browser subscription.

---

# 30. But React Has a Dedicated External Store Hook

For subscribing to external stores, React provides:

```text
useSyncExternalStore
```

Conceptually:

```jsx
const value =
  useSyncExternalStore(
    subscribe,
    getSnapshot,
    getServerSnapshot
  );
```

This is designed for external stores whose values can change outside React.

Examples:

- browser-backed state
- external state stores
- third-party store integrations

You do not need to master it here, but know that not every external subscription needs to be manually modeled with `useEffect + useState`.

---

# 31. Example: Browser Online Status with useSyncExternalStore

Conceptually:

```jsx
function subscribe(callback) {
  window.addEventListener(
    "online",
    callback
  );

  window.addEventListener(
    "offline",
    callback
  );

  return () => {
    window.removeEventListener(
      "online",
      callback
    );

    window.removeEventListener(
      "offline",
      callback
    );
  };
}

function getSnapshot() {
  return navigator.onLine;
}

function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    getSnapshot
  );
}
```

This expresses:

```text
external store
     ↓
React subscription API
     ↓
component snapshot
```

rather than manually synchronizing mirrored state.

---

# 32. Do Not Use Effects for Initialization That Can Happen Earlier

Suppose:

```jsx
useEffect(() => {
  initializeApp();
}, []);
```

Ask:

> Does this initialization truly depend on this component being mounted?

If it is global one-time application initialization unrelated to a component's presence, module initialization or an application bootstrap location may be more appropriate.

Do not attach global behavior to an arbitrary component merely to obtain an Effect.

---

# 33. Module-Level Initialization

For code that should run when a module/application environment initializes and is safe to do there:

```js
initializeSomething();

function App() {
  return <Main />;
}
```

But be careful with:

- server rendering
- browser-only APIs
- tests
- module evaluation environments

The point is not "always move it outside."

The point is:

> Match the lifecycle of the code to the lifecycle it actually needs.

---

# 34. Do Not Use Effects to Change State Just Because Props Changed ⭐⭐⭐⭐⭐

This pattern deserves suspicion:

```jsx
useEffect(() => {
  setSomething(
    calculateSomething(prop)
  );
}, [prop]);
```

Ask:

### Can it be derived?

```jsx
const something =
  calculateSomething(prop);
```

### Should the whole component reset?

```jsx
<Component key={id} />
```

### Can state ownership be changed?

Lift or colocate state appropriately.

Only keep the Effect when there is genuine external synchronization.

---

# 35. Real-World Example: CareerLoop Application Counts

Imagine:

```jsx
const [applications, setApplications] =
  useState([]);

const [appliedCount, setAppliedCount] =
  useState(0);

useEffect(() => {
  setAppliedCount(
    applications.filter(
      (application) =>
        application.status ===
        "applied"
    ).length
  );
}, [applications]);
```

This is redundant.

Better:

```jsx
const appliedCount =
  applications.filter(
    (application) =>
      application.status ===
      "applied"
  ).length;
```

Source of truth:

```text
applications
```

Derived value:

```text
appliedCount
```

No Effect.

---

# 36. Real-World Example: CodeBuddy Search

Bad:

```jsx
const [developers, setDevelopers] =
  useState([]);

const [query, setQuery] =
  useState("");

const [filtered, setFiltered] =
  useState([]);

useEffect(() => {
  setFiltered(
    developers.filter(
      (developer) =>
        developer.name
          .toLowerCase()
          .includes(
            query.toLowerCase()
          )
    )
  );
}, [developers, query]);
```

Better:

```jsx
const filtered =
  developers.filter(
    (developer) =>
      developer.name
        .toLowerCase()
        .includes(
          query.toLowerCase()
        )
  );
```

If this becomes measurably expensive:

```jsx
const filtered =
  useMemo(
    () =>
      developers.filter(
        (developer) =>
          developer.name
            .toLowerCase()
            .includes(
              query.toLowerCase()
            )
      ),
    [developers, query]
  );
```

Still no synchronization Effect.

---

# 37. Real-World Example: Chat Connection — Effect Required ⭐⭐⭐⭐⭐

```jsx
useEffect(() => {
  const socket =
    connectToConversation(
      conversationId
    );

  return () => {
    socket.disconnect();
  };
}, [conversationId]);
```

Why is this different?

Because:

```text
React component
      ↓
external socket system
```

There is genuine synchronization.

This is exactly what Effects are for.

---

# 38. A Powerful Question Before Every Effect ⭐⭐⭐⭐⭐

Ask:

> **What external system is this Effect synchronizing with?**

If you cannot name one, investigate whether the Effect can be removed.

Possible answer:

```text
"No external system.
I am only calculating state from other state."
```

That is a strong sign the Effect is unnecessary.

This is a heuristic, not a claim that every possible Effect without an obvious external API is invalid. The goal is to make Effects intentional rather than automatic.

---

# 39. Effect Removal Decision Table ⭐⭐⭐⭐⭐

| Requirement | Preferred approach |
|---|---|
| Calculate value from props/state | Calculate during render |
| Expensive pure calculation | Render; use `useMemo` only if justified |
| Respond to click/submit/type | Event handler |
| Share state between siblings | Lift state |
| Keep state near one component | Colocate state |
| Reset entire subtree for new entity | Change meaningful `key` |
| Avoid duplicate React state | Derive from source state |
| Notify parent about interaction | Callback/event handler |
| Subscribe to external store | Consider `useSyncExternalStore` |
| Connect to socket/service | Effect |
| Add browser listener | Effect or dedicated subscription abstraction |
| Manage timer tied to component | Effect |
| Synchronize third-party widget | Effect |
| Fetch client-side external data | Effect can work; consider data/framework abstractions |

---

# 40. Common Mistakes ⭐⭐⭐⭐⭐

### Mistake 1 — Storing derived values in state

Calculate them during rendering.

### Mistake 2 — Using an Effect for every prop change

A prop change does not automatically require synchronization.

### Mistake 3 — Using Effects for button actions

Put interaction-specific logic in the event handler.

### Mistake 4 — Chaining Effects

Prefer minimal source state and render-time derivation.

### Mistake 5 — Resetting an entire component with an Effect

A meaningful `key` may model identity better.

### Mistake 6 — Duplicating parent state in child state

Use props directly unless the local value has intentionally different meaning.

### Mistake 7 — Using an Effect for expensive calculations

Memoization, if actually necessary, is different from synchronization.

### Mistake 8 — Fighting dependency warnings

The Effect may be unnecessary or poorly structured.

### Mistake 9 — Using Effects as a generic lifecycle callback

Think in terms of synchronization processes.

### Mistake 10 — Assuming removing all Effects is the goal

Effects are correct and necessary when external synchronization exists.

---

# 41. Interview Questions ⭐⭐⭐⭐⭐

## Q1. What does "You Might Not Need an Effect" mean?

**Answer:** Many operations developers put in Effects—such as derived calculations, event-driven actions, and React-state synchronization—can be expressed more directly during render, through events, state ownership, or component identity.

---

## Q2. What is the primary purpose of useEffect?

**Answer:** To synchronize a React component with an external system.

---

## Q3. Should derived state be calculated with useEffect?

**Answer:** Usually no. If a value can be calculated from existing props/state during render, calculate it directly.

---

## Q4. Why is derived state in an Effect inefficient?

**Answer:** React first renders with the old derived state, commits, runs the Effect, updates state, and renders again.

---

## Q5. Where should logic caused by a button click live?

**Answer:** Usually in the button's event handler because the interaction is the reason the logic should run.

---

## Q6. How can you reset all local state when an entity changes?

**Answer:** Often by giving the subtree a meaningful key based on the entity identity.

---

## Q7. When should useMemo replace an Effect?

**Answer:** Not as a general replacement. If a pure render calculation is genuinely expensive, `useMemo` may cache it as a performance optimization instead of storing its result via an Effect.

---

## Q8. Why are chains of Effects problematic?

**Answer:** They cause extra renders, make data flow harder to follow, and create unnecessary synchronization between pieces of React state.

---

## Q9. Should child state mirror a prop using an Effect?

**Answer:** Usually no. Use the prop directly unless the child intentionally owns independent state, such as an editable draft.

---

## Q10. What is a useful question before writing an Effect?

**Answer:** "What external system am I synchronizing with?"

---

## Q11. Is data fetching a valid Effect use case?

**Answer:** Yes, client-side fetching is external synchronization, although frameworks or data libraries may provide better fetching, caching, and race-management abstractions.

---

## Q12. What Hook is designed for subscribing to external stores?

**Answer:** `useSyncExternalStore`.

---

# 42. Interview Scenario 1 ⭐⭐⭐⭐⭐

What is wrong with this?

```jsx
function Cart({
  items,
}) {
  const [total, setTotal] =
    useState(0);

  useEffect(() => {
    setTotal(
      items.reduce(
        (sum, item) =>
          sum + item.price,
        0
      )
    );
  }, [items]);

  return <p>{total}</p>;
}
```

Answer:

`total` is derived entirely from `items`.

Better:

```jsx
function Cart({
  items,
}) {
  const total =
    items.reduce(
      (sum, item) =>
        sum + item.price,
      0
    );

  return <p>{total}</p>;
}
```

---

# 43. Interview Scenario 2 ⭐⭐⭐⭐⭐

What is wrong with:

```jsx
useEffect(() => {
  if (clicked) {
    sendAnalytics();
  }
}, [clicked]);
```

if `clicked` becomes true only because a user pressed a button?

The analytics action is caused by the interaction.

Prefer:

```jsx
function handleClick() {
  sendAnalytics();
  // perform click behavior
}
```

---

# 44. Interview Scenario 3 ⭐⭐⭐⭐⭐

Should this Effect be removed?

```jsx
useEffect(() => {
  const socket =
    connect(roomId);

  return () => {
    socket.disconnect();
  };
}, [roomId]);
```

No.

This is genuine external synchronization:

```text
React
 ↕
socket connection
```

The Effect is appropriate.

---

# 45. Interview Scenario 4 ⭐⭐⭐⭐⭐

You have:

```jsx
const [products, setProducts] =
  useState([]);

const [query, setQuery] =
  useState("");

const [count, setCount] =
  useState(0);

useEffect(() => {
  setCount(
    products.filter(
      (product) =>
        product.name.includes(query)
    ).length
  );
}, [products, query]);
```

Better:

```jsx
const count =
  products.filter(
    (product) =>
      product.name.includes(query)
  ).length;
```

Reason:

```text
count
=
derived from products + query
```

No separate source of truth is needed.

---

# 46. Before and After Mental Model ⭐⭐⭐⭐⭐

Beginner approach:

```text
Something changed
      ↓
useEffect
      ↓
set another state
```

Better React approach:

```text
Something changed
      ↓
Can I derive the UI
during render?
      │
     yes
      ↓
calculate directly
```

Or:

```text
Did user cause it?
      │
     yes
      ↓
event handler
```

Or:

```text
Is this a new logical entity?
      │
     yes
      ↓
consider key / state ownership
```

Only then:

```text
External system involved?
      │
     yes
      ↓
Effect
```

---

# 47. Effect Elimination Checklist ⭐⭐⭐⭐⭐

Before adding an Effect, ask:

```text
1. Can I calculate this during render?

2. Am I storing data that can be
   derived from existing state/props?

3. Is this caused by a user event?

4. Can I perform the operation
   directly in that event handler?

5. Am I duplicating state?

6. Is state owned by the wrong component?

7. Should the component reset because
   its logical identity changed?

8. Can a meaningful key solve that reset?

9. Is this just an expensive calculation?
   If so, is memoization actually needed?

10. Is there an external system that
    genuinely requires synchronization?
```

If the answer to #10 is yes, an Effect may be appropriate.

---

# 48. Complete Mental Model ⭐⭐⭐⭐⭐

```text
                Need some logic
                      │
                      ↓
       Can it be calculated from
          props/state during render?
              │             │
             yes            no
              │             │
              ↓             ↓
          calculate      Was it caused
           directly      by user action?
                           │       │
                          yes      no
                           │       │
                           ↓       ↓
                        event   Is this about
                       handler  state ownership,
                                identity or reset?
                                  │       │
                                 yes      no
                                  │       │
                                  ↓       ↓
                              redesign   External
                              state/key  system?
                                          │
                                     ┌────┴────┐
                                    yes        no
                                     │          │
                                     ↓          ↓
                                   Effect   reconsider
                                            design
```

---

# 49. Quick Revision ⭐⭐⭐⭐⭐

### Calculate during render

```text
fullName
filtered list
total
count
validation
selected object from selectedId
```

### Use event handlers

```text
submit
purchase
delete
send message
user-triggered analytics
```

### Use component identity / state design

```text
reset form for another user
reset chat draft
avoid duplicate state
share sibling state
```

### Use Effects

```text
WebSocket
browser listener
timer
subscription
third-party widget
external system synchronization
client-side fetch when appropriate
```

---

# 50. Key Takeaways

- Effects are an escape hatch for synchronization with systems outside React.
- Do not use an Effect simply because one React value depends on another.
- Derived values should usually be calculated during render.
- Avoid storing redundant derived state.
- Derived-state Effects create unnecessary extra render cycles.
- Expensive pure calculations may use `useMemo` when optimization is justified; they do not require Effect-driven state.
- Logic caused by a specific user interaction usually belongs in an event handler.
- Avoid chains of Effects that transform React state into more React state.
- Keep source state minimal and derive the rest.
- Better state structure can eliminate many Effects.
- Store stable identifiers when possible and derive related objects from current data.
- Do not mirror props into state unless the local state intentionally has different semantics.
- A meaningful `key` can reset an entire subtree for a different logical entity.
- State ownership and controlled component design can remove synchronization Effects.
- Keep updater functions pure.
- Client-side fetching can be an Effect use case, but dedicated data/framework abstractions may be better.
- `useSyncExternalStore` exists for external-store subscriptions.
- Do not try to remove Effects that genuinely synchronize with sockets, browser APIs, timers, subscriptions, or third-party systems.
- Before writing an Effect, ask: **What external system am I synchronizing with?**

---

## Next Lesson

➡️ [Lesson 23 — Closures, Stale Closures and Effects ⭐⭐⭐⭐⭐](./23-stale-closures.md)
