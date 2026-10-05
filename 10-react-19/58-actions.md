# Lesson 58 — Actions ⭐⭐⭐⭐⭐

## 1. What Is an Action?

React 19 introduces a stronger model for coordinating asynchronous mutations.

By convention, a function participating in an async Transition is called an **Action**.

Typical mutation:

```text
user submits/change
       ↓
start async work
       ↓
pending UI
       ↓
server/API mutation
   ┌───┴────┐
 success   error
   ↓         ↓
update UI   recover/report
```

Actions integrate this workflow with React's rendering model.

---

## 2. Why Actions Matter ⭐⭐⭐⭐⭐

Without an Action-oriented abstraction, mutation code often manually coordinates:

```jsx
const [isPending, setIsPending] =
  useState(false);
const [error, setError] =
  useState(null);

async function handleSave() {
  setIsPending(true);
  setError(null);

  try {
    await save();
  } catch (error) {
    setError(error);
  } finally {
    setIsPending(false);
  }
}
```

This is valid, but repetitive.

React 19 provides primitives for common mutation concerns:

- pending state
- Action result state
- form integration
- optimistic updates
- error-boundary integration

---

## 3. Action vs Event Handler ⭐⭐⭐⭐⭐

An event handler describes the user interaction:

```jsx
function handleClick() {
  // user clicked
}
```

An Action represents mutation work that can participate in React's transition/action semantics:

```jsx
async function updateProfileAction(
  profile
) {
  await updateProfile(profile);
}
```

They can work together:

```text
event
  ↓
start Action
  ↓
async mutation
```

Do not use the words interchangeably.

---

## 4. Action vs Server Function ⭐⭐⭐⭐⭐

This is one of the most important distinctions.

```text
React Action
────────────
async mutation/Transition concept
can be client-side

Server Function
───────────────
function executed on server
requires server/framework integration
often used as an Action
```

Therefore:

> An Action is not automatically a Server Function.

---

# Part 1 — Actions with Transitions

## 5. startTransition with Async Work

Conceptually:

```jsx
startTransition(async () => {
  await updateName(name);

  // update UI
});
```

The async function participating in the Transition is conventionally called an Action.

React can track pending work around this transition-oriented workflow.

---

## 6. Important Async Transition Caveat ⭐⭐⭐⭐⭐

After an `await`, state updates that need to be marked as Transition updates may require another `startTransition`.

Conceptually:

```jsx
startTransition(async () => {
  await save();

  startTransition(() => {
    setResult("saved");
  });
});
```

This is a current React limitation to understand when manually coordinating async Transitions.

Action-specific APIs can handle common workflows more conveniently.

---

# Part 2 — Form Actions

## 7. Function-Based form action ⭐⭐⭐⭐⭐

React DOM lets you pass a function directly to `action`:

```jsx
async function saveApplication(
  formData
) {
  const company =
    formData.get("company");

  await saveToAPI({
    company,
  });
}

function ApplicationForm() {
  return (
    <form
      action={saveApplication}
    >
      <input
        name="company"
      />

      <button type="submit">
        Save
      </button>
    </form>
  );
}
```

React passes a `FormData` object to the Action.

---

## 8. No preventDefault Required

Traditional:

```jsx
function handleSubmit(event) {
  event.preventDefault();
  // ...
}

<form onSubmit={handleSubmit}>
```

Action model:

```jsx
<form action={saveApplication}>
```

React handles the function-based submission.

Do not combine patterns mechanically.

---

## 9. FormData ⭐⭐⭐⭐⭐

Inputs need names:

```jsx
<input
  name="company"
/>
```

Then:

```js
const company =
  formData.get("company");
```

Without a `name`, a normal form field is not represented under that key in submitted `FormData`.

---

## 10. Automatic Uncontrolled Form Reset

After a function-based form Action succeeds, React resets uncontrolled fields.

```jsx
<form action={save}>
  <input
    name="company"
    defaultValue=""
  />
</form>
```

Do not confuse this with controlled input state:

```jsx
<input
  value={company}
  onChange={...}
/>
```

Controlled state remains governed by React state.

---

## 11. Multiple Submit Actions with formAction

```jsx
<form action={saveDraft}>
  <input name="title" />

  <button type="submit">
    Save draft
  </button>

  <button
    type="submit"
    formAction={publish}
  >
    Publish
  </button>
</form>
```

This allows different submit buttons to perform different Actions.

---

# Part 3 — useActionState

## 12. useActionState ⭐⭐⭐⭐⭐

Import:

```jsx
import {
  useActionState,
} from "react";
```

Syntax:

```jsx
const [
  state,
  dispatchAction,
  isPending,
] = useActionState(
  reducerAction,
  initialState
);
```

It gives you:

```text
state
→ latest Action result

dispatchAction
→ trigger/wrapped Action

isPending
→ whether queued Action work is pending
```

---

## 13. Action Function Signature ⭐⭐⭐⭐⭐

With `useActionState`, the Action receives the previous state first.

```jsx
async function action(
  previousState,
  payload
) {
  // ...
  return nextState;
}
```

This matters especially with forms.

Without `useActionState`:

```jsx
async function save(
  formData
) {}
```

With `useActionState`:

```jsx
async function save(
  previousState,
  formData
) {}
```

A common bug is reading `FormData` from the wrong parameter.

---

## 14. Form Example

```jsx
const initialState = {
  error: null,
  success: false,
};

async function createApplication(
  previousState,
  formData
) {
  const company =
    formData.get("company");

  if (!company) {
    return {
      error:
        "Company is required.",
      success: false,
    };
  }

  await saveApplication({
    company,
  });

  return {
    error: null,
    success: true,
  };
}

function ApplicationForm() {
  const [
    state,
    submitAction,
    isPending,
  ] = useActionState(
    createApplication,
    initialState
  );

  return (
    <form action={submitAction}>
      <input name="company" />

      <button
        disabled={isPending}
      >
        {isPending
          ? "Saving..."
          : "Save"}
      </button>

      {state.error && (
        <p>{state.error}</p>
      )}
    </form>
  );
}
```

---

## 15. Known Errors vs Unexpected Errors ⭐⭐⭐⭐⭐

Expected business/validation failure:

```jsx
return {
  error:
    "Application already exists.",
};
```

Unexpected programming/system failure:

```jsx
throw error;
```

With `useActionState`, thrown errors can propagate to the nearest Error Boundary and queued Actions are cancelled.

Do not treat every validation message as an exception.

---

## 16. useActionState vs useReducer ⭐⭐⭐⭐⭐

They may look similar:

```text
previous state
+ payload
→ next state
```

But they have different purposes.

### useReducer

```text
UI state management
reducer should be pure
synchronous reducer model
```

### useActionState

```text
Action result/state management
Action may be async
Action may perform side effects
integrates with Action pending semantics
```

Do not use `useActionState` as a general replacement for `useReducer`.

---

## 17. Sequential Queuing ⭐⭐⭐⭐⭐

Multiple dispatches for the same `useActionState` are queued and executed sequentially.

Why?

Each Action receives the result of the previous Action:

```text
initial state = S0

Action A
S0 → S1

Action B
must receive S1
S1 → S2

Action C
must receive S2
S2 → S3
```

Therefore React cannot simply run all of them independently in parallel while preserving this state-reduction contract.

---

## 18. When Sequential Queuing Matters

Imagine clicking "Add" four times quickly.

If each Action waits one second:

```text
Action 1
  ↓
Action 2
  ↓
Action 3
  ↓
Action 4
```

The queue can take roughly the sum of their times.

For immediate feedback, `useOptimistic` may improve UX.

For independent parallel mutations, a different state/Transition architecture may be more appropriate.

---

## 19. Manual dispatchAction Must Be in an Action Context ⭐⭐⭐⭐⭐

If you manually call the dispatcher:

```jsx
dispatchAction(payload);
```

outside an Action context, pending semantics will not work correctly and React warns in development.

Use:

```jsx
startTransition(() => {
  dispatchAction(payload);
});
```

or pass the dispatcher to an Action prop such as:

```jsx
<form action={dispatchAction}>
```

where React provides the Action context automatically.

---

## 20. Stable Dispatcher Identity

The `dispatchAction` returned by `useActionState` has stable identity.

As with other stable React dispatch functions, dependency handling should follow the linter and actual reactive values.

Do not build unnecessary memoization around it.

---

## 21. No Built-In Reset Function

`useActionState` does not provide a dedicated built-in reset function.

Possible reset strategies include:

- designing the Action to handle a reset payload,
- remounting the state owner with a meaningful `key`,
- using form behavior where successful form submission resets uncontrolled fields.

Do not invent a nonexistent `resetActionState()` API.

---

# Part 4 — useFormStatus

## 22. useFormStatus ⭐⭐⭐⭐⭐

Import it from:

```jsx
import {
  useFormStatus,
} from "react-dom";
```

A descendant component can inspect its parent form:

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

