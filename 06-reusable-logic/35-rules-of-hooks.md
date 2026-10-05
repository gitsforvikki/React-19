# Lesson 35 — Rules of Hooks ⭐⭐⭐⭐⭐

## 1. Why Hooks Have Rules

Hooks look like normal JavaScript functions:

```jsx
useState()
useEffect()
useContext()
```

but React uses them to associate stateful behavior with a component across renders.

React needs ordinary Hook calls to occur in a predictable order.

```text
Render 1:
Hook #1 → state A
Hook #2 → effect B
Hook #3 → state C

Render 2:
Hook #1 → state A
Hook #2 → effect B
Hook #3 → state C
```

If the order changes, React can no longer reliably match Hook calls to their stored state/effect information.

---

## 2. The Two Core Rules ⭐⭐⭐⭐⭐

For ordinary Hooks:

1. **Only call Hooks at the top level.**
2. **Only call Hooks from React functions.**

React functions here means:

- function components
- custom Hooks

These rules apply to built-in Hooks and custom Hooks.

---

## 3. Rule 1 — Call Hooks at the Top Level ⭐⭐⭐⭐⭐

Correct:

```jsx
function Profile() {
  const [name, setName] =
    useState("");

  const user =
    useContext(UserContext);

  useEffect(() => {
    // ...
  }, []);

  // ...
}
```

The Hook calls are unconditional and occur in the same structural order on every render.

---

## 4. Don't Call Ordinary Hooks in Conditions

Wrong:

```jsx
function Profile({
  loggedIn,
}) {
  if (loggedIn) {
    const [user, setUser] =
      useState(null); // ❌
  }

  // ...
}
```

Why?

```text
Render A: loggedIn = true
Hook #1 exists

Render B: loggedIn = false
Hook #1 disappears
```

The Hook call structure changes.

---

## 5. Put the Condition Inside the Hook Logic

Wrong:

```jsx
if (enabled) {
  useEffect(() => {
    subscribe();
  }, []);
}
```

Correct:

```jsx
useEffect(() => {
  if (!enabled) {
    return;
  }

  const unsubscribe =
    subscribe();

  return unsubscribe;
}, [enabled]);
```

The Hook is always called; its behavior is conditional.

---

## 6. Don't Call Hooks in Loops ⭐⭐⭐⭐⭐

Wrong:

```jsx
for (const item of items) {
  const [open, setOpen] =
    useState(false); // ❌
}
```

The number/order of items can change, so Hook order can change.

Correct architecture:

```jsx
function Item({ item }) {
  const [open, setOpen] =
    useState(false);

  // ...
}

function List({ items }) {
  return items.map(
    (item) => (
      <Item
        key={item.id}
        item={item}
      />
    )
  );
}
```

Each component instance owns its own Hook sequence.

---

## 7. Don't Call Hooks After a Conditional Early Return ⭐⭐⭐⭐⭐

Wrong:

```jsx
function Profile({
  user,
}) {
  if (!user) {
    return null;
  }

  const [tab, setTab] =
    useState("posts"); // ❌
}
```

Some renders call the Hook; others return before it.

Better:

```jsx
function Profile({
  user,
}) {
  const [tab, setTab] =
    useState("posts");

  if (!user) {
    return null;
  }

  // ...
}
```

Or restructure the component so state belongs only in a child that renders when needed.

---

## 8. Don't Call Hooks in Event Handlers

Wrong:

```jsx
function Button() {
  function handleClick() {
    const [count, setCount] =
      useState(0); // ❌
  }

  return (
    <button
      onClick={handleClick}
    >
      Click
    </button>
  );
}
```

Event handlers execute outside the component's render Hook sequence.

Call the Hook at component top level, then use its values in the handler.

---

## 9. Don't Call Hooks in Nested Functions

Wrong:

```jsx
function Profile() {
  function loadUser() {
    const user =
      useContext(
        UserContext
      ); // ❌
  }

  // ...
}
```

