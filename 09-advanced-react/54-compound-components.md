# Lesson 54 — Compound Components Pattern

## 1. What Is the Compound Components Pattern?

The **Compound Components Pattern** is a component design pattern where multiple related components work together to provide one coordinated feature.

Think about normal HTML:

```html
<select>
  <option>React</option>
  <option>Node.js</option>
</select>
```

`select` and `option` are separate elements, but they are designed to work together.

React compound components use a similar idea.

Example API:

```jsx
<Tabs>
  <Tabs.List>
    <Tabs.Trigger value="profile">
      Profile
    </Tabs.Trigger>

    <Tabs.Trigger value="settings">
      Settings
    </Tabs.Trigger>
  </Tabs.List>

  <Tabs.Panel value="profile">
    Profile content
  </Tabs.Panel>

  <Tabs.Panel value="settings">
    Settings content
  </Tabs.Panel>
</Tabs>
```

The consumer controls the structure while the components coordinate shared behavior.

---

## 2. Why Use Compound Components?

Without compound components, a component API can become configuration-heavy:

```jsx
<Tabs
  tabs={[
    {
      label: "Profile",
      content: <Profile />,
    },
    {
      label: "Settings",
      content: <Settings />,
    },
  ]}
  activeTab={activeTab}
  onChange={setActiveTab}
/>
```

This may be fine for simple cases.

But reusable UI libraries often benefit from a compositional API:

```jsx
<Tabs>
  <Tabs.List>...</Tabs.List>
  <Tabs.Panel>...</Tabs.Panel>
</Tabs>
```

Benefits include:

- flexible composition
- readable relationships
- less prop drilling
- shared internal state
- customizable markup
- reusable component APIs

---

## 3. Core Mental Model ⭐⭐⭐⭐⭐

```text
                Tabs
                 │
        owns shared behavior/state
                 │
            Context Provider
                 │
       ┌─────────┴─────────┐
       ↓                   ↓
 Tabs.Trigger          Tabs.Panel
       │                   │
       └──── useContext ────┘
```

The parent coordinates the feature.

Child components consume the shared contract.

---

## 4. Basic Tabs Example

```jsx
import {
  createContext,
  useContext,
  useState,
} from "react";

const TabsContext =
  createContext(null);

function Tabs({
  defaultValue,
  children,
}) {
  const [value, setValue] =
    useState(defaultValue);

  return (
    <TabsContext.Provider
      value={{
        value,
        setValue,
      }}
    >
      {children}
    </TabsContext.Provider>
  );
}
```

Now child components can participate in the same Tabs state.

---

## 5. Trigger Component

```jsx
function TabsTrigger({
  value,
  children,
}) {
  const tabs =
    useContext(TabsContext);

  const isActive =
    tabs.value === value;

  return (
    <button
      type="button"
      aria-selected={isActive}
      onClick={() =>
        tabs.setValue(value)
      }
    >
      {children}
    </button>
  );
}
```

---

## 6. Panel Component

```jsx
function TabsPanel({
  value,
  children,
}) {
  const tabs =
    useContext(TabsContext);

  if (tabs.value !== value) {
    return null;
  }

  return (
    <div>
      {children}
    </div>
  );
}
```

The consumer does not manually wire `activeTab` into every child.

---

## 7. Exposing the Compound API

One possible API style:

```jsx
Tabs.Trigger = TabsTrigger;
Tabs.Panel = TabsPanel;
```

Then:

```jsx
<Tabs defaultValue="profile">
  <Tabs.Trigger value="profile">
    Profile
  </Tabs.Trigger>

  <Tabs.Trigger value="settings">
    Settings
  </Tabs.Trigger>

  <Tabs.Panel value="profile">
    <Profile />
  </Tabs.Panel>

  <Tabs.Panel value="settings">
    <Settings />
  </Tabs.Panel>
</Tabs>
```

Another perfectly valid style is to export the components separately:

```jsx
export {
  Tabs,
  TabsTrigger,
  TabsPanel,
};
```

The important concept is coordinated composition, not dot syntax itself.

---

## 8. Why Context Is Common ⭐⭐⭐⭐⭐

Compound components often need to share:

- current selected value
- callbacks
- IDs
- orientation
- disabled state
- configuration

Passing all of these manually:

```text
Tabs
 ↓ props
List
 ↓ props
Wrapper
 ↓ props
Trigger
```

creates unnecessary coupling.

Context lets participating components read the shared feature contract directly.

---

## 9. Context Is Not the Pattern Itself

Important distinction:

```text
Compound Components
→ API/design pattern

Context
→ one implementation mechanism
```

You can implement compound components using:

- Context
- child cloning in some older designs
- explicit props
- other internal coordination

Modern React generally favors Context for flexible nested composition.

---

## 10. Avoid cloneElement as the Default Modern Design

Older compound-component implementations often used:

```jsx
Children.map(
  children,
  child =>
    cloneElement(child, {
      activeValue,
    })
);
```

This can become fragile because:

- it often only sees direct children,
- wrappers can break assumptions,
- implicit prop injection is harder to follow,
- child structure becomes constrained.

Context is usually easier for flexible modern APIs.

`cloneElement` still exists, but it should not be your automatic first choice.

---

## 11. Guard the Context ⭐⭐⭐⭐⭐

A reusable component should fail clearly when a child is used outside its required parent.

```jsx
function useTabsContext() {
  const context =
    useContext(TabsContext);

  if (context === null) {
    throw new Error(
      "Tabs components must be used inside <Tabs>."
    );
  }

  return context;
}
```

Then:

```jsx
function TabsTrigger(props) {
  const tabs =
    useTabsContext();

  // ...
}
```

This creates a clearer developer experience.

---

## 12. Flexible Structure

With Context:

```jsx
<Tabs defaultValue="jobs">
  <div className="toolbar">
    <Tabs.Trigger value="jobs">
      Jobs
    </Tabs.Trigger>

    <Tabs.Trigger value="analytics">
      Analytics
    </Tabs.Trigger>
  </div>

  <main>
    <Tabs.Panel value="jobs">
      ...
    </Tabs.Panel>

    <Tabs.Panel value="analytics">
      ...
    </Tabs.Panel>
  </main>
</Tabs>
```

The child components do not have to be direct children of `Tabs`.

That flexibility is one reason Context works well here.

---

## 13. Controlled Compound Components ⭐⭐⭐⭐⭐

A reusable Tabs API may support controlled state:

```jsx
<Tabs
  value={tab}
  onValueChange={setTab}
>
  ...
</Tabs>
```

Now the parent owns the selected tab.

Mental model:

```text
Parent
  │
  ├── value
  └── onValueChange
          │
          ↓
         Tabs
          │
       Context
          ↓
       children
```

This is useful when another part of the application needs to coordinate the state.

---

## 14. Uncontrolled Compound Components

The component can also own state:

```jsx
<Tabs defaultValue="profile">
  ...
</Tabs>
```

This gives a simpler API when the consumer does not need external control.

A polished reusable component may support both patterns carefully.

---

## 15. Controlled vs Uncontrolled API Design

A common design:

```text
value
→ controlled current value

defaultValue
→ initial uncontrolled value

onValueChange
→ change notification
```

This resembles native form component conventions and many mature component libraries.

Avoid silently switching between controlled and uncontrolled behavior.

---

## 16. Accordion Example

Compound API:

```jsx
<Accordion>
  <Accordion.Item value="react">
    <Accordion.Trigger>
      What is React?
    </Accordion.Trigger>

    <Accordion.Content>
      React is a UI library.
    </Accordion.Content>
  </Accordion.Item>
</Accordion>
```

Relationships are visible from JSX structure instead of being hidden in a large configuration object.

---

## 17. Modal/Dialog Example

Conceptually:

```jsx
<Dialog>
  <Dialog.Trigger>
    Delete
  </Dialog.Trigger>

  <Dialog.Content>
    <Dialog.Title>
      Delete application?
    </Dialog.Title>

    <Dialog.Close>
      Cancel
    </Dialog.Close>
  </Dialog.Content>
</Dialog>
```

The related parts share open state and accessibility relationships.

Production dialogs require careful focus/accessibility behavior; use a mature accessible primitive when appropriate.

---

## 18. Compound Components vs One Giant Component ⭐⭐⭐⭐⭐

Giant API:

```jsx
<Dialog
  title="Delete?"
  description="..."
  cancelText="Cancel"
  confirmText="Delete"
  footerPosition="right"
  ...
/>
```

This can become rigid as customization grows.

Compound API:

```jsx
<Dialog>
  <Dialog.Title>...</Dialog.Title>
  <Dialog.Description>...</Dialog.Description>
  <CustomContent />
  <Dialog.Actions>...</Dialog.Actions>
</Dialog>
```

The consumer gets more composition freedom.

---

## 19. But Configuration APIs Are Sometimes Better

Do not force compound components everywhere.

For simple predictable data:

```jsx
<Select
  options={countries}
/>
```

may be clearer than manually composing dozens of children.

Choose the API based on:

- customization needs
- repeated structure
- accessibility requirements
- consumer ergonomics
- complexity

---

## 20. State Ownership ⭐⭐⭐⭐⭐

Ask:

> Which component owns the behavior?

Usually the compound root owns or coordinates shared state.

```text
Tabs
├── selected value
├── change function
└── configuration

children
→ consume shared contract
```

Avoid each child maintaining conflicting copies of shared state.

---

## 21. Separate Behavior from Markup

A good compound API can centralize behavior while allowing flexible presentation.

