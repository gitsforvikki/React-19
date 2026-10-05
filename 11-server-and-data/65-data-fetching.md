# Lesson 65 — Data Fetching Patterns, Race Conditions and Cancellation ⭐⭐⭐⭐⭐

## 1. Why Data Fetching Is More Than Calling fetch()

This looks simple:

```jsx
const response =
  await fetch("/api/users");
```

But production React must answer:

- When should the request start?
- What happens when inputs change?
- What if an older request finishes after a newer one?
- What happens when the component unmounts?
- How should loading, error, empty and success states work?
- Can the request be cancelled?
- Should this data be fetched in an Effect at all?
- Who owns caching and deduplication?

The difficult part is usually not HTTP.

It is coordinating **asynchronous work with React rendering**.

---

# Part 1 — Choose the Right Fetching Layer

## 2. Three Common Places for Data Fetching ⭐⭐⭐⭐⭐

Conceptually:

```text
Need data
   │
   ├── server-rendering/data layer
   │      ↓
   │   fetch near server
   │
   ├── client data library/cache
   │      ↓
   │   shared client server-state
   │
   └── Effect
          ↓
      manual synchronization
```

Do not automatically reach for `useEffect`.

---

## 3. Server-Side Fetching

When an RSC-capable environment can fetch data before interactive client execution:

```jsx
async function DeveloperList() {
  const developers =
    await getDevelopers();

  return (
    <List
      developers={developers}
    />
  );
}
```

Potential benefits:

- less client fetching code
- server-side credentials stay server-side
- data access close to its source
- reduced client waterfalls
- integration with server rendering/streaming

Exact caching and routing behavior is framework-specific and belongs in the framework repository.

---

## 4. Client Data Libraries

For complex client-side server state, a dedicated data layer can provide:

- caching
- deduplication
- retries
- background refetching
- stale-data policies
- request sharing
- mutation coordination

Examples of the category include query/cache libraries and framework data APIs.

React itself does not require one specific library.

---

## 5. Effect-Based Fetching

Effects remain valid when a Client Component must synchronize with a remote system because of client-side state.

Example:

```jsx
function SearchResults({
  query,
}) {
  const [
    results,
    setResults,
  ] = useState([]);

  useEffect(() => {
    // synchronize with
    // remote search service
  }, [query]);
}
```

But once you manually fetch in an Effect, **you own the lifecycle**.

---

# Part 2 — Basic Effect Fetching

## 6. Loading, Error and Data State

```jsx
function UserProfile({
  userId,
}) {
  const [
    user,
    setUser,
  ] = useState(null);

  const [
    error,
    setError,
  ] = useState(null);

  const [
    loading,
    setLoading,
  ] = useState(false);

  useEffect(() => {
    // request logic
  }, [userId]);
}
```

Your UI may have four meaningful states:

```text
loading
error
empty
success
```

Do not collapse them accidentally.

---

## 7. Do Not Make the Effect Callback async ⭐⭐⭐⭐⭐

Avoid:

```jsx
useEffect(async () => {
  const response =
    await fetch(url);
}, [url]);
```

An Effect callback may return:

```text
nothing
or
cleanup function
```

An `async` function returns a Promise.

Instead:

```jsx
useEffect(() => {
  async function load() {
    const response =
      await fetch(url);
  }

  load();
}, [url]);
```

---

# Part 3 — The Race Condition

## 8. Classic Search Race ⭐⭐⭐⭐⭐

User types:

```text
r
re
rea
react
```

Requests start:

```text
request("r")
request("re")
request("rea")
request("react")
```

Network completion order is not guaranteed.

Possible result:

```text
react response ── finishes first
r response ────── finishes last
```

If every response calls `setResults`, the UI may finally show results for **r**, even though the current query is **react**.

That is a race condition.

---

## 9. Race Condition Mental Model

```text
Render A: query = "react"
   │
   └── Request A ───────────────┐
                                │
Render B: query = "react js"    │
   │                            │
   └── Request B ───────┐       │
                        ↓       │
                   B completes  │
                   UI = B       │
                                ↓
                           A completes
                           UI = A ❌
```

