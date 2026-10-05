# Lesson 72 — Testing React Components

## 1. Why Test React Components?

A useful test gives confidence that important behavior still works after code changes.

The best question is not:

```text
Did my implementation run exactly
the way I wrote it?
```

It is:

> Does the feature behave correctly from the user's perspective?

Testing should make refactoring safer, not make implementation impossible to change.

---

# Part 1 — Testing Philosophy

## 2. Test Behavior Over Implementation Details ⭐⭐⭐⭐⭐

Suppose a button changes a message.

Weak test idea:

```text
inspect component state
verify setState called
```

Better:

```text
user clicks button
→ expected message appears
```

The user does not care which Hook or internal variable produced the result.

---

## 3. Confidence vs Quantity

More tests do not automatically mean better quality.

A thousand fragile implementation tests can provide less confidence than a smaller set covering critical user flows.

Prioritize:

```text
important behavior
+
realistic interaction
+
failure cases
```

---

# Part 2 — Types of Tests

## 4. Unit Tests

Test a small isolated unit.

Excellent candidates:

```js
calculateTotal()
filterApplications()
formatSalary()
validateEmail()
```

Pure functions are easy and fast to test.

---

## 5. Component Tests

Render a component and interact with it.

Example behavior:

```text
render form
→ enter values
→ click submit
→ validation/message appears
```

This verifies React UI behavior.

---

## 6. Integration Tests

Test multiple units working together.

Example:

```text
ApplicationForm
+
validation
+
mocked network boundary
+
success UI
```

These often provide high value because real bugs occur between components/modules.

---

## 7. End-to-End Tests

Exercise a real application flow in a browser-like environment:

```text
open app
→ sign in
→ add application
→ edit status
→ verify saved result
```

They provide broad confidence but are typically slower and more expensive to maintain.

---

# Part 3 — Practical Test Pyramid

## 8. Balanced Strategy

A useful mental model:

```text
             E2E
           /     \
       Integration
      /             \
 Unit / Component tests
```

Do not interpret this as a strict percentage rule.

Choose test levels based on risk and value.

---

# Part 4 — React Testing Library Philosophy

## 9. Query Like a User

With Testing Library-style tools, prefer queries that reflect accessible UI.

Example:

```js
screen.getByRole(
  "button",
  {
    name: /save/i,
  }
);
```

This is usually better than:

```js
container.querySelector(
  ".save-btn"
);
```

The first query verifies meaningful accessible behavior.

---

## 10. Query Priority

Common preference:

```text
role + accessible name
label text
placeholder when appropriate
visible text
test id as fallback
```

A test ID is useful when no semantic query fits, but should not be the default for ordinary accessible controls.

---

# Part 5 — Example Component

## 11. Counter

```jsx
function Counter() {
  const [
    count,
    setCount,
  ] = useState(0);

  return (
    <>
      <p>
        Count: {count}
      </p>

      <button
        onClick={() =>
          setCount(c => c + 1)
        }
      >
        Increment
      </button>
    </>
  );
}
```

Behavior test concept:

```js
render(<Counter />);

await user.click(
  screen.getByRole(
    "button",
    {
      name: /increment/i,
    }
  )
);

expect(
  screen.getByText(
    "Count: 1"
  )
).toBeVisible();
```

The test does not inspect `useState`.

---

# Part 6 — userEvent vs Low-Level Events

## 12. Prefer Realistic Interaction

High-level user-event tools simulate interactions closer to how users operate the interface.

Conceptually:

```js
await user.type(
  emailInput,
  "user@example.com"
);

await user.click(
  submitButton
);
```

This is generally more meaningful than manually dispatching isolated low-level events for ordinary interactions.

---

# Part 7 — Forms

## 13. Test the User Flow

Example:

```text
render signup form
→ submit invalid email
→ error appears
→ type valid email
→ submit
→ pending state
→ success
```

This provides more confidence than asserting internal validation function calls from the component.

Pure validation functions can separately have unit tests.

---

# Part 8 — Async UI

## 14. Async Queries

If UI appears later:

```js
const message =
  await screen.findByText(
    /saved successfully/i
  );
```

Conceptually:

```text
getBy...
→ expected now

findBy...
→ expected asynchronously
```

Use the API appropriate to the expected timing.

---

