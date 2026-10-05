# Lesson 08 — Conditional Rendering

## 1. What Is Conditional Rendering?

**Conditional rendering** means displaying different UI depending on a condition.

Examples:

- show Login or Logout
- show loading UI while data is loading
- show an error message when a request fails
- show an admin panel only for admins
- show an empty state when a list has no items

Mental model:

```text
Condition / State
       ↓
Choose UI
       ↓
React renders the selected result
```

React does not introduce a special template language for this. You use normal JavaScript.

---

## 2. Using if / else

```jsx
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>Welcome back!</h1>;
  }

  return <h1>Please log in.</h1>;
}
```

This is often the clearest choice when entire UI branches differ.

```text
isLoggedIn?
   ├── true  → Welcome
   └── false → Login message
```

---

## 3. Early Returns ⭐⭐⭐⭐⭐

Early returns are very useful for loading, error, permission, and empty states.

```jsx
function UserProfile({ user, isLoading, error }) {
  if (isLoading) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>Something went wrong.</p>;
  }

  if (!user) {
    return <p>User not found.</p>;
  }

  return (
    <section>
      <h1>{user.name}</h1>
      <p>{user.role}</p>
    </section>
  );
}
```

Flow:

```text
Loading? ──yes──→ Loading UI
   │ no
   ↓
Error? ───yes──→ Error UI
   │ no
   ↓
No user? ─yes──→ Empty UI
   │ no
   ↓
Success UI
```

This avoids deeply nested JSX.

---

## 4. Ternary Operator ⭐⭐⭐⭐⭐

Use a ternary when choosing between two values/UI branches:

```jsx
function AuthButton({ isLoggedIn }) {
  return (
    <button>
      {isLoggedIn ? "Logout" : "Login"}
    </button>
  );
}
```

Syntax:

```js
condition ? valueIfTrue : valueIfFalse
```

It also works with components:

```jsx
{isLoggedIn ? <Dashboard /> : <Login />}
```

Use ternaries when they remain readable.

---

## 5. Avoid Deeply Nested Ternaries

Hard to understand:

```jsx
{
  isLoading
    ? <Loading />
    : error
      ? <ErrorMessage />
      : user
        ? <Profile user={user} />
        : <NotFound />
}
```

Often clearer:

```jsx
if (isLoading) {
  return <Loading />;
}

if (error) {
  return <ErrorMessage />;
}

if (!user) {
  return <NotFound />;
}

return <Profile user={user} />;
```

Readability matters more than fitting everything into one expression.

---

## 6. Logical AND (&&) ⭐⭐⭐⭐⭐

Use `&&` when something should render only if a condition is true.

```jsx
function Dashboard({ isAdmin }) {
  return (
    <main>
      <h1>Dashboard</h1>

      {isAdmin && <AdminPanel />}
    </main>
  );
}
```

JavaScript behavior:

```text
true  && <AdminPanel /> → <AdminPanel />
false && <AdminPanel /> → false
```

React does not display `false` as visible child content.

---

## 7. The Famous && Zero Bug ⭐⭐⭐⭐⭐

This can be dangerous:

```jsx
{items.length && <ItemList items={items} />}
```

When:

```js
items.length === 0
```

the expression evaluates to:

```js
0
```

React can render that zero.

You may unexpectedly see:

```text
0
```

in the UI.

Use an explicit boolean condition:

```jsx
{items.length > 0 && <ItemList items={items} />}
```

or:

```jsx
{Boolean(items.length) && <ItemList items={items} />}
```

The first form is usually clearer.

---

## 8. Rendering Nothing with null ⭐⭐⭐⭐⭐

A component can return `null` when it should render nothing.

```jsx
function AdminPanel({ isAdmin }) {
  if (!isAdmin) {
    return null;
  }

  return <section>Admin controls</section>;
}
```

Mental model:

```text
isAdmin?
├── false → render nothing
└── true  → render AdminPanel UI
```

Returning `null` means the component has no rendered output for that render.

---

## 9. Conditionally Assigning JSX to a Variable

You can prepare UI before returning:

```jsx
function Status({ status }) {
  let message;

  if (status === "loading") {
    message = <p>Loading...</p>;
  } else if (status === "success") {
    message = <p>Loaded successfully.</p>;
  } else {
    message = <p>Something went wrong.</p>;
  }

  return <section>{message}</section>;
}
```

This is useful when you need conditional content inside a larger shared layout.

---

## 10. Conditional Components

Instead of changing only text:

```jsx
{isLoggedIn ? "Logout" : "Login"}
```

you can switch entire components:

```jsx
{isLoggedIn ? <UserMenu /> : <LoginButton />}
```

React conditions can choose any valid renderable UI.

---

## 11. Multiple Independent Conditions

Sometimes several pieces may independently appear:

```jsx
function ProductCard({ product, isAdmin, isOnSale }) {
  return (
    <article>
      <h2>{product.name}</h2>

      {isOnSale && <span>Sale</span>}

      {isAdmin && (
        <button>Edit Product</button>
      )}
    </article>
  );
}
```

These are not mutually exclusive. Both may render.

Use separate conditions when UI pieces are independent.

---

## 12. Conditional Rendering with Arrays

Empty state:

```jsx
function UserList({ users }) {
  if (users.length === 0) {
    return <p>No users found.</p>;
  }

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>
          {user.name}
        </li>
      ))}
    </ul>
  );
}
```

A production list often has several states:

```text
Data UI
├── Loading
├── Error
├── Empty
└── Success
```

Do not design only the success state.

---

## 13. Truthy and Falsy Values ⭐⭐⭐⭐⭐

Conditional rendering relies heavily on JavaScript truthiness.

Common falsy values include:

```js
false
0
""
null
undefined
NaN
```

Examples:

```js
Boolean(0)         // false
Boolean("")        // false
Boolean(null)      // false
Boolean([])        // true
Boolean({})        // true
```

A common mistake is assuming an empty array is falsy.

It is not:

```js
Boolean([]) // true
```

Therefore:

```jsx
{users && <UserList users={users} />}
```

will still render `UserList` when:

```js
users = []
```

If you mean "has at least one user":

```jsx
{users.length > 0 && <UserList users={users} />}
```

---

## 14. Be Careful with Valid Falsy Data

Suppose a product price can legitimately be zero:

```jsx
{price ? <p>₹{price}</p> : <p>No price</p>}
```

If:

```js
price = 0
```

the UI incorrectly displays "No price".

Better condition:

```jsx
{price != null ? <p>₹{price}</p> : <p>No price</p>}
```

The condition should reflect the actual business rule, not simply rely on truthiness.

---

## 15. && vs Ternary

Use `&&` when there is only a true branch:

```jsx
{isAdmin && <AdminPanel />}
```

Use a ternary when there are two alternatives:

```jsx
{isLoggedIn ? <Dashboard /> : <Login />}
```

Use `if`/early return when branching becomes complex:

```jsx
if (isLoading) return <Loading />;
if (error) return <ErrorMessage />;

return <Dashboard />;
```

Simple rule:

```text
Only render when true?
        ↓
       &&

Choose A or B?
        ↓
      ternary

Several complex states?
        ↓
 if / early return
```

---

## 16. Conditional Rendering vs Hiding with CSS ⭐⭐⭐⭐⭐

These are different.

Conditional rendering:

```jsx
{isOpen && <Modal />}
```

When `isOpen` is false, that Modal component is not included in that rendered branch.

CSS hiding:

```jsx
<div style={{ display: isOpen ? "block" : "none" }}>
  <ModalContent />
</div>
```

The element/component can remain in the React/DOM structure but is visually hidden.

This distinction can affect:

- component state
- DOM presence
- accessibility
- performance
- lifecycle/effects

Choose based on the intended behavior.

---

## 17. Conditional Rendering and Component State ⭐⭐⭐⭐⭐

Suppose:

```jsx
{showCounter && <Counter />}
```

When `showCounter` becomes false, the `Counter` is removed from that position in the tree.

When it later returns, React may create a fresh component instance, so its local state can reset.

Simplified:

```text
showCounter = true
      ↓
<Counter /> exists
      ↓
state = 5

showCounter = false
      ↓
Counter removed

showCounter = true
      ↓
new Counter instance
      ↓
initial state
```

