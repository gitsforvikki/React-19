# Lesson 04 — JSX Fundamentals

## 1. What is JSX?

**JSX (JavaScript Syntax Extension)** is syntax commonly used in React to describe UI inside JavaScript.

```jsx
const element = <h1>Hello React</h1>;
```

It looks similar to HTML, but it is **not HTML**. Tooling transforms JSX into JavaScript React can work with.

```text
JSX → JavaScript transformation → React UI description → DOM/UI
```

## 2. Why JSX?

JSX keeps the markup and JavaScript logic controlling that markup close together.

```jsx
function Greeting({ name, isLoggedIn }) {
  return (
    <h1>{isLoggedIn ? "Welcome, " + name : "Please sign in"}</h1>
  );
}
```

JSX is not a string. This:

```jsx
const heading = <h1>Hello</h1>;
```

is different from:

```js
const heading = "<h1>Hello</h1>";
```

## 3. JSX Gets Transformed ⭐⭐⭐⭐⭐

Browsers do not directly execute JSX as ordinary JavaScript source. Development/build tooling transforms it into JavaScript used by React's JSX runtime.

Historically, this was commonly explained as:

```jsx
<h1>Hello</h1>
```

becoming approximately:

```js
React.createElement("h1", null, "Hello");
```

Modern tooling commonly uses the newer JSX runtime, but the important mental model remains:

```text
JSX
 ↓
JavaScript
 ↓
React element description
 ↓
Rendering
```

## 4. Returning JSX

A component commonly returns JSX:

```jsx
function Profile() {
  return (
    <section>
      <h1>Vikash</h1>
      <p>React Developer</p>
    </section>
  );
}
```

## 5. One Enclosing JSX Tree ⭐⭐⭐⭐⭐

This is invalid:

```jsx
return (
  <h1>Hello</h1>
  <p>Welcome</p>
);
```

Wrap the siblings:

```jsx
return (
  <div>
    <h1>Hello</h1>
    <p>Welcome</p>
  </div>
);
```

Or use a Fragment when you do not want an unnecessary DOM wrapper:

```jsx
return (
  <>
    <h1>Hello</h1>
    <p>Welcome</p>
  </>
);
```

## 6. Fragments

Fragments group children without adding another wrapper DOM element.

Short syntax:

```jsx
<>
  <Header />
  <Main />
</>
```

Explicit syntax:

```jsx
import { Fragment } from "react";

<Fragment>
  <Header />
  <Main />
</Fragment>
```

The explicit form is useful when a Fragment needs a `key`.

## 7. JavaScript Expressions in JSX ⭐⭐⭐⭐⭐

Curly braces enter JavaScript expression mode:

```jsx
const name = "Vikash";

return <h1>Hello {name}</h1>;
```

Examples:

```jsx
<p>{2 + 3}</p>
<p>{user.name}</p>
<p>{price * quantity}</p>
<p>{isLoggedIn ? "Dashboard" : "Login"}</p>
<p>{formatDate(createdAt)}</p>
```

## 8. Expressions vs Statements ⭐⭐⭐⭐⭐

An expression produces a value:

```js
user.name
2 + 2
isLoggedIn ? "Yes" : "No"
items.map(...)
```

A normal `if` statement cannot simply be placed directly inside JSX braces.

Wrong:

```jsx
{
  if (isLoggedIn) {
    "Welcome"
  }
}
```

Instead:

```jsx
function Greeting({ isLoggedIn }) {
  let message;

  if (isLoggedIn) {
    message = "Welcome";
  } else {
    message = "Please login";
  }

  return <h1>{message}</h1>;
}
```

Or:

```jsx
<h1>{isLoggedIn ? "Welcome" : "Please login"}</h1>
```

## 9. JSX Attributes

Literal string:

```jsx
<button title="Save">Save</button>
```

JavaScript expression:

```jsx
<button disabled={isSaving}>Save</button>
```

Dynamic image:

```jsx
<img src={imageUrl} alt="User avatar" />
```

## 10. className and camelCase

