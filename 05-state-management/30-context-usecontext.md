# Lesson 30 — Context API and useContext ⭐⭐⭐⭐⭐

## 1. The Problem Context Solves

React normally passes data from parent to child through props.

```text
App
 ↓ theme
Layout
 ↓ theme
Sidebar
 ↓ theme
Button
```

If `Layout` and `Sidebar` do not use `theme` but only forward it, this becomes **prop drilling**.

Context lets an ancestor make a value available to descendants without explicitly forwarding the same prop through every intermediate component.

```text
Theme provider
      │
      ├───────────────┐
      ↓               ↓
   Sidebar          Content
      │               │
      ↓               ↓
   Button           Card
      └──── read context ────
```

Context is about **data distribution through a component subtree**.

---

## 2. Props Are Still the Default

Context is not a replacement for props.

Props are:

- explicit
- easy to trace
- reusable
- excellent for nearby parent-child communication

Use Context when information is genuinely needed by distant descendants or many components in a subtree.

Before Context, consider:

1. ordinary props
2. component composition / `children`
3. lifting state only as high as necessary

Then use Context when repeated deep passing becomes a real problem.

---

## 3. The Three Steps ⭐⭐⭐⭐⭐

Context has three core steps:

```text
1. create
2. provide
3. consume
```

### Create

```jsx
import { createContext } from "react";

export const ThemeContext =
  createContext("light");
```

### Provide — React 19 style

```jsx
<ThemeContext value="dark">
  <App />
</ThemeContext>
```

### Consume

```jsx
import { useContext } from "react";

function Button() {
  const theme =
    useContext(ThemeContext);

  return (
    <button>
      Theme: {theme}
    </button>
  );
}
```

---

## 4. React 19 Provider Syntax ⭐⭐⭐⭐⭐

In React 19, the context object itself can be rendered as the provider:

```jsx
<ThemeContext value={theme}>
  {children}
</ThemeContext>
```

Older code commonly uses:

```jsx
<ThemeContext.Provider
  value={theme}
>
  {children}
</ThemeContext.Provider>
```

You should recognize both.

For this React 19 repository, prefer the modern syntax in new examples.

---

## 5. Default Value

```jsx
const ThemeContext =
  createContext("light");
```

The default value is used when a component reads that context without a matching provider above it.

```jsx
function Button() {
  const theme =
    useContext(ThemeContext);

  return <p>{theme}</p>;
}
```

If there is no provider:

```text
theme = "light"
```

Important:

> The default value is a fallback. It does not dynamically change.

---

## 6. Nearest Provider Wins ⭐⭐⭐⭐⭐

Context is scoped by the component tree.

```jsx
<ThemeContext value="light">
  <Header />

  <ThemeContext value="dark">
    <AdminPanel />
  </ThemeContext>
</ThemeContext>
```

Inside `AdminPanel`:

```text
nearest ThemeContext
        ↓
      "dark"
```

Inside `Header`:

```text
nearest ThemeContext
        ↓
      "light"
```

This allows nested parts of the UI to override context.

---

## 7. Context Passes Through Intermediate Components

```text
Provider
   ↓
Layout
   ↓
Page
   ↓
Section
   ↓
Button
```

`Layout`, `Page`, and `Section` do not need to receive or forward the value.

`Button` can directly read:

```jsx
const theme =
  useContext(ThemeContext);
```

This is the core benefit.

---

## 8. Context with State ⭐⭐⭐⭐⭐

Context itself does not magically create state.

You commonly combine it with state:

```jsx
function ThemeProvider({
  children,
}) {
  const [theme, setTheme] =
    useState("light");

  return (
    <ThemeContext
      value={{
        theme,
        setTheme,
      }}
    >
      {children}
    </ThemeContext>
  );
}
```

Flow:

```text
useState
   ↓
provider value
   ↓
Context
   ↓
consumers
```

When the provided value changes, components reading that context receive the new value and React re-renders them as needed.

---

## 9. Context Does Not Mean Global State ⭐⭐⭐⭐⭐

A common misconception:

```text
Context = global state
```

Not necessarily.

Context is scoped to descendants of a provider.

```text
Provider A
 └── subtree A

Provider B
 └── subtree B
```

The same Context can have different values in different subtrees.

Better mental model:

> Context is a dependency/data channel through a React subtree.

---

## 10. Useful Context Examples

Good candidates include:

