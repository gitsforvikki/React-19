# Lesson 35 — Rules of Hooks ⭐⭐⭐⭐⭐

## Why Are There Rules?

React associates Hook state with a component's **Hook call order**. Ordinary Hooks must be called consistently on every render.

```text
Render 1: useState → useEffect → useContext
Render 2: useState → useEffect → useContext  ✅
```

## The Two Rules ⭐⭐⭐⭐⭐

1. **Call Hooks at the top level** of a function component or custom Hook.
2. **Call Hooks only from React function components or custom Hooks**—not ordinary functions.

### Incorrect — Conditional Hook

```jsx
function Profile({ enabled }) {
  if (enabled) {
    useEffect(() => subscribe(), []); // ❌
  }
}
```

### Correct — Condition Inside the Effect

```jsx
function Profile({ enabled }) {
  useEffect(() => {
    if (!enabled) return;
    return subscribe(); // subscribe() returns a cleanup function
  }, [enabled]);
}
```

## Other Common Violations

**Do not call ordinary Hooks** inside loops, `map`, conditions, event handlers, nested callbacks, `try/catch/finally`, or after a conditional early return.

Incorrect:

```jsx
function List({ items }) {
  return items.map(item => {
    const [open, setOpen] = useState(false); // ❌
    return <div key={item.id}>{item.title}</div>;
  });
}
```

Correct:

```jsx
function Item({ item }) {
  const [open, setOpen] = useState(false);
  return (
    <button onClick={() => setOpen(v => !v)}>
      {item.title} {open ? "▲" : "▼"}
    </button>
  );
}

function List({ items }) {
  return items.map(item => <Item key={item.id} item={item} />);
}
```

Each `Item` instance now has its own stable Hook calls.

Also put Hook calls **before** any conditional `return`, and render components with `<Child />` rather than calling `Child()` as a normal function.

## React 19 Exception — use()

React's special `use(resource)` API **can** be called inside conditions and loops, unlike ordinary Hooks. It still must be used in a component/Hook and **cannot** be called inside `try/catch`. This exception does not apply to `useState`, `useEffect`, `useContext`, or custom Hooks.

## ESLint and Debugging

Keep `eslint-plugin-react-hooks` enabled:

- `rules-of-hooks` checks where Hooks are called.
- `exhaustive-deps` checks dependencies of Hooks such as `useEffect`.

If you see an **Invalid Hook Call** warning, also consider mismatched React/renderer versions or duplicate React installations, not just Hook-rule violations.

## Interview Quick Check

**Why can't Hooks be conditional?** Changing call order prevents React from associating stored Hook state with the same call on future renders.

**Can I use Hooks inside `map`?** Not ordinary Hooks; extract a child component.

**Can I conditionally render a component that uses Hooks?** Yes. Each rendered component maintains its own Hook order.

**Are custom Hooks exempt?** No, they follow the same ordinary Hook rules.

---

➡️ [Lesson 36 — Reusable Component APIs and Composition Patterns](./36-reusable-component-patterns.md)
