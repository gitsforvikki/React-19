# Lesson 28 — React 19 Form Actions and useActionState ⭐⭐⭐⭐⭐

## 1. Why React 19 Form Actions Matter

Traditional async forms often require manual code for:

```text
preventDefault
pending state
error state
async submission
success state
reset behavior
optimistic feedback
```

React 19 introduces a stronger model around **Actions**.

An Action is a function used for an async transition or mutation.

Forms can call Actions directly:

```jsx
<form action={actionFunction}>
```

React can then coordinate the submission with form-specific APIs.

---

# 2. Traditional Submission vs Action

Traditional:

```jsx
<form onSubmit={handleSubmit}>
```

where you often:

```text
preventDefault()
create FormData
set pending
try request
catch error
finally clear pending
```

React 19 Action:

```jsx
<form action={saveProfile}>
```

where React passes:

```text
FormData
```

to the Action.

---

# 3. Basic Form Action ⭐⭐⭐⭐⭐

```jsx
function ProfileForm() {
  async function saveProfile(
    formData
  ) {
    const name =
      formData.get("name");

    await updateProfile({
      name,
    });
  }

  return (
    <form action={saveProfile}>
      <input
        name="name"
        required
      />

      <button type="submit">
        Save
      </button>
    </form>
  );
}
```

Flow:

```text
user submits
     ↓
<form action={...}>
     ↓
React creates/passes FormData
     ↓
Action runs
     ↓
async mutation completes
```

---

# 4. name Is Essential

```jsx
<input
  name="name"
/>
```

The Action receives a `FormData` object.

Read:

```jsx
const name =
  formData.get("name");
```

Without a useful `name`, the field will not be available as expected in the form submission.

---

# 5. Actions Are Not Limited to Server Code ⭐⭐⭐⭐⭐

The word **Action** is a React concept.

Do not automatically interpret:

```text
Action
=
Server Action
```

React Actions can participate in client-side async workflows.

Frameworks may provide additional server-side Action capabilities, but those are framework-specific.

This React repository focuses on the React concepts.

---

# 6. React Action Mental Model ⭐⭐⭐⭐⭐

```text
User intent
   ↓
Action starts
   ↓
async mutation
   ↓
React coordinates transition
   ↓
pending / result UI
```

Actions are especially useful for mutations:

- create
- update
- delete
- submit
- send
- purchase
- save

---

# 7. What Is useActionState? ⭐⭐⭐⭐⭐

`useActionState` manages state based on the result of an Action.

Import:

```jsx
import {
  useActionState,
} from "react";
```

Basic shape:

```jsx
const [
  state,
  formAction,
  isPending,
] = useActionState(
  action,
  initialState
);
```

It returns:

```text
state
→ latest returned Action state

formAction
→ Action to pass to form/action

isPending
→ whether Action is pending
```

---

# 8. useActionState Action Signature ⭐⭐⭐⭐⭐

When wrapped by `useActionState`, your Action receives the previous state first.

```jsx
async function submitForm(
  previousState,
  formData
) {
  // ...
}
```

This is a common interview trap.

Without `useActionState`:

```jsx
async function action(
  formData
) {}
```

With `useActionState`:

```jsx
async function action(
  previousState,
  formData
) {}
```

---

# 9. Basic useActionState Example ⭐⭐⭐⭐⭐

```jsx
import {
  useActionState,
} from "react";

const initialState = {
  message: "",
};

function SignupForm() {
  async function signup(
    previousState,
    formData
  ) {
    const email =
      formData.get("email");

    if (!email) {
      return {
        message:
          "Email is required",
      };
    }

    await createUser({
      email,
    });

    return {
      message:
        "Account created",
    };
  }

  const [
    state,
    formAction,
    isPending,
  ] = useActionState(
    signup,
    initialState
  );

  return (
    <form action={formAction}>
      <input
        name="email"
        type="email"
      />

      <button
        disabled={isPending}
      >
        {isPending
          ? "Creating..."
          : "Create account"}
      </button>

      <p>{state.message}</p>
    </form>
  );
}
```

