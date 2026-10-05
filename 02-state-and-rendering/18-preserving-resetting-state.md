# Lesson 18 — Preserving and Resetting State; Component Identity ⭐⭐⭐⭐⭐

## 1. Why This Topic Matters

A React component's state does not belong to the JSX tag itself.

React associates state with a component's **position and identity in the rendered tree**.

This explains questions such as:

- Why does state sometimes survive a re-render?
- Why does state reset when a component disappears?
- Why can changing a `key` reset a form?
- Why does moving a component change its state?
- Why can defining components inside other components unexpectedly reset state?

This is one of the most important React mental models.

---

## 2. State Is Tied to a Position in the UI Tree ⭐⭐⭐⭐⭐

Consider:

```jsx
function App() {
  return (
    <div>
      <Counter />
    </div>
  );
}
```

React conceptually sees:

```text
App
└── div
    └── Counter
```

The `Counter` state is associated with that position in the tree.

A useful mental model:

```text
state identity
=
component type
+
tree position
+
key when relevant
```

---

## 3. Re-rendering Does Not Automatically Reset State ⭐⭐⭐⭐⭐

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button
      onClick={() =>
        setCount((count) => count + 1)
      }
    >
      {count}
    </button>
  );
}
```

When state changes:

```text
Render #1
Counter at position X
count = 0

Render #2
Counter at position X
count = 1
```

React recognizes:

```text
same component type
+
same position
```

so state is preserved.

---

## 4. State Belongs to React, Not the JSX Object ⭐⭐⭐⭐⭐

Each render creates new JSX descriptions.

Yet state can survive because React matches the new tree with the previous tree.

```text
Previous tree
Counter at position X
state = 5
        ↓
reconciliation
        ↓
Next tree
Counter at position X
        ↓
same identity
        ↓
preserve state = 5
```

This connects directly to reconciliation.

---

## 5. Same Component at Same Position Preserves State ⭐⭐⭐⭐⭐

Consider:

```jsx
function App() {
  const [dark, setDark] =
    useState(false);

  return (
    <div
      className={
        dark ? "dark" : "light"
      }
    >
      <Counter />

      <button
        onClick={() =>
          setDark((dark) => !dark)
        }
      >
        Toggle Theme
      </button>
    </div>
  );
}
```

Changing `dark` re-renders `App`.

But `Counter` remains:

```text
same type
same tree position
```

so its state is preserved.

---

## 6. JSX Branches Do Not Define Identity by Themselves ⭐⭐⭐⭐⭐

Consider:

```jsx
function App({
  isFancy,
}) {
  if (isFancy) {
    return (
      <Counter className="fancy" />
    );
  }

  return (
    <Counter className="plain" />
  );
}
```

Although the JSX appears in two code branches, React can still see:

```text
Counter
at the same tree position
```

Therefore state can be preserved.

The important thing is the resulting tree, not the physical line where JSX was written.

---

## 7. Different Component Type Resets State ⭐⭐⭐⭐⭐

Suppose:

```jsx
function App({
  loggedIn,
}) {
  return (
    <div>
      {loggedIn
        ? <Dashboard />
        : <LoginForm />}
    </div>
  );
}
```

At the same position React sees:

```text
Before:
Dashboard

After:
LoginForm
```

The component type changed.

React removes the old subtree and creates a new one.

Its state is reset.

---

## 8. Type Change Diagram ⭐⭐⭐⭐⭐

```text
Previous tree

App
└── Dashboard
    └── state A

       ↓ condition changes

Next tree

App
└── LoginForm
    └── new state

Dashboard state is discarded.
```

---

## 9. Removing a Component Resets Its State ⭐⭐⭐⭐⭐

```jsx
function App() {
  const [show, setShow] =
    useState(true);

  return (
    <>
      {show && <Counter />}

      <button
        onClick={() =>
          setShow((show) => !show)
        }
      >
        Toggle
      </button>
    </>
  );
}
```

Flow:

```text
show = true
Counter exists
count = 5

show = false
Counter removed
state discarded

show = true
new Counter created
count = initial state
```

Unmounting removes that component instance's state.

---

## 10. State Is Isolated Between Component Instances ⭐⭐⭐⭐⭐

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

These are two independent component instances.

```text
App
├── Counter position A
│   └── count state
│
└── Counter position B
    └── separate count state
