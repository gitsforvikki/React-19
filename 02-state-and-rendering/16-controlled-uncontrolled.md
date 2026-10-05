# Lesson 16 — Controlled vs Uncontrolled Components ⭐⭐⭐⭐⭐

## 1. Why This Topic Matters

Forms are everywhere in React applications:

- login
- signup
- search
- profile editing
- job applications
- checkout
- filters

To handle form inputs correctly, you need to understand two important patterns:

```text
Controlled Components
        vs
Uncontrolled Components
```

The main difference is:

> **Who owns the current input value?**

```text
Controlled
React state owns the value

Uncontrolled
The DOM owns the value
```

This is a very common React interview topic.

---

## 2. What Is a Controlled Component? ⭐⭐⭐⭐⭐

A controlled form element gets its current value from React state.

Example:

```jsx
function SearchBox() {
  const [query, setQuery] = useState("");

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

Here:

```text
React state
query = ""
    ↓
value={query}
    ↓
<input>
    ↓
user types
    ↓
onChange
    ↓
setQuery(...)
    ↓
new render
    ↓
new value passed to input
```

React controls the input value.

---

## 3. Controlled Input Data Flow ⭐⭐⭐⭐⭐

The full flow is:

```text
STATE
  │
  ↓
value prop
  │
  ↓
INPUT
  │
  ↓
user types
  │
  ↓
onChange event
  │
  ↓
setState
  │
  ↓
RENDER
  │
  └────────→ updated value
```

This is React's one-way data flow applied to forms.

---

## 4. Controlled Input Example

```jsx
function ProfileForm() {
  const [name, setName] = useState("");

  return (
    <>
      <input
        value={name}
        onChange={(event) =>
          setName(event.target.value)
        }
      />

      <p>Hello {name}</p>
    </>
  );
}
```

If the user types:

```text
Vikash
```

the flow is:

```text
input event
    ↓
event.target.value = "Vikash"
    ↓
setName("Vikash")
    ↓
render
    ↓
name = "Vikash"
    ↓
input value + paragraph update
```

---

## 5. Why Is It Called "Controlled"?

Because React controls the value through:

```jsx
value={name}
```

The browser cannot independently keep a different final value.

React state is the source of truth.

```text
React state
    ↓
source of truth
    ↓
input value
```

---

## 6. value Without onChange ⭐⭐⭐⭐⭐

Consider:

```jsx
<input value="React" />
```

The input receives a fixed controlled value.

Typing cannot normally change the displayed value because React keeps providing:

```text
"React"
```

A controlled editable input normally needs:

```jsx
<input
  value={name}
  onChange={(event) =>
    setName(event.target.value)
  }
/>
```

Or it should intentionally be read-only.

---

## 7. Controlled Textarea

```jsx
function BioForm() {
  const [bio, setBio] = useState("");

  return (
    <textarea
      value={bio}
      onChange={(event) =>
        setBio(event.target.value)
      }
    />
  );
}
```

The same controlled pattern applies.

---

## 8. Controlled Select

```jsx
function RoleSelector() {
  const [role, setRole] = useState(
    "frontend"
  );

  return (
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

      <option value="fullstack">
        Full Stack
      </option>
    </select>
  );
}
```

Again:

```text
state → value → select
select → onChange → state
```

---

## 9. Controlled Checkbox ⭐⭐⭐⭐⭐

Checkboxes use `checked`, not `value`, to control whether they are selected.

```jsx
function Terms() {
  const [accepted, setAccepted] =
    useState(false);

  return (
    <input
      type="checkbox"
      checked={accepted}
      onChange={(event) =>
        setAccepted(event.target.checked)
      }
    />
  );
}
```

Important:

```text
Text input
value

