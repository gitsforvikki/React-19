# Lesson 67 — Component Architecture and Folder Structure ⭐⭐⭐⭐⭐

## 1. Why Architecture Matters

Small React applications can survive almost any folder structure.

Production applications cannot.

As an application grows, poor structure causes:

- giant components
- duplicated logic
- unclear ownership
- excessive prop drilling
- difficult testing
- circular dependencies
- unrelated files coupled together
- fear of changing existing code

Good architecture answers:

```text
Where should this code live?
Who owns this state?
Which component owns this responsibility?
What is reusable?
What is feature-specific?
Which direction may dependencies flow?
```

The goal is not to create the most folders.

The goal is to make change **predictable**.

---

# Part 1 — Start With Responsibilities

## 2. Think in Responsibilities, Not File Count ⭐⭐⭐⭐⭐

A component should represent a meaningful responsibility.

Example:

```text
ApplicationBoard
├── ApplicationFilters
├── ApplicationList
│   └── ApplicationCard
└── AddApplicationButton
```

Each component has a recognizable job.

Avoid rules such as:

```text
"Every component must be under 100 lines."
```

Line count can reveal complexity, but it is not architecture.

A 200-line cohesive component may be healthier than ten tiny components with unclear boundaries.

---

## 3. Cohesion and Coupling ⭐⭐⭐⭐⭐

### High cohesion

Code that belongs together stays together.

```text
ApplicationCard
├── card rendering
├── card-specific formatting
└── card-specific interactions
```

### Low coupling

A feature does not unnecessarily depend on unrelated features.

```text
applications feature
should not require
internal implementation of
profile feature
```

Good architecture aims for:

```text
HIGH COHESION
+
LOW COUPLING
```

---

# Part 2 — Component Boundaries

## 4. When Should You Extract a Component? ⭐⭐⭐⭐⭐

Consider extraction when a piece of UI:

- has its own responsibility
- is reused
- has meaningful independent state/behavior
- is complex enough to name
- improves readability
- needs an independent performance boundary
- is naturally testable in isolation

Example:

```jsx
function ApplicationCard({
  application,
}) {
  return (
    <article>
      <ApplicationHeader
        application={application}
      />

      <ApplicationStatus
        status={application.status}
      />

      <ApplicationActions
        applicationId={application.id}
      />
    </article>
  );
}
```

Extraction should make the architecture clearer, not merely create more files.

---

## 5. Do Not Over-Componentize

This:

```text
Card
├── CardTop
│   ├── CardTopLeft
│   └── CardTopRight
└── CardBottom
```

is not automatically better.

If these pieces:

- have no meaningful responsibility
- are never reused
- make navigation harder

keep them together.

A useful question:

> Can I give this extracted component a meaningful domain/UI responsibility name?

---

# Part 3 — State Ownership

## 6. State Architecture Is Component Architecture ⭐⭐⭐⭐⭐

Where state lives determines how components depend on one another.

Rule:

> Keep state as close as possible to the components that need it, but high enough to serve every consumer that must share it.

```text
           Parent
             │
      shared state here
        ┌────┴────┐
        ↓         ↓
      Child A   Child B
```

If only Child A needs the state:

```text
Parent
  │
  └── Child A
       └── state here
```

Do not lift state globally without a reason.

---

## 7. Local State First

Prefer:

```text
local component state
        ↓
lift when genuinely shared
        ↓
context when distant subtree needs it
        ↓
external store when application requirements justify it
```

This keeps dependencies narrow.

Global state is not a default architecture.

---

## 8. Separate State by Ownership

Examples:

### Local UI state

```text
modal open
selected tab
input draft
dropdown open
```

### Shared feature state

```text
application filters
checkout workflow
editor state
```

### Server state

```text
applications from database
developer profiles
orders
```

### URL/navigation state

```text
search query
page
sort
filter
```

Do not automatically place every category in one store.

---

# Part 4 — Organizing by Feature

## 9. Why Feature-Based Structure Scales ⭐⭐⭐⭐⭐

A simple beginner structure often looks like:

