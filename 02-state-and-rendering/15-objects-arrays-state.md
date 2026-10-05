# Lesson 15 — Updating Objects and Arrays in State

## 1. Why This Topic Matters

React state often contains more than numbers and strings.

Real applications frequently store:

```js
const user = {
  name: "Vikash",
  role: "Developer",
};
```

or:

```js
const applications = [
  { id: 1, company: "Google", status: "Applied" },
  { id: 2, company: "Microsoft", status: "Interview" },
];
```

The important React rule is:

> **Treat objects and arrays stored in state as immutable.**

Instead of changing the existing object or array, create a new one and pass that new value to the state setter.

---

## 2. Primitive State vs Object State

Primitive state is straightforward:

```jsx
const [count, setCount] = useState(0);

setCount(1);
```

For objects:

```jsx
const [user, setUser] = useState({
  name: "Vikash",
  role: "Developer",
});
```

Do not do:

```jsx
user.name = "Aman";
```

Instead create a new object:

```jsx
setUser({
  ...user,
  name: "Aman",
});
```

---

## 3. Why Mutation Is a Problem ⭐⭐⭐⭐⭐

Consider:

```jsx
const [user, setUser] = useState({
  name: "Vikash",
});

function handleChange() {
  user.name = "Aman";

  setUser(user);
}
```

The reference remains the same:

```text
Before:

user ───────→ Object A
               name: Vikash

After mutation:

user ───────→ Object A
               name: Aman
```

You changed the contents of the existing object.

React state should instead be treated as a snapshot.

A better update is:

```jsx
setUser({
  ...user,
  name: "Aman",
});
```

Now:

```text
Old state ─────→ Object A
                  name: Vikash

New state ─────→ Object B
                  name: Aman
```

The old state remains untouched.

---

## 4. State Should Be Treated as Read-Only ⭐⭐⭐⭐⭐

Although JavaScript objects are technically mutable, React state should be treated as read-only.

Think:

```text
Current state
     ↓
READ ONLY
     ↓
create new value
     ↓
setState(newValue)
```

Do not think:

```text
Current state
     ↓
modify directly
     ↓
reuse same reference
```

---

## 5. Updating an Object with Spread Syntax ⭐⭐⭐⭐⭐

Suppose:

```jsx
const [profile, setProfile] = useState({
  name: "Vikash",
  role: "Frontend Developer",
  city: "Bengaluru",
});
```

To change only `role`:

```jsx
setProfile({
  ...profile,
  role: "Full Stack Developer",
});
```

The spread syntax copies existing properties:

```text
...profile

name: Vikash
role: Frontend Developer
city: Bengaluru
```

Then:

```js
role: "Full Stack Developer"
```

overwrites the copied `role`.

Result:

```js
{
  name: "Vikash",
  role: "Full Stack Developer",
  city: "Bengaluru"
}
```

---

## 6. Spread Order Matters

Correct:

```jsx
setProfile({
  ...profile,
  role: "Full Stack Developer",
});
```

Here the new role wins.

But:

```jsx
setProfile({
  role: "Full Stack Developer",
  ...profile,
});
```

the old `profile.role` can overwrite the new value.

Remember:

```text
later property wins
```

---

## 7. Prefer Functional Updates When Based on Previous State ⭐⭐⭐⭐⭐

Instead of:

```jsx
setProfile({
  ...profile,
  role: "Full Stack Developer",
});
```

when the update logically depends on the previous state, prefer:

```jsx
setProfile((profile) => ({
  ...profile,
  role: "Full Stack Developer",
}));
```

This is especially important when multiple updates can be queued.

Mental model:

```text
previous pending state
        ↓
updater function
        ↓
new object
        ↓
next state
```

---

## 8. Updating State from Input Fields

```jsx
function ProfileForm() {
  const [profile, setProfile] = useState({
    name: "",
    email: "",
  });

  return (
    <>
      <input
        value={profile.name}
        onChange={(event) =>
          setProfile((profile) => ({
            ...profile,
            name: event.target.value,
          }))
        }
      />

      <input
        value={profile.email}
        onChange={(event) =>
          setProfile((profile) => ({
            ...profile,
            email: event.target.value,
          }))
        }
      />
    </>
  );
}
```

