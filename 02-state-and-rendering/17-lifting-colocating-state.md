# Lesson 17 — Lifting State Up and State Colocation ⭐⭐⭐⭐⭐

## 1. Why This Topic Matters

One of the most important React architecture decisions is:

> **Where should state live?**

If state lives too low, multiple components cannot coordinate.

If state lives too high, unrelated components may depend on it and unnecessary parts of the tree may re-render.

Two important ideas solve this:

- **Lifting state up** — move shared state to the closest common parent.
- **State colocation** — keep state as close as possible to the components that actually need it.

These ideas are complementary, not opposites.

---

## 2. The Core Rule ⭐⭐⭐⭐⭐

A useful rule is:

> Keep state as low as possible, but as high as necessary.

```text
Only one component needs it?
        ↓
Keep it local

Multiple siblings need it?
        ↓
Lift it to their closest common parent
```

---

## 3. What Is Lifting State Up? ⭐⭐⭐⭐⭐

Suppose two sibling components need the same value:

```text
        Parent
       /      \
 Input          Preview
```

If both maintain independent copies:

```text
Input
query = "react"

Preview
query = "react"
```

you now have duplicated state that must stay synchronized.

Instead:

```text
          Parent
     query = "react"
        /       \
       ↓         ↓
    Input      Preview
```

The parent becomes the **single source of truth**.

---

## 4. Basic Lifting State Example ⭐⭐⭐⭐⭐

```jsx
function SearchPage() {
  const [query, setQuery] = useState("");

  return (
    <>
      <SearchInput
        query={query}
        onQueryChange={setQuery}
      />

      <SearchPreview query={query} />
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
        onQueryChange(event.target.value)
      }
    />
  );
}

function SearchPreview({ query }) {
  return <p>Searching for: {query}</p>;
}
```

Flow:

```text
SearchPage state
      │
      ├── query prop ─────→ SearchInput
      │
      └── query prop ─────→ SearchPreview

SearchInput
      │
      └── onQueryChange
               ↓
          SearchPage
               ↓
          update state
               ↓
          new props flow down
```

---

## 5. Why Lift State? ⭐⭐⭐⭐⭐

Lift state when multiple components must coordinate around the same information.

Examples:

- search input + search results
- cart items + cart summary
- selected tab + tab content
- filters + product list
- selected developer + profile panel
- form fields + live preview

Without a shared owner, components can drift out of sync.

---

## 6. Single Source of Truth ⭐⭐⭐⭐⭐

For each piece of state, identify one owner.

Bad:

```text
Component A
selectedId = 10

Component B
selectedId = 10
```

Now both must somehow stay synchronized.

Better:

```text
        Parent
    selectedId = 10
       /       \
      ↓         ↓
Component A   Component B
```

The parent owns the state.

Children receive it through props.

---

## 7. Children Request Changes Through Callbacks

A child should not directly mutate parent state.

Instead:

```text
Parent state
    ↓
props
    ↓
Child
    ↓
event
    ↓
callback prop
    ↓
Parent setter
    ↓
new state
```

Example:

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <CounterButton
      count={count}
      onIncrement={() =>
        setCount((count) => count + 1)
      }
    />
  );
}

function CounterButton({
  count,
  onIncrement,
}) {
  return (
    <button onClick={onIncrement}>
      {count}
    </button>
  );
}
```

This preserves one-way data flow.

---

## 8. Lift State to the Closest Common Parent ⭐⭐⭐⭐⭐

Suppose:

```text
App
 └── Dashboard
      ├── SearchBar
      └── DeveloperList
```

If only `SearchBar` and `DeveloperList` need `query`, the likely owner is:

```text
Dashboard
```

not necessarily:

```text
App
```

Why?

Because state should not be lifted higher than necessary.

---

## 9. What Is State Colocation? ⭐⭐⭐⭐⭐

State colocation means:

> Put state close to where it is actually used.

Example:

```jsx
function DeveloperCard({
  developer,
}) {
  const [isExpanded, setIsExpanded] =
    useState(false);

  return (
    <article>
      <h2>{developer.name}</h2>

      <button
        onClick={() =>
          setIsExpanded(
            (expanded) => !expanded
          )
        }
      >
        Toggle Details
      </button>

      {isExpanded && (
        <p>{developer.bio}</p>
      )}
    </article>
  );
}
```

If no other component cares whether this card is expanded, the state belongs inside `DeveloperCard`.

There is no reason to move it to the application root.

---

## 10. Why Colocation Matters

Compare:

```text
App
 └── isCardExpanded
      ↓
   Dashboard
      ↓
 DeveloperList
      ↓
 DeveloperCard