The problem is not that both requests existed.

The problem is that **obsolete work was allowed to commit state**.

---

# Part 4 — Ignore Stale Results

## 10. Cleanup Flag Pattern ⭐⭐⭐⭐⭐

React documentation demonstrates the stale-result protection pattern:

```jsx
useEffect(() => {
  let ignore = false;

  async function load() {
    const result =
      await fetchUser(userId);

    if (!ignore) {
      setUser(result);
    }
  }

  load();

  return () => {
    ignore = true;
  };
}, [userId]);
```

Flow:

```text
Effect A starts
     │
dependency changes
     ↓
cleanup A
ignore = true
     │
Effect B starts
     │
A eventually finishes
     ↓
A checks ignore
     ↓
does NOT update state
```

This solves stale state commits even if the underlying operation cannot be cancelled.

---

## 11. Why the Flag Works

Each Effect execution has its own closure:

```text
Effect A
→ own ignore variable

Effect B
→ different ignore variable
```

Cleanup for A mutates A's closure.

So when A eventually resolves, it knows that its result is obsolete.

This connects directly to Lesson 23 on closures.

---

# Part 5 — AbortController

## 12. Cancelling fetch ⭐⭐⭐⭐⭐

The browser's `AbortController` can cancel a compatible `fetch` request.

```jsx
useEffect(() => {
  const controller =
    new AbortController();

  async function load() {
    try {
      const response =
        await fetch(
          `/api/users/${userId}`,
          {
            signal:
              controller.signal,
          }
        );

      const data =
        await response.json();

      setUser(data);
    } catch (error) {
      if (
        error.name !==
        "AbortError"
      ) {
        setError(error);
      }
    }
  }

  load();

  return () => {
    controller.abort();
  };
}, [userId]);
```

Cleanup cancels the old fetch when:

- dependency changes
- component unmounts

---

## 13. AbortController Mental Model

```text
Effect starts
    ↓
AbortController
    ↓
fetch(signal)
    │
dependency changes
    ↓
cleanup
    ↓
controller.abort()
    ↓
fetch rejects with
abort-related error
```

Cancellation avoids unnecessary network/work where supported.

---

# Part 6 — Abort Is Not the Whole Race-Safety Story

## 14. Cancellation vs Stale-Result Protection ⭐⭐⭐⭐⭐

These solve related but different problems.

### Abort

```text
Stop compatible work
```

### Ignore/request identity

```text
Prevent obsolete work
from committing UI state
```

Not every asynchronous operation is cancellable.

Even after a fetch succeeds, additional asynchronous processing may happen.

Therefore robust code often reasons about **both**:

```text
cancel what can be cancelled
+
ignore what is no longer current
```

---

## 15. Combined Pattern

```jsx
useEffect(() => {
  const controller =
    new AbortController();

  let ignore = false;

  async function load() {
    try {
      const response =
        await fetch(url, {
          signal:
            controller.signal,
        });

      const data =
        await response.json();

      if (!ignore) {
        setData(data);
      }
    } catch (error) {
      if (
        !ignore &&
        error.name !==
          "AbortError"
      ) {
        setError(error);
      }
    }
  }

  load();

  return () => {
    ignore = true;
    controller.abort();
  };
}, [url]);
```

This communicates both intentions clearly.

---

# Part 7 — Request Identity

## 16. Latest Request ID Pattern ⭐⭐⭐⭐⭐

Another strategy is explicit request identity.

```jsx
const latestRequest =
  useRef(0);

async function search(query) {
  const requestId =
    ++latestRequest.current;

  const results =
    await fetchResults(query);

  if (
    requestId !==
    latestRequest.current
  ) {
    return;
  }

  setResults(results);
}
```

Mental model:

```text
request 1 starts
request 2 starts
request 3 starts

latest = 3

request 1 completes
1 !== 3 → ignore

request 3 completes
3 === 3 → commit
```

This is useful when the logic is event-driven or not naturally represented by one Effect cleanup.

---