Every update:

1. copies the previous object
2. replaces one property
3. returns a new object

---

## 9. Dynamic Property Names

For larger forms:

```jsx
function handleChange(event) {
  const { name, value } = event.target;

  setProfile((profile) => ({
    ...profile,
    [name]: value,
  }));
}
```

Inputs:

```jsx
<input
  name="name"
  value={profile.name}
  onChange={handleChange}
/>

<input
  name="email"
  value={profile.email}
  onChange={handleChange}
/>
```

If:

```text
name = "email"
```

then:

```js
[name]: value
```

becomes conceptually:

```js
email: value
```

---

## 10. Nested Objects ⭐⭐⭐⭐⭐

Suppose:

```jsx
const [user, setUser] = useState({
  name: "Vikash",
  address: {
    city: "Bengaluru",
    country: "India",
  },
});
```

Wrong:

```jsx
user.address.city = "Pune";

setUser(user);
```

You mutated nested state.

Correct:

```jsx
setUser((user) => ({
  ...user,

  address: {
    ...user.address,
    city: "Pune",
  },
}));
```

You need to copy each changed level.

---

## 11. Nested Update Diagram ⭐⭐⭐⭐⭐

Original:

```text
user
 │
 ├── name
 │
 └── address ───→ Address A
                    ├── city
                    └── country
```

Update `city`:

```text
New User Object
 │
 ├── name
 │
 └── New Address Object
          ├── city: Pune
          └── country: India
```

The rule is:

> Create new objects along the path you are changing.

Unchanged values can still be shared safely.

---

## 12. Spread Syntax Is Shallow ⭐⭐⭐⭐⭐

This is important:

```js
const copy = {
  ...user,
};
```

Spread performs a **shallow copy**.

If:

```js
user.address
```

is an object, then initially:

```text
user.address ──────┐
                   ↓
                Object A
                   ↑
copy.address ──────┘
```

Both objects still reference the same nested address.

Therefore this is still wrong:

```jsx
const nextUser = {
  ...user,
};

nextUser.address.city = "Pune";
```

You have mutated the shared nested object.

---

## 13. Avoid Excessively Deep State

If updating state repeatedly requires:

```jsx
setState((state) => ({
  ...state,
  user: {
    ...state.user,
    profile: {
      ...state.user.profile,
      address: {
        ...state.user.profile.address,
        city: "Pune",
      },
    },
  },
}));
```

the state shape may be too deeply nested.

Consider whether the data can be:

- flattened
- split into separate state variables
- normalized
- managed with a reducer when complexity justifies it

State structure affects maintainability.

---

# Arrays in State

## 14. Arrays Must Also Be Treated as Immutable ⭐⭐⭐⭐⭐

Suppose:

```jsx
const [skills, setSkills] = useState([
  "React",
  "JavaScript",
]);
```

Do not mutate:

```jsx
skills.push("Node.js");

setSkills(skills);
```

Instead create a new array:

```jsx
setSkills((skills) => [
  ...skills,
  "Node.js",
]);
```

---

## 15. Mutation vs Non-Mutation Array Methods ⭐⭐⭐⭐⭐

Common mutating methods:

```text
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```

Be careful using them directly on state arrays.

Useful non-mutating patterns:

```text
spread [...]
map()
filter()
slice()
concat()
toSorted()
toReversed()
```

The goal is not merely memorizing method names.

The rule is:

> Do not mutate the existing state array.

---

## 16. Adding an Item

Existing:

```jsx
const [skills, setSkills] = useState([
  "React",
  "JavaScript",
]);
```

Add to end:

```jsx
setSkills((skills) => [
  ...skills,
  "Node.js",
]);
```

Result:

```js
[
  "React",
  "JavaScript",
  "Node.js"
]
```

---

