# Lesson 27 — Form Validation and Submission Patterns

## 1. What Is Form Validation?

Form validation checks whether submitted data satisfies the rules required by your application.

Examples:

```text
email must be valid
password must be at least 8 characters
company name is required
salary must be positive
end date cannot be before start date
```

Validation has two goals:

1. help the user correct mistakes
2. protect application/business rules

---

# 2. Validation Layers ⭐⭐⭐⭐⭐

A robust application may validate at multiple layers:

```text
Browser/native validation
        ↓
Client application validation
        ↓
Server/API validation
        ↓
Database/business constraints
```

These layers complement each other.

> **Client validation improves UX. Server validation protects correctness and security.**

Never trust client validation alone.

---

# 3. Native HTML Validation

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

The browser already understands:

- required
- email syntax
- minimum/maximum lengths
- number ranges
- patterns

Use native capabilities where they fit.

---

# 4. Client Validation

Example:

```jsx
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
      "Password must be at least 8 characters";
  }

  return errors;
}
```

Keep validation logic pure when possible:

```text
values
  ↓
validate()
  ↓
errors
```

No Effect is required.

---

# 5. Field-Level vs Form-Level Validation

Field-level:

```text
email
→ valid email?

password
→ minimum length?
```

Form-level/cross-field:

```text
password
+
confirmPassword
→ match?

startDate
+
endDate
→ valid range?
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

# 6. When Should Validation Run?

Common strategies:

```text
on change
on blur
on submit
or combination
```

### On change

Immediate feedback, but can be noisy.

### On blur

Useful after the user finishes a field.

### On submit

Simple and avoids premature errors.

A common UX:

```text
submit once
   ↓
show errors
   ↓
update relevant errors
as user fixes fields
```

Choose intentionally.

---

# 7. Do Not Show Errors Too Early

Imagine an empty email input immediately showing:

```text
Invalid email
```

before the user has typed anything.

Technically correct, poor UX.

Often track interaction such as:

```jsx
const [touched, setTouched] =
  useState({});
```

Then display errors when a field has been interacted with or after submission.

---

# 8. Derived Validation ⭐⭐⭐⭐⭐

If validation can be calculated:

```jsx
const emailError =
  !email.includes("@")
    ? "Invalid email"
    : null;
```

do not synchronize another state variable through an Effect:

```jsx
useEffect(() => {
  setEmailError(
    validateEmail(email)
  );
}, [email]);
```

Usually simpler:

```jsx
const emailError =
  validateEmail(email);
```

This avoids duplicate sources of truth.

---

# 9. Submission Validation Pattern

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
 ┌──┴──┐
yes   no
 │      │
 ↓      ↓
show   submit
errors data
```

---

# 10. Async Submission ⭐⭐⭐⭐⭐

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
      "Unable to create account"
    );
  } finally {
    setPending(false);
  }
}
```

Production forms need to model the entire async lifecycle.

---

# 11. Client Error vs Server Error ⭐⭐⭐⭐⭐

Client error:

```text
"Email is required"
```

Server error:

```text
"An account with this email already exists"
```

The client cannot reliably know every business rule.

Therefore:

```text
client validation
≠
replacement for server validation
```

---

# 12. Field Errors Returned by Server

A useful server response shape:

```js
{
  success: false,
  fieldErrors: {
    email:
      "Email already registered"
  },
  message:
    "Please correct the form"
}
```

Then the UI can display the error beside the relevant field.

React 19's `useActionState` is especially useful for returning this kind of form state.

---

# 13. Disable Submit or Allow Submit?

Two valid approaches exist.

### Disable invalid submit

```jsx
<button
  disabled={!isValid}
>
  Submit
</button>
```

### Allow submit and show errors

Useful when disabling the button would make it unclear why submission is unavailable.

There is no universal rule.

Accessibility and clarity matter more than simply disabling everything.

---

# 14. Pending State ⭐⭐⭐⭐⭐

When a request is in progress:

```text
user submits
    ↓
pending = true
    ↓
disable/relabel submit
    ↓
request completes
    ↓
pending = false
```

Example:

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

React 19's `useFormStatus` can provide form submission pending state to descendant components.

---

# 15. Preventing Duplicate Requests

Disabling a pending button helps UX, but backend correctness must not depend on it.

Users or networks can still produce repeated requests.

Server-side systems may need:

- idempotency
- unique constraints
- transaction rules
- duplicate detection

Frontend pending state is not a security boundary.

---

# 16. Success State

After successful submission you may:

- show success feedback
- reset form
- close a modal
- update parent state
- update displayed data
- navigate elsewhere

The correct behavior depends on product requirements.

Do not automatically reset a form before you know submission succeeded.

---

# 17. Error State Should Be Actionable

Weak:

```text
Something went wrong.
```

Better when possible:

```text
That email is already registered.
Try another email.
```

Errors should help users recover.

Do not expose sensitive internal server details.

---

# 18. Form State Machine Mental Model ⭐⭐⭐⭐⭐

Instead of thinking only in booleans:

```text
loading
error
success
```

think in states:

```text
IDLE
  ↓ submit
VALIDATING
  ├── invalid → INVALID
  └── valid
         ↓
      SUBMITTING
       ├── failure → ERROR
       └── success → SUCCESS
```

This helps prevent contradictory states such as:

```text
success = true
error = true
pending = true
```

at the same time.

---

# 19. Avoid Contradictory Form State

Instead of many booleans:

```jsx
const [loading, setLoading] =
  useState(false);

const [success, setSuccess] =
  useState(false);

const [failed, setFailed] =
  useState(false);
