# Lesson 32 — useReducer + Context Architecture

## 1. Why Combine Reducer and Context?

Lesson 31 solved:

> How do I centralize complex state transition logic?

`useReducer`.

Lesson 30 solved:

> How do I make a value available to distant descendants?

Context.

Combine them:

```text
useReducer
→ manages complex state

Context
→ distributes state + dispatch
```

This is a powerful architecture for state shared across a complex screen or feature subtree.

---

## 2. The Problem Before Context

```jsx
function App() {
  const [
    tasks,
    dispatch,
  ] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <Page
      tasks={tasks}
      dispatch={dispatch}
    />
  );
}
```

Then:

```text
App
 ↓ tasks + dispatch
Page
 ↓ tasks + dispatch
Panel
 ↓ tasks + dispatch
TaskList
 ↓
Task
```

Intermediate components may only forward values.

Context removes that repetitive distribution.

---

## 3. Target Architecture ⭐⭐⭐⭐⭐

```text
FeatureProvider
      │
      ├── useReducer
      │      ├── state
      │      └── dispatch
      │
      ├── StateContext
      │
      └── DispatchContext
             │
     ┌───────┴────────┐
     ↓                ↓
  Reader           Updater
 useStateCtx      useDispatchCtx
```

The Provider becomes the feature's state boundary.

---

## 4. Step 1 — Create the Reducer

```jsx
function tasksReducer(
  tasks,
  action
) {
  switch (action.type) {
    case "added":
      return [
        ...tasks,
        action.task,
      ];

    case "changed":
      return tasks.map(
        (task) =>
          task.id ===
          action.task.id
            ? action.task
            : task
      );

    case "deleted":
      return tasks.filter(
        (task) =>
          task.id !==
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

Reducer stays pure.

---

## 5. Step 2 — Create Contexts ⭐⭐⭐⭐⭐

```jsx
import {
  createContext,
} from "react";

const TasksContext =
  createContext(null);

const TasksDispatchContext =
  createContext(null);
```

One Context carries state.

One carries dispatch.

---

## 6. Why Two Contexts? ⭐⭐⭐⭐⭐

You could write:

```jsx
const TasksContext =
  createContext(null);
```

and provide:

```jsx
{
  tasks,
  dispatch,
}
```

That is valid.

But separating:

```text
TasksContext
TasksDispatchContext
```

creates clearer responsibilities.

A component that only needs dispatch can consume only the dispatch context.

It also gives you flexibility to optimize/change state distribution independently later.

Do not treat two contexts as a mandatory law—it is a useful architecture.

---

## 7. Step 3 — Build the Provider ⭐⭐⭐⭐⭐

```jsx
import {
  useReducer,
} from "react";

