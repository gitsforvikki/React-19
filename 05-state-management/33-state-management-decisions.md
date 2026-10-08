# Lesson 33 — When to Use Local State, Context or External State ⭐⭐⭐⭐⭐

## Choose Based on Who Needs the State

Start with the **smallest correct owner**, not a library.

| Situation | Use |
|---|---|
| One component needs it (modal, input) | `useState` |
| Nearby siblings need it | Lift state to common parent + props |
| Many deep descendants need the same value | Context |
| State updates have complex rules | `useReducer` |
| Complex state shared deeply in one feature | `useReducer` + Context |
| App-wide client state needs fine-grained selectors, subscriptions, or specialized tools | Consider Zustand / Redux Toolkit |
| Data comes from an API/database | Framework data fetching or server-state caching (e.g. TanStack Query) |
| Value should be shareable/bookmarkable | URL search parameters / router |

**Reducer and Context solve different problems:** reducer simplifies *updates*; Context simplifies *distribution*.

## Practical Examples

**Modal:** Keep `isOpen` in the component, or lift it to the nearest common parent of trigger and modal.

**Theme:** Context is useful when many distant components read it.

**Job applications:** Data fetched from the server needs a data-fetching/caching strategy. Temporary client filters may live in the page; put status/page filters in the URL if users should share or bookmark them.

**Complex application editor:** Use a local reducer; add Context only if deep descendants need access.

## Avoid Duplicate or Derived State

```jsx
const [applications, setApplications] = useState([]);
const [selectedId, setSelectedId] = useState(null);

// Derive this instead of keeping another synchronized state variable:
const selectedApplication = applications.find(app => app.id === selectedId);
```

One source of truth prevents unnecessary synchronization bugs.

## Common Mistakes

- Putting every dropdown, form input, and modal into a global store.
- Using Context to avoid a few clear props.
- Treating Context as a server cache or an authentication/authorization mechanism.
- Duplicating values in local state, Context, and URL without a clear owner.
- Choosing Redux/Zustand just because an app is large.

## Interview Quick Check

**What is state colocation?** Keeping state near the components that use it.

**When do you lift state?** When siblings need to coordinate using one source of truth.

**Context vs Redux/Zustand?** Context provides values through a tree; external stores offer independent store ownership and often selector-based subscriptions and additional tooling.

**Should everything be global?** No. Start local, lift when needed, and adopt specialized tools for concrete requirements.

---

## Section 5 Complete

- Lesson 30: Context distributes values.
- Lesson 31: `useReducer` centralizes transition logic.
- Lesson 32: Combine them for complex shared feature state.
- Lesson 33: Choose based on scope, data source, and update complexity.

➡️ [Lesson 34 — Custom Hooks ⭐⭐⭐⭐⭐](../06-reusable-logic/34-custom-hooks.md)
