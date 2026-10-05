# Lesson 66 — Suspense-Based Async UI

## 1. What Problem Does Suspense Solve?

Traditional async UI often manually coordinates:

```text
isLoading
data
error
```

Example:

```jsx
if (loading) {
  return <Spinner />;
}

if (error) {
  return <Error />;
}

return (
  <Content data={data} />
);
```

Suspense introduces a different model:

> A component can tell React that it is not ready to render yet, and React can show the nearest Suspense fallback.

```text
Component tries to render
        ↓
resource not ready
        ↓
component suspends
        ↓
nearest <Suspense>
shows fallback
        ↓
resource resolves
        ↓
React retries rendering
```

---

## 2. Suspense Boundary

```jsx
<Suspense
  fallback={
    <Loading />
  }
>
  <Comments />
</Suspense>
```

The boundary says:

```text
If something inside suspends,
temporarily show this fallback.
```

Suspense itself does not automatically fetch arbitrary data.

---

# Part 1 — What Can Activate Suspense?

## 3. Suspense-Aware Sources ⭐⭐⭐⭐⭐

Suspense can activate for supported sources such as:

- lazy-loaded component code with `lazy`
- cached Promises read with React's `use` API
- data sources integrated with Suspense-enabled frameworks/libraries

Do not assume:

```jsx
useEffect(() => {
  fetch(...);
}, []);
```

will automatically trigger a Suspense fallback.

It will not.

---

## 4. Effect Fetching vs Suspense Fetching ⭐⭐⭐⭐⭐

Effect model:

```text
render
  ↓
commit
  ↓
Effect starts request
  ↓
set loading state
  ↓
response
  ↓
set data
  ↓
render again
```

Suspense-aware model:

```text
render attempts to read resource
          ↓
resource pending
          ↓
suspend
          ↓
fallback
          ↓
resource resolves
          ↓
React retries render
```

These are different architectures.

---

# Part 2 — use() with a Promise

## 5. Reading a Promise with use() ⭐⭐⭐⭐⭐

From Lesson 59:

```jsx
import {
  use,
} from "react";

function Comments({
  commentsPromise,
}) {
  const comments =
    use(
      commentsPromise
    );

  return (
    <CommentList
      comments={comments}
    />
  );
}
```

If the Promise is pending:

```text
use(promise)
   ↓
component suspends
   ↓
nearest Suspense fallback
```

If it fulfills:

```text
React retries
   ↓
use(promise)
returns value
```

---

## 6. Rejected Promise

If the Promise rejects:

```text
Promise rejects
     ↓
use(promise)
     ↓
error propagates
     ↓
nearest Error Boundary
```

Therefore async UI often has two boundaries:

```jsx
<ErrorBoundary
  fallback={
    <ErrorMessage />
  }
>
  <Suspense
    fallback={
      <Loading />
    }
  >
    <Comments
      commentsPromise={
        commentsPromise
      }
    />
  </Suspense>
</ErrorBoundary>
```

Mental model:

```text
pending
→ Suspense

rejected
→ Error Boundary

fulfilled
→ content
```

---

# Part 3 — Promise Stability

## 7. Do Not Create a Fresh Promise on Every Client Render ⭐⭐⭐⭐⭐

Dangerous:

```jsx
function Comments() {
  const comments =
    use(
      fetch("/api/comments")
        .then(res =>
          res.json()
        )
    );

  // ...
}
```

A render can create a new Promise again.

This can lead to repeated suspension/request behavior and React warns about uncached Promises created inside Client Components.

Prefer Promises supplied by:

- a framework
- a Suspense-aware cache/library
- a server component
- another stable caching mechanism

---

## 8. Stable Promise Mental Model

Bad:

```text
render 1
→ Promise A

retry
→ Promise B

retry
→ Promise C
```

Better:

```text
stable resource key
      ↓
cached Promise A
      ↓
all retries read A
```

React needs the async resource to have stable identity across retries.

---

# Part 4 — Server Starts, Client Reads

