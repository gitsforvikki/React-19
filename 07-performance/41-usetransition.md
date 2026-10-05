# Lesson 41 — useTransition and Concurrent Rendering ⭐⭐⭐⭐⭐

## 1. Why Transitions Exist

Not every state update has the same urgency.

Imagine a search page:

```text
User types
   │
   ├── input text must update immediately
   │
   └── a very large results UI must recalculate/render
```

If expensive rendering competes with typing, the interface can feel unresponsive.

A **Transition** tells React that an update is non-urgent and may be rendered without blocking urgent interaction.

---

## 2. useTransition

```jsx
import { useTransition } from "react";

const [isPending, startTransition] =
  useTransition();
```

It returns:

- `isPending` — whether Transition work is pending.
- `startTransition` — marks state updates as Transition work.

---

## 3. Basic Example ⭐⭐⭐⭐⭐

```jsx
function SearchPage() {
  const [input, setInput] = useState("");
  const [query, setQuery] = useState("");

  const [isPending, startTransition] =
    useTransition();

  function handleChange(event) {
    const value = event.target.value;

    // Urgent update
    setInput(value);

    // Non-urgent update
    startTransition(() => {
      setQuery(value);
    });
  }

  return (
    <>
      <input
        value={input}
        onChange={handleChange}
      />

      {isPending && <p>Updating…</p>}

      <SearchResults query={query} />
    </>
  );
}
```

Mental model:

```text
setInput(value)
→ urgent
→ typing stays responsive

startTransition(...)
└── setQuery(value)
    → non-blocking update
    → expensive UI may catch up later
```

---

## 4. Concurrent Rendering Mental Model ⭐⭐⭐⭐⭐

Concurrent rendering does **not** mean React renders your components on multiple JavaScript threads.

A useful mental model is:

> React can perform lower-priority rendering in an interruptible way so urgent updates can take priority.

```text
Start Transition render
        │
        ↓
urgent update arrives
        │
        ↓
React prioritizes urgent work
        │
        ↓
latest Transition work continues/restarts
```

---

## 5. Urgent vs Non-Urgent Updates

Usually urgent:

- controlled input value
- immediate click feedback
- direct interaction feedback

Potentially non-urgent:

- expensive tab content
- a large filtered result view
- navigation-like content updates
- expensive visualizations

The distinction is about user experience, not a fixed list of APIs.

---

## 6. Controlled Inputs Must Stay Urgent ⭐⭐⭐⭐⭐

Do not do this:

```jsx
startTransition(() => {
  setText(event.target.value);
});
```

For a controlled input, React needs the input state to update immediately.

Instead:

```jsx
const value = event.target.value;

setText(value);

startTransition(() => {
  setSearchQuery(value);
});
```

Keep immediate interaction state separate from expensive downstream state when necessary.

---

## 7. startTransition Is About React Updates

```jsx
startTransition(() => {
  setTab("analytics");
});
```

It does not mean:

```text
run arbitrary JavaScript later
```

It marks React state updates as Transition work.

It is not a general background-thread API.

---

## 8. Transition Rendering Is Interruptible ⭐⭐⭐⭐⭐

Suppose:

```text
query = "r"
→ expensive Transition begins

then quickly:
query = "re"
→ urgent input update occurs
→ newer Transition work begins
```

React can prioritize the latest interaction instead of forcing obsolete lower-priority rendering to finish first.

---

## 9. isPending

```jsx
const [isPending, startTransition] =
  useTransition();
```

Use `isPending` for useful feedback:

```jsx
<button
  className={isPending ? "pending" : ""}
>
  Analytics
</button>
```

It can be better to keep existing useful content visible and indicate pending work than to replace the whole screen with a spinner.

---

## 10. Transitions and Suspense ⭐⭐⭐⭐⭐

Transitions integrate naturally with Suspense.

Conceptually:

```text
current useful UI
      │
start Transition
      │
new content suspends
      │
keep useful revealed UI when possible
+ indicate pending work
      │
new content becomes ready
      ↓
commit new UI
```

Suspense is covered deeply in Lesson 53.

---

## 11. Standalone startTransition

React also exposes a standalone `startTransition`.

Use `useTransition` when the component needs the `isPending` state.

```text
useTransition
→ start Transition + pending information

startTransition
→ mark update as Transition
  without this Hook's pending flag
```

---

## 12. Transition Is Not setTimeout

```text
startTransition
≠ wait 300ms
```

There is no fixed delay.

A Transition changes how React schedules rendering work.

---

## 13. Transition Is Not Debounce ⭐⭐⭐⭐⭐

Debounce:

```text
r → re → rea → react
                │
       wait for inactivity
                ↓
             run once
```

Transition:

```text
updates may begin immediately
but rendering is non-blocking
and interruptible
```

Debouncing is useful for reducing how often an external operation such as a search request occurs.

Transitions address React rendering responsiveness.

They can coexist because they solve different problems.

---

## 14. Heavy JavaScript Can Still Block