## 15. Avoid Arbitrary Sleep

Bad:

```js
await new Promise(
  resolve =>
    setTimeout(
      resolve,
      2000
    )
);
```

Tests become slow and flaky.

Wait for the observable condition instead.

---

# Part 9 — Loading, Error and Empty States

## 16. Test Production States

From Lesson 69:

```text
loading
success
empty
error
refreshing
mutation pending
mutation failure
```

Example scenarios:

```text
API pending
→ skeleton visible

API returns []
→ empty state visible

API rejects
→ retry UI visible
```

Do not test only successful data.

---

# Part 10 — Mock Boundaries, Not Everything

## 17. Over-Mocking Is Dangerous

If you mock:

- every child
- every Hook
- every helper
- every function

the test may only verify your mocks.

Prefer testing a meaningful slice and mocking true external boundaries where appropriate:

```text
network
browser API
time
third-party service
```

---

# Part 11 — Network Mocking

## 18. Test Through the Network Contract

Instead of mocking the component's internal custom Hook implementation, a network-level mock can let more real application code run:

```text
component
→ data layer
→ mocked HTTP boundary
→ response
→ UI
```

This makes refactoring internal modules less likely to break tests.

Exact tooling is project-dependent.

---

# Part 12 — Avoid Testing Framework Internals

## 19. Do Not Test React

Avoid tests whose only purpose is proving:

```text
useState works
useEffect runs
React renders props
```

React already has its own tests.

Test your application's behavior.

---

# Part 13 — Props

## 20. Test Meaningful Variants

For:

```jsx
<ApplicationStatus
  status="interview"
/>
```

test important states:

```text
applied
interview
offer
rejected
```

especially when behavior or semantics differ.

---

# Part 14 — Accessibility and Tests

## 21. Accessible Queries Improve Tests

If this works:

```js
getByRole(
  "button",
  {
    name:
      "Delete application",
  }
)
```

your component likely exposes useful semantics.

Testing by role/name encourages accessible markup.

It does not replace dedicated accessibility testing, but the concerns reinforce one another.

---

# Part 15 — Context and Providers

## 22. Render With Required Providers

If many tests need the same providers, create a test utility:

```jsx
function renderWithProviders(
  ui
) {
  return render(
    <ThemeProvider>
      <AuthProvider>
        {ui}
      </AuthProvider>
    </ThemeProvider>
  );
}
```

Only include providers genuinely required by the test.

Do not create enormous test wrappers that hide dependencies.

---

# Part 16 — Custom Hooks

## 23. Test Hook Behavior Through Components When Practical

If a custom Hook primarily supports visible component behavior, testing the feature through the component can provide stronger confidence.

Test the Hook directly when its reusable behavior is complex enough to deserve an isolated contract.

Do not automatically write isolated tests for every Hook.

---

# Part 17 — Effects

## 24. Test Observable Effect Results

Suppose an Effect subscribes to a chat connection.

Useful tests may verify:

```text
correct room connected
room changes
→ old connection cleaned
→ new connection established
unmount
→ cleanup
```

You care about the external synchronization contract, not the number of internal Effect calls in development.

---

# Part 18 — Strict Mode

## 25. Do Not Depend on Exact Effect Counts

Development Strict Mode can intentionally exercise setup/cleanup behavior.

A fragile test:

```text
expect(effect).toRunExactlyOnce()
```

may test an implementation assumption rather than the real contract.

Test whether resources are correctly established and cleaned up.

---

# Part 19 — Timers

## 26. Timer-Based UI

For debounce or delayed UI, controlled/fake timers can make tests deterministic.

Example concept:

```text
type query
→ advance debounce timer
→ request begins
```

Use timer mocking carefully and restore it after tests.

---

# Part 20 — Error Boundaries

## 27. Test Failure UI

If a component throws:

```text
ErrorBoundary
→ fallback visible
```

Production resilience should be testable.

Suppress expected test-environment logging only in a controlled way when necessary; do not hide unrelated failures.

---

# Part 21 — Optimistic UI

## 28. Test Both Outcomes

```text
click Connect
→ optimistic Connected UI
→ server success
→ remains Connected
```

and:

```text
click Connect
→ optimistic Connected UI
→ server failure
→ UI reconciles
→ error feedback
```