function TasksProvider({
  children,
}) {
  const [
    tasks,
    dispatch,
  ] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext
      value={tasks}
    >
      <TasksDispatchContext
        value={dispatch}
      >
        {children}
      </TasksDispatchContext>
    </TasksContext>
  );
}
```

React 19 provider syntax is used here.

---

## 8. Step 4 — Wrap the Feature

```jsx
function TaskApp() {
  return (
    <TasksProvider>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

Now descendants do not need state/dispatch prop drilling.

---

## 9. Step 5 — Read State

```jsx
function TaskList() {
  const tasks =
    useContext(
      TasksContext
    );

  return (
    <ul>
      {tasks.map(
        (task) => (
          <Task
            key={task.id}
            task={task}
          />
        )
      )}
    </ul>
  );
}
```

---

## 10. Step 6 — Dispatch Updates

```jsx
function DeleteButton({
  id,
}) {
  const dispatch =
    useContext(
      TasksDispatchContext
    );

  return (
    <button
      onClick={() =>
        dispatch({
          type: "deleted",
          id,
        })
      }
    >
      Delete
    </button>
  );
}
```

Flow:

```text
DeleteButton
    ↓
dispatch(action)
    ↓
Provider's reducer
    ↓
new tasks state
    ↓
StateContext value
    ↓
state consumers update
```

---

## 11. Custom Hooks Improve the API ⭐⭐⭐⭐⭐

```jsx
function useTasks() {
  const context =
    useContext(
      TasksContext
    );

  if (context === null) {
    throw new Error(
      "useTasks must be used inside TasksProvider"
    );
  }

  return context;
}
```

Dispatch Hook:

```jsx
function useTasksDispatch() {
  const context =
    useContext(
      TasksDispatchContext
    );

  if (context === null) {
    throw new Error(
      "useTasksDispatch must be used inside TasksProvider"
    );
  }

  return context;
}
```

Consumers become:

```jsx
const tasks = useTasks();

const dispatch =
  useTasksDispatch();
```

---

## 12. Encapsulate Wiring in One Module

A feature can expose:

```text
TasksProvider
useTasks()
useTasksDispatch()
```

while keeping internal details private:

```text
TasksContext
TasksDispatchContext
tasksReducer
```

This reduces coupling.

Consumers depend on the feature API rather than knowing exactly how Context is wired.

---

## 13. Full Data Flow ⭐⭐⭐⭐⭐

```text
             TasksProvider
                  │
             useReducer
            ┌─────┴─────┐
            ↓           ↓
          tasks      dispatch
            │           │
            ↓           ↓
    TasksContext   DispatchContext
            │           │
            ↓           ↓
        TaskList    DeleteButton
                         │
                         ↓
                 dispatch(action)
                         │
                         └───────┐
                                 ↓
                              reducer
                                 ↓
                             next tasks
                                 ↓
                           StateContext
```

This is one-way React data flow, not a separate paradigm.

---

## 14. CareerLoop Architecture ⭐⭐⭐⭐⭐

Suppose a job-tracking screen contains:

```text
ApplicationsPage
 ├── AddApplicationForm
 ├── FilterBar
 ├── ApplicationList
 │    └── ApplicationCard
 │         ├── StatusMenu
 │         └── DeleteButton
 └── ApplicationSummary
```

Many descendants need applications or need to update them.

Architecture:

```text
ApplicationsProvider
       │
       ├── applicationsReducer
       ├── ApplicationsContext
       └── ApplicationsDispatchContext
```

State readers:

- ApplicationList
- ApplicationSummary

Dispatch-only consumers:

- AddApplicationForm
- StatusMenu
- DeleteButton

This is a strong feature-scoped use case.

---

## 15. Example CareerLoop Reducer

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

---

## 16. Feature Provider

```jsx
function ApplicationsProvider({
  children,
}) {
  const [
    applications,
    dispatch,
  ] = useReducer(
    applicationsReducer,
    initialApplications
  );

  return (
    <ApplicationsContext
      value={applications}
    >
      <ApplicationsDispatchContext
        value={dispatch}
      >
        {children}
      </ApplicationsDispatchContext>
    </ApplicationsContext>
  );
}
```

This keeps ownership near the feature rather than automatically at application root.

---

## 17. Do Not Put Every Provider at App Root ⭐⭐⭐⭐⭐

Avoid:

```text
App
 └── Provider
      └── Provider
           └── Provider
                └── Provider
                     └── everything
```

if those values are only needed by small feature subtrees.

Prefer:

```text
App
 ├── Header
 └── ApplicationsPage
      └── ApplicationsProvider
           └── applications feature
```

Rule:

> Place a provider as low as possible while still covering all consumers.

This is state colocation applied to shared state.

---

## 18. Context Does Not Replace the Reducer

Context:

```text
transports value
```

Reducer:

```text
defines state transitions
```

They solve different problems.

You can use:

- Context without reducer
- reducer without Context
- reducer + Context

Choose based on requirements.

---

## 19. Reducer + Context Is Not Automatically Global State

If:

```jsx
<ApplicationsProvider>
  <ApplicationsPage />
</ApplicationsProvider>
```

then the state belongs to that subtree.

It does not become universally global just because Context is involved.

Scope remains architectural.

---

## 20. Side Effects Stay Outside the Reducer ⭐⭐⭐⭐⭐

Bad:

```jsx
case "deleted":
  await fetch(
    "/api/application"
  );

  return ...
```

Reducers cannot be async and should not perform external synchronization.

Instead:

```jsx
async function handleDelete(
  id
) {
  await deleteApplication(
    id
  );

  dispatch({
    type: "deleted",
    id,
  });
}
```

Or use an appropriate Action/data layer and dispatch the resulting event.

---

## 21. Where Should Async Logic Live?

Possible locations include:

- event handlers
- Actions
- custom Hooks
- data/service layer
- Effects when synchronizing with external systems

Then:

```text
external operation result
       ↓
dispatch(action)
       ↓
pure reducer
```

Do not turn the reducer into a service layer.

---

## 22. Context Performance Considerations ⭐⭐⭐⭐⭐

When the state Context value changes, components consuming it may render.

Splitting state and dispatch helps because `dispatch` itself has stable identity and dispatch-only consumers do not need to subscribe to state Context.

But this does not automatically solve every performance issue.

If a very large state object changes frequently, you may need:

- better state colocation
- smaller contexts
- feature boundaries
- external stores with selector-based subscriptions
- measured optimization

Architecture first, memoization second.

---

## 23. Split by Domain, Not Randomly

Good:

```text
ApplicationsProvider
AuthProvider
ThemeProvider
```

Potentially poor:

```text
StringContext
BooleanContext
ArrayContext
```

Context boundaries should represent meaningful responsibilities, not JavaScript data types.

---

## 24. Provider Composition

Multiple providers are normal:

```jsx
<AuthProvider>
  <ThemeProvider>
    <ApplicationsProvider>
      <App />
    </ApplicationsProvider>
  </ThemeProvider>
</AuthProvider>
```

But each provider should have a justified scope.

If provider nesting becomes hard to reason about, review whether state has been lifted too high or responsibilities are too broad.

---

## 25. Testing Architecture

Reducer:

```text
test pure transitions independently
```

Provider/consumers:

```text
test through realistic rendered behavior
```

Custom Hooks:

```text
ensure missing provider errors
and expected state/dispatch access
```

Separation makes responsibilities easier to test.

---

## 26. When This Pattern Is a Good Fit ⭐⭐⭐⭐⭐

Use reducer + Context when:

- a feature has complex transitions
- many distant descendants read/update the state
- prop drilling is becoming noisy
- state belongs to a React subtree
- built-in React primitives are sufficient

Examples:

- complex editor screen
- multi-step form workflow
- task manager
- shopping-cart UI state
- job application management screen

---

## 27. When It May Not Be Enough

Consider other approaches when requirements include:

- many unrelated app areas subscribing to small slices of a huge store
- sophisticated selector/subscription behavior
- complex external event sources
- specialized persistence/devtools/middleware
- server cache synchronization requirements

Do not choose a library simply because an app is "large"; choose it because the requirements justify it.

Lesson 33 gives the decision framework.

---

## 28. Common Mistakes ⭐⭐⭐⭐⭐

1. Putting API calls inside reducers.
2. Putting all application state into one reducer.
3. Putting every provider at the root.
4. Creating one giant Context value.
5. Confusing state management with state distribution.
6. Exposing raw contexts everywhere when custom Hooks would provide a cleaner API.
7. Mutating reducer state.
8. Storing derived values in reducer state.
9. Assuming Context + reducer automatically solves performance.
10. Using the pattern for trivial local component state.

---

## 29. Interview Questions ⭐⭐⭐⭐⭐

### Why combine useReducer and Context?

`useReducer` centralizes complex state transitions, while Context makes state and dispatch available to distant descendants without prop drilling.

### Why might state and dispatch use separate contexts?

To separate read and update responsibilities and allow dispatch-only consumers to avoid subscribing to the state Context.

### Does reducer + Context create global state?

Not necessarily. It is scoped to the provider subtree.

### Where should the Provider live?

As low as possible while still containing all components that need the shared state.

### Can the reducer make API requests?

No. Reducers must remain pure.

### What should actions describe?

Meaningful events/interactions that occurred.

### Is reducer + Context always better than useState?

No. It adds structure and indirection; use it when complexity and sharing justify it.

### How can you improve the Context API?

Wrap Context access in custom Hooks such as `useApplications` and `useApplicationsDispatch`.

### Does splitting state and dispatch contexts solve all performance issues?

No. It helps separate subscriptions, but state scope and update frequency still matter.

### When might an external store be useful?

When requirements call for more specialized subscriptions/selectors, tooling, persistence, or broad state coordination than the built-in pattern comfortably provides.

---

## 30. Complete Mental Model ⭐⭐⭐⭐⭐

```text
          FEATURE PROVIDER
                 │
            useReducer
         ┌───────┴────────┐
         ↓                ↓
       state           dispatch
         │                │
 StateContext       DispatchContext
         │                │
     consumers         consumers
         │                │
         │           dispatch(action)
         │                │
         └─────── reducer ◄┘
                    │
                    ↓
                 new state
                    │
                    └──→ StateContext
```

---

## 31. Key Takeaways

- Reducer and Context solve different problems.
- Reducer centralizes transitions; Context distributes values.
- Combining them is useful for complex shared feature state.
- Keep reducers pure and immutable.
- State and dispatch can be provided through separate contexts.
- Custom Hooks create cleaner consumer APIs.
- Providers should be scoped near their consumers.
- Reducer + Context does not automatically mean global state.
- Async work belongs outside the reducer.
- Do not store redundant derived data in reducer state.
- Splitting state/dispatch can improve architectural separation.
- Context performance depends heavily on scope and value changes.
- Built-in React state management is often sufficient for feature-level complexity.
- External stores are requirements-driven, not an automatic next step.

---

## Next Lesson

➡️ [Lesson 33 — When to Use Local State, Context or External State ⭐⭐⭐⭐⭐](./33-state-management-decisions.md)
