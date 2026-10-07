# Lesson 26 — Forms in React

React forms are built on normal HTML forms.

The main things to understand are:

- controlled inputs
- uncontrolled inputs
- form submission
- FormData
- basic validation
- pending and error UI

---

## 1. Controlled Form

A controlled input stores its value in React state.

```jsx
function LoginForm() {
  const [email, setEmail] =
    useState("");

  return (
    <input
      value={email}
      onChange={(e) =>
        setEmail(e.target.value)
      }
    />
  );
}
```

Flow:

```text
state
↓
value prop
↓
input
↓
onChange
↓
setState
```

Use controlled inputs when React needs the latest value immediately.

Examples:

- live validation
- search filtering
- conditional fields
- character counters

---

## 2. Multiple Fields

```jsx
const [form, setForm] =
  useState({
    name: "",
    email: "",
  });

function handleChange(event) {
  const { name, value } =
    event.target;

  setForm((current) => ({
    ...current,
    [name]: value,
  }));
}
```

Usage:

```jsx
<input
  name="name"
  value={form.name}
  onChange={handleChange}
/>

<input
  name="email"
  value={form.email}
  onChange={handleChange}
/>
```

`[name]: value` lets one handler update different fields.

---

## 3. Checkbox

Checkboxes use `checked`, not the normal text-input `value`.

```jsx
const [accepted, setAccepted] =
  useState(false);

<input
  type="checkbox"
  checked={accepted}
  onChange={(e) =>
    setAccepted(
      e.target.checked
    )
  }
/>
```

Remember:

```text
text input → value
checkbox   → checked
```

---

## 4. Form Submission

Use the form's `onSubmit`.

```jsx
function handleSubmit(event) {
  event.preventDefault();

  console.log(form);
}

<form onSubmit={handleSubmit}>
  ...
  <button type="submit">
    Save
  </button>
</form>
```

Why `onSubmit` instead of only button `onClick`?

Because forms can also submit through:

- Enter key
- accessibility tools
- native browser behavior

---

## 5. Button Types Matter

Submit button:

```jsx
<button type="submit">
  Save
</button>
```

Normal button inside a form:

```jsx
<button
  type="button"
  onClick={handleCancel}
>
  Cancel
</button>
```

Without `type="button"`, a button inside a form may submit it.

---

## 6. Uncontrolled Form + FormData

Not every input needs React state.

```jsx
function LoginForm() {
  function handleSubmit(event) {
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
    <form onSubmit={handleSubmit}>
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

Important:

> Form fields need a `name` to be read correctly through FormData.

---

## 7. Controlled vs Uncontrolled

Use controlled fields when React needs the current value during typing.

Use uncontrolled fields when you mainly need values at submit time.

```text
Controlled
→ React state owns value

Uncontrolled
→ browser owns value
→ read with FormData/ref
```

Neither is always better.

---

## 8. Native Validation

Use browser validation when it fits.

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

Useful attributes:

- `required`
- `minLength`
- `maxLength`
- `min`
- `max`
- `pattern`
- `type="email"`

---

## 9. File Inputs

File inputs are browser-controlled.

```jsx
<input
  name="resume"
  type="file"
/>
```

Read with:

```js
formData.get("resume")
```

Do not treat a file input like a normal controlled text field.

---

## 10. Pending and Error State

A real async form usually needs:

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

Traditional pattern:

```jsx
const [pending, setPending] =
  useState(false);

const [error, setError] =
  useState(null);
```

React 19 form APIs simplify this further in Lessons 28 and 29.

---

## Common Mistakes

### Mistake 1 — Handling submit only on the button

Use `onSubmit` on the form.

### Mistake 2 — Forgetting `preventDefault()`

Needed in traditional JavaScript-controlled form submission.

### Mistake 3 — Forgetting `name`

FormData depends on field names.

### Mistake 4 — Using `value` for checkbox state

Use `checked`.

### Mistake 5 — Controlling every field without a reason

Use the simplest model that fits the UI.

---

## Interview Questions

### What is a controlled input?

An input whose current value is managed by React state.

### What is an uncontrolled input?

An input whose current value is managed by the DOM/browser.

### Why use onSubmit?

It respects normal form behavior, including Enter-key submission.

### What is FormData?

A browser API used to read submitted form values.

### Why is name important?

It identifies form fields during submission.

---

## Quick Revision

```text
Need live value in React?
→ controlled

Need value mainly on submit?
→ uncontrolled + FormData

Submit logic?
→ form onSubmit

Checkbox?
→ checked

File input?
→ browser controlled
```

---

## Next Lesson

➡️ [Lesson 27 — Form Validation and Submission Patterns](./27-form-validation.md)
