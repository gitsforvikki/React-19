# Lesson 25 — DOM Refs and useImperativeHandle

Refs let React components perform necessary imperative operations.

Use them carefully.

Normal UI should still use:

```text
state
+
props
+
JSX
```

Refs are mainly for things like:

- focus
- scroll
- select text
- play or pause media
- measure DOM
- integrate DOM-based libraries

---

## 1. Basic DOM Ref

```jsx
function SearchInput() {
  const inputRef =
    useRef(null);

  function handleFocus() {
    inputRef.current
      ?.focus();
  }

  return (
    <>
      <input ref={inputRef} />

      <button
        onClick={handleFocus}
      >
        Focus
      </button>
    </>
  );
}
```

After commit:

```text
inputRef.current
→ DOM input element
```

---

## 2. Keep Normal UI Declarative

Avoid using refs to manually update UI:

```js
inputRef.current.value =
  "Vikash";
```

for normal application state.

Prefer:

```jsx
const [name, setName] =
  useState("");

<input
  value={name}
  onChange={(e) =>
    setName(e.target.value)
  }
/>
```

Rule:

> Use state for normal UI. Use refs for genuinely imperative operations.

---

## 3. Common DOM Ref Operations

### Focus

```js
inputRef.current?.focus();
```

### Scroll

```js
elementRef.current
  ?.scrollIntoView();
```

### Video

```js
videoRef.current?.play();
videoRef.current?.pause();
```

### Measure

```js
const rect =
  elementRef.current
    ?.getBoundingClientRect();
```

---

## 4. Timing Matters

This may fail:

```jsx
function handleOpen() {
  setOpen(true);
  inputRef.current?.focus();
}
```

The input may not exist in the DOM yet.

A common solution:

```jsx
useEffect(() => {
  if (open) {
    inputRef.current
      ?.focus();
  }
}, [open]);
```

The Effect runs after the DOM has been committed.

---

## 5. Refs in React 19 Components

In React 19, a function component can receive `ref` as a prop.

Example:

```jsx
function MyInput({
  ref,
}) {
  return (
    <input ref={ref} />
  );
}
```

Older React code often uses `forwardRef`, so you should still recognize it in existing projects.

---

## 6. What Is useImperativeHandle?

`useImperativeHandle` lets a child control what the parent receives through the ref.

Instead of exposing the whole DOM element, the child can expose only a small API.

Example:

```jsx
function SearchInput({
  ref,
}) {
  const inputRef =
    useRef(null);

  useImperativeHandle(
    ref,
    () => ({
      focus() {
        inputRef.current
          ?.focus();
      },
    }),
    []
  );

  return (
    <input ref={inputRef} />
  );
}
```

Parent:

```jsx
function Page() {
  const searchRef =
    useRef(null);

  return (
    <>
      <SearchInput
        ref={searchRef}
      />

      <button
        onClick={() =>
          searchRef.current
            ?.focus()
        }
      >
        Focus Search
      </button>
    </>
  );
}
```

The parent sees:

```text
searchRef.current.focus()
```

instead of the entire internal input element.

---

## 7. Why useImperativeHandle?

It improves encapsulation.

Without it:

```text
parent
→ full child DOM node
```

With it:

```text
parent
→ small child API
→ focus()
```

Expose only what the parent actually needs.

---

## 8. When useImperativeHandle Is Useful

Good examples:

- `focus()`
- `scrollToTop()`
- `select()`
- media controls
- limited third-party imperative methods

Avoid using it for normal app state such as:

```text
setUser()
setTheme()
setCart()
setLoggedIn()
```

Those should use normal React state and props.

---

## 9. Controlled API Is Often Better

Suppose a modal needs to open and close.

Imperative:

```js
modalRef.current.open();
```

Often better:

```jsx
<Modal
  open={open}
  onOpenChange={setOpen}
/>
```

Why?

Because modal visibility is normal UI state.

Use an imperative API only when the operation is truly imperative.

---

## 10. Callback Refs

React also allows a function as a ref:

```jsx
<div
  ref={(node) => {
    console.log(node);
  }}
/>
```

This is useful when you need logic when a node is attached or removed.

For simple DOM access, `useRef` is usually easier.

---

## 11. Dynamic Lists Need Multiple Refs

Do not use one ref for many elements if you need access to each one.

Bad:

```jsx
const itemRef =
  useRef(null);

items.map((item) => (
  <div
    key={item.id}
    ref={itemRef}
  />
));
```

If you need every DOM node, store them by ID:

```jsx
const itemRefs =
  useRef(new Map());
```

and use callback refs when needed.

This is an advanced pattern; use it only when direct DOM access is really necessary.

---

## 12. Third-Party Library Example

```jsx
function Chart({ data }) {
  const containerRef =
    useRef(null);

  useEffect(() => {
    const chart =
      createChart(
        containerRef.current,
        data
      );

    return () => {
      chart.destroy();
    };
  }, [data]);

  return (
    <div
      ref={containerRef}
    />
  );
}
```

Here:

```text
ref
→ gives DOM node

Effect
→ starts external library

cleanup
→ removes external resource
```

---

## Common Mistakes

### Mistake 1 — Using refs instead of state

Normal UI should stay declarative.

### Mistake 2 — Accessing DOM before it exists

The ref may still be `null`.

### Mistake 3 — Exposing the full DOM node when only one method is needed

Use `useImperativeHandle` for a smaller API.

### Mistake 4 — Using imperative APIs for normal parent-child data flow

Prefer props and state.

### Mistake 5 — Using one ref for many list elements

Use a ref collection only when required.

---

## Interview Questions

### What is a DOM ref?

A ref attached to a DOM element that gives imperative access to the browser node.

### When should you use a DOM ref?

For focus, scrolling, media control, measurement, selection, or DOM-based integrations.

### What is useImperativeHandle?

A Hook that controls what a child exposes to its parent through a ref.

### Why use useImperativeHandle?

To expose a small imperative API instead of the child's entire DOM implementation.

### Should refs replace props and state?

No. Props and state should remain the default for application data and UI.

### Does React 19 still require forwardRef?

No for new React 19 function components, but you should still recognize `forwardRef` in older code.

---

## Quick Revision

```text
Normal UI/data
→ state + props

Need DOM operation
→ ref

Parent needs only a small
imperative child API
→ useImperativeHandle
```

Main rule:

> Stay declarative by default. Use refs only when imperative behavior is genuinely needed.

---

## Section 3 — Effects and Refs Complete

You have covered:

- `useEffect`
- dependencies
- cleanup
- unnecessary Effects
- stale closures
- `useRef`
- DOM refs
- `useImperativeHandle`

---

## Next Lesson

➡️ [Lesson 26 — Forms in React](../04-forms/26-forms.md)
