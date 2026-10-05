# Lesson 60 — useOptimistic ⭐⭐⭐⭐⭐

## 1. What Is Optimistic UI?

Normally a mutation looks like:

```text
User action
   ↓
send request
   ↓
wait...
   ↓
server succeeds
   ↓
update UI
```

Optimistic UI changes the experience:

```text
User action
   ↓
update UI immediately
   ↓
run real mutation
   ↓
server result
 ┌────┴─────┐
success   failure
  ↓          ↓
confirm    reconcile /
state      roll back
```

The UI behaves as though the likely successful result has already happened.

---

## 2. What Is useOptimistic? ⭐⭐⭐⭐⭐

`useOptimistic` is a React Hook for temporarily showing an optimistic version of canonical state while an Action is in progress.

```jsx
import {
  useOptimistic,
} from "react";

const [
  optimisticState,
  setOptimistic,
] = useOptimistic(state);
```

It returns exactly two values:

```text
optimisticState
→ value React should display now

setOptimistic
→ request a temporary optimistic update
  during an Action/Transition
```

---

## 3. Canonical State vs Optimistic State ⭐⭐⭐⭐⭐

This distinction is essential.

Suppose:

```js
isLiked = false
```

The user clicks Like.

Before the server responds:

```text
canonical state
false

optimistic state
true
```

After the mutation completes, the canonical state should reflect the authoritative result.

```text
optimistic UI
is temporary

canonical state
is the lasting source of truth
```

---

## 4. Basic Example

```jsx
function LikeButton({
  isLiked,
  toggleLike,
}) {
  const [
    optimisticIsLiked,
    setOptimisticIsLiked,
  ] = useOptimistic(
    isLiked
  );

  function handleClick() {
    const nextValue =
      !optimisticIsLiked;

    startTransition(
      async () => {
        setOptimisticIsLiked(
          nextValue
        );

        await toggleLike(
          nextValue
        );
      }
    );
  }

  return (
    <button
      onClick={handleClick}
    >
      {optimisticIsLiked
        ? "❤️ Liked"
        : "🤍 Like"}
    </button>
  );
}
```

The visible UI changes immediately instead of waiting for the request.

---

## 5. Optimistic Updates Belong Inside an Action ⭐⭐⭐⭐⭐

The optimistic setter should be called inside an Action/Transition.

Correct:

```jsx
startTransition(
  async () => {
    setOptimistic(
      nextValue
    );

    await save(nextValue);
  }
);
```

When React calls an Action prop such as a form `action`, that Action context is already provided:

```jsx
async function submitAction(
  formData
) {
  setOptimistic(
    formData.get("name")
  );

  await updateName();
}

<form action={submitAction}>
```

You do not need to wrap that optimistic setter in another `startTransition` merely to establish the Action context.

---

## 6. What Happens When the Action Ends? ⭐⭐⭐⭐⭐

`useOptimistic` temporarily derives UI from the canonical value.

Conceptually:

```text
canonical value = A
       │
Action begins
       ↓
optimistic update = B
       │
UI shows B
       │
Action ends
       ↓
React renders from
latest canonical value
```

If the real source becomes B, the optimistic and confirmed UI agree.

If the mutation fails and canonical state remains A, React returns to A.

This is why optimistic state should not become a second permanent source of truth.

---

## 7. Automatic Reconciliation, Not Magic Server Rollback

It is useful to say:

> the optimistic UI rolls back on failure.

But understand what that means.

React is not reversing your database mutation.

React stops showing the temporary optimistic layer and renders from the canonical value again.

You are still responsible for:

- real server consistency
- error reporting
- retries where appropriate
- updating canonical data after success

---

## 8. updateFn / Reducer Form ⭐⭐⭐⭐⭐

For structured optimistic updates:

```jsx
const [
  optimisticMessages,
  addOptimisticMessage,
] = useOptimistic(
  messages,
  (
    currentMessages,
    newMessage
  ) => [
    ...currentMessages,
    {
      ...newMessage,
      pending: true,
    },
  ]
);
```

The second argument is a pure update/reducer function.

Conceptually:

```text
current optimistic state
+
optimistic action/value
        ↓
pure update function
        ↓
next optimistic state
```

---

## 9. Purity Matters ⭐⭐⭐⭐⭐

Do not perform side effects inside the optimistic reducer:

```jsx
useOptimistic(
  items,
  (state, item) => {
    fetch("/api/items"); // ❌

    return [
      ...state,
      item,
    ];
  }
);
```

