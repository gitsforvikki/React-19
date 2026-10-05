# Lesson 59 — use() API ⭐⭐⭐⭐⭐

## 1. What Is use()?

React 19 introduces the `use` API for reading the value of supported resources during rendering.

Syntax:

```jsx
import { use } from "react";

const value =
  use(resource);
```

Important resource types include:

- Promise
- Context

`use` is unusual because it behaves differently from ordinary Hooks.

---

## 2. Why use() Exists ⭐⭐⭐⭐⭐

React rendering sometimes depends on a resource that may not yet be ready.

Conceptually:

```text
render component
      ↓
read resource
      ↓
is it ready?
 ┌────┴─────┐
 yes        no
  ↓          ↓
value      suspend
             ↓
        Suspense fallback
```

`use` connects resource reading to React's rendering/Suspense model.

---

# Part 1 — Reading Promises

## 3. Basic Promise Example ⭐⭐⭐⭐⭐

```jsx
function Comments({
  commentsPromise,
}) {
  const comments =
    use(commentsPromise);

  return (
    <ul>
      {comments.map(
        comment => (
          <li key={comment.id}>
            {comment.text}
          </li>
        )
      )}
    </ul>
  );
}
```

Parent:

```jsx
<Suspense
  fallback={
    <p>Loading comments...</p>
  }
>
  <Comments
    commentsPromise={
      commentsPromise
    }
  />
</Suspense>
```

---

## 4. Pending Promise ⭐⭐⭐⭐⭐

When:

```js
use(promise)
```

receives a pending Promise:

```text
component cannot finish render
        ↓
component suspends
        ↓
nearest Suspense boundary
        ↓
fallback renders
        ↓
Promise resolves
        ↓
React retries component
        ↓
use returns resolved value
```

This is the key Promise mental model.

---

## 5. Resolved Promise

When the resource is available, `use` returns its value:

```jsx
const user =
  use(userPromise);

return (
  <h1>{user.name}</h1>
);
```

You do not write:

```jsx
await userPromise
```

inside an ordinary Client Component render.

`use` is the React resource-reading mechanism for this Suspense pattern.

---

## 6. Rejected Promise ⭐⭐⭐⭐⭐

If the Promise rejects:

```text
use(promise)
    ↓
Promise rejected
    ↓
error propagates
    ↓
nearest Error Boundary
```

A common structure:

```jsx
<ErrorBoundary>
  <Suspense
    fallback={<Loading />}
  >
    <Profile
      profilePromise={
        profilePromise
      }
    />
  </Suspense>
</ErrorBoundary>
```

States:

```text
pending
→ Suspense

fulfilled
→ Profile

rejected
→ Error Boundary
```

---

## 7. Do Not Wrap use(Promise) in try/catch ⭐⭐⭐⭐⭐

Do not use a normal local `try/catch` around `use(promise)` to model rejected Suspense resources.

Prefer React's Error Boundary model for rejected resources.

This keeps pending and failure handling aligned with React's rendering boundaries.

---

# Part 2 — Promise Stability and Caching

## 8. Do Not Create an Uncached Promise During Render ⭐⭐⭐⭐⭐

Bad:

```jsx
function Profile() {
  const profile =
    use(
      fetch("/api/profile")
        .then(res => res.json())
    );

  return (
    <h1>{profile.name}</h1>
  );
}
```

Why?

Each render creates a new Promise.

A render that suspends before first mount can be retried from scratch, producing another Promise.

This can repeatedly suspend and React can warn about an uncached Promise.

---

## 9. Stable Promise Mental Model

Prefer:

```text
create/cache Promise
       ↓
pass stable Promise
       ↓
use(stablePromise)
       ↓
Suspense coordinates rendering
```

The same logical resource should normally reuse the same Promise identity across relevant re-renders.

---

## 10. Where Should the Promise Come From?

Common sources:

- a Suspense-aware framework
- a data library/cache
- a Server Component passing a Promise
- a cache created outside the render retry path
- an event/loader that creates the Promise before it is read

Do not invent a production caching architecture casually.

Use the data layer/framework's supported pattern.

---

## 11. Server-to-Client Promise Pattern

Conceptually, a Server Component may create a Promise and pass it to a Client Component.

```text
Server Component
   │
   └── stable Promise
           │
           ↓
      Client Component
           │
       use(promise)
           │
           ↓
        Suspense
```

