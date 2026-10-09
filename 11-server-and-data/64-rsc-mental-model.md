# Lesson 64 — React Server Components Mental Model ⭐⭐⭐⭐⭐

# Part 0 — Start Here: RSC, SSR, Client Components and Hydration

> **Learning order:** First distinguish five terms, then follow one example through three rendering situations. The later parts of this lesson explain the underlying architecture more deeply. These examples use **Next.js App Router** as an illustration; React RSC itself is not a routing or caching framework.

## A. Five Terms You Must Not Mix Up ⭐⭐⭐⭐⭐

| Term | What it means |
| --- | --- |
| **RSC** (React Server Components) | React architecture/protocol that runs Server Components outside the browser and transports their results |
| **Server Component** | Its component function runs in the server-side RSC environment; its implementation is not hydrated in the browser |
| **Client Component** | Belongs to the client module graph, supports state/events/browser APIs, and gets client JavaScript |
| **SSR** (server-side rendering) | Generating initial **HTML** on the server; even Client Components can participate |
| **Hydration** | React uses client JavaScript to attach interactive behavior to existing server-rendered HTML |

**The critical distinction:** *Where a component's implementation runs* is different from *where its initial HTML is produced*. A Client Component may execute during server pre-rendering **and** again in the browser for hydration. RSC is not the same as SSR.

## B. Why RSC Exists: Keep Server-Only Work off the Browser

Imagine displaying Markdown from a database. If a Client Component imports and runs a Markdown parser, that parser may become part of the browser bundle. A Server Component can parse Markdown on the server and send the rendered **result**, without shipping the parser's code.