The reducer should calculate state.

Perform the actual mutation in the Action.

---

## 10. Optimistically Adding to a List ⭐⭐⭐⭐⭐

```jsx
function TodoList({
  todos,
  addTodo,
}) {
  const [
    optimisticTodos,
    addOptimisticTodo,
  ] = useOptimistic(
    todos,
    (current, todo) => [
      ...current,
      {
        ...todo,
        pending: true,
      },
    ]
  );

  function handleAdd(text) {
    const todo = {
      id:
        crypto.randomUUID(),
      text,
    };

    startTransition(
      async () => {
        addOptimisticTodo(
          todo
        );

        await addTodo(todo);
      }
    );
  }

  return (
    <ul>
      {optimisticTodos.map(
        todo => (
          <li key={todo.id}>
            {todo.text}
            {todo.pending &&
              " (Adding...)"}
          </li>
        )
      )}
    </ul>
  );
}
```

A temporary client ID gives the optimistic item stable identity.

---

## 11. Why Reducers Are Valuable When Base State Changes ⭐⭐⭐⭐⭐

Suppose another update changes the canonical list while your mutation is pending.

A reducer can be re-applied against the latest base state.

```text
base list A
   │
optimistic add X
   │
base changes to B
   │
React recalculates
B + optimistic X
```

This is safer than assuming the original base list will remain unchanged.

---

## 12. Multiple Related Values

Suppose Follow changes:

- `isFollowing`
- follower count

Keep them consistent in one optimistic state:

```jsx
const [
  optimisticUser,
  setFollowing,
] = useOptimistic(
  {
    isFollowing:
      user.isFollowing,
    followerCount:
      user.followerCount,
  },
  (
    current,
    isFollowing
  ) => ({
    isFollowing,
    followerCount:
      current.followerCount +
      (isFollowing
        ? 1
        : -1),
  })
);
```

One reducer preserves the relationship between the values.

---

## 13. Optimistic Delete

Instead of instantly removing an item, you may first mark it:

```jsx
const [
  optimisticItems,
  markDeleting,
] = useOptimistic(
  items,
  (current, id) =>
    current.map(item =>
      item.id === id
        ? {
            ...item,
            deleting: true,
          }
        : item
    )
);
```

UI:

```text
Application X
Deleting...
```

If deletion fails, canonical state still contains the item, so the temporary optimistic representation disappears when the Action settles.

---

## 14. Error Feedback Still Matters ⭐⭐⭐⭐⭐

Rollback alone may confuse the user.

```text
user deletes item
→ item appears deleting
→ request fails
→ item returns
```

Also communicate the failure:

```jsx
try {
  await deleteItem(id);
} catch (error) {
  setError(
    "Could not delete item."
  );
}
```

Optimistic UI improves responsiveness; it does not replace error UX.

---

## 15. useOptimistic Does Not Return isPending

The Hook returns:

```text
[
  optimisticState,
  optimisticSetter
]
```

not:

```text
[
  optimisticState,
  optimisticSetter,
  isPending
]
```

If you need transition-level pending state, use `useTransition`.

For item-level status, include a pending flag in your optimistic representation.

---

## 16. useOptimistic vs useState ⭐⭐⭐⭐⭐

### useState

```text
persistent component state
until explicitly changed
```

### useOptimistic

```text
temporary optimistic layer
during an Action
based on canonical state
```

Do not replace all local state with `useOptimistic`.

---

## 17. useOptimistic vs useTransition

`useTransition` answers:

```text
Is this Transition pending?
```

`useOptimistic` answers:

```text
What should the UI temporarily
look like while the Action runs?
```

They can work together:

```jsx
const [
  isPending,
  startTransition,
] = useTransition();

const [
  optimisticValue,
  setOptimisticValue,
] = useOptimistic(value);
```

---

## 18. useOptimistic vs useActionState

`useActionState`:

```text
manages result/state returned
by an Action
+
pending status
```

`useOptimistic`:

```text
temporarily derives expected UI
before Action completion
```

Common combination:

```text
useActionState
→ mutation result/error/pending

useOptimistic
→ immediate visual response
```

---

## 19. When Optimistic UI Is a Good Fit ⭐⭐⭐⭐⭐

Good candidates:

- likes
- follows
- connection requests
- adding messages
- adding comments
- simple status changes
- cart quantity changes
- deleting/restoring list items

These actions usually have a high chance of success and an obvious expected result.

---

## 20. When to Be More Careful

Avoid casually pretending success when:

- payment is unconfirmed
- inventory may be unavailable
- permission checks may fail
- destructive action has serious consequences
- result is determined by complex server logic

Example:

```text
"Payment successful"
```

should not be shown merely because the user clicked Pay.

You may optimistically show:

```text
"Processing payment..."
```

but confirmation must come from authoritative payment state.

---

## 21. CodeBuddy Example

Connection request:

```text
Connect
   ↓
optimistic "Requested"
   ↓
POST request
 ┌─┴────┐
ok     fail
 ↓       ↓
server  canonical
state   state remains
matches "Connect"
```

This is an excellent optimistic UI case.

---

## 22. CareerLoop Example

Changing application status:

```text
Applied
   ↓ user chooses
Interview
   ↓
UI immediately shows Interview
   ↓
database update
```

If the update fails, return to the canonical status and show an error.

---

## 23. Common Mistakes ⭐⭐⭐⭐⭐

1. Calling the optimistic setter outside an Action/Transition.
2. Treating optimistic state as the permanent source of truth.
3. Performing side effects inside the reducer.
4. Forgetting real error feedback.
5. Assuming React rolls back the server/database.
6. Showing optimistic success for high-risk operations like confirmed payments.
7. Mutating arrays/objects in the optimistic reducer.
8. Forgetting stable IDs for optimistic list items.
9. Ignoring canonical state changes while an Action is pending.
10. Expecting `useOptimistic` to return `isPending`.
11. Using optimistic UI when the expected result cannot be predicted safely.

---

## 24. Interview Questions ⭐⭐⭐⭐⭐

### What is useOptimistic?

A Hook that lets React temporarily display an optimistic version of canonical state while an Action is pending.

### What does it return?

The optimistic state and a function for applying optimistic updates.

### Where should the optimistic setter be called?

Inside an Action/Transition. Action props already provide that context.

### What happens when the Action completes?

React reconciles the optimistic state with the latest canonical value.

### What happens when the Action fails?

If canonical state was not changed to the optimistic result, the temporary optimistic UI disappears and the canonical value is shown again.

### Does React undo a database mutation?

No.

### Why use the reducer form?

It can derive optimistic state from the latest base state and keep related values consistent.

### Must the reducer be pure?

Yes.

### useOptimistic vs useState?

`useState` stores persistent component state; `useOptimistic` represents temporary expected state during an Action.

### useOptimistic vs useTransition?

`useTransition` exposes pending priority/transition state; `useOptimistic` controls what optimistic UI is displayed.

---

## 25. Interview-Ready Answer ⭐⭐⭐⭐⭐

```text
useOptimistic lets me show the expected result of a
mutation immediately while the real Action is still
running.

It takes a canonical value and optionally a pure
reducer. During an Action I apply an optimistic update,
so React temporarily renders that expected state.

When the Action settles, React reconciles back to the
latest canonical value. If the real mutation succeeds,
the canonical value should confirm the result. If it
fails and the canonical value remains unchanged, the
optimistic UI naturally disappears.

I use optimistic UI for high-confidence interactions
such as likes, follows, messages, and simple status
updates, but I do not optimistically claim authoritative
success for things like payments.
```

---

## 26. Complete Mental Model

```text
              CANONICAL STATE
                    A
                    │
                    ↓
             useOptimistic(A)
                    │
          user starts Action
                    ↓
          optimistic update B
                    │
                    ↓
               UI shows B
                    │
             real mutation
              ┌─────┴─────┐
              ↓           ↓
           success      failure
              │           │
     canonical → B    canonical = A
              │           │
              └─────┬─────┘
                    ↓
       React renders latest
          canonical state
```

---

## 27. Key Takeaways

- Optimistic UI makes mutation feedback immediate.
- `useOptimistic` creates a temporary optimistic layer over canonical state.
- It returns optimistic state and an optimistic setter/dispatcher.
- Optimistic setters belong inside Actions/Transitions.
- Action props already provide an Action context.
- The optional reducer must be pure.
- Reducers are useful when canonical state can change while work is pending.
- Optimistic state is not a second permanent source of truth.
- React reconciles to canonical state after the Action.
- React does not undo your server/database operations.
- Error messages and recovery UX still matter.
- `useOptimistic` does not return `isPending`.
- Use optimistic UI for predictable, high-confidence mutations.
- Do not optimistically claim authoritative success for sensitive operations.

---

## Next Lesson

➡️ [Lesson 61 — React 19 Ref Improvements](./61-ref-improvements.md)
