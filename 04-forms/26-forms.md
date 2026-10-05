# Lesson 26 — Forms in React

## 1. Why Forms Matter

Forms are one of the most common ways users send information to an application.

Examples:

- login and signup
- search
- profile editing
- checkout
- job applications
- filters
- comments and messages

A React form usually combines:

```text
input values
   +
validation
   +
submission
   +
pending/error/success UI
```

Before React 19 form Actions, React applications commonly handled all of these pieces manually. React 19 adds better primitives, but understanding normal form behavior is still essential.

---

# 2. Native HTML Form First

A normal HTML form:

```html
<form>
  <label>
    Email
    <input
      name="email"
      type="email"
    />
  </label>

  <button type="submit">
    Submit
  </button>
</form>
```

Important concepts:

- `form` groups form controls.
- `name` identifies submitted fields.
- `type` gives the browser input semantics.
- `label` improves accessibility.
- a submit button submits the form.

React builds on these browser fundamentals rather than replacing them.

---

# 3. Controlled Forms ⭐⭐⭐⭐⭐

A controlled input stores its current value in React state.

```jsx
import { useState } from "react";

function LoginForm() {
  const [email, setEmail] =
    useState("");

  return (
    <form>
      <label>
        Email
        <input
          type="email"
          value={email}
          onChange={(event) =>
            setEmail(
              event.target.value
            )
          }
        />
      </label>
    </form>
  );
}
```

Flow:

```text
React state
    ↓
value prop
    ↓
input
    ↓
onChange
    ↓
setState
    ↓
render
    ↓
new value prop
```

React state is the source of truth.

---

# 4. Multiple Controlled Fields

```jsx
function SignupForm() {
  const [form, setForm] =
    useState({
      name: "",
      email: "",
      password: "",
    });

  function handleChange(event) {
    const {
      name,
      value,
    } = event.target;

    setForm((form) => ({
      ...form,
      [name]: value,
    }));
  }

  return (
    <form>
      <input
        name="name"
        value={form.name}
        onChange={handleChange}
      />

      <input
        name="email"
        type="email"
        value={form.email}
        onChange={handleChange}
      />

      <input
        name="password"
        type="password"
        value={form.password}
        onChange={handleChange}
      />
    </form>
  );
}
```

Dynamic property:

```js
[name]: value
```

lets one handler update several fields.

---

# 5. Different Input Types

## Text

```jsx
<input
  value={name}
  onChange={(event) =>
    setName(event.target.value)
  }
/>
```

## Textarea

```jsx
<textarea
  value={bio}
  onChange={(event) =>
    setBio(event.target.value)
  }
/>
```

## Select

```jsx
<select
  value={role}
  onChange={(event) =>
    setRole(event.target.value)
  }
>
  <option value="frontend">
    Frontend
  </option>

  <option value="backend">
    Backend
  </option>
</select>
```

## Checkbox

Checkboxes normally use:

```text
checked
```

not `value`.

```jsx
<input
  type="checkbox"
  checked={accepted}
  onChange={(event) =>
    setAccepted(
      event.target.checked
    )
  }
/>
```

---

# 6. Form Submission ⭐⭐⭐⭐⭐

Traditional React pattern:

```jsx
function LoginForm() {
  const [email, setEmail] =
    useState("");

  function handleSubmit(event) {
    event.preventDefault();

    console.log(email);
  }

  return (
    <form
      onSubmit={handleSubmit}
    >
      <input
        value={email}
        onChange={(event) =>
          setEmail(
            event.target.value
          )
        }
      />

      <button type="submit">
        Login
      </button>
    </form>
  );
}
```

`preventDefault()` prevents the browser's normal page navigation/reload submission behavior when JavaScript is handling submission.

---

# 7. Use onSubmit, Not Only Button onClick ⭐⭐⭐⭐⭐

Avoid:

```jsx
<form>
  <input />

  <button
    onClick={handleSubmit}
  >
    Submit
  </button>
</form>
```

Prefer:

```jsx
<form
  onSubmit={handleSubmit}
>
  ...
</form>
```

Why?

Forms can be submitted through:

- button click
- Enter key
- accessibility tools
- browser form behavior

Handling the form's submit event respects these semantics.

---

# 8. Button Types Matter

Inside a form:

```jsx
<button type="submit">
  Save
</button>
```

submits it.