\`\`\`text
Client-side parsing             Server Component
browser downloads parser        server runs parser
browser executes parser         server creates React result
browser renders result          browser displays server-produced UI
\`\`\`

RSC can reduce shipped JavaScript and put data access near the database. It does **not** guarantee faster performance in every application; data fetching, caching, latency, and bundle size still matter.

## C. The Same Example: Profile + LikeButton

\`\`\`tsx
// app/profile/page.tsx — Server Component in Next.js App Router
import LikeButton from "./LikeButton";
import { getDeveloper } from "@/lib/developers";

export default async function Profile() {
  const developer = await getDeveloper("42");

  return (
    <article>
      <h1>{developer.name}</h1>
      <p>{developer.bio}</p>
      <LikeButton developerId={developer.id} />
    </article>
  );
}
\`\`\`

\`\`\`tsx
// app/profile/LikeButton.tsx — Client Component
"use client";

import { useState } from "react";

export default function LikeButton({ developerId }: { developerId: string }) {
  const [liked, setLiked] = useState(false);

  return (
    <button
      aria-pressed={liked}
      onClick={() => setLiked((previous) => !previous)}
    >
      {liked ? "Liked ❤️" : "Like 🤍"}
    </button>
  );
}
\`\`\`

The id is available for a future server mutation; this demo changes **local browser state only**.

\`\`\`text
Profile (Server Component)
├── name and bio (server-produced content)
└── LikeButton ("use client" boundary)
    ├── useState
    └── onClick
\`\`\`

**Important:** A Server Component may import/render a Client Component without becoming a Client Component itself. The "use client" boundary begins in **LikeButton.tsx**.

## D. What the Three Pieces Actually Contain ⭐⭐⭐⭐⭐

For a normal **initial full-page request**, a Next.js App Router application can involve:

**1. HTML — what the browser displays immediately**

\`\`\`html
<article>
  <h1>Vikash</h1>
  <p>Full Stack Developer</p>
  <button>Like 🤍</button>
</article>
\`\`\`

**2. RSC Payload (React Flight) — React's transport representation**

Conceptually, it describes the rendered server result and a reference to where the Client Component belongs:

\`\`\`js
// ILLUSTRATION ONLY — NOT actual Flight wire format
{
  type: "article",
  children: [
    { type: "h1", children: "Vikash" },
    { type: "p", children: "Full Stack Developer" },
    {
      type: "ClientReference",
      module: "LikeButton",
      export: "default",
      props: { developerId: "42" }
    }
  ]
}
\`\`\`

The **reference** identifies a client module/export. The framework's build metadata associates it with browser JavaScript chunks. **The reference is not the source code of LikeButton.** The payload may also include serialized props and streaming/Suspense information. Actual React Flight is a specialized serialized stream, not the JSON object above.

**3. Client JavaScript — how interaction works**

The browser loads JavaScript containing the compiled Client Component logic (\`useState\`, \`onClick\`, and dependencies) plus the React/framework runtime as needed. The Server Component's database query implementation **does not** go into the client bundle.

HTML, RSC data and browser JavaScript are **conceptually distinct**, not necessarily three separate network responses: initial HTML may be streamed with embedded RSC data, while JavaScript files are typically loaded as separate resources.

**Why isn't HTML enough?** The markup displays a button, but by itself doesn't tell React which client module/state/props to use to hydrate that button or how to reconcile later server updates. HTML displays; the RSC Payload describes the React result and client boundaries; client JS implements behavior.

## E. Exactly What Happens on Initial Load

\`\`\`text
BROWSER: GET /profile
          │
          ▼
SERVER: execute Profile; read database
          │
          ▼
SERVER: create RSC payload (server result + LikeButton reference/props)
          │
          ▼
SERVER: pre-render initial HTML, including <button>Like 🤍</button>
          │
          ▼
BROWSER: display initial HTML
          │
          ▼
BROWSER: load Client Component JavaScript
          │
          ▼
BROWSER: hydrate LikeButton (state + click handler)
          │
          ▼
USER CLICKS: setLiked updates UI locally (no server call in this demo)
\`\`\`

Hydration normally **reuses existing HTML** rather than deleting and rebuilding the entire document. The browser does **not** re-run \`Profile()\` or its database query.

On subsequent **client-side navigation**, Next.js can fetch a **new RSC Payload** and reconcile the existing React tree without retrieving a complete new HTML document. Thus initial load and client-side navigation are different flows.

## F. Three Scenarios: Exactly What Is Sent? ⭐⭐⭐⭐⭐

### Scenario 1 — Only a Server Component

\`\`\`tsx
// app/page.tsx — no "use client"
export default function Page() {
  return (
    <main>
      <h1>Vikash</h1>
      <p>Full Stack Developer</p>
    </main>
  );
}
\`\`\`

On a normal initial request, Next.js typically produces:

- **HTML:** \`<main><h1>Vikash</h1><p>Full Stack Developer</p></main>\`
- **RSC Payload:** the server-produced React result, **without** a reference to a Client Component for this page
- **Server Component implementation in browser:** **No**
- **Hydration of Page:** **No**

The RSC Payload still matters for subsequent React/Next navigation and updates: it is **not only for Client Components**. Framework runtime JavaScript may still load; no *page-specific* Client Component code is needed here. A shared layout could independently contain Client Components.

\`\`\`text
Server executes Page → RSC result + initial HTML → browser displays HTML
\`\`\`

### Scenario 2 — Only a Client Component

\`\`\`tsx
// app/page.tsx
"use client";
import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount((n) => n + 1)}>Count: {count}</button>;
}
\`\`\`

On a normal initial full-page request with Next.js pre-rendering enabled:

- **HTML:** \`<button>Count: 0</button>\`, generated by server-side pre-rendering
- **RSC Payload:** a **Client Component reference** (with serialized props); it does **not** contain \`useState\` implementation
- **Client JavaScript:** bundled \`Counter\` code, React and relevant dependencies
- **Browser:** displays HTML, loads JS, then **hydrates** Counter
- **Later click:** \`setCount\` re-renders in the browser; it does not require another RSC request

\`\`\`text
server pre-renders Counter HTML
       ↓
HTML + RSC client reference + JS resource links/chunks
       ↓
browser displays HTML → loads JS → hydrates Counter → clicks work
\`\`\`

A Client Component is **not synonymous with browser-only rendering**. Some framework patterns deliberately disable server pre-rendering for particular components; then the server may send a fallback rather than their final HTML.

### Scenario 3 — Server Profile + Client LikeButton

The example from Section C:

- **HTML:** initial developer details **and** initial button markup
- **RSC Payload:** server-produced profile tree + client reference to LikeButton + \`developerId\` prop
- **Client JavaScript:** LikeButton and its client-side dependencies (not \`getDeveloper\`)
- **Hydration:** applies to LikeButton, **not** Profile's server implementation

\`\`\`text
Server Profile
├── server-produced name and bio (no Profile hydration)
└── client LikeButton (pre-rendered initially; hydrated in browser)
\`\`\`

| On a normal initial Next.js request | Server-only page | Client-only page | Mixed page |
| --- | --- | --- | --- |
| Initial HTML | Yes | Yes | Yes |
| RSC Payload | Yes | Yes | Yes |
| Client Component reference for these components | No | Yes | Yes |
| Page-specific Client Component JS | No | Yes | Yes |
| Hydration of these components | No | Yes | Client components only |
| Server Component implementation shipped to browser | No | Not applicable | No |

Assume no additional Client Components in layouts/providers. Even a server-only page may load framework/runtime JavaScript. **Server-rendered does not necessarily mean static:** Server Components may read dynamic/request-time data, or they may run at build time depending on the framework.

## G. Why a Server Component Can Render a Client Component

\`\`\`tsx
// Server Component — allowed
import LikeButton from "./LikeButton";
export default function Profile() {
  return <LikeButton developerId="42" />;
}
\`\`\`

A server module can refer to a client boundary. However, a Client Component **cannot directly import a module that requires server-only capabilities** (such as a direct database query) and expect it to run as a Server Component:

\`\`\`tsx
// Client Component — invalid if ServerProfile uses server-only APIs
"use client";
import ServerProfile from "./ServerProfile";
export default function Dashboard() {
  return <ServerProfile />;
}
\`\`\`

To nest server-rendered content visually **inside** a Client Component, compose it from a Server Component:

\`\`\`tsx
// app/page.tsx — Server Component
import Dashboard from "./Dashboard"; // Client Component
import ServerProfile from "./ServerProfile"; // Server Component

export default function Page() {
  return (
    <Dashboard>
      <ServerProfile />
    </Dashboard>
  );
}
\`\`\`

\`\`\`tsx
// Dashboard.tsx — Client Component
"use client";
import type { ReactNode } from "react";

export default function Dashboard({ children }: { children: ReactNode }) {
  return <section>{children}</section>;
}
\`\`\`

**Import graph ≠ visual JSX nesting.** The Client Component can receive server-rendered React elements as \`children\` without directly importing the server implementation.

Also, a plain component file with no \`"use client"\` directive is **not permanently server-only**: when imported by a client module, compatible code becomes part of the client graph. A component that uses server-only APIs cannot be imported there.

## H. Interview Summary and Check

1. **RSC** describes server-produced React UI and Client Component boundaries through the Flight transport.
2. **SSR** produces initial HTML, potentially including Client Components.
3. **Hydration** attaches React interaction to the pre-rendered Client Components.
4. **Pure server page:** normally HTML + RSC result; no page-specific client component to hydrate.
5. **Pure client page:** normally HTML + RSC client reference/props + browser JavaScript; browser hydrates.
6. **Mixed page:** Server Components run on server; Client Components can pre-render there initially and hydrate in browser.
7. **Client reference ≠ client source code**; it identifies the module so the appropriate JS can be loaded.
8. **A server import of a client component does not turn the parent into a client component.**

**Practice:** Explain why \`<button>Like</button>\` in HTML is visible before hydration but its React \`onClick\` does not work until browser JavaScript loads.

**Further reading:** [React Server Components](https://react.dev/reference/rsc/server-components) · [Next.js Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)

---

## 1. Why RSC Feels Difficult

React Server Components are confusing when we try to fit them into only the old model:

```text
server returns HTML
+
browser runs React
```

RSC adds another representation between server-side React execution and client-side React.

A better mental model is:

```text
Server Component execution
        ↓