```text
src/
├── components/
├── hooks/
├── services/
├── utils/
└── pages/
```

This is acceptable initially.

But in a large application:

```text
components/
├── ApplicationCard.jsx
├── JobFilter.jsx
├── ProfileCard.jsx
├── CheckoutButton.jsx
├── ChatMessage.jsx
└── 100 more files...
```

unrelated features become mixed together.

---

## 10. Feature-Oriented Example ⭐⭐⭐⭐⭐

```text
src/
├── app/
│   ├── App.jsx
│   └── providers.jsx
│
├── features/
│   ├── applications/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── api/
│   │   ├── utils/
│   │   └── index.js
│   │
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── api/
│   │
│   └── profile/
│       ├── components/
│       ├── hooks/
│       └── api/
│
├── components/
│   └── ui/
│
├── hooks/
├── lib/
└── assets/
```

The exact names are flexible.

The principle matters more:

> Code that changes together should usually live near each other.

---

# Part 5 — Feature Code vs Shared Code

## 11. Feature-Specific Code

Suppose `ApplicationStatusBadge` is only meaningful inside CareerLoop applications.

Keep it in:

```text
features/applications/components/
```

Do not move it into global shared UI simply because it is a component.

---

## 12. Shared UI

Examples:

```text
Button
Input
Dialog
Spinner
Avatar
Tooltip
```

These can live in:

```text
components/ui/
```

They should generally avoid knowing about business domains.

Bad shared button:

```jsx
function Button({
  applicationStatus,
  recruiterId,
  ...
}) {
```

A generic UI primitive should not absorb feature-specific business logic.

---

## 13. Shared Does Not Mean "Maybe Reusable"

Do not prematurely create:

```text
shared/
utils/
common/
helpers/
```

for every piece of code.

Start close to the feature.

Move something to shared code when genuine reuse appears.

This avoids a global dumping ground.

---

# Part 6 — Colocation

## 14. Colocate Related Code ⭐⭐⭐⭐⭐

If a hook is used only by one feature:

```text
features/chat/
├── components/
├── hooks/
│   └── useTypingIndicator.js
└── ...
```

is often better than:

```text
src/hooks/useTypingIndicator.js
```

Colocation reduces the distance between related concepts.

---

## 15. Colocation Can Include Tests and Styles

Example:

```text
ApplicationCard/
├── ApplicationCard.jsx
├── ApplicationCard.test.jsx
└── ApplicationCard.module.css
```

or:

```text
applications/
├── components/
├── __tests__/
└── ...
```

There is no single mandatory React convention.

Choose a consistent structure that makes ownership obvious.

---

# Part 7 — Layers Inside a Feature

## 16. A Useful Feature Mental Model

```text
Feature
│
├── UI
│   └── components
│
├── behavior
│   └── hooks/state
│
├── external communication
│   └── api/services
│
└── pure domain helpers
    └── utils
```

Do not create every layer when the feature does not need it.

Architecture should grow with complexity.

---

# Part 8 — Dependency Direction

## 17. Keep Dependency Direction Predictable ⭐⭐⭐⭐⭐

A healthy high-level model:

```text
app
 ↓
features
 ↓
shared UI / shared utilities
```

Shared low-level modules should not depend on high-level feature modules.

Bad:

```text
shared Button
     ↓
imports applications feature
```

Better:

```text
applications feature
     ↓
imports shared Button
```

---

## 18. Avoid Circular Dependencies

Bad:

```text
applications
   ↓
profile
   ↓
applications
```

Circular dependencies make ownership unclear and can create runtime/build problems.

If two features need the same lower-level concept, consider extracting that concept to an appropriate shared layer.

Do not solve every cycle by dumping everything into `utils`.

---

# Part 9 — Public Feature APIs

## 19. Control What Other Features Import

A feature can expose a small public API:

```js
// features/applications/index.js

export {
  ApplicationList,
} from "./components/ApplicationList";

export {
  useApplications,
} from "./hooks/useApplications";
```

Consumers import from the feature boundary instead of deep internal paths.

