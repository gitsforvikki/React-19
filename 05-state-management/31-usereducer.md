# Lesson 31 — useReducer ⭐⭐⭐⭐⭐

## 1. Why useReducer Exists

`useState` is excellent for simple state.

But a component can become difficult to maintain when:

- state is complex
- many event handlers update it
- several fields change together
- update rules are scattered
- bugs come from inconsistent transitions

Example:

```text
handleAdd
handleEdit
handleDelete
handleArchive
handleRestore
handleReset
      ↓
all contain state-update logic
```

`useReducer` lets you move those state transition rules into one reducer function.

---

## 2. Core Mental Model ⭐⭐⭐⭐⭐

With `useState`:

```text
event
 ↓
setState(nextState)
 ↓
state changes
```

With `useReducer`:

```text
event
 ↓
dispatch(action)
 ↓
reducer(currentState, action)
 ↓
nextState
 ↓
render
```

The event describes **what happened**.

The reducer decides **how state changes**.

---

## 3. Basic Syntax ⭐⭐⭐⭐⭐

```jsx
import {
  useReducer,
} from "react";

const [
  state,
  dispatch,
] = useReducer(
  reducer,
  initialState
);
```

Reducer:

```jsx
function reducer(
  state,
  action
) {
  // return next state
}
```

---

## 4. Counter Example

```jsx
function reducer(
  state,
  action
) {
  switch (action.type) {
    case "increment":
      return {
        count:
          state.count + 1,
      };

    case "decrement":
      return {
        count:
          state.count - 1,
      };

    default:
      throw new Error(
        "Unknown action: " +
          action.type
      );
  }
}

function Counter() {
  const [
    state,
    dispatch,
  ] = useReducer(
    reducer,
    { count: 0 }
  );

  return (
    <>
      <p>{state.count}</p>

      <button
        onClick={() =>
          dispatch({
            type: "increment",
          })
        }
      >
        +
      </button>
    </>
  );
}
```

---

## 5. What Is an Action? ⭐⭐⭐⭐⭐

An action is a value describing what happened.

Common shape:

```js
{
  type: "application_added",
  application: newApplication
}
```

Another:

```js
{
  type: "status_changed",
  id: 12,
  status: "interview"
}
```

Actions should be meaningful descriptions of events/interactions.

Prefer:

```text
application_added
application_deleted
status_changed
form_reset
```

over vague names such as:

```text
update
change
doThing
```

---

## 6. Reducer Responsibility ⭐⭐⭐⭐⭐

Reducer:

```text
current state
+
action
↓
next state
```

Formally:

```js
nextState =
  reducer(
    currentState,
    action
  );
```

This makes state transition logic explicit and centralized.

---

## 7. Reducers Must Be Pure ⭐⭐⭐⭐⭐

A reducer must not perform side effects.

Do not:

```jsx
function reducer(
  state,
  action
) {
  fetch("/api/save");

  localStorage.setItem(
    "data",
    "..."
  );

  return state;
}
```

Avoid inside reducers:

- network requests
- timers
- DOM manipulation
- subscriptions
- logging that is relied upon as behavior
- mutating external variables

Reducer logic should calculate state only.

---

## 8. Do Not Mutate State ⭐⭐⭐⭐⭐

Wrong:

```jsx
case "added":
  state.items.push(
    action.item
  );

  return state;
```

Correct:

```jsx
case "added":
  return {
    ...state,
    items: [
      ...state.items,
      action.item,
    ],
  };
```

Reducers follow the same immutability rules as normal React state updates.

---

## 9. Array CRUD in a Reducer

### Add

```jsx
case "added":
  return [
    ...state,
    action.item,
  ];
```

### Update

```jsx
case "changed":
  return state.map(
    (item) =>
      item.id ===
      action.item.id
        ? action.item
        : item
  );
```

### Delete

```jsx
case "deleted":
  return state.filter(
    (item) =>
      item.id !== action.id
  );
```

These are the same immutable transformations from Lesson 15.

---

## 10. One Action Should Describe One Interaction

Suppose resetting a form changes five fields.

Prefer:

```jsx
dispatch({
  type: "form_reset",
});
```

rather than five separate implementation-oriented actions.

Why?

The action log should tell a story:

```text
user submitted form
user reset form
user changed status
```

Actions describe events, not individual assignment instructions.

---

## 11. useState vs useReducer ⭐⭐⭐⭐⭐

### useState

Best when:

- state is simple
- update logic is small
- transitions are obvious
- only a few setters exist

### useReducer

Useful when:

- state logic is complex
- many handlers update related state
- transitions benefit from centralization
- you want explicit action semantics
- reducer logic deserves isolated testing

Do not use a reducer just because it looks more advanced.

---

## 12. Refactoring useState to useReducer

Before:

```jsx
function handleDelete(id) {
  setApplications(
    applications.filter(
      (application) =>
        application.id !== id
    )
  );
}
```

After:

```jsx
function handleDelete(id) {
  dispatch({
    type:
      "application_deleted",
    id,
  });
}
```

Reducer:

```jsx
case "application_deleted":
  return state.filter(
    (application) =>
      application.id !==
      action.id
  );
```

The component says what happened.

The reducer contains update mechanics.

---

## 13. Reducer with Object State

```jsx
const initialState = {
  applications: [],
  selectedId: null,
  status: "idle",
};
```

Reducer:

```jsx
function reducer(
  state,
  action
) {
  switch (action.type) {
    case "selected":
      return {
        ...state,
        selectedId:
          action.id,
      };

    default:
      throw new Error(
        "Unknown action"
      );
  }
}
```

Always preserve unchanged properties when returning partial object changes.

---

## 14. Avoid Redundant State Even with Reducers

Bad:

```js
{
  applications: [...],
  selectedId: 5,
  selectedApplication: {...}
}
```

If selected application can be derived:

```jsx
const selectedApplication =
  state.applications.find(
    (item) =>
      item.id ===
      state.selectedId
  );
```

A reducer does not excuse poor state structure.

All earlier state-design principles still apply.

---

## 15. Avoid Contradictory State

Instead of:

```js
{
  isLoading: true,
  isSuccess: true,
  isError: true
}
```

consider:

```js
{
  status: "loading"
}
```

Possible values:

```text
idle
loading
success
error
```

Reducers are especially useful when state transitions resemble a state machine.

---

## 16. Lazy Initialization

`useReducer` supports an initializer:

```jsx
const [
  state,
  dispatch,
] = useReducer(
  reducer,
  initialArg,
  createInitialState
);
```

React calls the initializer to produce the initial state.

Useful when initial state creation is expensive or needs transformation.

Do not use lazy initialization unless it solves a real initialization need.

---

## 17. dispatch Identity

React provides a stable `dispatch` function identity.

You can pass it to descendants:

```jsx
<Toolbar
  dispatch={dispatch}
/>
```

or later through Context.

This makes `dispatch` a useful update interface for complex shared state.

---

## 18. Reducer and Event Handlers

Good separation:

```jsx
function handleArchive(id) {
  dispatch({
    type:
      "application_archived",
    id,
  });
}
```

Reducer:

```jsx
case "application_archived":
  return state.map(
    (application) =>
      application.id ===
      action.id
        ? {
            ...application,
            archived: true,
          }
        : application
  );
```

The event handler stays small and intention-focused.

---

## 19. Where Do Async Requests Go? ⭐⭐⭐⭐⭐

Not inside the reducer.

Instead:

```text
event / Action / Effect
       ↓
perform async work
       ↓
dispatch result action
       ↓
reducer updates state
```

Example:

```jsx
async function handleSave() {
  dispatch({
    type: "save_started",
  });

  try {
    const result =
      await saveData();

    dispatch({
      type: "save_succeeded",
      result,
    });
  } catch (error) {
    dispatch({
      type: "save_failed",
      message:
        "Unable to save",
    });
  }
}
```

Reducer remains pure.

---

## 20. Reducer State Transition Example

```text
idle
 │ SAVE
 ↓
saving
 ├── SUCCESS → success
 └── FAILURE → error
```

Actions:

```text
save_started
save_succeeded
save_failed
```

Reducer centralizes valid transitions.

---

## 21. CareerLoop Reducer ⭐⭐⭐⭐⭐

```jsx
function applicationsReducer(
  applications,
  action
) {
  switch (action.type) {
    case "added":
      return [
        ...applications,
        action.application,
      ];

    case "status_changed":
      return applications.map(
        (application) =>
          application.id ===
          action.id
            ? {
                ...application,
                status:
                  action.status,
              }
            : application
      );

    case "deleted":
      return applications.filter(
        (application) =>
          application.id !==
          action.id
      );

    default:
      throw new Error(
        "Unknown action: " +
          action.type
      );
  }
}
```

Component:

```jsx
const [
  applications,
  dispatch,
] = useReducer(
  applicationsReducer,
  initialApplications
);
```

---

## 22. Reducer Testing Benefit

Because a reducer is a pure function:

```js
const nextState =
  reducer(
    initialState,
    action
  );
```

you can test state transition logic independently.

Example mental test:

```text
Given:
applications = [A, B]

When:
{ type: "deleted", id: B.id }

Expect:
[A]
```

This is valuable when transition rules become complex.

---

## 23. Debugging Benefit

Actions can make debugging easier:

```text
application_added
status_changed
application_archived
application_deleted
```

You can reason:

```text
previous state
+
event
=
next state
```

This is often clearer than searching through many scattered setter calls.

---

## 24. Reducer File Organization

For a small component:

```text
ApplicationList.jsx
  ├── component
  └── reducer
```

For larger logic:

```text
applications/
  ├── ApplicationsPage.jsx
  └── applicationsReducer.js
```

Do not split files merely to create architecture. Extract when it improves readability, testing, or reuse.

---

## 25. Strict Mode and Reducers

In development Strict Mode, React may call reducer/initializer logic extra times to help detect impurities.

A pure reducer remains safe.

If duplicate execution causes incorrect behavior, that often reveals that the reducer contains mutation or side effects.

---

## 26. Reducer Is Not Redux

Both use reducer ideas, but:

```text
useReducer
→ built-in React Hook
→ component/subtree state

Redux
→ external state-management ecosystem
→ store, tooling, subscriptions, middleware, etc.
```

Knowing `useReducer` makes reducer-based libraries easier to understand, but they are not the same thing.

---

## 27. Common Mistakes ⭐⭐⭐⭐⭐

1. Mutating reducer state.
2. Performing API calls inside the reducer.
3. Using vague action types.
4. Creating too many tiny implementation-level actions.
5. Using `useReducer` for trivial state with no benefit.
6. Returning only one object property and accidentally deleting the others.
7. Storing redundant/duplicate data.
8. Silently returning state for unknown action types during development.
9. Putting event-specific side effects in the reducer.
10. Assuming `useReducer` makes state global.

---

## 28. Interview Questions ⭐⭐⭐⭐⭐

### What is useReducer?

A React Hook for managing state by dispatching actions to a reducer function that calculates the next state.

### What is a reducer?

A pure function receiving current state and an action and returning next state.

### What is dispatch?

A function used to send an action to the reducer.

### What is an action?

A description of what happened, usually represented by an object with a `type` and optional payload.

### useState vs useReducer?

Use `useState` for simpler state. Consider `useReducer` when related state transitions and update logic become complex or scattered.

### Can a reducer call an API?

No. Reducers should be pure.

### Can a reducer mutate state?

No. Return new objects/arrays for changed state.

### Does useReducer make state global?

No. The state still belongs to the component using the Hook unless distributed elsewhere.

### Why are reducers testable?

Because a pure reducer can be called with known state/action inputs and its returned state asserted independently.

### Is useReducer the same as Redux?

No.

---

## 29. Complete Mental Model ⭐⭐⭐⭐⭐

```text
        USER EVENT
            │
            ↓
     dispatch(action)
            │
            ↓
┌─────────────────────────┐
│ reducer(state, action)  │
│                         │
│ pure transition logic   │
└───────────┬─────────────┘
            ↓
        next state
            ↓
          render
```

Async work:

```text
event
 ↓
async request
 ↓
dispatch(result action)
 ↓
reducer
 ↓
state
```

---

## 30. Quick Revision

```jsx
const [
  state,
  dispatch,
] = useReducer(
  reducer,
  initialState
);
```

```jsx
dispatch({
  type: "deleted",
  id,
});
```

```jsx
function reducer(
  state,
  action
) {
  switch (action.type) {
    case "deleted":
      return state.filter(
        (item) =>
          item.id !==
          action.id
      );

    default:
      throw new Error(
        "Unknown action"
      );
  }
}
```

Remember:

```text
event → action
action → reducer
reducer → next state
```

---

## 31. Key Takeaways

- `useReducer` centralizes complex state update logic.
- It returns current state and `dispatch`.
- Actions describe what happened.
- Reducers calculate how state changes.
- Reducers must be pure.
- Never mutate reducer state.
- Keep side effects outside reducers.
- One action should generally represent one meaningful interaction.
- Good state structure still matters with reducers.
- Avoid redundant and contradictory state.
- Reducers can improve debugging and isolated testing.
- `useReducer` is not automatically better than `useState`.
- `useReducer` does not make state global.
- Reducer + Context is a powerful next step when distant descendants need state and dispatch.

---

## Next Lesson

➡️ [Lesson 32 — useReducer + Context Architecture](./32-reducer-context.md)