RSC representation
        ↓
React combines it with
Client Component references
        ↓
UI
```

This lesson focuses on that architecture.

It does **not** teach Next.js routing, caching, revalidation, or framework-specific implementation.

---

# Part 1 — The Fundamental Idea

## 2. Server Components Are Executed Before Client Bundling ⭐⭐⭐⭐⭐

Server Components execute in an environment separate from the client application.

Their source implementation is not sent to the browser as Server Component JavaScript.

Conceptually:

```jsx
async function Article() {
  const markdown =
    await readFile(...);

  const html =
    parseMarkdown(
      markdown
    );

  return (
    <article>
      ...
    </article>
  );
}
```

The browser does not need the server-only Markdown parser merely to display the rendered result.

---

## 3. Think "Component Output", Not Just HTML ⭐⭐⭐⭐⭐

A Server Component does not simply return an HTML string.

It returns React elements/component output.

That output may include:

- host elements such as `div`
- text
- nested Server Component results
- Client Component references
- serializable props
- Suspense-related information

Conceptually:

```text
Server Component tree
        ↓
React evaluates server pieces
        ↓
RSC representation
        ↓
contains:
- rendered server output
- data
- references to client modules
```

This is why RSC is more than traditional HTML templating.

---

# Part 2 — RSC Payload / Representation

## 4. What Is the RSC Payload? ⭐⭐⭐⭐⭐

At a conceptual level, React serializes Server Component rendering into a transport representation understood by React.

You can think of it as:

```text
RSC Payload
───────────
serialized React tree information
+
Server Component output
+
Client Component references
+
serializable data
+
Suspense/streaming information
```

It is **not simply HTML**.

The exact wire format is an implementation detail and should not be hard-coded into application logic.

---

## 5. Why Client Component References Are Needed

Suppose the server renders:

```jsx
<DeveloperProfile>
  <ConnectButton
    developerId="42"
  />