This can let a deeper interactive component define where the data is unwrapped/revealed.

Exact framework implementation belongs in the separate Next.js repository.

---

## 12. await vs use in Server Components ⭐⭐⭐⭐⭐

When fetching directly in a Server Component, React documentation generally recommends `async/await`.

Conceptually:

```jsx
async function ServerProfile() {
  const profile =
    await getProfile();

  return (
    <Profile
      profile={profile}
    />
  );
}
```

Use `use` when its resource-reading semantics are appropriate, especially for passing/reading resources at a chosen boundary.

Do not replace every server-side `await` with `use`.

---

# Part 3 — Reading Context

## 13. use(Context) ⭐⭐⭐⭐⭐

`use` can also read Context:

```jsx
const theme =
  use(ThemeContext);
```

Like `useContext`, React finds the nearest matching provider above the component.

---

## 14. Context Example

```jsx
const ThemeContext =
  createContext("light");

function Button() {
  const theme =
    use(ThemeContext);

  return (
    <button
      className={theme}
    >
      Save
    </button>
  );
}
```

Provider in React 19 can be:

```jsx
<ThemeContext value="dark">
  <Button />
</ThemeContext>
```

---

# Part 4 — use() vs Ordinary Hooks

## 15. use Can Be Called Conditionally ⭐⭐⭐⭐⭐

Ordinary Hooks:

```jsx
if (enabled) {
  useState(0); // ❌
}
```

But `use` has special rules:

```jsx
if (enabled) {
  const theme =
    use(ThemeContext); // ✅
}
```

This is a major interview distinction.

---

## 16. use Can Be Called in Loops

Conceptually:

```jsx
for (const context of contexts) {
  const value =
    use(context);

  // ...
}
```

Unlike ordinary Hooks, `use` can be called in loops.

However:

> It still must be called inside a React Component or custom Hook.

Do not call it from arbitrary utility functions.

---

## 17. Why Is use Different?

Ordinary Hooks rely heavily on stable call ordering to associate Hook state with a component.

`use(resource)` reads a resource rather than creating a normal positional state slot like `useState`.

React therefore gives it special calling semantics.

For interviews, remember the rule rather than overclaiming undocumented internals.

---

## 18. useContext vs use(Context) ⭐⭐⭐⭐⭐

### useContext

```jsx
const theme =
  useContext(ThemeContext);
```

Must follow normal Hook top-level calling rules.

### use

```jsx
if (needsTheme) {
  const theme =
    use(ThemeContext);
}
```

Can be conditional.

Both read the nearest matching Context value.

---

## 19. Do You Need to Replace useContext?

No.

Existing:

```jsx
const theme =
  useContext(ThemeContext);
```

is still valid.

Use `use(Context)` when its flexibility makes the code clearer.

Do not rewrite code without a reason.

---

# Part 5 — use() and Suspense

## 20. Suspense Relationship ⭐⭐⭐⭐⭐

`use(promise)` and Suspense work together:

```text
Component
   │
use(Promise)
   │
   ├── fulfilled → value
   │
   ├── pending → suspend
   │               ↓
   │           Suspense fallback
   │
   └── rejected → Error Boundary
```

This connects Lesson 53 directly to React 19.

---

## 21. Suspense Does Not Fetch for You

`Suspense` does not perform the network request.

`use` does not inherently decide where data comes from.

You still need:

```text
resource/data source
+
stable Promise semantics
+
Suspense boundary
```

React coordinates rendering around that resource.

---

## 22. useEffect Fetching vs use(Promise) ⭐⭐⭐⭐⭐

Effect model:

```text
render empty/loading state
       ↓
commit
       ↓
Effect runs
       ↓
fetch starts
       ↓
setState
       ↓
render data
```

Suspense resource model:

```text
render
  ↓
use(resource)
  ↓
pending?
  ↓
suspend
  ↓
boundary fallback
  ↓
resource resolves
  ↓
retry render
```

These are different architectures.

Do not wrap ordinary Effect fetching in Suspense and expect it to become Suspense-aware.

---

## 23. Avoid Fetch Waterfalls

If a request only starts when a deep component begins rendering:

```text
parent loads
   ↓
child renders
   ↓
child starts request
   ↓
grandchild later starts request
```