Use:

```jsx
<div className="card">Product</div>
```

Many DOM properties/events use camelCase:

```jsx
<button onClick={handleClick}>Save</button>

<input
  onChange={handleChange}
  autoFocus
  tabIndex={0}
/>
```

Inline style properties also use camelCase:

```jsx
<div
  style={{
    backgroundColor: "black",
    fontSize: "18px",
  }}
>
  Hello
</div>
```

## 11. Self-Closing Tags

Elements with no children must be closed:

```jsx
<img src="/logo.png" alt="Logo" />
<input type="text" />
<br />
<UserCard />
```

## 12. Component Names Start with Capital Letters ⭐⭐⭐⭐⭐

Custom component:

```jsx
function UserCard() {
  return <article>User</article>;
}

return <UserCard />;
```

Mental model:

```text
<div />       → platform/DOM element
<button />    → platform/DOM element

<UserCard />  → React component reference
<Navbar />    → React component reference
```

Lowercase JSX names are interpreted as built-in/platform elements, while uppercase identifiers represent components.

## 13. Rendering Variables

```jsx
function Product() {
  const name = "Keyboard";
  const price = 2500;

  return (
    <article>
      <h2>{name}</h2>
      <p>₹{price}</p>
    </article>
  );
}
```

Calculations can be prepared before JSX:

```jsx
const total = price * quantity;

return <p>Total: ₹{total}</p>;
```

This often keeps markup easier to read.

## 14. Plain Objects Cannot Be Rendered Directly ⭐⭐⭐⭐⭐

Wrong:

```jsx
const user = { name: "Vikash", age: 25 };

return <div>{user}</div>;
```

Render a property:

```jsx
return <div>{user.name}</div>;
```

For debugging:

```jsx
<pre>{JSON.stringify(user, null, 2)}</pre>
```

## 15. Values Rendered as Children

Strings and numbers can render as text:

```jsx
<p>{"Hello"}</p>
<p>{42}</p>
```

Values such as `null`, `undefined`, and booleans commonly represent no visible child output.

That enables patterns such as:

```jsx
{isAdmin && <AdminPanel />}
```

## 16. Be Careful with && and Numbers ⭐⭐⭐⭐⭐

A common bug:

```jsx
{items.length && <ItemList items={items} />}
```

When `items.length` is `0`, JavaScript returns `0`, and React can render that zero.

Prefer:

```jsx
{items.length > 0 && <ItemList items={items} />}
```

## 17. Rendering Arrays

React can render arrays of valid React nodes.

```jsx
const names = ["Aman", "Vikash", "Rahul"];

function Users() {
  return (
    <ul>
      {names.map((name) => (
        <li key={name}>{name}</li>
      ))}
    </ul>
  );
}
```

`map()` creates an array of React elements.

Keys are extremely important and have a dedicated lesson.

## 18. Event Handlers in JSX

Correct:

```jsx
function Button() {
  function handleClick() {
    console.log("Clicked");
  }

  return <button onClick={handleClick}>Click</button>;
}
```

Here we **pass** the function.

Usually wrong:

```jsx
onClick={handleClick()}
```

That invokes it during rendering.

With an argument:

```jsx
<button onClick={() => deleteUser(user.id)}>
  Delete
</button>
```

## 19. JSX Comments

Inside JSX:

```jsx
return (
  <div>
    {/* JSX comment */}
    <h1>Hello</h1>
  </div>
);
```

Outside JSX, use ordinary JavaScript comments.

## 20. Inline Styles

```jsx
<div style={{ color: "white", backgroundColor: "black" }}>
  Hello
</div>
```

Why two braces?

```text
outer { } → enter JavaScript expression mode
inner { } → JavaScript object
```

## 21. JSX and Security ⭐⭐⭐⭐⭐

React escapes ordinary string values rendered through JSX.

```jsx
const userInput = "<script>alert('hack')</script>";

return <p>{userInput}</p>;
```

The string is treated as text rather than automatically interpreted as raw HTML.