A nested function is not automatically a component or custom Hook.

If reusable Hook logic is needed, create a custom Hook with a valid API.

---

## 10. Don't Call Hooks in Ordinary Callbacks

Wrong:

```jsx
items.map((item) => {
  const [open, setOpen] =
    useState(false); // ❌

  return ...;
});
```

Extract a component:

```jsx
items.map((item) => (
  <Item
    key={item.id}
    item={item}
  />
));
```

Then `Item` may call Hooks at its top level.

---

## 11. Don't Call Ordinary Hooks in try/catch/finally

Wrong:

```jsx
try {
  const data =
    useSomething(); // ❌
} catch {
  // ...
}
```

Ordinary Hook calls should remain at top level, not inside control-flow constructs that may alter call order.

---

## 12. Rule 2 — Only Call Hooks from React Functions ⭐⭐⭐⭐⭐

Correct:

```jsx
function Profile() {
  const [name, setName] =
    useState("");
}
```

Correct:

```jsx
function useProfile() {
  const [profile, setProfile] =
    useState(null);

  return profile;
}
```

Wrong:

```jsx
function calculateTotal() {
  const [total, setTotal] =
    useState(0); // ❌
}
```

Ordinary JavaScript functions are not part of React's Hook execution model.

---

## 13. Components Must Be Called by React ⭐⭐⭐⭐⭐

Don't manually call a component as a normal function:

```jsx
function Parent() {
  return Child(); // ❌
}
```

Use JSX:

```jsx
function Parent() {
  return <Child />;
}
```

Why?

React should control component rendering and Hook execution.

This also preserves component identity and allows React to use its rendering model correctly.

---

## 14. Don't Dynamically Treat Hooks as Regular Values

Avoid patterns that make Hook execution dynamic or indirect.

Bad idea:

```jsx
const selectedHook =
  condition
    ? useA
    : useB;

const value =
  selectedHook();
```

Prefer explicit component/Hook structure where Hook calls are statically understandable.

React tooling works best when Hook usage is clear and predictable.

---

## 15. Why Call Order Matters Internally ⭐⭐⭐⭐⭐

Simplified mental model:

```text
Component instance
     │
     ├── Hook slot 1
     ├── Hook slot 2
     └── Hook slot 3
```

Code:

```jsx
const [name] =
  useState("");

useEffect(...);

const [age] =
  useState(0);
```

Conceptually:

```text
slot 1 → name state
slot 2 → effect
slot 3 → age state
```

If the second call disappears conditionally:

```text
slot mapping shifts
→ React cannot match calls correctly
```

This is why ordinary Hook order must remain stable.

---

## 16. The React 19 use() Exception ⭐⭐⭐⭐⭐

React's `use()` API is special.

Unlike ordinary Hooks, `use` can be called in conditions and loops.

Example:

```jsx
if (shouldRead) {
  const value =
    use(resource);
}
```

This does **not** mean all Hooks can now be conditional.

```text
useState      → top-level rule
useEffect     → top-level rule
useContext    → top-level rule
custom Hooks  → top-level rule

use()         → special exception
```

You will learn the `use()` API deeply in Lesson 59.

---

## 17. use() Still Has Restrictions

The special `use()` API should not be treated as unrestricted ordinary JavaScript.

For example, it should not be called inside `try/catch`.

Follow the documented `use` constraints rather than generalizing its exception to other Hooks.

Interview point:

> `use()` is a special React API with different call constraints from ordinary Hooks.

---

## 18. Hook Naming Enables Tooling

Custom Hook:

```jsx
function useOnlineStatus() {
  // ...
}
```

The `use` prefix signals:

```text
this function may call Hooks
→ Hook rules apply
→ lint tooling can analyze usage
```

This is more than a cosmetic naming convention.

---

## 19. eslint-plugin-react-hooks ⭐⭐⭐⭐⭐

React's Hooks ESLint tooling catches common violations.

Two historically important checks are:

```text
rules-of-hooks
exhaustive-deps
```

