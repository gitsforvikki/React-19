# Lesson 29 — useFormStatus and Optimistic UI

## 1. What This Lesson Solves

A good async form should communicate what is happening.

When a user submits:

```text
click Save
   ↓
request starts
   ↓
show pending UI
   ↓
request completes
   ↓
show result
```

React provides form-aware APIs for this workflow.

This lesson focuses on:

- `useFormStatus`
- pending submit buttons
- form status metadata
- optimistic UI concepts
- `useOptimistic`
- rollback/reconciliation thinking

`useOptimistic` receives a dedicated deep React 19 lesson later, so here we focus on how it fits form UX.

---

# 2. What Is useFormStatus? ⭐⭐⭐⭐⭐

`useFormStatus` lets a component read the status of its parent form submission.

Import:

```jsx
import {
  useFormStatus,
} from "react-dom";
```

Example:

```jsx
function SubmitButton() {
  const {
    pending,
  } = useFormStatus();

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

# 3. Important Import ⭐⭐⭐⭐⭐

A common interview/code mistake:

```jsx
import {
  useFormStatus,
} from "react";
```

For this API, use:

```jsx
import {
  useFormStatus,
} from "react-dom";
```

---

# 4. It Reads the Parent Form's Status ⭐⭐⭐⭐⭐

`useFormStatus` must be used by a component rendered **inside** the form whose status it wants to observe.

Correct:

```jsx
function SubmitButton() {
  const {
    pending,
  } = useFormStatus();

  return (
    <button
      disabled={pending}
    >
      Submit
    </button>
  );
}

function Form() {
  return (
    <form action={save}>
      <input name="name" />

      <SubmitButton />
    </form>
  );
}
```

Structure:

```text
<form>
   │
   ├── input
   │
   └── SubmitButton
          ↓
     useFormStatus()
          ↓
     parent form status
```

---

# 5. Why Calling It in the Same Component Can Be Wrong ⭐⭐⭐⭐⭐

This pattern is conceptually problematic:

```jsx
function Form() {
  const {
    pending,
  } = useFormStatus();

  return (
    <form action={save}>
      ...
    </form>
  );
}
```

The Hook reads status from a parent form, but this component is creating the form rather than being rendered inside it.

Extract a child:

```jsx
function SubmitButton() {
  const {
    pending,
  } = useFormStatus();

  // ...
}
```

This is a frequent interview trap.

---

# 6. Pending Submit Button ⭐⭐⭐⭐⭐

```jsx
function SubmitButton() {
  const {
    pending,
  } = useFormStatus();

  return (
    <button
      type="submit"
      disabled={pending}
    >
      {pending
        ? "Creating account..."
        : "Create account"}
    </button>
  );
}
```

This keeps pending behavior close to the button that needs it.

---

# 7. More Than pending

Modern `useFormStatus` status information includes values such as:

```text
pending
data
method
action
```

Conceptually:

- `pending`: whether parent form is submitting
- `data`: FormData being submitted
- `method`: submission method
- `action`: action associated with submission

Most everyday React UI primarily needs:

```text
pending
```

but understand that form status contains more context.

---

# 8. useFormStatus vs useActionState ⭐⭐⭐⭐⭐

These APIs overlap around pending UI but solve different problems.

`useActionState`:

```text
owns state returned by an Action
+
provides Action dispatch
+
provides pending status
```

`useFormStatus`:

```text
reads status of parent form
from a descendant component
```

Typical architecture:

```text
Form component
   ↓
useActionState
   ↓
state / formAction

SubmitButton child
   ↓
useFormStatus
   ↓
pending
```

---

# 9. Example Combining Them ⭐⭐⭐⭐⭐

```jsx
import {
  useActionState,
} from "react";

import {
  useFormStatus,
} from "react-dom";

const initialState = {
  message: "",
};

function SubmitButton() {
  const {
    pending,
  } = useFormStatus();

  return (
    <button
      disabled={pending}
    >
      {pending
        ? "Saving..."
        : "Save"}
    </button>
  );
}