Conceptually:

```text
Other feature
     ↓
applications/index
     ↓
approved public API
     ↓
internal implementation
```

Benefits:

- easier refactoring
- clearer ownership
- fewer accidental dependencies

Use barrel files deliberately; excessive barrels can also obscure dependency graphs or create cycles.

---

# Part 10 — UI vs Business Logic

## 20. Avoid Giant Components ⭐⭐⭐⭐⭐

Problem:

```jsx
function Checkout() {
  // fetch products
  // calculate totals
  // validate coupon
  // payment integration
  // inventory logic
  // modal state
  // analytics
  // 500 lines of JSX
}
```

This component owns too many responsibilities.

Better conceptual split:

```text
CheckoutFeature
├── CheckoutView
├── CartItems
├── OrderSummary
├── PaymentSection
├── useCheckout
└── pricing/domain helpers
```

But business-critical validation should still live in the authoritative server/domain layer where appropriate.

---

# Part 11 — Custom Hooks as Architecture

## 21. Extract Behavior, Not Random Code

A custom Hook can give complex React behavior a name:

```jsx
function useApplicationFilters() {
  // state
  // derived values
  // callbacks
}
```

Component:

```jsx
function ApplicationBoard() {
  const filters =
    useApplicationFilters();

  return (
    <ApplicationView
      {...filters}
    />
  );
}
```

A Hook is useful when it captures reusable or independently meaningful React behavior.

Do not create a custom Hook merely to hide three obvious lines.

---

# Part 12 — Composition Over Configuration Explosion

## 22. Avoid Giant Prop APIs ⭐⭐⭐⭐⭐

Problem:

```jsx
<Card
  showHeader
  showFooter
  compact
  withAvatar
  showActions
  actionPosition="right"
  headerVariant="large"
/>
```

As reusable components grow, boolean configuration can become difficult.

Composition may be clearer:

```jsx
<Card>
  <Card.Header>
    ...
  </Card.Header>

  <Card.Content>
    ...
  </Card.Content>

  <Card.Actions>
    ...
  </Card.Actions>
</Card>
```

Use the simplest API that fits the real reuse cases.

---

# Part 13 — Context Boundaries

## 23. Scope Context to the Domain That Needs It

Bad:

```text
AppGlobalContext
├── user
├── theme
├── cart
├── notifications
├── applications
├── modal
└── everything else
```

This creates broad coupling.

Prefer focused contexts when Context is appropriate:

```text
ThemeProvider
AuthProvider
CheckoutProvider
```

Do not create Context for state that can remain local.

---

# Part 14 — Provider Architecture

## 24. Keep Providers Understandable

Instead of deeply nesting providers directly in the root:

```jsx
function AppProviders({
  children,
}) {
  return (
    <ThemeProvider>
      <AuthProvider>
        {children}
      </AuthProvider>
    </ThemeProvider>
  );
}
```

This gives root composition a clear responsibility.

But do not add a provider wrapper merely because it looks architectural.

---

# Part 15 — Data Transformation

## 25. Keep Pure Transformations Pure

Example:

```js
export function getApplicationStats(
  applications
) {
  return {
    total:
      applications.length,

    interviews:
      applications.filter(
        item =>
          item.status ===
          "interview"
      ).length,
  };
}
```

Pure business/data transformations do not need to be React Hooks.

Benefits:

- easy testing
- reusable outside components
- no Hook restrictions
- clear separation

---

# Part 16 — Naming

## 26. Names Communicate Architecture ⭐⭐⭐⭐⭐

Prefer:

```text
ApplicationStatusFilter
useApplicationFilters
calculateOrderTotal
updateApplicationStatus
```

over:

```text
Component2
useData
helper
handleThing
```

Names should reveal responsibility.

Good naming reduces the amount of architecture documentation needed.

---

# Part 17 — Folder Depth

## 27. Avoid Premature Folder Mazes

Bad for a tiny feature:

```text
features/
└── auth/
    └── components/
        └── forms/
            └── login/
                └── fields/
                    └── ...
```

