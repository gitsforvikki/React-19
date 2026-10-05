# Lesson 61 — React 19 Ref Improvements

## 1. Why Refs Changed in React 19

Refs are React's imperative escape hatch.

Typical uses:

- focus an input
- scroll to an element
- measure DOM
- control media
- integrate with imperative libraries

React 19 simplifies ref handling and adds safer cleanup behavior.

The two major improvements are:

```text
1. ref as a prop
2. callback ref cleanup functions
```

---

## 2. React 18 Mental Model

Historically, `ref` was special.

For a custom function component:

```jsx
<MyInput ref={inputRef} />
```

React 18 commonly required:

```jsx
const MyInput =
  forwardRef(
    function MyInput(
      props,
      ref
    ) {
      return (
        <input
          {...props}
          ref={ref}
        />
      );
    }
  );
```

---

## 3. React 19: ref as a Prop ⭐⭐⭐⭐⭐

React 19 function components can receive `ref` directly as a prop:

```jsx
function MyInput({
  ref,
  ...props
}) {
  return (
    <input
      {...props}
      ref={ref}
    />
  );
}
```

Parent:

```jsx
function Form() {
  const inputRef =
    useRef(null);

  return (
    <MyInput
      ref={inputRef}
    />
  );
}
```

No `forwardRef` wrapper is required for new React 19 function-component code.

---

## 4. React 18 vs React 19 ⭐⭐⭐⭐⭐

```text
React <=18
──────────
Parent
  │ ref
  ↓
forwardRef(...)
  │
  ↓
Function Component
  │
  ↓
DOM node


React 19
────────
Parent
  │ ref
  ↓
Function Component
receives ref as prop
  │
  ↓
DOM node
```

This is the main interview distinction.

---

## 5. Is forwardRef Removed? ⭐⭐⭐⭐⭐

No.

Do not say:

```text
React 19 removed forwardRef
```

The correct statement is:

> In React 19, `forwardRef` is no longer necessary for new function components because `ref` can be passed as a prop. React documents `forwardRef` as an API planned for future deprecation.

You still need to recognize it in:

- React 18 code
- existing libraries
- applications supporting older React versions

---

## 6. Class Component Exception

Refs passed to class components still refer to the component instance.

They are not treated as ordinary props in the same way as refs passed to function components.

Modern application code generally favors function components, but this distinction matters when maintaining class-based code.

---

## 7. Passing the Ref Through

```jsx
function SearchInput({
  ref,
  label,
  ...props
}) {
  return (
    <label>
      {label}

      <input
        ref={ref}
        {...props}
      />
    </label>
  );
}
```

Parent:

```jsx
const searchRef =
  useRef(null);

<SearchInput
  ref={searchRef}
  label="Search"
/>
```

Now:

```jsx
searchRef.current
  ?.focus();
```

focuses the underlying input.

---

## 8. Ref Exposure Is Public API

When a component forwards/passes a ref to its DOM node:

```text
consumer
   ↓
gets access to
   ↓
DOM node
```

This is an API design decision.

Do not expose internal DOM nodes without considering encapsulation.

---

## 9. useImperativeHandle Still Matters ⭐⭐⭐⭐⭐

React 19's ref-as-prop improvement does not make `useImperativeHandle` obsolete.

Use it when you want to expose a restricted custom handle:

```jsx
function SearchInput({
  ref,
  ...props
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

      select() {
        inputRef.current
          ?.select();
      },
    }),
    []
  );

  return (
    <input
      ref={inputRef}
      {...props}
    />
  );
}
```

Parent sees:

```text
{
  focus(),
  select()
}
```

instead of the entire DOM node.

---

## 10. Declarative State Still Comes First

Do not turn ordinary state into an imperative ref API.

Prefer:

```jsx
<Modal
  open={isOpen}
  onOpenChange={setIsOpen}
/>
```

over:

```jsx
modalRef.current.open();
modalRef.current.close();
```

when `open` is ordinary application state.

Refs are best for naturally imperative operations.

---

# Part 2 — Callback Ref Cleanup

## 11. Callback Ref Basics

A callback ref receives a node when React attaches it:

```jsx
<input
  ref={(node) => {
    inputNode = node;
  }}
/>
```

Historically React also used `null` calls to indicate detachment.

React 19 adds an explicit cleanup-return mechanism.

---

## 12. Callback Ref Cleanup ⭐⭐⭐⭐⭐

React 19 lets a callback ref return a cleanup function:

```jsx
<div
  ref={(node) => {
    register(node);

    return () => {
      unregister(node);
    };
  }}
/>
```

Mental model:

```text
node attached
     ↓
callback ref runs
     ↓
setup/register
     ↓
returns cleanup
     ↓
node detached
     ↓
cleanup runs
```

