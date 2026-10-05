# Lesson 06 — Props and One-Way Data Flow ⭐⭐⭐⭐⭐

## 1. What Are Props?

**Props** are inputs passed from a parent component to a child component.

```jsx
function UserCard({ name }) {
  return <h2>{name}</h2>;
}

function App() {
  return <UserCard name="Vikash" />;
}
```

```text
App
 │ name="Vikash"
 ↓
UserCard
 ↓
UI
```

Props make components reusable with different data.

## 2. Props Are Component Inputs

A component can receive one props object:

```jsx
function UserCard(props) {
  return (
    <article>
      <h2>{props.name}</h2>
      <p>{props.role}</p>
    </article>
  );
}
```

Usage:

```jsx
<UserCard name="Vikash" role="React Developer" />
```

Conceptually the component receives values similar to:

```js
{
  name: "Vikash",
  role: "React Developer"
}
```

Render components through JSX so React manages them as part of its component tree.

## 3. Destructuring Props ⭐⭐⭐⭐⭐

Instead of repeatedly writing `props.name`:

```jsx
function UserCard({ name, role }) {
  return (
    <article>
      <h2>{name}</h2>
      <p>{role}</p>
    </article>
  );
}
```

Destructuring clearly shows which inputs the component uses.

## 4. Props Can Contain JavaScript Values

### String

```jsx
<User name="Vikash" />
```

### Number

```jsx
<Product price={2500} />
```

### Boolean

```jsx
<Button disabled={true} />
<Button disabled />
```

### Array

```jsx
<SkillList skills={["React", "JavaScript", "Node.js"]} />
```

### Object

```jsx
<UserCard user={{ name: "Vikash", role: "Developer" }} />
```

### Function

```jsx
<Button onSave={handleSave} />
```

### React content

```jsx
<Layout sidebar={<Sidebar />} />
```

Props are not limited to strings.

## 5. Literal vs Expression Props

Literal string:

```jsx
<User name="Vikash" />
```

JavaScript expression:

```jsx
<User name={user.name} />
<Product price={1000 + 500} />
<Button disabled={!isFormValid} />
```

Braces evaluate JavaScript expressions.

## 6. Default Values

Use JavaScript default parameters:

```jsx
function Button({
  label = "Submit",
  variant = "primary",
}) {
  return (
    <button className={"button " + variant}>
      {label}
    </button>
  );
}
```

A default applies when the received value is `undefined`. Passing `null` is different because it intentionally supplies `null`.

## 7. children Is a Prop ⭐⭐⭐⭐⭐

```jsx
<Card>
  <h2>Hello</h2>
  <p>Welcome</p>
</Card>
```

Nested content is available as `children`:

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}
```

This is central to React composition.

## 8. Props Are Read-Only ⭐⭐⭐⭐⭐

A component should never mutate props.

Wrong:

```jsx
function User({ user }) {
  user.name = "Changed";
  return <h2>{user.name}</h2>;
}
```

Think:

```text
Parent owns data
      ↓
Child receives data
      ↓
Child reads data
```

The receiver treats props as immutable inputs.

## 9. One-Way Data Flow ⭐⭐⭐⭐⭐

React uses unidirectional data flow.

```text
Parent
  │ props
  ↓
Child
```

Example:

```jsx
function App() {
  const username = "Vikash";
  return <Profile name={username} />;
}

function Profile({ name }) {
  return <h2>{name}</h2>;
}
```

The parent owns the value and the child receives it.

This gives React applications predictable ownership and easier debugging.

## 10. Child-to-Parent Communication ⭐⭐⭐⭐⭐

Props themselves flow downward, but the parent can pass a callback.

```jsx
function App() {
  function handleMessage(message) {
    console.log(message);
  }

  return <Child onMessage={handleMessage} />;
}

function Child({ onMessage }) {
  return (
    <button onClick={() => onMessage("Hello Parent")}>
      Send Message
    </button>
  );
}
```

Flow:

```text
Parent
  │ passes callback
  ↓
Child
  │ invokes callback(data)
  ↓
Parent handler
```

This is commonly called child-to-parent communication, but ownership remains explicit: the parent supplied the function.

## 11. Parent Owns State

A common pattern:

```jsx
import { useState } from "react";

function App() {
  const [count, setCount] = useState(0);

  return (
    <Counter
      count={count}
      onIncrement={() => setCount((current) => current + 1)}
    />
  );
}

function Counter({ count, onIncrement }) {
  return (
    <div>
      <p>{count}</p>
      <button onClick={onIncrement}>Increment</button>
    </div>
  );
}
```

```text
App owns count
   ↓
count prop
   ↓
Counter displays it
   ↓
calls onIncrement
   ↓
App updates state
   ↓
