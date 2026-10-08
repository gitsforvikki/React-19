# Lesson 36 — Reusable Component APIs and Composition Patterns

## Why Reusable Components?

Reuse **UI and markup** through components; reuse **stateful behavior** through custom Hooks. A good component API stays small, meaningful, and accessible.

## 1. Composition with children ⭐⭐⭐⭐⭐

`children` lets a wrapper accept flexible content.

```jsx
function Card({ children }) {
  return <section className="card">{children}</section>;
}

function ApplicationCard({ application }) {
  return (
    <Card>
      <h2>{application.company}</h2>
      <p>Status: {application.status}</p>
    </Card>
  );
}
```

The generic `Card` does not need to know about applications, users, or products.

## 2. Named Slots for Different Regions

When a component needs multiple content areas, pass JSX as props:

```jsx
function PageLayout({ header, sidebar, children }) {
  return (
    <div>
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{children}</main>
    </div>
  );
}

<PageLayout header={<Header />} sidebar={<Filters />}>
  <ApplicationList />
</PageLayout>
```

## 3. Controlled vs Uncontrolled State ⭐⭐⭐⭐⭐

**Controlled:** The parent owns the state.

```jsx
function SearchInput({ value, onValueChange }) {
  return <input value={value} onChange={e => onValueChange(e.target.value)} />;
}

function SearchPage() {
  const [query, setQuery] = useState("");
  return <SearchInput value={query} onValueChange={setQuery} />;
}
```

**Uncontrolled (internally managed):** The component owns its state; `defaultOpen` sets the initial value.

```jsx
function Expandable({ defaultOpen = false, children }) {
  const [open, setOpen] = useState(defaultOpen);
  return (
    <section>
      <button onClick={() => setOpen(v => !v)}>Toggle</button>
      {open && children}
    </section>
  );
}
```

Do **not** copy a controlled `value` prop into local state without a specific synchronization design.

## 4. Prefer Clear Props Over Many Flags

Avoid contradictory booleans:

```jsx
<Alert success warning error /> // ❌ unclear
```

Prefer a single meaningful choice:

```jsx
<Alert variant="error" />
```

Likewise, use `<Button variant="danger" size="small" />` instead of a long collection of style flags.

## 5. Composition vs More Abstraction

A generic `Card` can contain `ApplicationDetails`, `DeveloperProfile`, or `ProductInfo`; avoid making one giant component with dozens of unrelated props.

A **compound component** groups related UI, for example `<Tabs><Tabs.List />...<Tabs.Panel /></Tabs>`, often coordinated through Context. Recognize the idea; implementation comes in Lesson 54.

React prefers **composition over inheritance**: build larger UIs from small pieces.

## Common Mistakes

- Overengineering generic components before there are real reusable needs.
- Using many boolean props that allow contradictory combinations.
- Passing every prop blindly to native DOM elements.
- Replacing semantic `<button>` elements with clickable `<div>`s.
- Omitting accessible names for icon-only controls.
- Exposing imperative ref methods when simple props are enough.

## Interview Quick Check

**What is composition?** Combining smaller components through `children` or JSX props.

**When use named slots?** When the component has several distinct regions such as header, sidebar, and body.

**Controlled vs uncontrolled?** Controlled state comes from the parent; uncontrolled state is managed internally.

**Hook vs component?** Use a Hook for reusable behavior; a component for reusable UI.

---

## Section 6 — Reusable Logic Complete

- Lesson 34: Custom Hooks reuse React logic.
- Lesson 35: Hook rules ensure correct execution.
- Lesson 36: Composition and clear props make UI reusable.

➡️ [Lesson 37 — React.memo ⭐⭐⭐⭐⭐](../07-performance/37-react-memo.md)