A non-submit action should explicitly use:

```jsx
<button
  type="button"
  onClick={handleCancel}
>
  Cancel
</button>
```

Otherwise a button may accidentally submit the form.

---

# 9. Uncontrolled Forms ⭐⭐⭐⭐⭐

Not every field must be stored in React state.

```jsx
function LoginForm() {
  function handleSubmit(
    event
  ) {
    event.preventDefault();

    const formData =
      new FormData(
        event.currentTarget
      );

    console.log(
      formData.get("email")
    );
  }

  return (
    <form
      onSubmit={handleSubmit}
    >
      <input
        name="email"
        type="email"
      />

      <button type="submit">
        Login
      </button>
    </form>
  );
}
```

The browser owns the current input value.

At submission:

```text
form
 ↓
FormData
 ↓
submitted values
```

---

# 10. FormData ⭐⭐⭐⭐⭐

`FormData` is a browser API for reading form values.

```jsx
const formData =
  new FormData(
    event.currentTarget
  );

const email =
  formData.get("email");

const password =
  formData.get("password");
```

Important:

> A field generally needs a `name` to participate in FormData submission.

---

# 11. Convert FormData to an Object

For simple fields:

```jsx
const values =
  Object.fromEntries(
    formData.entries()
  );

console.log(values);
```

But remember that repeated field names can contain multiple values, so `getAll()` may be required.

Example:

```js
formData.getAll("skills");
```

---

# 12. Controlled vs Uncontrolled Form

Controlled:

```text
input
 ↓
onChange
 ↓
React state
 ↓
value prop
 ↓
input
```

Uncontrolled:

```text
browser DOM owns value
        ↓
submit
        ↓
FormData / ref
```

Use controlled fields when you need live React behavior such as:

- instant validation
- character count
- dependent fields
- live filtering
- conditional UI

Use uncontrolled/native form values when React does not need every keystroke.

---

# 13. Do Not Control Everything Automatically

This is valid:

```jsx
<input
  name="email"
  type="email"
/>
```

You do not always need:

```jsx
const [email, setEmail] =
  useState("");
```

Ask:

> Does React need the value before submission?

If no, browser-managed form state may be simpler.

---

# 14. File Inputs

File inputs are special.

```jsx
<input
  name="resume"
  type="file"
/>
```

The browser controls selected files.

Read them through:

```js
formData.get("resume")
```

or through the DOM/File API.

Do not try to control a file input like a normal text field.

---

# 15. Native Validation

Browsers already support useful validation attributes.

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

Useful native attributes include:

```text
required
minLength
maxLength
min
max
pattern
type="email"
type="url"
```

Use browser semantics when appropriate instead of rebuilding everything manually.

---

# 16. Resetting a Form

Native form reset:

```jsx
<form>
  ...
  <button type="reset">
    Reset
  </button>
</form>
```

For controlled state, reset React state:

```jsx
setForm({
  name: "",
  email: "",
});
```

For uncontrolled forms:

```js
event.currentTarget.reset();
```

may be appropriate after successful processing.

---

# 17. Submission State

A production form often has states such as:

```text
idle
 ↓
submitting
 ↓
success

or

submitting
 ↓
error
```

Traditional approach:

```jsx
const [pending, setPending] =
  useState(false);

const [error, setError] =
  useState(null);
```

Then submission manually updates them.

React 19 provides better form-oriented primitives, which you will learn in Lessons 28 and 29.

---

# 18. Async Submission Pattern

```jsx
async function handleSubmit(
  event
) {
  event.preventDefault();

  setPending(true);
  setError(null);

  try {
    await saveProfile(form);
  } catch (error) {
    setError(
      "Unable to save profile"
    );
  } finally {
    setPending(false);
  }
}
```

This is useful to understand because it shows what higher-level React form primitives help simplify.

---

# 19. Prevent Duplicate Submission

During submission:

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

This improves UX and reduces accidental repeated submissions.

But the server/API must still be designed safely; disabling a button is not a security or consistency guarantee.

---

# 20. Error Messages and Accessibility

Associate errors clearly with fields.

```jsx
<label htmlFor="email">
  Email
</label>

<input
  id="email"
  name="email"
  aria-describedby={
    error
      ? "email-error"
      : undefined
  }
/>

{error && (
  <p id="email-error">
    {error}
  </p>
)}
```

Good forms need both functionality and accessibility.

