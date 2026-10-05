# Lesson 52 — Error Boundaries ⭐⭐⭐⭐⭐

## 1. Why Error Boundaries Exist

JavaScript errors can occur while React is rendering UI.

Without an appropriate boundary, an error in one part of the component tree can prevent React from safely rendering that subtree and may cause a much larger UI failure.

An **Error Boundary** lets you define:

```text
"If this subtree throws,
show fallback UI instead."
```

---

## 2. Mental Model ⭐⭐⭐⭐⭐

```text
App
├── Header
├── ErrorBoundary
│   └── Dashboard
│       └── Chart ❌ throws
└── Footer
```

Instead of allowing the broken subtree to destroy the intended experience:

```text
ErrorBoundary catches
       ↓
renders fallback UI
```

The placement of boundaries determines the failure isolation area.

---

## 3. Traditional Error Boundary API

A class component becomes an Error Boundary by implementing one or both of:

```jsx
static getDerivedStateFromError(error)
```

and:

```jsx
componentDidCatch(
  error,
  info
)
```

Example:

```jsx
import { Component } from "react";

class ErrorBoundary extends Component {
  state = {
    hasError: false,
  };

  static getDerivedStateFromError() {
    return {
      hasError: true,
    };
  }

  componentDidCatch(
    error,
    info
  ) {
    console.error(
      error,
      info
    );
  }

  render() {
    if (this.state.hasError) {
      return (
        <h2>
          Something went wrong.
        </h2>
      );
    }

    return this.props.children;
  }
}
```

---

## 4. Why a Class Component? ⭐⭐⭐⭐⭐

In core React, there is still no direct function-component Hook equivalent that turns an arbitrary function component into an Error Boundary.

Production applications often use:

- a small class Error Boundary,
- a framework-provided error boundary mechanism,
- or a well-tested boundary library.

This is one of the remaining practical class-component concepts React developers should recognize.

---

## 5. getDerivedStateFromError

```jsx
static getDerivedStateFromError(
  error
) {
  return {
    hasError: true,
  };
}
```

Its role is to update state so the next render displays fallback UI.

Keep it focused on deriving fallback state.

---

## 6. componentDidCatch ⭐⭐⭐⭐⭐

```jsx
componentDidCatch(
  error,
  info
) {
  logError(
    error,
    info
  );
}
```

This is useful for reporting/logging errors.

For example:

```text
render error
   ↓
boundary catches
   ├── fallback UI
   └── report diagnostic information
```

Do not expose sensitive diagnostic information directly to end users.

---

## 7. What Error Boundaries Catch ⭐⭐⭐⭐⭐

They are designed for errors thrown by descendants during React rendering/lifecycle-related work that React routes through the boundary.

A practical mental model:

```text
child render/lifecycle error
→ nearest Error Boundary
→ fallback
```

---

## 8. What They Do Not Catch ⭐⭐⭐⭐⭐

Classic Error Boundaries do **not** act as universal JavaScript `try/catch`.

Important cases include errors from:

- event handlers
- arbitrary asynchronous callbacks such as `setTimeout`
- server-side rendering in the same client-boundary sense
- errors thrown by the boundary itself

Handle event/async operation failures using normal application error handling where appropriate.

---

## 9. Event Handler Errors

This is not what an Error Boundary is for:

```jsx
function handleClick() {
  try {
    riskyAction();
  } catch (error) {
    setError(error);
  }
}
```

Event handlers happen outside rendering.

Use explicit `try/catch`, Promise handling, state, notifications, or other appropriate error handling.

---

## 10. Async Request Errors

Suppose:

```jsx
async function handleSave() {
  try {
    await saveApplication();
  } catch (error) {
    setSaveError(error);
  }
}
```

This is an operation failure.

Model it as operation state:

```text
idle
saving
success
error
```

Do not expect an Error Boundary to replace normal request error handling.

---

## 11. Boundary Placement Is Architecture ⭐⭐⭐⭐⭐

One boundary around the entire application:

```text
App ErrorBoundary
└── everything
```

provides broad protection but poor isolation.

More granular boundaries:

```text
App
├── Header
├── ErrorBoundary
│   └── Feed
├── ErrorBoundary
│   └── Sidebar
└── Footer
```

