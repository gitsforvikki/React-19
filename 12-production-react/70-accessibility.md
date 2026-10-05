# Lesson 70 — Accessibility in React

## 1. What Is Accessibility?

Accessibility means building interfaces that people can use regardless of how they interact with the web.

Users may navigate with:

- keyboard
- screen reader
- voice control
- touch
- zoom
- switch devices
- reduced-motion settings

Accessibility is not a React-only feature.

React renders web interfaces, so the foundation is still:

```text
semantic HTML
+
keyboard behavior
+
accessible names
+
focus management
+
clear feedback
```

---

# Part 1 — Semantic HTML First

## 2. Prefer Native Elements

Bad:

```jsx
<div onClick={save}>
  Save
</div>
```

Better:

```jsx
<button onClick={save}>
  Save
</button>
```

A native button already provides:

- keyboard activation
- focusability
- button semantics
- expected browser behavior

Do not rebuild native behavior unnecessarily.

---

## 3. Semantic Structure

Prefer meaningful elements:

```jsx
<header />
<nav />
<main />
<section />
<article />
<footer />
```

Headings should describe hierarchy:

```text
h1
 ├── h2
 │    └── h3
 └── h2
```

Do not choose heading levels merely for visual size.

Use CSS for appearance.

---

# Part 2 — Accessible Names

## 4. Controls Need Understandable Names

Visible text:

```jsx
<button>
  Delete application
</button>
```

Icon-only control:

```jsx
<button
  aria-label="Delete application"
>
  <TrashIcon
    aria-hidden="true"
  />
</button>
```

The user should be able to understand the control without seeing the icon.

---

## 5. Images Need Appropriate Alternatives

Meaningful image:

```jsx
<img
  src={developer.avatar}
  alt={`${developer.name} profile`}
/>
```

Decorative image:

```jsx
<img
  src="/decoration.svg"
  alt=""
/>
```

Do not write redundant alt text such as:

```text
"image of..."
```

unless that wording is actually meaningful.

---

# Part 3 — Forms

## 6. Inputs Need Labels

Good:

```jsx
<label htmlFor="email">
  Email
</label>

<input
  id="email"
  name="email"
  type="email"
/>
```

A placeholder is not a replacement for a label.

---

## 7. Associate Errors With Fields

```jsx
<label htmlFor="email">
  Email
</label>

<input
  id="email"
  aria-invalid={
    Boolean(error)
  }
  aria-describedby={
    error
      ? "email-error"
      : undefined
  }
/>

{error && (
  <p id="email-error">
    {error}
  </p>
)}
```

This gives assistive technology a relationship between the field and its error.

---

## 8. Group Related Controls

For related radio buttons:

```jsx
<fieldset>
  <legend>
    Work preference
  </legend>

  ...
</fieldset>
```

Semantic grouping is often better than recreating relationships with ARIA.

---

# Part 4 — Keyboard Accessibility

## 9. Interactive UI Must Work Without a Mouse

Test:

```text
Tab
Shift + Tab
Enter
Space
Escape
Arrow keys where expected
```

The exact keyboard behavior depends on the widget.

A custom interactive component must follow expected interaction semantics.

---

## 10. Do Not Add onClick to Everything

This:

```jsx
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
>
  Connect
</div>
```

still requires you to implement keyboard behavior correctly.

Usually:

```jsx
<button
  onClick={handleClick}
>
  Connect
</button>
```

is simpler and more robust.

---

# Part 5 — Focus

## 11. Never Remove Focus Visibility Without Replacement

Avoid:

```css
outline: none;
```

unless you provide an equally clear focus indicator.

Keyboard users need to know:

```text
Where am I?
```

---

## 12. Focus Management

React refs can manage focus when the UI structure changes.

Example:

```jsx
const headingRef =
  useRef(null);

function handleComplete() {
  setComplete(true);

  // focus after the
  // new UI is committed
}
```

Focus management is useful for:

- dialogs
- validation failures
- route/view changes where appropriate
- dynamically inserted content
- restoring focus after overlays close

Do not move focus unexpectedly.

---

# Part 6 — Dialogs and Overlays

## 13. Accessible Modal Requirements

A modal generally needs:

- dialog semantics
- accessible name
- focus moved into it
- keyboard navigation contained appropriately
- Escape behavior when applicable
- focus restored to the trigger after closing
- background interaction handled correctly

Using a well-tested accessible primitive is often safer than implementing all dialog behavior manually.

---

# Part 7 — Dynamic Updates

## 14. Visual Updates May Need Announcement

Suppose:

```text
Application saved successfully
```

appears dynamically.

A screen-reader user may need notification.

An appropriate live region can help:

```jsx
<p
  role="status"
  aria-live="polite"
>
  {message}
</p>
```

Do not announce every tiny UI change.

Use live regions intentionally.

---

## 15. Errors May Need Stronger Attention

For urgent error feedback, semantics such as an alert may be appropriate:

```jsx
<div role="alert">
  Unable to save application.
</div>
```

Avoid overusing assertive announcements.

---

# Part 8 — Loading States

## 16. Communicate Busy Regions

```jsx
<section
  aria-busy={isLoading}
>
  ...
</section>
```

Also provide understandable visual/text feedback.

Do not rely only on a spinning animation.

---

# Part 9 — Color and Visual Meaning

## 17. Do Not Use Color Alone

Bad:

```text
red = rejected
green = accepted
```

Better:

```text
● Rejected
✓ Accepted
```

with sufficient visual contrast.

Users may not perceive color differences as expected.

---

# Part 10 — Motion

## 18. Respect Reduced Motion

Animations can create discomfort for some users.

CSS can respond to:

```css
@media (
  prefers-reduced-motion:
    reduce
) {
  /* reduce nonessential motion */
}
```

Do not make essential information depend only on animation.

---

# Part 11 — ARIA

## 19. First Rule: Prefer Native HTML

ARIA can add semantics when native HTML is insufficient.

It does not automatically add:

- keyboard behavior
- focus management
- event handling

A div with:

```jsx
role="button"
```

does not magically become a fully functional native button.

---

## 20. Common ARIA Attributes

Examples:

```text
aria-label
aria-labelledby
aria-describedby
aria-expanded
aria-controls
aria-current
aria-invalid
aria-busy
aria-live
```

Use ARIA according to the semantic relationship you actually need.

Incorrect ARIA can make accessibility worse.

---

# Part 12 — Conditional Rendering

## 21. Hidden vs Removed

Conditional rendering:

```jsx
{isOpen && (
  <Menu />
)}
```

removes the subtree when false.

CSS hiding and semantic hiding have different effects depending on technique.

When building interactive widgets, understand whether hidden content remains:

- focusable
- available to assistive technology
- present in the DOM

---

# Part 13 — Lists

## 22. Use List Semantics

Instead of:

```jsx
<div>
  <div>React</div>
  <div>Node</div>
</div>
```

when the content is truly a list:

```jsx
<ul>
  <li>React</li>
  <li>Node</li>
</ul>
```

Semantic HTML gives structure for free.

---

# Part 14 — Links vs Buttons

## 23. Choose by Behavior

Use a link when the action navigates:

```jsx
<a href="/profile">
  View profile
</a>
```

Use a button when it performs an action:

```jsx
<button
  onClick={follow}
>
  Follow
</button>
```

Do not style one as the other purely because of appearance.

---

# Part 15 — Disabled Controls

## 24. Disabled UX

A disabled button may not be focusable and therefore may make explanatory information harder to discover.

Sometimes it is better to:

- keep the action available
- validate on activation
- explain what is missing

The right behavior depends on the interaction.

Do not disable controls without considering how the user learns why.

---

# Part 16 — React Keys and Accessibility

## 25. Stable Identity Helps Interaction

Unstable keys can cause DOM nodes to be replaced unexpectedly.

That can affect:

- focus
- input state
- assistive-technology context

Stable keys are not only a performance/reconciliation concern.

They support predictable UI identity.

---

# Part 17 — Accessible Components

## 26. Build Accessibility Into Reusable Primitives

If your application has one reusable:

```text
Button
Dialog
Input
Select
Tooltip
```

correct accessibility in that primitive benefits every feature.

This is a major reason to use well-designed UI primitives.

---

# Part 18 — CodeBuddy Example

## 27. Developer Card

