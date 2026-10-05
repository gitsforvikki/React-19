# Lesson 36 — Reusable Component APIs and Composition Patterns

## 1. What Makes a Component Reusable?

A reusable component is not merely a component used twice.

Good reusable components have:

- a clear responsibility
- a predictable API
- useful customization
- sensible defaults
- composition where flexibility is needed
- accessibility built into the abstraction
- minimal knowledge of their consumers

The goal is:

> Reuse behavior and structure without making the API harder than writing the UI directly.

---

## 2. Reuse Through Props

Basic reuse:

```jsx
function Avatar({
  src,
  alt,
  size = 40,
}) {
  return (
    <img
      src={src}
      alt={alt}
      width={size}
      height={size}
    />
  );
}
```

Different configuration:

```jsx
<Avatar
  src="/vikash.jpg"
  alt="Vikash"
  size={64}
/>

<Avatar
  src="/alex.jpg"
  alt="Alex"
/>
```

Props act like configuration inputs for a component.

---

## 3. Design Props Around Intent ⭐⭐⭐⭐⭐

Less expressive:

```jsx
<Button
  red
  rounded
  large
  bold
  shadow
/>
```

This can grow into many interacting booleans.

Often clearer:

```jsx
<Button
  variant="danger"
  size="large"
/>
```

The API describes meaningful design choices rather than implementation details.

---

## 4. Avoid Boolean Prop Explosion

Suppose:

```jsx
<Alert
  success
  warning
  error
  info
/>
```

What happens if:

```jsx
<Alert
  success
  error
/>
```

The API permits contradictory states.

Better:

```jsx
<Alert
  variant="success"
/>
```

Principle:

> Prefer APIs that make invalid combinations difficult to express.

---

## 5. Composition with children ⭐⭐⭐⭐⭐

A wrapper does not need to know all content in advance.

```jsx
function Card({
  children,
}) {
  return (
    <div className="card">
      {children}
    </div>
  );
}
```

Usage:

```jsx
<Card>
  <h2>Frontend Developer</h2>
  <p>React + Next.js</p>
</Card>
```

Mental model:

```text
<Card>
   [ flexible content slot ]
</Card>
```

`children` is one of React's most important composition mechanisms.

---

## 6. Composition Over Giant Configuration Objects

Avoid trying to control every visual detail through a huge API:

```jsx
<Card
  title="..."
  subtitle="..."
  image="..."
  footerText="..."
  footerIcon="..."
  action1="..."
  action2="..."
/>
```

If content varies substantially, composition may be clearer:

```jsx
<Card>
  <CardHeader />
  <CardBody />
  <CardActions />
</Card>
```

Let React's component tree express structure.

---

## 7. Named Slots with Props

Sometimes one `children` slot is not enough.

```jsx
function PageLayout({
  header,
  sidebar,
  children,
}) {
  return (
    <div>
      <header>
        {header}
      </header>

      <aside>
        {sidebar}
      </aside>

      <main>
        {children}
      </main>
    </div>
  );
}
```

Usage:

```jsx
<PageLayout
  header={<Header />}
  sidebar={<Filters />}
>
  <ApplicationList />
</PageLayout>
```

Props can carry JSX, not just strings/numbers.

---

## 8. children vs Named Slots

Use `children` when:

- one main content area exists
- nesting reads naturally

Use named JSX props when:

- several distinct regions exist
- position/meaning should be explicit

Example:

```text
Modal
 ├── title
 ├── children
 └── footer
```

The best API is the one that makes usage easy to understand.

---

## 9. Controlled Component API ⭐⭐⭐⭐⭐

Reusable components often need parent-controlled state.

```jsx
function Modal({
  open,
  onOpenChange,
  children,
}) {
  if (!open) {
    return null;
  }

  return (
    <div>
      {children}

      <button
        onClick={() =>
          onOpenChange(false)
        }
      >
        Close
      </button>
    </div>
  );
}
```

Usage:

```jsx
<Modal
  open={open}
  onOpenChange={setOpen}
>
  ...
</Modal>
```

The parent owns the source of truth.

---

## 10. Semantic Callback Props

Prefer callback names that describe the event:

```jsx
<SearchInput
  value={query}
  onQueryChange={setQuery}
/>
```

rather than exposing implementation details:

```jsx
<SearchInput
  setState={setQuery}
/>
```

Good APIs communicate domain intent.

---

## 11. Uncontrolled Component API

Sometimes a reusable component can manage its own state.

```jsx
function AccordionItem({
  defaultOpen = false,
  children,
}) {
  const [open, setOpen] =
    useState(defaultOpen);

  // ...
}
```

Usage:

```jsx
<AccordionItem
  defaultOpen
>
  ...
</AccordionItem>
```

Naming convention:

```text
value / open
→ controlled current value

defaultValue / defaultOpen
→ uncontrolled initial value
```

This connects to Lesson 16.

---

## 12. Controlled + Uncontrolled API Design

A component library may intentionally support both modes.

Conceptually:

```jsx
<Dialog
  open={open}
  onOpenChange={setOpen}
/>
```

or:

```jsx
<Dialog
  defaultOpen
/>
```

This is more complex to implement correctly, so only support both when consumers genuinely need both.

Do not add flexibility without a use case.

---

## 13. State Colocation in Reusable Components

A reusable component should own state that is truly internal.

Example:

```text
Tooltip
 └── internal hover/focus details
```

But state that consumers must coordinate may need to be controlled externally.

Ask:

> Does the parent need to observe or coordinate this value?

If yes, a controlled API may be appropriate.

---

## 14. Compound Components ⭐⭐⭐⭐⭐

A compound component API lets related components work together through composition.

Example usage:

```jsx
<Tabs>
  <Tabs.List>
    <Tabs.Trigger
      value="profile"
    >
      Profile
    </Tabs.Trigger>

    <Tabs.Trigger
      value="settings"
    >
      Settings
    </Tabs.Trigger>
  </Tabs.List>

  <Tabs.Panel
    value="profile"
  >
    Profile content
  </Tabs.Panel>

  <Tabs.Panel
    value="settings"
  >
    Settings content
  </Tabs.Panel>
</Tabs>
```

This can be easier to extend than a giant configuration prop.

Lesson 54 covers compound components deeply; here focus on the API design idea.

---

## 15. How Compound Components Coordinate

Typical mental model:

```text
Tabs
 ├── owns/provides active value
 │
 └── Context
      ├── Tabs.List
      ├── Tabs.Trigger
      └── Tabs.Panel
```

Context can coordinate closely related child components without exposing wiring to consumers.

Use this pattern when components conceptually form one reusable system.

---

## 16. Composition vs Inheritance ⭐⭐⭐⭐⭐

React favors composition.

Instead of building class-like UI inheritance hierarchies:

```text
BaseCard
  ↓
UserCard
  ↓
AdminUserCard
```

compose behavior/structure:

```jsx
<Card>
  <UserDetails />
  <AdminActions />
</Card>
```

Composition is usually more flexible because pieces can be rearranged rather than locked into an inheritance tree.

---

## 17. Headless Component / Hook Idea

Sometimes reusable logic should not dictate markup.

Example:

```jsx
const {
  open,
  toggle,
  triggerProps,
  contentProps,
} = useDisclosure();
```

Consumer decides presentation.

This style separates:

```text
behavior
from
visual representation
```

Custom Hooks are often useful for headless reusable behavior.

But do not create complicated headless abstractions unless flexibility is needed.

---

## 18. Reusable Hook vs Reusable Component ⭐⭐⭐⭐⭐

Choose a **component** when reuse includes UI/markup:

```jsx
<Button />
<Card />
<Modal />
```

Choose a **custom Hook** when reuse is primarily behavior:

```jsx
useOnlineStatus()
useMediaQuery()
useApplicationFilters()
```

Sometimes use both:

```text
useDialog behavior
      ↓
Dialog component
      ↓
consumer UI
```

---

## 19. Pass Components/JSX Instead of Flags

Suppose a card has optional actions.

Instead of:

```jsx
<Card
  showDelete
  showEdit
  showShare
/>
```

consider:

```jsx
<Card
  actions={
    <>
      <EditButton />
      <DeleteButton />
    </>
  }
/>
```

This gives the caller compositional control and prevents the Card from knowing every future action type.

---

## 20. Avoid Components That Know Too Much

Poor abstraction:

```text
UniversalCard
 ├── user mode
 ├── product mode
 ├── job mode
 ├── developer mode
 ├── admin mode
 └── 40 conditional props
```

Better:

```text
Card primitive
 ├── ApplicationCard
 ├── DeveloperCard
 └── ProductCard
```

Reuse the stable primitive; keep domain-specific behavior in domain components.

---

## 21. Props Should Have Clear Ownership

Avoid APIs like:

```jsx
<Component
  data={hugeObject}
/>
```

when the component really needs:

```jsx
<Component
  name={user.name}
  avatarUrl={user.avatarUrl}
/>
```

But don't blindly split every object either.

Decision:

> Pass the data shape that best represents the component's responsibility.

Explicit APIs reduce accidental coupling.

---

## 22. Prop Spreading with Restraint

This can be useful for wrapper primitives:

```jsx
function Button({
  children,
  ...buttonProps
}) {
  return (
    <button
      {...buttonProps}
    >
      {children}
    </button>
  );
}
```

But uncontrolled spreading can accidentally forward:

- invalid DOM attributes
- private/internal props
- unexpected handlers

Use prop spreading intentionally.

---

## 23. Wrapper Components Should Preserve Native Semantics

Reusable UI should not throw away browser behavior.

Prefer:

```jsx
function Button(props) {
  return <button {...props} />;
}
```

over:

```jsx
function Button({
  onClick,
}) {
  return (
    <div
      onClick={onClick}
    />
  );
}
```

A real `button` provides keyboard and accessibility semantics that a `div` does not automatically provide.

---

## 24. Accessibility Is Part of the API ⭐⭐⭐⭐⭐

For reusable components, accessibility should be designed into the abstraction.

A reusable `IconButton` may require an accessible label:

```jsx
<IconButton
  aria-label="Delete application"
>
  <TrashIcon />
</IconButton>
```

If accessibility is left entirely to every consumer, the reusable abstraction may spread mistakes across the application.

---

## 25. Ref Exposure Should Be Intentional

Some reusable components need to expose a DOM ref for:

- focus
- measurement
- integration with another system

In React 19, `ref` can be received as a prop by function components in supported cases, reducing the need for `forwardRef` in new React 19 code.

Ref design is covered more deeply in Lesson 56.

Principle here:

> Expose imperative access only when consumers actually need it.

---

## 26. Avoid Overusing Imperative APIs

Prefer:

```jsx
<Modal
  open={open}
/>
```

over:

```jsx
modalRef.current.open();
```

when state/props can express the UI declaratively.

Imperative handles are escape hatches, not the default reusable component API.

---

## 27. Stable Public API, Flexible Internals

Consumers should depend on:

```jsx
<Button
  variant="primary"
>
  Save
</Button>
```

not on internal DOM structure.

Then implementation can change:

```text
CSS classes
internal wrappers
icons
loading markup
```

without forcing all callers to change.

This is encapsulation.

---

## 28. API Surface Area

Every prop is part of a component's public API.

More props mean:

- more combinations
- more documentation
- more tests
- more compatibility burden

Do not expose configuration merely because implementation can support it.

Prefer the smallest API that covers real use cases.

---

## 29. Sensible Defaults

Good:

```jsx
function Button({
  type = "button",
  size = "medium",
  children,
}) {
  // ...
}
```

Defaults reduce repetitive usage.

But defaults should be unsurprising and safe.

For reusable form buttons, for example, explicitly considering `type` prevents accidental submit behavior.

---

## 30. Avoid Mirroring Props into State

Bad reusable component:

```jsx
function Input({
  value,
}) {
  const [
    internalValue,
    setInternalValue,
  ] = useState(value);
}
```

Now there may be two sources of truth.

Choose intentionally:

```text
controlled → value prop is source of truth
uncontrolled → internal state + defaultValue
```

Do not accidentally mix them.

---

## 31. CareerLoop Example — ApplicationCard

Too configurable:

```jsx
<ApplicationCard
  showCompany
  showStatus
  showDelete
  showEdit
  showNotes
  showDate
  editable
  compact
  ...
/>
```

Better separation:

```jsx
<Card>
  <ApplicationHeader
    application={
      application
    }
  />

  <ApplicationDetails
    application={
      application
    }
  />

  <ApplicationActions
    applicationId={
      application.id
    }
  />
</Card>
```

Composition prevents one component from becoming a conditional maze.

---

## 32. CodeBuddy Example — DeveloperCard

Reusable primitive:

```jsx
<Card>
  <DeveloperProfile
    developer={developer}
  />

  <CardActions>
    <ConnectButton
      developerId={
        developer.id
      }
    />

    <MessageButton
      developerId={
        developer.id
      }
    />
  </CardActions>
</Card>
```

The generic Card does not need to know about networking, connections, or messaging.

---

## 33. Composition Can Reduce Prop Drilling

Instead of:

```text
Layout
 ↓ user
Sidebar
 ↓ user
Avatar
```

the component that already knows `user` can sometimes create:

```jsx
<Layout
  sidebar={
    <Avatar user={user} />
  }
/>
```

Now `Layout` does not need to understand the user data.

This is one alternative to Context.

---

## 34. Render Props — Recognize the Pattern

A render prop passes a function that returns UI:

```jsx
<DataProvider
  render={(data) => (
    <List data={data} />
  )}
/>
```

or:

```jsx
<DataProvider>
  {(data) => (
    <List data={data} />
  )}
</DataProvider>
```

This pattern remains useful to recognize, though custom Hooks often provide a simpler way to reuse non-visual logic in function-component code.

Lesson 55 covers render props and HOCs in detail.

---

## 35. HOCs — Recognize the Pattern

Higher-order component:

```jsx
const EnhancedComponent =
  withSomething(
    Component
  );
```

