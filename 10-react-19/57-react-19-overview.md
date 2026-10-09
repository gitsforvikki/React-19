# Lesson 57 — What Changed in React 19 ⭐⭐⭐⭐⭐

## 1. Why React 19 Matters

React 19 is a major React release that improves how applications handle:

- asynchronous mutations
- forms
- optimistic UI
- Suspense-aware resource reading
- refs
- document metadata
- stylesheets and scripts
- resource loading
- hydration diagnostics
- server/client integration

The most important change is not one isolated Hook.

React 19 makes **async UI workflows more deeply integrated with React**.

---

## 2. React 19 Mental Model ⭐⭐⭐⭐⭐

Before React 19, a mutation often required manually coordinating:

```text
submit event
   ↓
pending state
   ↓
API request
   ↓
error handling
   ↓
success state
   ↓
optimistic state
   ↓
form reset
```

React 19 introduces primitives that let React participate in more of this workflow:

```text
Action
   │
   ├── pending state
   ├── async transition
   ├── error integration
   ├── optimistic UI
   └── form integration
```

This does not remove the need to understand async JavaScript or application state.

It provides better primitives for coordinating them.

---

## 3. Major React 19 Features ⭐⭐⭐⭐⭐

Key additions/improvements include:

```text
Actions
├── useActionState
├── form action / formAction
├── useFormStatus
└── useOptimistic

use() API
├── Promise/resource reading
└── Context reading

Ref improvements
└── ref as a prop

Document/resource improvements
├── title/meta/link metadata
├── stylesheets
├── async scripts
└── preload/preinit APIs

Other improvements
├── hydration diagnostics
├── Suspense improvements
└── server integration
```

Lessons 58–62 study the most important pieces separately.

---

# Part 1 — Actions

## 4. Actions ⭐⭐⭐⭐⭐

React 19 uses the term **Action** for functions used in React's async transition/action model.

By convention, functions that use async transitions are called Actions.

Conceptually:

```jsx
async function updateNameAction(
  newName
) {
  await updateName(newName);
}
```

React's Action-related APIs help coordinate asynchronous mutations with UI state.

---

## 5. Actions Are Not the Same as Server Actions ⭐⭐⭐⭐⭐

This distinction is essential.

```text
Action
→ React concept for mutation/transition workflows

Server Function / commonly "Server Action"
→ function executed on the server through
  framework/server integration
```

An Action can run on the client.

A Server Function can be used as an Action.

Therefore:

```text
Action ≠ automatically server code
```

Do not teach every React Action as a Server Action.

---

## 6. Pending State

Before:

```jsx
const [isPending, setIsPending] =
  useState(false);

async function handleSubmit() {
  setIsPending(true);

  try {
    await save();
  } finally {
    setIsPending(false);
  }
}
```

React 19 Action APIs can integrate pending state with the async operation.

This reduces repetitive coordination code.

---

## 7. useActionState ⭐⭐⭐⭐⭐

```jsx
const [
  state,
  dispatchAction,
  isPending,
] = useActionState(
  action,
  initialState
);
```

It manages state representing the result of an Action.

The Action receives:

```text
previousState
+
action payload
```

and returns the next state.

Unlike a `useReducer` reducer, the Action may be async and perform side effects.

Lesson 58 covers this deeply.

---

## 8. Form Actions ⭐⭐⭐⭐⭐

React DOM forms can receive a function:

```jsx
<form action={saveProfile}>
  <input name="name" />

  <button type="submit">
    Save
  </button>
</form>
```

React calls the function with submitted `FormData`.

When a function is used as the form Action, React runs the submission as an Action/Transition.

You do not need the traditional:

```jsx
event.preventDefault();
```

for this Action-based submission pattern.

---

## 9. Successful Form Action Reset

When a function-based `<form action>` completes successfully, React resets **uncontrolled** form fields automatically.

This does not mean React magically resets arbitrary controlled state.

Controlled fields still follow their state source of truth.

---

## 10. formAction

Individual submit controls can choose a different Action:

```jsx
<form action={saveDraft}>
  <input name="title" />

  <button type="submit">
    Save draft
  </button>

  <button
    type="submit"
    formAction={publish}
  >
    Publish
  </button>
</form>
```

The button's `formAction` can override the form's Action.

---

## 11. useFormStatus

`useFormStatus` comes from `react-dom`. It tells a **component inside a form** whether that form's Action is currently submitting.

### The problem

When someone clicks Save, we want the button to display **Saving...** and become disabled until the request finishes. Without form status, we might manually manage `isLoading` state.

### Practical example

