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

function AddTask() {
  const dispatch = useTasksDispatch();

  function handleAdd() {
    dispatch({
      type: "added",
      task: { id: crypto.randomUUID(), title: "Learn React" }
    });
  }

  return <button onClick={handleAdd}>Add task</button>;
}

function TaskApp() {
  return (
    <TasksProvider>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

Any descendant can call `useTasks()` to read tasks and `useTasksDispatch()` to update them.

## Explanation — Step by Step

### 1. Why combine them?

Imagine `TaskApp → Dashboard → TaskList → DeleteButton`. Without Context, the parent must pass `tasks` and `dispatch` through intermediate components (**prop drilling**).

- **`useReducer`** manages tasks and decides how they change.
- **Context** makes tasks and dispatch accessible to deeply nested descendants.
- **`dispatch`** sends a request such as `{ type: "deleted", id: 2 }`.

### 2. Why create two contexts?

```jsx
const TasksContext = createContext(null);         // Current tasks
const TasksDispatchContext = createContext(null); // Update function
```

`TasksContext` is for components that **read** tasks. `TasksDispatchContext` is for components that **send actions**. Separating them means a dispatch-only consumer does not subscribe to task state changes.

### 3. What does TasksProvider do?

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, []);
```

The provider **owns** the state. It supplies `tasks` through `TasksContext` and `dispatch` through `TasksDispatchContext`. Its `children` are the components wrapped by `<TasksProvider>...</TasksProvider>`.

React 19 lets you write `<TasksContext value={tasks}>` instead of the older `<TasksContext.Provider value={tasks}>`.

### 4. Why use custom hooks?

`useTasks()` is a short, reusable way to call `useContext(TasksContext)`. `useTasksDispatch()` does the same for dispatch. Both throw a helpful error if used without the provider.

### 5. What happens when Delete is clicked?

```text
Click Delete
  ↓
dispatch({ type: "deleted", id: task.id })
  ↓
tasksReducer(currentTasks, action)
  ↓
Returns a new array without that task
  ↓
TasksProvider receives updated tasks
  ↓
TaskList reads the new context value and re-renders
```

For **Add**, the `AddTask` component dispatches `{ type: "added", task: ... }`, and the reducer returns `[...tasks, action.task]`.

The original example started with `[]` and only had a Delete button, so nothing appeared initially. The example above now includes **AddTask**, making the add/delete flow usable.

## Benefits and When to Use It

1. **Avoid prop drilling:** Descendants read state and dispatch directly.
2. **Centralize updates:** The reducer contains add/delete logic in one place.
3. **Cleaner components:** UI components dispatch meaningful actions instead of repeating array updates.
4. **Reusable access:** Custom hooks provide a simple API.
5. **Separate readers and updaters:** Dispatch-only components need not subscribe to changes in `TasksContext`.

**Do not overuse it.** For a single small component, `useState` or a local `useReducer` is usually simpler. This architecture is helpful when complex state is shared across multiple deeply nested components.

---

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