## 17. Adding to the Beginning

```jsx
setSkills((skills) => [
  "HTML",
  ...skills,
]);
```

Result:

```text
HTML
React
JavaScript
```

---

## 18. Removing an Item with filter() ⭐⭐⭐⭐⭐

```jsx
const [applications, setApplications] = useState([
  { id: 1, company: "Google" },
  { id: 2, company: "Microsoft" },
]);
```

Remove application 1:

```jsx
setApplications((applications) =>
  applications.filter(
    (application) => application.id !== 1
  )
);
```

`filter()` returns a new array.

---

## 19. Updating an Item with map() ⭐⭐⭐⭐⭐

Suppose:

```jsx
const [applications, setApplications] = useState([
  {
    id: 1,
    company: "Google",
    status: "Applied",
  },
  {
    id: 2,
    company: "Microsoft",
    status: "Applied",
  },
]);
```

Update application 2:

```jsx
setApplications((applications) =>
  applications.map((application) =>
    application.id === 2
      ? {
          ...application,
          status: "Interview",
        }
      : application
  )
);
```

Mental model:

```text
map each item
     ↓
Is this the target?
     │
 ┌───┴────┐
 yes      no
  ↓        ↓
new item  existing item
     │
     └────┬────┘
          ↓
      new array
```

---

## 20. Why Returning Existing Unchanged Items Is Fine

In:

```jsx
application.id === 2
  ? { ...application, status: "Interview" }
  : application
```

the unchanged items can keep their existing references.

You only need a new object for an item that changes.

Conceptually:

```text
Old array
├── Object A
├── Object B
└── Object C

New array
├── Object A       ← unchanged, shared
├── Object B2      ← changed, new
└── Object C       ← unchanged, shared
```

This is called **structural sharing**.

It avoids unnecessary copying while preserving immutability.

---

## 21. Replacing an Item

You can use `map()`:

```jsx
setApplications((applications) =>
  applications.map((application) =>
    application.id === updatedApplication.id
      ? updatedApplication
      : application
  )
);
```

This creates a new array while replacing only the target element.

---

## 22. Inserting at a Specific Position

Suppose:

```jsx
const nextSkill = "TypeScript";
const insertAt = 1;
```

You can use `slice()`:

```jsx
setSkills((skills) => [
  ...skills.slice(0, insertAt),
  nextSkill,
  ...skills.slice(insertAt),
]);
```

If:

```text
React
JavaScript
Node.js
```

the result can be:

```text
React
TypeScript
JavaScript
Node.js
```

without mutating the original array.

---

## 23. Sorting Arrays Safely ⭐⭐⭐⭐⭐

This is dangerous:

```jsx
applications.sort(
  (a, b) => a.company.localeCompare(b.company)
);
```

`sort()` mutates the array.

Safer:

```jsx
const sortedApplications = [
  ...applications,
].sort((a, b) =>
  a.company.localeCompare(b.company)
);
```

Or with modern JavaScript:

```jsx
const sortedApplications =
  applications.toSorted((a, b) =>
    a.company.localeCompare(b.company)
  );
```

`toSorted()` returns a new array.

---

## 24. Reversing Arrays Safely

Avoid:

```jsx
applications.reverse();
```

Safer:

```jsx
const reversed = [
  ...applications,
].reverse();
```

Or:

```jsx
const reversed =
  applications.toReversed();
```

Again:

> Do not mutate the state array itself.

---

## 25. Nested Objects Inside Arrays ⭐⭐⭐⭐⭐

This is a common real-world case:

```jsx
const [developers, setDevelopers] = useState([
  {
    id: 1,
    name: "Vikash",
    profile: {
      connected: false,
    },
  },
]);
```

To change `connected`:

```jsx
setDevelopers((developers) =>
  developers.map((developer) =>
    developer.id === 1
      ? {
          ...developer,
          profile: {
            ...developer.profile,
            connected: true,
          },
        }
      : developer
  )
);
```

Notice the copies:

```text
new array
   ↓
new target developer object
   ↓
new target profile object
```