</DeveloperProfile>
```

`ConnectButton` needs browser JavaScript.

The server representation therefore needs to communicate conceptually:

```text
At this position:
render/use this Client Component
with these serializable props.
```

The client bundle contains the actual interactive implementation.

---

## 6. The Browser Does Not Download Server Component Implementation ⭐⭐⭐⭐⭐

Imagine:

```text
Server component imports:
- database library
- markdown parser
- server formatter
- filesystem utility
```

If these remain entirely in the server graph:

```text
browser receives
→ rendered result / RSC representation

browser does NOT need
→ those server implementation libraries
```

This is a major RSC benefit.

---

# Part 3 — RSC and HTML

## 7. RSC Payload Is Not Initial HTML ⭐⭐⭐⭐⭐

Keep these separate:

```text
RSC representation
→ describes React component result

HTML
→ browser document markup
```

A framework can additionally use SSR to turn the initial combined React tree into HTML.

---

## 8. Initial Request Mental Model

A framework using both RSC and SSR may conceptually do:

```text
Request
   ↓
Render Server Components
   ↓
RSC representation
   ↓
combine with Client Component references
   ↓
SSR
   ↓
initial HTML
   ↓
browser displays page
   ↓
client bundles load
   ↓
Client Components hydrate
```

Implementation details vary by framework.

The conceptual separation is what matters.

---

## 9. Later RSC Updates

After the initial page, an RSC-capable framework may request new Server Component output without replacing the whole document.

Conceptually:

```text
existing client tree
      │
request server tree update
      ↓
new RSC payload
      ↓
React merges/reconciles
new server result
with existing client tree
```

This allows server-centric rendering without reverting to full-page MPA navigation for every update.

---

# Part 4 — State Preservation

## 10. Why Client State Can Survive Server Updates ⭐⭐⭐⭐⭐

Suppose:

```text
Server Page
└── Client SearchPanel
      └── local state