Deep nesting increases navigation cost.

Start simple.

Add hierarchy when it represents real complexity.

---

# Part 18 — Example: CareerLoop

## 28. Production-Oriented Structure

```text
src/
├── app/
│   ├── App.jsx
│   └── providers.jsx
│
├── features/
│   ├── applications/
│   │   ├── components/
│   │   │   ├── ApplicationCard.jsx
│   │   │   ├── ApplicationList.jsx
│   │   │   └── ApplicationFilters.jsx
│   │   ├── hooks/
│   │   │   └── useApplicationFilters.js
│   │   ├── api/
│   │   │   └── applications.api.js
│   │   ├── utils/
│   │   │   └── applicationStats.js
│   │   └── index.js
│   │
│   └── auth/
│       └── ...
│
├── components/
│   └── ui/
│       ├── Button.jsx
│       ├── Input.jsx
│       └── Dialog.jsx
│
└── lib/
    └── ...
```

This is an example, not a required React standard.

---

# Part 19 — Example: CodeBuddy

## 29. Feature Boundaries

```text
features/
├── discovery/
│   ├── DeveloperCard
│   ├── DiscoveryFeed
│   └── useDiscovery
│
├── connections/
│   ├── ConnectionButton
│   └── ConnectionList
│
├── chat/
│   ├── ChatWindow
│   ├── MessageList
│   └── useSocketChat
│
└── subscription/
    ├── PricingCards
    └── SubscriptionStatus
```

A chat-specific socket hook should usually remain in the chat feature unless other domains genuinely need the same abstraction.

---

# Part 20 — Architecture Evolution

## 30. Do Not Design for Imaginary Scale ⭐⭐⭐⭐⭐

A healthy evolution:

```text
small app
   ↓
simple folders
   ↓
feature becomes complex
   ↓
colocate feature code
   ↓
real reuse appears
   ↓
extract shared abstraction
```

Bad evolution:

```text
day 1
   ↓
20 architectural layers
   ↓
most folders contain one file
   ↓
developers fight the structure
```

Architecture should respond to actual pressure.

---

# Part 21 — Refactoring Signals

## 31. Signs a Component/Feature Needs Refactoring

Look for:

- unrelated reasons to change
- difficult naming
- deeply nested conditional JSX
- many unrelated state variables
- repeated logic
- very large prop surfaces
- prop drilling across unrelated layers
- Effects coordinating other Effects
- domain logic embedded everywhere in JSX
- circular imports
- shared folders becoming dumping grounds
- tests requiring huge setup for simple behavior

These are signals, not automatic rules.

---

# Part 22 — Common Mistakes

## 32. Common Architecture Mistakes ⭐⭐⭐⭐⭐

1. Organizing a large app only by file type.
2. Making every component globally reusable.
3. Moving feature-specific code into `shared` too early.
4. Creating giant components with many responsibilities.
5. Extracting meaningless one-line components everywhere.
6. Putting all state in a global store.
7. Putting all application state in one Context.
8. Lifting state higher than necessary.
9. Mixing server state and UI state without clear ownership.
10. Allowing low-level shared modules to import feature code.
11. Deep-importing every feature's internals.
12. Creating circular dependencies.
13. Using Hooks for logic that could be a pure function.
14. Building complex abstractions before genuine reuse exists.
15. Using line count as the only extraction rule.
16. Copying a folder structure from another project without understanding why it exists.
17. Mixing framework-specific folder conventions with core React architecture rules.

---

# Part 23 — Architecture Checklist

## 33. Before Adding Code, Ask ⭐⭐⭐⭐⭐

```text
1. Which feature owns this?
2. Is it feature-specific or truly shared?
3. Is this UI, React behavior, domain logic,
   or external communication?
4. Who should own the state?
5. Can state remain local?
6. Is the component responsibility clear?
7. Am I introducing an unnecessary dependency?
8. Does this abstraction exist because of real reuse?
9. Can another feature depend only on a public API?
10. Will a future developer know where to change this?
```

---

# Part 24 — Interview Questions