Checkbox
checked
```

---

## 10. Controlled Radio Buttons

```jsx
function Experience() {
  const [level, setLevel] =
    useState("junior");

  return (
    <>
      <label>
        <input
          type="radio"
          value="junior"
          checked={level === "junior"}
          onChange={(event) =>
            setLevel(event.target.value)
          }
        />
        Junior
      </label>

      <label>
        <input
          type="radio"
          value="mid"
          checked={level === "mid"}
          onChange={(event) =>
            setLevel(event.target.value)
          }
        />
        Mid
      </label>
    </>
  );
}
```

---

# Uncontrolled Components

## 11. What Is an Uncontrolled Component? ⭐⭐⭐⭐⭐

An uncontrolled input stores its current value in the DOM rather than React state.

Example:

```jsx
function SearchBox() {
  const inputRef = useRef(null);

  function handleSubmit(event) {
    event.preventDefault();

    console.log(
      inputRef.current.value
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} />

      <button type="submit">
        Search
      </button>
    </form>
  );
}
```

React does not store every typed character in state.

The browser input owns its current value.

When needed, code reads the value from the DOM.

---

## 12. Uncontrolled Data Flow ⭐⭐⭐⭐⭐

```text
User types
    ↓
DOM input stores value
    ↓
React does not update state
for every character
    ↓
ref.current.value
    ↓
read value when needed
```

Compare:

```text
CONTROLLED

State
 ↓
Input
 ↓
onChange
 ↓
State


UNCONTROLLED

DOM Input
 ↓
stores its own value
 ↓
Ref reads it when needed
```

---

## 13. defaultValue ⭐⭐⭐⭐⭐

For an uncontrolled input with an initial value:

```jsx
<input
  defaultValue="Vikash"
/>
```

`defaultValue` provides the initial value.

After that, the DOM manages the current value.

Compare:

```jsx
<input value={name} />
```

vs:

```jsx
<input defaultValue="Vikash" />
```

Mental model:

```text
value
  ↓
controlled current value

defaultValue
  ↓
uncontrolled initial value
```

---

## 14. defaultChecked

For an uncontrolled checkbox:

```jsx
<input
  type="checkbox"
  defaultChecked={true}
/>
```

Compare:

```text
checked
  ↓
controlled

defaultChecked
  ↓
uncontrolled initial state
```

---

## 15. value vs defaultValue ⭐⭐⭐⭐⭐

### Controlled

```jsx
<input
  value={name}
  onChange={(event) =>
    setName(event.target.value)
  }
/>
```

React owns the current value.

### Uncontrolled

```jsx
<input
  defaultValue="Vikash"
  ref={inputRef}
/>
```

The DOM owns the current value after initialization.

This distinction is important.

---

## 16. Controlled vs Uncontrolled Comparison ⭐⭐⭐⭐⭐

| Feature | Controlled | Uncontrolled |
|---|---|---|
| Current value stored by | React state | DOM |
| Usually reads with | state variable | ref / form data |
| Uses | `value` / `checked` | `defaultValue` / `defaultChecked` |
| React updated on every edit | Usually yes | Not required |
| Easy live validation | Yes | Less direct |
| Easy conditional UI | Yes | Less direct |
| Simple one-time form reading | More code | Often simple |
| Source of truth | React | DOM |

---

## 17. When Controlled Components Are Useful ⭐⭐⭐⭐⭐

Controlled inputs are especially useful when the UI must react immediately to the current value.

Examples:

### Live validation

```jsx
const isValid =
  email.includes("@");
```

### Character counter

```jsx
<p>
  {bio.length}/200
</p>
```

### Search filtering

```jsx
const filteredUsers =
  users.filter((user) =>
    user.name.includes(query)
  );
```

### Disable submit button

```jsx
<button
  disabled={!email || !password}
>
  Login
