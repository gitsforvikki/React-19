# Lesson 09 — Events and Event Handling

## 1. What Is Event Handling?

An **event** represents something that happens in the user interface.

Common browser events include:

- click
- input/change
- submit
- focus
- blur
- keyboard events
- mouse/pointer events

React lets components respond to these events using event handler functions.

```jsx
function Button() {
  function handleClick() {
    console.log("Button clicked");
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

Mental model:

```text
User interaction
      ↓
Browser event
      ↓
React event handler
      ↓
Application logic
      ↓
Possible state update
      ↓
UI renders again
```

---

## 2. Event Prop Naming

React event props normally use camelCase:

```jsx
onClick
onChange
onSubmit
onFocus
onBlur
onKeyDown
onMouseEnter
```

Example:

```jsx
<button onClick={handleClick}>
  Save
</button>
```

This differs from writing lowercase HTML event attributes such as `onclick`.

---

## 3. Pass a Function, Do Not Call It ⭐⭐⭐⭐⭐

Correct:

```jsx
<button onClick={handleClick}>
  Click
</button>
```

Here you **pass the function**.

Wrong in most cases:

```jsx
<button onClick={handleClick()}>
  Click
</button>
```

This calls `handleClick` while the component is rendering.

Think:

```text
onClick={handleClick}
        ↓
"Run this later when clicked"

onClick={handleClick()}
        ↓
"Run this now during render"
```

This is one of the most common beginner mistakes.

---

## 4. Inline Event Handlers

You can define a handler inline:

```jsx
<button
  onClick={() => {
    console.log("Clicked");
  }}
>
  Click
</button>
```

For very small logic, this is fine.

For larger behavior, a named function is often clearer:

```jsx
function handleDelete() {
  // validation
  // state update
  // other logic
}

return (
  <button onClick={handleDelete}>
    Delete
  </button>
);
```

Choose readability over rigid rules.

---

## 5. Passing Arguments to Event Handlers ⭐⭐⭐⭐⭐

Suppose:

```js
function deleteUser(id) {
  console.log("Delete:", id);
}
```

Wrong:

```jsx
<button onClick={deleteUser(user.id)}>
  Delete
</button>
```

That calls the function during rendering.

Use a wrapper function:

```jsx
<button onClick={() => deleteUser(user.id)}>
  Delete
</button>
```

Flow:

```text
Render
  ↓
create callback
  ↓
user clicks
  ↓
callback executes
  ↓
deleteUser(user.id)
```

---

## 6. The Event Object ⭐⭐⭐⭐⭐

React passes an event object to event handlers.

```jsx
function handleClick(event) {
  console.log(event);
}

return (
  <button onClick={handleClick}>
    Click
  </button>
);
```

You can commonly access values such as:

```js
event.target
event.currentTarget
event.type
```

For forms:

```js
event.preventDefault()
```

For propagation:

```js
event.stopPropagation()
```

---

## 7. target vs currentTarget ⭐⭐⭐⭐⭐

This is a useful interview concept.

Suppose:

```jsx
function handleClick(event) {
  console.log("target:", event.target);
  console.log("currentTarget:", event.currentTarget);
}

return (
  <button onClick={handleClick}>
    <span>Save</span>
  </button>
);
```

If the user clicks the `span`:

```text
event.target
     ↓
element where the event originated
     ↓
<span>

event.currentTarget
     ↓
element whose handler is currently running
     ↓
<button>
```

Simplified:

> `target` = where the event came from.

> `currentTarget` = element whose current handler is executing.

---

## 8. Event Handlers Can Update State

A common React interaction:

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  function handleIncrement() {
    setCount((current) => current + 1);
  }

  return (
    <div>
      <p>{count}</p>
      <button onClick={handleIncrement}>
        Increment
      </button>
    </div>
  );
}
```

Flow:

```text
Click
 ↓
handleIncrement
 ↓
setCount
 ↓
React schedules state update
 ↓
component renders again
 ↓
updated UI
```

State gets a dedicated lesson next.

---

## 9. Event Handler Props ⭐⭐⭐⭐⭐

A parent can pass behavior to a child:

```jsx
function App() {
  function handleSave() {
    console.log("Saved");
  }

  return <SaveButton onSave={handleSave} />;
}

function SaveButton({ onSave }) {
  return (
    <button onClick={onSave}>
      Save
    </button>
  );
}
```

Flow:

```text
Parent
  │
  │ passes onSave
  ↓
Child
  │
  │ click
  ↓
calls parent's function
```

This connects event handling with one-way data flow.

---

## 10. Naming Event Handler Props

For built-in DOM elements, use React's standard event props:

```jsx
<button onClick={...} />
<form onSubmit={...} />
<input onChange={...} />
```

For your own components, you control the prop API:

```jsx
<ProductCard
  onAddToCart={handleAddToCart}
  onWishlist={handleWishlist}
/>
```

