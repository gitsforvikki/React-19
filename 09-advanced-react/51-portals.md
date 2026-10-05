# Lesson 51 — Portals

## 1. What Is a Portal?

A **Portal** lets a React component render some of its children into a different place in the DOM while those children remain part of the same React tree.

React DOM provides:

```jsx
createPortal(children, domNode)
```

Import it from:

```jsx
import { createPortal } from "react-dom";
```

---

## 2. Why Do We Need Portals?

Consider:

```text
App
└── Card
    └── Modal
```

The Card may have CSS such as:

```css
overflow: hidden;
```

or create a stacking context.

Rendering the modal physically inside that DOM subtree can make overlays difficult.

A Portal lets the modal DOM appear somewhere such as:

```html
<body>
  <div id="root"></div>
  <div id="modal-root"></div>
</body>
```

while the modal still logically belongs to its React parent.

---

## 3. Basic Example ⭐⭐⭐⭐⭐

HTML:

```html
<div id="root"></div>
<div id="modal-root"></div>
```

React:

```jsx
import { createPortal } from "react-dom";

function Modal({ children }) {
  const modalRoot =
    document.getElementById(
      "modal-root"
    );

  return createPortal(
    children,
    modalRoot
  );
}
```

Usage:

```jsx
function App() {
  return (
    <Modal>
      <div>
        Modal content
      </div>
    </Modal>
  );
}
```

DOM placement and React ownership are now different.

---

## 4. The Most Important Mental Model ⭐⭐⭐⭐⭐

```text
React tree:

App
└── Card
    └── Modal
        └── Dialog

DOM tree:

body
├── #root
│   └── Card DOM
│
└── #modal-root
    └── Dialog DOM
```

The Portal changes:

```text
physical DOM placement
```

It does **not** change:

```text
React parent-child relationship
```

This explains Context and event behavior.

---

## 5. Context Still Works Through Portals

Because Portal children remain in the same React tree, they can consume Context from their logical React ancestors.

```jsx
<ThemeContext.Provider
  value="dark"
>
  <Modal>
    <Dialog />
  </Modal>
</ThemeContext.Provider>
```

`Dialog` can still read:

```jsx
useContext(ThemeContext)
```

even though its DOM is rendered elsewhere.

---

## 6. Events Bubble Through the React Tree ⭐⭐⭐⭐⭐

Suppose:

```jsx
function Parent() {
  return (
    <div
      onClick={() =>
        console.log("parent")
      }
    >
      <Modal>
        <button>
          Click
        </button>
      </Modal>
    </div>
  );
}
```

Even if the button's DOM lives under `#modal-root`, React event propagation follows the React tree.

Conceptually:

```text
button click
    ↓
Modal
    ↓
Parent React ancestor
```

This is a very common interview question.

---

## 7. DOM Position vs React Event Propagation

Do not assume:

```text
different DOM parent
→ unrelated React event tree
```

For Portal children:

```text
DOM placement
≠
React ownership
```

Native DOM APIs still observe the actual DOM structure, while React's event model follows the React component tree.

---

## 8. Stopping Propagation

If a modal click should not trigger an ancestor React handler:

```jsx
function Dialog() {
  return (
    <div
      onClick={(event) =>
        event.stopPropagation()
      }
    >
      ...
    </div>
  );
}
```

But do not add `stopPropagation` automatically.

First decide whether the ancestor should semantically receive the event.

---

## 9. Typical Portal Use Cases

Portals are useful for UI that must visually escape normal layout constraints:

- modal dialogs
- tooltips
- popovers
- dropdown overlays
- notifications/toasts
- floating menus

Portals are not required for every overlay. Use them when DOM placement actually benefits from escaping an ancestor.

---

## 10. Modal Example

```jsx
function Modal({
  open,
  children,
}) {
  if (!open) {
    return null;
  }

  return createPortal(
    <div className="backdrop">
      <div className="dialog">
        {children}
      </div>
    </div>,
    document.body
  );
}
```

Then:

```jsx
<Modal open={isOpen}>
  <h2>Delete application?</h2>
</Modal>
```

The dialog can physically live near the document body instead of deep inside page layout.

---

## 11. Portal Does Not Automatically Make a Modal Accessible ⭐⭐⭐⭐⭐

A Portal only changes rendering location.

A production dialog still needs correct behavior such as:

- semantic dialog structure
- accessible name
- keyboard interaction
- focus management
- restoring focus after closing
- Escape behavior where appropriate
- preventing interaction with background content where appropriate

Do not confuse:

```text
Portal
with
accessible modal implementation
```

---

## 12. Focus Management

When a dialog opens, focus often needs to move into it.

When it closes, focus should generally return to the element that opened it.

This may involve:

- DOM refs
- effects/layout effects where appropriate
- a well-tested accessible dialog primitive

Portals make DOM placement easier but do not implement focus behavior for you.

---

## 13. Portal Target Must Exist

```jsx
const target =
  document.getElementById(
    "modal-root"
  );
```

If your code expects a target, ensure it exists before passing it to `createPortal`.

In browser-only React applications, a dedicated root can be declared in HTML.

Framework/server-rendering environments require additional care around DOM availability; those framework details belong in their dedicated repository.