```jsx
import { useFormStatus } from "react-dom";

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? "Saving..." : "Save"}
    </button>
  );
}

function ProfileForm() {
  async function saveProfile(formData) {
    await updateProfile(formData);
  }

  return (
    <form action={saveProfile}>
      <input name="name" placeholder="Your name" />
      <SubmitButton />
    </form>
  );
}
```

### What happens?

```text
Before clicking Save  → pending = false → "Save"
While Action runs     → pending = true  → "Saving..." (disabled)
After Action settles  → pending = false → "Save"
```

React tracks the submission automatically; no separate `useState` is needed for this button.

### What does "descendant component" mean?

```text
ProfileForm
└── <form action={saveProfile}>
    ├── <input />
    └── <SubmitButton />  ← calls useFormStatus()
```

`SubmitButton` is **inside** the form, so it can read that parent form's status.

**Important:** Calling `useFormStatus()` in `ProfileForm` itself does **not** track the `<form>` that `ProfileForm` returns. The hook reads the closest *parent* form above the component calling it. Put it in a child component rendered inside the form.

### Reusable button

```jsx
function SubmitButton({ label }) {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? "Please wait..." : label}
    </button>
  );
}

// Each button reads its own enclosing form:
<form action={loginAction}>
  <SubmitButton label="Login" />
</form>

<form action={signupAction}>
  <SubmitButton label="Signup" />
</form>
```

### Common mistakes and interview points

- Import `useFormStatus` from **`react-dom`**, not `react`.
- It tracks the nearest parent form's submission, not an arbitrary form elsewhere.
- It works with React form Actions (`action` / `formAction`); it does not automatically track a traditional async `onSubmit` handler.
- Besides `pending`, it provides `data`, `method`, and `action`.
- **Interview answer:** `useFormStatus` lets a descendant of a form read its submission status, enabling reusable loading/disabled submit buttons without manually managing pending state.

---

## 12. useOptimistic

Optimistic UI means showing the expected successful state before the server/network operation finishes.

```text
user action
    ↓
optimistic UI immediately
    ↓
request runs
 ┌──┴───┐
 ↓      ↓
success error
 ↓      ↓
confirm rollback/reconcile
```

React 19's `useOptimistic` supports this pattern.

Lesson 60 covers it deeply.

---

# Part 2 — use()

## 13. The use() API ⭐⭐⭐⭐⭐

React 19 introduces `use` for reading supported resources during render.

```jsx
import { use } from "react";

const value =
  use(resource);
```

Important supported resources include:

- Promises
- Context

---

## 14. Reading a Promise

```jsx
function Profile({
  profilePromise,
}) {
  const profile =
    use(profilePromise);

  return (
    <h1>{profile.name}</h1>
  );
}
```

If the Promise is pending:

```text
component suspends
→ nearest Suspense fallback
```

If it resolves:

```text
React retries
→ use returns value
```

If it rejects:

```text
error propagates
→ nearest Error Boundary
```

---

## 15. use() Is Different from Ordinary Hooks ⭐⭐⭐⭐⭐

Unlike ordinary Hooks such as `useState`, `use` may be called in conditions and loops.

Example:

```jsx
function Heading({
  showTheme,
}) {
  if (showTheme) {
    const theme =
      use(ThemeContext);

    return (
      <h1 className={theme}>
        Hello
      </h1>
    );
  }

  return <h1>Hello</h1>;
}
```

However, `use` still must be called from a React Component or Hook.

Lesson 59 covers the rules carefully.

---

# Part 3 — Ref Improvements

## 16. ref as a Prop ⭐⭐⭐⭐⭐

React 19 lets function components receive `ref` as a prop.

```jsx
function MyInput({
  ref,
  ...props
}) {
  return (
    <input
      ref={ref}
      {...props}
    />
  );
}
```

Parent:

```jsx
<MyInput ref={inputRef} />
```

New React 19 function components generally no longer need `forwardRef`.

You studied this deeply in Lesson 56.

---

## 17. forwardRef Status

Do not say:

```text
forwardRef has been removed
```

That is incorrect.

The modern distinction is:

```text
React <=18
→ forwardRef commonly needed

React 19
→ function components can receive ref as a prop
→ forwardRef no longer necessary for new code
→ still exists for compatibility
```

---

## 18. Callback Ref Cleanup

React 19 supports cleanup returned from callback refs:

```jsx
<div
  ref={(node) => {
    register(node);

    return () => {
      unregister(node);
    };
  }}
/>
```

Avoid accidentally returning a value from an assignment-style ref callback.

---

# Part 4 — Context Improvements