Inside:

```jsx
function ProductCard({
  onAddToCart,
  onWishlist,
}) {
  return (
    <>
      <button onClick={onAddToCart}>
        Add to cart
      </button>

      <button onClick={onWishlist}>
        Wishlist
      </button>
    </>
  );
}
```

Names beginning with `on` clearly communicate that the prop represents an event-like callback.

---

## 11. Form Submission and preventDefault() ⭐⭐⭐⭐⭐

A browser normally performs default behavior when a form is submitted.

In many client-side React flows, you may want to handle submission in JavaScript:

```jsx
function LoginForm() {
  function handleSubmit(event) {
    event.preventDefault();

    console.log("Handle form submission");
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" />
      <button type="submit">
        Login
      </button>
    </form>
  );
}
```

`preventDefault()` prevents the browser's default action for that event.

Important:

> It does not stop event propagation.

That is a different concept.

---

## 12. Event Propagation / Bubbling ⭐⭐⭐⭐⭐

Browser events commonly propagate through the DOM.

Example:

```jsx
function App() {
  function handleParentClick() {
    console.log("Parent clicked");
  }

  function handleButtonClick() {
    console.log("Button clicked");
  }

  return (
    <div onClick={handleParentClick}>
      <button onClick={handleButtonClick}>
        Click
      </button>
    </div>
  );
}
```

Clicking the button can produce:

```text
Button clicked
Parent clicked
```

Simplified bubbling model:

```text
button
  ↓
div
  ↓
higher ancestors
```

The event originates lower in the tree and can bubble upward.

---

## 13. stopPropagation() ⭐⭐⭐⭐⭐

If the child event should not continue bubbling:

```jsx
function handleButtonClick(event) {
  event.stopPropagation();

  console.log("Button clicked");
}
```

Now the parent's bubbling handler will not run for that event.

Use this intentionally.

Do not automatically stop propagation everywhere, because event propagation is useful and often expected.

---

## 14. preventDefault vs stopPropagation ⭐⭐⭐⭐⭐

These solve different problems.

### preventDefault

Prevents the browser's default behavior.

```js
event.preventDefault();
```

Example:

```text
form submit
   ↓
prevent browser's default submission/navigation behavior
```

### stopPropagation

Stops the event from continuing through propagation.

```js
event.stopPropagation();
```

Example:

```text
button click
   ↓
do not let ancestor click handler respond
```

Remember:

```text
preventDefault
      ↓
default browser behavior

stopPropagation
      ↓
event propagation
```

They are not interchangeable.

---

## 15. Event Capturing

Events can also be handled during the capture phase.

React supports capture handlers such as:

```jsx
<div onClickCapture={handleCapture}>
  <button onClick={handleClick}>
    Click
  </button>
</div>
```

Simplified event flow:

```text
Capture Phase
     ↓
ancestor → target

Target
     ↓

Bubble Phase
     ↓
target → ancestor
```

You will use bubbling far more often in normal React code, but understanding capture is useful for interviews and advanced event handling.

---

## 16. Input Events

A common input pattern:

```jsx
function SearchBox() {
  function handleChange(event) {
    console.log(event.target.value);
  }

  return (
    <input
      type="text"
      onChange={handleChange}
    />
  );
}
```

Each change lets you read:

```js
event.target.value
```

Later, controlled components will connect this to state:

```jsx
<input
  value={query}
  onChange={(event) => setQuery(event.target.value)}
/>
```

---

## 17. Keyboard Events

Example:

```jsx
function SearchInput() {
  function handleKeyDown(event) {
    if (event.key === "Enter") {
      console.log("Search");
    }
  }

  return (
    <input onKeyDown={handleKeyDown} />
  );
}
```

Useful properties include:

```js
event.key
event.ctrlKey
event.shiftKey
event.altKey
event.metaKey
```

Prefer semantic form behavior when appropriate rather than recreating form submission entirely through keyboard handlers.

---

## 18. Event Handling and Accessibility

Avoid making non-interactive elements behave like buttons without handling accessibility correctly.

Less appropriate:

```jsx
<div onClick={handleSave}>
  Save
</div>
```

Prefer the semantic element:

```jsx
<button onClick={handleSave}>
  Save
</button>
```

Why?

A real button already provides important behavior for:

- keyboard interaction
- focus
- assistive technology
- semantics

Use the correct HTML element whenever possible.

---

## 19. Event Handlers vs Effects ⭐⭐⭐⭐⭐

A very important React distinction:

### Event handler

Runs because a specific user interaction occurred.

```jsx
function handleBuy() {
  purchaseProduct();
}
```

### Effect

Runs because rendering caused a synchronization requirement.

Later you will study:

```jsx
useEffect(() => {
  // synchronize with external system
}, []);
```