function ProfileForm() {
  const [
    state,
    formAction,
  ] = useActionState(
    saveProfile,
    initialState
  );

  return (
    <form action={formAction}>
      <input name="name" />

      <SubmitButton />

      <p>{state.message}</p>
    </form>
  );
}
```

Each Hook is used where it fits best.

---

# 10. What Is Optimistic UI? ⭐⭐⭐⭐⭐

Optimistic UI updates the interface **before the real async operation has finished**, assuming it will probably succeed.

Normal/pessimistic:

```text
click
 ↓
wait for server
 ↓
success
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

This can make interfaces feel much faster.

---

# 11. Examples of Optimistic UI

Common examples:

- like button
- sending a message
- adding a comment
- marking a task complete
- following a user
- adding an item to a list

Suppose a message takes 800 ms to save.

Without optimism:

```text
Send
 ↓
800 ms waiting
 ↓
message appears
```

With optimism:

```text
Send
 ↓
message appears immediately
 ↓
server confirms later
```

---

# 12. What Is useOptimistic? ⭐⭐⭐⭐⭐

React provides:

```jsx
import {
  useOptimistic,
} from "react";
```

Conceptual syntax:

```jsx
const [
  optimisticState,
  addOptimistic,
] = useOptimistic(
  state,
  updateFn
);
```

It lets you temporarily render an optimistic version of state while an Action is pending.

---

# 13. Basic Optimistic Example ⭐⭐⭐⭐⭐

Suppose:

```jsx
function MessageList({
  messages,
  sendMessage,
}) {
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
        sending: true,
      },
    ]
  );

  // ...
}
```

When submission starts:

```text
real messages
+
new temporary message
        ↓
optimisticMessages
```

The user sees the message immediately.

---

# 14. Optimistic Update Function Must Be Pure ⭐⭐⭐⭐⭐

The optimistic update function:

```jsx
(
  currentState,
  optimisticValue
) => nextOptimisticState
```

should be pure.

Bad:

```js
currentState.push(
  optimisticValue
);

return currentState;
```

Better:

```js
return [
  ...currentState,
  optimisticValue,
];
```

This follows the same immutability principles learned for state.

---

# 15. Optimistic Message Form

Conceptual example:

```jsx
function Messages({
  messages,
}) {
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

  async function send(
    formData
  ) {
    const text =
      formData.get("message");

    addOptimisticMessage(text);

    await sendMessage(text);
  }

  return (
    <>
      <ul>
        {optimisticMessages.map(
          (message) => (
            <li key={message.id}>
              {message.text}

              {message.sending &&
                " (Sending...)"}
            </li>
          )
        )}
      </ul>

      <form action={send}>
        <input
          name="message"
        />

        <button>
          Send
        </button>
      </form>
    </>
  );
}
```

This demonstrates the mental model; production identity/reconciliation should use appropriate stable server/client identifiers.

---

# 16. Optimistic State Is Temporary ⭐⭐⭐⭐⭐

Think:

```text
authoritative state
      ↓
useOptimistic
      +
optimistic input
      ↓
temporary optimistic view
```

When the underlying authoritative state changes after the operation completes, the UI reconciles with the real result.

Optimistic state should not become a second independent source of truth.

---

# 17. Optimistic UI Is Not Fake Permanent State

Wrong mental model:

```text
optimistic state
=
new permanent source of truth
```

Correct:

```text
authoritative state
+
temporary assumption
=
optimistic view
```

Eventually:

```text
server/real result
wins
```

---

# 18. What If the Request Fails? ⭐⭐⭐⭐⭐

Optimistic interfaces must consider failure.

Example:

```text
user sends message
      ↓
message appears as "Sending..."
      ↓
request fails
      ↓
remove/revert or mark failed
      ↓
allow retry
```

Good UX may show:

```text
Failed to send — Retry
```

instead of silently removing user work.

---

# 19. Optimistic UI Requires Reconciliation Thinking

Suppose you optimistically add:

```js
{
  id: "temp-123",
  text: "Hello",
  sending: true
}
```

The server returns:

```js
{
  id: "server-987",
  text: "Hello"
}
```

Your application needs a strategy for matching temporary UI with authoritative data.

Possible concerns:

- temporary IDs
- server IDs
- duplicates
- ordering
- failure
- retries

React provides the UI primitive; your data model still matters.

---

