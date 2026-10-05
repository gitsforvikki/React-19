# Lesson 39 — useCallback ⭐⭐⭐⭐⭐

## 1. What Is useCallback?

`useCallback` is a React Hook that caches a **function definition/reference** between renders.

```jsx
const cachedFunction =
  useCallback(
    functionDefinition,
    dependencies
  );
```

Mental model:

```text
render
  ↓
dependencies changed?
 ┌──────┴──────┐
no            yes
↓              ↓
return        return
previous      new function
function      reference
reference
```

---

## 2. Why Function Identity Matters ⭐⭐⭐⭐⭐

Every render normally creates new function objects:

```jsx
function Parent() {
  function handleSave() {
    // ...
  }

  return (
    <Child
      onSave={handleSave}
    />
  );
}
```

Across renders:

```text
render 1
handleSave → function A

render 2
handleSave → function B
```

Even if source code looks identical:

```js
Object.is(
  functionA,
  functionB
) === false
```

Usually this is completely fine.

It matters when stable identity is part of an optimization/dependency relationship.

---

## 3. Basic Syntax

```jsx
import {
  useCallback,
} from "react";

const handleSave =
  useCallback(() => {
    saveApplication(id);
  }, [id]);
```

If `id` is unchanged, React can return the same cached function reference.

If `id` changes, React returns the new function from the current render.

---

## 4. useCallback Caches the Function, Not Its Result ⭐⭐⭐⭐⭐

```jsx
const handleSubmit =
  useCallback(
    (data) => {
      return submit(data);
    },
    []
  );
```

React does not execute `handleSubmit` during `useCallback`.

It caches the function reference.

Compare:

```text
useMemo
→ calls calculation when needed
→ caches returned VALUE

useCallback
→ does not call callback
→ caches FUNCTION reference
```

---

## 5. Most Important Use Case — memoized Child ⭐⭐⭐⭐⭐

Child:

```jsx
const SaveButton =
  memo(function SaveButton({
    onSave,
  }) {
    console.log(
      "SaveButton render"
    );

    return (
      <button
        onClick={onSave}
      >
        Save
      </button>
    );
  });
```

Parent without `useCallback`:

```jsx
function Page() {
  const handleSave = () => {
    save();
  };

  return (
    <SaveButton
      onSave={handleSave}
    />
  );
}
```

Each parent render creates a new function.

So:

```text
memo child
+
new function prop
=
props changed
→ child renders
```

---

## 6. Stabilizing the Function Prop

```jsx
const handleSave =
  useCallback(() => {
    save();
  }, []);
```

Now:

```text
parent render
     ↓
same dependencies
     ↓
same handleSave reference
     ↓
memo child can potentially
skip rendering
```

This is one of the clearest reasons to use `useCallback`.

---

## 7. useCallback Alone Does Not Stop Child Rendering ⭐⭐⭐⭐⭐

Important interview trap.

```jsx
const handleSave =
  useCallback(
    () => save(),
    []
  );

return (
  <Child
    onSave={handleSave}
  />
);
```

If `Child` is an ordinary component, parent rendering normally renders Child too.

`useCallback` only stabilizes the function reference.

It does **not** independently memoize the child component.

Common combination:

```text
memo
+
useCallback
```

when function prop identity is preventing a useful memo skip.

---

## 8. Dependencies ⭐⭐⭐⭐⭐

```jsx
const handleSubmit =
  useCallback(
    (details) => {
      postOrder({
        productId,
        referrer,
        details,
      });
    },
    [
      productId,
      referrer,
    ]
  );
```

Every reactive value used by the function belongs in the dependency list.

Missing dependencies can create stale closures.

---

## 9. Stale Closure Problem

Wrong:

```jsx
const handleSave =
  useCallback(() => {
    console.log(count);
  }, []); // ❌ count omitted
```

The cached function may keep seeing the `count` from the render where it was created.

Correct:

```jsx
const handleSave =
  useCallback(() => {
    console.log(count);
  }, [count]);
```

Do not remove dependencies merely to keep a function "stable".

Correctness comes first.

---

## 10. Functional State Updates Can Reduce Dependencies ⭐⭐⭐⭐⭐

Suppose:

```jsx
const addApplication =
  useCallback(
    (application) => {
      setApplications([
        ...applications,
        application,
      ]);
    },
    [applications]
  );
```

Because the callback only needs previous state to calculate next state, use an updater:

```jsx
const addApplication =
  useCallback(
    (application) => {
      setApplications(
        (applications) => [
          ...applications,
          application,
        ]
      );
    },
    []
  );
```

Now the callback does not need to read `applications` from its closure.

This is a meaningful way to remove a dependency—not by lying to the linter.

---

## 11. useCallback as an Effect Dependency

Suppose:

```jsx
function createOptions() {
  return {
    roomId,
  };
}

useEffect(() => {
  const options =
    createOptions();

  // connect...
}, [createOptions]);
```