- theme
- current account/user information needed broadly
- locale
- accessibility preferences
- routing-related information
- complex screen state shared by distant descendants
- state + dispatch from a reducer

But context should be driven by **scope**, not by the fact that data exists.

---

## 11. Prop Drilling Is Not Automatically Bad

This is completely reasonable:

```text
Page
 ↓ filter
FilterPanel
 ↓ filter
FilterInput
```

Two or three explicit prop levels may be clearer than Context.

Context becomes valuable when many intermediate components merely forward data they do not use, or many distant descendants require the same dependency.

---

## 12. Composition Can Avoid Context

Instead of:

```jsx
<Layout user={user} />
```

and forwarding `user` deeply, sometimes you can compose:

```jsx
<Layout>
  <UserPanel user={user} />
</Layout>
```

The component that knows the data can create the child that needs it.

So the decision is not:

```text
props vs Context only
```

It can also include component composition.

---

## 13. Context and State Ownership

Suppose:

```text
ApplicationsPage
     ↓ owns filters
 FilterContext
   /       \
FilterBar  ApplicationList
```

The page may still **own** the state.

Context only changes how that state reaches consumers.

This distinction is crucial:

```text
state ownership
≠
state distribution
```

Context primarily helps with distribution.

---

## 14. Reading Context Is Reactive ⭐⭐⭐⭐⭐

```jsx
const value =
  useContext(MyContext);
```

The component subscribes to that context value.

When the provider supplies a changed value, consumers reading it can re-render.

This means Context is not merely a one-time lookup.

---

## 15. Provider Value Identity ⭐⭐⭐⭐⭐

Consider:

```jsx
<AuthContext
  value={{
    user,
    logout,
  }}
>
  {children}
</AuthContext>
```

An object literal creates a new object on every provider render.

For many applications this is perfectly acceptable.

If profiling shows context propagation is causing meaningful unnecessary work, you can consider stabilizing provider values or splitting contexts.

Do not add memoization automatically without evidence.

---

## 16. Split Context by Responsibility

Instead of one huge context:

```js
{
  user,
  theme,
  cart,
  notifications,
  filters,
  modal,
  ...
}
```

prefer cohesive responsibilities.

For example:

```text
AuthContext
ThemeContext
ApplicationFiltersContext
```

Why?

Because unrelated data:

- changes for different reasons
- has different consumers
- has different ownership/scope

A giant Context often creates unnecessary coupling.

---

## 17. Separate State and Update Contexts

For complex shared state, a useful pattern is:

```text
StateContext
DispatchContext
```

Components that only dispatch actions do not need the state value from the same context object.

You will implement this properly in Lesson 32 with `useReducer`.

---

## 18. Custom Hook Around Context

Instead of repeating:

```jsx
useContext(AuthContext)
```

everywhere:

```jsx
export function useAuth() {
  const context =
    useContext(AuthContext);

  if (context === null) {
    throw new Error(
      "useAuth must be used inside AuthProvider"
    );
  }

  return context;
}
```

Consumer:

```jsx
const {
  user,
  logout,
} = useAuth();
```

Benefits:

- clearer API
- provider validation
- implementation can evolve
- consumers do not need context details

Custom Hooks are covered deeply in Section 6.

---

## 19. Why null Is Often a Useful Default

```jsx
const AuthContext =
  createContext(null);
```

Then a custom Hook can detect missing provider:

```jsx
if (context === null) {
  throw new Error(
    "Missing AuthProvider"
  );
}
```

This can catch configuration mistakes early.

For contexts with a meaningful natural fallback, another default may be appropriate.

---

## 20. Context Does Not Bypass One-Way Data Flow

Context still flows from ancestor to descendant:

```text
Provider value
      ↓
descendant consumers
```

A consumer does not directly mutate the provider.

If updates are required, provide an update function:

```jsx
value={{
  theme,
  setTheme,
}}
```

or a reducer `dispatch`.

So the mental model remains:

```text
data ↓
actions/events ↑ through functions
```

---

## 21. CareerLoop Example

Suppose both `FilterBar` and deeply nested application components need filters.

```jsx
const FiltersContext =
  createContext(null);

function ApplicationsProvider({
  children,
}) {
  const [filters, setFilters] =
    useState({
      status: "all",
      search: "",
    });

  return (
    <FiltersContext
      value={{
        filters,
        setFilters,
      }}
    >
      {children}
    </FiltersContext>
  );
}
```