# Part 8 — State Reset During New Requests

## 17. Avoid Showing Incorrect Old Data

When `userId` changes, should old data remain visible?

It depends on UX.

Option A:

```jsx
setUser(null);
setLoading(true);
```

shows loading instead of the previous user's profile.

Option B keeps stale data visible while refreshing.

Neither is universally correct.

The UI should intentionally distinguish:

```text
no data yet
vs
stale data being refreshed
```

---

## 18. Separate Initial Loading from Refreshing

A mature model might distinguish:

```text
status = "idle"
status = "loading"
status = "success"
status = "error"

isRefreshing = true/false
```

Why?

Initial loading:

```text
no useful data exists
```

Refreshing:

```text
useful old data exists
but newer data is loading
```

These can deserve different UI.

---

# Part 9 — Error Handling

## 19. fetch Does Not Reject for Every HTTP Error ⭐⭐⭐⭐⭐

A common mistake:

```jsx
const response =
  await fetch(url);

const data =
  await response.json();
```

HTTP 404/500 responses do not automatically behave like network rejection.

Check the response:

```jsx
if (!response.ok) {
  throw new Error(
    `Request failed: ${response.status}`
  );
}
```

Then parse the expected body.

---

## 20. Abort Is Usually Not a User Error

If a request was intentionally cancelled because the query changed:

```text
old request aborted
```

you normally should not show:

```text
"Something went wrong"
```

That cancellation is expected lifecycle behavior.

---

# Part 10 — Dependency Correctness

## 21. Fetch Dependencies Must Be Honest ⭐⭐⭐⭐⭐

```jsx
useEffect(() => {
  loadUser(userId);
}, []); // ❌
```

If the Effect uses `userId` reactively:

```jsx
useEffect(() => {
  // ...
}, [userId]);
```

Do not remove dependencies just to stop requests.

Fix the architecture instead.

---

## 22. Unstable Objects Can Refetch Repeatedly

```jsx
const options = {
  page,
  sort,
};

useEffect(() => {
  fetchData(options);
}, [options]);
```

A new object is created on every render.

Therefore:

```text
render
→ new options identity
→ dependency changed
→ Effect
→ state update
→ render
→ new options
→ ...
```

Prefer primitive dependencies or create the object inside the Effect where appropriate.

---

# Part 11 — Waterfalls

## 23. What Is a Fetch Waterfall? ⭐⭐⭐⭐⭐

```text
Parent renders
   ↓
Parent Effect fetches
   ↓
Parent receives data
   ↓
Child renders
   ↓
Child Effect fetches
```

The second request cannot even start until the first client round trip and render complete.

This is a waterfall.

---

## 24. Parallelize Independent Work

Bad:

```js
const user =
  await getUser();

const jobs =
  await getJobs();
```

if jobs does not depend on user.

Better:

```js
const userPromise =
  getUser();

const jobsPromise =
  getJobs();

const [
  user,
  jobs,
] = await Promise.all([
  userPromise,
  jobsPromise,
]);
```

Do not parallelize operations that truly depend on one another.

---

## 25. Why Effect-Only Fetching Can Cause Waterfalls

Effects run after rendering/commit.

Nested components may therefore discover their data needs gradually.

Server fetching, route loaders, cache libraries, or render-integrated Suspense architectures can often start work earlier.

This is one reason React documentation recommends considering framework or cache-based data fetching instead of writing every request manually in Effects.

---

# Part 12 — Caching and Deduplication

## 26. Manual Effect Fetching Does Not Automatically Cache

Two components:

```text
Component A
→ fetch /api/user/1

Component B
→ fetch /api/user/1
```

Manual Effects may produce two requests.

A shared data layer can provide:

```text
same resource key
      ↓
shared request/cache
      ↓
multiple consumers
```

This is an architectural concern beyond one component.

---

## 27. Server State Is Different from Local UI State ⭐⭐⭐⭐⭐

Server state:

- exists remotely
- may change outside this component
- can become stale
- may need refetching
- may be shared
- may require caching

Local UI state:

- selected tab
- modal visibility
- draft text
- local toggle