```

Updating one does not update the other.

The component function is the same, but each tree position has its own state.

---

## 11. Position Matters ⭐⭐⭐⭐⭐

Suppose:

```jsx
function App({
  showFirst,
}) {
  return (
    <>
      {showFirst && <Counter />}
      <p>Hello</p>
    </>
  );
}
```

Changing tree structure can change which component occupies a position.

This is why React's reconciliation rules and keys matter.

Do not think of state as attached merely to a variable name or JSX line.

Think in terms of the rendered component tree.

---

## 12. Component Identity ⭐⭐⭐⭐⭐

A useful simplified identity model is:

```text
Component identity
      ↓
type
+
position among siblings
+
key when supplied
```

When identity stays compatible:

```text
preserve state
```

When identity changes:

```text
reset old state
create new component state
```

This is a conceptual model; React's reconciliation implementation is more sophisticated, but this is the correct practical reasoning model.

---

# Keys and State

## 13. Keys Are Not Only for Lists ⭐⭐⭐⭐⭐

You learned that keys help React identify list items.

Keys can also tell React that two components at the same position represent different identities.

Example:

```jsx
function Profile({
  user,
}) {
  return (
    <ProfileForm
      key={user.id}
      user={user}
    />
  );
}
```

When:

```text
user.id changes
```

the key changes.

React treats the form as a different component instance and resets its local state.

---

## 14. Resetting State with a Key ⭐⭐⭐⭐⭐

Suppose:

```jsx
function Chat({
  contact,
}) {
  const [message, setMessage] =
    useState("");

  // ...
}
```

Parent:

```jsx
<Chat
  key={contact.id}
  contact={contact}
/>
```

When changing from:

```text
contact.id = 10
```

to:

```text
contact.id = 20
```

React sees:

```text
Chat key=10
       ↓
different identity
       ↓
Chat key=20
```

The old draft state is discarded.

This can prevent accidentally sending one person's draft message to another person.

---

## 15. Preserve vs Reset with Key ⭐⭐⭐⭐⭐

Without changing key:

```jsx
<Chat contact={contact} />
```

React may preserve local state when the component remains at the same position.

With identity key:

```jsx
<Chat
  key={contact.id}
  contact={contact}
/>
```

a contact change intentionally resets the local state.

Decision:

```text
Same logical entity?
      ↓
preserve state

Different logical entity?
      ↓
consider distinct key
      ↓
reset state
```

---

## 16. Keys Give Identity Within the Parent

Keys are meaningful relative to sibling positions under a parent.

You do not need globally unique keys across the whole application.

Example:

```jsx
{users.map((user) => (
  <UserCard
    key={user.id}
    user={user}
  />
))}
```

The key helps React identify each sibling `UserCard`.

---

## 17. Why Index Keys Can Cause State Bugs ⭐⭐⭐⭐⭐

Suppose:

```text
Index 0 → Alice
Index 1 → Bob
Index 2 → Carol
```

Each row has local input state.

If Alice is removed:

```text
Index 0 → Bob
Index 1 → Carol
```

With index keys, React may associate the state previously at index 0 with the new item at index 0.

This can make state appear to move to the wrong row.

Stable IDs are safer when items can:

- reorder
- be inserted
- be removed
- be filtered

---

## 18. Identity and List Reordering ⭐⭐⭐⭐⭐

Stable keys:

```jsx
items.map((item) => (
  <Row
    key={item.id}
    item={item}
  />
))
```

allow React to understand:

```text
Row 42 moved
```

instead of:

```text
the component at index 0
became some different data
```

Keys help preserve the correct state with the correct logical item.

---

## 19. Random Keys Destroy State ⭐⭐⭐⭐⭐

Bad:

```jsx
<Component
  key={Math.random()}
/>
```

Every render produces a new key.

React sees:

```text
old identity
    ↓
gone

new identity
    ↓