allow one feature to fail while others remain usable.

Choose boundaries around meaningful product regions.

---

## 12. Do Not Wrap Every Component

Too granular:

```text
ErrorBoundary
└── Button

ErrorBoundary
└── Label

ErrorBoundary
└── Icon
```

creates noise and often poor UX.

Good boundaries usually align with:

- page/route regions
- widgets
- independently recoverable features
- risky third-party UI
- major data visualization areas

---

## 13. Fallback UI ⭐⭐⭐⭐⭐

A useful fallback should tell the user:

- what failed at an appropriate level,
- whether they can retry,
- whether other UI remains usable.

Example:

```jsx
function ErrorFallback() {
  return (
    <section>
      <h2>
        Unable to show analytics.
      </h2>

      <p>
        Try again in a moment.
      </p>
    </section>
  );
}
```

Avoid exposing stack traces to users.

---

## 14. Resetting an Error Boundary

Once a boundary enters its fallback state, you need an explicit recovery strategy.

Possible approaches:

- reset boundary state with a retry action,
- remount the boundary using a changed `key`,
- use a boundary library that supports reset behavior,
- let navigation remount the failed region.

The exact approach depends on what caused the error.

---

## 15. Retry Example

A custom class boundary could expose:

```jsx
handleRetry = () => {
  this.setState({
    hasError: false,
  });
};
```

Then fallback:

```jsx
<button
  onClick={
    this.handleRetry
  }
>
  Try again
</button>
```

But resetting only helps if the underlying problem may have changed.

If the child deterministically throws every render, retrying simply fails again.

---

## 16. Reset with key

Component identity can be used intentionally:

```jsx
<ErrorBoundary
  key={resourceId}
>
  <ResourceView
    id={resourceId}
  />
</ErrorBoundary>
```

When `resourceId` changes, the boundary can be remounted with fresh state.

Use keys semantically, not randomly.

---

## 17. Error Boundary Cannot Catch Its Own Error ⭐⭐⭐⭐⭐

If the fallback rendering itself throws, the same boundary cannot catch itself.

The error travels upward to the next boundary.

```text
OuterBoundary
└── InnerBoundary
    └── child throws
        ↓
Inner fallback throws
        ↓
OuterBoundary
```

Fallback UI should therefore be simple and reliable.

---

## 18. Error Boundaries and Suspense ⭐⭐⭐⭐⭐

They solve different problems.

```text
Suspense
→ "not ready yet"

Error Boundary
→ "rendering/loading failed"
```

A common conceptual structure:

```jsx
<ErrorBoundary>
  <Suspense
    fallback={
      <Loading />
    }
  >
    <Feature />
  </Suspense>
</ErrorBoundary>
```

Waiting and failure are separate UI states.

---

## 19. Lazy Loading Example

```jsx
<ErrorBoundary>
  <Suspense
    fallback={
      <p>Loading editor…</p>
    }
  >
    <LazyEditor />
  </Suspense>
</ErrorBoundary>
```

If the code is still loading:

```text
Suspense fallback
```

If loading/rendering fails in a way routed to the boundary:

```text
Error Boundary fallback
```

---

## 20. Error Boundaries Are Not Validation

Do not use them for expected input errors:

```text
email is invalid
password too short
required field missing
```

These are normal application states.

Use validation UI.

Error Boundaries are for exceptional failures in a React subtree.

---

## 21. Error Boundaries Are Not API Error UI

A `404`, `401`, or failed form submission may be an expected state that your component should render explicitly.

Do not throw everything merely so an Error Boundary can display it.

Distinguish:

```text
expected domain state
vs
unexpected rendering failure
```

---

## 22. Logging in Production

A production boundary often reports errors to an observability service.

Useful diagnostic context can include:

- error message
- stack
- component stack
- application version
- feature/page identifier

Be careful with user privacy and secrets.

Never intentionally log:

- passwords
- auth tokens
- payment secrets
- sensitive personal data

---

## 23. Development vs Production

Development may show detailed error overlays and debugging information.

Production should provide:

- stable fallback UI
- safe logging/reporting
- recovery where possible