This makes setup and teardown symmetrical.

---

## 13. Cleanup and null Behavior ⭐⭐⭐⭐⭐

If a callback ref returns a cleanup function, React uses that cleanup when the ref is detached.

React does not additionally need to call that ref callback with `null` for the same cleanup path.

For older callback refs that do not return cleanup, existing null-based patterns remain relevant for compatibility.

Future React versions may further move away from null-based cleanup behavior.

---

## 14. Avoid Implicit Callback Ref Returns ⭐⭐⭐⭐⭐

Problem:

```jsx
<div
  ref={node =>
    (instance = node)
  }
/>
```

That expression returns the assigned node.

With React 19 cleanup semantics, TypeScript rejects non-cleanup return values because a returned function now has meaning.

Use:

```jsx
<div
  ref={(node) => {
    instance = node;
  }}
/>
```

Block bodies make it clear that nothing is returned.

---

## 15. Dynamic Collection Refs

Callback cleanup works nicely with a Map:

```jsx
const nodeMap =
  useRef(new Map());

{items.map(item => (
  <div
    key={item.id}
    ref={(node) => {
      nodeMap.current.set(
        item.id,
        node
      );

      return () => {
        nodeMap.current.delete(
          item.id
        );
      };
    }}
  >
    {item.name}
  </div>
))}
```

This is useful for:

- scrolling to a specific item
- focus management
- measuring list items

---

## 16. Callback Ref Cleanup Also Works Beyond DOM Nodes

The cleanup model applies to callback refs generally, including refs associated with class component instances and custom imperative handles.

The important concept is:

```text
ref attachment can have
explicit symmetric cleanup
```

---

# Part 3 — TypeScript Changes

## 17. useRef Requires an Argument in React 19 Types ⭐⭐⭐⭐⭐

React 19 TypeScript definitions require an initial argument.

Avoid:

```tsx
const ref =
  useRef<HTMLInputElement>();
```

Use an intentional initial value:

```tsx
const ref =
  useRef<HTMLInputElement>(
    null
  );
```

or where appropriate:

```tsx
const ref =
  useRef<number | undefined>(
    undefined
  );
```

This makes the initial ref state explicit.

---

## 18. Ref Objects Are Mutable

React 19 TypeScript ref object types were simplified around mutability.

Conceptually:

```tsx
const timerRef =
  useRef<number | null>(
    null
  );

timerRef.current = 10;
```

A ref object is designed as a mutable container.

Remember: mutating it does not trigger rendering.

---

## 19. Typing ref as a Prop

Conceptually:

```tsx
type Props = {
  ref?:
    React.Ref<HTMLInputElement>;
  placeholder?: string;
};

function MyInput({
  ref,
  ...props
}: Props) {
  return (
    <input
      ref={ref}
      {...props}
    />
  );
}
```

In production, combine the ref type with appropriate intrinsic element props instead of manually re-declaring every input attribute.

---

## 20. element.ref Deprecation

React 19 treats `ref` as a regular prop on React elements.

Code that introspects:

```js
element.ref
```

is deprecated.

Use:

```js
element.props.ref
```

when library-level element introspection is genuinely necessary.

Most application components should not inspect React element internals at all.

---

# Part 4 — Ref Lifecycle

## 21. Ref Timing

Refs are connected during the commit process.

```text
Render
  ↓
Reconciliation
  ↓
Commit
  ├── DOM updated
  └── refs attached/updated
```

Do not expect a newly mounted DOM node to already exist in the ref during render.

---

## 22. Conditional Node

```jsx
{editing && (
  <input
    ref={inputRef}
  />
)}
```

When the input is absent:

```js
inputRef.current
// null
```

Always account for mount/unmount lifecycle.

---

## 23. Strict Mode

Development Strict Mode can expose missing ref cleanup by running extra setup/cleanup checks.

Your callback ref logic should therefore be symmetrical and safe to repeat.

Do not "fix" Strict Mode by hiding cleanup bugs.

---

# Part 5 — Choosing the Right Ref Pattern

## 24. Object Ref

Use:

```jsx
const ref =
  useRef(null);
```

when you primarily need a stable mutable reference.

Example:

```jsx
<input ref={ref} />
```

---

## 25. Callback Ref

Use a callback ref when attachment itself needs custom logic:

```jsx
ref={(node) => {
  register(node);

  return () => {
    unregister(node);
  };
}}
```

Useful for dynamic node registration and lifecycle integration.

---

## 26. Custom Imperative Handle

Use:

```jsx
useImperativeHandle(
  ref,
  () => ({
    focus() {},
  }),
  []
);
```

