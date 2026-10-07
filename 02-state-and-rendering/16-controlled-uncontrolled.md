# Lesson 16 — Controlled vs Uncontrolled Components

The main question is:

> Who owns the current input value?

```text
Controlled
→ React state owns the value

Uncontrolled
→ DOM owns the value
```

---

## 1. Controlled Components

A controlled input gets its value from React state.

```jsx
function SearchBox() {
  const [query, setQuery] =
    useState("");

  return (
    <input
      value={query}
      onChange={(event) =>
        setQuery(event.target.value)
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
 ↓
new render
```

React state is the source of truth.

---

## 2. Checkbox Uses checked

For checkboxes, use `checked` instead of `value`:

```jsx
const [accepted, setAccepted] =
  useState(false);

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

Remember:

```text
text input → value
checkbox   → checked
```

---

## 3. Uncontrolled Components

An uncontrolled input stores its current value in the DOM.

You can read it with a ref:

```jsx
function Form() {
  const inputRef =
    useRef(null);

  function handleSubmit(
    event
  ) {
    event.preventDefault();

    console.log(
      inputRef.current.value
    );
  }

  return (
    <form
      onSubmit={handleSubmit}
    >
      <input
        ref={inputRef}
      />

      <button>
        Submit
      </button>
    </form>
  );
}
```

React does not store every typed character in state.

---

## 4. value vs defaultValue

Controlled:

```jsx
<input
  value={name}
  onChange={...}
/>
```

Uncontrolled:

```jsx
<input
  defaultValue="Vikash"
/>
```

Mental model:

```text
value
→ current value controlled by React

defaultValue
→ initial value, then DOM controls it
```

For checkboxes:

```text
checked
→ controlled

defaultChecked
→ uncontrolled initial state
```

---

## 5. When to Use Controlled Inputs

Controlled inputs are useful when React needs the latest value immediately.

Examples:

- live validation
- search filtering
- character counter
- disabling buttons
- conditional fields

Example:

```jsx
const [email, setEmail] =
  useState("");

const isValid =
  email.includes("@");

<input
  value={email}
  onChange={(e) =>
    setEmail(e.target.value)
  }
/>

<button
  disabled={!isValid}
>
  Submit
</button>
```

---

## 6. When to Use Uncontrolled Inputs

Use uncontrolled inputs when:

- you only need the value on submit
- the form is simple
- you are integrating with DOM-based code
- file input is involved

Example with `FormData`:

```jsx
function SignupForm() {
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
        defaultValue=""
      />

      <button>
        Submit
      </button>
    </form>
  );
}
```

---

## 7. File Inputs Are Uncontrolled

File inputs are controlled by the browser.

```jsx
const fileRef =
  useRef(null);

<input
  type="file"
  ref={fileRef}
/>
```

Read:

```js
fileRef.current.files[0]
```

---

## 8. Do Not Switch Between Controlled and Uncontrolled

Bad:

```jsx
const [name, setName] =
  useState(undefined);

<input
  value={name}
  onChange={(e) =>
    setName(e.target.value)
  }
/>
```

Start controlled text inputs with:

```js
useState("")
```

and checkboxes with:

```js
useState(false)
```

This keeps ownership consistent.

---

## 9. Controlled Custom Components

The same concept also applies to reusable components.

Controlled:

```jsx
<Modal
  open={isOpen}
  onOpenChange={setIsOpen}
/>
```

Parent owns the state.

Uncontrolled:

```jsx
<Modal
  defaultOpen={true}
/>
```

The component manages its own state internally.

---

## Controlled vs Uncontrolled

| Controlled | Uncontrolled |
|---|---|
| React owns value | DOM owns value |
| Uses `value` / `checked` | Uses `defaultValue` / `defaultChecked` |
| Easy live validation | Simple one-time reading |
| State updates on edits | No React state required for every edit |
| Best when UI depends on value | Best when value is only needed later |

---

## Common Mistakes

### Mistake 1 — value without onChange

```jsx
<input value="React" />
```

This behaves like a read-only controlled input.

### Mistake 2 — Switching ownership

Do not move from `undefined` to a controlled string later.

### Mistake 3 — Assuming controlled is always better

Both patterns are valid.

Choose based on the UI requirement.

---

## Interview Questions

### What is a controlled component?

A component whose important value is controlled by React state or parent props.

### What is an uncontrolled input?

An input whose current value is managed by the DOM and read when needed.

### What is the difference between value and defaultValue?

`value` controls the current value; `defaultValue` only sets the initial uncontrolled value.

### Why are file inputs usually uncontrolled?

Because the browser manages the selected file list.

---

## Quick Revision

```text
Controlled
React state → value → input

Uncontrolled
DOM stores value → read with ref/FormData
```

Remember:

1. `value` / `checked` → controlled
2. `defaultValue` / `defaultChecked` → uncontrolled
3. Use controlled inputs when the UI depends on the current value
4. Use uncontrolled inputs when you only need the value occasionally

---

## Next Lesson

➡️ [Lesson 17 — Lifting State Up and State Colocation](./17-lifting-colocating-state.md)
