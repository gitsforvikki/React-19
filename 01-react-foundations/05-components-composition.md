# Lesson 05 — Components and Component Composition ⭐⭐⭐⭐⭐

## 1. What is a React Component?

A **React component** is a reusable unit of UI.

In modern React, components are normally JavaScript functions that return JSX:

```jsx
function Welcome() {
  return <h1>Welcome to React</h1>;
}
```

Use it like:

```jsx
function App() {
  return <Welcome />;
}
```

Mental model:

```text
Component
   │
   ├── receives inputs
   │
   ↓
JavaScript logic
   │
   ↓
returns JSX
   │
   ↓
describes UI
```

A useful simplified idea is:

```text
UI = Component(props, state)
```

Props and state will get dedicated lessons.

---

## 2. Why Components Matter

Imagine an e-commerce page without components:

```text
One huge file
├── Navbar markup
├── Search markup
├── Product markup
├── Cart markup
├── Footer markup
└── all related logic
```

As the application grows, this becomes difficult to understand and maintain.

With components:

```text
App
├── Navbar
│   └── SearchBar
├── ProductList
│   ├── ProductCard
│   ├── ProductCard
│   └── ProductCard
├── Cart
└── Footer
```

Each component represents a meaningful part of the interface.

Benefits include:

- reusability
- maintainability
- readability
- separation of responsibilities
- easier testing
- easier team collaboration
- easier refactoring

---

## 3. Defining a Component

A component is usually a function whose name starts with an uppercase letter:

```jsx
function ProductCard() {
  return (
    <article>
      <h2>Keyboard</h2>
      <p>₹2,500</p>
    </article>
  );
}
```

Use it with JSX:

```jsx
<ProductCard />
```

### Why uppercase?

React distinguishes platform elements from custom components:

```jsx
<div />
<button />
<section />
```

are DOM elements.

```jsx
<ProductCard />
<Navbar />
<UserProfile />
```

refer to components.

---

## 4. A Component is More Than Reusable Markup

A component can combine:

- markup
- behavior
- state
- event handling
- derived values
- other components

Example:

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <section>
      <p>Count: {count}</p>
      <button onClick={() => setCount((current) => current + 1)}>
        Increment
      </button>
    </section>
  );
}
```

The component owns both the UI description and behavior associated with that UI.

---

## 5. Component Tree ⭐⭐⭐⭐⭐

React applications form a **tree of components**.

```jsx
function App() {
  return (
    <>
      <Navbar />
      <MainContent />
      <Footer />
    </>
  );
}
```

Suppose `MainContent` renders more components:

```jsx
function MainContent() {
  return (
    <main>
      <Sidebar />
      <ProductList />
    </main>
  );
}
```

Then:

```text
App
├── Navbar
├── MainContent
│   ├── Sidebar
│   └── ProductList
└── Footer
```

This hierarchy is fundamental to understanding:

- props
- state ownership
- Context
- rendering
- reconciliation
- performance

---

## 6. Parent and Child Components

If one component renders another component:

```jsx
function App() {
  return <Navbar />;
}
```

then:

```text
App      = parent
Navbar   = child
```

A child can itself be a parent:

```text
App
 ↓
Navbar
 ↓
UserMenu
```

Here:

- `App` is parent of `Navbar`
- `Navbar` is child of `App`
- `Navbar` is also parent of `UserMenu`

---

## 7. Component Composition ⭐⭐⭐⭐⭐

**Composition** means building larger components by combining smaller components.

Example:

```jsx
function Avatar() {
  return <img src="/avatar.jpg" alt="User" />;
}

function UserInfo() {
  return (
    <div>
      <h2>Vikash</h2>
      <p>React Developer</p>
    </div>
  );
}

function ProfileCard() {
  return (
    <article>
      <Avatar />
      <UserInfo />
    </article>
  );
}
```

Composition:

```text
ProfileCard
├── Avatar
└── UserInfo
```

Instead of building one giant component, we compose the UI from focused pieces.

---

## 8. Why React Prefers Composition

Composition allows behavior and UI to be assembled without relying on complicated inheritance hierarchies.

Think:

```text
Small Components
      ↓
Combined Together
      ↓
Larger Feature
      ↓
Complete Application
```

Example:

```text
Button
Input
Avatar
   ↓
LoginForm
UserCard
   ↓