Do not model all remote server data as if it were ordinary permanent local state.

---

# Part 13 — Event Fetching vs Effect Fetching

## 28. User-Caused Request

Suppose the user clicks:

```text
Download Report
```

The request exists because of the click.

Use the event handler:

```jsx
async function handleDownload() {
  await downloadReport();
}
```

Do not create state merely so an Effect notices the click.

---

## 29. Synchronization Request

Suppose a component displays results for current `query`.

```text
query changes
→ displayed remote results
  must synchronize
```

An Effect can represent this synchronization when a better framework/cache layer is not being used.

This repeats Lesson 22's key rule:

> Event-caused logic belongs in event handlers; synchronization belongs in Effects.

---

# Part 14 — Debouncing

## 30. Debounce Does Not Replace Race Protection ⭐⭐⭐⭐⭐

Debouncing:

```text
wait before starting request
to reduce request frequency
```

Race protection:

```text
prevent obsolete request
from committing state
```

They solve different problems.

Even with a debounce, race conditions can still occur.

---

## 31. Debounced Search Example

```jsx
useEffect(() => {
  const timer =
    setTimeout(() => {
      // start request
    }, 300);

  return () => {
    clearTimeout(timer);
  };
}, [query]);
```

Then the actual request should still use appropriate stale-result/cancellation handling.

---

# Part 15 — Strict Mode

## 32. Why You May See Extra Requests in Development

Strict Mode can run an extra setup/cleanup cycle in development to expose broken Effect logic.

Correct code should tolerate:

```text
setup
cleanup
setup
```

Do not disable correctness mechanisms merely to hide duplicate development requests.

A cache/deduplication layer can also make repeated consumers more efficient.

---

# Part 16 — CodeBuddy Example

## 33. Developer Search

```text
query = "rea"
   ↓
request A

query = "react"
   ↓
cleanup A
abort A
request B

B completes
   ↓
request still current?
YES
   ↓
show results
```

This prevents a slower old search from replacing newer results.

---

# Part 17 — CareerLoop Example

## 34. Filtered Applications

Filters:

```text
status
location
search
page
```

If they drive a remote query:

```text
filters change
     ↓
old request becomes obsolete
     ↓
cancel/ignore old request
     ↓
fetch current filters
     ↓
commit only current result
```

If the framework/server layer can own the query, prefer that architecture instead of duplicating remote state unnecessarily in Effects.

---

# Part 18 — Common Mistakes

## 35. Common Mistakes ⭐⭐⭐⭐⭐

1. Making the Effect callback `async`.
2. Ignoring race conditions.
3. Assuming responses finish in request order.
4. Using AbortController as the only possible stale-result guarantee.
5. Showing aborts as user-facing failures.
6. Forgetting `response.ok`.
7. Omitting reactive dependencies.
8. Using unstable object/function dependencies.
9. Fetching user-event mutations indirectly through Effects.
10. Creating client waterfalls with nested Effects.
11. Assuming manual Effect fetching gives caching/deduplication.
12. Treating remote server state exactly like local UI state.
13. Debouncing without race protection.
14. Disabling Strict Mode to hide broken cleanup.
15. Keeping stale previous data without intentionally designing that UX.
16. Starting sequential requests that could run in parallel.
17. Fetching in Effects when the framework/data layer already provides a better mechanism.

---

# Part 19 — Decision Tree

## 36. How Should I Fetch? ⭐⭐⭐⭐⭐

```text
Need remote data?
     │
     ↓
Can server/framework data layer
fetch it naturally?
     │
   YES ─────→ prefer that
     │
     NO
     ↓
Is this shared/cacheable
client server-state?
     │
   YES ─────→ consider data/cache library
     │
     NO
     ↓
Is it synchronization caused
by component being active or
reactive inputs changing?
     │
   YES ─────→ Effect can be appropriate
     │
     NO
     ↓
Was it directly caused
by user action?
     │
   YES ─────→ event handler / Action
```

---

# Part 20 — Interview Questions

