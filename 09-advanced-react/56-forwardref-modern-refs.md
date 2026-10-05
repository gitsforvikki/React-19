# Lesson 56 — forwardRef and Modern Ref Handling

## 1. Why This Lesson Matters

Refs let a parent access an imperative capability such as:

- focusing an input
- selecting text
- scrolling to an element
- controlling media
- integrating with a DOM/third-party API

Historically, passing a ref through a function component required `forwardRef`.

**React 19 changes this model.**

For new React 19 function components:

> `ref` can be received as a prop.

So this lesson teaches:

1. the modern React 19 approach first,
2. `forwardRef` because you will see it in React 18/older code and libraries,
3. how to design safe ref APIs.

---

## 2. Quick Ref Recap

```jsx
const inputRef =
  useRef(null);

<input ref={inputRef} />
```

Later:

```jsx
inputRef.current?.focus();
```

A ref is an imperative escape hatch.

It should not replace normal props/state data flow.

---

# Part 1 — Modern React 19 Ref Handling

## 3. ref as a Prop ⭐⭐⭐⭐⭐

In React 19, a function component can receive `ref` as a prop.

```jsx
function MyInput({
  ref,
  ...props
}) {
  return (
    <input
      ref={ref}
      {...props}
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
    <>
      <MyInput
        ref={inputRef}
      />

      <button
        onClick={() =>
          inputRef.current
            ?.focus()
        }
      >
        Focus
      </button>
    </>
  );
}
```

No `forwardRef` is required for this new React 19 function-component pattern.

---

## 4. Mental Model

```text
Parent
  │
  │ ref
  ↓
MyInput
  │
  │ ref
  ↓
<input>
  │
  ↓
DOM node
```

The custom component chooses whether and where to pass the ref.

---

## 5. Why This Is Simpler

Older pattern:

```jsx
const MyInput =
  forwardRef(
    function MyInput(
      props,
      ref
    ) {
      return (
        <input
          ref={ref}
          {...props}
        />
      );
    }
  );
```

React 19:

```jsx
function MyInput({
  ref,
  ...props
}) {
  return (
    <input
      ref={ref}
      {...props}
    />
  );
}
```

This removes an extra wrapper API from common new code.

---

## 6. Ref Exposure Is an API Decision ⭐⭐⭐⭐⭐

When you write:

```jsx
function MyInput({
  ref,
  ...props
}) {
  return (
    <input
      ref={ref}
      {...props}
    />
  );
}
```

you are exposing the underlying input DOM node.

Consumers can now call DOM methods through it.

That is part of your component's public API.

Do not expose DOM nodes automatically just because it is possible.

---

# Part 2 — useImperativeHandle

## 7. Restricting the Imperative API ⭐⭐⭐⭐⭐

Instead of exposing the entire DOM node, expose selected methods.

```jsx
function MyInput({
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

Parent:

```jsx
const ref =
  useRef(null);

<MyInput ref={ref} />

<button
  onClick={() =>
    ref.current?.focus()
  }
>
  Focus
</button>
```

Now the parent receives a custom handle instead of unrestricted DOM access.

---

## 8. Encapsulation

Without a custom handle:

```text
Parent
→ complete input DOM node
→ many imperative operations
```

With `useImperativeHandle`:

```text
Parent
→ {
    focus(),
    select()
  }
```

This gives the component control over its imperative contract.

---

## 9. Do Not Use Imperative APIs for Normal State

Bad design:

```jsx
ref.current.open();
ref.current.close();
```

when the component naturally supports:

```jsx
<Modal
  open={isOpen}
  onOpenChange={setIsOpen}
/>
```

Prefer declarative props/state for normal UI state.

Use imperative methods for actions that are naturally imperative:

- focus
- scroll
- selection
- media methods
- integration with imperative external APIs

---

## 10. Controlled Inputs and Imperative Mutation

Be careful with a method like:

```js
clear() {
  inputRef.current.value = "";
}
```

If the input is controlled:

```jsx
<input
  value={value}
  onChange={...}
