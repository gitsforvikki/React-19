# Lesson 02 — SPA, MPA and How React Applications Work

## 1. Why Learn SPA and MPA Before Going Deeper into React?

React is commonly used to build highly interactive web applications. To understand how a React application behaves in the browser, you should first understand two common application models:

- **MPA — Multi-Page Application**
- **SPA — Single-Page Application**

The important difference is not simply how many screens the application has. It is mainly about **how navigation and page updates are handled**.

---

## 2. What is a Multi-Page Application (MPA)?

In a traditional Multi-Page Application, navigating to another page commonly causes the browser to request a new HTML document from the server.

Example:

```text
User opens /products
        ↓
Browser sends request
        ↓
Server returns products HTML
        ↓
Browser loads document

User opens /profile
        ↓
Browser sends another request
        ↓
Server returns profile HTML
        ↓
Browser loads new document
```

A simplified flow:

```text
Browser
   │
   │ GET /products
   ↓
Server
   │
   │ HTML document
   ↓
Browser

Navigation

Browser
   │
   │ GET /profile
   ↓
Server
   │
   │ another HTML document
   ↓
Browser
```

The browser performs a **document navigation**.

---

## 3. What is a Single-Page Application (SPA)? ⭐⭐⭐⭐⭐

A Single-Page Application usually loads an application shell and JavaScript into the browser. After that, client-side JavaScript can update the visible UI and handle many navigations without requesting a completely new HTML document for every screen.

Simplified initial request:

```text
Browser
   │
   │ Initial Request
   ↓
Server
   │
   ├── HTML
   ├── JavaScript
   └── CSS
        ↓
Browser loads application
```

After the application is running:

```text
User Interaction / Navigation
             ↓
      JavaScript executes
             ↓
        State changes
             ↓
       React renders
             ↓
      UI gets updated
```

The application may still communicate with a server for data:

```text
React Application
       │
       │ API Request
       ↓
Backend / API
       │
       │ JSON Data
       ↓
React Application
       │
       ↓
Update State
       │
       ↓
Render UI
```

So **SPA does not mean "no server requests."**

It means that many UI transitions happen inside the already-loaded application instead of requiring a full document navigation every time.

---

## 4. "Single Page" Does Not Mean One Screen

This is a common misunderstanding.

An SPA can have many apparent pages:

```text
/
├── /login
├── /feed
├── /profile
├── /connections
└── /settings
```

To the user, these look like separate pages.

But in a client-rendered SPA, JavaScript can inspect the URL and decide which component tree to render without loading a completely new document.

For example:

```text
URL: /profile

        ↓

Client-side Router

        ↓

<Profile />
```

Libraries such as React Router can provide this routing behavior.

React itself is primarily concerned with the UI; **routing is a separate concern**.

---

## 5. SPA vs MPA

| Feature | SPA | Traditional MPA |
|---|---|---|
| Initial document | Application document loaded initially | HTML document requested for a page |
| Navigation | Often handled client-side | Often loads another document |
| Full document reload | Usually avoided for internal navigation | Common |
| UI updates | JavaScript updates the current application | Server may return a new document |
| Client-side state | Often preserved more naturally | Can reset across document navigation unless persisted |
| JavaScript reliance | Usually higher | Can be lower |
| Initial JS cost | Can be significant | Depends on architecture |
| Routing | Often client-side | Commonly server-driven |

Neither architecture is automatically better in every situation.

---

## 6. Where Does React Fit?

React is responsible for **building and updating UI**.

A simplified browser-side React application can be viewed like this:

```text
index.html
    │
    ↓
JavaScript bundle/module
    │
    ↓
React application starts
    │
    ↓
React renders component tree
    │
    ↓
DOM reflects the UI
```

A typical HTML entry point contains an element such as:

```html
<div id="root"></div>
```

React can attach to that root:

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

Conceptually:

```text
HTML

<div id="root"></div>

        ↓

createRoot(...)

        ↓

<App />

        ↓

Component Tree

        ↓

Browser UI
```

---

## 7. React and ReactDOM

You will commonly see two packages:

### react

Provides React APIs for defining and composing UI behavior.

Examples include:

```js
useState
useEffect
useContext
useReducer
useMemo
useTransition
```

### react-dom

Provides APIs that integrate React with the browser DOM.

For example:

```js
import { createRoot } from "react-dom/client";
```

A useful simplified mental model is:

```text
React
   ↓
Component/UI model

ReactDOM
   ↓
Browser DOM integration
```

---

## 8. What Happens When a React SPA Starts? ⭐⭐⭐⭐⭐

Consider a Vite React application.

The simplified flow is:

```text
1. Browser requests application
            ↓
2. HTML is loaded
            ↓
3. JavaScript modules are loaded
            ↓
4. React application executes
            ↓
5. createRoot() creates a React root
            ↓
6. <App /> is rendered
            ↓
7. Child components are evaluated
            ↓
8. React commits required DOM changes
            ↓
9. Browser displays interactive UI
```

Example:

```jsx
function App() {
  return (
    <main>
      <Navbar />
      <Home />
      <Footer />
    </main>
  );
}
```

Component tree:

```text
App
│
├── Navbar
├── Home
└── Footer
```

React works with this component tree to determine the UI.

---

## 9. What Happens After a User Interaction?

Suppose we have:

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

When the button is clicked:

```text
User clicks button
        ↓
Event handler runs
        ↓
setCount() requests state update
        ↓
React schedules an update
        ↓
Counter renders again
        ↓
React determines required changes
        ↓
DOM change is committed
        ↓
Browser displays new count
```

Notice what did **not** happen:

```text
❌ Entire browser page reload
❌ New HTML document required
❌ Entire DOM blindly recreated
```

This state-driven update model is fundamental to React.

---

## 10. Client-Side Routing in an SPA

Suppose a user is currently at:

```text
/feed
```

and clicks:

```text
/profile
```

With client-side routing, the simplified flow can be:

```text
Click Profile
     ↓
Router updates browser history/URL
     ↓
Router matches /profile
     ↓
Profile component is rendered
     ↓
Relevant UI changes
```

Instead of:

```text
Click Profile
     ↓
Request entirely new HTML document
     ↓
Browser reloads document
```

The browser's **History API** is commonly used by client-side routers to manage URLs without traditional full-page navigation.

---

## 11. Does React Include Routing?

No.

React itself does not provide a general-purpose browser routing solution for SPA navigation.

A React SPA may use a library such as:

```text
React
   │
   └── React Router
          │
          ├── /
          ├── /products
          ├── /profile
          └── /settings
```

Frameworks such as Next.js provide their own routing systems.

We will keep framework-specific routing in the separate Next.js repository.

---

## 12. How APIs Fit Into a React SPA

A common production architecture is:

```text
Browser
   │
   ↓
React Frontend
   │
   │ HTTP / WebSocket
   ↓
Backend API
   │
   ↓
Database
```

For example:

```text
Developer opens connections
          ↓
React component needs connection data
          ↓
Frontend requests /api/connections
          ↓
Backend queries database
          ↓
JSON response
          ↓
Frontend updates state
          ↓
React renders connection cards
```

React does **not** replace the backend.

React primarily manages the UI.

---

## 13. Advantages of SPA Architecture

### Smooth interactions

Many UI changes can happen without full document reloads.

### Rich client-side experiences

SPAs are well suited for highly interactive applications such as:

- dashboards
- social platforms
- admin applications
- chat applications
- project management tools

### Reusable UI

React components make complex screens easier to compose.

### Client-side state

Application state can remain available while users move through different views, depending on how the app is designed.

---

## 14. Challenges of SPA Architecture

SPA architecture also has tradeoffs.

### 1. Initial JavaScript

Large applications can ship significant JavaScript, affecting startup performance.

### 2. SEO considerations

Pure client-side rendering can require additional care for search-engine-visible content.

### 3. Loading states

Data often arrives asynchronously, so applications need good:

- loading states
- error states
- empty states

### 4. Client complexity

Routing, caching, state, authentication, and data synchronization can become complex.

### 5. JavaScript dependency

A heavily client-rendered application depends strongly on JavaScript execution.

Modern frameworks can combine client and server rendering approaches to address different requirements.

---

## 15. Important: React Does Not Mean SPA ⭐⭐⭐⭐⭐

This distinction is important.

React is a **UI library**.

SPA is an **application/navigation architecture**.

Therefore:

```text
React ≠ SPA
```

React can participate in different rendering/application architectures.

For example, React may be used with:

```text
Client-Side Rendering
Server-Side Rendering
Static Generation
Streaming
Server Components
Hybrid architectures
```

These concepts become especially important when working with frameworks such as Next.js.

---

## 16. CSR vs SPA