## 9. Important RSC Pattern ⭐⭐⭐⭐⭐

Server Component:

```jsx
function Page() {
  const commentsPromise =
    getComments();

  return (
    <Suspense
      fallback={
        <CommentsSkeleton />
      }
    >
      <Comments
        commentsPromise={
          commentsPromise
        }
      />
    </Suspense>
  );
}
```

Client Component:

```jsx
"use client";

function Comments({
  commentsPromise,
}) {
  const comments =
    use(
      commentsPromise
    );

  return (
    <CommentList
      comments={comments}
    />
  );
}
```

The server starts the work.

The client component can read the Promise through `use`.

---

## 10. Why This Pattern Helps

Instead of:

```text
browser loads JS
      ↓
component hydrates
      ↓
Effect runs
      ↓
request starts
```

you can conceptually have:

```text
server render begins
      ↓
request starts early
      ↓
higher-priority UI continues
      ↓
Promise flows to consumer
      ↓
Suspense coordinates reveal
```

This can avoid some client-side fetch waterfalls.

---

# Part 5 — Streaming

## 11. Suspense Enables Progressive UI ⭐⭐⭐⭐⭐

Suppose:

```jsx
<Page>
  <Header />

  <Suspense
    fallback={
      <ProfileSkeleton />
    }
  >
    <Profile />
  </Suspense>

  <Suspense
    fallback={
      <FeedSkeleton />
    }
  >
    <Feed />
  </Suspense>
</Page>
```

The whole page does not necessarily need to wait for the slowest section.

Conceptually:

```text
Header ready ─────────────→ show

Profile pending
└── skeleton ─────────────→ show
    profile ready ────────→ replace

Feed pending
└── skeleton ─────────────→ show
    feed ready ───────────→ replace
```

An RSC/server-rendering environment can stream progressively available content.

---

## 12. One Giant Boundary vs Multiple Boundaries

One boundary:

```jsx
<Suspense
  fallback={<PageSkeleton />}
>
  <Profile />
  <Feed />
  <Suggestions />
</Suspense>
```

If one child suspends, the boundary may hide the entire group.

Multiple boundaries:

```text
Profile boundary
Feed boundary
Suggestions boundary
```

allow independent reveal.

But too many tiny boundaries can make UI noisy and architecture difficult.

Boundary placement is a UX decision.

---

# Part 6 — Nested Suspense

## 13. Progressive Reveal

```jsx
<Suspense
  fallback={<PageSkeleton />}
>
  <Profile />

  <Suspense
    fallback={
      <PostsSkeleton />
    }
  >
    <Posts />
  </Suspense>
</Suspense>
```

Mental model:

```text
Outer content
     ↓
Profile becomes available
     ↓
show Profile
     │
     └── Posts still pending
           ↓
       inner fallback
           ↓
       Posts ready
           ↓
       reveal Posts
```

Nested boundaries can match meaningful loading stages.

---

# Part 7 — Suspense Is Not a Spinner API

## 14. Think in Reveal Groups ⭐⭐⭐⭐⭐

Do not ask only:

```text
Where can I put a spinner?
```

Ask:

```text
Which pieces of the UI
should appear together?
```

Examples:

```text
Profile header
→ one meaningful group

Recommendations
→ can arrive later

Comments
→ can arrive independently
```

Suspense boundaries define **loading/reveal architecture**.

---

# Part 8 — Avoiding Bad Fallback UX

## 15. Fallback Flashing

If a visible subtree suspends again, immediately replacing useful content with a large spinner can feel disruptive.

```text
useful content
   ↓
small update
   ↓
whole screen spinner ❌
```

For navigation/update scenarios, React transitions can help keep already revealed content visible while new content prepares.

---

# Part 9 — Suspense + Transitions

## 16. startTransition and Suspense ⭐⭐⭐⭐⭐

Suppose changing a tab causes new content to suspend.

```jsx
startTransition(() => {
  setTab(nextTab);
});
```

A Transition tells React:

> This update is non-urgent. Avoid replacing already revealed content with a fallback when React can keep the current UI visible while preparing the next UI.

This can produce smoother navigation.

---

## 17. Urgent vs Transition Update

```text
Typing into input
→ urgent
→ update immediately

Switching heavy content
→ can be Transition
→ keep responsive/current UI
  while preparing next view
```

Do not put controlled input updates themselves inside a Transition.

---

## 18. Pending Indicator

```jsx
const [
  isPending,
  startTransition,
] = useTransition();

function selectTab(tab) {
  startTransition(() => {
    setTab(tab);
  });
}
```

Instead of replacing all content with a spinner, you can keep it visible and show:

```text
small pending indicator
opacity change
progress affordance
```

This often gives better continuity.

---

# Part 10 — Suspense + deferred values

## 19. useDeferredValue Pattern

Suppose search results suspend when query changes.

```jsx
const deferredQuery =
  useDeferredValue(query);

const isStale =
  query !== deferredQuery;
```

The input can update immediately while the result subtree uses the deferred value.

Conceptually:

```text
input
query = "react"
   │
   ├── input UI updates now
   │
   └── results still use old
       deferred query
           ↓
       new results prepare
           ↓
       deferred value catches up
```

This can preserve useful previous results instead of immediately showing a fallback.

---

# Part 11 — Suspense and Errors

## 20. Suspense Does Not Handle Errors ⭐⭐⭐⭐⭐

Suspense handles:

```text
not ready yet
```

Error Boundaries handle:

```text
rendering/resource failure
```

Architecture:

```text
ErrorBoundary
    │
    └── Suspense
           │
           └── AsyncContent
```

You need both when you want separate loading and failure UI.

---

## 21. Boundary Placement Changes Error Scope

```jsx
<ErrorBoundary>
  <Suspense>
    <Profile />
    <Feed />
  </Suspense>
</ErrorBoundary>
```

A failure may replace the entire group.

Separate boundaries can isolate failures:

```text
Profile error boundary
Feed error boundary
```

Again, boundary design is product/UX architecture.

---

# Part 12 — Suspense and Effects

## 22. Effects in Hidden/Suspended Trees

React may clean up layout Effects when already-visible content must be hidden because it suspends, then run them again when the content is revealed.

Why?

A layout Effect often assumes its DOM is visible and measurable.

Therefore Effects—especially layout Effects—must have correct cleanup.

This connects to Lesson 21.

---

# Part 13 — Data Fetching Is Not Automatically Suspense-Compatible

## 23. Important Warning ⭐⭐⭐⭐⭐

Do not invent a custom pattern like:

```js
if (!data) {
  throw fetch(url);
}
```

without a stable Suspense-aware cache.

Suspense integration requires careful Promise identity, caching, errors and retry behavior.

Prefer supported framework/library mechanisms or stable Promises provided from the server.

---

# Part 14 — Suspense and lazy()

## 24. Code Loading

Suspense is also used for code splitting:

```jsx
const AdminPanel =
  lazy(() =>
    import("./AdminPanel")
  );

<Suspense
  fallback={
    <LoadingPanel />
  }
>
  <AdminPanel />
</Suspense>
```

Here the suspended resource is the component module, not application data.

Same boundary model:

```text
resource unavailable
→ suspend
→ fallback
→ resource ready
→ retry/reveal
```

---

# Part 15 — Waterfalls

## 25. Suspense Does Not Automatically Remove All Waterfalls ⭐⭐⭐⭐⭐

Bad async architecture can still do:

```text
fetch profile
    ↓
wait
    ↓
discover posts request
    ↓
wait
    ↓
discover comments request
```

Suspense can coordinate loading UI, but request scheduling still matters.

Start independent work as early and concurrently as possible.

---

## 26. Parallel Suspense Work

Conceptually:

```text
start Profile ─────────┐
start Feed ────────────┼──→ resolve independently
start Suggestions ─────┘

UI boundaries reveal
as useful groups become ready
```