```text
Root
→ behavior/state

Trigger
→ interaction

Panel
→ visibility/content

consumer
→ composition/layout
```

This is especially valuable in design systems.

---

## 22. Accessibility Must Be Part of the Contract ⭐⭐⭐⭐⭐

A Tabs component is not complete merely because clicking buttons changes content.

A production Tabs implementation may need:

- correct ARIA roles
- `aria-selected`
- relationships between tab and panel
- keyboard arrow navigation
- focus management
- disabled behavior

Reusable components should encapsulate difficult accessibility behavior rather than make every consumer reimplement it.

---

## 23. Performance Considerations

Context updates cause consuming components to update.

For a small coordinated component tree, this is usually appropriate.

If the Context value is unnecessarily large or rapidly changing, consider:

- splitting contexts
- reducing shared values
- colocating state
- profiling before optimization

Do not prematurely complicate a simple component API.

---

## 24. CareerLoop Example

A status filter could expose:

```jsx
<ApplicationStatusTabs
  defaultValue="all"
>
  <ApplicationStatusTabs.List>
    <ApplicationStatusTabs.Trigger
      value="all"
    >
      All
    </ApplicationStatusTabs.Trigger>

    <ApplicationStatusTabs.Trigger
      value="interview"
    >
      Interview
    </ApplicationStatusTabs.Trigger>
  </ApplicationStatusTabs.List>

  <ApplicationStatusTabs.Panel
    value="interview"
  >
    <InterviewApplications />
  </ApplicationStatusTabs.Panel>
</ApplicationStatusTabs>
```

The API makes the relationship between status controls and content explicit.

---

## 25. CodeBuddy Example

A profile card may have compound sections:

```jsx
<Profile>
  <Profile.Header />
  <Profile.Skills />
  <Profile.Actions />
</Profile>
```

Only use shared Context if these parts genuinely need coordinated data/behavior.

Do not add Context simply because dot syntax looks attractive.

---

## 26. When to Use Compound Components

Good fit:

- Tabs
- Accordion
- Dialog
- Menu
- Select
- Dropdown
- multi-part reusable widgets
- design-system primitives

Especially useful when related parts need shared behavior but consumers need layout flexibility.

---

## 27. When Not to Use Them

Avoid when:

- one simple component is sufficient,
- parts do not share behavior,
- a data/configuration prop is much clearer,
- the abstraction makes normal usage harder,
- consumers need to understand too much hidden magic.

Patterns are tools, not requirements.

---

## 28. Common Mistakes ⭐⭐⭐⭐⭐

1. Thinking dot syntax itself defines compound components.
2. Creating Context when components do not share behavior.
3. Keeping duplicated state in each child.
4. Overusing `cloneElement` for implicit prop injection.
5. Forgetting to guard required Context.
6. Ignoring controlled/uncontrolled semantics.
7. Building a visually working but inaccessible Tabs/Dialog API.
8. Making the abstraction more complicated than the feature.
9. Putting unrelated state into the compound Context.
10. Optimizing Context before profiling.

---

## 29. Interview Questions ⭐⭐⭐⭐⭐

### What is the Compound Components Pattern?

A pattern where related components coordinate shared state/behavior while exposing a compositional API.

### Give an example.

Tabs with `Tabs.Trigger` and `Tabs.Panel`, or an Accordion with Trigger and Content components.

### How do compound components communicate?

Modern implementations commonly use Context.

### Is Context the same thing as compound components?

No. Context is an implementation mechanism; compound components are an API/composition pattern.

### Why use this pattern?

It provides flexible composition while centralizing shared behavior.

### Controlled vs uncontrolled compound component?

A controlled root receives current state and change callbacks; an uncontrolled root owns its internal state, often initialized with a default value.

### Why can cloneElement-based implementations be fragile?

They often depend on direct-child structure and implicit prop injection.

### What is an important production concern?

Accessibility behavior must be part of reusable component design.

---

## 30. Complete Mental Model

```text
              Compound Root
                   │
          state + behavior
                   │
              Context
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
    Trigger       List        Panel
       │                       │
 interaction                content
       │                       │
       └──── shared contract ──┘

Consumer controls composition
Root coordinates behavior
```

---

## 31. Key Takeaways

- Compound components are a reusable component API pattern.
- Related components coordinate shared behavior.
- Context is a common modern implementation technique.
- Dot syntax is optional; it is not the pattern itself.
- Root components typically own or coordinate shared state.
- Controlled and uncontrolled APIs can both be supported.
- Prefer clear Context-based coordination over fragile implicit child manipulation.
- Reusable UI must include accessibility behavior.
- Compound components are powerful for design systems and multi-part widgets.
- Do not use the pattern when a simple component/configuration API is clearer.

---

## Next Lesson

➡️ [Lesson 55 — Higher-Order Components and Render Props](./55-hoc-render-props.md)
