# Lesson 01 — What is React and Why React?

## 1. What is React?

**React** is a JavaScript library for building user interfaces, especially interactive web applications.

Instead of manually finding DOM elements and changing them whenever application data changes, React lets us describe **what the UI should look like for the current state**. React then handles updating the required parts of the UI.

React applications are primarily built from **components**.

```jsx
function Welcome() {
  return <h1>Hello, React!</h1>;
}
```

Here, `Welcome` is a React component.

---

## 2. Why Was React Needed?

Imagine building an application using only JavaScript.

```html
<h1 id="count">0</h1>
<button id="increment">Increment</button>
```

```js
let count = 0;

const countElement = document.getElementById("count");
const button = document.getElementById("increment");

button.addEventListener("click", () => {
  count++;
  countElement.textContent = count;
});
```

This is manageable for a tiny application.

But in a large application, you may have:

- hundreds of UI elements
- many events
- shared application data
- forms
- API responses
- loading states
- error states
- multiple screens

Manually keeping the DOM synchronized with application data becomes difficult.

React solves this by making the UI **declarative**.

---

## 3. Imperative vs Declarative UI ⭐⭐⭐⭐⭐

### Imperative approach

With normal DOM manipulation, we tell the browser **how** to update the UI.

```js
const heading = document.getElementById("heading");

if (isLoggedIn) {
  heading.textContent = "Welcome back!";
} else {
  heading.textContent = "Please login";
}
```

We manually describe the steps.

### Declarative approach

With React, we describe **what** the UI should be.

```jsx
function Header({ isLoggedIn }) {
  return (
    <h1>
      {isLoggedIn ? "Welcome back!" : "Please login"}
    </h1>
  );
}
```

When `isLoggedIn` changes, React takes care of updating the UI.

### Mental model

```text
Application State
       ↓
React Components
       ↓
      UI
```

A useful simplified idea is:

```text
UI = f(state)
```

The UI is a result of the application's current state.

---

## 4. Component-Based Architecture ⭐⭐⭐⭐⭐

React applications are divided into small reusable pieces called **components**.

For example:

```text
App
├── Navbar
├── Sidebar
├── ProductList
│   ├── ProductCard
│   ├── ProductCard
│   └── ProductCard
└── Footer
```

Each component can manage a specific responsibility.

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

The same component can be reused:

```jsx
<ProductCard name="Laptop" price={50000} />
<ProductCard name="Keyboard" price={2000} />
<ProductCard name="Mouse" price={1000} />
```

### Benefits

Component-based architecture provides:

- reusability
- maintainability
- separation of concerns
- easier testing
- easier collaboration
- consistent UI

We will study components deeply in Lesson 05.

---

## 5. React is a Library, Not a Complete Framework

React mainly focuses on the **UI layer**.

React provides concepts such as:

- components
- JSX
- props
- state
- hooks
- rendering
- context
- transitions
- Suspense

A React application may use other libraries for additional requirements.

For example:

```text
React
  │
  ├── Routing → React Router
  ├── Global State → Redux / Zustand
  ├── Data Fetching → fetch / TanStack Query
  └── Framework → Next.js
```

This is one important difference between **React** and a framework such as **Next.js**.

---

## 6. How React Updates the UI

At a high level:

```text
User Interaction
      ↓
State Changes
      ↓
Component Renders
      ↓
React compares the required UI
      ↓
Necessary DOM updates are committed
      ↓
Browser displays updated UI
```

Example:

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

When the button is clicked:

```text
Click
  ↓
setCount(...)
  ↓
State update is scheduled
  ↓
Counter renders again
  ↓
React determines what changed
  ↓
DOM is updated
```

Do not memorize the internals yet. Rendering, reconciliation, Fiber, batching, and the render/commit phases have dedicated lessons later.

---

## 7. Core Characteristics of React

### 1. Declarative

Describe the UI for the current state rather than manually manipulating the DOM.

### 2. Component-Based

Build applications from reusable components.

### 3. State-Driven

UI can change in response to application state.

### 4. One-Way Data Flow

