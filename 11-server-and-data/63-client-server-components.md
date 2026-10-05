# Lesson 63 — Client Components vs Server Components Concepts ⭐⭐⭐⭐⭐

## 1. Why This Topic Matters

Modern React applications can split a component tree across two environments:

```text
SERVER
  │
  │ rendered result / component data
  ↓
CLIENT
```

This gives React applications a way to combine:

- server-side data access
- smaller client JavaScript bundles
- interactive browser UI
- streaming and Suspense
- server-only dependencies
- client-side state and browser APIs

The important skill is not memorizing directives.

It is understanding **which work belongs in which environment**.

---

## 2. The Core Mental Model ⭐⭐⭐⭐⭐

Think of a React Server Components application as two connected worlds:

```text
┌──────────────────── SERVER ────────────────────┐
│                                               │
│ Server Components                             │
│                                               │
│ - access server resources                     │
│ - can be async                                │
│ - render before client execution              │
│ - code is not shipped as Server Component JS  │
│                                               │
└──────────────────────┬────────────────────────┘
                       │
             serialized RSC output
                       │
                       ↓
┌──────────────────── CLIENT ────────────────────┐
│                                               │
│ Client Components                             │
│                                               │
│ - state                                       │
│ - event handlers                              │
│ - Effects                                     │
│ - browser APIs                                │
│ - interactive UI                              │
│                                               │
└───────────────────────────────────────────────┘
```

The goal is:

> Keep work on the server when it does not need browser interactivity, and move only interactive boundaries to the client.

---

# Part 1 — Server Components

## 3. What Is a Server Component? ⭐⭐⭐⭐⭐

A Server Component is a component usage rendered in the React Server Components server environment.

Conceptually:

```jsx
async function DeveloperProfile({
  id,
}) {
  const developer =
    await getDeveloper(id);

  return (
    <section>
      <h1>
        {developer.name}
      </h1>
    </section>
  );
}
```

It can perform server-side work during rendering.

Depending on the framework/build setup, Server Components may run:

- during a build
- on a server for a request
- in another server-like environment supported by the framework

Do not reduce the concept to only:

```text
"runs once per HTTP request"
```

That is not always true.

---

## 4. There Is No "use server" Directive for Server Components ⭐⭐⭐⭐⭐

This is one of the most important interview points.

Incorrect:

```text
"use server"
→ marks a Server Component
```

Correct:

```text
Server Component
→ no Server Component directive

"use server"
→ marks Server Functions
```

A Server Component is determined by the RSC module/render environment, not by placing `"use server"` at the top of the component.

---

## 5. What Server Components Can Do

Server Components can access server-side resources such as:

```text
database
filesystem
server-only APIs
private services
backend libraries
environment-specific resources
```

Example:

```jsx
async function Applications() {
  const applications =
    await db.application
      .findMany();

  return (
    <ApplicationList
      applications={
        applications
      }
    />
  );
}
```

The database library does not need to become part of the browser bundle.

---

## 6. Async Server Components ⭐⭐⭐⭐⭐

Server Components can be async:

```jsx
async function Profile({
  id,
}) {
  const profile =
    await getProfile(id);

  return (
    <ProfileView
      profile={profile}
    />
  );
}
```

This makes server-side data access fit naturally into rendering.

```text
Server Component renders
        ↓
await server data
        ↓
produce component output
```

---

## 7. Server Components Reduce Client JavaScript

Suppose rendering Markdown needs a large parser:

```text
Without RSC
browser downloads parser
browser executes parser
browser renders HTML

With Server Component
server uses parser
server produces result
browser receives rendered result
```

The parser can remain server-only.

This can reduce:

- bundle size
- client parsing
- client execution work

---

## 8. Server Components Cannot Use Interactive Client State ⭐⭐⭐⭐⭐

Server Components do not persist in the browser as interactive component instances.

Therefore they cannot use ordinary interactive state such as:

```jsx
useState(...)
```

for browser interaction.

They also cannot directly define browser event handlers such as:

```jsx
<button
  onClick={() => {
    // ...
  }}
>
```

because the Server Component's function is not shipped to the browser as interactive component code.

---

## 9. Server Components and Hooks

Do not memorize:

```text
"Server Components cannot use any Hooks"
```

That statement is too broad.

They cannot use most client-oriented Hooks that depend on persistent client lifecycle/state, such as:

- `useState`
- `useEffect`
- `useLayoutEffect`
- `useReducer`
- most other client-state/lifecycle Hooks

Some React APIs have server-compatible use cases.

The important mental model is:

> Server Components do not have persistent browser state or browser lifecycle.

---

# Part 2 — Client Components

## 10. What Is a Client Component? ⭐⭐⭐⭐⭐

A Client Component is a component usage evaluated as part of the client module graph.

It can use browser-side React features:

```jsx
"use client";

import {
  useState,
} from "react";

export default function LikeButton() {
  const [
    liked,
    setLiked,
  ] = useState(false);

  return (
    <button
      onClick={() =>
        setLiked(
          current =>
            !current
        )
      }
    >
      {liked
        ? "Liked"
        : "Like"}
    </button>
  );
}
```

Client Components are where interactivity lives.

---

## 11. What Requires a Client Component? ⭐⭐⭐⭐⭐

Typical reasons:

### State

```jsx
useState(...)
useReducer(...)
```

### Effects

```jsx
useEffect(...)
useLayoutEffect(...)
```

### Event handlers

```jsx
onClick={...}
onChange={...}
onSubmit={...}
```

### Browser APIs

```js
window
document
localStorage
navigator
IntersectionObserver
```

### Interactive third-party libraries

Libraries using client Hooks or browser APIs need a client environment.

---

## 12. "use client" Defines a Boundary ⭐⭐⭐⭐⭐

```jsx
"use client";

import {
  useState,
} from "react";
```

The directive must appear at the beginning of the module before imports or other executable code.

It tells an RSC-compatible bundler:

```text
this module
+
its transitive client dependencies
        ↓
belong to client code
```

You do **not** need to add `"use client"` to every component file inside that client subtree.

---

## 13. The Client Boundary Can Grow

Suppose:

```text
SearchPanel.js
"use client"
    │
    ├── SearchInput.js
    ├── FilterButton.js
    └── formatters.js
```

If those modules are imported as dependencies of the client-marked module, they become part of the client module graph for that usage.

Therefore placing `"use client"` too high can unnecessarily move more code into the browser bundle.

---

## 14. A Component Definition Is Not Always Permanently "Server" or "Client" ⭐⭐⭐⭐⭐

This is a subtle React concept.

A component definition without its own `"use client"` directive may sometimes be used in a server context and sometimes be pulled into a client module graph.

Conceptually:

```text
FancyText definition
       │
       ├── used from server graph
       │      ↓
       │   Server Component usage
       │
       └── imported into client graph
              ↓
           Client Component usage
```

The environment of a component usage depends on the module/render graph.

Do not think only in terms of filename labels.

---

# Part 3 — Server vs Client

## 15. Comparison ⭐⭐⭐⭐⭐

| Concern | Server Component | Client Component |
|---|---|---|
| Server data access | ✅ | Through network/server interfaces |
| Async component | ✅ | Not as an async Client Component |
| `useState` | ❌ | ✅ |
| Event handlers | ❌ | ✅ |
| `useEffect` | ❌ | ✅ |
| Browser APIs | ❌ | ✅ |
| Server-only dependencies | ✅ | ❌ |
| Code required in client JS bundle | Server component implementation: no | Yes |
| Interactive UI | Compose Client Component | ✅ |

The goal is not:

```text
Server = good
Client = bad
```

The goal is:

```text
choose the correct environment
for each responsibility
```

---