```

with:

```text
DeveloperCard
 └── isExpanded
```

The second design has:

- simpler ownership
- fewer props
- fewer dependencies
- easier component reuse
- smaller re-render scope

---

## 11. "Lift Everything Up" Is Not Good Architecture ⭐⭐⭐⭐⭐

A common mistake is assuming all state should live at the top.

Bad:

```jsx
function App() {
  const [search, setSearch] = useState("");
  const [modalOpen, setModalOpen] = useState(false);
  const [hoveredCard, setHoveredCard] = useState(null);
  const [draftText, setDraftText] = useState("");
  const [activeTooltip, setActiveTooltip] = useState(null);

  // ...
}
```

If these values belong to unrelated subtrees, `App` becomes an unnecessary state manager.

Better:

```text
App
├── SearchFeature
│    └── search state
│
├── ModalFeature
│    └── modal state
│
└── Editor
     └── draft state
```

---

## 12. State Ownership Decision ⭐⭐⭐⭐⭐

Ask:

### Question 1
Which components need to read this value?

### Question 2
Which components need to change it?

### Question 3
What is their closest common parent?

That parent is often the correct owner.

```text
Need state in A only?
     ↓
A owns it

Need state in A + B?
     ↓
closest common parent owns it

Need state across distant application areas?
     ↓
consider Context/external state
only when justified
```

---

## 13. Example: Temperature Inputs

Suppose Celsius and Fahrenheit inputs must stay synchronized.

Bad:

```text
CelsiusInput
celsius state

FahrenheitInput
fahrenheit state
```

Two independent states can become inconsistent.

Better:

```text
      TemperatureCalculator
          temperature
              │
        ┌─────┴─────┐
        ↓           ↓
    Celsius     Fahrenheit
```

The parent owns the canonical state and derives/passes the required values.

---

## 14. Do Not Duplicate Derived State ⭐⭐⭐⭐⭐

Suppose:

```jsx
const [items, setItems] = useState([]);
const [itemCount, setItemCount] = useState(0);
```

If:

```js
itemCount === items.length
```

always, then `itemCount` does not need separate state.

Prefer:

```jsx
const itemCount = items.length;
```

State ownership is not only about component location.

It is also about avoiding multiple sources of truth.

---

## 15. Example: Product Filter

```jsx
function ProductsPage({
  products,
}) {
  const [query, setQuery] = useState("");

  const filteredProducts =
    products.filter((product) =>
      product.name
        .toLowerCase()
        .includes(query.toLowerCase())
    );

  return (
    <>
      <ProductSearch
        query={query}
        onQueryChange={setQuery}
      />

      <ProductList
        products={filteredProducts}
      />
    </>
  );
}
```

Here:

```text
ProductsPage
├── owns query
├── derives filteredProducts
├── controls ProductSearch
└── supplies ProductList
```

This is a clean single-source-of-truth design.

---

## 16. Example: Local UI State Should Stay Local

```jsx
function ProductCard({
  product,
}) {
  const [showDetails, setShowDetails] =
    useState(false);

  // ...
}
```

If only the card uses `showDetails`, keep it there.

Do not lift state merely because lifting state is a React concept.

---

## 17. Lifting State Can Make a Component Controlled ⭐⭐⭐⭐⭐

Local component:

```jsx
function Accordion() {
  const [open, setOpen] = useState(false);

  // ...
}
```

After lifting:

```jsx
function Accordion({
  open,
  onOpenChange,
}) {
  // ...
}
```

Now the parent controls it:

```jsx
function Page() {
  const [open, setOpen] = useState(false);

  return (
    <Accordion
      open={open}
      onOpenChange={setOpen}
    />
  );
}
```

This connects Lesson 16 with lifting state.

---

## 18. Coordinating Multiple Components ⭐⭐⭐⭐⭐

Suppose only one accordion panel should be open.

If every panel owns independent state:

```text
Panel A → open
Panel B → open
Panel C → closed
```

multiple panels can remain open.

If the parent owns:

```jsx
const [activePanelId, setActivePanelId] =
  useState(null);
