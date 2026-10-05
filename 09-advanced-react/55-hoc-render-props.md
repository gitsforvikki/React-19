# Lesson 55 — Higher-Order Components and Render Props

## 1. Why Learn These Patterns?

Before Hooks, React applications commonly reused stateful logic with:

- **Higher-Order Components (HOCs)**
- **Render Props**

Modern React usually prefers **Custom Hooks** for reusable logic.

However, HOCs and render props still matter because:

- older production code uses them,
- libraries may expose them,
- interviews still ask about them,
- they teach important composition principles.

This lesson focuses on understanding and maintaining these patterns—not using them automatically in new React 19 code.

---

# Part 1 — Higher-Order Components

## 2. What Is a Higher-Order Component? ⭐⭐⭐⭐⭐

A Higher-Order Component is a function that:

```text
takes a component
      ↓
returns an enhanced component
```

Conceptually:

```js
const EnhancedComponent =
  withSomething(
    OriginalComponent
  );
```

A HOC is a **pattern**, not a special React API.

---

## 3. Basic HOC Example

```jsx
function withLoading(
  WrappedComponent
) {
  return function WithLoading({
    isLoading,
    ...props
  }) {
    if (isLoading) {
      return <p>Loading…</p>;
    }

    return (
      <WrappedComponent
        {...props}
      />
    );
  };
}
```

Usage:

```jsx
const UserListWithLoading =
  withLoading(UserList);
```

Then:

```jsx
<UserListWithLoading
  isLoading={isLoading}
  users={users}
/>
```

---

## 4. HOC Mental Model

```text
UserList
   │
   ↓
withLoading(UserList)
   │
   ↓
Enhanced component
   │
   ├── handles loading behavior
   └── renders UserList
```

The original component can stay focused on displaying users.

---

## 5. HOCs Should Compose, Not Mutate ⭐⭐⭐⭐⭐

Avoid changing the original component:

```js
function badHOC(Component) {
  Component.someProperty =
    "changed";

  return Component;
}
```

Prefer wrapping:

```jsx
function goodHOC(
  Component
) {
  return function Wrapper(
    props
  ) {
    return (
      <Component
        {...props}
      />
    );
  };
}
```

Composition is predictable.

Mutation creates hidden coupling.

---

## 6. Prop Injection

A HOC can provide additional props:

```jsx
function withCurrentUser(
  Component
) {
  return function WithUser(
    props
  ) {
    const user =
      useCurrentUser();

    return (
      <Component
        {...props}
        user={user}
      />
    );
  };
}
```

Usage:

```jsx
const ProfileWithUser =
  withCurrentUser(Profile);
```

This pattern was common before Custom Hooks made direct logic reuse easier.

---

## 7. Prop Collision Problem

Suppose the consumer passes:

```jsx
<EnhancedProfile
  user={someUser}
/>
```

but the HOC also injects `user`.

Which value wins depends on prop spread order.

```jsx
<Component
  {...props}
  user={user}
/>
```

The injected value wins.

HOCs should clearly document injected props and avoid collisions.

---

## 8. Pass Unrelated Props Through ⭐⭐⭐⭐⭐

A good HOC generally passes props it does not own:

```jsx
<Component
  {...props}
/>
```

Otherwise the enhanced component can unexpectedly lose normal component inputs.

---

## 9. Do Not Create HOCs During Render ⭐⭐⭐⭐⭐

Wrong:

```jsx
function App() {
  const Enhanced =
    withLoading(UserList);

  return <Enhanced />;
}
```

This creates a new component type on each render.

That can cause remounting and state loss.

Prefer module scope:

```jsx
const EnhancedUserList =
  withLoading(UserList);

function App() {
  return (
    <EnhancedUserList />
  );
}
```

This connects directly to component identity from Lesson 18.

---

## 10. Multiple HOCs and Wrapper Hell

Legacy code may look like:

```js
export default withAuth(
  withTheme(
    withAnalytics(
      UserProfile
    )
  )
);
```

Conceptually:

```text
withAuth
 └── withTheme
      └── withAnalytics
           └── UserProfile
```

Problems:

- difficult debugging
- deeply nested wrappers
- prop collisions
- unclear data origin
- harder TypeScript types

Custom Hooks often flatten this structure.

---

## 11. Static Properties Caveat

Wrapping a component does not automatically copy every custom static property from the wrapped component.

Legacy library code sometimes uses utilities to hoist non-React statics.

For modern application code, prefer simpler composition and avoid relying heavily on component statics.

Know this issue when maintaining HOC-heavy libraries.

---

## 12. Refs and HOCs

Historically, `ref` was not treated like an ordinary prop, so HOCs required special forwarding patterns.

React 19 makes `ref` available as a prop to function components, simplifying new designs.

Ref handling is covered fully in Lesson 56.