Do not rely on the development overlay as production error handling.

---

## 24. React 19 Error Reporting APIs

Modern React root creation supports error-reporting callbacks such as:

- `onCaughtError`
- `onUncaughtError`
- `onRecoverableError`

These can support centralized reporting/observability.

They do **not** replace thoughtful Error Boundary placement and user-facing recovery UI.

---

## 25. CareerLoop Example

```text
ApplicationsPage
├── Search
├── ErrorBoundary
│   └── AnalyticsWidget
└── ApplicationList
```

If an analytics visualization unexpectedly throws, the application tracker can remain usable.

Fallback:

```text
"Analytics couldn't be displayed."
[Try again]
```

This is better isolation than replacing the entire application.

---

## 26. CodeBuddy Example

```text
ProfilePage
├── UserInfo
├── ErrorBoundary
│   └── RecommendationWidget
└── Connections
```

A failure in recommendations should not necessarily remove profile information and connections.

Boundary placement reflects product-level failure isolation.

---

## 27. Common Mistakes ⭐⭐⭐⭐⭐

1. Thinking Error Boundaries catch every JavaScript error.
2. Expecting them to catch event-handler errors.
3. Using them instead of normal async request handling.
4. Using one giant boundary for every feature.
5. Wrapping every tiny component.
6. Showing technical stack traces to users.
7. Making fallback UI complex enough to fail itself.
8. Retrying without fixing/changing the underlying cause.
9. Treating expected validation/domain errors as exceptional crashes.
10. Forgetting production logging and recovery design.

---

## 28. Interview Questions ⭐⭐⭐⭐⭐

### What is an Error Boundary?

A React component boundary that catches eligible errors in its descendant subtree and renders fallback UI instead.

### How do you create one in core React?

Traditionally with a class component implementing `getDerivedStateFromError` and/or `componentDidCatch`.

### Is there a direct useErrorBoundary Hook in core React?

No direct Hook turns an arbitrary function component into an Error Boundary.

### What does getDerivedStateFromError do?

Updates boundary state so fallback UI can render.

### What does componentDidCatch do?

It is commonly used for side effects such as error reporting/logging.

### Do Error Boundaries catch event-handler errors?

No. Handle those with normal error handling.

### Can a boundary catch an error in itself?

No. That error propagates to an ancestor boundary.

### Error Boundary vs Suspense?

Error Boundary handles failure; Suspense handles supported waiting/loading.

### Why use multiple boundaries?

To isolate failures so one feature can fail without taking down unrelated UI.

---

## 29. Complete Mental Model

```text
                App
                 │
        ┌────────┴─────────┐
        ↓                  ↓
 ErrorBoundary A      ErrorBoundary B
        │                  │
     Feature A           Feature B
        │                  │
        ❌
        │
        ↓
Boundary A catches
        │
        ↓
Feature A fallback

Feature B remains usable
```

---

## 30. Interview-Ready Summary ⭐⭐⭐⭐⭐

```text
An Error Boundary isolates unexpected failures in
a React subtree and renders fallback UI instead.

In core React, the traditional implementation uses
a class component with getDerivedStateFromError for
fallback state and componentDidCatch for reporting.

Boundaries do not act like universal try/catch:
event handlers and ordinary async operation errors
need their own handling.

I place boundaries around meaningful recoverable
regions rather than every component. Suspense handles
supported waiting states, while Error Boundaries
handle failures.
```

---

## 31. Key Takeaways

- Error Boundaries isolate failures in React subtrees.
- Core React's traditional boundary mechanism uses class lifecycle APIs.
- `getDerivedStateFromError` controls fallback state.
- `componentDidCatch` is useful for logging/reporting.
- Boundaries do not catch every possible JavaScript error.
- Event handlers and normal async failures need explicit handling.
- Boundary placement determines failure isolation.
- Keep fallback UI simple, safe, and recoverable.
- Expected validation/domain errors are not necessarily Error Boundary cases.
- Suspense and Error Boundaries solve waiting and failure respectively.
- React 19 root error callbacks can complement production observability.

---

## Next Lesson

➡️ [Lesson 53 — Suspense ⭐⭐⭐⭐⭐](./53-suspense.md)