```

When new server output arrives, React can reconcile it with the existing tree.

If the Client Component keeps the same identity/position/key, its local state can be preserved.

This connects directly to:

- Lesson 18 — component identity
- Lesson 46 — reconciliation
- Lesson 49 — keys

RSC does not replace React's identity rules.

---

## 11. When Client State Can Reset

State can still reset if identity changes.

Examples:

- different component type
- changed key
- removed/re-added subtree
- framework navigation semantics that intentionally replace a subtree

Mental model:

```text
RSC update
+
reconciliation
+
component identity
=
preserve or reset client state
```

---

# Part 5 — Client Boundaries

## 12. "use client" Is a Module Boundary ⭐⭐⭐⭐⭐

```jsx
"use client";

import SearchInput
  from "./SearchInput";
```

This creates a client module graph boundary.

Conceptually:

```text
Server graph
   │
   ├── server modules
   │
   └── "use client" boundary
             │
             ↓
        client module graph
             ├── component
             └── dependencies
```

The boundary is about bundling/evaluation, not merely visual placement.

---

## 13. One Directive Can Pull Many Modules Client-Side

```text
Dashboard.js
"use client"
    │
    ├── Chart.js
    ├── Table.js
    ├── utils.js
    └── date-library
```

Those imports can become client dependencies.

This is why architecture should place client boundaries deliberately.

---

## 14. Composition Can Avoid Expanding the Client Bundle ⭐⭐⭐⭐⭐

Suppose an interactive shell needs server-rendered content.

Instead of importing server implementation into the client shell:

```text
ClientShell
  imports ServerContent ❌
```

compose on the server:

```jsx
<ClientShell>
  <ServerContent />
</ClientShell>
```

Conceptually:

```text
Server
├── creates ServerContent
└── passes result as children
      ↓
   ClientShell
```

The client shell does not need the ServerContent implementation.

---

# Part 6 — Serialization Boundary

## 15. Crossing Environments Requires Serialization ⭐⭐⭐⭐⭐

Server and client do not share the same JavaScript memory.

Therefore:

```text
Server value
    ↓
serialization protocol
    ↓
Client value/reference
```

This boundary is fundamental.

---

## 16. Supported Values Are Broader Than JSON

Common supported values include:

- primitives
- arrays
- plain objects
- Date
- Map
- Set
- typed arrays / ArrayBuffer
- globally registered symbols
- Promises
- React elements
- Server Function references

Unsupported examples include ordinary arbitrary functions and most custom class instances.

Always check current React/framework support for edge cases.

---

## 17. Why Ordinary Closures Cannot Cross

Server closure:

```js
const secret =
  "server-only";

function handler() {
  console.log(secret);
}
```

You cannot simply serialize that executable closure into the browser while preserving its server environment.

Instead:

```text
ordinary server function
→ cannot cross as normal prop

Server Function reference
→ supported special reference
```

---

# Part 7 — Server Functions

## 18. Server Function Mental Model ⭐⭐⭐⭐⭐

```jsx
async function updateApplication(
  id,
  status
) {
  "use server";

  // authorize
  // validate
  // mutate database
}
```

The client does not receive this implementation.

It receives a reference understood by React/framework infrastructure.

```text
Client
  │
call reference
  ↓
network boundary
  ↓
Server
  │
execute function
  ↓
serialized result
  ↓
Client
```

---

## 19. Server Function vs Server Action

Current React terminology distinguishes them.

```text
Server Function
→ function callable on server
  from client through RSC infrastructure

Server Action
→ a Server Function used in
  an Action context
```

Historically, "Server Action" was often used for all Server Functions.

For precise modern terminology, use **Server Function** as the broader term.

---

## 20. Server Functions Are Mainly for Mutations ⭐⭐⭐⭐⭐

Use Server Functions for operations such as:

- create record
- update profile
- delete item
- submit form
- change status

They are not the recommended general mechanism for data fetching.

Server Components themselves can fetch/read server data during render.

```text
Read data
→ Server Component rendering