Every changed level gets a new reference.

---

## 26. Array Copy Alone Is Not Enough ⭐⭐⭐⭐⭐

Suppose:

```jsx
const nextDevelopers = [...developers];

nextDevelopers[0].name = "Aman";
```

You created a new array, but its objects are still shared.

```text
developers[0] ───────┐
                     ↓
                  Object A
                     ↑
nextDevelopers[0] ───┘
```

Changing `nextDevelopers[0]` mutates the object inside the original state too.

Correct:

```jsx
const nextDevelopers = developers.map(
  (developer) =>
    developer.id === targetId
      ? {
          ...developer,
          name: "Aman",
        }
      : developer
);
```

---

## 27. Real-World Example: Career Application Status

```jsx
function Applications() {
  const [applications, setApplications] = useState([
    {
      id: 1,
      company: "ABC Tech",
      status: "Applied",
    },
    {
      id: 2,
      company: "XYZ Labs",
      status: "Screening",
    },
  ]);

  function updateStatus(id, status) {
    setApplications((applications) =>
      applications.map((application) =>
        application.id === id
          ? {
              ...application,
              status,
            }
          : application
      )
    );
  }

  return applications.map((application) => (
    <article key={application.id}>
      <h2>{application.company}</h2>
      <p>{application.status}</p>

      <button
        onClick={() =>
          updateStatus(
            application.id,
            "Interview"
          )
        }
      >
        Move to Interview
      </button>
    </article>
  ));
}
```

Flow:

```text
User clicks
     ↓
updateStatus(id)
     ↓
functional state updater
     ↓
map old array
     ↓
create new target object
     ↓
return new array
     ↓
React receives new state
     ↓
render
```

---

## 28. Real-World Example: Delete Application

```jsx
function deleteApplication(id) {
  setApplications((applications) =>
    applications.filter(
      (application) => application.id !== id
    )
  );
}
```

Simple mental model:

```text
ADD
  → spread

REMOVE
  → filter

UPDATE
  → map
```

This is worth memorizing.

---

## 29. Real-World Example: Add Application

```jsx
function addApplication(newApplication) {
  setApplications((applications) => [
    ...applications,
    newApplication,
  ]);
}
```

CRUD-style array operations:

```text
Create → spread
Read   → find/filter
Update → map
Delete → filter
```

---

## 30. Objects Are Not Really "Nested" Internally

Consider:

```js
const address = {
  city: "Bengaluru",
};

const user = {
  name: "Vikash",
  address,
};
```

A useful reference model is:

```text
user.address ─────┐
                  ↓
              Address Object
                  ↑
address ──────────┘
```

Objects can reference other objects.

This is why mutating a supposedly "nested" object can affect multiple references.

Understanding references is more accurate than imagining everything as independent boxes inside boxes.

---

## 31. Why Immutability Helps React ⭐⭐⭐⭐⭐

Immutability provides several benefits.

### Predictable snapshots

Old render values are not unexpectedly changed.

### Easier change detection

References can help determine whether data changed.

```text
oldObject !== newObject
```

### Easier debugging

Previous state remains understandable.

### Memoization

APIs such as `React.memo`, `useMemo`, and `useCallback` often rely on stable reference behavior.

### Concurrent React

Avoiding mutation makes rendering safer when React manages work across renders.

---

## 32. Object Identity ⭐⭐⭐⭐⭐

JavaScript:

```js
const a = {
  name: "React",
};

const b = {
  name: "React",
};

console.log(a === b); // false
```

Although their contents look identical, they are different objects.

```text
a ───→ Object A

b ───→ Object B
```

But:

```js
const c = a;

console.log(a === c); // true
```

because:

```text
a ───┐
     ↓
  Object A
     ↑
c ───┘
```

Reference identity matters heavily in React performance and state updates.

---

## 33. Should You Deep Clone the Entire State?

Usually, no.

You normally copy only the changed path.

For:

```js
{
  profile: {
    address: {
      city: "Bengaluru"
    }
  },
  settings: {
    theme: "dark"
  }
}
```