`createOptions` changes every render, so the Effect repeatedly runs.

One possible solution:

```jsx
const createOptions =
  useCallback(() => {
    return {
      roomId,
    };
  }, [roomId]);
```

Then the Effect runs when `roomId` meaningfully changes.

But there may be an even simpler solution.

---

## 12. Often Better: Move the Function Inside the Effect ⭐⭐⭐⭐⭐

Instead of memoizing:

```jsx
const createOptions =
  useCallback(
    () => ({
      roomId,
    }),
    [roomId]
  );

useEffect(() => {
  const options =
    createOptions();

  connect(options);
}, [createOptions]);
```

write:

```jsx
useEffect(() => {
  function createOptions() {
    return {
      roomId,
    };
  }

  const options =
    createOptions();

  connect(options);
}, [roomId]);
```

No `useCallback` needed.

This is an important optimization principle:

> Remove unnecessary dependencies before memoizing them.

---

## 13. useCallback in Custom Hooks ⭐⭐⭐⭐⭐

A custom Hook that returns functions may benefit from stable callback references.

```jsx
function useApplications() {
  const dispatch =
    useApplicationsDispatch();

  const deleteApplication =
    useCallback(
      (id) => {
        dispatch({
          type: "deleted",
          id,
        });
      },
      [dispatch]
    );

  return {
    deleteApplication,
  };
}
```

This can let consumers optimize based on stable returned functions.

Do it when the custom Hook API benefits from stable identities, not automatically.

---

## 14. dispatch and State Setters Are Already Stable

React gives stable identities for functions such as state setters and reducer dispatch.

```jsx
const [
  count,
  setCount,
] = useState(0);

const [
  state,
  dispatch,
] = useReducer(
  reducer,
  initialState
);
```

You do not need:

```jsx
const stableSetCount =
  useCallback(
    setCount,
    []
  );
```

That adds no useful value.

---

## 15. useCallback Does Not Prevent Function Creation ⭐⭐⭐⭐⭐

A common misconception:

```text
useCallback
→ JavaScript does not create function
```

Not the right mental model.

During rendering, your function expression exists in the current render.

React can return the previously cached function when dependencies are unchanged.

Think:

```text
useCallback
→ stable returned function identity
```

not:

```text
→ eliminates all function allocation/work
```

---

## 16. useCallback Is Equivalent in Spirit to useMemo Returning a Function

Conceptually:

```jsx
useCallback(
  fn,
  dependencies
);
```

is similar to:

```jsx
useMemo(
  () => fn,
  dependencies
);
```

`useCallback` simply provides a clearer API for function memoization.

---

## 17. Don't useCallback Every Event Handler ⭐⭐⭐⭐⭐

This:

```jsx
const handleClick =
  useCallback(() => {
    setOpen(true);
  }, []);
```

is not automatically better than:

```jsx
function handleClick() {
  setOpen(true);
}
```

If:

- the function is not passed to a memoized child,
- it is not a Hook dependency,
- stable identity is not otherwise needed,

`useCallback` often provides no useful optimization.

---

## 18. Memoization Has Costs

`useCallback` adds:

- dependency comparison
- cached value bookkeeping
- more code
- more mental overhead
- stale closure risk if dependencies are wrong

The question is not:

> Can I use `useCallback` here?

The question is:

> Does stable function identity provide a measurable or architectural benefit here?

---

## 19. One Unstable Prop Can Still Break memo

Suppose:

```jsx
const handleSave =
  useCallback(
    () => save(),
    []
  );

return (
  <MemoChild
    onSave={handleSave}
    options={{
      compact: true,
    }}
  />
);
```

`onSave` is stable.

But:

```jsx
options={{
  compact: true,
}}
```

is new every render.

So the child may still render.

Memoization must be considered across all relevant props.

---

## 20. useCallback with Multiple Dependencies

```jsx
const handleConnect =
  useCallback(
    () => {
      sendConnectionRequest(
        currentUserId,
        developerId
      );
    },
    [
      currentUserId,
      developerId,
    ]
  );
```

If either dependency changes:

```text
new function reference
```

That is correct because the behavior now depends on new values.

Do not force stability across semantic changes.

---

## 21. Function Defined Outside Component

If a function does not depend on component props/state, it may not need to be recreated inside the component at all.

```jsx
function formatApplication(
  application
) {
  return application.company;
}

function List({
  applications,
}) {
  // use formatApplication
}
```

No `useCallback` is required.

This can be simpler than memoizing a function with no reactive dependencies.

---

## 22. Event Handler vs Memoized Callback

Normal:

```jsx
<button
  onClick={() =>
    setOpen(true)
  }
>
  Open
</button>
```

This is usually fine.

Do not treat inline event handlers as inherently bad.

Optimize identity only when it affects a measured memoized/dependency boundary.

---

## 23. React Compiler Perspective ⭐⭐⭐⭐⭐