## 23. Why useFormStatus Is Useful

Without it:

```text
Form
  │
  └── pending prop
       ↓
   Button wrapper
       ↓
   SubmitButton
```

With it:

```text
<form>
  │
  └── descendant SubmitButton
          ↓
      useFormStatus()
```

This is especially useful in design systems.

---

## 24. Parent Form Relationship

`useFormStatus` reads status from a parent form.

Do not call it in the same component and expect it to observe a form that the component returns below the Hook.

A common pattern is:

```jsx
function Form() {
  return (
    <form action={action}>
      <SubmitButton />
    </form>
  );
}

function SubmitButton() {
  const status =
    useFormStatus();

  // ...
}
```

The status consumer is inside the form subtree.

---

# Part 5 — Optimistic UI

## 25. Actions + useOptimistic

Actions work naturally with optimistic updates.

```text
confirmed state
      │
user mutation
      ↓
optimistic state immediately
      │
Action pending
      ↓
server result
   ┌──┴───┐
success  failure
   ↓       ↓
confirm  revert/reconcile
```

Lesson 60 covers `useOptimistic` in detail.

---

# Part 6 — Error Handling

## 26. Error Boundaries

Actions can integrate with React's error model.

For unexpected failures:

```text
Action throws
    ↓
React error propagation
    ↓
nearest Error Boundary
```

This does not eliminate the need for domain-specific error states.

---

## 27. Validation Errors

Expected:

```jsx
return {
  error:
    "Please enter a valid URL.",
};
```

Unexpected:

```jsx
throw new Error(
  "Database response malformed"
);
```

Good applications distinguish recoverable user-facing states from exceptional failures.

---

# Part 7 — Server Functions

## 28. Server Function as a Form Action

In a compatible React Server Components framework:

```jsx
async function save(
  formData
) {
  "use server";

  // server-only mutation
}
```

can be passed to:

```jsx
<form action={save}>
```

The framework creates the server/client transport.

Core React concepts should not be confused with framework-specific implementation.

---

## 29. Progressive Enhancement

Server Functions used with forms can support submission before client JavaScript has loaded in compatible server-rendering setups.

`useActionState` also has an optional `permalink` parameter for specific progressive-enhancement scenarios.

This is advanced framework/server integration.

Know what it is, but keep implementation details in the framework repository.

---

## 30. Security Reminder ⭐⭐⭐⭐⭐

A Server Function must be treated like a server endpoint.

Never assume:

```text
"called from my React UI"
→ trusted request
```

Server-side mutations still require appropriate:

- authentication
- authorization
- input validation
- business-rule validation

UI visibility is not security.

---

# Part 8 — Practical Examples

## 31. CareerLoop — Add Application

```jsx
const initialState = {
  error: null,
};

async function addApplication(
  previousState,
  formData
) {
  const company =
    formData.get("company");

  const role =
    formData.get("role");

  if (!company || !role) {
    return {
      error:
        "Company and role are required.",
    };
  }

  await createApplication({
    company,
    role,
  });

  return {
    error: null,
  };
}
```

UI:

```jsx
function ApplicationForm() {
  const [
    state,
    action,
    isPending,
  ] = useActionState(
    addApplication,
    initialState
  );

  return (
    <form action={action}>
      <input name="company" />
      <input name="role" />

      <button
        disabled={isPending}
      >
        {isPending
          ? "Adding..."
          : "Add application"}
      </button>

      {state.error && (
        <p>{state.error}</p>
      )}
    </form>
  );
}
```

---

## 32. CodeBuddy — Connection Request

Conceptual Action flow:

```text
Connect button
     ↓
Action
     ↓
POST connection request
     ↓
pending UI
     ↓
result
  ┌──┴──┐
success error
```

If immediate feedback is desirable:

```text
Action
+
useOptimistic
```

can represent "Requested" while the network operation is still pending.

---

# Part 9 — Choosing the Right Tool

## 33. Decision Guide ⭐⭐⭐⭐⭐

```text
Normal synchronous UI state?
→ useState / useReducer

Async mutation with Action result?
→ useActionState

Need form descendant pending state?
→ useFormStatus

Need immediate expected mutation UI?
→ useOptimistic

Need low-level non-urgent rendering priority?
→ useTransition / startTransition

Need server execution?
→ Server Function + framework infrastructure
```

These APIs complement rather than replace one another.

---

## 34. Actions vs useEffect ⭐⭐⭐⭐⭐

Do not put user-triggered mutations into Effects merely because they are asynchronous.

Bad mental model:

```text
user clicks
→ set flag
→ Effect notices flag
→ perform mutation
```

If the mutation is caused by a user action, perform it from the event/Action workflow.

Effects are for synchronization with external systems caused by rendering/lifecycle, not for hiding event logic.

---

## 35. Actions vs Data Fetching

Actions are primarily mutation-oriented.

Examples:

- submit form
- update profile
- send message
- add to cart
- change application status

Reading/querying data has different concerns such as Suspense resources, caching, server rendering, or data libraries.

Do not call every async function an Action.

---

## 36. Common Mistakes ⭐⭐⭐⭐⭐

1. Calling every Action a Server Action.
2. Forgetting that `useActionState` passes previous state first.
3. Reading FormData from the wrong argument.
4. Calling the dispatcher manually outside an Action/Transition context.
5. Assuming Actions replace all application state.
6. Assuming successful form reset clears controlled state.
7. Treating expected validation errors as crashes.
8. Using Effects for user-triggered mutations.
9. Ignoring sequential queuing in `useActionState`.
10. Using `useActionState` when independent work needs parallel execution.
11. Assuming client UI checks secure Server Functions.
12. Forgetting that traditional `onSubmit` remains valid.

---

## 37. Interview Questions ⭐⭐⭐⭐⭐

### What is an Action in React 19?

A function participating in React's transition/action model, commonly for asynchronous mutations.

### Is an Action the same as a Server Action?

No. Actions can be client-side; Server Functions execute on the server.

### What does useActionState return?

Current Action result state, an Action dispatcher, and an `isPending` flag.

### What arguments does the useActionState Action receive?

Previous state first, then the dispatched payload. For form Actions, the FormData is therefore the second argument.

### Can the Action be async?

Yes.

### useActionState vs useReducer?

`useReducer` manages UI state with a pure reducer; `useActionState` manages Action state and its reducer Action may perform side effects and be async.

### How are repeated useActionState dispatches processed?

They are queued sequentially because each Action receives the previous Action's result.

### How do you manually call dispatchAction?

Inside an Action context, commonly by wrapping it with `startTransition`, unless React invokes it through an Action prop.

### What does useFormStatus do?

It lets descendants read the status of their parent form.

### What happens after a successful function form Action?

Uncontrolled fields are automatically reset.

### Are Actions only for forms?

No.

---

## 38. Interview-Ready Answer ⭐⭐⭐⭐⭐

```text
React 19 Actions are a model for coordinating
asynchronous mutations with React's transition system.

They can integrate pending state, errors, forms, and
optimistic updates. Actions are not automatically
server-side; a Server Function can be used as an
Action, but the concepts are different.

useActionState is useful when I need the result and
pending state of an Action. Its Action receives the
previous state first and the payload second, and
multiple dispatches are processed sequentially.

For forms, React can call a function passed to action
with FormData and automatically reset uncontrolled
fields after a successful submission. useFormStatus
lets descendant form controls observe pending state.

I still use normal state, event handlers, and data
fetching patterns where Actions are not the right
abstraction.
```

---

## 39. Complete Mental Model ⭐⭐⭐⭐⭐

```text
                 USER MUTATION
                      │
                      ↓
                    Action
                      │
        ┌─────────────┼──────────────┐
        ↓             ↓              ↓
    pending        result          error
        │             │              │
        ↓             ↓              ↓
useActionState   next UI state   ErrorBoundary
        │
        ├── form action
        ├── useFormStatus
        └── useOptimistic
                 │
                 ↓
          responsive mutation UI

Server Function?
→ optional server implementation
→ not the definition of Action
```

---

## 40. Key Takeaways

- Actions model async mutation workflows.
- Actions are not synonymous with Server Functions.
- Function form Actions receive `FormData`.
- Action-based forms do not require manual `preventDefault`.
- Successful form Actions reset uncontrolled fields.
- `formAction` can select different submit Actions.
- `useActionState` returns state, dispatcher, and pending status.
- Its Action receives previous state before its payload.
- Actions may be async and perform side effects.
- Repeated `useActionState` dispatches are sequentially queued.
- Manual dispatcher calls need an Action/Transition context.
- `useFormStatus` reads parent form status.
- `useOptimistic` complements Actions for immediate feedback.
- Expected validation errors and unexpected exceptions should be modeled differently.
- Server Functions still require real server-side security.
- Actions do not replace normal state, Effects, or data-fetching architecture.

---

## Next Lesson

➡️ [Lesson 59 — use() API ⭐⭐⭐⭐⭐](./59-use-api.md)
