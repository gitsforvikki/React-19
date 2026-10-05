# Lesson 25 — DOM Refs and useImperativeHandle

## 1. Where This Lesson Fits

In Lesson 24, you learned that `useRef` can:

- preserve mutable values without causing re-renders
- hold references to DOM elements

This lesson goes deeper into the DOM-ref side.

You will learn:

- how React attaches refs to DOM elements
- when direct DOM access is appropriate
- focusing, scrolling, and measuring elements
- callback refs
- managing refs for dynamic lists
- exposing a child DOM node to a parent
- React 19's modern `ref` handling
- `useImperativeHandle`
- exposing a limited imperative API instead of the entire DOM node
- when **not** to use imperative refs

The main principle is:

> **Use refs as an escape hatch for imperative behavior. Keep normal application data and UI flow declarative.**

---

# 2. Declarative React vs Imperative DOM

React is primarily declarative.

Instead of manually saying:

```js
element.textContent = "Loading...";
element.classList.add("active");
```

you normally describe the desired UI:

```jsx
function Status({
  loading,
}) {
  return (
    <p
      className={
        loading ? "active" : ""
      }
    >
      {loading
        ? "Loading..."
        : "Ready"}
    </p>
  );
}
```

Flow:

```text
state / props
     ↓
render
     ↓
JSX
     ↓
React updates DOM
```

This should remain your default approach.

---

# 3. When Direct DOM Access Is Appropriate

Some operations are inherently imperative.

Examples:

```text
focus an input
scroll an element
play/pause video
measure DOM geometry
select text
integrate DOM-based library
control browser-native imperative API
```

For these operations:

```text
React ref
   ↓
DOM node
   ↓
imperative method
```

is appropriate.

---

# 4. Basic DOM Ref ⭐⭐⭐⭐⭐

```jsx
import { useRef } from "react";

function SearchBox() {
  const inputRef =
    useRef(null);

  function handleFocus() {
    inputRef.current?.focus();
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

After React commits the input:

```text
inputRef.current
        ↓
HTMLInputElement
```

The parent JavaScript code can now call DOM methods such as:

```js
inputRef.current.focus();
```

---

# 5. DOM Ref Lifecycle ⭐⭐⭐⭐⭐

Initially:

```text
inputRef.current = null
```

React renders:

```jsx
<input ref={inputRef} />
```

During commit:

```text
React attaches DOM node
        ↓
inputRef.current = inputElement
```

When the node is removed:

```text
React detaches ref
        ↓
inputRef.current = null
```

Diagram:

```text
Render
  ↓
Reconciliation
  ↓
Commit
  ↓
DOM node attached
  ↓
ref populated
```

This is why DOM refs should not generally be expected to contain their node during rendering.

---

# 6. Focus Example

```jsx
function LoginForm() {
  const emailRef =
    useRef(null);

  function handleInvalidLogin() {
    emailRef.current?.focus();
  }

  return (
    <>
      <input
        ref={emailRef}
        type="email"
      />

      <button
        onClick={
          handleInvalidLogin
        }
      >
        Login
      </button>
    </>
  );
}
```

Direct focus is a good ref use case because focus is an imperative browser action.

---

# 7. Scrolling to an Element

```jsx
function Chat() {
  const bottomRef =
    useRef(null);

  function scrollToBottom() {
    bottomRef.current
      ?.scrollIntoView({
        behavior: "smooth",
      });
  }

  return (
    <>
      <div>
        {/* messages */}

        <div ref={bottomRef} />
      </div>

      <button
        onClick={scrollToBottom}
      >
        Latest message
      </button>
    </>
  );
}
```

Here React renders the content declaratively, while the ref performs one imperative action:

```text
scrollIntoView()
```

---

# 8. Controlling Media

```jsx
function VideoPlayer() {
  const videoRef =
    useRef(null);

  function handlePlay() {
    videoRef.current?.play();
  }

  function handlePause() {
    videoRef.current?.pause();
  }

  return (
    <>
      <video
        ref={videoRef}
        src="/demo.mp4"
      />

      <button
        onClick={handlePlay}
      >
        Play
      </button>

      <button
        onClick={handlePause}
      >
        Pause
      </button>
    </>
  );
}
```

Methods such as:

```text
play()
pause()
focus()
scrollIntoView()
```

are imperative browser APIs.

---

# 9. Measuring a DOM Element

```jsx
function Card() {
  const cardRef =
    useRef(null);

  useEffect(() => {
    const rect =
      cardRef.current
        ?.getBoundingClientRect();

    console.log(rect);
  }, []);

  return (
    <div ref={cardRef}>
      Developer Card
    </div>
  );
}
```

The DOM must exist before it can be measured.

Therefore measurement happens after React commits the node.

---

# 10. useEffect vs useLayoutEffect for DOM Work

For many DOM integrations:

```text
useEffect
```

is enough.

But if you must:

1. measure layout
2. immediately update something based on that measurement
3. avoid the user seeing an intermediate visual position

then:

```text
useLayoutEffect
```

may be required.

Conceptually:

```text
useLayoutEffect
      ↓