---

# 10. Data Flow Diagram ⭐⭐⭐⭐⭐

```text
initialState
     ↓
useActionState
     ↓
state + formAction + isPending
     ↓
<form action={formAction}>
     ↓
submit
     ↓
action(previousState, formData)
     ↓
return next state
     ↓
component renders returned state
```

---

# 11. Returning Validation Errors ⭐⭐⭐⭐⭐

```jsx
const initialState = {
  success: false,
  message: "",
  errors: {},
};

async function createAccount(
  previousState,
  formData
) {
  const email =
    formData.get("email");

  const password =
    formData.get("password");

  const errors = {};

  if (!email) {
    errors.email =
      "Email is required";
  }

  if (
    !password ||
    password.length < 8
  ) {
    errors.password =
      "Use at least 8 characters";
  }

  if (
    Object.keys(errors).length
  ) {
    return {
      success: false,
      message:
        "Please correct the form",
      errors,
    };
  }

  await register({
    email,
    password,
  });

  return {
    success: true,
    message:
      "Account created",
    errors: {},
  };
}
```

The returned object becomes the next Action state.

---

# 12. Render Field Errors

```jsx
<input
  name="email"
  type="email"
  aria-invalid={
    Boolean(
      state.errors.email
    )
  }
/>

{state.errors.email && (
  <p>
    {state.errors.email}
  </p>
)}
```

The Action result can naturally represent server-side validation.

---

# 13. Why useActionState Is Useful ⭐⭐⭐⭐⭐

Without it you may manually coordinate:

```text
form state
pending state
server errors
success result
async lifecycle
```

With `useActionState`:

```text
Action returns state
       ↓
React exposes latest result
       +
pending status
```

This makes mutation-oriented UI easier to model.

---

# 14. isPending ⭐⭐⭐⭐⭐

```jsx
const [
  state,
  formAction,
  isPending,
] = useActionState(
  saveApplication,
  initialState
);
```

Then:

```jsx
<button
  disabled={isPending}
>
  {isPending
    ? "Saving..."
    : "Save"}
</button>
```

This gives pending state directly for that Action workflow.

---

# 15. useActionState vs useState ⭐⭐⭐⭐⭐

`useState`:

```text
general local state
```

`useActionState`:

```text
state produced by an Action
+
Action dispatch workflow
+
pending status
```

Do not replace all `useState` with `useActionState`.

Use it when state is naturally the result of an Action/mutation.

---

# 16. useActionState vs useReducer

Both can calculate new state from previous state.

But their purposes differ.

```text
useReducer
→ general state transitions
→ synchronous reducer model

useActionState
→ Action result state
→ async/action workflow
→ pending integration
```

`useReducer` is covered in Section 5.

---

# 17. previousState ⭐⭐⭐⭐⭐

Because the Action receives previous state:

```jsx
async function increment(
  previousState,
  formData
) {
  return previousState + 1;
}
```

it can derive the next result from earlier Action state.

But do not force previous state into the logic when it is not needed.

It is simply part of the `useActionState` Action signature.

---

# 18. Passing Additional Arguments with bind ⭐⭐⭐⭐⭐

Suppose an Action needs a product ID plus FormData.

You can bind an argument:

```jsx
function Product({
  productId,
}) {
  const updateWithId =
    updateProduct.bind(
      null,
      productId
    );

  return (
    <form
      action={updateWithId}
    >
      <input name="quantity" />

      <button>
        Update
      </button>
    </form>
  );
}
```

Conceptually:

```text
productId
+
FormData
→ Action
```

With `useActionState`, be especially careful about argument order because previous state is inserted by the Hook.

---

# 19. Action Validation Still Matters

Actions do not remove validation requirements.

You still need:

```text
parse input
validate
authorize where trusted
perform mutation
return safe result
```

React improves UI coordination; it does not automatically make data valid or secure.

---

# 20. Do Not Trust FormData ⭐⭐⭐⭐⭐