React also provides the explicit raw-HTML escape hatch named `dangerouslySetInnerHTML`.

Never insert untrusted HTML through it without appropriate sanitization and a clear security reason, because unsafe raw HTML can introduce XSS vulnerabilities.

## 22. Keep JSX Readable

JSX can contain JavaScript expressions, but complicated transformations can often be prepared before the markup.

Instead of deeply nesting logic inside JSX:

```jsx
const activeUserNames = users
  .filter((user) => user.active)
  .sort((a, b) => a.name.localeCompare(b.name))
  .map((user) => user.name)
  .join(", ");

return <p>{activeUserNames}</p>;
```

Readable components are more important than trying to place every calculation directly in JSX.

## 23. Common JSX Mistakes

1. Returning siblings without an enclosing element or Fragment.
2. Using `class` instead of `className`.
3. Forgetting to close tags.
4. Putting statements directly inside JSX braces.
5. Rendering plain objects as children.
6. Calling handlers during rendering instead of passing them.
7. Naming custom components with lowercase names.
8. Injecting unsafe raw HTML.
9. Using a numeric value directly on the left side of `&&` without considering `0`.

## 24. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is JSX?

**Answer:** JSX is a JavaScript syntax extension commonly used with React to describe UI. Tooling transforms it into JavaScript used by React's JSX runtime.

### Q2. Is JSX HTML?

**Answer:** No. JSX resembles HTML but follows JavaScript and React rules.

### Q3. Is JSX mandatory?

**Answer:** No. React can be used without JSX, but JSX is the standard readable approach for describing component trees.

### Q4. Why do sibling JSX elements need a wrapper?

**Answer:** A JSX expression must result in one enclosing value/tree. Siblings can be grouped with an element or Fragment.

### Q5. Why className instead of class?

**Answer:** React DOM JSX uses the `className` property to specify the DOM element's CSS class.

### Q6. Can an if statement be written directly inside JSX braces?

**Answer:** JSX braces accept expressions. A normal `if` statement is generally handled before the JSX or replaced with an expression such as a ternary.

### Q7. Why uppercase component names?

**Answer:** Lowercase JSX tags are treated as platform elements, while uppercase identifiers are treated as component references.

### Q8. Why can items.length && <List /> show 0?

**Answer:** JavaScript's `&&` returns the first falsy operand. If the length is zero, the expression returns `0`, which React can render.

### Q9. Why is raw HTML insertion dangerous?

**Answer:** Inserting unsanitized raw HTML can create XSS vulnerabilities. Ordinary JSX text values are escaped, but raw HTML bypasses that normal protection.

### Q10. What happens to JSX before execution?

**Answer:** Development/build tooling transforms JSX into JavaScript that works with React's JSX runtime.

## 25. Mental Model

```text
JavaScript Component
        ↓
       JSX
        ↓
Tooling transforms JSX
        ↓
React UI descriptions
        ↓
React rendering
        ↓
DOM updates
        ↓
Browser UI
```

## 26. Quick Revision

```text
JSX
├── JavaScript syntax extension
├── describes UI
├── transformed into JavaScript
├── { } for JavaScript expressions
├── CapitalCase for components
├── className for CSS classes
├── camelCase DOM properties/events
├── tags must be closed
└── Fragment groups without extra DOM wrapper
```

## 27. Key Takeaways

- JSX is a JavaScript syntax extension, not HTML.
- Tooling transforms JSX into JavaScript used by React.
- Use braces for JavaScript **expressions**.
- Ordinary statements such as `if` are usually handled before returned JSX.
- Custom component names should begin with uppercase letters.
- Use `className` and camelCase DOM properties.
- Close JSX tags.
- Fragments group siblings without an unnecessary DOM element.
- Plain objects cannot normally be rendered directly as children.
- Be careful when using numbers with conditional `&&`.
- React escapes normal text values, while raw HTML requires special security care.

---

## Next Lesson

➡️ [Lesson 05 — Components and Component Composition ⭐⭐⭐⭐⭐](./05-components-composition.md)