This is different from sequentially discovering every request.

---

# Part 16 — Loading UX Architecture

## 27. Skeleton vs Spinner

Skeletons are useful when:

- layout is predictable
- preserving visual structure matters

Spinners are useful when:

- operation is small
- layout is unknown
- generic waiting indicator is sufficient

Suspense does not dictate which fallback you must use.

---

## 28. Preserve Layout

Poor fallback:

```text
large content
→ tiny spinner
→ large content
```

This may cause layout shift.

Better:

```text
content-sized skeleton
→ real content
```

Design fallbacks to approximate the final UI structure where appropriate.

---

# Part 17 — CodeBuddy Example

## 29. Developer Profile

```text
Profile page
├── basic profile ───────── ready early
├── Connections
│    └── Suspense
│         └── skeleton
└── Recent activity
     └── Suspense
          └── skeleton
```

The user can begin reading the profile without waiting for every secondary section.

---

# Part 18 — CareerLoop Example

## 30. Dashboard

```text
CareerLoop Dashboard
├── header
├── summary cards
│    └── Suspense boundary
├── applications table
│    └── Suspense boundary
└── analytics
     └── Suspense boundary
```

The exact data APIs, routing and caching belong to the framework layer.

React's concern here is how async component output is coordinated and revealed.

---

# Part 19 — Common Mistakes

## 31. Common Mistakes ⭐⭐⭐⭐⭐

1. Thinking Suspense automatically fetches data.
2. Expecting an Effect-based fetch to trigger Suspense automatically.
3. Creating a new uncached Promise on every Client Component render.
4. Thinking Suspense handles rejected Promises as loading forever.
5. Forgetting an Error Boundary for failure UI.
6. Putting the whole page behind one giant fallback without considering UX.
7. Creating dozens of meaningless tiny boundaries.
8. Replacing useful visible UI with a disruptive spinner during non-urgent updates.
9. Forgetting transitions/deferred values for smoother updates.
10. Assuming Suspense eliminates data waterfalls automatically.
11. Throwing arbitrary fetch Promises without a stable cache architecture.
12. Forgetting cleanup for Effects in trees that can hide/reveal.
13. Mixing framework-specific caching rules into core React mental models.
14. Assuming every Promise source is automatically safe for `use`.
15. Treating Suspense only as a loading-spinner feature rather than reveal architecture.

---

# Part 20 — Decision Guide

## 32. Should This Use Suspense? ⭐⭐⭐⭐⭐

```text
Is the async resource integrated
with Suspense?
        │
      NO
        ↓
use supported fetching/lifecycle
architecture instead

      YES
        ↓
What UI should appear together?
        │
        ↓
place meaningful Suspense boundary
        │
        ↓
Need failure UI?
        │
      YES
        ↓
add Error Boundary
        │
        ↓
Could an update hide useful
existing content?
        │
      YES
        ↓
consider Transition or
deferred value
```

---

# Part 21 — Interview Questions

## 33. What Is Suspense? ⭐⭐⭐⭐⭐

A React mechanism that lets a component suspend rendering while a supported resource is unavailable, allowing the nearest Suspense boundary to show fallback UI until React can retry the render.

---

## 34. Does Suspense Fetch Data?

No.

Suspense coordinates rendering around supported asynchronous resources. The data source/framework/library is responsible for producing a Suspense-compatible resource.

---

## 35. Does useEffect Fetching Trigger Suspense?

Not automatically.

Effect fetching occurs after commit and normally manages its own loading state.

---

## 36. How Does use(Promise) Work with Suspense?

If the Promise is pending, the component suspends. If it fulfills, `use` returns its value. If it rejects, the error propagates to an Error Boundary.

---

## 37. Why Must Promises Be Stable?

React may retry rendering. Recreating an uncached Promise on each retry can repeatedly restart async work and suspension.

---

## 38. Suspense vs Error Boundary?

```text
Suspense
→ pending/not ready

Error Boundary
→ failure
```

They solve different states.

---