</button>
```

### Conditional UI

```jsx
{role === "company" && (
  <CompanyFields />
)}
```

React already knows the latest form value.

---

## 18. When Uncontrolled Components Are Useful

Uncontrolled inputs can be useful when:

- you only need values on submission
- the form is simple
- you are integrating with non-React DOM code
- a form library manages DOM registration internally
- the input is naturally DOM-controlled, such as file selection

Example:

```jsx
function SimpleForm() {
  const nameRef = useRef(null);

  function handleSubmit(event) {
    event.preventDefault();

    console.log(
      nameRef.current.value
    );
  }

  return (
    <form onSubmit={handleSubmit}>
      <input ref={nameRef} />

      <button>
        Submit
      </button>
    </form>
  );
}
```

---

## 19. File Inputs Are Special ⭐⭐⭐⭐⭐

An HTML file input cannot be controlled in the normal text-input way.

Example:

```jsx
function FileUpload() {
  const fileRef = useRef(null);

  function handleSubmit(event) {
    event.preventDefault();

    const file =
      fileRef.current.files[0];

    console.log(file);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="file"
        ref={fileRef}
      />

      <button>
        Upload
      </button>
    </form>
  );
}
```

The selected file list is controlled by the browser.

This is a classic interview example of an uncontrolled input.

---

## 20. FormData Can Read Uncontrolled Form Values

You do not always need a separate ref for every field.

```jsx
function SignupForm() {
  function handleSubmit(event) {
    event.preventDefault();

    const formData =
      new FormData(event.currentTarget);

    const name =
      formData.get("name");

    const email =
      formData.get("email");

    console.log({
      name,
      email,
    });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        defaultValue=""
      />

      <input
        name="email"
        type="email"
        defaultValue=""
      />

      <button type="submit">
        Sign Up
      </button>
    </form>
  );
}
```

The browser owns the field values, and you read them when submitting.

This pattern is especially relevant to modern form workflows.

---

## 21. Do Not Switch Between Controlled and Uncontrolled ⭐⭐⭐⭐⭐

This is a common React warning.

Problem:

```jsx
const [name, setName] =
  useState(undefined);

return (
  <input
    value={name}
    onChange={(event) =>
      setName(event.target.value)
    }
  />
);
```

Initially:

```text
name = undefined
```

The input may behave as uncontrolled.

Later:

```text
name = "Vikash"
```

Now it becomes controlled.

React warns because the ownership model changed during the component's lifetime.

---

## 22. Correct Controlled Initialization ⭐⭐⭐⭐⭐

For text inputs:

```jsx
const [name, setName] =
  useState("");
```

For checkbox:

```jsx
const [accepted, setAccepted] =
  useState(false);
```

Keep the input consistently controlled.

If incoming data may be missing:

```jsx
<input
  value={user.name ?? ""}
  onChange={handleChange}
/>
```

This prevents `undefined` or `null` from unexpectedly changing the input ownership model.

---

## 23. Controlled Does Not Mean "Component Has State"

The term controlled can apply more broadly than HTML form fields.

Consider:

```jsx
function Accordion({
  isOpen,
  onToggle,
}) {
  return (
    <button onClick={onToggle}>
      {isOpen ? "Close" : "Open"}
    </button>
  );
}
```

Parent:

```jsx
function Page() {
  const [isOpen, setIsOpen] =
    useState(false);

  return (
    <Accordion
      isOpen={isOpen}
      onToggle={() =>
        setIsOpen((open) => !open)
      }
    />
  );
}
```

The parent controls the Accordion's important state.

This is a **controlled component API**.

---

## 24. Uncontrolled Custom Component

A component can manage its own internal state:

```jsx
function Accordion() {
  const [isOpen, setIsOpen] =
    useState(false);

  return (
    <button
      onClick={() =>
        setIsOpen((open) => !open)
      }
    >
      {isOpen ? "Close" : "Open"}
    </button>
  );
}
```

Here the parent does not control `isOpen`.

The component owns it internally.

---

## 25. Controlled Custom Component Pattern ⭐⭐⭐⭐⭐

Controlled:

```jsx
<Modal
  open={isModalOpen}
  onOpenChange={setIsModalOpen}