mount new component
```

Effects include:

- state resets
- DOM recreated
- input focus may be lost
- unnecessary work

Do not use unstable random keys.

---

# Intentional State Reset

## 20. Resetting a Form by Changing Key ⭐⭐⭐⭐⭐

Suppose:

```jsx
function EditUser({
  user,
}) {
  return (
    <UserForm
      key={user.id}
      user={user}
    />
  );
}
```

Inside:

```jsx
function UserForm({
  user,
}) {
  const [name, setName] =
    useState(user.name);

  // ...
}
```

Because `user.id` identifies the logical form instance, changing users remounts the form and initializes state from the new user.

This is one legitimate way to intentionally reset local draft state.

---

## 21. Resetting State Manually vs with Identity

Manual:

```jsx
setName("");
setEmail("");
setBio("");
```

Identity reset:

```jsx
<Form key={formId} />
```

Neither is universally better.

Use explicit setters when:

- only certain fields should reset
- reset behavior is part of an event
- preserving other local state matters

Use a key when:

- the whole subtree represents a different logical entity
- all local state should start fresh

---

## 22. Do Not Abuse Keys to Fix Unknown Bugs

Bad approach:

```jsx
<Component key={Date.now()} />
```

just because:

> "The component is behaving strangely."

A key reset destroys state rather than solving the underlying ownership or synchronization problem.

Use identity changes intentionally.

---

# A Critical Pitfall

## 23. Do Not Define Component Functions Inside Components ⭐⭐⭐⭐⭐

Bad:

```jsx
function Parent() {
  function Child() {
    const [count, setCount] =
      useState(0);

    return (
      <button
        onClick={() =>
          setCount((count) => count + 1)
        }
      >
        {count}
      </button>
    );
  }

  return <Child />;
}
```

Every time `Parent` renders, JavaScript creates a new `Child` function.

React can see a different component type.

This can cause the child's state to reset unexpectedly.

---

## 24. Define Components at the Top Level ⭐⭐⭐⭐⭐

Better:

```jsx
function Child() {
  const [count, setCount] =
    useState(0);

  return (
    <button
      onClick={() =>
        setCount((count) => count + 1)
      }
    >
      {count}
    </button>
  );
}

function Parent() {
  return <Child />;
}
```

Now the component type is stable across parent renders.

Rule:

> Define component functions at the top level, not inside another component's render function.

---

## 25. Why Nested Definitions Break Identity

Each parent render can create:

```text
Render #1
Child function A

Render #2
Child function B
```

Even though both functions are named `Child`, their function references are different.

Conceptually:

```text
type A !== type B
```

React may therefore replace the subtree instead of preserving its state.

---

## 26. Conditional Rendering and Preservation

Consider:

```jsx
{isLoggedIn ? (
  <Dashboard />
) : (
  <Login />
)}
```

Switching branches changes component type at that position.

State resets.

But:

```jsx
<Dashboard
  theme={
    dark ? "dark" : "light"
  }
/>
```

only changes a prop.

The component type and position remain the same, so its state can be preserved.

---

## 27. Same Type, Different Props Usually Preserves State ⭐⭐⭐⭐⭐

```jsx
<Profile user={selectedUser} />
```

Changing:

```text
selectedUser A
      ↓
selectedUser B
```

does not automatically mean:

```text
new Profile identity
```

If `Profile` remains the same type at the same position with the same key, local state is normally preserved.

This is why sometimes you intentionally add:

```jsx
key={selectedUser.id}
```

when the state should reset for each user.

---

## 28. Props Change vs Identity Change ⭐⭐⭐⭐⭐

### Props change

```text
Profile
key unchanged
position unchanged
type unchanged
      ↓
preserve local state
      ↓
render with new props
```

### Identity change

```text
Profile key=A
      ↓
Profile key=B
      ↓
old state discarded
new state initialized
```

This distinction is extremely important.

---

## 29. Real-World Example: Chat Draft ⭐⭐⭐⭐⭐

Imagine:

```text
Contacts
├── Alice
├── Bob
└── Carol

Chat panel
└── message draft
```

If the user types:

```text
"Hi Alice"
```

then switches to Bob, should that draft remain?

Usually:

```text
No
```

You can model each conversation as a different identity:

```jsx
<Chat
  key={selectedContact.id}
  contact={selectedContact}
/>
```

Now switching contact resets the chat's local draft state.

---

## 30. Real-World Example: Product Quantity

Suppose:

```jsx
<ProductDetails
  product={product}