If only city changes, you need new references along that changed path.

You do not need to manually recreate every unrelated nested object.

This preserves structural sharing.

---

## 34. JSON Deep Cloning Is Usually a Bad State Strategy

Avoid using:

```js
JSON.parse(
  JSON.stringify(state)
);
```

as a general React update strategy.

Problems include:

- unnecessary work
- loss of some JavaScript value types
- poor intent/readability
- copying far more than necessary

Use explicit immutable updates or an appropriate state-management helper when complexity justifies it.

---

## 35. Immer Preview

Libraries such as **Immer** can make deeply nested immutable updates easier.

Conceptually, code can look mutation-like:

```js
draft.user.address.city = "Pune";
```

while Immer produces an immutable next state.

This can be useful in reducers or complex state structures.

But first understand manual immutability.

If you do not understand references and immutable updates, Immer can hide important React fundamentals.

---

## 36. Avoid Redundant State

Suppose:

```jsx
const [items, setItems] = useState([]);
const [itemCount, setItemCount] = useState(0);
```

If count always equals:

```js
items.length
```

you may not need separate state.

Instead:

```jsx
const itemCount = items.length;
```

This avoids keeping two values synchronized.

Before updating complex objects or arrays, ask:

> Does this value need to be state at all?

---

## 37. State Shape Matters

Instead of:

```jsx
const [user, setUser] = useState({
  form: {
    preferences: {
      ui: {
        theme: {
          mode: "dark",
        },
      },
    },
  },
});
```

consider whether your application really needs such deep nesting.

Good state design reduces update complexity.

Possible strategies:

```text
deep nested state
       ↓
consider
       ↓
flattening
splitting state
normalization
useReducer
external state when justified
```

---

## 38. Common Mistakes

### Mistake 1 — Mutating object state

```jsx
user.name = "Aman";
```

### Mistake 2 — Mutating array state

```jsx
items.push(newItem);
```

### Mistake 3 — Using sort directly on state

```jsx
items.sort();
```

### Mistake 4 — Copying only the outer object

```jsx
const nextUser = {
  ...user,
};

nextUser.address.city = "Pune";
```

The nested object is still shared.

### Mistake 5 — Copying the array but mutating an item

```jsx
const next = [...items];

next[0].name = "Changed";
```

The item object can still be shared.

### Mistake 6 — Deep cloning everything

Usually unnecessary and inefficient.

### Mistake 7 — Storing derived information separately

This creates synchronization problems.

### Mistake 8 — Making state unnecessarily deeply nested

Complex state shapes produce complex updates.

---

## 39. Interview Questions ⭐⭐⭐⭐⭐

### Q1. Why should React state not be mutated directly?

**Answer:** State should behave as an immutable render snapshot. Creating new references makes updates predictable and works well with React's rendering and equality model.

### Q2. How do you update one property in an object state?

```jsx
setUser((user) => ({
  ...user,
  name: "Aman",
}));
```

### Q3. How do you update a nested object?

**Answer:** Create new objects at every changed level.

```jsx
setUser((user) => ({
  ...user,
  address: {
    ...user.address,
    city: "Pune",
  },
}));
```

### Q4. Is spread syntax a deep copy?

**Answer:** No. Object and array spread are shallow copies.

### Q5. How do you add an item to array state?

```jsx
setItems((items) => [
  ...items,
  newItem,
]);
```

### Q6. How do you remove an item?

```jsx
setItems((items) =>
  items.filter((item) => item.id !== id)
);
```

### Q7. How do you update one item?

```jsx
setItems((items) =>
  items.map((item) =>
    item.id === id
      ? { ...item, status: "Done" }
      : item
  )
);
```

### Q8. Why is `sort()` dangerous with state arrays?

**Answer:** `sort()` mutates the original array. Copy first or use `toSorted()`.

### Q9. If you spread an array, are its objects also copied?

**Answer:** No. The array is new, but object elements still reference the same objects unless those objects are copied separately.