React Compiler can automatically memoize functions in compiled code.

Therefore modern guidance is not:

```text
wrap every handler in useCallback
```

Instead:

```text
write pure clear components
      ↓
compiler can optimize
      ↓
manual useCallback when:
- precise identity is needed
- existing/non-compiled code benefits
- profiling shows value
```

Manual `useCallback` remains important to understand and remains available as an escape hatch/control mechanism.

---

## 24. CodeBuddy Example

Memoized card:

```jsx
const DeveloperCard =
  memo(function DeveloperCard({
    developer,
    onConnect,
  }) {
    // ...
  });
```

Parent:

```jsx
const handleConnect =
  useCallback(
    (developerId) => {
      sendConnectionRequest(
        developerId
      );
    },
    []
  );
```

Then:

```jsx
<DeveloperCard
  developer={developer}
  onConnect={
    handleConnect
  }
/>
```

If card rendering is expensive and other props are stable, preserving callback identity may help cards skip unrelated parent renders.

---

## 25. CareerLoop Example

```jsx
const ApplicationCard =
  memo(
    function ApplicationCard({
      application,
      onDelete,
    }) {
      // ...
    }
  );
```

Parent:

```jsx
const handleDelete =
  useCallback(
    (id) => {
      setApplications(
        (applications) =>
          applications.filter(
            (application) =>
              application.id !==
              id
          )
      );
    },
    []
  );
```

Functional update avoids depending on the current applications array.

---

## 26. Common Mistakes ⭐⭐⭐⭐⭐

1. Wrapping every function in `useCallback`.
2. Believing `useCallback` prevents child renders by itself.
3. Omitting dependencies to force stable identity.
4. Creating stale closures.
5. Memoizing state setters or reducer dispatch unnecessarily.
6. Using `useCallback` when function could live outside the component.
7. Memoizing a function that is only used locally and has no identity-sensitive role.
8. Forgetting another always-new prop can defeat child memoization.
9. Using `useCallback` instead of simplifying Effect dependencies.
10. Assuming it prevents function creation entirely.
11. Ignoring React Compiler's automatic function memoization.

---

## 27. Interview Questions ⭐⭐⭐⭐⭐

### What is useCallback?

A Hook that caches a function definition/reference between renders while its dependencies remain unchanged.

### What does useCallback return?

A cached function reference.

### Does useCallback execute the function?

No.

### useCallback vs useMemo?

`useCallback` caches a function; `useMemo` caches the result of a calculation.

### Does useCallback prevent a child from rendering?

Not by itself. It can help a memoized child skip rendering by keeping a function prop stable.

### When is useCallback useful?

Commonly when passing callbacks to memoized children or when a stable function is a dependency of another Hook.

### Why are dependencies important?

The function closes over reactive values; missing dependencies can create stale behavior.

### How can functional state updates reduce dependencies?

If the callback only needs previous state to compute next state, pass an updater to the setter instead of reading state from the closure.

### Should every event handler use useCallback?

No.

### Does useCallback prevent function creation?

No. Think of it as preserving the returned function identity when dependencies are unchanged.

### Are setState and reducer dispatch stable?

Yes; wrapping them just to stabilize them is unnecessary.

### How does React Compiler affect useCallback?

It can automatically memoize functions, reducing the need for manual `useCallback` in compiled code.

---

## 28. Complete Mental Model

```text
             component render
                    │
                    ↓
                function
                    │
                    ↓
              useCallback
                    │
         compare dependencies
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
      unchanged             changed
          │                   │
          ↓                   ↓
 return previous         return current
function reference      function reference
          │
          ↓
useful for identity-sensitive
memo/dependency boundaries
```

---

## 29. Quick Revision

```jsx
const handleSave =
  useCallback(() => {
    save(id);
  }, [id]);
```

Remember:

```text
useCallback
→ memoizes FUNCTION identity

useMemo
→ memoizes VALUE/result

memo
→ memoizes COMPONENT rendering
```

---

## 30. Key Takeaways

- `useCallback` preserves a function reference between renders while dependencies are unchanged.
- It caches the function, not the result of calling it.
- Functions normally have new identities across renders.
- New function props can defeat useful `memo` optimization.
- `useCallback` alone does not prevent child rendering.
- Include every reactive dependency.
- Never omit dependencies merely to force stability.
- Functional state updates can legitimately remove state dependencies.
- Often moving a function inside an Effect is simpler than memoizing it.
- State setters and reducer dispatch already have stable identities.
- Inline functions are not inherently a performance problem.
- Memoization adds complexity and bookkeeping.
- Use stable callback identity only where it matters.
- React Compiler can automatically memoize functions.
- Manual `useCallback` remains important for existing code, precise control, and interviews.

---

## Next Lesson

➡️ [Lesson 40 — React.memo vs useMemo vs useCallback ⭐⭐⭐⭐⭐](./40-memoization-comparison.md)