```

then:

```text
Parent
activePanelId = "B"
      │
      ├── A open? false
      ├── B open? true
      └── C open? false
```

The shared parent can enforce the coordination rule.

---

## 19. Real-World CodeBuddy Example

Imagine:

```text
DiscoveryPage
├── FilterBar
├── DeveloperList
└── ResultsCount
```

All three depend on filters.

Good ownership:

```text
DiscoveryPage
filters state
    │
    ├── FilterBar
    ├── DeveloperList
    └── ResultsCount
```

But each `DeveloperCard` might own:

```text
isBioExpanded
```

because that state is card-specific.

This demonstrates both principles together:

```text
shared filters
    → lift up

card expansion
    → colocate
```

---

## 20. Prop Drilling Can Appear After Lifting State

Suppose:

```text
App
 ↓
Dashboard
 ↓
Content
 ↓
Toolbar
 ↓
SearchInput
```

If `App` owns query and only `SearchInput` needs it, passing props through every layer may be unnecessary.

Before immediately using Context, ask:

> Is the state owned too high?

Maybe query should live in `Content` or `Toolbar`.

State colocation can often reduce prop drilling.

---

## 21. Prop Drilling Is Not Automatically Bad ⭐⭐⭐⭐⭐

Passing props through a few levels is normal React.

Do not immediately replace:

```text
Parent
 ↓
Child
 ↓
Grandchild
```

with global state or Context.

Prop drilling becomes problematic when:

- many intermediate components pass unrelated props
- state is genuinely needed across distant branches
- component APIs become difficult to maintain

Context is covered later.

---

## 22. State Ownership and Re-render Scope ⭐⭐⭐⭐⭐

Suppose state lives at:

```text
App
```

When it changes, React begins rendering from the component whose state changed.

That can cause descendants to be considered for rendering.

If state only belongs to:

```text
SearchBox
```

keeping it there can isolate updates to a smaller subtree.

Therefore state colocation can improve both architecture and performance.

But:

> Do not move state only for performance without considering correct ownership first.

Correct data ownership comes first.

---

## 23. Local State vs Shared State

### Local state

Used by one component or one small subtree.

Examples:

- dropdown open state
- tooltip visibility
- local input draft
- card expansion

### Shared state

Used by multiple components that must stay synchronized.

Examples:

- selected product
- active filter
- current cart
- selected tab controlling another view

Shared state often belongs in a common ancestor.

---

## 24. Server Data Is Not Automatically Local UI State

Suppose data comes from an API:

```text
GET /developers
```

Do not automatically copy every response into many component states.

Think about:

- who owns the data
- who needs it
- whether it is server/cache data
- whether local state represents UI interaction around it

State ownership applies to more than just `useState`.

---

## 25. Avoid Mirroring Props into State ⭐⭐⭐⭐⭐

Bad:

```jsx
function Profile({
  user,
}) {
  const [localUser, setLocalUser] =
    useState(user);

  // ...
}
```

Now you potentially have:

```text
Parent
user
  ↓
Child
localUser
```

Two copies can diverge.

If the child only needs to display `user`, use the prop directly.

If the child intentionally needs an editable draft, that is a different requirement and should be designed explicitly.

---

## 26. Intentional Draft State

Sometimes copying initial data is valid.

Example:

```jsx
function EditProfile({
  user,
}) {
  const [draft, setDraft] =
    useState(() => ({
      name: user.name,
      bio: user.bio,
    }));

  // user edits draft before saving
}
```

Here:

```text
user
  ↓
saved source data

draft
  ↓