Page
   ↓
Application
```

A famous React design principle is:

> Prefer composition over inheritance for UI reuse.

---

## 9. Reusing Components with Props

A hard-coded component is not very reusable:

```jsx
function ProductCard() {
  return (
    <article>
      <h2>Keyboard</h2>
      <p>₹2500</p>
    </article>
  );
}
```

A reusable component accepts data:

```jsx
function ProductCard({ name, price }) {
  return (
    <article>
      <h2>{name}</h2>
      <p>₹{price}</p>
    </article>
  );
}
```

Now:

```jsx
<ProductCard name="Keyboard" price={2500} />
<ProductCard name="Mouse" price={1000} />
<ProductCard name="Monitor" price={15000} />
```

Same component definition, different inputs.

Props are covered deeply in Lesson 06.

---

## 10. Composition with children ⭐⭐⭐⭐⭐

One of React's most useful composition mechanisms is `children`.

Example:

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}
```

Use:

```jsx
<Card>
  <h2>React</h2>
  <p>Component composition is powerful.</p>
</Card>
```

Conceptually:

```text
<Card>
    │
    └── children
         ├── <h2 />
         └── <p />
```

The `Card` component controls the outer structure while the parent decides what content goes inside.

This makes the component flexible.

---

## 11. children is a Prop

This:

```jsx
<Card>
  <h2>Hello</h2>
</Card>
```

conceptually provides the nested content through the `children` prop.

So:

```jsx
function Card({ children }) {
  return <div>{children}</div>;
}
```

The child content can be:

- text
- an element
- multiple elements
- another component
- conditionally produced content

---

## 12. Wrapper Components

Composition is useful for reusable layout/wrapper components.

```jsx
function Modal({ children }) {
  return (
    <div className="modal-backdrop">
      <div className="modal">{children}</div>
    </div>
  );
}
```

Use:

```jsx
<Modal>
  <h2>Delete account?</h2>
  <p>This action cannot be undone.</p>
  <button>Confirm</button>
</Modal>
```

The modal provides structure. The caller provides content.

This is generally more reusable than creating separate modal components for every message.

---

## 13. Multiple Composition Slots

Sometimes a component needs multiple customizable areas.

Example:

```jsx
function PageLayout({ header, sidebar, children }) {
  return (
    <div>
      <header>{header}</header>

      <div className="layout">
        <aside>{sidebar}</aside>
        <main>{children}</main>
      </div>
    </div>
  );
}
```

Usage:

```jsx
<PageLayout
  header={<Navbar />}
  sidebar={<Sidebar />}
>
  <Dashboard />
</PageLayout>
```

Mental model:

```text
PageLayout
├── header slot
├── sidebar slot
└── children slot
```

Props can therefore carry not only data but also React elements.

---

## 14. Extracting Components

Suppose:

```jsx
function ProfilePage() {
  return (
    <main>
      <div>
        <img src="/avatar.jpg" alt="Vikash" />
        <h2>Vikash</h2>
        <p>React Developer</p>
      </div>

      <section>
        <h3>Recent Posts</h3>
        {/* many post elements */}
      </section>

      <section>
        <h3>Connections</h3>
        {/* many connection elements */}
      </section>
    </main>
  );
}
```

Possible extraction:

```text
ProfilePage
├── ProfileHeader
├── PostList
│   └── PostCard
└── ConnectionList
    └── ConnectionCard
```

This makes responsibilities easier to understand.

---

## 15. When Should You Create a Component?

There is no fixed rule like:

> Every 20 lines must become a component.

Good reasons to extract a component include:

### 1. Reuse

The same UI appears in multiple places.

### 2. Clear responsibility

A part of the UI represents a meaningful concept.

### 3. Complexity

A parent component is becoming difficult to understand.

### 4. Independent behavior

A section has its own state or event logic.

### 5. Testing

A meaningful UI unit benefits from isolated testing.

### 6. Readability

A named component explains intent better than a large block of markup.

---

## 16. Do Not Over-Componentize

This is also important.

Bad extraction can produce unnecessary indirection:

```text
App
 ↓
Page
 ↓
PageContent
 ↓
ContentWrapper
 ↓
ContentInner
 ↓
TextWrapper
 ↓
Text
```

If each component adds no meaningful responsibility, reuse, abstraction, or behavior, the code becomes harder to navigate.