/>
```

React state is the source of truth.

Directly changing the DOM value can conflict with that state.

Prefer clearing through the state/callback API.

---

## 11. useImperativeHandle Dependencies

Syntax:

```jsx
useImperativeHandle(
  ref,
  createHandle,
  dependencies
);
```

If the handle uses reactive values, include them correctly.

```jsx
useImperativeHandle(
  ref,
  () => ({
    report() {
      console.log(value);
    },
  }),
  [value]
);
```

Do not omit dependencies merely to keep handle identity stable.

Correctness comes first.

---

# Part 3 — forwardRef

## 12. What Is forwardRef? ⭐⭐⭐⭐⭐

Historically, function components could not receive the special `ref` attribute like ordinary props.

`forwardRef` let a component receive a ref and forward it.

```jsx
import {
  forwardRef,
} from "react";

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

This is essential knowledge for React 18 and older codebases.

---

## 13. React 19 Status of forwardRef ⭐⭐⭐⭐⭐

In React 19, function components can receive `ref` as a prop.

Therefore `forwardRef` is no longer necessary for new function-component code using the modern model.

React documentation describes `forwardRef` as no longer necessary in React 19 and planned for future deprecation.

Important interview wording:

```text
React <= 18:
forwardRef commonly required

React 19:
ref can be passed as a prop
forwardRef is legacy-compatible knowledge
```

Do not say that `forwardRef` has already disappeared from React—it still exists.

---

## 14. Why You Still Need to Understand forwardRef

You will encounter it in:

- existing React applications
- component libraries
- code written for React 18
- tutorials
- interview questions
- libraries supporting multiple React versions

Knowing modern React does not mean ignoring existing production code.

---

## 15. Legacy forwardRef + useImperativeHandle

Older style:

```jsx
const MyInput =
  forwardRef(
    function MyInput(
      props,
      ref
    ) {
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
        <input
          ref={inputRef}
          {...props}
        />
      );
    }
  );
```

Modern React 19 code can receive `ref` directly instead.

---

# Part 4 — Callback Refs

## 16. Callback Refs

A ref can also be a function:

```jsx
<div
  ref={(node) => {
    // node when attached
    // null when detached,
    // unless using cleanup-return form
  }}
/>
```

Callback refs are useful when you need custom behavior when nodes attach/detach.

---

## 17. React 19 Callback Ref Cleanup ⭐⭐⭐⭐⭐

React 19 supports returning a cleanup function from a callback ref.

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

This gives callback refs an explicit cleanup mechanism.

---

## 18. Avoid Implicit Returns in Ref Callbacks

Avoid:

```jsx
<div
  ref={node =>
    (instance = node)
  }
/>
```

In React 19, a returned value from a callback ref has cleanup semantics, and this expression returns the assigned node.

Prefer a block:

```jsx
<div
  ref={(node) => {
    instance = node;
  }}
/>
```

This is also clearer for TypeScript.

---

## 19. Dynamic Lists of Refs

You cannot call `useRef` inside a loop.

For dynamic nodes, a Map stored in one ref can be useful:

```jsx
const itemRefs =
  useRef(new Map());
```

Then callback refs can register nodes:

```jsx
{items.map(item => (
  <div
    key={item.id}
    ref={(node) => {
      if (node) {
        itemRefs.current.set(
          item.id,
          node
        );
      }

      return () => {
        itemRefs.current.delete(
          item.id
        );
      };
    }}
  >
    {item.name}
  </div>
))}
```

This is useful for focus/scroll behavior across dynamic collections.

---

# Part 5 — DOM Ref Timing and Measurement

## 20. When React Sets the Ref ⭐⭐⭐⭐⭐

Conceptually:

```text
Render phase
→ determine UI

Commit phase
→ update DOM
→ attach/update refs
```

Do not expect the final DOM ref during rendering.

Use it from event handlers or appropriate Effects after commit.

---

## 21. Conditional DOM and null

```jsx
{show && (
  <input ref={inputRef} />
)}
```

When `show` is false:

```js
inputRef.current
// null
```

Always handle the possibility that a DOM node is not mounted.

---

## 22. Focus

```jsx
function handleEdit() {
  inputRef.current
    ?.focus();
}
```

This is a natural ref use because focus is inherently imperative.

---

## 23. Scroll

```jsx
function scrollToError() {
  errorRef.current
    ?.scrollIntoView({
      behavior: "smooth",
      block: "center",
    });
}
```

Again, this is an imperative DOM action.

---

## 24. Text Selection

```jsx
inputRef.current
  ?.select();
```