new count flows down
```

This pattern appears everywhere in React.

## 12. Props vs State Preview ⭐⭐⭐⭐⭐

| Props | State |
|---|---|
| Passed to component | Owned by component |
| Read-only to receiver | Updated through React state APIs |
| Configure component | Represents changing owned data |
| Caller controls value | Owning component controls value |

We will study this deeply in Lesson 11.

## 13. Object Prop vs Separate Props

Separate:

```jsx
<UserCard
  name={user.name}
  role={user.role}
  avatar={user.avatar}
/>
```

Object:

```jsx
<UserCard user={user} />
```

Neither is always better.

Separate props can make dependencies explicit. An object is convenient when the component genuinely works with that object as a unit.

## 14. Spread Props

```jsx
const user = {
  name: "Vikash",
  role: "Developer",
};

<UserCard {...user} />
```

This is roughly equivalent to passing those properties individually.

Spread syntax can be useful, but excessive spreading can hide which inputs a component actually depends on.

## 15. Forwarding DOM Props

Reusable components sometimes intentionally forward remaining props:

```jsx
function Button({ children, variant = "primary", ...rest }) {
  return (
    <button
      className={"button " + variant}
      {...rest}
    >
      {children}
    </button>
  );
}
```

Usage:

```jsx
<Button
  variant="danger"
  disabled={isDeleting}
  aria-label="Delete account"
  onClick={handleDelete}
>
  Delete
</Button>
```

This is useful for reusable UI primitives, but APIs should remain intentional.

## 16. Prop Drilling ⭐⭐⭐⭐⭐

Suppose:

```text
App
 ↓
Dashboard
 ↓
Sidebar
 ↓
UserMenu
 ↓
Avatar
```

If user data is passed through every intermediate component to reach `Avatar`, this is called **prop drilling**.

Prop drilling is not automatically bad.

Explicit props are often the simplest design for small component trees.

It becomes inconvenient when many intermediate components only forward data they do not otherwise need.

Possible solutions later include:

- composition
- state colocation
- Context
- external state management when justified

## 17. Avoid Unnecessary Prop-to-State Copies ⭐⭐⭐⭐⭐

A common mistake:

```jsx
function Profile({ name }) {
  const [localName, setLocalName] = useState(name);

  return <h2>{localName}</h2>;
}
```

If the component only needs to display `name`, use:

```jsx
function Profile({ name }) {
  return <h2>{name}</h2>;
}
```

Copying creates two values that may need synchronization:

```text
name prop
   ↓
localName state
```

A prop changing later does not automatically reset independently stored state.

There are valid cases for state initialized from a prop, but it should represent intentionally independent state.

## 18. Derived Values

If a value can be calculated from existing inputs, often calculate it directly:

```jsx
function CartSummary({ price, quantity }) {
  const total = price * quantity;

  return <p>Total: ₹{total}</p>;
}
```

Avoid unnecessary synchronized state.

```text
price + quantity
       ↓
     total
```

This reduces sources of truth.

## 19. Object and Function Reference Identity ⭐⭐⭐⭐⭐

This parent creates a new object and function during each render:

```jsx
<UserCard
  user={{ name: "Vikash" }}
  onClick={() => console.log("clicked")}
/>
```

In JavaScript:

```js
{} === {} // false

(() => {}) === (() => {}) // false
```

This matters later for:

- React.memo
- useMemo
- useCallback
- Effect dependencies
- performance

Do not memoize everything just because of this. First understand reference identity.

## 20. Props and Re-rendering ⭐⭐⭐⭐⭐

A common misconception is:

> A child renders only when its props change.

That is not the default rule.

When a parent renders, React normally evaluates the child elements/subtree produced by that parent as part of rendering.

Later, memoization can sometimes let React skip component work.

Also:

```text
component render ≠ DOM update
```

React can render a component and determine that no DOM mutation is required.

## 21. Never Mutate Object Props

Wrong:

```jsx
function Profile({ user }) {
  user.skills.push("React");

  return <p>{user.skills.join(", ")}</p>;
}
```

Instead, let the owner update the data:

```jsx
function App() {
  const [user, setUser] = useState({
    name: "Vikash",
    skills: [],
  });

  function addSkill(skill) {
    setUser((current) => ({
      ...current,
      skills: [...current.skills, skill],
    }));
  }

  return <Profile user={user} onAddSkill={addSkill} />;
}
```

The child requests the change through the API provided by the owner.

## 22. Real-World Example

```jsx
function DeveloperCard({
  developer,
  onConnect,
  onSkip,
}) {
  return (
    <article>
      <img src={developer.avatar} alt={developer.name} />

      <h2>{developer.name}</h2>
      <p>{developer.role}</p>

      <ul>
        {developer.skills.map((skill) => (
          <li key={skill}>{skill}</li>
        ))}
      </ul>

      <button onClick={() => onSkip(developer.id)}>
        Skip
      </button>

      <button onClick={() => onConnect(developer.id)}>
        Connect
      </button>
    </article>
  );
}
```

Parent:

```jsx
<DeveloperCard
  developer={developer}
  onConnect={handleConnect}
  onSkip={handleSkip}