If logic should happen because the user clicked **Buy**, put it in the click handler rather than moving it into an Effect unnecessarily.

Mental model:

```text
User caused it?
     ↓
Event handler

Component needs synchronization because it rendered?
     ↓
Effect
```

This distinction prevents many bad `useEffect` patterns.

---

## 20. Keep Rendering Pure ⭐⭐⭐⭐⭐

Do not perform event-like side effects directly during rendering.

Wrong:

```jsx
function Checkout({ shouldBuy }) {
  if (shouldBuy) {
    purchaseProduct();
  }

  return <CheckoutPage />;
}
```

Rendering can happen more than once, so this is unsafe.

If the purchase is triggered by a user click:

```jsx
function Checkout() {
  function handlePurchase() {
    purchaseProduct();
  }

  return (
    <button onClick={handlePurchase}>
      Buy
    </button>
  );
}
```

Rendering calculates UI.

Event handlers perform interaction-driven work.

---

## 21. Handler Functions Can Read Props and State

Handlers are defined inside the component, so they can use values from that render:

```jsx
function ProductCard({ product }) {
  function handleBuy() {
    console.log("Buying:", product.id);
  }

  return (
    <button onClick={handleBuy}>
      Buy {product.name}
    </button>
  );
}
```

This is convenient, but later it connects to JavaScript closures and stale-closure concepts.

We will study that deeply in the Effects section.

---

## 22. Async Event Handlers

Event handlers can start asynchronous work:

```jsx
function SaveButton() {
  async function handleSave() {
    try {
      await saveProfile();
      console.log("Saved");
    } catch (error) {
      console.error("Save failed", error);
    }
  }

  return (
    <button onClick={handleSave}>
      Save
    </button>
  );
}
```

Production code should normally consider:

- loading/pending state
- preventing duplicate submissions
- errors
- success feedback
- cancellation/race conditions where relevant

These topics will appear later.

---

## 23. Avoid Duplicate Submissions

Suppose saving takes two seconds.

A user might click repeatedly:

```text
Click
Click
Click
 ↓
3 requests
```

A common pattern is to disable the action while pending:

```jsx
<button
  disabled={isSaving}
  onClick={handleSave}
>
  {isSaving ? "Saving..." : "Save"}
</button>
```

The exact implementation depends on how pending state is managed.

React 19 also introduces useful action/form patterns that we will cover later.

---

## 24. Event Delegation: Conceptual View

Browsers support event bubbling, which makes event delegation possible.

Instead of conceptually requiring unrelated independent global listeners for every interaction, event systems can take advantage of propagation.

React provides its own event handling abstraction on top of browser events.

For interviews, understand:

```text
Browser event
     ↓
React event system
     ↓
your handler
```

You usually work with React's declarative event props rather than manually attaching DOM listeners for normal component interactions.

---

## 25. React Event Object

React event handlers receive an event object that follows the DOM event model closely.

Example:

```jsx
function handleClick(event) {
  console.log(event.type);
  console.log(event.target);
  console.log(event.currentTarget);
}
```

Modern React event objects do not require the old `event.persist()` pattern that you may encounter in outdated tutorials.

If you see tutorials focused on pooled SyntheticEvents requiring `persist()`, recognize that this is legacy guidance.

---

## 26. Declarative vs Imperative Event Handling

In plain DOM JavaScript you may write:

```js
const button = document.querySelector("#save");

button.addEventListener("click", handleSave);
```

In React:

```jsx
<button onClick={handleSave}>
  Save
</button>
```

React lets the event relationship be described directly in the component's UI declaration.

This keeps the handler associated with the element/component that uses it.

---

## 27. Real-World Example: Developer Card

```jsx
function DeveloperCard({
  developer,
  onConnect,
  onSkip,
}) {
  function handleConnect() {
    onConnect(developer.id);
  }

  function handleSkip() {
    onSkip(developer.id);
  }

  return (
    <article>
      <h2>{developer.name}</h2>

      <button onClick={handleSkip}>
        Skip
      </button>

      <button onClick={handleConnect}>
        Connect
      </button>
    </article>
  );
}
```

Flow:

```text
User clicks Connect
        ↓
handleConnect
        ↓
onConnect(developer.id)
        ↓
parent handler
        ↓
application logic/state update
```

The child handles the interaction while the parent can own the larger application behavior.

---

## 28. Common Mistakes

### Mistake 1 — Calling instead of passing a handler

Wrong:

```jsx
onClick={handleClick()}
```

Correct:

```jsx
onClick={handleClick}
```

### Mistake 2 — Passing arguments incorrectly

Wrong:

```jsx
onClick={deleteUser(user.id)}
```

Correct:

```jsx
onClick={() => deleteUser(user.id)}
```

### Mistake 3 — Confusing preventDefault and stopPropagation

They solve different problems.

