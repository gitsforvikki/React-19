# Lesson 27 — Form Validation and Submission Patterns

Form validation checks whether form data follows your application's rules.

Examples:

- email must be valid
- password must be long enough
- required fields cannot be empty
- confirm password must match
- end date cannot be before start date

---

## 1. Validation Should Exist at Multiple Layers

A good application may validate at:

```text
browser
↓
client
↓
server/API
↓
database/business rules
```

Important:

> Client validation improves UX. Server validation protects correctness and security.

Never trust client-side validation alone.

---

## 2. Native Validation First

```jsx
<input
  name="email"
  type="email"
  required
/>

<input
  name="password"
  type="password"
  minLength={8}
  required
/>
```

Use browser validation when possible.

---

## 3. Client Validation Function

Keep validation logic simple and pure.

```js
function validate(values) {
  const errors = {};

  if (!values.email.trim()) {
    errors.email =
      "Email is required";
  }

  if (
    values.password.length < 8
  ) {
    errors.password =
      "Minimum 8 characters";
  }

  return errors;
}
```

Mental model:

```text
form values
↓
validate()
↓
errors
```

No Effect is needed.

---

## 4. Field-Level vs Form-Level Validation

Field-level:

```text
email
→ valid?

password
→ long enough?
```

Cross-field:

```text
password
+
confirmPassword
→ match?
```

Example:

```js
if (
  values.password !==
  values.confirmPassword
) {
  errors.confirmPassword =
    "Passwords do not match";
}
```

---

## 5. When to Show Errors

Common choices:

- on change
- on blur
- on submit

Do not show aggressive errors before the user has interacted.

A simple pattern is:

```text
submit
↓
show errors
↓
user fixes fields
```

For many forms, submit-time validation is enough.

---

## 6. Derived Validation

Do not store validation in separate state if it can be calculated directly.

Avoid:

```jsx
useEffect(() => {
  setEmailError(
    validateEmail(email)
  );
}, [email]);
```

Prefer:

```jsx
const emailError =
  validateEmail(email);
```

This avoids duplicate state.

---

## 7. Submit Validation Pattern

```jsx
function handleSubmit(event) {
  event.preventDefault();

  const nextErrors =
    validate(form);

  setErrors(nextErrors);

  if (
    Object.keys(
      nextErrors
    ).length > 0
  ) {
    return;
  }

  save(form);
}
```

Flow:

```text
submit
↓
validate
↓
errors?
├─ yes → show errors
└─ no  → submit
```

---

## 8. Async Submission

```jsx
async function handleSubmit(
  event
) {
  event.preventDefault();

  const nextErrors =
    validate(form);

  if (
    Object.keys(
      nextErrors
    ).length
  ) {
    setErrors(nextErrors);
    return;
  }

  setPending(true);
  setServerError(null);

  try {
    await createAccount(form);
  } catch {
    setServerError(
      "Something went wrong"
    );
  } finally {
    setPending(false);
  }
}
```

This is the traditional pattern that React 19 Actions simplify.

---

## 9. Prevent Duplicate Submission

```jsx
<button
  type="submit"
  disabled={pending}
>
  {pending
    ? "Saving..."
    : "Save"}
</button>
```

This improves UX, but backend logic must still handle duplicate requests safely.

---

## 10. Server Errors vs Field Errors

Field error:

```text
Email is required
```

Server/general error:

```text
Unable to create account
```

Keep them conceptually separate.

Example state:

```js
{
  errors: {
    email: "Invalid email"
  },
  message:
    "Please correct the form"
}
```

---

## 11. Accessibility

Connect errors to fields when possible.

```jsx
<input
  id="email"
  name="email"
  aria-invalid={
    Boolean(errors.email)
  }
  aria-describedby={
    errors.email
      ? "email-error"
      : undefined
  }
/>

{errors.email && (
  <p id="email-error">
    {errors.email}
  </p>
)}
```

---

## Common Mistakes

### Mistake 1 — Trusting client validation

Always validate again at the trusted server/API boundary.

### Mistake 2 — Storing derived validation in state unnecessarily

Calculate it directly.

### Mistake 3 — Submitting even when validation failed

Return early.

### Mistake 4 — Mixing field errors and system errors

Keep their meaning clear.

### Mistake 5 — Forgetting pending state

Users should know when a form is being submitted.

---

## Interview Questions

### Why is client validation not enough?

Because users can bypass it. The server must validate trusted mutations.

### Should validation always be stored in state?

No. Derived validation can often be calculated directly.

### What is cross-field validation?

Validation that depends on more than one field, such as password confirmation.

### Why disable a submit button while pending?

To improve UX and reduce accidental repeated submissions.

### Where should final validation happen?

At the trusted server/API boundary.

---

## Quick Revision

```text
Native validation
→ simple browser rules

Client validation
→ UX

Server validation
→ correctness/security

Derived validation
→ calculate directly

Async submit
→ pending + error + success
```

---

## Next Lesson

➡️ [Lesson 28 — React 19 Form Actions and useActionState](./28-form-actions-useactionstate.md)
