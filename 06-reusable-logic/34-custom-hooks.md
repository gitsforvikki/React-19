# Lesson 34 — Custom Hooks ⭐⭐⭐⭐⭐

## What Is a Custom Hook?

A **custom Hook** is a function whose name starts with `use` and that lets components **reuse React logic**, not JSX. Extract one when several components need the same behavior or the behavior makes a component difficult to read.

## Example — useDebouncedValue

Suppose CareerLoop should search applications only after typing pauses.

```jsx
import { useEffect, useState } from "react";

function useDebouncedValue(value, delay = 300) {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debounced;
}

function SearchInput() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebouncedValue(query, 300);

  useEffect(() => {
    if (!debouncedQuery) return;
    // Trigger a search here, or pass debouncedQuery to a data-fetching hook.
  }, [debouncedQuery]);

  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

**How it works:** each keystroke changes `query`; the Hook schedules an update after 300 ms. Typing again cancels the previous timer. Consumers receive the delayed value.

## Example — useOnlineStatus

Suppose both a header and a Save button need to know whether the browser is online. Instead of duplicating event listeners, extract one Hook.

```jsx
import { useSyncExternalStore } from "react";

function subscribe(callback) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);
  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}

function getSnapshot() {
  return navigator.onLine;
}

function getServerSnapshot() {
  return true; // Consistent server/hydration fallback
}

function useOnlineStatus() {
  return useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
}

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <p>{isOnline ? "Online" : "Offline"}</p>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();
  return <button disabled={!isOnline}>Save</button>;
}
```

**How it works:** the Hook subscribes to browser online/offline events, reads the current status, and unsubscribes when no longer needed. `useSyncExternalStore` is designed for subscribing to browser/external state.

**Benefits:** one reusable subscription, automatic cleanup, and simple consumers. Both components read the same browser status (unlike independent `useState` values). **Note:** `navigator.onLine` is only a browser connectivity hint; it does not guarantee that your API is reachable.

## Example — useChatRoom

Suppose CodeBuddy has multiple chat screens. Each needs to connect to a room and disconnect when the room changes or the screen unmounts.

```jsx
import { useEffect } from "react";

// createConnection is your app's WebSocket/Socket.IO connection helper.
function useChatRoom({ roomId, serverUrl }) {
  useEffect(() => {
    if (!roomId) return;

    const connection = createConnection({ roomId, serverUrl });
    connection.connect();

    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}

function ChatRoom({ roomId }) {
  useChatRoom({ roomId, serverUrl: "https://chat.example.com" });
  return <h2>Chat room: {roomId}</h2>;
}
```

**How it works:** when `roomId` or `serverUrl` changes, React cleans up the previous connection and starts a new one. Cleanup also runs on unmount. In development Strict Mode, an extra setup/cleanup cycle helps reveal incorrect cleanup.

**Benefits:** avoids repeated connection logic, keeps the UI focused on rendering, and prevents stale connections. This is a **pattern example**: you must implement `createConnection` for your Socket.IO/WebSocket client, and manage message listeners and authentication as needed.

---

## Important Concepts

- **Custom Hooks reuse logic, not state.** Two components calling a Hook that contains `useState` normally get separate state instances. To share one state value, use a common parent, Context, or a store.
- Hooks can accept arguments and return values, objects, arrays, or functions.
- Custom Hooks can call built-in Hooks and other custom Hooks, but must follow the **Rules of Hooks** (Lesson 35).
- Effects inside custom Hooks still need correct dependencies and cleanup.
- If logic is plain JavaScript without React Hooks, prefer a normal utility function.
- A helpful Context wrapper is `useTasks()`, which reads Context and checks that its provider exists.

## Common Mistakes

- Assuming a custom Hook automatically shares state across components.
- Calling Hooks conditionally or directly inside event handlers.
- Moving unnecessary Effects into a custom Hook instead of removing them.
- Building a Hook for every trivial helper or adding `useMemo` without a need.

## Interview Quick Check

**Custom Hook vs component?** Hook reuses behavior; component reuses UI.

**Custom Hook vs utility?** A Hook participates in React's Hook system; a utility is ordinary JavaScript.

**Why start with use?** React tooling can recognize and check Hook rules.

---

➡️ [Lesson 35 — Rules of Hooks ⭐⭐⭐⭐⭐](./35-rules-of-hooks.md)