## 34. How Do You Structure a Large React Application? ⭐⭐⭐⭐⭐

A strong answer:

```text
I prefer feature-oriented organization for domain code,
with reusable low-level UI and utilities separated from
feature-specific implementation.

Within a feature I colocate its components, Hooks, API
integration and pure helpers when those layers are
actually needed.

I keep state close to its consumers, control dependency
direction, avoid premature shared abstractions and expose
small feature APIs rather than allowing arbitrary deep
imports.
```

---

## 35. Feature-Based vs Type-Based Folders?

Type-based folders are simple for small applications.

Feature-based organization often scales better because code that changes together remains colocated.

Hybrid structures are common and practical.

---

## 36. When Do You Extract a Component?

When it has a meaningful responsibility, reuse, independent behavior, complexity, testing value, or improves readability.

Not merely because it exceeded an arbitrary line count.

---

## 37. Where Should State Live?

As close as possible to where it is needed, but high enough to serve all components that genuinely share it.

---

## 38. What Is Colocation?

Keeping code near the feature/component that owns and changes with it, rather than placing everything in global folders.

---

## 39. What Is High Cohesion?

A module contains closely related responsibilities.

---

## 40. What Is Low Coupling?

A module depends on as little unrelated implementation as practical.

---

## 41. Why Avoid Premature Abstraction?

Because abstractions created before real usage patterns are known often encode wrong assumptions and increase complexity.

---

# Part 25 — Interview-Ready Summary

## 42. Final Answer ⭐⭐⭐⭐⭐

```text
My React architecture is responsibility and
feature-oriented rather than based on arbitrary file
size rules.

I keep state close to its consumers, colocate
feature-specific components, Hooks, API code and pure
helpers, and move something into shared code only when
there is genuine cross-feature reuse.

I aim for high cohesion and low coupling. High-level
features may depend on reusable lower-level UI and
utilities, but shared modules should not depend back on
feature internals.

I also avoid premature abstraction, giant global
contexts and unnecessary global state. The folder
structure should make ownership and dependency direction
obvious and should evolve as the application grows.
```

---

# Part 26 — Mental Model

## 43. Production Architecture ⭐⭐⭐⭐⭐

```text
                    APPLICATION
                         │
                  app composition
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   Feature A         Feature B        Feature C
        │                │                │
  ┌─────┼─────┐    ┌─────┼─────┐          │
  ↓     ↓     ↓    ↓     ↓     ↓          ↓
 UI   Hooks   API  UI   Hooks   API       ...
  │
  └──────────────┐
                 ↓
          shared primitives
          shared utilities


State:
local first
   ↓
lift only when shared
   ↓
context/store when justified


Architecture goal:
HIGH COHESION
+
LOW COUPLING
+
CLEAR OWNERSHIP
+
PREDICTABLE DEPENDENCIES
```

---

## 44. Key Takeaways

- Architecture is primarily about ownership, responsibilities and dependencies.
- Component line count is not an architecture rule.
- Aim for high cohesion and low coupling.
- Extract components around meaningful responsibilities.
- Avoid both giant components and meaningless over-componentization.
- State ownership is a major part of component architecture.
- Keep state local until it genuinely needs to be shared.
- Feature-oriented organization scales well for domain-heavy applications.
- Keep feature-specific code inside its feature.
- Shared UI should remain domain-agnostic where practical.
- Colocate code that changes together.
- Use layers inside features only when complexity requires them.
- Keep dependency direction predictable.
- Avoid circular dependencies.
- Feature public APIs can protect internal implementation.
- Use custom Hooks for meaningful React behavior.
- Keep pure domain transformations as plain functions.
- Prefer composition over configuration explosion.
- Scope Context rather than creating one global context.
- Good names communicate responsibility.
- Avoid unnecessary folder depth.
- Let architecture evolve from real requirements.
- Do not mix framework-specific filesystem rules with core React architecture.

---

## Next Lesson

➡️ [Lesson 68 — Separation of Concerns and Container/Presentation Patterns](./68-separation-of-concerns.md)