Modern React linting also includes additional rules related to React correctness and compiler-compatible patterns.

Do not casually disable Hook lint rules just to remove warnings.

A warning often points to a real design problem.

---

## 20. rules-of-hooks vs exhaustive-deps

These solve different problems.

### rules-of-hooks

Checks **where/how Hooks are called**.

Example problem:

```jsx
if (x) {
  useEffect(...);
}
```

### exhaustive-deps

Checks reactive dependencies used by Hooks such as Effects.

Example:

```jsx
useEffect(() => {
  connect(roomId);
}, []); // roomId missing
```

Do not confuse the two.

---

## 21. Rules of Hooks vs Rules of React

Rules of Hooks are part of a broader React programming model.

Other important principles include:

- components and Hooks should be pure during render
- props/state are immutable snapshots
- side effects belong outside render
- React should call components and Hooks

Following these rules helps React safely pause, restart, and optimize rendering.

---

## 22. Purity and Hook Rules Work Together ⭐⭐⭐⭐⭐

Correct Hook order is not enough.

Bad:

```jsx
function useCounter() {
  globalCounter++; // ❌ render side effect

  const [count] =
    useState(0);

  return count;
}
```

The Hook call is top-level, but render purity is violated.

A correct React Hook should satisfy both:

```text
valid Hook call structure
+
pure render-time behavior
```

---

## 23. Strict Mode Helps Expose Problems

In development, Strict Mode may intentionally invoke render-related logic extra times.

Pure components and Hooks remain correct.

If your Hook depends on "render only happens once", that is a warning sign.

Design Hooks around React's rendering model, not around assumptions about exact render counts.

---

## 24. Invalid Hook Call Errors

An invalid Hook call warning can have multiple causes, including:

1. breaking Hook rules
2. mismatched React / renderer versions
3. multiple React copies in the same application

So if the code appears structurally correct, investigate dependency installation as well.

---

## 25. Custom Hooks Do Not Escape the Rules

This is wrong:

```jsx
function useMaybeState(
  enabled
) {
  if (enabled) {
    return useState(0); // ❌
  }

  return [0, () => {}];
}
```

Wrapping an invalid Hook call inside a custom Hook does not make it valid.

---

## 26. Correct Conditional Behavior Pattern

Instead of conditionally calling a Hook:

```jsx
function useSubscription(
  enabled
) {
  useEffect(() => {
    if (!enabled) {
      return;
    }

    const unsubscribe =
      subscribe();

    return unsubscribe;
  }, [enabled]);
}
```

The Hook sequence remains stable while behavior responds to inputs.

---

## 27. Conditional Component Rendering Is Fine

This is valid:

```jsx
function Page({
  showChat,
}) {
  return (
    <>
      {showChat &&
        <ChatRoom />}
    </>
  );
}
```

`ChatRoom` may call Hooks normally.

Why?

React conditionally creates/removes a **component instance**; it is not conditionally changing Hook order inside the same component render.

This distinction is important.

---

## 28. Hooks in Separate Components

Suppose every list item needs local state.

Correct:

```jsx
function Row({ item }) {
  const [selected, setSelected] =
    useState(false);

  // ...
}

function List({ items }) {
  return items.map(
    (item) => (
      <Row
        key={item.id}
        item={item}
      />
    )
  );
}
```

Each `Row` has its own Hook slots.

This is the React way to model variable numbers of stateful items.

---

## 29. CareerLoop Example

Wrong:

```jsx
applications.map(
  (application) => {
    const [expanded, setExpanded] =
      useState(false); // ❌

    return ...;
  }
);
```

Correct:

```jsx
function ApplicationCard({
  application,
}) {
  const [
    expanded,
    setExpanded,
  ] = useState(false);

  // ...
}
```

Then:

```jsx
applications.map(
  (application) => (
    <ApplicationCard
      key={application.id}
      application={
        application
      }
    />
  )
);
```

---

## 30. Common Mistakes ⭐⭐⭐⭐⭐