### Mistake 4 — Performing side effects during render

Interaction-driven side effects belong in handlers.

### Mistake 5 — Using div instead of semantic button

Use semantic HTML whenever possible.

### Mistake 6 — Ignoring duplicate async submissions

Pending actions often need protection and feedback.

### Mistake 7 — Using manual DOM listeners unnecessarily

Normal React interactions should generally use React event props.

---

## 29. Interview Questions ⭐⭐⭐⭐⭐

### Q1. How are events handled in React?

**Answer:** Event handler functions are passed to React event props such as `onClick`, `onChange`, and `onSubmit`.

### Q2. What is the difference between onClick={handleClick} and onClick={handleClick()}?

**Answer:** The first passes the function so it runs when the event occurs. The second invokes the function during rendering and passes its return value.

### Q3. How do you pass an argument to an event handler?

**Answer:** A common approach is to use a wrapper function such as `onClick={() => deleteUser(id)}`.

### Q4. What is the event object?

**Answer:** It represents information about the event and provides APIs such as `target`, `currentTarget`, `preventDefault()`, and `stopPropagation()`.

### Q5. target vs currentTarget?

**Answer:** `target` is the element where the event originated, while `currentTarget` is the element whose event handler is currently executing.

### Q6. What does preventDefault do?

**Answer:** It prevents the browser's default action associated with an event, such as default form submission behavior.

### Q7. What does stopPropagation do?

**Answer:** It stops the event from continuing through the propagation path.

### Q8. What is event bubbling?

**Answer:** It is the phase where an event originating at a descendant propagates upward through ancestors.

### Q9. Can event handlers update state?

**Answer:** Yes. User events commonly trigger state updates, which cause React to render the affected UI again.

### Q10. Event handler vs Effect?

**Answer:** Event handlers perform logic caused by specific user interactions. Effects synchronize a component with external systems as a consequence of rendering/state.

### Q11. Why should side effects not happen during render?

**Answer:** Rendering should remain pure and React may render components multiple times. Interaction-driven side effects should occur in handlers instead.

### Q12. Do we still need event.persist() in modern React?

**Answer:** No. That pattern came from older event pooling behavior and is not required in modern React.

---

## 30. Interview Scenario

### Requirement

A card itself opens details when clicked, but its Delete button should delete without opening the card.

```jsx
function UserCard({ user, onOpen, onDelete }) {
  function handleDelete(event) {
    event.stopPropagation();
    onDelete(user.id);
  }

  return (
    <article onClick={() => onOpen(user.id)}>
      <h2>{user.name}</h2>

      <button onClick={handleDelete}>
        Delete
      </button>
    </article>
  );
}
```

Without `stopPropagation()`:

```text
Delete button clicked
       ↓
delete handler
       ↓
event bubbles
       ↓
card click handler
       ↓
details also open ❌
```

With it:

```text
Delete button clicked
       ↓
delete handler
       ↓
stopPropagation()
       ↓
delete only ✓
```

Use this pattern only when the desired interaction genuinely requires stopping propagation.

---

## 31. Event Handling Mental Model

```text
User
 ↓
Interaction
 ↓
Browser Event
 ↓
React Event Prop
 ↓
Handler Function
 ├── read event
 ├── call callback
 ├── update state
 └── start async action
       ↓
possible render
       ↓
updated UI
```

---

## 32. Quick Revision

```text
React Events
├── onClick
├── onChange
├── onSubmit
├── onFocus / onBlur
└── keyboard/pointer events
```

Remember:

```jsx
onClick={handleClick}             // pass function

onClick={() => deleteUser(id)}    // pass arguments

event.preventDefault()            // stop default action

event.stopPropagation()           // stop propagation
```

And:

```text
target        = event origin
currentTarget = current handler element
```

Most important distinction:

```text
Rendering
   ↓
calculate UI

Event Handler
   ↓
respond to interaction
```

---

## 33. Key Takeaways

- React handles interactions through event props such as `onClick`, `onChange`, and `onSubmit`.
- Pass handler functions rather than invoking them during render.
- Use wrapper functions when arguments need to be passed.
- React supplies an event object to handlers.
- Understand `target` vs `currentTarget`.
- `preventDefault()` prevents browser default behavior.
- `stopPropagation()` controls event propagation.
- Events commonly bubble from descendants toward ancestors.
- Parent callbacks let child interactions request parent-owned actions.
- Keep rendering pure; interaction-driven side effects belong in event handlers.
- Prefer semantic interactive HTML such as `button`.
- Async actions should consider pending, duplicate submissions, success, and errors.
- Modern React does not require the old `event.persist()` pattern.
- Event handling is the bridge between **user interaction and state updates**.

---

## Next Lesson

➡️ [Lesson 10 — State and useState ⭐⭐⭐⭐⭐](../02-state-and-rendering/10-state-usestate.md)
