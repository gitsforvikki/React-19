# Lesson 31 — useReducer ⭐⭐⭐⭐⭐

## Why useReducer?

Use `useState` for simple updates. Choose **`useReducer`** when related state has multiple actions or complex transition rules.

```text
User event → dispatch(action) → reducer(state, action) → next state → render
```

`dispatch` describes **what happened**; the reducer defines **how state changes**.

## Practical Example — Applications

```jsx
import { useReducer } from "react";

const initialApplications = [];

function applicationsReducer(state, action) {
  switch (action.type) {
    case "added":
      return [...state, action.application];
    case "status_changed":
      return state.map(app =>
        app.id === action.id ? { ...app, status: action.status } : app
      );
    case "deleted":
      return state.filter(app => app.id !== action.id);
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

function Applications() {
  const [applications, dispatch] = useReducer(
    applicationsReducer,
    initialApplications
  );

  return (
    <>
      <button onClick={() => dispatch({
        type: "added",
        application: { id: crypto.randomUUID(), status: "applied" }
      })}>
        Add application
      </button>
      <p>Total: {applications.length}</p>
    </>
  );
}
```

Typical actions:

```jsx
dispatch({ type: "status_changed", id, status: "interview" });
dispatch({ type: "deleted", id });
```

## Important Rules

- Signature: `const [state, dispatch] = useReducer(reducer, initialState)`.
- **Pure reducer:** same state + action → same result. No fetching, timers, or external mutations inside the reducer.
- **Immutable updates:** return new arrays/objects for changes; never `push` into existing state or edit its properties.
- Preserve unchanged object fields with `...state` when appropriate.
- Do async work in handlers or a data layer, then dispatch success/failure actions.
- `dispatch` has a stable identity; updates take effect in the **next render**, not immediately in the running handler.
- The reducer manages state for its component. It does not make state global.

## Common Mistakes

- Using a reducer for a simple boolean toggle.
- Mutating state and returning the same reference.
- Calling APIs inside the reducer.
- Storing values that can be derived from existing state (e.g. a selected object when its ID and list already exist).

## Interview Quick Check

**useState vs useReducer?** Both manage local React state. Reducers help organize complex, related transitions as actions.

**What is an action?** Usually an object with a `type` and optional payload.

**Why must reducers be pure?** React may invoke them more than once in development Strict Mode; pure logic is predictable and independently testable.

---

➡️ [Lesson 32 — useReducer + Context Architecture](./32-reducer-context.md)