Data normally flows from parent components to child components through props.

```text
Parent
  │
  │ props
  ↓
Child
```

This makes application behavior easier to reason about.

### 5. Reusable

Components and custom hooks can be reused throughout an application.

---

## 8. React vs Vanilla JavaScript

| React | Vanilla JavaScript |
|---|---|
| Component-based UI | No built-in component model |
| Declarative rendering | Usually imperative DOM manipulation |
| State-driven UI model | State/DOM synchronization is manual |
| Reusable component composition | Reuse must be designed manually |
| React manages UI updates | Developer directly manipulates DOM |

Vanilla JavaScript is not bad. React becomes especially useful when UI complexity and application state grow.

---

## 9. React vs Next.js

A common interview confusion is treating React and Next.js as the same thing.

```text
React
│
└── UI library
     │
     └── Next.js
         Framework built around React
```

React teaches you how to build and reason about components and UI.

Next.js adds framework capabilities such as routing, rendering strategies, server integration, caching, and application-level conventions.

Those Next.js-specific concepts belong in the separate Next.js learning repository.

---

## 10. Common Misunderstandings

### "React is a programming language."

Incorrect.

React is a **JavaScript library**.

### "React replaces JavaScript."

Incorrect.

Strong JavaScript knowledge is essential for React.

### "React directly updates the entire DOM every time state changes."

Incorrect.

A component render does **not** mean React blindly rewrites the whole DOM. React determines what host changes are necessary and commits those changes.

We will understand this properly in the lessons on rendering and reconciliation.

### "React is only for single-page applications."

React is commonly used for SPA interfaces, but React itself is not limited to one application architecture.

---

## 11. When is React Useful?

React is a strong choice for applications with:

- interactive interfaces
- frequently changing data
- reusable UI
- complex state
- dashboards
- e-commerce interfaces
- social applications
- admin panels
- large frontend codebases

For example, a developer networking application might contain:

```text
App
├── Navigation
├── DeveloperFeed
│   └── DeveloperCard
├── Connections
├── Chat
│   └── Message
└── Profile
```

React's component model makes these UI pieces easier to organize and reuse.

---

## 12. Interview Perspective ⭐⭐⭐⭐⭐

### Q1. What is React?

**Answer:** React is a JavaScript library for building declarative, component-based user interfaces. It allows developers to describe UI based on state and efficiently synchronize changes with the rendered interface.

### Q2. Why do we use React?

**Answer:** React helps manage complex interactive UIs using reusable components, declarative rendering, predictable data flow, and state-driven updates.

### Q3. Is React a library or framework?

**Answer:** React is primarily a UI library. It focuses on rendering and component composition, while additional concerns such as routing or broader application architecture may come from other libraries or frameworks.

### Q4. What does declarative UI mean?

**Answer:** We describe what the UI should look like for a given state, and React handles synchronizing the rendered UI when that state changes.

### Q5. What is component-based architecture?

**Answer:** It is an approach where an interface is divided into smaller reusable and composable pieces called components.

### Q6. Why not manipulate the DOM manually?

**Answer:** Manual DOM manipulation becomes difficult to maintain as application state and UI complexity grow. React provides a state-driven model that keeps UI synchronization manageable.

---

## 13. Quick Revision

```text
React
│
├── JavaScript UI library
├── Declarative
├── Component-based
├── State-driven
├── One-way data flow
└── Reusable UI
```

### Remember

> In React, think primarily in terms of **state → rendered UI**, rather than manually changing individual DOM elements.

---

## 14. Key Takeaways

- React is a **JavaScript library for building user interfaces**.
- React applications are composed from **components**.
- React uses a **declarative** programming model.
- UI is driven by **state and props**.
- Data generally flows **parent → child**.
- React helps keep application data and UI synchronized.
- React is a library; **Next.js is a framework built around React**.
- Understanding JavaScript remains essential for becoming strong in React.

---

## Next Lesson

➡️ [Lesson 02 — SPA, MPA and How React Applications Work](../01-react-foundations/02-spa-mpa-react.md)