Do not blindly copy legacy `forwardRef` HOC patterns into new React 19 code.

---

# Part 2 — Render Props

## 13. What Is a Render Prop? ⭐⭐⭐⭐⭐

A render prop is a prop whose value is a function that a component calls to decide what UI to render.

Example:

```jsx
<DataProvider
  render={(data) => (
    <UserList
      users={data}
    />
  )}
/>
```

The component owns reusable behavior.

The consumer controls presentation.

---

## 14. children as a Function

A common variation:

```jsx
<MousePosition>
  {({ x, y }) => (
    <p>
      {x}, {y}
    </p>
  )}
</MousePosition>
```

Here `children` is a function.

This is still the render-prop pattern.

The prop does not have to literally be named `render`.

---

## 15. Render Prop Example

```jsx
function Toggle({
  children,
}) {
  const [on, setOn] =
    useState(false);

  function toggle() {
    setOn(value => !value);
  }

  return children({
    on,
    toggle,
  });
}
```

Usage:

```jsx
<Toggle>
  {({ on, toggle }) => (
    <>
      <p>
        {on ? "ON" : "OFF"}
      </p>

      <button
        onClick={toggle}
      >
        Toggle
      </button>
    </>
  )}
</Toggle>
```

---

## 16. Render Prop Mental Model

```text
Toggle
  │
  ├── owns state
  ├── owns behavior
  │
  └── calls consumer function
             │
             ↓
       consumer decides UI
```

This cleanly separates reusable behavior from presentation.

---

## 17. Why Render Props Were Useful

Before Hooks, a function component could not simply call a reusable stateful Hook.

Render props allowed one stateful component to expose behavior to arbitrary UI.

Common historical uses included:

- mouse position
- data loading
- form state
- subscriptions
- feature flags
- authentication state

---

## 18. Render Props and Flexibility

One behavior can produce different UIs:

```jsx
<Toggle>
  {({ on, toggle }) => (
    <Switch
      checked={on}
      onClick={toggle}
    />
  )}
</Toggle>
```

or:

```jsx
<Toggle>
  {({ on, toggle }) => (
    <Menu
      open={on}
      onToggle={toggle}
    />
  )}
</Toggle>
```

The behavior component does not need to know the presentation.

---

## 19. Render Prop Drawback: Nesting

Multiple render props can become:

```jsx
<Auth>
  {user => (
    <Theme>
      {theme => (
        <Permissions>
          {permissions => (
            <Page
              user={user}
              theme={theme}
              permissions={
                permissions
              }
            />
          )}
        </Permissions>
      )}
    </Theme>
  )}
</Auth>
```

This is sometimes called callback/nesting hell.

Custom Hooks can often flatten it:

```jsx
const user = useAuth();
const theme = useTheme();
const permissions =
  usePermissions();
```

---

## 20. Function Identity and Render Props

This:

```jsx
<DataProvider>
  {data => (
    <Result data={data} />
  )}
</DataProvider>
```

creates a new function each parent render.

Usually this is fine.

But if the provider is memoized and function identity becomes important, inline render functions can defeat certain memoization strategies.

As always: profile before optimizing.

---

# Part 3 — HOC vs Render Props vs Custom Hooks

## 21. Comparison ⭐⭐⭐⭐⭐

| Pattern | Reuses | How consumer receives logic |
|---|---|---|
| HOC | component behavior | enhanced component / injected props |
| Render Prop | component behavior | callback arguments |
| Custom Hook | stateful logic | direct function return values |

Modern default for reusable stateful logic:

```text
Custom Hook
```

when a Hook naturally fits.

---

## 22. Same Idea as a Custom Hook

Render prop:

```jsx
<Toggle>
  {({ on, toggle }) => (
    <Button
      on={on}
      onClick={toggle}
    />
  )}
</Toggle>
```

Custom Hook:

```jsx
function Button() {
  const {
    on,
    toggle,
  } = useToggle();

  return (
    <button
      onClick={toggle}
    >
      {on ? "ON" : "OFF"}
    </button>
  );
}
```

Hooks often remove wrapper components and make data origin clearer.

---

## 23. When HOCs Still Make Sense

HOCs can still be useful when the abstraction is specifically about **wrapping a component**.

Examples may include:

- legacy libraries
- compatibility adapters
- cross-cutting wrappers
- APIs intentionally designed around component enhancement

Do not rewrite stable HOC code merely because Hooks exist.

Evaluate maintainability and value.

---

## 24. When Render Props Still Make Sense

Render props remain useful when a component intentionally owns a behavior/lifecycle and consumers need complete control over rendering.

Some headless component APIs use function children because the render callback is itself a clean public API.

Modern does not mean "render props are forbidden."

---

## 25. Custom Hooks Are Usually the Modern Logic-Sharing Default ⭐⭐⭐⭐⭐