Mutate server state
→ Server Function / Action
```

This separation is useful for architecture.

---

# Part 8 — Async Rendering and Suspense

## 21. Async Server Components ⭐⭐⭐⭐⭐

```jsx
async function Application({
  id,
}) {
  const application =
    await getApplication(id);

  return (
    <ApplicationDetails
      application={
        application
      }
    />
  );
}
```

React can suspend server rendering while the awaited data is unavailable.

This allows async data access to participate directly in the component tree.

---

## 22. Start Promise on Server, Read on Client ⭐⭐⭐⭐⭐

React supports an important pattern:

```jsx
// conceptual Server Component

async function Page() {
  const profile =
    await getProfile();

  const commentsPromise =
    getComments();

  return (
    <>
      <Profile
        profile={profile}
      />

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
    </>
  );
}
```

Client:

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

Flow:

```text
server starts comments Promise
          ↓
does not block higher-priority UI
          ↓
Promise crosses supported boundary
          ↓
Client Component use(promise)
          ↓
Suspense fallback if pending
```

This connects Lessons 53 and 59 with RSC.

---

## 23. Avoid Accidental Waterfalls ⭐⭐⭐⭐⭐

Bad sequence:

```text
fetch A
  ↓ await
fetch B
  ↓ await
fetch C
```

when B and C did not actually depend on A.

Better:

```text
start A ──────┐
start B ──────┼→ await when needed
start C ──────┘
```

RSC makes server-side fetching convenient, but poor async structure can still create waterfalls.

---

# Part 9 — Server Components Without a Runtime Web Server

## 24. "Server" Does Not Always Mean Per-Request Server ⭐⭐⭐⭐⭐

A Server Component may execute at build time.

Example concept:

```text
CI/build machine
     ↓
read Markdown/files/CMS
     ↓
render Server Components
     ↓
static output
     ↓
CDN
```

Therefore:

> Server Component means it runs in the server-side RSC environment, not necessarily that every render requires a long-running Node server.

---

# Part 10 — RSC and Hydration

## 25. What Gets Hydrated? ⭐⭐⭐⭐⭐

Server Component implementation itself does not hydrate as an interactive browser component.

Interactive Client Components do.

Conceptually:

```text
Server Component
→ rendered result
→ no Server Component implementation hydration

Client Component
→ initial rendered UI may come from server
→ client JS loads
→ hydration attaches interactivity
```

This distinction is critical.

---

## 26. Hydration Is Not "Re-running All Server Components in Browser"

The browser does not download the server-only component implementation and rerun database/filesystem logic.

It uses:

- initial HTML where applicable
- RSC representation
- Client Component bundles

to reconstruct and activate the application.

---

# Part 11 — Performance Mental Model

## 27. What RSC Can Improve

Potential benefits:

### Less JavaScript

Server-only component/dependency code stays off the client.

### Data locality

Server Components can read data close to its source.

### Fewer client fetch waterfalls

Some data can be fetched before the browser starts interactive rendering.

### Streaming

Suspense boundaries can progressively reveal UI.

---

## 28. RSC Is Not Automatically Faster ⭐⭐⭐⭐⭐

Performance can still be poor because of:

- slow database queries
- sequential server waterfalls
- oversized serialized data
- too many client boundaries
- large Client Component bundles
- unnecessary re-fetching
- poor caching strategy
- excessive network latency

RSC provides architecture primitives, not automatic performance.

---

## 29. Client Boundary Cost

Moving a component into the client graph may require shipping:

```text
component code
+
transitive imports
+
supporting libraries
```

So ask:

```text
Does this code actually need
browser interactivity?
```

But do not split components so aggressively that architecture becomes unreadable.

---

# Part 12 — Security Mental Model

## 30. Serialization Is a Trust Boundary ⭐⭐⭐⭐⭐

Before passing server data to a Client Component, ask:

```text
Am I comfortable with the browser
receiving this value?
```

Never pass:

- passwords
- private keys
- database credentials
- unrestricted internal records
- secrets from environment variables

The client can inspect data it receives.

---

## 31. Server Function Calls Are Requests

Think of a Server Function call like an endpoint invocation:

```text
untrusted client input
       ↓
Server Function
       ↓