/>
```

Mental model:

```text
Parent
  │
  ├── owns state
  │
  ↓
open prop
  │
  ↓
Modal
  │
  ↓
onOpenChange
  │
  ↓
Parent updates state
```

This is a common reusable component design pattern.

You will see similar APIs in UI libraries.

---

## 26. defaultValue-Like APIs for Custom Components

Reusable components sometimes support an uncontrolled mode:

```jsx
<Accordion
  defaultOpen={true}
/>
```

Conceptually:

```text
defaultOpen
     ↓
initial internal state
     ↓
Accordion manages itself
```

And controlled mode:

```jsx
<Accordion
  open={isOpen}
  onOpenChange={setIsOpen}
/>
```

Conceptually:

```text
Parent owns current value
```

This pattern is useful when designing reusable component libraries.

---

## 27. Controlled Components and Single Source of Truth ⭐⭐⭐⭐⭐

Suppose two components need the same search query.

Bad architecture:

```text
SearchInput
query = "react"

SearchResults
query = "react"
```

Now there are two separate states to synchronize.

Better:

```text
Parent
query = "react"
   │
   ├──────────────┐
   ↓              ↓
SearchInput   SearchResults
```

The parent owns the state and passes it down.

This creates a single source of truth.

This connects controlled components with **lifting state up** in Lesson 17.

---

## 28. Controlled Forms and Validation

```jsx
function SignupForm() {
  const [email, setEmail] =
    useState("");

  const isValid =
    email.length > 0 &&
    email.includes("@");

  return (
    <>
      <input
        type="email"
        value={email}
        onChange={(event) =>
          setEmail(event.target.value)
        }
      />

      {!isValid && email && (
        <p>
          Enter a valid email.
        </p>
      )}

      <button disabled={!isValid}>
        Create Account
      </button>
    </>
  );
}
```

Because React owns `email`, it can immediately derive:

- validation state
- button state
- error messages
- conditional UI

---

## 29. Do Not Store Derived Validation Unnecessarily

Avoid:

```jsx
const [email, setEmail] =
  useState("");

const [isValid, setIsValid] =
  useState(false);
```

if `isValid` can always be calculated from `email`.

Prefer:

```jsx
const isValid =
  email.includes("@");
