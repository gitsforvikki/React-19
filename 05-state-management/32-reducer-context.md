# Lesson 32 — useReducer + Context Architecture

## Why Combine Them?

- `useReducer` owns state and centralizes updates.
- Context shares `state` and `dispatch` with deeply nested components.

Use this pattern for **complex state shared inside one feature**, not for every component.

```text
FeatureProvider (useReducer)
  ├─ StateContext → readers
  └─ DispatchContext → buttons/forms → dispatch(action) → reducer
```

## Practical Example — Task Feature

```jsx
import { createContext, useContext, useReducer } from "react";

const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);

function tasksReducer(tasks, action) {
  switch (action.type) {
    case "added":
      return [...tasks, action.task];
    case "deleted":
      return tasks.filter(task => task.id !== action.id);
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, []);

  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        {children}
      </TasksDispatchContext>
    </TasksContext>
  );
}

function useTasks() {
  const tasks = useContext(TasksContext);
  if (tasks === null) throw new Error("Missing TasksProvider");
  return tasks;
}

function useTasksDispatch() {
  const dispatch = useContext(TasksDispatchContext);
  if (dispatch === null) throw new Error("Missing TasksProvider");
  return dispatch;
}

function TaskList() {
  const tasks = useTasks();
  const dispatch = useTasksDispatch();

  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          {task.title}
          <button onClick={() => dispatch({ type: "deleted", id: task.id })}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}

function TaskApp() {
  return (
    <TasksProvider>
      <TaskList />
    </TasksProvider>
  );
}
```

Any descendant can call `useTasks()` to read tasks and `useTasksDispatch()` to update them.

## Design Rules

- Keep the provider **close to the feature** that owns the state.
- Separate state and dispatch contexts when helpful. A dispatch-only consumer need not subscribe to state context changes; `dispatch` is stable.
- Expose custom hooks instead of repeating raw `useContext` everywhere.
- Keep reducers pure. API calls belong in event handlers, actions, or a data layer.
- Context consumers reading a large changing state value still re-render when that value changes. Split state by responsibility if needed.

## Common Mistakes

- Putting every feature into one giant reducer/provider.
- Using reducer + Context when `useState` and props are enough.
- Assuming separate contexts eliminate *all* unnecessary rendering.
- Treating the shared reducer as a server-data cache.

## Interview Quick Check

**Why reducer + Context?** Reducer manages complex state transitions; Context delivers state and dispatch across a subtree.

**Why split contexts?** Dispatch-only components avoid subscribing to changes in the state context.

**Is this global state?** Not necessarily; it is scoped to its provider's descendants.

---

➡️ [Lesson 33 — When to Use Local State, Context or External State ⭐⭐⭐⭐⭐](./33-state-management-decisions.md)