authentication
       ↓
authorization
       ↓
validation
       ↓
mutation
```

Do not trust arguments merely because React serialized them.

---

# Part 13 — CodeBuddy Architecture

## 32. Example Mental Model

```text
SERVER
DeveloperPage
├── getDeveloper()
├── getConnectionSummary()
├── ProfileInfo
└── passes data/output
          │
          ↓
CLIENT
ConnectionControls
├── useOptimistic
├── click handlers
└── calls Server Function/API
```

The expensive server/data code stays server-side.

Only interaction logic becomes client JavaScript.

---

# Part 14 — CareerLoop Architecture

## 33. Example Mental Model

```text
SERVER
ApplicationsPage
├── read applications
├── read summary
├── render table rows
└── interactive boundaries
       │
       ├── StatusSelector (Client)
       └── SearchControls (Client)
```

A status mutation may call a Server Function:

```text
StatusSelector
      ↓
Action
      ↓
Server Function reference
      ↓
database mutation
      ↓
new authoritative server state
```

Exact framework revalidation/caching behavior is intentionally outside this React lesson.

---

# Part 15 — Common Misconceptions

## 34. Misconception: RSC Means SSR

Wrong.

```text
RSC
→ server/client component architecture

SSR
→ HTML generation
```

They can cooperate.

---

## 35. Misconception: Server Components Need "use server"

Wrong.

`"use server"` marks Server Functions.

---

## 36. Misconception: Client Component Means Browser-Only Initial Rendering

Wrong.

A framework may include Client Components in server-generated initial HTML and hydrate them later.

---

## 37. Misconception: Server Components Hydrate

The Server Component implementation itself does not hydrate in the browser.

Client Components are the interactive/hydrated pieces.

---

## 38. Misconception: RSC Payload Is HTML

Wrong.

The RSC representation carries React tree information, data, and client references.

HTML is a separate representation.

---

## 39. Misconception: Everything Must Be Serializable JSON

Too simplistic.

React's RSC serialization supports more types than JSON, including certain built-ins, Promises, React elements, and Server Function references.

---

## 40. Misconception: Server Functions Are Secure Automatically

Wrong.

They still receive untrusted requests/input and need authentication, authorization, and validation.

---

# Part 16 — Debugging Mental Model

## 41. When Something Fails, Ask Where It Runs ⭐⭐⭐⭐⭐

Before debugging, classify the code:

```text
Where is this module evaluated?

SERVER?
→ server APIs available
→ browser APIs unavailable

CLIENT?
→ browser APIs available
→ server-only modules unavailable
```

Many RSC errors are boundary errors rather than ordinary React rendering bugs.

---

## 42. Debugging Checklist

Ask:

1. Is this module in the server or client graph?
2. Did `"use client"` pull this dependency into the browser graph?
3. Am I using state/effects/events in server code?
4. Am I importing a server-only dependency into client code?
5. Is the value crossing the boundary serializable?
6. Am I passing a normal function instead of a Server Function?
7. Did component identity change and reset client state?
8. Did sequential data fetching create a waterfall?
9. Am I exposing server-sensitive data to the client?

---

# Part 17 — Interview Questions

## 43. What Is the RSC Payload? ⭐⭐⭐⭐⭐

A React transport representation containing Server Component rendering output, serializable data, Client Component references, and information React needs to reconstruct/update the component tree. It is not simply HTML.

---

## 44. Does the Browser Receive Server Component Source Code?

Not as the Server Component implementation needed to execute that server logic. Server-only component code and dependencies can remain outside the client bundle.

---

## 45. RSC vs SSR?

RSC defines a server/client component architecture and transport model.

SSR converts a React tree into initial HTML.

A framework can combine them.

---

## 46. What Hydrates?

Interactive Client Components.

Server Component implementation itself does not hydrate in the browser.

---

## 47. Why Can Client State Survive an RSC Update?

React reconciles new server output with the existing tree. If a Client Component preserves its identity, its state can remain.

---

## 48. Why Must Boundary Values Be Serializable?

Server and browser run in different JavaScript environments and do not share memory, so values must cross through React's transport protocol.

---

## 49. What Is a Server Function?

An async function exposed through RSC infrastructure so client code can invoke server-side execution through a serialized reference/network call.

---

## 50. Server Function vs Server Action?

Server Function is the broader concept. A Server Function used in an Action context is a Server Action.

---

## 51. Should Server Functions Be Used for General Data Fetching?

React recommends them primarily for mutations. Server Components can read/fetch server data during rendering.

---

## 52. Can Server Components Run Without a Long-Running Web Server?

Yes. They can run at build time in supported architectures.

---

# Part 18 — Interview-Ready Summary

## 53. Interview Answer ⭐⭐⭐⭐⭐

```text
React Server Components execute part of the React tree
in a server-side RSC environment before client
execution. Their implementation and server-only
dependencies do not need to be shipped as client
JavaScript.