temporary unsaved edits
```

These values have different meanings.

That is not accidental duplicated state.

---

## 27. State Colocation and Reusability

A reusable component should not depend on distant application state unless necessary.

Better:

```jsx
<DeveloperCard
  developer={developer}
  onConnect={handleConnect}
/>
```

with local presentation state inside the card.

This makes the component easier to:

- reuse
- test
- understand
- move elsewhere

---

## 28. Common Mistakes ⭐⭐⭐⭐⭐

### Mistake 1 — Duplicating shared state in siblings

Use one common owner.

### Mistake 2 — Lifting all state to App

Keep unrelated state closer to where it is used.

### Mistake 3 — Copying props into state without a reason

This creates multiple sources of truth.

### Mistake 4 — Using Context immediately to avoid a few props

First verify whether ownership is correct.

### Mistake 5 — Keeping state too low when siblings must coordinate

Lift it to the closest common parent.

### Mistake 6 — Storing derived values as separate state

Calculate them when possible.

### Mistake 7 — Optimizing re-renders before fixing ownership

Correct architecture should come first.

---

## 29. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What does lifting state up mean?

**Answer:** Moving state from child components to their closest common parent so multiple components can share one source of truth.

### Q2. What is state colocation?

**Answer:** Keeping state as close as possible to the components that actually need it.

### Q3. Are lifting state and colocation contradictory?

**Answer:** No. Keep state low by default, but lift it high enough for every component that must coordinate around it.

### Q4. Where should shared sibling state live?

**Answer:** Usually in their closest common ancestor.

### Q5. What is a single source of truth?

**Answer:** One authoritative owner for a piece of state rather than duplicated synchronized copies.

### Q6. How does a child update parent-owned state?

**Answer:** The parent passes a callback prop, and the child calls it to request the update.

### Q7. Why can excessive lifting hurt performance?

**Answer:** Updates in a high-level component can cause larger descendant subtrees to render. More importantly, it also creates unnecessary dependencies.

### Q8. Is prop drilling always bad?

**Answer:** No. Passing props is normal. It becomes problematic when many unrelated intermediate components must forward data across distant parts of the tree.

### Q9. Should props usually be copied into state?

**Answer:** No. Use props directly unless the local state represents intentionally different information, such as an editable draft.

### Q10. What is the rule for choosing state ownership?

**Answer:** Keep state as low as possible but as high as necessary.

---

## 30. Interview Scenario ⭐⭐⭐⭐⭐

You have:

```text
ProductsPage
├── SearchInput
├── ProductList
└── ResultCount
```

All three depend on the search query.

Where should query state live?

**Answer:**

```text
ProductsPage
```

because it is the closest common owner coordinating those children.

But if a single `ProductCard` has a local tooltip:

```text
ProductCard
└── tooltip state
```

keep that state inside the card.

---

## 31. Complete Mental Model ⭐⭐⭐⭐⭐

```text
Start with the state
       ↓
Who needs it?
       ↓
Only one component?
       │
       └── colocate there
       
Multiple components?
       │
       ↓
Find closest common parent
       │
       └── lift state there
                ↓
          pass value down
                ↓
          pass callbacks down
                ↓
          children request changes
```

Remember:

```text
LOW AS POSSIBLE
HIGH AS NECESSARY
```

---

## 32. Key Takeaways

- State ownership is a core React architecture decision.
- Lifting state up gives multiple components a shared source of truth.
- Shared sibling state usually belongs in their closest common parent.
- State colocation keeps state near the components that actually use it.
- Lifting state and colocation work together.
- Do not lift all state to the application root.
- Children receive parent-owned state through props.
- Children request parent updates through callback props.
- Avoid duplicated state that must remain synchronized.
- Avoid copying props into state without an intentional reason.
- Derived values usually do not need separate state.
- Prop drilling is not automatically a problem.
- Before introducing Context, check whether state ownership is too high.
- Colocation can reduce dependencies and limit the scope of updates.
- The key rule is: **keep state as low as possible, but as high as necessary.**

---

## Next Lesson

➡️ [Lesson 18 — Preserving and Resetting State; Component Identity ⭐⭐⭐⭐⭐](./18-preserving-resetting-state.md)