If you need:

```text
reuse state + effects + callbacks
across components
```

start by considering:

```js
function useSomething() {
  // reusable React logic
}
```

Benefits:

- less wrapper nesting
- explicit dependencies
- easier composition
- direct access to returned values
- natural function-component usage

Rules of Hooks still apply.

---

## 26. HOC vs Compound Components

Do not confuse the patterns.

```text
HOC
→ Component → EnhancedComponent

Compound Components
→ related components cooperate
  through shared contract
```

Example:

```jsx
withAuth(Profile)
```

vs:

```jsx
<Tabs>
  <Tabs.Trigger />
  <Tabs.Panel />
</Tabs>
```

---

## 27. Render Props vs children Content

Normal children:

```jsx
<Card>
  <Profile />
</Card>
```

Function children:

```jsx
<DataProvider>
  {data => (
    <Profile data={data} />
  )}
</DataProvider>
```

The second is a render-prop-style API because the child is invoked as a function.

---

## 28. CareerLoop Example

Legacy HOC:

```jsx
const ProtectedApplications =
  withAuth(
    ApplicationsPage
  );
```

A modern app may instead read auth through a Hook inside the appropriate component or use a routing/framework authorization architecture.

The HOC is still important to recognize when reading older code.

---

## 29. CodeBuddy Example

Render-prop-style online-status provider:

```jsx
<OnlineStatus>
  {({ online }) => (
    <DeveloperCard
      online={online}
    />
  )}
</OnlineStatus>
```

A modern implementation may expose:

```jsx
const online =
  useOnlineStatus();
```

if a Custom Hook provides a cleaner API.

---

## 30. Common HOC Mistakes ⭐⭐⭐⭐⭐

1. Mutating the wrapped component.
2. Creating the HOC inside render.
3. Losing unrelated props.
4. Injecting props with unclear names.
5. Creating prop collisions.
6. Building many nested wrappers.
7. Assuming statics automatically carry over.
8. Copying legacy ref-forwarding patterns without considering React 19.
9. Using a HOC when a Custom Hook is substantially clearer.

---

## 31. Common Render Prop Mistakes

1. Excessive nested callbacks.
2. Making the render callback contract unclear.
3. Confusing normal children with function children.
4. Over-optimizing inline callback identity.
5. Using render props when a simple Hook would make logic clearer.

---

## 32. Interview Questions ⭐⭐⭐⭐⭐

### What is a Higher-Order Component?

A function that takes a component and returns an enhanced component.

### Is HOC a React API?

No. It is a composition pattern.

### Should a HOC mutate the wrapped component?

No. Prefer composition.

### Why should you not create a HOC inside render?

It creates a new component type and can cause remounting/state loss.

### What is a render prop?

A function prop that a component calls to determine what to render.

### Does the prop have to be called render?

No. Function children are a common render-prop form.

### HOC vs render prop?

A HOC wraps/enhances a component; a render prop exposes behavior through a callback.

### What is the modern default for reusable stateful logic?

Usually a Custom Hook.

### Are HOCs and render props obsolete?

No. They remain valid patterns and are common in legacy/library code, but Hooks often provide a simpler modern solution.

---

## 33. Interview-Ready Summary ⭐⭐⭐⭐⭐

```text
HOCs and render props are React composition patterns
used heavily before Hooks for sharing stateful behavior.

A HOC takes a component and returns an enhanced
component, usually injecting behavior or props.

A render-prop component owns behavior and calls a
consumer-provided function to decide what UI to render.

Both remain useful to understand for existing code and
some library APIs, but for new React code I usually
consider a Custom Hook first when the goal is simply
reusing stateful logic.

I also avoid creating HOCs during render because that
changes component identity and can reset state.
```

---

## 34. Complete Mental Model

```text
HOC
────
Component
   │
   ↓
withFeature(Component)
   │
   ↓
EnhancedComponent


Render Prop
───────────
BehaviorComponent
   │
   ├── owns logic
   │
   └── calls function
           │
           ↓
       consumer UI


Custom Hook
───────────
Component
   │
   ↓
useFeature()
   │
   ↓
values + behavior
```

---

## 35. Key Takeaways

- HOCs and render props are patterns, not special React syntax.
- HOCs transform/wrap components.
- HOCs should compose rather than mutate.
- Never create enhanced component types repeatedly during render.
- Render props expose behavior through a function.
- Function children are a common render-prop form.
- Both patterns can create nesting/indirection.
- Custom Hooks are usually the modern default for reusable stateful logic.
- HOCs/render props are still important for legacy code, libraries, and interviews.
- Choose the pattern that creates the clearest public API rather than following fashion.

---

## Next Lesson

➡️ [Lesson 56 — forwardRef and Modern Ref Handling](./56-forwardref-modern-refs.md)