Their output is represented through the RSC transport,
which is not simply HTML. It can describe rendered
server output, serializable data, Client Component
references, Promises, and Server Function references.

Frameworks can combine RSC with SSR: RSC determines the
server/client component architecture, while SSR can
turn the initial combined tree into HTML. Client
Components then load their JavaScript and hydrate for
interactivity.

"use client" defines a client module boundary.
There is no Server Component directive. "use server"
marks Server Functions, which are callable server-side
operations and should generally be used for mutations.

I treat the server/client serialization boundary as
both a performance boundary and a security boundary.
```

---

# Part 19 — Complete RSC Mental Model

## 54. Architecture Diagram ⭐⭐⭐⭐⭐

```text
                     DATA / SERVER RESOURCES
                              │
                              ↓
                    SERVER COMPONENT TREE
                              │
                 async render / await data
                              │
                              ↓
                    RSC REPRESENTATION
                  ┌───────────┼────────────┐
                  │           │            │
                  ↓           ↓            ↓
             server output   data    client references
                  │           │            │
                  └───────────┼────────────┘
                              ↓
                    COMBINED REACT TREE
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ↓                   ↓
              optional SSR        RSC transport
                    │                   │
                    ↓                   │
              initial HTML              │
                    │                   │
                    └─────────┬─────────┘
                              ↓
                           BROWSER
                              │
                     Client JS bundles
                              │
                              ↓
                  hydrate Client Components
                              │
                              ↓
                      INTERACTIVE APP


Later server update:
Client/framework request
        ↓
new RSC representation
        ↓
React reconciliation
        ↓
preserve compatible client state
```

---

## 55. Key Takeaways ⭐⭐⭐⭐⭐

- RSC is an architecture for splitting React execution across server and client.
- Server Components execute before client-side component execution/bundling.
- Their implementation can stay out of the client JavaScript bundle.
- Server Component output is not merely an HTML string.
- The RSC representation carries React tree information and client references.
- RSC payload and HTML are different representations.
- RSC and SSR solve different problems but can work together.
- Client Components provide browser interactivity.
- Client Component code can be hydrated after initial server rendering.
- `"use client"` creates a client module boundary.
- Composition lets server output flow through interactive Client Components without importing server implementation into the client graph.
- Server/client values cross a serialization boundary.
- React supports more serializable types than plain JSON.
- Server Functions cross as special references, not shipped implementations.
- `"use server"` marks Server Functions, not Server Components.
- Server Functions are mainly mutation-oriented.
- Async Server Components can directly await data.
- A Promise may be started on the server and read with `use` in a Client Component.
- Poor async design can still create waterfalls.
- Server Components may run at build time; "server" does not always mean per-request Node server.
- Client state can survive RSC updates when component identity is preserved.
- RSC can reduce client JavaScript, but it does not automatically make an application fast.
- Treat the serialization boundary as a security boundary.
- Framework-specific routing, caching, revalidation, and deployment remain separate concerns.

---

## Next Lesson

➡️ [Lesson 65 — Data Fetching Patterns, Race Conditions and Cancellation ⭐⭐⭐⭐⭐](./65-data-fetching.md)