# Part 4 — Composition

## 16. Server Component Can Render a Client Component ⭐⭐⭐⭐⭐

This is the normal pattern.

```jsx
// Server Component

import LikeButton
  from "./LikeButton";

async function Profile({
  id,
}) {
  const developer =
    await getDeveloper(id);

  return (
    <article>
      <h1>
        {developer.name}
      </h1>

      <LikeButton
        developerId={id}
      />
    </article>
  );
}
```

`LikeButton`:

```jsx
"use client";

function LikeButton({
  developerId,
}) {
  // interactive logic
}
```

Architecture:

```text
Server Profile
├── static/server-rendered info
└── Client LikeButton
      └── interactive island
```

---

## 17. Keep Client Boundaries Small ⭐⭐⭐⭐⭐

Bad architecture:

```text
Whole page
"use client"
    ↓
everything enters client graph
```

Better:

```text
Server page
├── server content
├── server content
└── small Client SearchBox
```

Push interactivity downward when practical.

This can reduce client JavaScript.

---

## 18. Server Content Can Be Passed Through Client Components ⭐⭐⭐⭐⭐

A useful composition pattern is:

```jsx
// Server Component

<ClientShell>
  <ServerContent />
</ClientShell>
```

The Client Component can receive already-produced React content through props such as `children`.

Conceptually:

```text
Server creates ServerContent
          │
          ↓
passes rendered component data
through ClientShell
          │
          ↓
client shell provides interaction
around it
```

This avoids forcing the server content's implementation into the client bundle.

---

## 19. Important Import Direction Mental Model

A Client Component should not directly import arbitrary Server Component implementation code as though it could execute that server code in the browser.

Instead, compose from the server side:

```text
Server Component
   │
   ├── ClientShell
   │       │
   │       └── children slot
   │
   └── ServerContent
           │
           └── passed into slot
```

Composition preserves the boundary.

---

# Part 5 — Passing Data Across the Boundary

## 20. Props Must Be Serializable ⭐⭐⭐⭐⭐

Data crossing from Server Components into Client Components must use React-supported serializable values.

Common examples include:

- strings
- numbers
- booleans
- null/undefined
- arrays
- plain serializable objects
- Date
- Map / Set
- supported typed arrays
- Promises
- React elements
- Server Function references

Do not oversimplify this to:

```text
"only JSON values"
```

React's supported serialization model is broader than JSON.

---

## 21. Ordinary Functions Cannot Simply Cross

Bad conceptual example:

```jsx
<ClientButton
  onClick={() => {
    deleteFromDatabase();
  }}
/>
```

A normal server closure cannot simply be serialized into browser JavaScript.

A callable server operation must use the supported **Server Function** mechanism.

---

## 22. Server Functions Are Special Serializable References

A Server Function can cross the boundary as a reference.

```text
Server Function
      ↓
serializable reference
      ↓
Client Component
      ↓
call reference
      ↓
network request
      ↓
function executes on server
```

This is different from shipping the function implementation to the browser.

---

# Part 6 — "use server"

## 23. What "use server" Actually Means ⭐⭐⭐⭐⭐

```jsx
async function updateProfile(
  data
) {
  "use server";

  // server mutation
}
```

This marks the async function as a **Server Function** callable through the RSC framework infrastructure.

It does not turn the containing component into a Server Component.

---

## 24. Server Function vs Server Component ⭐⭐⭐⭐⭐

```text
Server Component
────────────────
renders UI/component output
no "use server" directive required

Server Function
───────────────
callable server-side function
can be marked with "use server"
commonly performs mutations
```

Interviewers often test this exact distinction.

---

# Part 7 — RSC vs SSR

## 25. Server Components Are Not the Same as SSR ⭐⭐⭐⭐⭐

SSR means:

```text
React component tree
       ↓
server renders HTML
       ↓
browser receives HTML
       ↓
client JS hydrates interactive tree
```

RSC means:

```text
Server Components
       ↓
server component output/protocol
       ↓
combined with Client Components
       ↓
React reconstructs component tree
```

They solve different problems.

---

## 26. They Can Work Together

An RSC framework may:

1. render Server Components,
2. combine their output with Client Component references,
3. server-render the resulting application to HTML for the initial page,
4. send client JavaScript for interactive Client Components,
5. hydrate those interactive pieces.

Conceptually:

```text
RSC render
    ↓
React component result
    ↓
optional SSR
    ↓
HTML
    ↓
browser
    ↓
Client Component hydration
```

So:

```text
RSC ≠ SSR
but
RSC + SSR can cooperate
```

---

## 27. "Client Component" Does Not Mean "Only Rendered in Browser" ⭐⭐⭐⭐⭐

A common mistake:

```text
Client Component
=
no server rendering
```

Not necessarily.

A framework may use Client Components when producing initial server-rendered HTML and then hydrate them in the browser.

The important distinction is:

> Client Component code participates in the client module/runtime and can become interactive in the browser.

---

# Part 8 — Security

## 28. Server Code Does Not Automatically Make Data Safe ⭐⭐⭐⭐⭐

Server Components can access sensitive resources, but do not pass secrets into Client Components.

Bad:

```jsx
<ClientProfile
  databasePassword={
    process.env.DB_PASSWORD
  }
/>
```

Anything intentionally serialized across the client boundary should be considered visible to the client.

---

## 29. Server Functions Need Authorization

A hidden button is not security.

```text
Client calls Server Function
        ↓
server receives request
        ↓
authenticate
        ↓
authorize
        ↓
validate input
        ↓
perform mutation
```

Never trust Server Function arguments merely because the function was called from your React UI.

---

# Part 9 — Practical Architecture

## 30. CodeBuddy Example

```text
DeveloperProfile (Server)
├── fetch developer
├── fetch public profile
├── ProfileDetails (Server)
└── ConnectionButton (Client)
       ├── click handler
       ├── optimistic UI
       └── mutation Action
```

Why?

Profile data itself does not need browser state.

The connection button does.

---

## 31. CareerLoop Example

```text
ApplicationsPage (Server)
├── read applications
├── calculate initial summary
├── ApplicationList
└── Filters (Client)
       ├── interactive state
       └── browser events
```

Do not move the entire page to the client simply because one filter is interactive.

---

# Part 10 — Decision Guide

## 32. Server or Client? ⭐⭐⭐⭐⭐

Ask:

```text
Does it need:
- state?
- Effects?
- event handlers?
- browser APIs?

YES
→ Client Component

NO
↓
Can it benefit from:
- server data access?
- server-only libraries?
- less client JavaScript?

YES
→ Server Component is a strong candidate
```

---

## 33. Boundary Design Principle ⭐⭐⭐⭐⭐

A useful architecture rule:

```text
Server by default where possible
        ↓
move interactivity downward
        ↓
create focused client boundaries
```

But this is not a universal performance law.

Measure real applications and choose boundaries that keep code understandable.

---

# Part 11 — Common Mistakes

## 34. Common Mistakes ⭐⭐⭐⭐⭐

1. Thinking `"use server"` marks a Server Component.
2. Thinking every component needs either `"use server"` or `"use client"`.
3. Marking an entire page client-side because one button needs state.
4. Assuming Client Components cannot participate in initial server rendering.
5. Equating RSC with SSR.
6. Passing arbitrary non-serializable values across the boundary.
7. Treating supported RSC serialization as only JSON.
8. Trying to directly import server implementation code into a client module.
9. Using Server Components for browser event handlers.
10. Passing secrets to Client Components.
11. Assuming Server Functions are trusted because they run on the server.
12. Thinking a component definition without `"use client"` can never be evaluated in a client graph.

---

# Part 12 — Interview Questions

## 35. What Is a Server Component? ⭐⭐⭐⭐⭐