---

## 14. Rendering Into document.body

You can use:

```jsx
createPortal(
  <Dialog />,
  document.body
)
```

This is convenient for overlays.

However, consider:

- styling
- accessibility
- ownership
- testing
- interaction with other body-level UI

A dedicated portal container can sometimes be clearer.

---

## 15. Changing the Portal Target

If the `domNode` passed to `createPortal` changes, React moves/recreates the portal content for the new destination as appropriate.

Do not casually switch targets if preserving subtree state/identity matters.

Treat the target as part of the Portal's rendering structure.

---

## 16. Optional Portal Key

The API also supports an optional key:

```jsx
createPortal(
  children,
  domNode,
  key
)
```

The key can help React identify portals in situations involving multiple/dynamic portals.

Most basic modal use cases do not need to provide it manually.

---

## 17. CSS Problems Portals Can Solve

A common case:

```text
ancestor
├── overflow: hidden
└── child overlay gets clipped
```

Portal:

```text
React child
    │
    └── DOM rendered under body
        → escapes clipping ancestor
```

Portals can also simplify certain stacking-context problems.

But understand CSS first; not every z-index issue requires a Portal.

---

## 18. Portals and Positioning

A tooltip rendered under `document.body` no longer shares the same DOM coordinate context as its logical trigger.

You may need to calculate its position using:

- `getBoundingClientRect()`
- scroll offsets
- resize/scroll observers
- a positioning library

A Portal solves placement ownership, not geometry automatically.

---

## 19. Portals Do Not Create a New React Root

This is important.

```jsx
createPortal(...)
```

does not mean:

```text
createRoot(...)
```

A Portal remains attached to the existing React tree.

That is why Context and React event propagation continue to work.

---

## 20. Portal vs Separate React Root ⭐⭐⭐⭐⭐

### Portal

```text
same React tree
same Context
React events follow logical ancestors
different DOM location
```

### Separate createRoot

```text
separate React root/tree
not automatically the same Context ancestry
independent root lifecycle
```

Do not use another root merely to build a modal.

---

## 21. Portals and State

Because the Portal content is still a React component subtree, it can use normal React state:

```jsx
function Dialog() {
  const [step, setStep] =
    useState(1);

  // ...
}
```

Its physical DOM location does not change normal state semantics.

---

## 22. CareerLoop Example

Suppose each application card has a Delete action.

Bad DOM layout:

```text
ApplicationCard
└── overflow-hidden container
    └── confirmation dialog
        → may be clipped
```

A Portal can render the confirmation dialog under a top-level overlay container while keeping the dialog logically owned by the card/page that opened it.

---

## 23. CodeBuddy Example

A developer profile card may open a detailed overlay.

```text
DeveloperCard
    │
    └── ProfileDialog (React tree)
              │
              ↓ Portal
        body/#modal-root (DOM)
```

The dialog can still consume app Context and invoke callback props from its React ancestors.

---

## 24. Common Mistakes ⭐⭐⭐⭐⭐

1. Thinking a Portal creates another React application.
2. Assuming Context stops working.
3. Assuming React events follow only the physical DOM parent.
4. Using Portals to fix every CSS issue.
5. Forgetting accessibility and focus management.
6. Assuming a Portal automatically creates a modal.
7. Passing a missing DOM target.
8. Ignoring positioning when portaling tooltips/popovers.
9. Creating a separate React root when a Portal is the correct tool.
10. Forgetting that the portal subtree still follows normal React state/lifecycle rules.

---

## 25. Interview Questions ⭐⭐⭐⭐⭐

### What is a React Portal?

A Portal renders React children into a different DOM node while keeping them in the same React tree.

### Why are Portals useful?

They help overlays escape layout constraints such as clipping and stacking contexts.

### Does Context work through a Portal?

Yes.

### How do React events propagate from Portal children?

According to the React tree, not merely the physical DOM hierarchy.

### Does createPortal create a new React root?

No.

### Portal vs createRoot?

A Portal stays in the same React tree; `createRoot` creates a separate React root.

### Does using a Portal make a modal accessible?

No. Accessibility, focus, keyboard behavior, and dialog semantics must still be implemented.

### Common use cases?

Dialogs, tooltips, popovers, floating menus, and toasts.

---

## 26. Complete Mental Model

```text
          REACT TREE
              App
               │
              Card
               │
             Modal
               │
             Dialog
               │
               │ createPortal
               ↓
          PHYSICAL DOM
body
├── #root
│   └── Card DOM
│
└── #modal-root
    └── Dialog DOM

Context/events/state
→ follow React ownership

layout/DOM APIs
→ see physical DOM placement
```

---

## 27. Key Takeaways

- Portals change DOM placement, not React ownership.
- Use `createPortal` from `react-dom`.
- Portal children remain in the same React tree.
- Context continues to work.
- React events propagate according to the React tree.
- Portals are useful for overlays that must escape layout constraints.
- They do not automatically solve accessibility, focus, or positioning.
- A Portal is not a separate React root.
- Use them intentionally rather than as a universal CSS workaround.

---

## Next Lesson

➡️ [Lesson 52 — Error Boundaries ⭐⭐⭐⭐⭐](./52-error-boundaries.md)