```

This avoids duplicated state.

---

## 30. Multiple Controlled Inputs

```jsx
function LoginForm() {
  const [form, setForm] = useState({
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
        name="email"
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

This combines:

- controlled inputs
- object state
- immutable updates
- dynamic property names

---

## 31. Does Controlled Mean Better? ⭐⭐⭐⭐⭐

Not always.

Do not memorize:

```text
Controlled = good
Uncontrolled = bad
```

Both patterns are valid.

Choose based on requirements.

Use controlled state when React needs the current value for application behavior.

Use uncontrolled state when the browser can own the value and you only need to read it at specific times.

---

## 32. Performance Considerations

A controlled input normally updates state as the user edits it:

```text
type character
     ↓
onChange
     ↓
setState
     ↓
render
```

This is normal React behavior.

Do not automatically convert forms to uncontrolled just because rendering occurs.

If a large form has a real measured performance issue, consider:

- better component boundaries
- state colocation
- specialized form libraries
- uncontrolled registration patterns where appropriate
- profiling before optimizing

---

## 33. State Colocation in Forms

Suppose only `SearchBox` needs its query.

Instead of storing query at the application root:

```text
App
 ↓
many components re-render
 ↓
SearchBox
```

it may be better to keep it close:

```text
SearchBox
   ↓
local query state
```

But if other components need the query, lifting it may be correct.

The right owner is:

> The closest common component that needs to coordinate that state.

Lesson 17 covers this deeply.

---

## 34. Refs Do Not Cause Re-renders ⭐⭐⭐⭐⭐

In an uncontrolled input:

```jsx
const inputRef = useRef(null);
```

Reading:

```js
inputRef.current.value
```

does not itself cause React to render.

This is different from:

```jsx
setName(...)
```

which schedules state-related rendering.

That distinction is important when comparing state and refs.

---

## 35. Real-World Example: Search Filter

Controlled input:

```jsx
function DeveloperSearch({
  developers,
}) {
  const [query, setQuery] =
    useState("");

  const filteredDevelopers =
    developers.filter((developer) =>
      developer.name
        .toLowerCase()
        .includes(
          query.toLowerCase()
        )
    );

  return (
    <>
      <input
        value={query}
        onChange={(event) =>
          setQuery(event.target.value)
        }
        placeholder="Search developers"
      />

      {filteredDevelopers.map(
        (developer) => (
          <p key={developer.id}>
            {developer.name}
          </p>
        )
      )}
    </>
  );
}
```

Controlled state makes sense because every query change affects the rendered results.

---

## 36. Real-World Example: Simple Submission

Suppose a field is only needed when submitting:

```jsx
function ReferralForm() {
  function handleSubmit(event) {
    event.preventDefault();

    const formData =
      new FormData(
        event.currentTarget
      );

    const referralCode =
      formData.get(
        "referralCode"
      );

    console.log(referralCode);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="referralCode"
        defaultValue=""
      />

      <button type="submit">
        Apply
      </button>
    </form>
  );
}
```

React may not need to store every keystroke.

An uncontrolled approach can be perfectly reasonable.

---

## 37. Controlled vs Uncontrolled Decision Guide ⭐⭐⭐⭐⭐

Ask:

### Does React need the current value continuously?

Examples:

- live validation
- search filtering
- conditional fields
- live preview
- character count

Prefer:

```text
Controlled
```

### Do I mostly need the value on submit?

Prefer considering:

```text
Uncontrolled
```

### Is this a file input?

Usually:

```text
Uncontrolled
```

### Does a parent need to coordinate a reusable component's state?

Prefer:

```text
Controlled component API
```

### Can the component manage itself independently?

An uncontrolled/internal-state API may be simpler.

---

## 38. Common Mistakes ⭐⭐⭐⭐⭐

### Mistake 1 — value without onChange

```jsx
<input value={name} />
```

This can make an intended editable input effectively read-only.

### Mistake 2 — Using value for checkbox state

Wrong:

```jsx
<input
  type="checkbox"
  value={accepted}
/>
```

Use:

```jsx
checked={accepted}
```

### Mistake 3 — Switching controlled → uncontrolled

```jsx
value={undefined}
```

after previously providing a string can cause ownership problems.

### Mistake 4 — Switching uncontrolled → controlled

Starting without a controlled value and later supplying one can produce warnings.

### Mistake 5 — Using defaultValue when you expect React to keep controlling the value

`defaultValue` is for initialization, not continuous control.

### Mistake 6 — Using refs when the UI needs live state

If React needs to render based on every change, state is usually clearer.

### Mistake 7 — Putting every form field in global state

Keep state as local as requirements allow.

### Mistake 8 — Assuming uncontrolled means React cannot submit the form

Values can be read using refs or `FormData`.

---

## 39. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is a controlled component?

**Answer:** A component whose important current value is controlled by React state or by a parent through props.

### Q2. What is an uncontrolled input?

**Answer:** An input whose current value is maintained by the DOM and is read when needed using mechanisms such as refs or FormData.

### Q3. What is the difference between value and defaultValue?

**Answer:** `value` controls the current value, while `defaultValue` provides an initial value for an uncontrolled input.

### Q4. What is the difference between checked and defaultChecked?

**Answer:** `checked` controls the current checkbox/radio state, while `defaultChecked` initializes an uncontrolled one.

### Q5. Why does a controlled input need onChange?

**Answer:** Because React controls the value. The handler updates state when the user edits the input so the next render can provide the new value.

### Q6. Which is the source of truth in a controlled input?

**Answer:** React state.

### Q7. Which is the source of truth in an uncontrolled input?

**Answer:** The DOM element.

### Q8. Can an input switch between controlled and uncontrolled?

**Answer:** It should generally remain consistently controlled or uncontrolled for its lifetime. Switching can produce warnings and confusing behavior.

### Q9. Why is a file input normally uncontrolled?

**Answer:** The browser controls the selected files; application code reads the file list rather than setting it like a normal text value.

### Q10. Are controlled components always better?

**Answer:** No. Controlled components are useful when React needs the current value continuously. Uncontrolled inputs can be simpler when values are mainly needed at submission time.

### Q11. Can controlled/uncontrolled apply to custom components?

**Answer:** Yes. A custom component is controlled when a parent owns its important state and passes the current value plus change callbacks.

### Q12. Do refs trigger re-renders?

**Answer:** No. Changing or reading a ref does not by itself cause a component to render.

---

## 40. Interview Scenario ⭐⭐⭐⭐⭐

What is wrong with this?

```jsx
function Form() {
  const [name, setName] =
    useState();

  return (
    <input
      value={name}
      onChange={(event) =>
        setName(
          event.target.value
        )
      }
    />
  );
}
```

Initially:

```text
name = undefined
```

Later:

```text
name = "Vikash"
```

The input can move from uncontrolled behavior to controlled behavior.

Better:

```jsx
const [name, setName] =
  useState("");
```

Now it is controlled from the beginning.

---

## 41. Another Interview Scenario ⭐⭐⭐⭐⭐

Which pattern should you choose for a live search field?

Requirements:

```text
User types
   ↓
results immediately filter
   ↓
result count updates
   ↓
clear button appears/disappears
```

Controlled state is a natural choice:

```text
query state
    ↓
input
    ↓
filter results
    ↓
result count
    ↓
conditional UI
```

React needs the current value to calculate UI continuously.

---

## 42. Complete Mental Model ⭐⭐⭐⭐⭐

### Controlled

```text
React State
     │
     ↓
 value / checked
     │
     ↓
    Input
     │
     ↓
 User interaction
     │
     ↓
 onChange
     │
     ↓
 setState
     │
     └──────────→ Next Render
```

### Uncontrolled

```text
DOM Input
    │
    ↓
User interaction
    │
    ↓
DOM stores current value
    │
    ↓
ref / FormData
    │
    ↓
read when needed
```

---

## 43. Quick Revision

```text
Controlled
  → React owns current value
  → value / checked
  → onChange + state
```

```text
Uncontrolled
  → DOM owns current value
  → defaultValue / defaultChecked
  → ref or FormData when needed
```

Remember:

```text
value        → current controlled value
defaultValue → initial uncontrolled value

checked        → current controlled checkbox state
defaultChecked → initial uncontrolled checkbox state
```

And:

```text
Do not switch ownership
during the component lifetime.
```

---

## 44. Key Takeaways

- Controlled inputs use React state as the source of truth.
- Uncontrolled inputs let the DOM own the current value.
- Controlled text inputs typically use `value` and `onChange`.
- Controlled checkboxes/radios use `checked`.
- Uncontrolled inputs can use `defaultValue` or `defaultChecked`.
- Refs and `FormData` can read uncontrolled values when needed.
- File inputs are an important uncontrolled-input example.
- Do not casually switch an input between controlled and uncontrolled.
- Initialize controlled text state with a suitable value such as an empty string rather than `undefined`.
- Controlled forms are useful for live validation, filtering, conditional UI, and previews.
- Uncontrolled forms can be useful when values are mainly needed at submission.
- Controlled/uncontrolled is also a reusable component API concept, not only an HTML form concept.
- Parent-controlled component APIs support a single source of truth.
- Refs do not trigger re-renders.
- Neither pattern is universally better; choose according to ownership and UI requirements.

---

## Next Lesson

➡️ [Lesson 17 — Lifting State Up and State Colocation ⭐⭐⭐⭐⭐](./17-lifting-colocating-state.md)