This connects conditional rendering to **component identity**.

We will study state preservation/resetting deeply in Lesson 18.

---

## 18. Conditional Rendering and Hooks ⭐⭐⭐⭐⭐

Do **not** conditionally call Hooks like this:

```jsx
function Profile({ isLoggedIn }) {
  if (isLoggedIn) {
    const [user, setUser] = useState(null); // ❌
  }

  // ...
}
```

Hooks have rules about where they can be called.

Instead, call the Hook at the component's top level:

```jsx
function Profile({ isLoggedIn }) {
  const [user, setUser] = useState(null);

  if (!isLoggedIn) {
    return <Login />;
  }

  return <UserProfile user={user} />;
}
```

Conditional **rendering** is normal.

Conditional **Hook calls** are generally not.

The Rules of Hooks have a dedicated lesson later.

---

## 19. Authentication Example

```jsx
function App({ user, isLoading }) {
  if (isLoading) {
    return <LoadingScreen />;
  }

  if (!user) {
    return <LoginPage />;
  }

  return <Dashboard user={user} />;
}
```

This is clearer than deeply nesting conditions.

Flow:

```text
Loading?
├── yes → LoadingScreen
└── no
     ↓
   User?
   ├── no  → LoginPage
   └── yes → Dashboard
```

---

## 20. Role-Based UI Example

```jsx
function Dashboard({ user }) {
  return (
    <main>
      <h1>Welcome {user.name}</h1>

      {user.role === "admin" && (
        <AdminTools />
      )}
    </main>
  );
}
```

### Security warning

Conditional rendering is **not authorization security**.

Hiding:

```jsx
{user.role === "admin" && <DeleteButton />}
```

only controls the UI.

The backend must still enforce authorization for protected operations.

```text
Frontend condition
      ↓
User experience

Backend authorization
      ↓
Actual security
```

Never rely on hidden buttons to protect sensitive actions.

---

## 21. Loading, Error, Empty, Success Pattern ⭐⭐⭐⭐⭐

A robust data-driven component often considers four states:

```jsx
function ApplicationList({
  applications,
  isLoading,
  error,
}) {
  if (isLoading) {
    return <p>Loading applications...</p>;
  }

  if (error) {
    return <p>Failed to load applications.</p>;
  }

  if (applications.length === 0) {
    return <p>No applications yet.</p>;
  }

  return (
    <ul>
      {applications.map((application) => (
        <li key={application.id}>
          {application.company}
        </li>
      ))}
    </ul>
  );
}
```

Mental model:

```text
Request State
├── Loading
├── Error
├── Empty Success
└── Success with Data
```

This is an important production habit.

---

## 22. Avoid Impossible UI States

Suppose you manage multiple booleans:

```js
isLoading
isSuccess
isError
```

You might accidentally create:

```text
isLoading = true
isSuccess = true
isError   = true
```

Sometimes a single status value is easier to reason about:

```js
status = "loading"
status = "success"
status = "error"
```

Then:

```jsx
if (status === "loading") {
  return <Loading />;
}

if (status === "error") {
  return <ErrorMessage />;
}

return <Content />;
```

This is a general state-design principle that becomes more important later.

---

## 23. Extract Complex Conditional UI

If conditional markup becomes large:

```jsx
function Dashboard({ user }) {
  return (
    <main>
      {user.role === "admin" ? (
        // 80 lines of admin JSX
      ) : (
        // 70 lines of user JSX
      )}
    </main>
  );
}
```

Consider extracting meaningful components:

```jsx
function Dashboard({ user }) {
  return user.role === "admin"
    ? <AdminDashboard user={user} />
    : <UserDashboard user={user} />;
}
```

The condition remains obvious while each branch owns its own UI.

---

## 24. Common Mistakes

### Mistake 1 — Deeply nested ternaries

Prefer readable branching or extracted components.

### Mistake 2 — Using a number directly with &&

```jsx
{items.length && <List />}
```

can render `0`.

### Mistake 3 — Assuming empty arrays/objects are falsy

They are truthy in JavaScript.

### Mistake 4 — Using truthiness when zero or empty string is valid data

Write a condition matching the actual requirement.

### Mistake 5 — Calling Hooks conditionally

