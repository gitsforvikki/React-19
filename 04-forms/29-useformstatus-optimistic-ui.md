# Lesson 29 — useFormStatus and Optimistic UI

React 19 provides useful APIs for better async form UX.

This lesson focuses on:

- `useFormStatus`
- pending submit UI
- optimistic UI
- `useOptimistic`

---

## 1. What Is useFormStatus?

`useFormStatus` lets a child component read the status of its parent form.

Import:

```jsx
import {
  useFormStatus,
} from "react-dom";
```

Example:

```jsx
function SubmitButton() {
  const { pending } =
    useFormStatus();

  return (
    <button
      type="submit"
      disabled={pending}
    >
      {pending
        ? "Saving..."
        : "Save"}
    </button>
  );
}
```

---

## 2. It Must Be Inside the Form

Correct:

```jsx
function SubmitButton() {
  const { pending } =
    useFormStatus();

  return (
    <button
      disabled={pending}
    >
      Save
    </button>
  );
}

function ProfileForm() {
  return (
    <form action={save}>
      <input name="name" />
      <SubmitButton />
    </form>
  );
}
```

Why?

Because `useFormStatus` reads the nearest parent form's status.

---

## 3. useFormStatus vs useActionState

`useActionState`:

```text
Action result state
+
Action function
+
pending state
```

`useFormStatus`:

```text
reads parent form status
from a child component
```

A common pattern is:

```text
Form component
→ useActionState

SubmitButton child
→ useFormStatus
```

---

## 4. What Is Optimistic UI?

Optimistic UI shows the expected result before the async operation finishes.

Normal:

```text
click
↓
wait
↓
server success
↓
update UI
```

Optimistic:

```text
click
↓
update UI immediately
↓
request runs
↓
confirm or recover
```

Good examples:

- like button
- sending a message
- task toggle
- adding a comment

---

## 5. What Is useOptimistic?

```jsx
const [
  optimisticState,
  addOptimistic,
] = useOptimistic(
  state,
  updateFn
);
```

It lets you temporarily show an expected next state while the async work is still running.

---

## 6. Basic Optimistic Example

```jsx
const [
  optimisticMessages,
  addOptimisticMessage,
] = useOptimistic(
  messages,
  (current, text) => [
    ...current,
    {
      id:
        crypto.randomUUID(),
      text,
      sending: true,
    },
  ]
);
```

When sending:

```jsx
async function send(
  formData
) {
  const text =
    formData.get("message");

  addOptimisticMessage(text);

  await sendMessage(text);
}
```

The user sees the message immediately.

---

## 7. Optimistic State Is Temporary

Think:

```text
real state
+
temporary expected change
=
optimistic UI
```

The real server result is still authoritative.

Optimistic state should not become a second permanent source of truth.

---

## 8. Handle Failure

Optimistic UI must have a failure strategy.

Example:

```text
message appears
↓
request fails
↓
mark failed
or remove/revert
↓
allow retry
```

Do not silently pretend success.

---

## 9. Pending UI vs Optimistic UI

Pending UI:

```text
Saving...
```

Optimistic UI:

```text
show expected result immediately
```

They solve different problems and can be used together.

---

## 10. When Not to Be Optimistic

Be careful with critical operations.

Examples:

- payment success
- account deletion
- permission/security changes

Do not show final success before authoritative confirmation.

---

## Common Mistakes

### Mistake 1 — Importing useFormStatus from react

Use:

```js
react-dom
```

### Mistake 2 — Using useFormStatus outside the form it should observe

It needs a parent form.

### Mistake 3 — Treating optimistic state as permanent truth

It is temporary.

### Mistake 4 — Ignoring failure and rollback

Optimistic UI needs recovery.

### Mistake 5 — Using optimism for critical unconfirmed success

Use pending UI instead.

---

## Interview Questions

### What is useFormStatus?

A Hook from `react-dom` that reads the submission status of the nearest parent form.

### Why is SubmitButton often a separate component?

Because the component using `useFormStatus` must be rendered inside the form.

### What is optimistic UI?

Showing the expected result before the async operation is confirmed.

### What is useOptimistic?

A Hook for creating a temporary optimistic version of state.

### Is optimistic state the source of truth?

No. The authoritative result still wins.

### useActionState vs useFormStatus?

`useActionState` manages Action result state. `useFormStatus` reads form submission status.

---

## Quick Revision

```text
useActionState
→ Action result + pending

useFormStatus
→ parent form status

useOptimistic
→ temporary expected UI
```

Remember:

```text
pending UI
→ waiting feedback

optimistic UI
→ expected result shown early
```

---

## Section 4 — Forms Complete

You now understand:

- React form basics
- validation
- FormData
- React 19 Actions
- `useActionState`
- `useFormStatus`
- optimistic UI
- `useOptimistic`

---

## Next Lesson

➡️ [Lesson 30 — Context API and useContext](../05-state-management/30-context-usecontext.md)