## 19. Context Provider Shorthand

In React 19, a Context object itself can be rendered as a provider.

Traditional:

```jsx
<ThemeContext.Provider
  value="dark"
>
  <App />
</ThemeContext.Provider>
```

React 19:

```jsx
<ThemeContext value="dark">
  <App />
</ThemeContext>
```

Older `.Provider` syntax remains important when reading existing code.

---

## 20. use(Context)

React 19 also allows:

```jsx
const theme =
  use(ThemeContext);
```

A major difference from `useContext` is that `use` can be used conditionally.

This does not mean every existing `useContext` call must be rewritten.

Choose clarity and compatibility appropriately.

---

# Part 5 — Document and Resource Improvements

## 21. Document Metadata

React 19 can recognize metadata rendered from components:

```jsx
function Profile({
  user,
}) {
  return (
    <>
      <title>
        {user.name} | CodeBuddy
      </title>

      <meta
        name="description"
        content={
          user.bio
        }
      />

      <main>
        ...
      </main>
    </>
  );
}
```

React can hoist supported metadata such as `title`, `meta`, and relevant `link` elements into the document head.

Lesson 62 covers this area.

---

## 22. Stylesheet Support

React 19 improves integration for stylesheets rendered as part of component trees.

React can coordinate stylesheet loading with rendering and Suspense and deduplicate matching resources.

This helps colocate resource declarations with the components that depend on them.

---

## 23. Async Script Support

Async scripts can be declared in component trees:

```jsx
<script
  async
  src="/analytics.js"
/>
```

React can deduplicate identical async script resources and coordinate them appropriately across rendering environments.

---

## 24. Resource Preloading APIs

React DOM includes resource hint APIs such as:

```js
prefetchDNS()
preconnect()
preload()
preinit()
```

They let applications communicate resource priority earlier.

Examples:

```text
prefetchDNS
→ resolve host early

preconnect
→ establish connection early

preload
→ fetch resource early

preinit
→ eagerly load/initialize supported resource
```

These are performance tools, not APIs to call everywhere.

---

# Part 6 — Suspense and Rendering Improvements

## 25. Suspense Improvements

React 19 includes improvements to how suspended trees are prepared.

A key high-level idea is **pre-warming**:

```text
boundary suspends
     ↓
fallback can commit
     ↓
React can prepare suspended siblings/tree
for a later render
```

You do not need to memorize implementation internals for everyday work.

Understand that Suspense behavior/performance continued evolving in React 19.

---

## 26. Better Hydration Error Diagnostics

Hydration mismatches are difficult when server HTML and client output differ.

React 19 improves hydration diagnostics so developers receive clearer mismatch information rather than multiple vague warnings.

Conceptually:

```text
server output
      ≠
client output
      ↓
clearer diagnostic diff/context
```

This is primarily a debugging improvement.

---

# Part 7 — Server Concepts

## 27. Server Functions

A function marked with:

```js
"use server";
```

is a **Server Function** in the React Server Components ecosystem.

A framework provides the infrastructure to call it from the client.

Important:

```text
"use server"
does NOT mean
"This is a Server Component"
```

There is no `"use server"` directive for declaring Server Components.

The directive marks Server Functions.

---

## 28. React vs Framework Responsibilities ⭐⭐⭐⭐⭐

React provides primitives and specifications.

Frameworks commonly provide:

- routing
- bundling
- server execution
- Server Component infrastructure
- caching strategies
- deployment integration

Therefore this React repository teaches the React mental model.

Framework-specific Next.js implementation belongs in the separate Next.js repository.

---

# Part 8 — Migration Mindset

## 29. React 19 Does Not Mean Rewrite Everything

Existing code using:

- `onSubmit`
- `useState`
- `useEffect`
- `useContext`
- `forwardRef`

does not become automatically invalid.

React 19 adds better tools for particular problems.

Ask:

```text
Does the new primitive simplify
this actual workflow?
```

Do not migrate just for novelty.

---

## 30. Traditional onSubmit Is Still Valid

This remains valid:

```jsx
function handleSubmit(
  event
) {
  event.preventDefault();

  // submit manually
}

<form onSubmit={handleSubmit}>
  ...
</form>
```

Form Actions are an additional React 19 model.

Understanding both is important for production code.

---

## 31. React 19 Feature Map ⭐⭐⭐⭐⭐

```text
Mutation workflow?
→ Actions

Need Action result + pending state?
→ useActionState

Need descendant form pending state?
→ useFormStatus

Need immediate expected UI?
→ useOptimistic

Need to read Promise/resource?
→ use()

Need Context conditionally?
→ use(Context)

Need ref through function component?
→ ref prop

Need component-level metadata/resources?
→ React 19 DOM improvements
```