Even if:

```jsx
<input
  type="number"
  min="1"
/>
```

you must still validate at the trusted mutation boundary.

FormData values are external input.

React form Actions do not change that security rule.

---

# 21. Form Action and Uncontrolled Inputs

Actions pair naturally with native form fields:

```jsx
<form action={formAction}>
  <input
    name="company"
  />

  <input
    name="role"
  />

  <button>
    Save
  </button>
</form>
```

You do not need state for every keystroke if the values are only required during submission.

This can reduce unnecessary form state.

---

# 22. Controlled Fields Still Have a Place

You may still need controlled state for:

- live search
- dependent selects
- live preview
- conditional fields
- character count
- complex client interaction

Actions do not eliminate controlled components.

Use each tool according to the UI requirement.

---

# 23. Action Errors

Expected form errors should often become useful returned state.

Example:

```js
return {
  success: false,
  message:
    "Please fix the highlighted fields",
  errors,
};
```

Unexpected system failures may require a different error handling strategy.

Do not expose stack traces or sensitive infrastructure information to users.

---

# 24. Action State Should Be Serializable-Looking and UI-Friendly

A practical state shape:

```js
{
  success: false,
  message: "",
  errors: {
    email: ""
  }
}
```

is easier for UI rendering than throwing for every expected validation problem.

Separate:

```text
expected user-correctable failure
from
unexpected application failure
```

---

# 25. React 19 Form Reset Behavior ⭐⭐⭐⭐⭐

When a form Action succeeds, uncontrolled form fields can participate in React's form reset behavior.

However, controlled fields are still controlled by your React state and must be updated through that state if you want them reset.

Mental model:

```text
uncontrolled native fields
→ browser/React form behavior

controlled fields
→ React state remains source of truth
```

Do not assume an Action magically resets controlled state.

---

# 26. Progressive Enhancement Concept

React's form Action model is designed to align closely with native forms.

In frameworks that support server functions, forms can also participate in progressive-enhancement workflows.

The exact server/navigation implementation is framework-specific.

For this React lesson, remember:

```text
React form Actions
build on native form semantics
rather than replacing them
```

---

# 27. CareerLoop Example ⭐⭐⭐⭐⭐

```jsx
import {
  useActionState,
} from "react";

const initialState = {
  success: false,
  message: "",
  errors: {},
};

async function saveApplication(
  previousState,
  formData
) {
  const company =
    formData.get("company");

  const role =
    formData.get("role");

  const errors = {};

  if (!company?.trim()) {
    errors.company =
      "Company is required";
  }

  if (!role?.trim()) {
    errors.role =
      "Role is required";
  }

  if (
    Object.keys(errors).length
  ) {
    return {
      success: false,
      message:
        "Please correct the form",
      errors,
    };
  }

  await createApplication({
    company,
    role,
  });

  return {
    success: true,
    message:
      "Application saved",
    errors: {},
  };
}

function ApplicationForm() {
  const [
    state,
    formAction,
    isPending,
  ] = useActionState(
    saveApplication,
    initialState
  );

  return (
    <form action={formAction}>
      <input
        name="company"
        aria-invalid={
          Boolean(
            state.errors.company
          )
        }
      />

      {state.errors.company && (
        <p>
          {state.errors.company}
        </p>
      )}

      <input
        name="role"
        aria-invalid={
          Boolean(
            state.errors.role
          )
        }
      />

      {state.errors.role && (
        <p>
          {state.errors.role}
        </p>
      )}

      <button
        disabled={isPending}
      >
        {isPending
          ? "Saving..."
          : "Save application"}
      </button>

      {state.message && (
        <p>{state.message}</p>
      )}
    </form>
  );
}
```

---

# 28. Common Mistakes ⭐⭐⭐⭐⭐

## Mistake 1 — Confusing React Actions with framework Server Actions

Actions are a React concept; server execution details depend on the environment/framework.

## Mistake 2 — Forgetting previousState in useActionState Actions