## 37. What Is a Data Fetch Race Condition? ⭐⭐⭐⭐⭐

When multiple requests are in flight and an older request completes later than a newer request, allowing stale data to overwrite the current UI.

---

## 38. How Do You Prevent It?

Common techniques:

- Effect cleanup with an ignore flag
- request identity/version checks
- AbortController where cancellation is supported
- data libraries that manage stale requests

---

## 39. AbortController vs Ignore Flag?

`AbortController` attempts to cancel compatible underlying work.

An ignore flag prevents obsolete results from committing React state.

They are related but not identical.

---

## 40. Why Not Make useEffect(async () => ...)?

Because an Effect callback's return value is reserved for cleanup, while an async function returns a Promise.

---

## 41. Why Check response.ok?

`fetch` does not reject simply because the server returned an HTTP error status such as 404 or 500.

---

## 42. Why Can Fetching in Effects Cause Waterfalls?

Effects start after rendering, so child data requirements may not begin until parent requests and renders complete.

---

## 43. When Should Data Fetching Be in an Event Handler?

When the request exists because of a specific user action such as submitting, downloading, deleting, or explicitly refreshing.

---

## 44. Does Debouncing Prevent Race Conditions?

No. It reduces how often requests start; it does not guarantee response ordering.

---

## 45. Why Consider a Data Library?

For shared server state, caching, deduplication, stale policies, retries, refetching and mutation coordination.

---

# Part 21 — Interview-Ready Answer

## 46. Interview Summary ⭐⭐⭐⭐⭐

```text
When I fetch data in a React Effect, I treat the
request as synchronization with an external system and
handle the complete lifecycle.

The biggest correctness issue is race conditions:
responses do not necessarily finish in the order they
were started. I prevent obsolete requests from updating
state using cleanup/request identity, and I also use
AbortController when the underlying fetch can be
cancelled.

I keep cancellation and stale-result protection as
separate concepts because not every async operation is
cancellable.

I also avoid Effect fetching when a server/framework
data layer or a client cache library is more suitable,
because manual Effects do not automatically provide
caching, deduplication or waterfall prevention.
```

---

# Part 22 — Complete Mental Model

## 47. Data Fetch Lifecycle ⭐⭐⭐⭐⭐

```text
               REACTIVE INPUT
                     │
                     ↓
                start request
                     │
          ┌──────────┴──────────┐
          │                     │
          ↓                     ↓
    input unchanged       input changes /
                          component unmounts
          │                     │
          │                     ↓
          │                  cleanup
          │               ┌─────┴─────┐
          │               ↓           ↓
          │            abort      mark stale
          │
          ↓
      response
          │
          ↓
   still current?
      ┌───┴───┐
      │       │
     yes      no
      │       │
      ↓       ↓
 commit UI   ignore


Architecture before Effect:
server/framework fetch?
client cache?
event handler?
Effect synchronization?
```

---

## 48. Key Takeaways

- Data fetching is an async lifecycle problem, not merely a `fetch()` call.
- Prefer the appropriate data layer before defaulting to Effects.
- Do not make an Effect callback async.
- Network responses can complete out of order.
- Stale results must not overwrite current UI.
- Cleanup flags use closure semantics to invalidate old Effect work.
- `AbortController` cancels compatible fetch work.
- Cancellation and stale-result protection are different guarantees.
- Request IDs are another useful latest-result strategy.
- Handle loading, error, empty, success and refreshing states intentionally.
- Check `response.ok` for HTTP failures.
- Effect dependencies must honestly represent reactive inputs.
- Avoid unstable dependencies that cause repeated fetching.
- Parallelize independent requests where appropriate.
- Manual Effect fetching does not automatically cache or deduplicate.
- Server state differs from local UI state.
- Put user-caused requests in event handlers/Actions.
- Debouncing reduces frequency but does not solve races.
- Correct Effect code should survive Strict Mode setup/cleanup checks.
- Consider framework/server fetching or client cache libraries for larger applications.

---

## Next Lesson

➡️ [Lesson 66 — Suspense-Based Async UI](./66-suspense-async-ui.md)