/>
```

Inside:

```jsx
const [quantity, setQuantity] =
  useState(1);
```

If navigating from Product A to Product B while reusing the same component position, quantity might remain.

If each product should begin at quantity 1:

```jsx
<ProductDetails
  key={product.id}
  product={product}
/>
```

This intentionally resets local state when product identity changes.

---

## 31. When You Want State Preserved

Preserve state when the component represents the same logical thing.

Examples:

- theme prop changes
- parent re-renders
- unrelated sibling changes
- validation message appears
- data props update but component identity remains conceptually the same

Do not add a new key just to force fresh rendering.

---

## 32. When You Want State Reset

Reset state when the UI represents a new logical entity or workflow.

Examples:

- editing a different user
- opening a different chat
- switching to a different product form
- starting a completely new wizard session
- resetting an entire form subtree

Changing identity with a key can make that intention explicit.

---

## 33. State Preservation Is About Tree Structure ⭐⭐⭐⭐⭐

Do not reason only from source code:

```text
"These JSX tags are on different lines."
```

React reasons about the resulting UI tree.

Ask:

```text
What component type is at this position?

What key identifies it?

Did it remain in the tree?
```

These questions predict preservation much better.

---

## 34. Connection to Reconciliation ⭐⭐⭐⭐⭐

React compares the previous and next trees.

Simplified:

```text
Previous element
      ↓
compare
      ↓
Next element

same compatible identity?
      │
   yes│
      ↓
reuse component instance
preserve state

different identity?
      │
      ↓
remove old subtree
create new subtree
reset state
```

Reconciliation is covered deeply later in the React Internals section.

---

## 35. State Is Not Stored in the DOM

A React component's state is not simply stored inside its rendered DOM node.

React manages component state and associates it with the component tree.

The DOM is the committed UI output.

This is why reasoning about React identity is different from reasoning only about HTML elements.

---

## 36. Common Mistakes ⭐⭐⭐⭐⭐

### Mistake 1 — Expecting every re-render to reset state

Re-rendering normally preserves state when identity remains the same.

### Mistake 2 — Expecting new props to automatically reset local state

Same type + position + key generally preserves local state.

### Mistake 3 — Using array indexes as keys for dynamic stateful lists

State can become associated with the wrong logical item.

### Mistake 4 — Using random keys

This can remount components every render and destroy state.

### Mistake 5 — Defining components inside components

The component type can become unstable and state can reset.

### Mistake 6 — Adding keys everywhere

Keys should express meaningful identity, not be used blindly.

### Mistake 7 — Using a key to hide an ownership bug

Understand why state should reset before forcing remounting.

### Mistake 8 — Assuming JSX source position defines identity

The rendered tree is what matters.

---

## 37. Interview Questions ⭐⭐⭐⭐⭐

### Q1. When does React preserve component state?

**Answer:** When React can match the component to the same compatible identity in the next rendered tree, typically based on type, position, and key where applicable.

### Q2. Does a re-render reset state?

**Answer:** No. Re-renders normally preserve state if component identity remains the same.

### Q3. What happens when a component is removed from the tree?

**Answer:** Its component instance is unmounted and its local state is discarded.

### Q4. Can changing props reset state automatically?

**Answer:** Usually no. If type, position, and key remain compatible, local state is preserved while the component renders with new props.

### Q5. How can you intentionally reset component state?

**Answer:** Change its identity, commonly by providing a different meaningful `key`, or remove and recreate the component.

### Q6. Why can index keys cause state bugs?

**Answer:** When items reorder, insert, or delete, indexes can identify positions rather than logical items, causing state to be reused for the wrong data.

### Q7. Why are random keys bad?

**Answer:** They change every render, causing React to treat components as new instances and remount them unnecessarily.

### Q8. Why should component functions not usually be declared inside another component?

**Answer:** A new function object can be created on each parent render, changing the component type identity and causing subtree state to reset.

### Q9. Do keys need to be globally unique?

**Answer:** No. They need to distinguish sibling items within the relevant parent.

### Q10. What is the practical identity mental model?

**Answer:** Component type + tree position + key when relevant.

---

## 38. Interview Scenario ⭐⭐⭐⭐⭐

Consider:

```jsx
function App({
  user,
}) {
  return (
    <ProfileForm
      user={user}
    />
  );
}
```

Inside:

```jsx
function ProfileForm({
  user,
}) {
  const [name, setName] =
    useState(user.name);

  // ...
}
```

The selected user changes.

Will `name` automatically reset to the new `user.name`?

**Answer:**

Not necessarily.

If `ProfileForm` remains the same component at the same position with the same key, React preserves its local state.

If each user should have a fresh form:

```jsx
<ProfileForm
  key={user.id}
  user={user}