Actions should be understandable:

```jsx
<button
  aria-label={
    `Connect with ${name}`
  }
>
  <ConnectIcon
    aria-hidden="true"
  />
</button>
```

If a swipe interaction exists, provide equivalent accessible controls such as:

```text
Pass
Connect
```

Do not make drag/swipe the only way to perform an essential action.

---

# Part 19 — CareerLoop Example

## 28. Application Status

Do not expose status only by color.

Better:

```text
Interview — blue badge
Rejected — red badge
Offer — green badge
```

The text communicates meaning even when color does not.

---

# Part 20 — Testing Accessibility

## 29. Manual Keyboard Test

Before shipping:

```text
1. Put mouse aside.
2. Tab through interface.
3. Can every action be reached?
4. Is focus visible?
5. Is focus order logical?
6. Can overlays be opened/closed?
7. Does focus return appropriately?
```

---

## 30. Automated Checks

Automated accessibility tools can catch many issues such as:

- missing labels
- invalid ARIA
- contrast problems in some tooling
- structural issues

But automated testing cannot prove that the experience is accessible.

Manual keyboard and assistive-technology testing remain valuable.

---

# Part 21 — Common Mistakes

## 31. Common Mistakes

1. Using clickable divs instead of native controls.
2. Removing focus outlines.
3. Using placeholders instead of labels.
4. Using color as the only status signal.
5. Giving icon buttons no accessible name.
6. Writing unnecessary/redundant alt text.
7. Adding ARIA when native HTML already solves the problem.
8. Adding ARIA roles without implementing keyboard behavior.
9. Forgetting focus management for dialogs.
10. Moving focus unexpectedly.
11. Making drag/swipe the only interaction.
12. Forgetting dynamic announcements where needed.
13. Making animations mandatory.
14. Using unstable keys that unexpectedly replace focused nodes.
15. Assuming automated tools guarantee accessibility.

---

# Part 22 — Interview Questions

## 32. How Do You Make React Apps Accessible?

Start with semantic HTML, native interactive controls, labels, keyboard support, visible focus, accessible names, proper focus management and meaningful feedback. Add ARIA only where native semantics are insufficient.

---

## 33. Why Prefer button Over div onClick?

A native button already provides expected semantics, keyboard interaction and focus behavior.

---

## 34. What Is aria-describedby Used For?

To associate an element with additional descriptive content, commonly help text or validation errors.

---

## 35. Link vs Button?

A link navigates to a resource/location.

A button performs an action.

---

## 36. Can Automated Tests Guarantee Accessibility?

No.

They are useful for detecting classes of issues, but manual keyboard testing and real assistive-technology evaluation are also important.

---

# Part 23 — Mental Model

## 37. Accessibility

```text
               ACCESSIBLE REACT UI
                       │
       ┌───────────────┼───────────────┐
       ↓               ↓               ↓
 semantic HTML     keyboard/focus   communication
       │               │               │
 native controls   visible focus    labels/names
 headings/lists    logical order     errors/status
 forms/labels      modal focus       live updates
       │               │               │
       └───────────────┼───────────────┘
                       ↓
                 ARIA only when
                 semantics need it
                       ↓
                 test with users'
                 interaction modes
```

---

## 38. Key Takeaways

- Accessibility starts with the web platform, not React-specific APIs.
- Prefer semantic HTML and native controls.
- Every interactive control needs an understandable accessible name.
- Labels and field errors need correct relationships.
- Interfaces must work with keyboard navigation.
- Keep focus visible.
- Manage focus intentionally when UI structure changes.
- Dialogs require careful focus and keyboard behavior.
- Dynamic updates may need accessible announcements.
- Do not communicate meaning with color alone.
- Respect reduced-motion preferences.
- ARIA supplements semantics; it does not replace behavior.
- Use links for navigation and buttons for actions.
- Stable component/DOM identity supports predictable focus.
- Build accessibility into reusable UI primitives.
- Provide alternatives to gesture-only interactions.
- Automated accessibility checks are useful but incomplete.
- Test important flows manually with a keyboard.

---

## Next Lesson

➡️ [Lesson 71 — React Security Fundamentals ⭐⭐⭐⭐⭐](./71-security.md)