---

## 32. CareerLoop Example

Submitting a new job application can involve:

```text
<form action={...}>
      │
      ├── FormData
      ├── pending state
      ├── validation result
      └── successful reset
```

React 19 Actions can simplify this workflow.

An optimistic status change could later use `useOptimistic`.

---

## 33. CodeBuddy Example

A connection request can conceptually become:

```text
user clicks Connect
       ↓
Action starts
       ↓
optimistic "Requested"
       ↓
network mutation
   ┌───┴────┐
success   failure
   ↓         ↓
confirm    restore/error
```

React 19's Action and optimistic primitives fit this kind of mutation-oriented UX.

---

## 34. Common Mistakes ⭐⭐⭐⭐⭐

1. Calling every Action a Server Action.
2. Assuming Actions only work with forms.
3. Thinking `useActionState` is simply renamed `useState`.
4. Thinking `use` is identical to ordinary Hooks.
5. Creating uncached Promises repeatedly during render.
6. Assuming Suspense automatically tracks `useEffect` fetching.
7. Saying `forwardRef` was removed.
8. Assuming successful form Actions reset controlled React state.
9. Thinking `"use server"` declares a Server Component.
10. Mixing framework-specific behavior with core React concepts.
11. Rewriting stable code just because React 19 provides a newer primitive.

---

## 35. Interview Questions ⭐⭐⭐⭐⭐

### What are the most important React 19 additions?

Actions and form integration, `useActionState`, `useOptimistic`, `use`, ref-as-prop, Context/provider improvements, document/resource handling, and rendering/hydration improvements.

### What is an Action?

A React convention/model for functions participating in async transition/mutation workflows.

### Is every Action a Server Action?

No.

### What does useActionState provide?

The current Action result state, a dispatcher/wrapped Action, and pending state.

### What does use() do?

It reads supported resources such as Promises and Context during render.

### Can use() be called conditionally?

Yes, unlike ordinary Hooks, while still being called inside a Component or Hook.

### What happens when use() reads a pending Promise?

The component suspends and the nearest Suspense boundary can show its fallback.

### What changed with refs?

Function components can receive `ref` as a prop.

### Was forwardRef removed?

No.

### What changed with forms?

Functions can be passed to `action`/`formAction`, integrating submission with React Actions.

### What does "use server" mean?

It marks Server Functions; it does not declare a Server Component.

---

## 36. Interview-Ready Summary ⭐⭐⭐⭐⭐

```text
React 19 improves React's async mutation and resource
model.

Actions integrate asynchronous mutations with pending
state, error handling, optimistic updates, and forms.
useActionState manages an Action's result and pending
state, while useOptimistic supports immediate expected
UI.

The use API can read Promises and Context during render
and integrates Promise reads with Suspense and Error
Boundaries.

React 19 also allows function components to receive ref
as a prop, improves document metadata and resource
handling, and improves hydration/Suspense behavior.

I keep Actions separate from Server Functions:
an Action is a React async workflow concept, while a
Server Function executes on the server through
framework infrastructure.
```

---

## 37. Complete Mental Model ⭐⭐⭐⭐⭐

```text
                    REACT 19
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     Actions          use()       Platform/API
        │              │          improvements
   ┌────┼────┐      ┌──┴──┐          │
   ↓    ↓    ↓      ↓     ↓          ├─ ref prop
ActionState Form  Promise Context     ├─ metadata
Optimistic Status   │                 ├─ resources
                    ↓                 └─ hydration
                 Suspense /
              Error Boundary
```

---

## 38. Key Takeaways

- React 19 deeply improves async UI coordination.
- Actions are broader than Server Functions.
- `useActionState` manages Action result state and pending state.
- Function-based form Actions integrate submissions with Transitions.
- Successful form Actions reset uncontrolled fields.
- `useFormStatus` exposes parent form status.
- `useOptimistic` supports optimistic UI.
- `use()` reads Promises and Context.
- `use()` has more flexible calling rules than ordinary Hooks.
- Function components can receive `ref` as a prop.
- `forwardRef` still exists but is no longer necessary for new React 19 function components.
- React 19 improves metadata, stylesheets, scripts, resource loading, Suspense, and hydration diagnostics.
- `"use server"` marks Server Functions, not Server Components.
- Framework-specific implementations belong in the framework repository.

---

## Next Lesson

➡️ [Lesson 58 — Actions ⭐⭐⭐⭐⭐](./58-actions.md)