These terms are related but are not exactly identical.

### SPA

Describes how the application behaves across navigation — typically keeping one application running while switching views client-side.

### CSR — Client-Side Rendering

Describes **where much of the UI rendering work occurs**.

Simplified CSR:

```text
Server
  ↓
HTML shell + JavaScript
  ↓
Browser
  ↓
JavaScript executes
  ↓
UI is rendered
```

An SPA is often client-side rendered, but you should not treat **SPA** and **CSR** as perfect synonyms.

---

## 17. MPA Does Not Mean "Old Technology"

Another common misunderstanding is:

> SPA = modern, MPA = outdated.

That is incorrect.

Different architectures solve different problems.

Modern applications can use combinations of:

- server-rendered pages
- client-side navigation
- static pages
- interactive React components
- streamed content

Architecture should be chosen based on requirements rather than labels.

---

## 18. Common Mistakes

### Mistake 1: "SPA means the website has only one URL."

Incorrect.

An SPA can have many URLs and views.

### Mistake 2: "SPA never talks to a server."

Incorrect.

SPAs frequently communicate with backend APIs.

### Mistake 3: "React automatically provides routing."

Incorrect.

React focuses on UI. Routing requires another solution or framework.

### Mistake 4: "Every React application is an SPA."

Incorrect.

React can be used in multiple architectures.

### Mistake 5: "A React re-render means the whole webpage reloads."

Incorrect.

A React render is part of React's UI update process; it is not the same as a browser document reload.

---

## 19. Interview Perspective ⭐⭐⭐⭐⭐

### Q1. What is an SPA?

**Answer:** A Single-Page Application is an application architecture where the browser typically keeps the same loaded application running and JavaScript updates views and handles many navigations without requesting a completely new HTML document for every screen.

### Q2. What is an MPA?

**Answer:** A Multi-Page Application traditionally uses document navigation, where navigating between pages causes the browser to request another HTML document from the server.

### Q3. Is every React application an SPA?

**Answer:** No. React is a UI library and can be used with client-side, server-side, static, streaming, and hybrid architectures.

### Q4. Does an SPA make server requests?

**Answer:** Yes. SPAs commonly make API requests to retrieve or modify data. The key distinction is that these requests do not necessarily cause a full document navigation.

### Q5. Does React provide routing?

**Answer:** React itself does not provide a complete browser routing system. React applications can use routing libraries such as React Router, while frameworks such as Next.js provide their own routing solutions.

### Q6. What happens when React state changes?

**Answer:** React schedules an update, renders the affected component tree to determine the desired UI, and then commits the necessary changes to the host environment such as the browser DOM.

### Q7. What is the difference between SPA and CSR?

**Answer:** SPA describes an application/navigation model, while CSR describes rendering UI primarily in the browser. They commonly appear together but describe different concerns.

---

## 20. Interview Diagram

```text
                 React Application
                        │
              ┌─────────┴─────────┐
              │                   │
         User Interaction     API Response
              │                   │
              └─────────┬─────────┘
                        ↓
                   State Update
                        ↓
                  React Renders
                        ↓
              Determine UI Changes
                        ↓
                  Commit to DOM
                        ↓
                    Browser UI
```

This mental model will become much more detailed when we study rendering, reconciliation, and Fiber.

---

## 21. Quick Revision

```text
MPA
Navigation
   ↓
Server request
   ↓
New HTML document
   ↓
Browser document load
```

```text
Typical SPA Navigation
Navigation
   ↓
Client-side router
   ↓
Component/view changes
   ↓
No full document reload
```

```text
React Update
State Change
   ↓
Render
   ↓
Determine Changes
   ↓
Commit
   ↓
Updated UI
```

---

## 22. Key Takeaways

- **SPA** means Single-Page Application.
- **MPA** means Multi-Page Application.
- An SPA can contain many routes and screens.
- SPA navigation commonly avoids full document reloads.
- SPAs can and usually do communicate with servers.
- React manages **UI**, not backend/database responsibilities.
- React itself does not automatically provide routing.
- **React and SPA are not the same concept.**
- **SPA and CSR are related but not identical.**
- React can be used in client-rendered, server-rendered, static, and hybrid architectures.
- Understanding **state → render → commit → UI** gives you the foundation for later React internals.

---

## Next Lesson

➡️ [Lesson 03 — Setting Up React with Vite](./03-react-vite-setup.md)