The wrapped Action receives previous state before the normal Action arguments.

## Mistake 3 — Forgetting name attributes

FormData depends on named form controls.

## Mistake 4 — Trusting client FormData

Always validate at the trusted mutation boundary.

## Mistake 5 — Using useActionState for unrelated general state

It is designed around Action-produced state.

## Mistake 6 — Keeping unnecessary duplicate pending state

If `isPending` already models the Action's pending state, do not duplicate it without a reason.

## Mistake 7 — Expecting Actions to eliminate controlled inputs

Controlled state is still useful for interactive form behavior.

## Mistake 8 — Returning unsafe internal errors to users

Return safe, actionable UI state.

## Mistake 9 — Forgetting accessibility for returned field errors

Connect errors to fields and communicate invalid state.

## Mistake 10 — Assuming frontend Actions enforce authorization

Trusted authorization must happen where the mutation is actually protected.

---

# 29. Interview Questions ⭐⭐⭐⭐⭐

## What is an Action in React 19?

An Action is a function used for mutations or async transitions that React can coordinate with pending and result UI.

## How can a form call an Action?

By passing a function to the form's `action` prop.

## What does the form Action receive?

For a normal function Action, React provides the submitted `FormData`.

## What does useActionState return?

The current Action state, a dispatch/action function, and an `isPending` boolean.

## What is the useActionState Action signature?

The Action receives the previous Action state as its first argument, followed by the normal action arguments such as FormData.

## useState vs useActionState?

`useState` is general local state. `useActionState` manages state produced by an Action and integrates with its pending workflow.

## Do Actions replace validation?

No.

## Are React Actions the same as Server Actions?

No. React defines Action concepts; frameworks may provide server-side execution mechanisms.

## Do Actions eliminate controlled inputs?

No. Controlled fields remain useful when React needs live field values.

## Why are Actions useful for forms?

They align async mutations, returned form state, and pending UI with the form submission workflow.

---

# 30. Complete Mental Model ⭐⭐⭐⭐⭐

```text
          FORM
           │
           ↓
 action={formAction}
           │
           ↓
      user submits
           │
           ↓
       FormData
           │
           ↓
Action(previousState, formData)
           │
      ┌────┴────┐
      ↓         ↓
 validation   mutation
 error         success
      │         │
      └────┬────┘
           ↓
    return next state
           ↓
      useActionState
           ↓
 state + isPending
           ↓
        render UI
```

---

# 31. Quick Revision ⭐⭐⭐⭐⭐

```jsx
const [
  state,
  formAction,
  isPending,
] = useActionState(
  action,
  initialState
);
```

Action:

```jsx
async function action(
  previousState,
  formData
) {
  // validate
  // mutate

  return nextState;
}
```

Form:

```jsx
<form action={formAction}>
  ...
</form>
```

Remember:

```text
Action
→ mutation workflow

useActionState
→ result state + pending

FormData
→ submitted fields

server/trusted boundary
→ validate + authorize
```

---

# 32. Key Takeaways

- React 19 Actions improve async mutation workflows.
- Forms can receive functions through the `action` prop.
- A form Action receives submitted FormData.
- `useActionState` connects Action results to rendered state.
- It returns state, an Action function, and pending status.
- The wrapped Action receives previous state as its first argument.
- Action state is useful for validation errors, messages, and mutation results.
- Actions work naturally with uncontrolled/native form fields.
- Controlled inputs remain useful when live React interaction is required.
- React Actions are not synonymous with framework Server Actions.
- Actions do not replace input validation, authorization, or backend correctness.
- Do not duplicate pending state when React already provides the state you need.
- Use accessible field-error UI.
- Keep expected user-correctable failures separate from unexpected system failures.
- Form Actions build on native form semantics.
- The most important interview detail is the `useActionState` flow: **previous state + Action arguments → returned next state + pending UI**.

---

## Next Lesson

➡️ [Lesson 29 — useFormStatus and Optimistic UI](./29-useformstatus-optimistic-ui.md)