# 20. When Optimistic UI Is Appropriate ⭐⭐⭐⭐⭐

Good candidates:

```text
high probability of success
+
easy to reconcile/recover
+
instant feedback improves UX
```

Examples:

- likes
- simple comments
- messages
- task toggles

Be more cautious with operations where pretending success could seriously mislead the user.

Examples:

- payment completed
- account permanently deleted
- security-sensitive permission change

You may still show immediate progress, but do not falsely represent an unconfirmed critical result as final success.

---

# 21. Pending UI vs Optimistic UI ⭐⭐⭐⭐⭐

Pending UI:

```text
request started
↓
show "Saving..."
↓
wait
↓
show result
```

Optimistic UI:

```text
request started
↓
show expected result immediately
↓
mark temporary/pending if useful
↓
confirm or recover
```

They can work together.

---

# 22. useFormStatus vs useOptimistic ⭐⭐⭐⭐⭐

`useFormStatus` answers:

> Is this parent form currently submitting, and what is its submission status?

`useOptimistic` answers:

> What temporary UI should I show while an Action is in progress?

Different responsibilities:

```text
useFormStatus
→ submission status

useOptimistic
→ temporary expected state
```

---

# 23. Three React 19 Form Tools Together ⭐⭐⭐⭐⭐

```text
useActionState
      ↓
Action result state
validation / success / error

useFormStatus
      ↓
parent form pending status

useOptimistic
      ↓
temporary expected UI
during async Action
```

These APIs complement each other.

---

# 24. CareerLoop Example: Optimistic Status Update

Suppose an application moves:

```text
Applied
→ Interview
```

You could immediately show:

```text
Interview (Updating...)
```

while the request runs.

If successful:

```text
Interview
```

If failed:

```text
Applied
+
"Update failed"
```

This makes the UI responsive without pretending the backend result is guaranteed.

---

# 25. CodeBuddy Example: Optimistic Connection Request

User clicks:

```text
Connect
```

Optimistic UI:

```text
Connect
   ↓
Request Sent...
```

Request succeeds:

```text
Request Sent
```

Request fails:

```text
Connect
+
error/retry feedback
```

This is a natural optimistic interaction.

---

# 26. Do Not Duplicate Form Pending State Unnecessarily

If a submit button can read:

```jsx
const {
  pending,
} = useFormStatus();
```

you may not need:

```jsx
const [pending, setPending] =
  useState(false);
```

Duplicated state can drift out of sync.

Use the state source that already represents the workflow.

---

# 27. Scope Matters ⭐⭐⭐⭐⭐

`useFormStatus` is tied to the parent form.

That is useful because different forms can have independent pending states.

```text
ProfileForm
 └── SubmitButton
     → ProfileForm status

PasswordForm
 └── SubmitButton
     → PasswordForm status
```

You do not need one global `isSubmitting` boolean for every form.

This is state colocation applied to form status.

---

# 28. Accessibility During Pending State

A disabled button can communicate that submission is unavailable, but also provide meaningful text:

```jsx
<button
  disabled={pending}
>
  {pending
    ? "Saving profile..."
    : "Save profile"}
</button>
```

For status messages, accessible live regions may be appropriate depending on the UI.

Do not rely only on visual spinners.

---

# 29. Common Mistakes ⭐⭐⭐⭐⭐

## Mistake 1 — Importing useFormStatus from react

Use `react-dom`.

## Mistake 2 — Calling useFormStatus outside the form it should observe

It reads the nearest parent form's status.

## Mistake 3 — Calling it in the component that creates the form and expecting that form's status

Extract a descendant component such as `SubmitButton`.

## Mistake 4 — Duplicating pending state unnecessarily

Prefer the form/Action status already provided.

## Mistake 5 — Treating optimistic state as authoritative permanent data

It is a temporary expected view.

## Mistake 6 — Ignoring failure/rollback behavior

Optimistic UX must have a recovery strategy.

## Mistake 7 — Mutating optimistic state

Keep the optimistic update function pure.

## Mistake 8 — Optimistically claiming critical success too early

Do not show unconfirmed payment/security-sensitive results as final success.

## Mistake 9 — Ignoring identity when optimistic items become server items

Plan temporary/server ID reconciliation.