Tree:

```text
ApplicationsProvider
      │
      ├── FilterBar
      │
      └── ApplicationArea
             ↓
        ApplicationList
             ↓
        ApplicationCard
```

Any required descendant can read the filters without every intermediate component forwarding them.

---

## 22. Context Performance Mental Model ⭐⭐⭐⭐⭐

Suppose:

```text
Context value changes
       ↓
components consuming
that Context may render
```

Therefore avoid using one rapidly changing Context for a huge unrelated subtree merely for convenience.

Possible design improvements:

- colocate state lower
- split contexts
- separate state and dispatch
- compose components differently
- only optimize identity after profiling

Context is not inherently slow. Poor scope and overly broad values are the usual architectural problem.

---

## 23. Context vs Props

Use props when:

- parent-child relationship is direct
- only a few levels need the data
- explicit dependencies improve clarity
- component should be reusable with different values

Use Context when:

- many distant descendants need the same dependency
- intermediate components should not care about it
- the value conceptually belongs to a subtree/environment

---

## 24. Context vs External State

Context answers:

> How can descendants access a value without explicit prop passing?

It does not automatically provide:

- specialized selectors
- normalized entity storage
- advanced devtools
- external subscriptions
- persistence
- server-data caching

Those are separate concerns.

Do not choose Context or an external library based only on app size. Choose based on requirements.

---

## 25. Common Mistakes ⭐⭐⭐⭐⭐

1. Putting every piece of state into Context.
2. Treating Context as automatically global.
3. Using Context to avoid simple, clear props.
4. Creating one giant application context.
5. Forgetting that consumers re-render when their consumed value changes.
6. Copying context values into local state unnecessarily.
7. Mutating an object supplied through Context.
8. Using Context to hide unclear state ownership.
9. Memoizing every provider value without measuring.
10. Assuming Context itself manages state.

---

## 26. Interview Questions ⭐⭐⭐⭐⭐

### What is React Context?

A way for an ancestor to provide a value to descendants without explicitly passing that value through every intermediate component.

### What is prop drilling?

Passing props through components that do not need the data themselves only so deeper descendants can receive it.

### Does Context replace props?

No. Props remain the normal explicit communication mechanism.

### What does useContext do?

It reads and subscribes a component to the nearest matching Context provider above it.

### Which provider value is used?

The value from the nearest matching provider above the consumer.

### What happens without a provider?

The default value passed to `createContext` is used.

### Does Context create state?

No. Context distributes a value. State can be created with Hooks such as `useState` or `useReducer` and then provided through Context.

### Is Context global state?

Not inherently. Its value is scoped to the provider's descendant subtree.

### Can Context cause re-renders?

Yes. Consumers can re-render when the provided Context value changes.

### When should Context be avoided?

When ordinary props or composition are simpler, or when the state can be colocated close to its consumers.

---

## 27. Complete Mental Model

```text
          STATE OWNER
              │
              ↓
         Context value
              │
       ┌──────┴──────┐
       ↓             ↓
   consumer       intermediate
                      │
                      ↓
                   consumer

read:
useContext(Context)

update:
callback / setter / dispatch
provided by owner
```

---

## 28. Quick Revision

```jsx
const ThemeContext =
  createContext("light");
```

React 19 provider:

```jsx
<ThemeContext value={theme}>
  <App />
</ThemeContext>
```

Consumer:

```jsx
const theme =
  useContext(ThemeContext);
```

Remember:

```text
Context
→ distributes data

useState/useReducer
→ owns/manages state

props
→ still the default

nearest provider
→ wins
```

---

## 29. Key Takeaways

- Context solves deep/repetitive data passing through a subtree.
- Props remain the default for ordinary component communication.
- Context follows create → provide → consume.
- React 19 supports rendering the Context itself as a provider.
- `useContext` reads the nearest provider's value.
- A Context default is only a fallback when no provider exists.
- Context can carry state, functions, dispatch, or other dependencies.
- Context does not itself create or own state.
- State ownership and state distribution are different concerns.
- Context values are reactive for consumers.
- Scope Context as narrowly as practical.
- Avoid giant unrelated Context values.
- Custom Hooks can provide safer Context APIs.
- Context is not automatically a replacement for external state-management tools.

---

## Next Lesson

➡️ [Lesson 31 — useReducer ⭐⭐⭐⭐⭐](./31-usereducer.md)
