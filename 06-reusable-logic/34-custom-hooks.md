# Lesson 34 — Custom Hooks ⭐⭐⭐⭐⭐

## 1. What Is a Custom Hook?

A **custom Hook** is a JavaScript function that lets you extract and reuse React stateful logic.

```jsx
function useOnlineStatus() {
  // React Hook logic
}
```

Its name starts with `use`, and it may call other Hooks.

The main goal is:

> Reuse **logic**, not UI markup.

---

## 2. Why Custom Hooks Matter ⭐⭐⭐⭐⭐

Suppose two components need to track whether the browser is online.

Without extraction:

```text
StatusBar
 ├── useState
 ├── useEffect
 └── online/offline listeners

SaveButton
 ├── useState
 ├── useEffect
 └── same listeners again
```

The behavior is duplicated.

Extract it:

```text
        useOnlineStatus()
          /          \
         ↓            ↓
   StatusBar      SaveButton
```

Now both components reuse the same **behavioral logic**.

---

## 3. Basic Custom Hook Example

```jsx
import {
  useEffect,
  useState,
} from "react";

export function useOnlineStatus() {
  const [isOnline, setIsOnline] =
    useState(navigator.onLine);

  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }

    function handleOffline() {
      setIsOnline(false);
    }

    window.addEventListener(
      "online",
      handleOnline
    );

    window.addEventListener(
      "offline",
      handleOffline
    );

    return () => {
      window.removeEventListener(
        "online",
        handleOnline
      );

      window.removeEventListener(
        "offline",
        handleOffline
      );
    };
  }, []);

  return isOnline;
}
```

Consumer:

```jsx
function StatusBar() {
  const isOnline =
    useOnlineStatus();

  return (
    <p>
      {isOnline
        ? "Online"
        : "Offline"}
    </p>
  );
}
```

The component expresses **what it needs**, while the Hook hides **how it works**.

---

## 4. Custom Hooks Share Logic, Not State ⭐⭐⭐⭐⭐

This is one of the most important interview concepts.

```jsx
function A() {
  const count = useCounter();
}

function B() {
  const count = useCounter();
}
```

If `useCounter` contains `useState`, A and B get **independent state**.

```text
A
└── useCounter
    └── its own state

B
└── useCounter
    └── its own state
```

A custom Hook does not automatically create shared/global state.

If components need the **same state**, lift it, use Context, or use another appropriate shared-state mechanism.

---

## 5. Naming Rule ⭐⭐⭐⭐⭐

Custom Hooks should start with:

```text
use + CapitalizedPurpose
```

Examples:

```text
useOnlineStatus
useWindowWidth
useChatRoom
useApplications
useDebouncedValue
```

Why?

The `use` prefix tells developers and React tooling that the function follows Hook rules.

A normal utility that does not call Hooks should not pretend to be one.

```js
function formatDate(date) {
  // normal utility
}
```

---

## 6. Custom Hook vs Normal Function

### Normal utility

```js
function calculateTotal(items) {
  return items.reduce(
    (total, item) =>
      total + item.price,
    0
  );
}
```

No React state/effects/context are needed.

### Custom Hook

```jsx
function useWindowWidth() {
  const [width, setWidth] =
    useState(window.innerWidth);

  // subscription logic...

  return width;
}
```

Use a custom Hook when reusable logic needs React Hooks or represents React-specific reusable behavior.

---

## 7. Extracting a Hook Step by Step ⭐⭐⭐⭐⭐

Suppose:

```jsx
function SearchPage() {
  const [query, setQuery] =
    useState("");

  const debouncedQuery =
    // debounce logic...

  // ...
}
```

If the debounce behavior is reusable, extract:

```jsx
function useDebouncedValue(
  value,
  delay
) {
  const [
    debouncedValue,
    setDebouncedValue,
  ] = useState(value);

  useEffect(() => {
    const id = setTimeout(
      () => {
        setDebouncedValue(value);
      },
      delay
    );

    return () =>
      clearTimeout(id);
  }, [value, delay]);

  return debouncedValue;
}
```