## Mistake 10 — Using optimistic UI where normal pending feedback is clearer

Optimization of perceived speed should not reduce correctness or trust.

---

# 30. Interview Questions ⭐⭐⭐⭐⭐

## What is useFormStatus?

A React DOM Hook that gives a descendant component information about the submission status of its parent form.

## Where is useFormStatus imported from?

`react-dom`.

## Why is SubmitButton often extracted into its own component?

Because `useFormStatus` reads the status of a parent form, so the component using it needs to be rendered inside that form.

## What is optimistic UI?

UI that immediately displays the expected result of an async operation before authoritative confirmation arrives.

## What is useOptimistic?

A React Hook for deriving a temporary optimistic version of state while an Action is in progress.

## Is optimistic state the source of truth?

No. It is temporary UI layered over authoritative state.

## What happens when an optimistic request fails?

The UI should reconcile/revert or mark the operation failed and provide recovery where appropriate.

## useActionState vs useFormStatus?

`useActionState` manages state returned by an Action and exposes pending status. `useFormStatus` reads parent form submission status from a descendant.

## useFormStatus vs useOptimistic?

The first describes form submission status; the second creates temporary expected UI.

## Should payment success normally be optimistic?

Do not represent an unconfirmed payment as final success. Pending/progress UI is safer until authoritative confirmation.

---

# 31. Interview Scenario ⭐⭐⭐⭐⭐

You have:

```jsx
function SaveButton() {
  const {
    pending,
  } = useFormStatus();

  return (
    <button
      disabled={pending}
    >
      {pending
        ? "Saving..."
        : "Save"}
    </button>
  );
}
```

Where must it render?

```jsx
<form action={save}>
  ...
  <SaveButton />
</form>
```

because it reads the parent form's status.

---

# 32. Complete Form Architecture Mental Model ⭐⭐⭐⭐⭐

```text
                 FORM
                  │
          ┌───────┴────────┐
          ↓                ↓
       fields          SubmitButton
          │                │
          │          useFormStatus
          │                │
          │             pending
          │
          ↓
        Action
          │
    ┌─────┴──────┐
    ↓            ↓
validation     mutation
    │            │
    └──────┬─────┘
           ↓
     useActionState
           ↓
 error / success state

During suitable mutation:
           │
           ↓
      useOptimistic
           ↓
 temporary expected UI
           ↓
 authoritative result
           ↓
 reconcile
```

---

# 33. Section 4 Revision ⭐⭐⭐⭐⭐

You now understand the complete progression:

```text
Lesson 26
Forms fundamentals
controlled / uncontrolled
FormData / submit

        ↓

Lesson 27
Validation
client + server
pending/error/success

        ↓

Lesson 28
React 19 Actions
useActionState

        ↓

Lesson 29
useFormStatus
optimistic UI
```

---

# 34. Key Takeaways

- `useFormStatus` reads submission status from a parent form.
- Import `useFormStatus` from `react-dom`.
- A submit-button child is a natural place to use it.
- `pending` enables clear submission feedback and duplicate-click protection.
- `useActionState` and `useFormStatus` solve related but different problems.
- Optimistic UI displays the expected result before the async operation is confirmed.
- `useOptimistic` helps derive temporary optimistic state.
- Optimistic update functions should be pure.
- Optimistic state is not authoritative state.
- Failure and reconciliation must be designed explicitly.
- Temporary and server identities may need reconciliation.
- Pending UI and optimistic UI can be combined.
- Use optimism when success is likely and recovery is clear.
- Do not falsely represent unconfirmed critical operations as final success.
- Form status should stay scoped to the form that owns the submission.
- React 19's form APIs reduce manual coordination while preserving normal validation and security responsibilities.

---

## Section 4 — Forms Completed ✅

You have completed:

- Lesson 26 — Forms in React
- Lesson 27 — Form Validation and Submission Patterns
- Lesson 28 — React 19 Form Actions and useActionState ⭐⭐⭐⭐⭐
- Lesson 29 — useFormStatus and Optimistic UI

---

## Next Section — Sharing and Managing State

➡️ [Lesson 30 — Context API and useContext ⭐⭐⭐⭐⭐](../05-state-management/30-context-usecontext.md)