1. Calling `useState` inside an `if`.
2. Calling Hooks inside `map`.
3. Calling Hooks after a conditional early return.
4. Calling Hooks inside click handlers.
5. Calling Hooks inside ordinary helper functions.
6. Calling Hooks inside nested callbacks.
7. Hiding violations inside custom Hooks.
8. Manually calling component functions.
9. Disabling lint rules instead of fixing architecture.
10. Confusing `rules-of-hooks` with `exhaustive-deps`.
11. Assuming `use()` makes all Hooks conditional.
12. Following call-order rules while still performing side effects during render.

---

## 31. Interview Questions ⭐⭐⭐⭐⭐

### What are the Rules of Hooks?

Ordinary Hooks should be called at the top level and only from React function components or custom Hooks.

### Why can't Hooks be conditional?

React relies on stable Hook call structure/order to associate Hook state with the correct calls across renders.

### Can Hooks be called in loops?

Ordinary Hooks cannot. Extract a component/custom architecture so each component instance has a stable Hook sequence.

### Can Hooks be called in event handlers?

No. Event handlers execute outside React's render Hook sequence.

### Can a custom Hook call other Hooks?

Yes, while following Hook rules.

### Can a normal utility call useState?

No.

### Can a component be called like a normal function?

It should not be. Render it with JSX so React controls component execution.

### What does eslint-plugin-react-hooks do?

It statically checks Hook-related correctness, including Hook call rules and reactive dependency issues.

### Is use() subject to exactly the same top-level rule?

No. React's `use()` API is a special exception that can be called conditionally and in loops, though it has its own restrictions.

### Why does Hook order matter?

React associates Hook state/effect data with calls across renders; changing ordinary Hook order can shift that association.

---

## 32. Debugging Checklist

When you see a Hook error, check:

```text
□ Hook inside if?
□ Hook inside loop/map?
□ Hook after early return?
□ Hook inside event handler?
□ Hook inside nested function?
□ Hook inside try/catch?
□ Hook called from plain JS utility?
□ component manually invoked?
□ mismatched React packages?
□ duplicate React installation?
```

---

## 33. Complete Mental Model

```text
          React renders component
                   │
                   ↓
        same ordinary Hook order
       ┌───────────┼───────────┐
       ↓           ↓           ↓
    Hook #1     Hook #2     Hook #3
       │           │           │
     state       effect       state
       │           │           │
       └───────────┼───────────┘
                   ↓
        React can preserve
        correct Hook identity
```

Special note:

```text
use()
→ different API semantics
→ conditional/loop usage allowed
→ does not change rules for other Hooks
```

---

## 34. Quick Revision ⭐⭐⭐⭐⭐

Ordinary Hooks:

```text
✅ top level of component
✅ top level of custom Hook

❌ conditions
❌ loops
❌ callbacks
❌ event handlers
❌ after conditional return
❌ ordinary JS functions
❌ try/catch/finally
```

Remember:

```text
stable call structure
→ stable Hook association
```

---

## 35. Key Takeaways

- Ordinary Hooks must have predictable call structure.
- Call them at the top level of components/custom Hooks.
- Do not call them in conditions, loops, callbacks, or event handlers.
- Do not put them after conditional early returns.
- Extract stateful list items into components rather than calling Hooks in `map`.
- React should invoke components through JSX.
- Custom Hooks obey the same rules.
- Hook naming enables tooling to understand custom Hooks.
- `rules-of-hooks` and `exhaustive-deps` solve different problems.
- Render purity is required in addition to valid Hook order.
- Keep Hook lint rules enabled.
- React's `use()` API is a special exception; do not generalize its behavior to other Hooks.
- Invalid Hook call warnings can also come from dependency/version problems.
- Understanding the reason behind the rules is more valuable than memorizing them.

---

## Next Lesson

➡️ [Lesson 36 — Reusable Component APIs and Composition Patterns](./36-reusable-component-patterns.md)