Conditional UI is fine; conditional Hook calls violate the Rules of Hooks.

### Mistake 6 — Treating UI hiding as security

Backend authorization must protect sensitive operations.

### Mistake 7 — Ignoring loading/error/empty states

Production UIs need more than the successful-data branch.

### Mistake 8 — Forgetting that unmounting can reset local state

Conditional presence affects component identity and state lifetime.

---

## 25. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is conditional rendering in React?

**Answer:** Conditional rendering means using JavaScript conditions to decide which React elements or components should be included in the rendered UI.

### Q2. What techniques can be used?

**Answer:** Common techniques include `if/else`, early returns, ternary expressions, logical `&&`, variables containing JSX, and returning `null`.

### Q3. When should you use &&?

**Answer:** When UI should appear only when a condition is true and there is no alternative false branch.

### Q4. Why can && accidentally render 0?

**Answer:** JavaScript returns the first falsy operand from an `&&` expression. If that value is numeric zero, React can render it as text.

### Q5. What does returning null from a component do?

**Answer:** It tells React that the component has no visible rendered output for that render.

### Q6. Is an empty array falsy?

**Answer:** No. Arrays and objects are truthy even when empty.

### Q7. Can Hooks be called conditionally?

**Answer:** Hooks generally must be called at the top level of a component or custom Hook, not inside conditions, loops, or nested functions.

### Q8. What happens to state when a condition removes a component?

**Answer:** If the component is removed from the React tree, its local state is destroyed. Rendering it again can create a fresh instance with initial state.

### Q9. Is hiding an admin button sufficient authorization?

**Answer:** No. Conditional rendering controls UI only. Protected backend operations must enforce authorization independently.

### Q10. Ternary vs early return?

**Answer:** Ternaries are useful for concise two-way choices, while early returns are often clearer for multiple states such as loading, error, empty, and success.

---

## 26. Interview Scenario

### Requirement

Display:

- loading UI while fetching
- error UI if the request fails
- empty UI if there are no jobs
- job list otherwise

A clean approach:

```jsx
function JobList({ jobs, isLoading, error }) {
  if (isLoading) {
    return <Loading />;
  }

  if (error) {
    return <ErrorMessage />;
  }

  if (jobs.length === 0) {
    return <EmptyState />;
  }

  return (
    <div>
      {jobs.map((job) => (
        <JobCard key={job.id} job={job} />
      ))}
    </div>
  );
}
```

Why this is good:

```text
Each state
   ↓
clear branch
   ↓
simple success JSX
   ↓
easy to read/debug
```

---

## 27. Decision Guide

```text
Need to stop rendering early?
        ↓
       if + return

Need A or B?
        ↓
      ternary

Need something only when true?
        ↓
        &&

Need intentionally no UI?
        ↓
       null

Condition is becoming huge?
        ↓
extract a component
```

---

## 28. Quick Revision

```text
Conditional Rendering
├── if / else
├── early return
├── ternary
├── &&
├── JSX variables
└── null
```

Remember:

```jsx
isAdmin && <AdminPanel />

isLoggedIn ? <Dashboard /> : <Login />

if (isLoading) return <Loading />;

if (!visible) return null;
```

Important JavaScript:

```text
0          → falsy, but React can render it
""         → falsy
null       → falsy
undefined  → falsy
[]         → truthy
{}         → truthy
```

---

## 29. Key Takeaways

- React uses normal JavaScript for conditional rendering.
- Use early returns for clear multi-state UI.
- Use ternaries for concise A-or-B choices.
- Use `&&` for render-only-when-true cases.
- Be careful when the left side of `&&` can be `0`.
- Empty arrays and objects are truthy.
- Conditions should represent actual business rules rather than blindly relying on truthiness.
- Return `null` when a component intentionally renders nothing.
- Conditional removal of a component can reset its local state.
- Conditional rendering does not mean Hooks may be called conditionally.
- UI conditions are not a replacement for backend authorization.
- Production data-driven UI should account for **loading, error, empty, and success** states.
- Extract large conditional branches into meaningful components.

---

## Next Lesson

➡️ [Lesson 09 — Events and Event Handling](./09-events.md)