Usage:

```jsx
const debouncedQuery =
  useDebouncedValue(
    query,
    300
  );
```

---

## 8. Custom Hooks Can Accept Inputs

Hooks are functions, so they can receive arguments.

```jsx
function useChatRoom({
  roomId,
  serverUrl,
}) {
  useEffect(() => {
    const connection =
      createConnection({
        roomId,
        serverUrl,
      });

    connection.connect();

    return () =>
      connection.disconnect();
  }, [roomId, serverUrl]);
}
```

Usage:

```jsx
useChatRoom({
  roomId,
  serverUrl,
});
```

Reactive values passed into a Hook still participate in normal React reactivity.

---

## 9. Custom Hooks Can Return Anything Useful

A Hook can return:

### One value

```jsx
return isOnline;
```

### Array

```jsx
return [
  value,
  setValue,
];
```

### Object

```jsx
return {
  data,
  error,
  status,
  retry,
};
```

Choose an API that is clear for consumers.

---

## 10. Array vs Object Return

Tuple-like return:

```jsx
const [
  value,
  setValue,
] = useSomething();
```

Useful when positions have obvious meaning, similar to `useState`.

Object return:

```jsx
const {
  data,
  error,
  retry,
} = useSomething();
```

Useful when:

- many values are returned
- consumers may use only some
- named fields improve readability

There is no universal rule.

---

## 11. A Custom Hook Can Use Other Custom Hooks

```text
Component
   ↓
useApplicationSearch
   ├── useDebouncedValue
   ├── useContext
   └── useMemo
```

Custom Hooks can compose smaller Hooks into higher-level behavior.

This is one of React's strongest composition mechanisms.

---

## 12. Keep Hooks Focused on a Purpose ⭐⭐⭐⭐⭐

Good:

```text
useOnlineStatus
useChatRoom
useMediaQuery
useApplicationFilters
```

Less useful abstraction:

```text
useDoStuff
useEverything
useHelper
```

A good custom Hook should have a clear responsibility.

If naming the Hook is difficult, its responsibility may be unclear.

---

## 13. Avoid Fake Lifecycle Hooks

Be cautious with abstractions such as:

```text
useMount
useEffectOnce
useUpdateEffect
```

They often hide Effect dependency problems rather than modeling a meaningful application behavior.

Prefer purpose-driven APIs:

```text
useChatRoom
useAnalyticsImpression
useSocket
```

The abstraction should describe **what synchronization is happening**, not merely disguise `useEffect`.

---

## 14. Custom Hooks and Effects ⭐⭐⭐⭐⭐

A custom Hook does not make a bad Effect correct.

If this logic is unnecessary:

```jsx
useEffect(() => {
  setFullName(
    firstName + " " + lastName
  );
}, [firstName, lastName]);
```

moving it into:

```jsx
useFullName(...)
```

does not improve the architecture.

Derived value:

```jsx
const fullName =
  firstName + " " + lastName;
```

First decide whether an Effect is needed; then extract reusable synchronization logic when appropriate.

---

## 15. Custom Hooks Must Remain Pure During Render

The Hook function itself runs during rendering.

Therefore:

```jsx
function useSomething() {
  // render-time logic must be pure

  useEffect(() => {
    // external synchronization
  }, []);
}
```

Do not perform side effects directly in the Hook body:

```jsx
function useBadHook() {
  localStorage.setItem(
    "x",
    "1"
  ); // ❌ during render
}
```

The same purity expectations that apply to components apply to custom Hooks.

---

## 16. Rules of Hooks Still Apply ⭐⭐⭐⭐⭐

Inside a custom Hook:

```jsx
function useProfile() {
  const [user, setUser] =
    useState(null);

  useEffect(() => {
    // ...
  }, []);

  return user;
}
```

Hooks must follow React's Hook rules.