Good component design is about **useful boundaries**, not maximum component count.

---

## 17. Component Responsibility ⭐⭐⭐⭐⭐

A component should ideally have a clear reason to exist.

For example:

```text
ProductCard
├── product image
├── product name
├── price
└── add-to-cart action
```

This is cohesive: everything belongs to the concept of a product card.

A component that manages unrelated responsibilities is harder to maintain:

```text
Dashboard
├── navbar
├── authentication logic
├── product filtering
├── payment handling
├── chat socket
├── profile editing
└── analytics
```

As complexity grows, split responsibilities into meaningful components/hooks/modules.

---

## 18. Component Purity ⭐⭐⭐⭐⭐

React expects components to behave like **pure calculations during rendering**.

A pure function has the basic idea:

```text
same inputs
    ↓
same result
```

A component should calculate JSX from its inputs rather than changing unrelated external values during render.

Good:

```jsx
function Price({ amount }) {
  const formatted = "₹" + amount;

  return <span>{formatted}</span>;
}
```

Problematic:

```jsx
let renderCount = 0;

function Profile() {
  renderCount++;

  return <p>Rendered {renderCount} times</p>;
}
```

The component modifies external state while rendering.

Rendering should primarily **calculate UI**, not perform unrelated side effects.

---

## 19. Why Purity Matters

React may render components more than once.

Future lessons will cover:

- state updates
- StrictMode
- concurrent rendering
- interrupted rendering
- reconciliation

If rendering itself performs uncontrolled side effects, repeated or interrupted rendering can cause bugs.

Mental model:

```text
Render Phase
     ↓
Calculate desired UI
     ↓
Should be pure

Effects / Events
     ↓
Perform appropriate side effects
```

This distinction becomes extremely important later.

---

## 20. Do Not Define Components Inside Components ⭐⭐⭐⭐⭐

Avoid this pattern:

```jsx
function App() {
  function Profile() {
    return <h2>Profile</h2>;
  }

  return <Profile />;
}
```

Prefer:

```jsx
function Profile() {
  return <h2>Profile</h2>;
}

function App() {
  return <Profile />;
}
```

Why?

When a component function is defined inside another component, a new component function identity is created whenever the parent renders.

This can cause React to treat the nested component as a different component type and reset its state.

This becomes clearer when we study **component identity and preserving/resetting state**.

---

## 21. Component Identity ⭐⭐⭐⭐⭐

React cares about **what component type appears at a particular position in the UI tree**.

For example:

```jsx
function App() {
  return <Counter />;
}
```

If React sees the same `Counter` component type at the same relevant tree position across renders, its state can be preserved.

If the component type changes:

```text
Previous:
App
└── Counter

Next:
App
└── Profile
```

React does not treat `Profile` as the old `Counter`.

Component identity is essential for understanding:

- state preservation
- state reset
- keys
- reconciliation

We will study it deeply in Lesson 18.

---

## 22. Composition vs Conditional Duplication

Instead of duplicating large UI blocks:

```jsx
if (isAdmin) {
  return (
    <Card>
      <h2>Admin</h2>
      <p>Welcome</p>
    </Card>
  );
}

return (
  <Card>
    <h2>User</h2>
    <p>Welcome</p>
  </Card>
);
```

you can compose the changing part:

```jsx
return (
  <Card>
    <h2>{isAdmin ? "Admin" : "User"}</h2>
    <p>Welcome</p>
  </Card>
);
```

Good composition reduces duplication while keeping intent clear.

---

## 23. Composition vs Inheritance ⭐⭐⭐⭐⭐

In object-oriented design, reuse is sometimes modeled with inheritance:

```text
BaseCard
  ↑
AdminCard
  ↑
SpecialAdminCard
```

React UI is normally better modeled by combining components:

```text
Card
├── Avatar
├── UserInfo
└── Actions
```

Instead of asking:

> Which UI component should this inherit from?

React developers commonly ask:

> Which components should this component contain?

---

## 24. Presentational Composition Example

Reusable pieces:

```jsx
function Avatar({ src, name }) {
  return <img src={src} alt={name} />;
}

function Button({ children, onClick }) {
  return <button onClick={onClick}>{children}</button>;
}

function UserCard({ user, onConnect }) {
  return (
    <article>
      <Avatar src={user.avatar} name={user.name} />

      <div>
        <h2>{user.name}</h2>
        <p>{user.role}</p>
      </div>

      <Button onClick={() => onConnect(user.id)}>
        Connect
      </Button>
    </article>
  );
}
```

Tree:

```text
UserCard
├── Avatar
├── User information
└── Button
```

Each component has a focused responsibility.

---

## 25. Real-World Example: Developer Networking App

A developer networking page could be designed as:

```text
App
├── Navbar
│   ├── Logo
│   ├── Search
│   └── UserMenu
│
├── DiscoveryPage
│   ├── FilterBar
│   └── DeveloperFeed
│       └── DeveloperCard
│           ├── Avatar
│           ├── SkillList
│           └── ConnectionActions
│
└── Footer
```

This hierarchy gives each feature a natural place.

Later, data and state can flow through these boundaries using:

- props
- state
- Context
- external state management when justified

---

## 26. Reusable Component API Design ⭐⭐⭐⭐⭐

A reusable component should expose enough flexibility without making its API unnecessarily complicated.

Example:

```jsx
function Button({
  children,
  variant = "primary",
  disabled = false,
  onClick,
}) {
  return (
    <button
      className={"button button-" + variant}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

Usage:

```jsx
<Button onClick={saveProfile}>
  Save
</Button>

<Button variant="danger" onClick={deleteProfile}>
  Delete
</Button>
```

The component provides a consistent abstraction while allowing controlled customization.

---

## 27. Avoid Components with Too Many Configuration Props

A warning sign:

```jsx
<Card
  showHeader
  showFooter
  showAvatar
  showActions
  showMenu
  compact
  horizontal
  bordered
  editable
  adminMode
  ...
/>
```

Many boolean props can indicate that one component is trying to represent too many different structures.

Composition may produce a cleaner API.

Instead of making one component understand every possible arrangement, allow callers to compose meaningful pieces.

This idea becomes especially useful when designing reusable component libraries.

---

## 28. Props vs Composition

Suppose a card needs a footer.

Option 1 — pass simple configuration:

```jsx
<Card showFooter />
```

Option 2 — compose actual content:

```jsx
<Card>
  <CardContent />
  <CardFooter>
    <Button>Save</Button>
  </CardFooter>
</Card>
```

The second approach can be more flexible because the parent controls what is composed.

Neither pattern is automatically correct in every case.

Choose the simplest API that fits the actual use case.

---

## 29. Components Should Not Be Called Like Normal Functions ⭐⭐⭐⭐⭐

Given:

```jsx
function Profile() {
  return <h1>Profile</h1>;
}
```

Prefer rendering:

```jsx
<Profile />
```

rather than manually invoking:

```jsx
Profile()
```

Using JSX tells React that `Profile` is a component in the React tree.

This allows React to correctly participate in:

- component identity
- hooks
- reconciliation
- state lifecycle
- developer tooling

Think:

```text
<Profile />
    ↓
React owns component execution
```

---

## 30. Component Rendering Does Not Mean DOM Recreation ⭐⭐⭐⭐⭐

When a component renders again:

```text
Component function executes
        ↓
New UI description is produced
        ↓
React compares/reconciles
        ↓