HOCs are still important for understanding existing libraries/codebases, but modern function-component logic reuse often favors Hooks and composition.

Detailed coverage comes in Lesson 55.

---

## 36. Reusability Has a Cost

Every abstraction adds:

```text
API decisions
indirection
documentation
testing
constraints
```

Therefore:

> Do not abstract because two pieces of code merely look similar.

Abstract when they share a stable concept.

Premature generic components often become harder to use than duplicated straightforward components.

---

## 37. A Practical Extraction Rule

Ask:

1. Is the responsibility clear?
2. Are the use cases genuinely similar?
3. Which parts are stable?
4. Which parts must vary?
5. Is variation better expressed by props or composition?
6. Does the abstraction make caller code clearer?
7. Does it preserve accessibility?
8. Is the API smaller than the complexity it hides?

If not, keep the components separate for now.

---

## 38. Common Mistakes ⭐⭐⭐⭐⭐

1. Creating one universal component for unrelated domains.
2. Adding dozens of boolean props.
3. Making contradictory prop combinations possible.
4. Using Context when simple composition would solve the problem.
5. Putting reusable behavior in UI components when a Hook is more appropriate.
6. Making every component "headless" without a need.
7. Mirroring controlled props into state.
8. Forwarding every prop blindly to the DOM.
9. Using `div` instead of semantic native controls.
10. Exposing imperative refs when declarative props are enough.
11. Over-abstracting after seeing only one use case.
12. Making component internals part of the public API.
13. Ignoring accessibility in reusable primitives.

---

## 39. Interview Questions ⭐⭐⭐⭐⭐

### What is component composition?

Building larger UI by combining smaller components and passing JSX/components as children or props.

### Why does React favor composition?

It provides flexible reuse without rigid inheritance hierarchies.

### What is the children prop?

The JSX nested inside a component tag, available to that component through `children`.

### When would you use named slots?

When a reusable component has multiple distinct customizable regions such as header, sidebar, body, or footer.

### What is a controlled reusable component?

A component whose important state is owned by its parent and supplied through props with callbacks for requested changes.

### Why avoid many boolean props?

They create a large number of combinations and can allow contradictory/unclear states.

### Custom Hook vs reusable component?

Use a Hook primarily for reusable behavior/stateful logic and a component for reusable UI/markup.

### What is a compound component?

A set of related components designed to compose together under a shared parent, often coordinating through Context.

### Composition vs inheritance?

React generally favors composition: combine independent pieces rather than extending UI classes through inheritance hierarchies.

### Why is accessibility important in reusable components?

A mistake in a reusable primitive is repeated everywhere it is used, so semantic and accessible behavior should be part of the component contract.

---

## 40. Complete Mental Model

```text
              REUSE NEED
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
      behavior              UI
        │                   │
  custom Hook          component
                            │
                ┌───────────┼───────────┐
                ↓           ↓           ↓
              props      children    slots
                            │
                            ↓
                       composition

Complex related UI system
          ↓
 compound components
          ↓
 Context for internal coordination
 when justified
```

---

## 41. Section 6 Revision ⭐⭐⭐⭐⭐

```text
Lesson 34
Custom Hooks
→ reuse stateful React logic

Lesson 35
Rules of Hooks
→ keep Hook execution valid
  and predictable

Lesson 36
Reusable component APIs
→ reuse UI through clear props
  and composition
```

Core distinction:

```text
logic reuse
→ custom Hooks

UI reuse
→ components

flexible UI structure
→ composition
```

---

## 42. Key Takeaways

- Reusable components need clear, intentional APIs.
- Props configure components; `children` composes content.
- Named JSX props work well for multiple slots.
- Prefer meaningful variants over boolean prop explosions.
- Controlled APIs enable parent coordination.
- Uncontrolled APIs can simplify self-contained behavior.
- Do not accidentally maintain two sources of truth.
- Compound components can model cohesive UI systems.
- React favors composition over inheritance.
- Use custom Hooks for reusable behavior and components for reusable UI.
- Headless patterns are useful when behavior must be presentation-independent.
- Preserve native semantics and accessibility.
- Keep imperative/ref APIs as escape hatches.
- Public API surface should be as small as practical.
- Stable abstractions should hide implementation details.
- Do not generalize unrelated components into a universal component.
- Reusability should reduce complexity, not merely move it.

---

## Section 6 — Reusable Logic Completed ✅

You have completed:

- Lesson 34 — Custom Hooks ⭐⭐⭐⭐⭐
- Lesson 35 — Rules of Hooks ⭐⭐⭐⭐⭐
- Lesson 36 — Reusable Component APIs and Composition Patterns

---

## Next Section — Performance

➡️ [Lesson 37 — React.memo ⭐⭐⭐⭐⭐](../07-performance/37-react-memo.md)