/>
```

The component contract is clear: it receives data and callbacks rather than owning unrelated application behavior.

## 23. Complete One-Way Flow ⭐⭐⭐⭐⭐

```text
                 Parent
                   │
        ┌──────────┴──────────┐
        │                     │
     data prop           callback prop
        │                     │
        └──────────┬──────────┘
                   ↓
                 Child
                   │
             user interaction
                   │
                   ↓
            calls callback
                   │
                   ↓
             Parent handler
                   │
                   ↓
              state update
                   │
                   ↓
          new props flow down
```

This cycle explains a large amount of everyday React code.

## 24. Props as a Component Contract

Think of props as the component's public API.

```jsx
function Button({
  children,
  variant,
  disabled,
  onClick,
}) {
  // ...
}
```

A good prop API is:

- understandable
- intentional
- predictable
- reasonably small
- difficult to misuse

## 25. Common Mistakes

1. Mutating props.
2. Copying every prop into state.
3. Assuming props can only contain strings.
4. Thinking callbacks create two-way data binding.
5. Overusing prop spreading.
6. Treating all prop drilling as a problem.
7. Assuming children render only when props change.
8. Storing values that can simply be derived from existing props/state.

## 26. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What are props?

**Answer:** Props are read-only inputs supplied to a component. They allow components to receive data, behavior, and content from their caller.

### Q2. Can a child modify props?

**Answer:** No. A component should treat props as immutable inputs. Parent-owned data should be updated by its owner.

### Q3. What is one-way data flow?

**Answer:** Data normally flows from parent components to child components through props, making ownership and changes predictable.

### Q4. How does a child communicate with its parent?

**Answer:** The parent passes a callback as a prop and the child invokes it, optionally passing data as arguments.

### Q5. What is prop drilling?

**Answer:** Passing props through intermediate components to reach deeper descendants. It is not inherently bad, but excessive drilling can motivate a different state/composition design.

### Q6. Props vs state?

**Answer:** Props are inputs controlled by the caller, while state is changing data owned by the component that declares it.

### Q7. Why avoid automatically copying props into state?

**Answer:** It creates duplicated sources of truth and can cause synchronization bugs.

### Q8. Can functions be props?

**Answer:** Yes. Callback props are a fundamental React communication pattern.

### Q9. Is children a prop?

**Answer:** Yes. JSX nested between a component's tags is received through the `children` prop.

### Q10. Does a child render only when props change?

**Answer:** No. Parent rendering can cause child rendering too. Memoization may allow React to skip some work when appropriate.

### Q11. Why do object/function props matter for memoization?

**Answer:** New object and function expressions create new references, which matters for shallow equality and dependency comparisons.

### Q12. Why treat props as immutable?

**Answer:** React's predictable data-flow model depends on components treating their render inputs as read-only rather than silently mutating data owned elsewhere.

## 27. Interview Scenario

A `ProductList` owns products and each card can request deletion.

Parent:

```jsx
function ProductList() {
  const [products, setProducts] = useState(initialProducts);

  function handleDelete(id) {
    setProducts((current) =>
      current.filter((product) => product.id !== id)
    );
  }

  return products.map((product) => (
    <ProductCard
      key={product.id}
      product={product}
      onDelete={handleDelete}
    />
  ));
}
```

Child:

```jsx
function ProductCard({ product, onDelete }) {
  return (
    <article>
      <h2>{product.name}</h2>
      <button onClick={() => onDelete(product.id)}>
        Delete
      </button>
    </article>
  );
}
```

Flow:

```text
ProductList owns products
          ↓
product prop
          ↓
ProductCard
          ↓
onDelete(id)
          ↓
ProductList updates state
          ↓
new products flow down
```

This is classic React one-way data flow.

## 28. Quick Revision

```text
Props
├── component inputs
├── passed by caller/parent
├── read-only to receiver
├── can contain JavaScript values
├── children is a prop
└── form a component's public API
```

```text
Parent → data → Child

Parent → callback → Child
                    ↓
              invokes callback
                    ↓
                  Parent
```

Remember:

```text
Props = external inputs
State = owned changing data

Do not mutate props.
Do not duplicate props into state without a reason.
Prop drilling is not automatically bad.
```

## 29. Key Takeaways

- Props are inputs to components.
- Props make components configurable and reusable.
- Props can carry many JavaScript value types.
- Destructuring props is common and readable.
- `children` is a composition prop.
- Treat props as read-only.
- React follows parent-to-child one-way data flow.
- Children can request parent actions through callback props.
- The state owner should normally perform state updates.
- Avoid unnecessary prop-to-state copies and duplicated sources of truth.
- Derive calculable values instead of storing unnecessary state.
- Prop drilling is acceptable until it becomes a genuine design problem.
- Object/function reference identity matters for later optimization concepts.
- A child does not render only when props change.
- Think of props as a component's public contract.

---

## Next Lesson

➡️ [Lesson 07 — Rendering Lists and Understanding Keys ⭐⭐⭐⭐⭐](./07-lists-and-keys.md)