runs after DOM commit
but before browser paint
```

while:

```text
useEffect
      ↓
generally lets browser paint first
```

Use `useLayoutEffect` only when visual timing actually requires it.

---

# 11. Do Not Manipulate React-Owned DOM Unnecessarily ⭐⭐⭐⭐⭐

Avoid patterns such as:

```jsx
function Modal() {
  const modalRef =
    useRef(null);

  function hide() {
    modalRef.current.style.display =
      "none";
  }

  return (
    <div ref={modalRef}>
      Modal
    </div>
  );
}
```

If visibility is application state, prefer:

```jsx
function Modal() {
  const [open, setOpen] =
    useState(true);

  return (
    <>
      {open && (
        <div>Modal</div>
      )}

      <button
        onClick={() =>
          setOpen(false)
        }
      >
        Close
      </button>
    </>
  );
}
```

Rule:

```text
UI structure/style based on app state
→ React state + JSX

imperative browser action
→ ref
```

---

# 12. Refs to Custom Components ⭐⭐⭐⭐⭐

Suppose you create:

```jsx
function SearchInput(props) {
  return <input {...props} />;
}
```

A parent may want imperative access:

```jsx
function Page() {
  const inputRef =
    useRef(null);

  return (
    <SearchInput
      ref={inputRef}
    />
  );
}
```

In modern React 19, `ref` can be received as a prop by function components.

Example:

```jsx
function SearchInput({
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

This is an important React 19 improvement.

---

# 13. React 19 Ref as a Prop ⭐⭐⭐⭐⭐

React 19 allows function components to access `ref` as a prop.

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

Usage:

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

Then:

```text
inputRef.current
       ↓
underlying input DOM node
```

---

# 14. What About forwardRef? ⭐⭐⭐⭐⭐

Historically, function components used:

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
          ref={ref}
          {...props}
        />
      );
    }
  );
```

This pattern is still important to recognize when working with existing React code.

However, in React 19:

> **Function components can receive `ref` as a prop, so new React 19 code generally does not need `forwardRef` for this purpose.**

You will revisit this distinction in Lesson 56.

---

# 15. Why Exposing the Entire DOM Node Can Be Too Powerful

Suppose:

```jsx
function SearchInput({
  ref,
}) {
  return (
    <input ref={ref} />
  );
}
```

The parent receives the entire input DOM element.

It can now call:

```js
ref.current.focus();
ref.current.blur();
ref.current.remove();
ref.current.style.display =
  "none";
ref.current.value =
  "something";