Only required host/DOM changes are committed
```

Do not confuse:

```text
component render
```

with:

```text
entire DOM recreated
```

Rendering and DOM mutation are different concepts.

This will be covered deeply in later rendering and reconciliation lessons.

---

## 31. Common Component Design Mistakes

### Mistake 1 — Giant components

One component handles too many unrelated responsibilities.

### Mistake 2 — Over-componentization

Tiny wrappers are extracted without meaningful reuse or responsibility.

### Mistake 3 — Hard-coded reusable UI

A component contains fixed data when it should receive inputs.

### Mistake 4 — Defining components inside components

This can create unstable component identity and state-reset problems.

### Mistake 5 — Side effects during rendering

Rendering should primarily calculate UI.

### Mistake 6 — Too many boolean configuration props

The component may be representing too many variations.

### Mistake 7 — Duplicating markup instead of composing

Reusable wrappers and child content can often remove duplication.

### Mistake 8 — Calling component functions manually

Render components through JSX and let React manage them.

### Mistake 9 — Extracting by line count alone

Component boundaries should represent meaningful concepts/responsibilities.

---

## 32. Interview Questions ⭐⭐⭐⭐⭐

### Q1. What is a React component?

**Answer:** A React component is a reusable unit of UI, commonly represented by a JavaScript function that receives inputs such as props and returns JSX describing what should be rendered.

### Q2. What is component composition?

**Answer:** Component composition is the practice of building larger UI structures by combining smaller components.

### Q3. Why does React prefer composition over inheritance?

**Answer:** UI is naturally hierarchical, and composition allows components to be combined and customized without creating rigid inheritance hierarchies.

### Q4. What is the children prop?

**Answer:** `children` represents content nested between a component's opening and closing JSX tags and is commonly used to build reusable wrapper/layout components.

### Q5. When should you extract a component?

**Answer:** When a UI section has a meaningful responsibility, is reused, has independent behavior, becomes complex, or when extraction significantly improves readability/testability.

### Q6. Should every small JSX block become a component?

**Answer:** No. Excessive extraction creates unnecessary indirection. Components should have useful boundaries.

### Q7. What does component purity mean?

**Answer:** During rendering, a component should behave like a pure calculation of its inputs and avoid uncontrolled side effects or mutation of external values.

### Q8. Why should components not usually be defined inside other components?

**Answer:** The nested component function is recreated when the parent renders, giving it a new component type identity and potentially causing React to reset its state.

### Q9. What is a component tree?

**Answer:** It is the hierarchical structure formed when components render other components as parents and children.

### Q10. Does component re-rendering recreate the whole DOM?

**Answer:** No. Rendering creates a new UI description. React reconciles it with the previous tree and commits only the required host/DOM changes.

### Q11. Can components be passed as props?

**Answer:** React elements or component-related values can be passed through props. This enables flexible composition patterns such as layout slots.

### Q12. Why use JSX instead of calling a component function directly?

**Answer:** JSX lets React manage the component as part of its tree, preserving React's component identity, hooks, state, reconciliation, and lifecycle semantics.

---

## 33. Interview Scenario

### Question

You have three pages that need the same modal shell but different content. Would you create three separate modal implementations?

### Better reasoning

Create a reusable modal shell:

```jsx
function Modal({ children }) {
  return (
    <div className="backdrop">
      <div className="modal">
        {children}
      </div>
    </div>
  );
}
```

Then compose:

```jsx
<Modal>
  <DeleteAccount />
</Modal>
```

or:

```jsx
<Modal>
  <EditProfile />
</Modal>
```

The shell is reusable while the content remains flexible.

---

## 34. Component Design Mental Model

When designing a component, ask:

```text
What responsibility does this component have?
              ↓
What data does it need?
              ↓
What behavior does it own?
              ↓
What should be configurable?
              ↓
Can content be composed through children?
              ↓
Is this abstraction actually reusable/useful?
```

---

## 35. Quick Revision

```text
React Component
│
├── JavaScript function
├── receives inputs
├── returns JSX
├── can own state/behavior
└── can render other components
```

```text
Composition
│
├── combine small components
├── build larger features
├── reuse UI structures
├── children for nested content
└── prefer useful boundaries over inheritance
```

Important principles:

```text
Components should be:
├── focused
├── reusable when useful
├── composable
├── predictable during render
└── organized around meaningful responsibilities
```

---

## 36. Key Takeaways

- Components are the fundamental building blocks of React applications.
- Modern React components are normally JavaScript functions returning JSX.
- Components form a hierarchical **component tree**.
- **Composition** builds larger UI from smaller components.
- React generally favors **composition over inheritance**.
- The `children` prop enables flexible wrapper/layout components.
- Props make components reusable with different inputs.
- Extract components based on meaningful responsibilities, not arbitrary line counts.
- Avoid both giant components and unnecessary over-componentization.
- Rendering should remain **pure**; side effects do not belong directly in render logic.
- Avoid defining stateful component types inside other components.
- Component identity affects whether React preserves or resets state.
- A component re-render does not mean the entire DOM is recreated.
- Use JSX such as `<Profile />` rather than manually invoking component functions.
- Good component APIs balance consistency, simplicity, and composition.

---

## Next Lesson

➡️ [Lesson 06 — Props and One-Way Data Flow ⭐⭐⭐⭐⭐](./06-props-data-flow.md)