## 39. Why Use Multiple Suspense Boundaries?

To let meaningful UI sections load and reveal independently instead of blocking an entire page on the slowest resource.

---

## 40. How Do Transitions Help Suspense?

They mark non-urgent updates so React can often keep already revealed content visible while preparing the next suspended view.

---

## 41. How Does useDeferredValue Help?

It lets urgent UI such as an input use the newest value while a slower subtree temporarily renders with a deferred older value.

---

## 42. Does Suspense Prevent Waterfalls?

No. Request scheduling must still start independent work early and in parallel.

---

# Part 22 — Interview-Ready Answer

## 43. Interview Summary ⭐⭐⭐⭐⭐

```text
Suspense is React's mechanism for coordinating UI when
a supported resource is not ready during rendering.

When a component suspends, React shows the nearest
Suspense fallback and retries the render when the
resource becomes available. With use(Promise), a
pending Promise suspends, a fulfilled Promise provides
its value, and a rejected Promise is handled by an
Error Boundary.

Suspense does not automatically make useEffect-based
fetching Suspense-aware, and I avoid creating uncached
Promises during Client Component renders because React
may retry those renders.

I place Suspense boundaries around meaningful reveal
groups, use Error Boundaries for failures, and use
Transitions or deferred values when I want to preserve
already useful UI while new async content prepares.
Suspense coordinates loading UI, but good request
scheduling is still required to avoid waterfalls.
```

---

# Part 23 — Complete Mental Model

## 44. Async UI Architecture ⭐⭐⭐⭐⭐

```text
                 ASYNC RESOURCE
                       │
              component reads it
                       │
          ┌────────────┼────────────┐
          │            │            │
          ↓            ↓            ↓
       pending      fulfilled     rejected
          │            │            │
          ↓            ↓            ↓
       suspend        value        throw error
          │            │            │
          ↓            ↓            ↓
     <Suspense>      content   <ErrorBoundary>
       fallback                    fallback


Meaningful boundaries
        ↓
progressive reveal / streaming

Non-urgent update
        ↓
Transition / deferred value
        ↓
preserve useful existing UI

Still required:
stable resources
correct caching
early request scheduling
race/error handling
```

---

## 45. Key Takeaways

- Suspense coordinates rendering when supported async resources are not ready.
- Suspense does not automatically fetch data.
- Effect-based fetching does not automatically activate Suspense.
- `use(Promise)` integrates Promise state with rendering.
- Pending Promises suspend.
- Fulfilled Promises return values.
- Rejected Promises propagate to Error Boundaries.
- Promise identity must be stable across render retries.
- Avoid creating uncached Promises during Client Component render.
- Server-started Promises can be consumed by Client Components with `use` in supported RSC architectures.
- Suspense boundaries define meaningful reveal groups.
- Nested boundaries support progressive reveal.
- Server rendering can stream content around Suspense boundaries.
- Suspense is not merely a spinner API.
- Error Boundaries and Suspense handle different states.
- Transitions can prevent disruptive fallback replacement during non-urgent updates.
- `useDeferredValue` can keep old results visible while new results prepare.
- Suspense also supports lazy-loaded code.
- Suspense does not automatically remove request waterfalls.
- Start independent async work early and concurrently.
- Design fallbacks to preserve useful layout.
- Framework-specific routing, caching and revalidation remain outside this core React lesson.

---

# Section 11 — Server and Data Concepts Completed ✅

Completed lessons:

- Lesson 63 — Client Components vs Server Components Concepts ⭐⭐⭐⭐⭐
- Lesson 64 — React Server Components Mental Model ⭐⭐⭐⭐⭐
- Lesson 65 — Data Fetching Patterns, Race Conditions and Cancellation ⭐⭐⭐⭐⭐
- Lesson 66 — Suspense-Based Async UI

---

## Next Section — Production React

➡️ [Lesson 67 — Component Architecture and Folder Structure ⭐⭐⭐⭐⭐](../12-production-react/67-component-architecture.md)