A component may expose this safely through a custom imperative handle.

---

## 25. Media Control

```jsx
videoRef.current
  ?.play();

videoRef.current
  ?.pause();
```

Browser media APIs are another appropriate imperative use case.

Remember that `play()` can return a Promise and may be rejected by browser autoplay policies.

---

## 26. Measuring DOM with useLayoutEffect

Sometimes you need layout information:

```jsx
useLayoutEffect(() => {
  const rect =
    ref.current
      ?.getBoundingClientRect();

  // measure before browser paint
}, []);
```

Use `useLayoutEffect` when the measurement must affect layout before paint.

Do not use it for ordinary logic because it can block painting.

---

## 27. Do Not Mutate React-Owned DOM Arbitrarily ⭐⭐⭐⭐⭐

Dangerous pattern:

```js
containerRef.current
  .innerHTML = "...";
```

React believes it owns that DOM subtree.

Manual mutations can conflict with reconciliation.

Refs are appropriate for imperative operations that do not fight React's ownership model.

---

# Part 6 — Ref Design Decisions

## 28. Props vs Ref ⭐⭐⭐⭐⭐

Ask:

```text
Can this behavior be expressed declaratively?
```

If yes:

```jsx
<Modal open={isOpen} />
```

prefer props/state.

If the action is naturally imperative:

```jsx
inputRef.current?.focus();
```

a ref may be appropriate.

---

## 29. Expose DOM Node vs Custom Handle

Expose DOM directly when consumers genuinely need normal DOM capabilities:

```jsx
<MyInput ref={inputRef} />
```

Expose a custom handle when you want stronger encapsulation:

```text
{
  focus(),
  select()
}
```

Choose intentionally because ref behavior is part of the public component API.

---

## 30. Ref vs State

```text
State
→ visible/rendered information
→ changing it causes rendering

Ref
→ mutable imperative information
→ changing current does not render
```

Never store visible UI state only in a ref.

---

## 31. Ref vs Event Callback

Sometimes the parent does not need a ref at all.

Instead of:

```text
parent reaches into child
→ commands internal behavior
```

the child can expose normal callbacks/props.

Prefer normal React data flow whenever it models the requirement clearly.

---

## 32. Ref Handling in Reusable Libraries

When designing a reusable component, changing what a ref points to can be a breaking API change.

Example:

```text
version 1
ref → <input>

version 2
ref → wrapper <div>
```

Consumers using `focus()` may break.

Treat ref semantics as part of your public API.

---

## 33. HOCs and React 19 Refs

Legacy HOCs often needed explicit ref forwarding.

In React 19, new function-component APIs can accept `ref` as a prop.

But when maintaining libraries that support React 18 and 19 simultaneously, compatibility requirements may still determine the implementation.

Do not mechanically modernize a shared library without checking its supported React versions.

---

## 34. TypeScript Concept

In React 19 TypeScript code, component props can include a ref type.

Conceptually:

```tsx
function MyInput({
  ref,
  ...props
}: {
  ref?: React.Ref<HTMLInputElement>;
}) {
  return (
    <input
      ref={ref}
      {...props}
    />
  );
}
```

In real reusable components, combine this with the appropriate intrinsic input prop types rather than manually recreating every HTML input prop.

The key concept is that `ref` is now part of the function component's props contract.

---

## 35. CareerLoop Example

After validation, you may want to focus the first invalid field:

```jsx
const companyRef =
  useRef(null);

function handleInvalid() {
  companyRef.current
    ?.focus();
}
```

A reusable `TextField` in React 19 can accept and pass its `ref` prop to the underlying input.

---

## 36. CodeBuddy Example

A chat UI may expose:

```text
MessageComposer handle
├── focus()
└── select()
```

The parent can focus the composer after selecting a conversation without receiving unrestricted access to every internal DOM node.

---

## 37. Common Mistakes ⭐⭐⭐⭐⭐

1. Teaching `forwardRef` as mandatory for new React 19 function components.
2. Claiming `forwardRef` has already been removed.
3. Exposing DOM nodes without considering the public API.
4. Using refs instead of state for visible UI.
5. Using imperative open/close when declarative props are clearer.
6. Directly mutating controlled input values.
7. Reading DOM refs during render.
8. Forgetting that refs can be `null`.
9. Mutating React-managed DOM in conflicting ways.
10. Omitting reactive dependencies from `useImperativeHandle`.
11. Returning accidental values from callback refs in React 19.
12. Calling Hooks such as `useRef` inside loops.
13. Ignoring React-version compatibility in libraries.