```jsx
startTransition(() => {
  const result = hugeCalculation();
  setResult(result);
});
```

A huge synchronous JavaScript calculation can still block the main thread.

Transitions primarily help React schedule updates/rendering.

For CPU-heavy work also consider:

- better algorithms
- less data
- memoization where justified
- Web Workers
- virtualization

---

## 15. Async Transition Actions in React 19 ⭐⭐⭐⭐⭐

React 19 supports async functions passed to `startTransition`.

These functions are often called **Actions**.

```jsx
startTransition(async () => {
  const result = await saveData();

  startTransition(() => {
    setData(result);
  });
});
```

Important current caveat:

> State updates after an `await` may need another `startTransition` to be marked as Transition updates.

Do not confuse React Transition Actions with Redux action objects.

---

## 16. Transitions Do Not Replace Application State

React may batch multiple ongoing Transitions.

Do not use exact Transition timing as a business rule.

Application semantics such as:

- saved
- failed
- selected
- paid
- connected

should be represented by application state.

---

## 17. Tab Example ⭐⭐⭐⭐⭐

```jsx
function Tabs() {
  const [tab, setTab] = useState("overview");

  const [isPending, startTransition] =
    useTransition();

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }

  return (
    <>
      <TabButtons
        tab={tab}
        onSelect={selectTab}
      />

      {isPending && <span>Updating…</span>}

      <TabContent tab={tab} />
    </>
  );
}
```

If `TabContent` is expensive, the surrounding interface can remain responsive.

---

## 18. CareerLoop Example

Suppose changing:

```text
All → Applied → Interview → Offer
```

causes thousands of complex application cards to render.

```jsx
function selectStatus(status) {
  startTransition(() => {
    setStatus(status);
  });
}
```

This is useful only if profiling shows the update is expensive enough to affect responsiveness.

---

## 19. CodeBuddy Example

A discovery search could use:

```jsx
setInput(value);

startTransition(() => {
  setDiscoveryQuery(value);
});
```

The input stays urgent while expensive discovery UI is non-urgent.

Network request debouncing/cancellation remains a separate concern.

---

## 20. useTransition vs useDeferredValue ⭐⭐⭐⭐⭐

```text
useTransition
→ you control the state update
→ mark that update non-urgent

useDeferredValue
→ you already have a value
→ let expensive consumers lag behind
```

Lesson 42 covers `useDeferredValue`.

---

## 21. useTransition vs useMemo

```text
useTransition
→ scheduling / responsiveness

useMemo
→ cache a calculation result
```

One does not replace the other.

---

## 22. When to Use useTransition

Consider it when:

1. a state update causes expensive React rendering,
2. the update is non-urgent,
3. urgent interaction should remain responsive,
4. you control the update,
5. measurement or user experience shows a real problem.

---

## 23. Common Mistakes ⭐⭐⭐⭐⭐

1. Transitioning controlled input state.
2. Thinking Transition is a fixed delay.
3. Calling it debounce.
4. Expecting CPU-heavy JavaScript to move to another thread.
5. Wrapping every update in `startTransition`.
6. Ignoring the post-`await` Transition caveat.
7. Using it instead of fixing excessive work.
8. Treating `isPending` as generic network-loading state.
9. Assuming concurrent rendering means multithreading.
10. Depending on Transition timing for correctness.

---

## 24. Interview Questions ⭐⭐⭐⭐⭐

### What is useTransition?

A Hook for marking state updates as non-blocking Transition updates while exposing whether Transition work is pending.

### What does it return?

`[isPending, startTransition]`.

### What is a Transition?

A non-urgent React update whose rendering can be interrupted by more urgent work.

### Does it use a fixed delay?

No.

### Is it debounce?

No.

### Should controlled input state be transitioned?

Usually no. Keep the input value urgent and transition expensive downstream UI.

### Does concurrent rendering mean multiple JavaScript threads?

No.

### Does startTransition make heavy synchronous JavaScript non-blocking?

No.

### useTransition vs useDeferredValue?

Transition an update you control; defer a value you already have.

---

## 25. Complete Mental Model

```text
             User interaction
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
     urgent update      non-urgent update
          │                   │
          ↓             startTransition
 immediate render             │
          │                   ↓
          │            interruptible work
          └──── priority ─────┘
                    │
                    ↓
              responsive UI
```

---

## 26. Key Takeaways

- `useTransition` marks state updates as non-blocking.
- It returns `isPending` and `startTransition`.
- Transition rendering can be interrupted by urgent updates.
- Concurrent rendering is not JavaScript multithreading.
- Controlled input state should remain urgent.
- Transitions are not debounce or setTimeout.
- They do not make expensive synchronous JavaScript free.
- They work well with Suspense.
- React 19 supports async Transition Actions.
- Post-`await` updates may need another `startTransition`.
- Use state for business semantics, not Transition timing.
- Profile before adding concurrency optimizations.

---

## Next Lesson

➡️ [Lesson 42 — useDeferredValue](./42-usedeferredvalue.md)
