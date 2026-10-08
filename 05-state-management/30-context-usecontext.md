# Lesson 30 — Context API and useContext ⭐⭐⭐⭐⭐

## What and When

**Context** shares a value with deeply nested descendants without passing it through every intermediate component (**prop drilling**). Use regular props for nearby components; use Context for values such as theme, locale, or shared feature state.

Context **distributes** values; it does not create or manage state.

## Create → Provide → Consume

```jsx
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext(null);

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  return (
    <ThemeContext value={{ theme, setTheme }}>
      {children}
    </ThemeContext>
  );
}

function ThemeButton() {
  const context = useContext(ThemeContext);
  if (context === null) throw new Error("Missing ThemeProvider");

  const { theme, setTheme } = context;
  return (
    <button onClick={() => setTheme(t => t === "light" ? "dark" : "light")}>
      Theme: {theme}
    </button>
  );
}

function App() {
  return (
    <ThemeProvider>
      <ThemeButton />
    </ThemeProvider>
  );
}
```

React 19 supports `<ThemeContext value={...}>`; older projects use `<ThemeContext.Provider value={...}>`.

## Rules to Remember

- `useContext(Context)` reads the **nearest provider above** the component. Without one, it returns the `createContext` default.
- Consumers update when their context value changes. An object passed as `value={{ ... }}` is new on every provider render, which can trigger unnecessary consumer renders.
- Keep providers near the components that need them; split unrelated values into separate contexts.
- A small custom hook (e.g. `useTheme()`) can centralize the missing-provider check.
- Consider props or component composition first. Context does **not** replace server-data caching, authentication checks, or a specialized external store.

## Common Mistakes

- Putting every local state variable into one global-looking context.
- Mutating context state instead of using its setter/dispatch.
- Expecting `React.memo` to block updates caused by a context value changing.
- Calling `useContext` in the same component **above** the provider it renders; that provider only applies to descendants.

## Interview Quick Check

**Context vs props?** Props are explicit component inputs; Context avoids repeated passing to deep descendants.

**Does Context manage state?** No. `useState` or `useReducer` can own state and pass it through Context.

**What causes context consumers to update?** A changed provider value (compared with `Object.is`).

---

➡️ [Lesson 31 — useReducer ⭐⭐⭐⭐⭐](./31-usereducer.md)