---

# 21. Derived Form State

Avoid storing values that can be calculated.

Bad:

```jsx
const [password, setPassword] =
  useState("");

const [isValid, setIsValid] =
  useState(false);
```

if validity is simply:

```jsx
const isValid =
  password.length >= 8;
```

Derived values should normally be calculated during render.

This follows Lesson 22: you often do not need an Effect to synchronize derived state.

---

# 22. Form State Shape

For related fields:

```jsx
const [form, setForm] =
  useState({
    title: "",
    company: "",
    status: "applied",
  });
```

can be convenient.

But do not create one huge object merely because all values happen to be in one form.

Choose state structure based on update and ownership needs.

---

# 23. CareerLoop Example

```jsx
function ApplicationForm() {
  const [form, setForm] =
    useState({
      company: "",
      role: "",
      status: "applied",
    });

  function handleChange(event) {
    const {
      name,
      value,
    } = event.target;

    setForm((form) => ({
      ...form,
      [name]: value,
    }));
  }

  function handleSubmit(event) {
    event.preventDefault();

    console.log(form);
  }

  return (
    <form
      onSubmit={handleSubmit}
    >
      <input
        name="company"
        value={form.company}
        onChange={handleChange}
        required
      />

      <input
        name="role"
        value={form.role}
        onChange={handleChange}
        required
      />

      <select
        name="status"
        value={form.status}
        onChange={handleChange}
      >
        <option value="applied">
          Applied
        </option>

        <option value="interview">
          Interview
        </option>

        <option value="offer">
          Offer
        </option>
      </select>

      <button type="submit">
        Save application
      </button>
    </form>
  );
}
```

---

# 24. Common Mistakes ⭐⭐⭐⭐⭐

- handling submission only through button `onClick`
- forgetting `event.preventDefault()` in traditional JS-controlled submission
- forgetting `name` when using FormData
- using `value` for checkbox state instead of `checked`
- accidentally switching controlled fields between `undefined` and strings
- controlling every input without a reason
- storing derived validation state
- using Effects to synchronize fields unnecessarily
- forgetting `type="button"` for non-submit buttons
- allowing duplicate submission without considering pending state
- manually manipulating input DOM when state/props should control it
- ignoring native HTML semantics and accessibility

---

# 25. Interview Questions ⭐⭐⭐⭐⭐

## What is a controlled input?

An input whose current value is controlled by React state through a value/checked prop and an update handler.

## What is an uncontrolled input?

An input whose current value is primarily managed by the DOM/browser and is read when needed through FormData or a ref.

## Controlled vs uncontrolled?

Controlled fields are useful when React needs live access to the value. Uncontrolled fields can be simpler when values are mainly needed at submission.

## Why use onSubmit instead of only button onClick?

Because form submission includes keyboard and native browser/accessibility behavior, not only button clicks.

## What does preventDefault do?

It prevents the browser's default form submission/navigation behavior when JavaScript is handling submission.

## What is FormData?

A browser API representing form fields and values, commonly used to read or send form submissions.

## Why is name important?

Named successful form controls participate in form submission and can be retrieved through FormData.

## Should validation always be stored in state?

No. If validation can be derived from current values, calculate it rather than duplicating state.

---

# 26. Mental Model

```text
FORM
 │
 ├── controlled fields
 │      ↓
 │   React state
 │
 ├── uncontrolled fields
 │      ↓
 │   browser DOM
 │
 ├── validation
 │      ↓
 │   native + application rules
 │
 └── submit
        ↓
     process data
        ↓
 pending / error / success
```

---

# 27. Key Takeaways

- React forms build on native HTML form behavior.
- Use semantic `form`, `label`, input types, and submit buttons.
- Controlled inputs use React state as the source of truth.
- Uncontrolled inputs let the browser own current values.
- Use `onSubmit` for traditional React submission handling.
- FormData is especially useful for browser-managed fields.
- Fields need meaningful `name` attributes for form submission.
- Checkboxes normally use `checked`.
- File inputs are browser-controlled.
- Native validation can solve many basic validation requirements.
- Do not store derived form state unnecessarily.
- Pending/error/success are important production form states.
- React 19 provides Actions and form-specific Hooks that improve async form workflows.

---

## Next Lesson

➡️ [Lesson 27 — Form Validation and Submission Patterns](./27-form-validation.md)