---

## 38. Interview Questions ⭐⭐⭐⭐⭐

### What is forwardRef?

An API historically used to let a function component receive a ref and forward it to another node/component.

### Is forwardRef required in React 19?

No. React 19 function components can receive `ref` as a prop.

### Has forwardRef been removed?

No. It still exists, but React 19 documentation indicates it is no longer necessary for new function-component code and is planned for future deprecation.

### What is useImperativeHandle?

A Hook that customizes the value exposed through a ref.

### Why use a custom imperative handle?

To expose a small intentional API such as `focus()` rather than the entire DOM node.

### When should refs be used?

For imperative operations such as focus, selection, scrolling, measurement, media control, or external imperative APIs.

### Should refs replace props/state?

No.

### When are DOM refs attached?

During the commit process, after the DOM node exists.

### What changed for callback refs in React 19?

A callback ref can return a cleanup function.

### Why avoid implicit assignment returns in callback refs?

React 19 treats returned functions as cleanup; block-bodied callbacks avoid accidental return values and TypeScript ambiguity.

### Ref vs state?

State drives rendering; changing `ref.current` does not trigger rendering.

---

## 39. React 18 vs React 19 ⭐⭐⭐⭐⭐

```text
React 18 and earlier
────────────────────
Parent
  ↓ ref
forwardRef(...)
  ↓
function component
  ↓
DOM node


React 19
────────
Parent
  ↓ ref
function component
  receives ref as prop
  ↓
DOM node
```

This is the most important version distinction to remember.

---

## 40. Interview-Ready Answer ⭐⭐⭐⭐⭐

```text
Historically, forwardRef was used when a parent needed
to pass a ref through a function component to a DOM
node or expose an imperative handle.

In React 19, ref can be received directly as a prop by
function components, so forwardRef is no longer
necessary for new React 19 component code, although it
still matters for older code and compatibility.

I use refs only for imperative behavior such as focus,
scrolling, selection, measurement, or integration with
imperative APIs. If I don't want to expose the complete
DOM node, I use useImperativeHandle to expose a small
intentional API.

For ordinary UI state, I prefer declarative props and
state rather than imperative refs.
```

---

## 41. Complete Mental Model

```text
                 Parent
                   │
                   │ ref
                   ↓
        React 19 Function Component
                   │
          ┌────────┴─────────┐
          ↓                  ↓
   forward directly    useImperativeHandle
          │                  │
          ↓                  ↓
      DOM node         custom handle
                       {
                         focus(),
                         select()
                       }


Legacy React <=18
Parent
  ↓
forwardRef
  ↓
function component
  ↓
DOM/custom handle
```

---

## 42. Key Takeaways

- Refs are imperative escape hatches.
- React 19 function components can receive `ref` as a prop.
- New React 19 code generally does not need `forwardRef`.
- `forwardRef` remains important for React 18/legacy/library code.
- It has not yet disappeared from React.
- `useImperativeHandle` exposes a controlled imperative API.
- Prefer declarative props/state for ordinary UI behavior.
- Do not directly mutate controlled values behind React's state.
- Callback refs can return cleanup functions in React 19.
- Avoid accidental implicit callback-ref returns.
- Refs are attached during commit, not available as final DOM nodes during render.
- DOM refs may be `null`.
- Treat ref behavior as part of reusable component API design.
- Use `useLayoutEffect` only when layout measurement must occur before paint.

---

# Section 9 — Advanced React Completed ✅

Completed lessons:

- Lesson 51 — Portals
- Lesson 52 — Error Boundaries ⭐⭐⭐⭐⭐
- Lesson 53 — Suspense ⭐⭐⭐⭐⭐
- Lesson 54 — Compound Components Pattern
- Lesson 55 — Higher-Order Components and Render Props
- Lesson 56 — forwardRef and Modern Ref Handling

---

## Next Section — React 19

➡️ [Lesson 57 — What Changed in React 19 ⭐⭐⭐⭐⭐](../10-react-19/57-react-19-overview.md)