A component usage rendered in the React Server Components server environment. Its implementation does not need to be shipped as client-side component JavaScript, and it can access server-side resources.

---

## 36. What Is a Client Component?

A component usage that belongs to the client module graph and can use interactive React features, event handlers, and browser APIs.

---

## 37. Does "use server" Mark a Server Component?

No.

There is no Server Component directive.

`"use server"` marks Server Functions.

---

## 38. What Does "use client" Do?

It defines a client module boundary. The marked module and its transitive dependencies become client code for that graph.

---

## 39. Can a Server Component Render a Client Component?

Yes. This is a fundamental composition pattern.

---

## 40. Can a Client Component Receive Server-Rendered Content?

Yes. A Server Component can pass server-rendered React content to a Client Component through serializable props such as `children`.

---

## 41. Are Server Components the Same as SSR?

No.

RSC determines where component code is evaluated and how server/client component output is represented.

SSR produces initial HTML.

They can be used together.

---

## 42. Can Client Components Be Server-Rendered Initially?

Yes, depending on the framework. "Client Component" does not simply mean "browser-only HTML rendering."

---

## 43. Why Use Server Components?

Major reasons include:

- server-side data access
- server-only dependencies
- reduced client JavaScript
- avoiding some client data-fetch waterfalls
- composition with interactive Client Components

---

## 44. Why Use Client Components?

Use them when the UI requires:

- state
- effects
- event handlers
- browser APIs
- interactive libraries

---

# Part 13 — Interview-Ready Summary

## 45. Interview Answer ⭐⭐⭐⭐⭐

```text
React Server Components let part of the React tree
render in a server environment before client execution.
They can access server resources, can be async, and
their component implementation does not need to be
shipped to the browser.

Client Components belong to the client module graph and
are used for interactivity such as state, Effects,
event handlers, and browser APIs.

"use client" creates the client module boundary.
There is no directive that marks Server Components;
"use server" instead marks Server Functions.

I also keep RSC separate from SSR. RSC is about the
server/client component architecture and protocol,
while SSR is about generating initial HTML. A framework
can use both together.

My usual architecture is to keep server-capable work
on the server and introduce focused Client Components
only where interactivity is required.
```

---

## 46. Final Mental Model ⭐⭐⭐⭐⭐

```text
                    REACT TREE
                        │
          ┌─────────────┴─────────────┐
          ↓                           ↓
  SERVER COMPONENTS            CLIENT COMPONENTS
          │                           │
 server data                    state
 async render                   Effects
 server libraries               events
 less client JS                 browser APIs
          │                           │
          └──────────┬────────────────┘
                     ↓
             composed React UI


"use client"
→ client module boundary

"use server"
→ Server Function
→ NOT Server Component marker

RSC
→ component architecture/protocol

SSR
→ initial HTML generation
```

---

## 47. Key Takeaways

- Server and Client Components divide work by execution environment.
- Server Components can run at build time or in a server environment depending on the setup.
- Server Components can be async and access server resources.
- Their implementation does not need to become browser component JavaScript.
- Server Components cannot own interactive browser state or event handlers.
- Client Components provide state, Effects, events, and browser APIs.
- `"use client"` creates a client module boundary.
- Its transitive dependencies join the client graph for that usage.
- There is no `"use server"` directive for Server Components.
- `"use server"` marks Server Functions.
- Server Components can render Client Components.
- Server-rendered content can be passed through Client Components via composition.
- Values crossing the boundary must follow React's supported serialization rules.
- RSC and SSR are different concepts but can work together.
- Client Components may still participate in initial server rendering.
- Keep client boundaries focused when practical.
- Never serialize secrets to client code.
- Server Functions still require authentication, authorization, and validation.

---

## Next Lesson

➡️ [Lesson 64 — React Server Components Mental Model ⭐⭐⭐⭐⭐](./64-rsc-mental-model.md)