### Q10. What is structural sharing?

**Answer:** Creating new references only for changed parts while safely reusing unchanged references.

### Q11. Why does immutability help memoization?

**Answer:** Reference equality can efficiently indicate whether values changed. Mutation can keep the same reference even when internal data changes.

### Q12. Should you deep clone state on every update?

**Answer:** Usually no. Copy only the changed path and preserve unchanged references.

---

## 40. Interview Scenario ⭐⭐⭐⭐⭐

What is wrong here?

```jsx
function updateStatus(id) {
  const nextApplications = [
    ...applications,
  ];

  const application =
    nextApplications.find(
      (application) => application.id === id
    );

  application.status = "Interview";

  setApplications(nextApplications);
}
```

At first glance the array was copied.

But:

```text
applications
      │
      └── Object A

nextApplications
      │
      └── Object A
```

The target application object is still shared.

So:

```js
application.status = "Interview";
```

mutates an object belonging to the previous state.

Correct:

```jsx
setApplications((applications) =>
  applications.map((application) =>
    application.id === id
      ? {
          ...application,
          status: "Interview",
        }
      : application
  )
);
```

---

## 41. CRUD Cheat Sheet ⭐⭐⭐⭐⭐

### Add

```jsx
setItems((items) => [
  ...items,
  newItem,
]);
```

### Remove

```jsx
setItems((items) =>
  items.filter((item) => item.id !== id)
);
```

### Update

```jsx
setItems((items) =>
  items.map((item) =>
    item.id === id
      ? { ...item, ...changes }
      : item
  )
);
```

### Replace

```jsx
setItems((items) =>
  items.map((item) =>
    item.id === replacement.id
      ? replacement
      : item
  )
);
```

### Sort

```jsx
const sorted = items.toSorted(compareFn);
```

or:

```jsx
const sorted = [...items].sort(compareFn);
```

---

## 42. Complete Mental Model ⭐⭐⭐⭐⭐

```text
CURRENT STATE
      │
      │ do not mutate
      ↓
READ EXISTING VALUE
      │
      ↓
CREATE NEW OBJECT / ARRAY
      │
      ├── reuse unchanged references
      │
      └── create new changed references
      ↓
setState(newValue)
      │
      ↓
NEXT RENDER
```

For nested data:

```text
Changed path:

new root
   ↓
new changed child
   ↓
new changed nested child

Unchanged branches:
reuse safely
```

---

## 43. Quick Revision

Objects:

```jsx
setUser((user) => ({
  ...user,
  name: "Aman",
}));
```

Nested objects:

```jsx
setUser((user) => ({
  ...user,
  address: {
    ...user.address,
    city: "Pune",
  },
}));
```

Arrays:

```text
Add    → spread
Remove → filter
Update → map
Sort   → toSorted or copy + sort
```

Remember:

```text
spread = shallow copy
```

and:

```text
new array
≠
all nested objects automatically copied
```

---

## 44. Key Takeaways

- Treat React state objects and arrays as immutable.
- Do not directly mutate objects stored in state.
- Do not use mutating array methods directly on state.
- Use object spread to create updated object copies.
- Spread syntax performs only a shallow copy.
- For nested updates, create new objects along the changed path.
- Use array spread for adding items.
- Use `filter()` for removing items.
- Use `map()` for updating or replacing items.
- Copy before using mutating operations such as `sort()`, or use non-mutating alternatives such as `toSorted()`.
- A copied array can still contain references to the original item objects.
- Only changed objects need new references; unchanged objects can be structurally shared.
- Prefer functional updater syntax when the next state depends on previous state.
- Immutability supports predictable state snapshots, debugging, memoization, and modern React rendering.
- Avoid unnecessary deep cloning.
- Avoid redundant and excessively nested state.
- Good state structure makes immutable updates much easier.

---

## Next Lesson

➡️ [Lesson 16 — Controlled vs Uncontrolled Components ⭐⭐⭐⭐⭐](./16-controlled-uncontrolled.md)
