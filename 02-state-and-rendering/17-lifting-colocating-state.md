# Lesson 17 — Lifting State Up and State Colocation

One of the most important React design questions is:

> Where should state live?

The best rule is:

> Keep state as low as possible, but as high as necessary.

---

## 1. State Colocation

If only one component needs a piece of state, keep it local.

```jsx
function DeveloperCard({
  developer,
}) {
  const [
    expanded,
    setExpanded,
  ] = useState(false);

  return (
    <>
      <button
        onClick={() =>
          setExpanded(
            (value) => !value
          )
        }
      >
        Toggle
      </button>

      {expanded && (
        <p>
          {developer.bio}
        </p>
      )}
    </>
  );
}
```

If no other component needs `expanded`, there is no reason to move it higher.

This is **state colocation**.

---

## 2. Lifting State Up

If multiple sibling components need the same state, move it to their closest common parent.

Bad:

```text
SearchInput
query = "react"

SearchResults
query = "react"
```

Now you have duplicated state.

Better:

```text
        SearchPage
      query = "react"
        /       \
       ↓         ↓
SearchInput   SearchResults
```

---

## 3. Basic Example

```jsx
function SearchPage() {
  const [query, setQuery] =
    useState("");

  return (
    <>
      <SearchInput
        query={query}
        onQueryChange={
          setQuery
        }
      />

      <SearchResults
        query={query}
      />
    </>
  );
}

function SearchInput({
  query,
  onQueryChange,
}) {
  return (
    <input
      value={query}
      onChange={(event) =>
        onQueryChange(
          event.target.value
        )
      }
    />
  );
}
```

The parent owns the state.

Children receive data and callbacks through props.

---

## 4. Single Source of Truth

For each important piece of shared state, choose one owner.

```text
Parent
selectedId = 10
   /      \
  ↓        ↓
List    Details
```

Both children use the same source of truth.

This avoids synchronization bugs.

---

## 5. Lift to the Closest Common Parent

Suppose:

```text
App
 └── Dashboard
      ├── SearchBar
      └── DeveloperList
```

If only `SearchBar` and `DeveloperList` need the query, put it in:

```text
Dashboard
```

not automatically in:

```text
App
```

Do not lift state higher than necessary.

---

## 6. Child Requests Changes Through Callbacks

Data flows down:

```text
Parent state
   ↓
props
   ↓
Child
```

Changes flow up through callback props:

```text
Child event
   ↓
callback prop
   ↓
Parent setter
```

Example:

```jsx
function Parent() {
  const [count, setCount] =
    useState(0);

  return (
    <CounterButton
      count={count}
      onIncrement={() =>
        setCount(
          (count) =>
            count + 1
        )
      }
    />
  );
}
```

---

## 7. Do Not Duplicate Derived State

Bad:

```jsx
const [users, setUsers] =
  useState([]);

const [userCount, setUserCount] =
  useState(0);
```

if count is always:

```js
users.length
```

Prefer:

```js
const userCount =
  users.length;
```

Store the source state, derive the rest.

---

## 8. Do Not Lift Everything

This is unnecessary:

```text
App
 └── modalHoverState
      ↓
many components
      ↓
tiny child
```

If only the child needs that state, keep it there.

Lifting everything can cause:

- prop drilling
- unnecessary coupling
- harder code
- wider re-render scope

---

## 9. Practical Decision Rule

Ask:

```text
Who needs this state?
```

If one component needs it:

```text
keep it local
```

If siblings need it:

```text
lift it to closest common parent
```

If many distant components need it:

```text
consider Context
or another state solution
```

Do not reach for global state first.

---

## Common Mistakes

### Mistake 1 — Duplicating the same state in multiple components

This creates synchronization problems.

### Mistake 2 — Lifting state too high

Keep state close to where it is used.

### Mistake 3 — Storing derived values separately

Calculate them when possible.

### Mistake 4 — Using global state for local UI behavior

Local state should usually remain local.

---

## Interview Questions

### What does lifting state up mean?

Moving shared state to the closest common parent so multiple children can use one source of truth.

### What is state colocation?

Keeping state as close as possible to the component that actually needs it.

### Are lifting state and colocation opposites?

No. They work together.

### What is the main rule?

Keep state as low as possible, but as high as necessary.

---

## Quick Revision

```text
One component needs it
→ local state

Multiple siblings need it
→ lift to closest common parent

Value can be calculated
→ derive it, don't duplicate state
```

Remember:

> State ownership should follow who needs to read and change the data.

---

## Next Lesson

➡️ [Lesson 18 — Preserving and Resetting State; Component Identity](./18-preserving-resetting-state.md)