when the parent needs a restricted imperative API instead of full internal DOM access.

---

## 27. Decision Guide ⭐⭐⭐⭐⭐

```text
Need rendered/visible state?
→ useState

Need non-rendering mutable value?
→ useRef

Need DOM node?
→ DOM ref

Need custom attach/detach behavior?
→ callback ref

Need limited imperative child API?
→ useImperativeHandle

Need ref through React 19 function component?
→ receive ref as prop

Maintaining React 18-compatible code?
→ forwardRef may still be required
```

---

## 28. CareerLoop Example

Reusable field:

```jsx
function TextField({
  ref,
  label,
  ...props
}) {
  return (
    <label>
      {label}
      <input
        ref={ref}
        {...props}
      />
    </label>
  );
}
```

On validation failure:

```jsx
companyRef.current
  ?.focus();
```

No `forwardRef` is necessary in a React 19-only function-component codebase.

---

## 29. CodeBuddy Example

Dynamic conversation items:

```text
conversationId
     ↓
Map<id, DOM node>
     ↓
callback ref registration
     ↓
scroll/focus selected chat
```

Callback cleanup removes stale nodes when conversations unmount.

---

## 30. Common Mistakes ⭐⭐⭐⭐⭐

1. Saying React 19 removed `forwardRef`.
2. Using `forwardRef` by default in new React 19-only function components.
3. Assuming class component refs become ordinary props.
4. Returning an assigned DOM node from a callback ref.
5. Forgetting callback-ref cleanup.
6. Expecting ref mutation to trigger rendering.
7. Using refs for visible application state.
8. Exposing internal DOM nodes unnecessarily.
9. Reading newly attached DOM refs during render.
10. Calling `useRef()` with no argument in React 19 TypeScript.
11. Depending on deprecated `element.ref`.
12. Using imperative APIs where props/state are clearer.

---

## 31. Interview Questions ⭐⭐⭐⭐⭐

### What is the biggest ref change in React 19?

Function components can receive `ref` directly as a prop.

### Is forwardRef removed?

No. It is no longer necessary for new React 19 function components and is planned for future deprecation.

### Are class component refs passed as normal props?

No. They still refer to the class instance.

### What changed for callback refs?

They may return cleanup functions.

### What happens if a callback ref returns cleanup?

React invokes that cleanup when the ref is detached instead of needing the old null callback for that cleanup path.

### Why avoid implicit assignment returns?

The returned node can be confused with the new cleanup-function return contract and is rejected by React 19 TypeScript types.

### Does useImperativeHandle still matter?

Yes. It controls the imperative API exposed through a ref.

### What TypeScript change affects useRef?

React 19 types require an initial argument.

### What happened to element.ref?

It is deprecated in favor of `element.props.ref`.

---

## 32. Interview-Ready Answer ⭐⭐⭐⭐⭐

```text
React 19 simplifies refs by allowing function
components to receive ref directly as a prop, so new
React 19-only function components no longer need
forwardRef.

forwardRef has not been removed; it remains important
for older React code and compatibility.

React 19 also lets callback refs return cleanup
functions, which gives ref setup and teardown a
symmetric lifecycle. Because a callback return value
now has cleanup meaning, I avoid implicit assignment
returns.

I still treat refs as an imperative escape hatch:
normal UI state should remain declarative, and I use
useImperativeHandle when I want to expose a restricted
imperative API instead of an entire DOM node.
```

---

## 33. Complete Mental Model

```text
                 REACT 19 REFS
                       │
          ┌────────────┴────────────┐
          ↓                         ↓
      ref as prop             callback cleanup
          │                         │
 Parent → Component           attach node
          │                         ↓
          ↓                    setup/register
       DOM node                    │
          │                         ↓
          └─ or ───────────── cleanup on detach
             ↓
      useImperativeHandle
             ↓
        custom API


Legacy compatibility
→ forwardRef still exists
```

---

## 34. Key Takeaways

- React 19 function components can receive `ref` as a prop.
- New React 19-only function components generally do not need `forwardRef`.
- `forwardRef` has not been removed.
- Class component refs remain special instance refs.
- Callback refs can return cleanup functions.
- Explicit cleanup makes attachment/detachment symmetrical.
- Avoid implicit assignment returns from callback refs.
- React 19 TypeScript requires an argument for `useRef`.
- `element.ref` is deprecated in favor of `element.props.ref`.
- `useImperativeHandle` remains useful for controlled imperative APIs.
- Refs are attached during commit.
- Ref mutation does not cause rendering.
- Prefer declarative props/state for ordinary application state.

---

## Next Lesson

➡️ [Lesson 62 — Document Metadata, Stylesheets and Resource Preloading](./62-metadata-resources.md)