/>
```

can intentionally give each user a different component identity.

---

## 39. Another Interview Scenario ⭐⭐⭐⭐⭐

What is wrong here?

```jsx
function App() {
  const [theme, setTheme] =
    useState("light");

  function Counter() {
    const [count, setCount] =
      useState(0);

    return (
      <button
        onClick={() =>
          setCount((count) => count + 1)
        }
      >
        {count}
      </button>
    );
  }

  return (
    <>
      <Counter />

      <button
        onClick={() =>
          setTheme(
            theme === "light"
              ? "dark"
              : "light"
          )
        }
      >
        Change Theme
      </button>
    </>
  );
}
```

Every `App` render creates a new `Counter` component function.

This can make React treat the child as a different component type and reset its state.

Move `Counter` to top-level module scope.

---

## 40. Preservation Decision Guide ⭐⭐⭐⭐⭐

Ask:

```text
Is this still the same logical component/entity?
             │
        ┌────┴────┐
       yes        no
        │          │
        ↓          ↓
preserve state   reset state
        │          │
same stable     different meaningful
identity        identity/key
```

Examples:

```text
Theme changed
→ same entity
→ preserve

Selected chat changed
→ different conversation
→ often reset draft

List reordered
→ same items moved
→ stable item IDs preserve correct state
```

---

## 41. Complete Mental Model ⭐⭐⭐⭐⭐

```text
Previous React Tree
        │
        ↓
Next React Tree
        │
        ↓
React compares identity
        │
   ┌────┴─────┐
   │          │
same        different
identity    identity
   │          │
   ↓          ↓
reuse       remove old
instance    create new
   │          │
   ↓          ↓
preserve    initialize
state       fresh state
```

Remember:

```text
state is tied to
a component's identity
in the rendered tree
```

---

## 42. Quick Revision

State is generally preserved when:

```text
same type
+
same relevant position
+
same key
```

State resets when:

```text
component removed
OR
type changes
OR
meaningful key changes
OR
identity otherwise changes
```

Keys:

```text
stable key
→ preserve correct logical identity

changed key
→ intentionally create new identity

random key
→ accidental remount every render
```

And:

```text
Never define ordinary component
functions inside another component
when you expect stable identity.
```

---

## 43. Key Takeaways

- React associates state with component identity in the rendered tree.
- Re-rendering alone does not reset local state.
- Same compatible component identity generally preserves state.
- Removing a component discards its local state.
- Different instances of the same component have independent state.
- Changing component type at a position resets the old subtree.
- Changing ordinary props does not automatically reset local state.
- Keys help React identify logical component instances.
- A meaningful key change can intentionally reset a component and its subtree.
- Stable keys preserve state with the correct list items during reordering.
- Index keys can cause state bugs in dynamic lists.
- Random keys cause unnecessary remounting and state loss.
- Component functions should normally be defined at top-level module scope, not inside other components.
- Use key-based resets intentionally, not as a generic bug fix.
- Think about the resulting component tree rather than JSX source-code lines.
- The practical identity model is **type + position + key when relevant**.
- Understanding identity prepares you for reconciliation, keys, and advanced rendering behavior.

---

## Section 2 Complete

You have now covered the core React state and rendering mental model:

```text
useState
   ↓
Props vs State
   ↓
Rendering / Re-rendering
   ↓
State as a Snapshot
   ↓
Batching + Update Queue
   ↓
Immutable Object/Array Updates
   ↓
Controlled vs Uncontrolled
   ↓
State Ownership + Colocation
   ↓
Component Identity
```

The next section moves into **Effects and Refs**.

---

## Next Lesson

➡️ [Lesson 19 — useEffect Fundamentals ⭐⭐⭐⭐⭐](../03-effects-and-refs/19-useeffect.md)
