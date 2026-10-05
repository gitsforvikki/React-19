# Lesson 03 — Setting Up React with Vite

## 1. React vs Vite

React is a **JavaScript UI library**. Vite is development/build tooling commonly used with React.

```text
React → components, state, rendering, hooks
Vite  → dev server, module handling, fast updates, production build
```

They solve different problems.

## 2. Prerequisites

Check Node.js and npm:

```bash
node --version
npm --version
```

Node.js is needed because development tools such as npm and Vite run in the Node.js environment. This does not mean client-side React code runs on Node.js in the browser.

## 3. Create a React Project

A common Vite command is:

```bash
npm create vite@latest
```

Choose React as the framework and JavaScript or TypeScript as the variant.

You can also start with a project name:

```bash
npm create vite@latest react-app
cd react-app
npm install
npm run dev
```

### What the commands mean

- `npm create vite@latest` — runs Vite's current project initializer.
- `npm install` — installs project dependencies.
- `npm run dev` — runs the development script from `package.json`.

## 4. Important Dependency Files

A project commonly contains:

```text
package.json
package-lock.json
node_modules/
```

### package.json

Contains project metadata, dependencies and scripts.

### package-lock.json

Records resolved dependency versions/tree to make installations more reproducible.

### node_modules

Contains installed packages. It normally should **not** be committed to Git.

## 5. Common Vite Scripts

A Vite project has scripts conceptually similar to:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

Therefore:

```text
npm run dev
    ↓
package.json
    ↓
"dev": "vite"
    ↓
development server
```

## 6. Typical Project Structure

A generated project may look approximately like:

```text
react-app/
├── public/
├── src/
│   ├── assets/
│   ├── App.jsx
│   ├── App.css
│   ├── index.css
│   └── main.jsx
├── .gitignore
├── eslint.config.js
├── index.html
├── package.json
├── package-lock.json
└── vite.config.js
```

Generated files can change between Vite versions. Learn the purpose of the important files rather than memorizing every generated filename.

## 7. index.html

The HTML entry document contains the DOM container where React attaches.

Conceptually:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>React App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

Important connection:

```text
index.html
   ├── #root
   └── loads main.jsx
             ↓
        React starts
```

## 8. main.jsx ⭐⭐⭐⭐⭐

A typical entry module:

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css";
import App from "./App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

### Step 1 — Find the DOM container

```js
document.getElementById("root")
```

### Step 2 — Create the React root

```jsx
createRoot(document.getElementById("root"))
```

### Step 3 — Render the component tree

```jsx
.render(
  <StrictMode>
    <App />
  </StrictMode>
)
```

## 9. Application Startup Flow ⭐⭐⭐⭐⭐

Remember this mental model:

```text
Browser requests application
          ↓
      index.html
          ↓
     main.jsx executes
          ↓
   find DOM #root
          ↓
   createRoot(#root)
          ↓
    render(<App />)
          ↓
 component tree renders
          ↓
React commits required DOM
          ↓
    browser displays UI
```

In short:

```text
HTML → JavaScript → React → Components → DOM
```

## 10. App.jsx

A simple root component:

```jsx
function App() {
  return (
    <main>
      <h1>React Learning</h1>
      <p>Welcome to React.</p>
    </main>
  );
}

export default App;
```

`main.jsx` imports and renders it:

```jsx
import App from "./App.jsx";
```

As the application grows:

```text
App
├── Navbar
├── Home
└── Footer
```

## 11. Why .jsx?

JSX allows syntax such as:

```jsx
const element = <h1>Hello React</h1>;
```

It looks similar to HTML but is JSX syntax inside JavaScript. The `.jsx` extension makes JSX-containing source files explicit.

We will study JSX deeply in Lesson 04.

## 12. ES Modules

Modern React projects commonly use ES modules.

Default export:

```jsx
export default App;
```

Default import:

```jsx
import App from "./App.jsx";
```

Named export:

```js
export function calculateTotal() {
  // ...
}
```

Named import:

```js
import { calculateTotal } from "./utils.js";
```

JavaScript module knowledge is important because React applications are composed from many modules.

## 13. What Does Vite Do?

Simplified development flow:

```text
Source Code
├── JSX
├── JavaScript
├── CSS
└── Assets
      ↓
     Vite
      ├── development server
      ├── module processing
      ├── source transformations
      └── development updates
             ↓
          Browser
```

## 14. HMR and Fast Development

**HMR** means Hot Module Replacement.

When a source module changes:

```text
Developer edits source
        ↓
Vite detects change
        ↓
updated module is sent
        ↓
application updates
```

This avoids a traditional full reload cycle for every compatible edit.

React development tooling can also preserve component state during compatible edits through Fast Refresh behavior.

## 15. Development vs Production

Development:

```bash
npm run dev
```

Priorities include:

- fast feedback
- useful errors
- debugging
- fast source updates

Production build:

```bash
npm run build
```

Vite creates deployable production assets, commonly in a `dist/` directory.

```text
Source Code
    ↓
npm run build
    ↓
Vite production build
    ↓
dist/
├── index.html
└── assets/
```

To locally inspect the production build:

```bash
npm run preview
```

`vite preview` is mainly for local previewing. It is not itself a complete production hosting strategy.

## 16. Environment Variables ⭐⭐⭐⭐⭐

Client-exposed Vite variables normally use the `VITE_` prefix.

```env
VITE_API_BASE_URL=https://api.example.com
```

Read it with:

```js
const apiUrl = import.meta.env.VITE_API_BASE_URL;
```

### Security rule

Anything exposed to browser JavaScript must be treated as visible to users.

Never expose secrets such as:

```text
❌ database passwords
❌ private API secrets
❌ payment secret keys
❌ JWT signing secrets
```

A frontend environment variable is configuration, not a secure secret vault.

## 17. public vs Source Assets

A Vite project can contain:

```text
public/
src/assets/
```

An imported source asset can participate in the build pipeline:

```jsx
import logo from "./assets/logo.png";

function Header() {
  return <img src={logo} alt="Logo" />;
}
```

Files in `public` are useful when an asset needs to be served as-is from the public root.

## 18. vite.config.js

Vite can be configured through its configuration file.

Conceptually:

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
});
```

Configuration may later include:

- aliases
- plugins
- build options
- development server options

Keep configuration minimal until the project has a real need.

## 19. StrictMode ⭐⭐⭐⭐

You may see:

```jsx
<StrictMode>
  <App />
</StrictMode>
```

StrictMode enables additional **development-only checks** that help expose problematic patterns and side effects.

It does not add visible UI.

Developers sometimes notice additional development-only rendering or Effect behavior under StrictMode. Do not remove StrictMode simply to hide that behavior. We will understand this properly in the `useEffect` lessons.

## 20. Folder Structure

A larger application might use:

```text
src/
├── components/
├── hooks/
├── pages/
├── services/
├── utils/
├── assets/
├── App.jsx
├── main.jsx
└── index.css
```

There is no universal React folder structure.

A better principle is:

> Organize code according to application size, features, responsibilities and maintainability.

Production architecture has a dedicated lesson later.

## 21. Common Setup Mistakes

### Committing node_modules

Do not normally commit `node_modules/`. Dependencies can be installed from the package files.

### Putting secrets in VITE_* variables

Client-side values are inspectable by users.

### Confusing React with Vite

React is the UI library; Vite is tooling.

### Thinking Node.js runs browser React code

Node.js runs development tooling. Client-side React code executes in the browser.

### Treating the dev server as production deployment

`npm run dev` is for development. Production should use a proper build/deployment strategy.

### Over-configuring early

Do not customize tooling without a real requirement.

## 22. Interview Questions ⭐⭐⭐⭐⭐

### Q1. Why use Vite with React?

**Answer:** React provides the UI model, while Vite provides development and build tooling such as the development server, module processing, fast updates and production builds.

### Q2. What is the role of main.jsx?

**Answer:** It is commonly the browser-side application entry module where the React root is created and the top-level component tree is rendered.

### Q3. What does createRoot() do?

**Answer:** It creates a React root associated with a DOM container so React can manage and render a component tree there.

### Q4. npm run dev vs npm run build?

**Answer:** `npm run dev` starts the development environment, while `npm run build` creates production assets.

### Q5. What is HMR?

**Answer:** Hot Module Replacement updates changed modules during development without requiring a traditional full reload for every compatible source change.

### Q6. What is StrictMode?

**Answer:** StrictMode enables additional development-only checks that help reveal problematic patterns and side effects.

### Q7. Can frontend environment variables contain secrets?

**Answer:** No real secrets should be exposed to browser code because users can inspect values delivered to the client.

### Q8. Why is Node.js needed?

**Answer:** Node.js runs development tooling such as npm and Vite. It does not mean client React components execute in Node.js.

## 23. Complete Development Flow

```text
Developer
   ↓
JSX / JS / CSS
   ↓
Vite
   ├── serves modules
   ├── transforms source
   └── handles dev updates
          ↓
       Browser
          ↓
        React
          ↓
   Component Tree
          ↓
        DOM/UI
```

Production:

```text
Source
  ↓
npm run build
  ↓
Vite
  ↓
Production Assets
  ↓
Hosting
  ↓
Browser
```

## 24. Quick Revision

```text
React          = UI library
Vite           = development/build tooling
Node.js        = runs development tooling
npm            = package manager/CLI
main.jsx       = application entry module
createRoot()   = creates a React root
App.jsx        = commonly top-level component
npm run dev    = development
npm run build  = production build
npm run preview= local production-build preview
```

## 25. Key Takeaways

- React and Vite solve different problems.
- `index.html` provides the DOM container used by React.
- `main.jsx` commonly creates the React root.
- `App.jsx` commonly represents the top-level application component.
- Understand the flow **HTML → main.jsx → createRoot → App → component tree → DOM**.
- Vite provides fast development tooling and production builds.
- StrictMode provides useful development checks.
- Never expose real secrets in browser-side environment variables.
- Learn the purpose of the setup rather than memorizing boilerplate.

---

## Next Lesson

➡️ [Lesson 04 — JSX Fundamentals](./04-jsx-fundamentals.md)