Do not conditionally call ordinary Hooks:

```jsx
function useProfile(enabled) {
  if (enabled) {
    useEffect(() => {
      // ❌
    }, []);
  }
}
```

Lesson 35 covers these rules deeply.

---

## 17. Put the Condition Inside the Hook Logic

Instead of:

```jsx
if (enabled) {
  useEffect(...); // ❌
}
```

write:

```jsx
useEffect(() => {
  if (!enabled) {
    return;
  }

  // synchronize...
}, [enabled]);
```

Hook call order remains stable.

---

## 18. Custom Hook with Context

Lesson 30 used:

```jsx
function useAuth() {
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

This is an excellent custom Hook use case.

It:

- hides Context implementation
- validates provider usage
- creates a clean feature API

---

## 19. Custom Hook with Reducer + Context

From Section 5:

```jsx
const applications =
  useApplications();

const dispatch =
  useApplicationsDispatch();
```

Consumers do not need to know:

```text
which Context?
which reducer?
which provider value shape?
```

Custom Hooks create an abstraction boundary around reusable feature logic.

---

## 20. CareerLoop Example — useApplicationFilters

```jsx
function useApplicationFilters(
  applications,
  query,
  status
) {
  return useMemo(() => {
    return applications.filter(
      (application) => {
        const matchesQuery =
          application.company
            .toLowerCase()
            .includes(
              query.toLowerCase()
            );

        const matchesStatus =
          status === "all" ||
          application.status ===
            status;

        return (
          matchesQuery &&
          matchesStatus
        );
      }
    );
  }, [
    applications,
    query,
    status,
  ]);
}
```

Usage:

```jsx
const visibleApplications =
  useApplicationFilters(
    applications,
    query,
    status
  );
```

Important: if filtering is cheap, a custom Hook may still improve domain readability, but `useMemo` itself should not be added automatically. Performance memoization should be justified.

---

## 21. CodeBuddy Example — useConnection

A feature Hook could expose intent:

```jsx
const {
  status,
  sendRequest,
  cancelRequest,
} = useConnection(
  developerId
);
```

The component can focus on UI:

```jsx
<button
  onClick={sendRequest}
>
  Connect
</button>
```

The Hook can encapsulate reusable connection behavior without moving the visual markup out of the component.

---

## 22. Stateful Logic vs Shared State ⭐⭐⭐⭐⭐

Remember:

```text
Custom Hook
→ reuse implementation/behavior

Context/store
→ share the same state value
```

Calling:

```jsx
useCounter()
```

twice usually creates two independent Hook state instances.

This distinction is frequently asked in interviews.

---

## 23. Don't Over-Abstract Too Early

Duplicating two simple lines is not automatically a reason for a custom Hook.

Extract when:

- behavior has a clear name
- logic is reused
- a component has too much behavioral detail
- external synchronization deserves isolation
- a feature API becomes clearer

Avoid creating dozens of tiny Hooks that make execution flow harder to follow.

---

## 24. Hook API Design ⭐⭐⭐⭐⭐

A good custom Hook should make correct usage easy.

Prefer:

```jsx
useChatRoom({
  roomId,
  serverUrl,
});
```

over an overly generic API such as:

```jsx
useLifecycleCallback(
  callback,
  dependencies
);
```

A focused API constrains behavior and communicates intent.

---

## 25. Dependency Management Still Matters

If a custom Hook uses:

```jsx
useEffect(() => {
  // uses roomId
  // uses serverUrl
}, [roomId, serverUrl]);
```

extracting the Effect does not remove its reactive dependencies.

Do not hide missing dependencies behind an abstraction.

Keep the Hooks linter enabled.

---

## 26. Testing Custom Hooks

Prefer testing observable behavior through components/features when possible.

For reusable domain logic, you may also test the Hook through a suitable React testing environment.

Focus on:

- returned values
- state transitions
- subscription cleanup
- reactions to changed inputs

Do not test internal implementation details unnecessarily.

---

## 27. File Organization

Simple feature:

```text
applications/
 ├── ApplicationList.jsx
 └── useApplications.js