you can create a waterfall.

Preloading/creating resources earlier can improve parallelism.

Data-loading architecture matters as much as the `use` call itself.

---

# Part 6 — Error Handling

## 24. Error Boundary Example

```jsx
function Application({
  applicationPromise,
}) {
  const application =
    use(applicationPromise);

  return (
    <h2>
      {application.company}
    </h2>
  );
}
```

Boundary:

```jsx
<ErrorBoundary>
  <Suspense
    fallback={
      <ApplicationSkeleton />
    }
  >
    <Application
      applicationPromise={
        applicationPromise
      }
    />
  </Suspense>
</ErrorBoundary>
```

This cleanly separates:

```text
waiting
from
failure
```

---

## 25. Handling Known Promise Rejection Before use

If you intentionally want to transform a Promise rejection into a normal value, that transformation can happen in the resource layer before `use`.

Conceptually:

```js
const safePromise =
  originalPromise.catch(
    error => fallbackValue
  );
```

Then:

```jsx
const value =
  use(safePromise);
```

Use this only when the fallback value is genuinely part of your domain model.

Unexpected failures are often better handled by Error Boundaries.

---

# Part 7 — Conditional Resource Reading

## 26. Conditional Context Reading

```jsx
function Message({
  styled,
}) {
  if (!styled) {
    return <p>Hello</p>;
  }

  const theme =
    use(ThemeContext);

  return (
    <p className={theme}>
      Hello
    </p>
  );
}
```

This is legal with `use`.

It would not be legal to conditionally call `useContext` in the same way.

---

## 27. Conditional Promise Reading

```jsx
function Details({
  showDetails,
  detailsPromise,
}) {
  if (!showDetails) {
    return null;
  }

  const details =
    use(detailsPromise);

  return (
    <DetailsView
      details={details}
    />
  );
}
```

The component only reads/suspends on the Promise when the details are needed.

---

## 28. Do Not Inspect Promise Internals to Skip use ⭐⭐⭐⭐⭐

Avoid manually doing:

```jsx
if (
  promise.status ===
  "fulfilled"
) {
  return promise.value;
}

return use(promise);
```

Always let `use(promise)` read the resource.

React uses the resource read to coordinate Suspense behavior and tooling.

---

# Part 8 — Practical Examples

## 29. CareerLoop Example

Suppose a stable Promise for application analytics is prepared earlier:

```jsx
function Analytics({
  analyticsPromise,
}) {
  const analytics =
    use(analyticsPromise);

  return (
    <section>
      <p>
        Applied:
        {analytics.applied}
      </p>

      <p>
        Interviews:
        {analytics.interviews}
      </p>
    </section>
  );
}
```

Then:

```jsx
<Suspense
  fallback={
    <AnalyticsSkeleton />
  }
>
  <Analytics
    analyticsPromise={
      analyticsPromise
    }
  />
</Suspense>
```

The lesson is the resource/Suspense relationship—not a framework-specific fetching implementation.

---

## 30. CodeBuddy Example

A profile feature can receive a stable resource:

```jsx
function MutualConnections({
  connectionsPromise,
}) {
  const connections =
    use(
      connectionsPromise
    );

  return (
    <ConnectionList
      connections={
        connections
      }
    />
  );
}
```

UI states:

```text
pending
→ MutualConnections skeleton

resolved
→ list

rejected
→ Error Boundary
```

---

# Part 9 — Choosing use()

## 31. Decision Guide ⭐⭐⭐⭐⭐

```text
Need normal local state?
→ useState

Need side-effect synchronization?
→ useEffect

Need Context at normal top level?
→ useContext or use(Context)

Need Context conditionally?
→ use(Context)

Need to read a Suspense-aware Promise?
→ use(Promise)

Need to perform mutation?
→ Action-related APIs

Need async Server Component data directly?
→ usually async/await
```

Do not use one API for every async problem.

---

## 32. use() Is Not a General Replacement for await

`await` remains normal JavaScript.

`use` is specifically integrated with React rendering and supported resources.

Think:

```text
await
→ JavaScript async control flow

use(Promise)
→ React render-time resource reading
  + Suspense/Error Boundary integration
```

---

## 33. use() Is Not a Replacement for useEffect

`useEffect`:

```text
synchronize after commit
with external system
```

`use`:

```text
read supported resource
during render
```

Completely different responsibilities.

---

## 34. use() Is Not a Mutation API

Do not use `use` to perform:

- POST mutations
- deleting records
- sending messages
- changing application status

Those are Action/event/mutation workflows.

`use` reads resources.

---

## 35. Common Mistakes ⭐⭐⭐⭐⭐

1. Calling `use` outside a Component or Hook.
2. Assuming it follows all ordinary Hook calling restrictions.
3. Thinking `use` can only read Promises.
4. Creating a new uncached Promise on every render.
5. Calling `fetch()` directly inside `use` without stable caching.
6. Thinking Suspense itself performs fetching.
7. Using `try/catch` around `use(promise)` instead of an Error Boundary.
8. Replacing all server-side `await` with `use`.
9. Replacing every `useContext` call without a reason.
10. Reading Promise status/value manually instead of calling `use`.
11. Using `use` for mutations.
12. Ignoring waterfalls and resource-preloading architecture.

---

## 36. Interview Questions ⭐⭐⭐⭐⭐

### What is use() in React 19?

An API for reading supported resources such as Promises and Context during rendering.

### What happens when use receives a pending Promise?

The component suspends and the nearest Suspense boundary can render its fallback.

### What happens when the Promise rejects?

The error propagates to the nearest Error Boundary.

### Can use() be called conditionally?

Yes.

### Can use() be called in loops?

Yes.

### Can it be called from an arbitrary JavaScript function?

No. It must be called from a React Component or Hook.

### use(Context) vs useContext?

Both can read Context, but `use` can be called conditionally or in loops.

### Should you create fetch Promises directly during render?

No. Promises read by `use` should have stable/cached semantics.

### Does use() fetch data automatically?

No. It reads a resource supplied by your data/resource layer.

### use() vs useEffect?

`use` reads resources during render; `useEffect` synchronizes with external systems after commit.

### use() vs await?

`await` is JavaScript async control flow; `use(Promise)` integrates resource reading with React rendering/Suspense.

---

## 37. Interview-Ready Answer ⭐⭐⭐⭐⭐

```text
React 19's use API reads supported resources during
render, especially Promises and Context.

When use reads a pending Promise, the component
suspends and the nearest Suspense boundary displays its
fallback. If the Promise rejects, the error propagates
to an Error Boundary.

Unlike ordinary Hooks, use can be called inside
conditions and loops, although it still must be called
inside a React Component or Hook.

For Promises, stable resource identity is critical. I
would not create a new fetch Promise on every render;
I would use a framework, Suspense-aware cache, or a
stable Promise created before the render that reads it.

use is a resource-reading API, not a replacement for
Effects, Actions, or all JavaScript await expressions.
```

---

## 38. Complete Mental Model ⭐⭐⭐⭐⭐

```text
                     use(resource)
                          │
            ┌─────────────┴─────────────┐
            ↓                           ↓
         Promise                     Context
            │                           │
     ┌──────┼──────┐                    ↓
     ↓      ↓      ↓              nearest provider
 fulfilled pending rejected              │
     │      │      │                    ↓
     ↓      ↓      ↓                   value
   value  suspend  error
            │      │
            ↓      ↓
        Suspense  Error
        fallback  Boundary


Special calling rule:
use() may appear in conditions/loops,
but only inside Components/Hooks.
```

---

## 39. Key Takeaways

- `use` reads supported resources during render.
- It can read Promises and Context.
- A pending Promise suspends the component.
- Suspense handles the waiting UI.
- Rejected Promises propagate to Error Boundaries.
- Promise identity/caching is critical.
- Do not create a fresh fetch Promise on every render.
- Prefer framework/data-layer resource caching rather than ad hoc production caches.
- `use` may be called conditionally and in loops.
- It must still be called inside a Component or Hook.
- `use(Context)` is more flexible in placement than `useContext`.
- Existing `useContext` remains valid.
- In Server Components, direct async data fetching usually favors `async/await`.
- `use` is not a mutation API.
- `use` is not a replacement for `useEffect`.
- Suspense coordinates waiting; it does not perform the request itself.
- Avoid request waterfalls by designing resource creation/preloading carefully.

---

## Next Lesson

➡️ [Lesson 60 — useOptimistic ⭐⭐⭐⭐⭐](./60-useoptimistic.md)