```

you can sometimes model:

```jsx
const [status, setStatus] =
  useState("idle");
```

with:

```text
idle
submitting
success
error
```

Choose a state shape that cannot easily represent impossible combinations.

---

# 20. Error Object Shape

For field errors:

```js
{
  company: "Company is required",
  role: "Role is required"
}
```

This makes rendering straightforward:

```jsx
{errors.company && (
  <p>{errors.company}</p>
)}
```

---

# 21. Accessibility of Validation Errors ⭐⭐⭐⭐⭐

Use labels:

```jsx
<label htmlFor="email">
  Email
</label>
```

Mark invalid fields:

```jsx
<input
  id="email"
  aria-invalid={
    Boolean(errors.email)
  }
/>
```

Associate descriptions:

```jsx
aria-describedby="email-error"
```

Then:

```jsx
<p id="email-error">
  {errors.email}
</p>
```

For important form-level status updates, an appropriate live region may help assistive technology announce changes.

---

# 22. Focus the First Invalid Field

For long forms, focusing the first invalid field can improve usability.

This may involve a DOM ref:

```text
submit
  ↓
validation fails
  ↓
identify first invalid field
  ↓
focus()
```

Use imperative focus for accessibility/UX, while keeping the validation data itself declarative.

---

# 23. Validation Libraries

For large applications, libraries can reduce repetitive form code.

Examples of responsibilities a library may help with:

- registration
- validation
- touched/dirty state
- error mapping
- performance
- schema integration

But learn the underlying concepts first:

```text
values
validation
errors
submission
pending
server response
```

Libraries do not replace these fundamentals.

---

# 24. Schema Validation Concept

A schema can define rules in one place.

Conceptually:

```js
const schema = {
  email: "valid email",
  password: "minimum 8",
};
```

In real applications, schema libraries can provide runtime parsing and typed validation.

Important principle:

> Validate again at the trusted server boundary even if the client uses the same schema.

---

# 25. Security Rule ⭐⭐⭐⭐⭐

Never trust values merely because they came from your React form.

A user can bypass the UI and send requests directly.

Therefore:

```text
React validation
→ user experience

server validation
→ trust boundary
```

Authorization must also happen on the server.

For example:

```text
hidden admin field
or disabled admin button
```

does not create authorization.

---

# 26. Race and Submission Problems

Async forms can encounter:

- repeated clicks
- slow responses
- component unmount
- stale response ordering
- navigating away
- server validation changes

Design submission as an asynchronous workflow rather than assuming every request succeeds instantly.

---

# 27. CareerLoop Example

```jsx
function validateApplication(
  values
) {
  const errors = {};

  if (!values.company.trim()) {
    errors.company =
      "Company is required";
  }

  if (!values.role.trim()) {
    errors.role =
      "Role is required";
  }

  return errors;
}
```

Submission:

```jsx
async function handleSubmit(
  event
) {
  event.preventDefault();

  const errors =
    validateApplication(form);

  setErrors(errors);

  if (
    Object.keys(errors).length
  ) {
    return;
  }

  setStatus("submitting");

  try {
    await createApplication(
      form
    );

    setStatus("success");
  } catch {
    setStatus("error");
  }
}
```

This is the traditional form workflow you should understand before React 19 Actions.

---

# 28. Common Mistakes ⭐⭐⭐⭐⭐

- trusting only client validation
- storing every derived validation result in state
- validating through unnecessary Effects
- showing errors before the user has interacted
- clearing form before successful submission
- ignoring async pending state
- allowing contradictory status booleans
- disabling submit without explaining invalid fields
- exposing raw server errors
- forgetting accessible labels/error associations
- assuming a disabled button prevents duplicate backend operations
- putting authorization logic only in the UI

---

# 29. Interview Questions ⭐⭐⭐⭐⭐

## Why validate on both client and server?

Client validation improves UX; server validation protects the trusted application boundary because clients can be bypassed.

## What is derived validation?

Validation calculated directly from current form values rather than duplicated into synchronized state.

## When should validation run?

It depends on UX: change, blur, submit, or a combination.

## What is field-level validation?

Rules concerning one field.

## What is form-level validation?

Rules involving the whole form or relationships between fields.

## Why model form submission as states?

It prevents contradictory booleans and makes transitions such as idle → submitting → success/error explicit.

## Should pending UI guarantee no duplicate server request?

No. It helps UX, while server-side correctness must independently handle duplicates when necessary.

## Should authorization be enforced by hiding controls?

No. UI restrictions are not security boundaries; authorization must be enforced by trusted server code.

---

# 30. Quick Revision

```text
INPUT
  ↓
validation
  ↓
errors?
 ┌──┴──┐
yes   no
 │      │
 ↓      ↓
fix   submit
       ↓
    pending
     ┌─┴─┐
  error success
```

Remember:

```text
client validation
→ UX

server validation
→ correctness/security
```

---

# 31. Key Takeaways

- Validation should help users and protect application rules.
- Use native browser validation where useful.
- Keep validation functions pure when possible.
- Do not use Effects for values that can simply be derived.
- Validation can run on change, blur, submit, or a combination.
- Cross-field rules require form-level validation.
- Server validation is mandatory for trusted decisions.
- Pending, success, and error are part of the form workflow.
- Model impossible states out of your state shape where possible.
- Field errors should be actionable and accessible.
- Frontend authorization is never sufficient.
- React 19 Actions can simplify async submission state while preserving these same fundamentals.

---

## Next Lesson

➡️ [Lesson 28 — React 19 Form Actions and useActionState ⭐⭐⭐⭐⭐](./28-form-actions-useactionstate.md)