Testing only optimistic success misses the difficult path.

---

# Part 22 — Race Conditions

## 29. Test Out-of-Order Responses

From Lesson 65:

```text
request A starts
request B starts
B resolves
A resolves later
```

Expected:

```text
UI remains B
```

This is a valuable integration test for search/filter interfaces.

---

# Part 23 — What Not to Test

## 30. Low-Value Tests

Usually avoid tests like:

```text
component has class "card"
private state equals X
specific helper called 3 times
component contains exactly 4 divs
Hook declaration order
internal function name
```

unless that implementation detail itself is an intentional public contract.

---

# Part 24 — Snapshot Tests

## 31. Use Snapshots Selectively

Large snapshots often become:

```text
test fails
→ developer updates snapshot
→ nobody reviews meaningful difference
```

Snapshots can be useful for small, stable serialized output.

They should not replace behavioral assertions.

---

# Part 25 — Test Isolation

## 32. Tests Should Not Depend on Order

Bad:

```text
test 2 only passes
if test 1 ran first
```

Each test should establish the state it requires.

Reset mocks, storage, timers and shared state appropriately.

---

# Part 26 — Determinism

## 33. Avoid Random/Real External Dependencies

Flaky tests often depend on:

- real network
- current date/time
- random values
- test execution order
- unresolved async work

Control external boundaries so tests produce consistent results.

---

# Part 27 — CareerLoop Example

## 34. Application Creation Flow

High-value integration test:

```text
render Add Application form
      ↓
enter company/title
      ↓
submit
      ↓
pending UI appears
      ↓
mock server succeeds
      ↓
new application appears
      ↓
form closes/resets as designed
```

Failure test:

```text
submit
→ server rejects
→ values remain
→ actionable error appears
```

---

# Part 28 — CodeBuddy Example

## 35. Connection Flow

```text
render developer card
→ click Connect
→ pending/optimistic state
→ successful response
→ Connected
```

Also test:

```text
server failure
→ state reconciles
→ user receives feedback
```

For chat:

```text
conversation changes
→ previous subscription cleaned
→ correct new room used
```

---

# Part 29 — Security Tests

## 36. Frontend Security Expectations

React tests can verify UX:

```text
non-admin does not see admin button
```

But that does **not** prove authorization.

Server/API tests must verify:

```text
non-admin directly calls endpoint
→ denied
```

This distinction from Lesson 71 is critical.

---

# Part 30 — Test Naming

## 37. Name Behavior

Prefer:

```text
shows validation error when email is invalid
preserves form values when submission fails
disables submit while request is pending
shows empty state when no applications exist
```

over:

```text
test1
works correctly
handleSubmit test
```

The test name should document expected behavior.

---

# Part 31 — Arrange, Act, Assert

## 38. Simple Structure

```text
ARRANGE
render/setup

ACT
user performs behavior

ASSERT
observe result
```

Example:

```js
// Arrange
render(<Counter />);

// Act
await user.click(
  screen.getByRole(
    "button",
    { name: /increment/i }
  )
);

// Assert
expect(
  screen.getByText("Count: 1")
).toBeVisible();
```

This makes tests easy to scan.

---

# Part 32 — Testing Checklist

## 39. Before Shipping a Feature

Ask:

```text
1. Is the main success flow tested?
2. Is validation tested?
3. Is loading/pending behavior tested?
4. Is empty state tested?
5. Is error/retry behavior tested?
6. Are important permissions represented in UI tests?
7. Are server permissions separately tested server-side?
8. Are important user interactions tested realistically?
9. Are async assertions waiting for observable UI?
10. Are network/external boundaries controlled?
11. Are race conditions relevant?
12. Is optimistic rollback relevant?
13. Are tests independent?
14. Are tests coupled to implementation details?
15. Would these tests survive a safe internal refactor?
```

---

# Part 33 — Common Mistakes

## 40. Common Testing Mistakes

1. Testing implementation details.
2. Inspecting state instead of visible behavior.
3. Using CSS selectors when accessible queries are available.
4. Testing only success.
5. Using arbitrary timeouts/sleeps.
6. Mocking every internal module.
7. Testing React itself.
8. Writing isolated Hook tests without a meaningful reason.
9. Depending on exact Effect execution counts.
10. Ignoring race conditions.
11. Ignoring optimistic failure/rollback.
12. Using giant snapshots as primary assertions.
13. Sharing state between tests.
14. Calling real production APIs in ordinary component tests.
15. Writing vague test names.
16. Assuming hidden admin UI proves security.
17. Making tests so implementation-specific that refactoring breaks everything.