```

Larger feature:

```text
applications/
 ├── components/
 ├── hooks/
 │    ├── useApplicationFilters.js
 │    └── useApplications.js
 └── ...
```

Organize around maintainability, not arbitrary folder rules.

---

## 28. Common Mistakes ⭐⭐⭐⭐⭐

1. Thinking custom Hooks share state automatically.
2. Naming Hook functions without the `use` prefix.
3. Prefixing ordinary utilities with `use`.
4. Calling Hooks conditionally inside a custom Hook.
5. Hiding unnecessary Effects inside custom Hooks.
6. Building vague lifecycle wrappers like `useEffectOnce`.
7. Performing side effects directly during render.
8. Creating overly generic Hook APIs.
9. Ignoring Effect dependencies after extraction.
10. Over-abstracting trivial logic.
11. Mixing unrelated responsibilities into one giant Hook.
12. Adding memoization inside every custom Hook without measuring.

---

## 29. Interview Questions ⭐⭐⭐⭐⭐

### What is a custom Hook?

A JavaScript function whose name starts with `use` and that encapsulates reusable React Hook-based logic.

### What do custom Hooks reuse?

Stateful/reusable logic and behavior, not UI markup.

### Do two components calling the same custom Hook share state?

No. Each call has independent state unless the Hook connects to an intentionally shared external source or Context/store.

### Why must a custom Hook start with use?

It communicates that the function follows Hook rules and enables React tooling/linting to recognize it as a Hook.

### Can custom Hooks call other Hooks?

Yes.

### Can a custom Hook return JSX?

Technically a function can return many JavaScript values, but custom Hooks are intended for reusable logic. Reusable UI should normally be expressed as components.

### Custom Hook vs component?

A component primarily describes UI; a custom Hook encapsulates reusable React logic.

### Custom Hook vs utility function?

A utility is ordinary JavaScript logic. A custom Hook participates in React's Hook system and may call Hooks.

### Do Hook rules apply inside custom Hooks?

Yes.

### When should you create a custom Hook?

When reusable React behavior has a clear purpose or extracting it makes component intent and responsibilities clearer.

---

## 30. Complete Mental Model

```text
          reusable behavior
                 │
                 ↓
          custom Hook
       ┌─────────┼─────────┐
       ↓         ↓         ↓
   useState   useEffect  useContext
       │         │         │
       └─────────┼─────────┘
                 ↓
        clear Hook API
                 │
        ┌────────┴────────┐
        ↓                 ↓
   Component A       Component B

Each Hook call has its own
stateful Hook instance unless
connected to shared state.
```

---

## 31. Quick Revision

```jsx
function useSomething(input) {
  const [state, setState] =
    useState(...);

  useEffect(() => {
    // synchronize if needed
  }, [input]);

  return state;
}
```

Remember:

```text
custom Hook
= reusable React logic

component
= reusable UI

utility
= reusable plain JS logic
```

---

## 32. Key Takeaways

- Custom Hooks reuse React logic between components.
- Their names begin with `use`.
- They may call built-in and other custom Hooks.
- Custom Hooks share logic, not state itself.
- Each call normally gets independent state.
- Inputs remain reactive.
- A Hook can return values, arrays, objects, and functions.
- Keep Hook APIs focused on meaningful use cases.
- Custom Hooks must obey Hook rules and render purity.
- Extracting an Effect does not fix an unnecessary or incorrect Effect.
- Context-access Hooks are a useful abstraction pattern.
- Do not create custom Hooks only to look more advanced.
- Prefer clear domain intent over generic lifecycle abstractions.
- Keep dependency linting enabled.
- Good custom Hooks make components describe intent instead of implementation details.

---

## Next Lesson

➡️ [Lesson 35 — Rules of Hooks ⭐⭐⭐⭐⭐](./35-rules-of-hooks.md)