```

But perhaps the component should only allow:

```text
focus()
```

This is where `useImperativeHandle` becomes useful.

---

# 16. What Is useImperativeHandle? ⭐⭐⭐⭐⭐

`useImperativeHandle` lets a component customize the value exposed through its ref.

Instead of exposing:

```text
entire DOM node
```

you can expose:

```text
small controlled imperative API
```

Import:

```jsx
import {
  useImperativeHandle,
  useRef,
} from "react";
```

---

# 17. Basic useImperativeHandle Example ⭐⭐⭐⭐⭐

```jsx
function MyInput({
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
function Form() {
  const inputRef =
    useRef(null);

  function handleFocus() {
    inputRef.current?.focus();
  }

  return (
    <>
      <MyInput
        ref={inputRef}
      />

      <button
        onClick={handleFocus}
      >
        Focus
      </button>
    </>
  );
}
```

The parent sees:

```js
{
  focus() {
    // ...
  }
}
```

instead of the entire DOM element.

---

# 18. useImperativeHandle Syntax ⭐⭐⭐⭐⭐

General form:

```jsx
useImperativeHandle(
  ref,
  createHandle,
  dependencies
);
```

Example:

```jsx
useImperativeHandle(
  ref,
  () => ({
    focus() {
      inputRef.current?.focus();
    },

    clear() {
      // imperative operation
    },
  }),
  []
);
```

Arguments:

```text
ref
↓
ref received from parent

createHandle
↓
returns value exposed to parent

dependencies
↓
reactive values used while
creating the handle
```

---

# 19. DOM Ref vs Imperative Handle ⭐⭐⭐⭐⭐

Direct DOM exposure:

```text
Parent
  ↓
ref
  ↓
<input DOM node>
  ↓
all DOM methods/properties
```

With `useImperativeHandle`:

```text
Parent
  ↓
ref
  ↓
{
  focus(),
  clear()
}
  ↓
Child controls implementation
```

This creates a better abstraction boundary.

---

# 20. Why Restrict the Imperative API?

Suppose your reusable component internally changes from:

```jsx
<input />
```

to:

```jsx
<div>
  <input />
  <button />
</div>
```

If parents depend directly on the DOM structure, refactoring becomes harder.

But if parents only know:

```js
searchRef.current.focus();
```

the child can change its internal DOM while preserving its public imperative API.

Think:

```text
implementation details
      ↓
hidden inside child

public imperative methods
      ↓
stable interface
```

---

# 21. Example: Search Input Imperative API ⭐⭐⭐⭐⭐

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
      placeholder="Search developers"
    />
  );
}
```

Parent:

```jsx
function DeveloperSearch() {
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
        Focus search
      </button>
    </>
  );
}
```

The parent knows only the public methods.

---

# 22. Do Not Use Imperative Handles for Normal Data Flow ⭐⭐⭐⭐⭐

Bad design:

```jsx
formRef.current
  .setUsername("Vikash");

formRef.current
  .setEmail("...");
```

if ordinary props/state can model the same behavior.

Prefer:

```jsx
<Form
  username={username}
  email={email}
/>
```

Normal React communication should remain:

```text
state
  ↓
props
  ↓
child

events
  ↑
callbacks
  ↑
parent
```

Imperative refs are escape hatches, not a replacement for props.

---

# 23. Good useImperativeHandle Use Cases

Appropriate examples include exposing focused imperative capabilities such as:

```text
focus()
scrollToTop()
openNativePicker()
play()
pause()
selectText()
resetImperativeWidget()
```

especially when wrapping:

- browser APIs
- third-party widgets
- reusable UI primitives

Use the smallest API needed.

---

# 24. Poor useImperativeHandle Use Cases

Avoid using it for ordinary application state:

```text
setUser()
setProducts()
setTheme()
setLoggedIn()
setCartItems()
```

These should generally flow through React's normal state architecture.

---

# 25. Imperative Handle with Dependencies

Suppose an exposed method depends on reactive values:

```jsx
function Editor({
  ref,
  documentId,
}) {
  useImperativeHandle(
    ref,
    () => ({
      save() {
        saveDocument(
          documentId
        );
      },
    }),
    [documentId]
  );

  return <EditorUI />;
}
```

Because the created handle uses:

```text
documentId
```

it belongs in the dependency list.

When it changes, React can recreate the handle with the new value.

---

# 26. Dependency Rules Still Matter ⭐⭐⭐⭐⭐

Do not write:

```jsx
useImperativeHandle(
  ref,
  () => ({
    save() {
      saveDocument(
        documentId
      );
    },
  }),
  []
);
```

if the handle needs the latest `documentId`.

This can create the same type of stale-closure problem discussed in Lesson 23.

Remember:

```text
Hooks using closures
      ↓
reactive values
      ↓
dependency reasoning
```

still applies.

---

# 27. Callback Refs ⭐⭐⭐⭐⭐

React also supports functions as refs.

```jsx
function Example() {
  return (
    <input
      ref={(node) => {
        console.log(node);
      }}
    />
  );
}
```

React calls the ref callback when it needs to attach the node.

Conceptually:

```text
DOM attached
    ↓
ref callback(node)
```

Historically, detachment commonly involved calling the callback with:

```text
null
```

Modern React also supports ref callback cleanup functions.

---

# 28. React 19 Ref Callback Cleanup ⭐⭐⭐⭐⭐

A ref callback can return a cleanup function.

Conceptually:

```jsx
<div
  ref={(node) => {
    if (!node) {
      return;
    }

    // setup using node

    return () => {
      // cleanup for node
    };
  }}
/>
```

This is useful when attaching imperative behavior directly through a ref callback.

The setup and cleanup stay together.

---

# 29. Callback Ref Example

```jsx
function MeasuredBox() {
  return (
    <div
      ref={(node) => {
        if (!node) {
          return;
        }

        const observer =
          new ResizeObserver(
            () => {
              console.log(
                node.getBoundingClientRect()
              );
            }
          );

        observer.observe(node);

        return () => {
          observer.disconnect();
        };
      }}
    >
      Resize me
    </div>
  );
}
```

This directly associates the observer lifecycle with the DOM node.

Use such patterns when they genuinely simplify an imperative integration.

---

# 30. Object Refs vs Callback Refs

Object ref:

```jsx
const ref =
  useRef(null);

<div ref={ref} />
```

Useful when you want:

```text
stable ref object
+
later imperative access
```

Callback ref:

```jsx
<div
  ref={(node) => {
    // react to attachment
  }}
/>
```

Useful when you need:

```text
logic when a specific node
is attached/detached
```

Neither is universally better.

Choose based on the lifecycle you need.

---

# 31. Dynamic Lists and Refs ⭐⭐⭐⭐⭐

This is incorrect if you expect one ref to represent every list item:

```jsx
const itemRef =
  useRef(null);

items.map((item) => (
  <div
    key={item.id}
    ref={itemRef}
  >
    {item.name}
  </div>
));
```

One ref object cannot meaningfully represent all nodes simultaneously.

For dynamic collections, use a data structure such as:

```jsx
const itemRefs =
  useRef(new Map());
```

and callback refs.

---

# 32. Dynamic List Ref Pattern

Conceptually:

```jsx
const itemRefs =
  useRef(new Map());

function getRefCallback(id) {
  return (node) => {
    if (node) {
      itemRefs.current.set(
        id,
        node
      );
    } else {
      itemRefs.current.delete(
        id
      );
    }
  };
}
```

Then:

```jsx
{items.map((item) => (
  <div
    key={item.id}
    ref={getRefCallback(
      item.id
    )}
  >
    {item.name}
  </div>
))}
```

Now:

```text
itemRefs.current
        ↓
Map
├── id 1 → DOM node 1
├── id 2 → DOM node 2
└── id 3 → DOM node 3
```

This can support operations such as scrolling to a specific list item.

---

# 33. Example: Scroll to a Developer

```jsx
function DeveloperList({
  developers,
}) {
  const developerRefs =
    useRef(new Map());

  function scrollToDeveloper(
    id
  ) {
    developerRefs.current
      .get(id)
      ?.scrollIntoView({
        behavior: "smooth",
      });
  }

  // ...
}
```

The map stores DOM references by stable developer ID.

This is useful when an imperative action must target a particular rendered node.

---

# 34. Refs and Conditional Rendering

Suppose:

```jsx
{open && (
  <input ref={inputRef} />
)}
```

When:

```text
open = false
```

the input does not exist.

Therefore:

```text
inputRef.current = null
```

You cannot focus a node before it exists.

Correct timing matters.

---

# 35. Common Timing Bug ⭐⭐⭐⭐⭐

Problem:

```jsx
function handleOpen() {
  setOpen(true);

  inputRef.current?.focus();
}
```

At this exact moment, React may not yet have committed the newly rendered input.

Flow:

```text
setOpen(true)
      ↓
render requested

focus immediately
      ↓
old DOM still present
      ↓
inputRef.current may be null
```

A better approach can be:

```jsx
useEffect(() => {
  if (open) {
    inputRef.current?.focus();
  }
}, [open]);
```

Now focus occurs after the input has been committed.

---

# 36. flushSync Is a Rare Escape Hatch

React provides `flushSync` for rare cases where an imperative DOM operation must happen immediately after forcing a React update to commit.

Conceptually:

```text
force state update commit
        ↓
DOM now updated
        ↓
perform imperative DOM action
```

Do not use `flushSync` as the normal solution.

It can hurt performance and breaks normal batching assumptions.

Most UI should rely on ordinary React timing.

---

# 37. Refs and Third-Party Libraries

Suppose a chart library expects a DOM node:

```jsx
function Chart({
  data,
}) {
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
    <div ref={containerRef} />
  );
}
```

Here:

```text
ref
→ gives DOM node

Effect
→ synchronizes third-party system

cleanup
→ destroys external resource
```

This connects Lessons 19, 21, and 24.

---

# 38. Real-World CodeBuddy Example: Chat Composer ⭐⭐⭐⭐⭐

Imagine the parent needs only to focus the composer.

Child:

```jsx
function ChatComposer({
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
    <textarea
      ref={inputRef}
      placeholder="Write a message..."
    />
  );
}
```

Parent:

```jsx
function Conversation() {
  const composerRef =
    useRef(null);

  function handleReply() {
    composerRef.current
      ?.focus();
  }

  return (
    <>
      <ChatComposer
        ref={composerRef}
      />

      <button
        onClick={handleReply}
      >
        Reply
      </button>
    </>
  );
}
```

Public API:

```text
ChatComposer
     ↓
focus()
```

Internal DOM remains private.

---

# 39. Why This Is Better Than Exposing textarea

Without `useImperativeHandle`:

```text
Parent
  ↓
<textarea DOM>
  ↓
can manipulate everything
```

With it:

```text
Parent
  ↓
ChatComposer API
  ↓
focus()
```

This follows the principle:

> **Expose the minimum imperative capability required.**

---

# 40. Imperative API vs Controlled API ⭐⭐⭐⭐⭐

Suppose a modal needs to open and close.

You could create:

```js
modalRef.current.open();
modalRef.current.close();
```

But often a controlled API is clearer:

```jsx
<Modal
  open={open}
  onOpenChange={setOpen}
/>
```

Why?

Because modal visibility is normal UI state.

Prefer:

```text
declarative controlled API
```

unless the operation is genuinely imperative.

---

# 41. When useImperativeHandle Is Appropriate

Ask:

```text
Does the parent need an imperative
operation that is difficult or unnatural
to express through props?
```

Examples:

```text
focus
scroll
select
play/pause
native picker
imperative third-party API
```

Then `useImperativeHandle` may be appropriate.

If the parent is trying to change ordinary application data, use state and props.

---

# 42. Encapsulation Mental Model ⭐⭐⭐⭐⭐

```text
Child component
┌────────────────────────────┐
│ internal DOM               │
│ internal refs              │
│ internal implementation    │
│                            │
│ public imperative handle:  │
│   focus()                  │
│   select()                 │
└─────────────┬──────────────┘
              │
              ↓
            Parent
```

The parent should not need to know how the child implements those operations.

---

# 43. Common Mistakes ⭐⭐⭐⭐⭐

## Mistake 1 — Using refs instead of state for normal UI

Use state when UI should react to changes.

---

## Mistake 2 — Manually changing DOM that React owns

Prefer JSX and state for normal UI updates.

---

## Mistake 3 — Trying to access a DOM ref before commit

The node may still be `null`.

---

## Mistake 4 — Exposing an entire DOM node unnecessarily

Use `useImperativeHandle` when a smaller API is safer.

---

## Mistake 5 — Using imperative methods for ordinary parent-child data flow

Prefer props and callback props.

---

## Mistake 6 — Forgetting dependencies in useImperativeHandle

Reactive values used to create the handle must be considered.

---

## Mistake 7 — Using one object ref for many dynamic list elements

Use a ref collection and callback refs when needed.

---

## Mistake 8 — Using unstable/random keys with ref collections

Use stable item identity.

---

## Mistake 9 — Assuming forwardRef is required in new React 19 code

React 19 supports `ref` as a prop for function components.

Still recognize `forwardRef` in existing code.

---

## Mistake 10 — Overusing useLayoutEffect

Use it only when DOM measurement/mutation must happen before paint.

---

# 44. Interview Questions ⭐⭐⭐⭐⭐

## Q1. What is a DOM ref in React?

**Answer:** A ref attached to a DOM element gives imperative access to the underlying browser DOM node after React commits it.

---

## Q2. When should you use a DOM ref?

**Answer:** For imperative operations such as focus, scrolling, measurement, media control, selection, or integration with DOM-based third-party libraries.

---

## Q3. Should refs replace normal state-driven UI?

**Answer:** No. Normal UI should remain declarative using state, props, and JSX.

---

## Q4. When is ref.current populated with a DOM node?

**Answer:** During the commit process after React attaches the DOM element.

---

## Q5. What is useImperativeHandle?

**Answer:** A Hook that customizes the value a component exposes through its ref, allowing it to expose a limited imperative API instead of its entire internal DOM node.

---

## Q6. Why use useImperativeHandle instead of exposing a DOM node?

**Answer:** It improves encapsulation by exposing only the operations the parent actually needs.

---

## Q7. Give an example of an imperative handle.

**Answer:**

```js
{
  focus(),
  select()
}
```

instead of exposing the full input element.

---

## Q8. Does React 19 require forwardRef for function components?

**Answer:** No. React 19 supports receiving `ref` as a prop. `forwardRef` remains important to recognize in older/existing React code.

---

## Q9. What is a callback ref?

**Answer:** A function passed to the `ref` attribute that React invokes when attaching or detaching a node. Modern React can also use a cleanup function returned from the ref callback.

---

## Q10. Why might useLayoutEffect be used with a DOM ref?

**Answer:** For layout measurement or DOM work that must happen after commit but before the browser paints to avoid visible flicker.

---

## Q11. Should useImperativeHandle be used for updating application state?

**Answer:** Usually no. Application state should generally use React's declarative state/props architecture.

---

## Q12. How can you manage refs for a dynamic list?

**Answer:** Store the DOM nodes in a ref-backed collection such as a Map, typically populated through callback refs using stable item IDs.

---

# 45. Interview Scenario 1 ⭐⭐⭐⭐⭐

Why can this fail?

```jsx
function handleClick() {
  setOpen(true);
  inputRef.current?.focus();
}
```

Because:

```text
setOpen
does not mean DOM is
already committed synchronously
at the next line
```

The input may not exist yet.

A post-commit Effect tied to `open` may be more appropriate.

---

# 46. Interview Scenario 2 ⭐⭐⭐⭐⭐

A reusable `SearchInput` should let its parent focus it but should not expose the whole DOM node.

Use:

```text
internal DOM ref
+
useImperativeHandle
+
public focus() method
```

This keeps the child implementation encapsulated.

---

# 47. Interview Scenario 3 ⭐⭐⭐⭐⭐

Should a parent open a modal using:

```js
modalRef.current.open();
```

or:

```jsx
<Modal
  open={open}
  onOpenChange={setOpen}
/>
```

For ordinary application UI state, the controlled declarative API is usually preferable.

Imperative handles should be reserved for genuinely imperative behavior.

---

# 48. Interview Scenario 4 ⭐⭐⭐⭐⭐

What is wrong here?

```jsx
const ref =
  useRef(null);

return items.map(
  (item) => (
    <div
      key={item.id}
      ref={ref}
    >
      {item.name}
    </div>
  )
);
```

A single ref object is being attached to multiple elements.

If you need direct access to each node, use a collection such as:

```text
Map<itemId, DOMNode>
```

with callback refs.

---

# 49. Complete Mental Model ⭐⭐⭐⭐⭐

```text
                Need to interact
                with child/DOM
                     │
                     ↓
          Can normal props/state
             express the behavior?
              │              │
             yes             no
              │              │
              ↓              ↓
       use declarative     Is operation
       React data flow     imperative?
                              │
                        ┌─────┴─────┐
                       no          yes
                        │            │
                        ↓            ↓
                  reconsider      use ref
                    design           │
                                     ↓
                         Need entire DOM node?
                              │          │
                             yes         no
                              │          │
                              ↓          ↓
                         expose ref   expose limited
                                     imperative API
                                           │
                                           ↓
                                  useImperativeHandle
```

---

# 50. Section 3 Complete Mental Model ⭐⭐⭐⭐⭐

You have now connected all major Effect and ref concepts:

```text
useEffect
   ↓
synchronize with external systems
   │
   ├── dependencies
   │      ↓
   │   reactive values
   │
   ├── cleanup
   │      ↓
   │   undo synchronization
   │
   ├── closures
   │      ↓
   │   render snapshots
   │
   └── avoid unnecessary Effects
          ↓
      derive/render/event logic


useRef
   ↓
persistent non-reactive value
   │
   ├── timer/resource handle
   ├── latest mutable value
   └── DOM node
          ↓
   imperative DOM interaction
          ↓
   useImperativeHandle
          ↓
   controlled imperative API
```

---

# 51. Quick Revision

### DOM ref

```jsx
const inputRef =
  useRef(null);

<input ref={inputRef} />
```

### Imperative DOM action

```jsx
inputRef.current?.focus();
```

### React 19 custom component ref

```jsx
function MyInput({
  ref,
}) {
  return (
    <input ref={ref} />
  );
}
```

### Limited imperative API

```jsx
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
```

Remember:

```text
normal UI/data
→ state + props

imperative DOM operation
→ ref

restricted child imperative API
→ useImperativeHandle
```

---

# 52. Key Takeaways

- React should normally control the DOM declaratively.
- DOM refs are an escape hatch for necessary imperative operations.
- Common DOM-ref use cases include focus, scrolling, measurement, media control, selection, and third-party integration.
- React assigns DOM nodes to refs during the commit process.
- A DOM ref can be `null` before attachment and after removal.
- Do not manually mutate React-owned DOM for normal application state.
- Timing matters when an element is conditionally rendered.
- `useEffect` can perform DOM work after commit.
- `useLayoutEffect` is useful only when layout-sensitive work must occur before paint.
- React 19 allows function components to receive `ref` as a prop.
- `forwardRef` is still important to understand for existing React code, but new React 19 code can often avoid it.
- `useImperativeHandle` customizes what a parent receives through a component ref.
- Prefer exposing a small imperative API rather than an entire internal DOM node.
- Imperative handles should not replace normal props/state communication.
- Dependency rules still apply to values used when creating imperative handles.
- Callback refs are useful when logic must run as nodes attach or detach.
- React 19 supports cleanup functions returned from callback refs.
- Dynamic collections of DOM nodes can be managed with callback refs and a ref-backed Map.
- Use stable IDs for dynamic ref collections.
- The central rule is: **stay declarative by default and use refs only where imperative behavior is genuinely required.**

---

## Section 3 — Effects and Refs Completed ✅

You have completed:

- Lesson 19 — useEffect Fundamentals
- Lesson 20 — Dependency Arrays and Reactive Dependencies
- Lesson 21 — Effect Cleanup and Memory Leaks
- Lesson 22 — You Might Not Need an Effect
- Lesson 23 — Closures, Stale Closures and Effects
- Lesson 24 — useRef
- Lesson 25 — DOM Refs and useImperativeHandle

---

## Next Section — Forms

➡️ [Lesson 26 — Forms in React](../04-forms/26-forms.md)