---

# Part 34 — Interview Questions

## 41. What Should You Test in React?

Test important user-visible behavior, interactions, async states and feature contracts rather than React internals or private implementation details.

---

## 42. Unit vs Integration vs E2E?

Unit tests isolate small logic.

Integration tests verify multiple pieces working together.

E2E tests verify complete application flows through a real or realistic browser environment.

---

## 43. Why Prefer getByRole?

It queries the interface through semantics users and assistive technologies understand, making tests closer to real usage and encouraging accessible markup.

---

## 44. getBy vs findBy?

Conceptually:

```text
getBy
→ element expected now

findBy
→ element expected later/asynchronously
```

---

## 45. Why Avoid Implementation Details?

Because implementation can change while user behavior remains correct. Tests should enable safe refactoring rather than prevent it.

---

## 46. What Should Be Mocked?

Mock true external boundaries when necessary—such as network, browser APIs, time or third-party services—while keeping enough real application code running to provide confidence.

---

## 47. Why Are Integration Tests Valuable?

Many bugs happen where components, state, data layers and user interactions meet. Integration tests exercise those relationships.

---

# Part 35 — Interview-Ready Answer

## 48. Final Answer

```text
I test React applications primarily from the user's
perspective. I prefer semantic queries such as role and
accessible name, perform realistic interactions, and
assert visible behavior rather than component state or
Hook implementation.

I use unit tests for pure business logic, component and
integration tests for important UI flows, and E2E tests
for critical end-to-end journeys.

For async features I test loading, success, empty,
error, retry and mutation states, and when relevant I
also test race conditions and optimistic rollback.

I mock external boundaries rather than every internal
module, avoid arbitrary sleeps and large low-value
snapshots, and keep tests independent and deterministic.
The goal is confidence that survives internal
refactoring.
```

---

# Part 36 — Mental Model

## 49. Testing Strategy

```text
                  USER BEHAVIOR
                       │
              interact with UI
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      unit          component     integration
   pure logic       UI behavior   feature flow
        │              │              │
        └──────────────┼──────────────┘
                       ↓
                 critical journeys
                       ↓
                      E2E


Test:
observable behavior
loading/error/empty
async results
failure/retry
race/rollback when relevant

Avoid:
private state
implementation calls
arbitrary sleeps
over-mocking
fragile snapshots
```

---

## 50. Key Takeaways

- Tests should provide confidence in user-visible behavior.
- Prefer behavior over implementation-detail testing.
- Unit tests are excellent for pure functions.
- Component tests verify interactive UI.
- Integration tests provide strong confidence across boundaries.
- E2E tests protect critical complete journeys.
- Query UI semantically where possible.
- Prefer realistic user interactions.
- Use async queries for UI that appears later.
- Avoid arbitrary sleeps.
- Test loading, success, empty and error paths.
- Mock meaningful external boundaries, not everything.
- Do not test React's own implementation.
- Test custom Hooks directly only when their independent contract warrants it.
- Test Effect outcomes and cleanup contracts rather than exact execution counts.
- Test optimistic success and rollback.
- Test race conditions when they can affect correctness.
- Use snapshots selectively.
- Keep tests isolated and deterministic.
- Name tests after behavior.
- Frontend permission tests do not replace server authorization tests.
- Good tests should survive safe internal refactoring.

---

# Section 12 — Production React Completed ✅

Completed lessons:

- Lesson 67 — Component Architecture and Folder Structure ⭐⭐⭐⭐⭐
- Lesson 68 — Separation of Concerns and Container/Presentation Patterns
- Lesson 69 — Error, Loading and Empty-State Design
- Lesson 70 — Accessibility in React
- Lesson 71 — React Security Fundamentals ⭐⭐⭐⭐⭐
- Lesson 72 — Testing React Components

---

## Next Section — Interview Mastery

➡️ [Lesson 73 — Most Important React Interview Questions ⭐⭐⭐⭐⭐](../13-interview-mastery/73-react-interview-questions.md)
