# Lesson 28 — React 19 Form Actions and useActionState

React 19 improves async form workflows with **Actions** and `useActionState`.

The main idea:

> A form can call an async function directly, and React can connect its result and pending state to the UI.

---

## 1. Basic Form Action

```jsx
async function saveProfile(
  formData
) {
  const name =
    formData.get("name");

  await updateProfile({
    name,
  });
}

function ProfileForm() {
  return (
    <form action={saveProfile}>
      <input
        name="name"
        required
      />

      <button>
        Save
      </button>
    </form>
  );
}
```

Flow:

```text
submit form
↓
React creates FormData
↓
Action runs
↓
async work completes
```

---

## 2. Action Is Not the Same as Server Action

Do not confuse:

```text
React Action
```

with:

```text
framework Server Action
```

React defines the Action pattern.

Frameworks such as Next.js may add server-side behavior on top of it.

---

## 3. What Is useActionState?

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

It gives you:

```text
state
→ latest value returned by the Action

formAction
→ function used by the form

isPending
→ whether the Action is running
```

---

## 4. Important Action Signature

With normal form Action:

```jsx
async function action(
  formData
) {
  // ...
}
```

With `useActionState`:

```jsx
async function action(
  previousState,
  formData
) {
  // ...
}
```

This is an important interview point.

---

## 5. Basic useActionState Example

```jsx
import {
  useActionState,
} from "react";

const initialState = {
  message: "",
  errors: {},
};

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
      errors: {
        email:
          "Email is required",
      },
    };
  }

  await createUser({
    email,
  });

  return {
    message:
      "Account created",
    errors: {},
  };
}

function SignupForm() {
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

## 6. Action State Is Good for Validation Results

An Action can return:

```js
{
  success: false,
  message:
    "Please correct the form",
  errors: {
    email:
      "Invalid email"
  }
}
```

That returned object becomes the new `state`.

This works well for:

- field errors
- success messages
- mutation results

---

## 7. FormData Still Needs name

```jsx
<input
  name="company"
/>
```

Then:

```js
formData.get("company")
```

Without a useful `name`, the field will not be available as expected.

---

## 8. Actions Do Not Replace Validation or Security

You still need to:

```text
read input
↓
validate
↓
authorize
↓
perform mutation
↓
return safe result
```

Never trust FormData just because it came through a React Action.

---

## 9. Controlled Inputs Still Matter

Actions work nicely with uncontrolled/native form fields.

But controlled fields are still useful for:

- live validation
- dependent fields
- live preview
- conditional UI
- character count

React 19 Actions do not replace controlled inputs.

---

## 10. useState vs useActionState

`useState`:

```text
general component state
```

`useActionState`:

```text
state produced by an Action
+
Action function
+
pending status
```

Do not replace all local state with `useActionState`.

---

## Common Mistakes

### Mistake 1 — Forgetting previousState

A `useActionState` Action receives it first.

### Mistake 2 — Forgetting field names

FormData needs named controls.

### Mistake 3 — Trusting frontend validation

Validate again where the mutation is protected.

### Mistake 4 — Duplicating pending state

Use `isPending` when it already represents the workflow.

### Mistake 5 — Confusing React Actions with Server Actions

They are related concepts, not the same thing.

---

## Interview Questions

### What is an Action in React 19?

A function used for mutation or async workflows that React can coordinate with form state and pending UI.

### What does a normal form Action receive?

FormData.

### What does useActionState return?

Current Action state, an Action function, and an `isPending` boolean.

### What is the wrapped Action signature?

```js
(previousState, formData)
```

### Do Actions remove the need for validation?

No.

---

## Quick Revision

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
  // return next state
}
```

Main flow:

```text
form
↓
FormData
↓
Action
↓
returned state
↓
UI
```

---

## Next Lesson

➡️ [Lesson 29 — useFormStatus and Optimistic UI](./29-useformstatus-optimistic-ui.md)
