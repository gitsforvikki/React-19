# Lesson 07 — Rendering Lists and Understanding Keys ⭐⭐⭐⭐⭐

## 1. Why List Rendering Matters

Real React applications frequently render collections:

- products
- users
- messages
- notifications
- orders
- comments
- skills
- job applications

Instead of manually writing repeated JSX, we normally transform data into React elements.

```text
Data Array
    ↓
  map()
    ↓
React Elements
    ↓
Rendered List
```

---

## 2. Basic List Rendering with map()

Given:

```js
const skills = ["React", "JavaScript", "Node.js"];
```

Render them:

```jsx
function SkillList() {
  return (
    <ul>
      {skills.map((skill) => (
        <li key={skill}>{skill}</li>
      ))}
    </ul>
  );
}
```

JavaScript's `map()` transforms every item into a React element.

Conceptually:

```text
["React", "JavaScript", "Node.js"]
              ↓
             map
              ↓
[
  <li>React</li>,
  <li>JavaScript</li>,
  <li>Node.js</li>
]
              ↓
            React
              ↓
             UI
```

---

## 3. Rendering Object Arrays

Real data usually consists of objects:

```js
const developers = [
  {
    id: "dev-101",
    name: "Vikash",
    role: "Frontend Developer",
  },
  {
    id: "dev-102",
    name: "Aman",
    role: "Backend Developer",
  },
];
```

Render:

```jsx
function DeveloperList() {
  return (
    <ul>
      {developers.map((developer) => (
        <li key={developer.id}>
          <h2>{developer.name}</h2>
          <p>{developer.role}</p>
        </li>
      ))}
    </ul>
  );
}
```

The stable database/application ID is an excellent key.

---

## 4. Extracting a List Item Component

As an item becomes complex, extract it:

```jsx
function DeveloperCard({ developer }) {
  return (
    <article>
      <h2>{developer.name}</h2>
      <p>{developer.role}</p>
    </article>
  );
}

function DeveloperList({ developers }) {
  return (
    <section>
      {developers.map((developer) => (
        <DeveloperCard
          key={developer.id}
          developer={developer}
        />
      ))}
    </section>
  );
}
```

Notice where the key is placed:

```jsx
<DeveloperCard key={developer.id} />
```

The key belongs on the element directly created inside the `map()`.

---

## 5. What Is a key? ⭐⭐⭐⭐⭐

A `key` is a special value React uses to identify items among siblings.

```jsx
{developers.map((developer) => (
  <DeveloperCard
    key={developer.id}
    developer={developer}
  />
))}
```

Think:

```text
Previous List

key=a → Alice
key=b → Bob
key=c → Carol

New List

key=b → Bob
key=c → Carol
key=d → David
```

Keys help React understand:

- `a` disappeared
- `b` is still Bob
- `c` is still Carol
- `d` is new

Keys represent **identity among siblings**.

---

## 6. Why React Needs Keys ⭐⭐⭐⭐⭐

Suppose:

```text
Before
A
B
C

After
X
A
B
C
```

React needs a way to match old children with new children.

With stable keys:

```text
Before
key=A → A
key=B → B
key=C → C

After
key=X → X
key=A → A
key=B → B
key=C → C
```

React can identify the existing items even though their positions moved.

This matters for both:

- efficient reconciliation
- preserving the correct component/DOM state

---

## 7. Keys and Reconciliation ⭐⭐⭐⭐⭐

During reconciliation, React compares the previous and next UI trees.

For list children, keys help React match corresponding siblings.

```text
Previous Children
┌───────┬───────┬───────┐
│ key A │ key B │ key C │
└───────┴───────┴───────┘
              ↓
         reconciliation
              ↓
Next Children
┌───────┬───────┬───────┐
│ key B │ key C │ key D │
└───────┴───────┴───────┘
```

React can reason:

```text
A → removed
B → preserved/moved
C → preserved/moved
D → created
```

We will revisit the internals in the dedicated reconciliation lesson.

---

## 8. What Makes a Good Key?

A good key should be:

### Unique among siblings

```jsx
key={developer.id}
```

### Stable

The same logical item should keep the same key between renders.

### Derived from the data

Prefer IDs already belonging to the data.

Good examples:

```jsx
key={user.id}
key={product.id}
key={message.id}
key={order.id}
```

---

## 9. Keys Need Only Be Unique Among Siblings

Keys do not need to be globally unique across the entire application.

This is valid:

```jsx
function UserList({ users }) {
  return users.map((user) => (
    <UserCard key={user.id} user={user} />
  ));
}

function AdminList({ admins }) {
  return admins.map((admin) => (
    <AdminCard key={admin.id} admin={admin} />
  ));
}
```

The two lists are separate sibling sets.

Think:

```text
Keys identify items relative to their siblings.
```

---

## 10. Why Array Index Is Often a Bad Key ⭐⭐⭐⭐⭐

You may see:

```jsx
{users.map((user, index) => (
  <UserCard
    key={index}
    user={user}
  />
))}
```

This is dangerous when items can:

- be inserted
- be deleted
- be reordered
- be filtered
- be sorted

Why?

Because the index describes **position**, not item identity.

---

## 11. Index Key Bug Example ⭐⭐⭐⭐⭐

Suppose:

```text
Index 0 → Alice
Index 1 → Bob
Index 2 → Carol
```

Keys:

```text
key=0 → Alice
key=1 → Bob
key=2 → Carol
```

Now remove Alice.

The new array becomes:

```text
Index 0 → Bob
Index 1 → Carol
```

But React sees:

```text
key=0 existed before
key=1 existed before
```

The identity mapping has effectively changed:

```text
Before:
key=0 → Alice
key=1 → Bob

After:
key=0 → Bob
key=1 → Carol
```

This can cause component state or DOM state to become associated with the wrong logical item.

---

## 12. Stateful List Example

Imagine each row has an input:

```jsx
function UserRow({ user }) {
  const [note, setNote] = useState("");

  return (
    <div>
      <span>{user.name}</span>

      <input
        value={note}
        onChange={(event) => setNote(event.target.value)}
      />
    </div>
  );
}
```

Rendered with index keys:

```jsx
{users.map((user, index) => (
  <UserRow key={index} user={user} />
))}
```

A user types:

```text
Alice → "Frontend"
Bob   → "Backend"
Carol → ""
```

If Alice is removed, positions shift.

React may preserve the state associated with the position/key rather than the logical user you intended.

The visible result can look like:

```text
Bob   → "Frontend" ❌
Carol → "Backend"  ❌
```

This is why stable identity matters.

---

## 13. When Is an Index Key Acceptable?

An index can sometimes be acceptable when the list is truly static and identity does not matter independently.

For example, if:

- items never reorder
- items are never inserted/deleted
- items do not have meaningful stable IDs
- item state is not affected by position changes

Even then, if a natural stable identifier exists, prefer it.

Rule of thumb:

> Do not use the array index merely because it is convenient.

---

## 14. Never Generate Random Keys During Render ⭐⭐⭐⭐⭐

Bad:

```jsx
{users.map((user) => (
  <UserCard
    key={Math.random()}
    user={user}
  />
))}
```

Also problematic:

```jsx
key={crypto.randomUUID()}
```

if generated freshly during every render.

Why?

Every render creates different keys:

```text
Render 1:
A → key 123
B → key 456

Render 2:
A → key 891
B → key 234
```

React sees completely different identities.

This can cause components to be removed and recreated, losing local state and defeating useful reconciliation.

If an item needs an ID, generate/store it when the data item is created, not while rendering the list.

---

## 15. key Is Not Passed as a Normal Prop ⭐⭐⭐⭐⭐

Consider:

```jsx
<UserCard
  key={user.id}
  name={user.name}
/>
```

Inside:

```jsx
function UserCard(props) {
  console.log(props.key);
}
```

Do not expect `props.key` to contain the key.

`key` is a special React field used during reconciliation.

If the component needs the ID, pass it explicitly:

```jsx
<UserCard
  key={user.id}
  userId={user.id}
  name={user.name}
/>
```

Then:

```jsx
function UserCard({ userId, name }) {
  // use userId
}
```

---

## 16. Where Should the key Go? ⭐⭐⭐⭐⭐

Wrong placement:

```jsx
function UserCard({ user }) {
  return (
    <article key={user.id}>
      {user.name}
    </article>
  );
}

function UserList({ users }) {
  return users.map((user) => (
    <UserCard user={user} />
  ));
}
```

The list-producing code needs the key on the element being returned from the map:

```jsx
function UserList({ users }) {
  return users.map((user) => (
    <UserCard
      key={user.id}
      user={user}
    />
  ));
}
```

Rule:

> Put the key on the element/component directly created as part of the list.

---

## 17. Rendering Multiple Elements per Item

Suppose every item needs two sibling elements:

```jsx
users.map((user) => (
  <>
    <h2>{user.name}</h2>
    <p>{user.role}</p>
  </>
))
```

The short Fragment syntax cannot receive a key.

Use `Fragment`:

```jsx
import { Fragment } from "react";

users.map((user) => (
  <Fragment key={user.id}>
    <h2>{user.name}</h2>
    <p>{user.role}</p>
  </Fragment>
))
```

Now each grouped item has stable identity.

---

## 18. Filtering and Mapping

A common pattern:

```jsx
function ActiveUsers({ users }) {
  return (
    <ul>
      {users
        .filter((user) => user.isActive)
        .map((user) => (
          <li key={user.id}>
            {user.name}
          </li>
        ))}
    </ul>
  );
}
```

Flow:

```text
users
  ↓
filter active users
  ↓
map to JSX
  ↓
render
```

The key should still represent the original item's stable identity.

---

## 19. Sorting Lists Safely

Be careful with JavaScript's mutating methods.

This can mutate a prop/state array:

```js
users.sort(...)
```

Safer when the source should remain unchanged:

```js
const sortedUsers = [...users].sort((a, b) =>
  a.name.localeCompare(b.name)
);
```

Or use a non-mutating array method such as `toSorted()` when appropriate for your supported JavaScript environment:

```js
const sortedUsers = users.toSorted((a, b) =>
  a.name.localeCompare(b.name)
);
```

Then render:

```jsx
{sortedUsers.map((user) => (
  <UserCard key={user.id} user={user} />
))}
```

React state/props should be treated immutably.

---

## 20. Empty Lists

A good UI handles an empty collection.

```jsx
function UserList({ users }) {
  if (users.length === 0) {
    return <p>No users found.</p>;
  }

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

Production UIs should consider:

```text
Loading
Error
Empty
Success/List
```

These UI states are covered more deeply later.

---

## 21. Keys Are About Identity, Not Just Warnings ⭐⭐⭐⭐⭐

A beginner often thinks:

> I add a key only because React shows a console warning.

That misses the real purpose.

A key answers:

> Which new child corresponds to which previous child?

Think:

```text
key = identity hint among siblings
```

Correct keys help React preserve the right:

- component identity
- component state
- DOM nodes/state
- relationship between old and new children

---

## 22. Key + Type + Position Mental Model ⭐⭐⭐⭐⭐

React's reconciliation behavior depends on the structure of the tree.

For list children, a useful simplified mental model is:

```text
Component/Element Type
        +
       Key
        +
Position/Parent Context
        ↓
React determines identity
        ↓
Preserve, move, create, or remove
```

Do not treat this as the complete internal algorithm, but it is useful for reasoning about list identity.

---

## 23. Keys Can Intentionally Reset State ⭐⭐⭐⭐⭐

Keys are not only for arrays.

Changing a component's key can intentionally tell React:

> Treat this as a different component instance.

Example:

```jsx
<ProfileForm
  key={selectedUser.id}
  user={selectedUser}
/>
```

When `selectedUser.id` changes, React can treat the form as a new identity, resetting its local state.

This is powerful, but use it intentionally.

We will study this deeply in **Lesson 18 — Preserving and Resetting State**.

---

## 24. Bad Key Choices

### Array index for dynamic lists

```jsx
key={index}
```

Can break identity after reorder/insert/delete.

### Random value each render

```jsx
key={Math.random()}
```

Destroys stable identity.

### Non-unique value

```jsx
key={user.role}
```

Bad if multiple users have the same role.

### Changing value unrelated to item identity

A key should represent the logical item, not transient display state.

---

## 25. Good Key Choices

Best:

```jsx
key={user.id}
```

Also potentially valid if naturally unique and stable among siblings:

```jsx
key={country.code}
key={route.path}
key={databaseRecord.uuid}
```

For data created locally, create a stable ID when the record is created:

```js
const newTask = {
  id: crypto.randomUUID(),
  title: "Learn React keys",
};
```

Then later:

```jsx
<Task key={task.id} task={task} />
```

The important distinction is that the ID is stored with the data instead of regenerated during every render.

---

## 26. Real-World Example: Messages

```jsx
function MessageList({ messages }) {
  return (
    <div>
      {messages.map((message) => (
        <Message
          key={message.id}
          message={message}
        />
      ))}
    </div>
  );
}
```

Why `message.id`?

Messages can be:

- added
- deleted
- paginated
- reordered by data updates

A stable message ID continues to represent the same logical message.

---

## 27. Real-World Example: Job Applications

```jsx
function ApplicationList({ applications }) {
  return (
    <section>
      {applications.map((application) => (
        <ApplicationCard
          key={application.id}
          application={application}
        />
      ))}
    </section>
  );
}
```

Even if an application's status changes:

```text
Applied → Interview → Offer
```

its identity remains:

```text
application.id = same stable ID
```

The key should normally represent **who the item is**, not a property that changes over its lifetime.

---

## 28. Common Mistakes

### Mistake 1 — Forgetting key

Every element directly produced as part of a list needs an appropriate key.

### Mistake 2 — Using index automatically

Position is not stable identity when the list changes.

### Mistake 3 — Using random keys

Random keys force unstable identities.

### Mistake 4 — Putting key inside the extracted child

The key belongs where the list is constructed.

### Mistake 5 — Trying to read props.key

`key` is special to React and is not delivered as a normal prop.

### Mistake 6 — Mutating an array before rendering

Methods such as `sort()` mutate the original array.

### Mistake 7 — Using a non-unique field

A field like role/category may repeat.

### Mistake 8 — Thinking keys are only performance optimizations

Keys fundamentally help React track identity and preserve the correct state.

---

## 29. Interview Questions ⭐⭐⭐⭐⭐

### Q1. How do you render a list in React?

**Answer:** A common approach is to use JavaScript array methods such as `map()` to transform data items into React elements.

### Q2. What is a key in React?

**Answer:** A key is a special value that identifies a child among its siblings so React can match children between renders during reconciliation.

### Q3. Why are keys important?

**Answer:** Keys help React preserve the correct identity and state of list items when items are inserted, removed, or reordered.

### Q4. Why should we avoid array indexes as keys?

**Answer:** An index represents position rather than stable item identity. If the list changes order or items are inserted/deleted, state can become associated with the wrong logical item.

### Q5. When can an index key be acceptable?

**Answer:** It may be acceptable for a truly static list that never reorders, inserts, or removes items and has no better stable identity.

### Q6. Why is Math.random() a bad key?

**Answer:** It creates a new key every render, so React cannot match the new child with the previous one and may recreate components and lose local state.

### Q7. Is key available through props?

**Answer:** No. `key` is a special React field. If the child needs the same value, pass it separately as another prop.

### Q8. Where should a key be placed?

**Answer:** On the element or component directly created while rendering the list.

### Q9. Do keys need to be globally unique?

**Answer:** No. They need to be unique among siblings in the same list.

### Q10. Can keys be used outside lists?

**Answer:** Yes. A changed key can intentionally give a component a new identity and reset its local state.

### Q11. Is a key only a performance optimization?

**Answer:** No. Its primary conceptual role is identity during reconciliation. Correct identity also affects state preservation and DOM reuse.

### Q12. Should an ID be generated during render?

**Answer:** Not if it is being used as a stable list key. Generate/store the ID when the data item is created so it remains stable across renders.

---

## 30. Interview Scenario ⭐⭐⭐⭐⭐

### Question

You render editable todo rows:

```jsx
{todos.map((todo, index) => (
  <TodoItem key={index} todo={todo} />
))}
```

Users can insert, delete and reorder todos. Is this safe?

### Answer

No.

Because `index` represents the item's current position. When the list changes, the same index can refer to a different todo.

Use a stable ID:

```jsx
{todos.map((todo) => (
  <TodoItem
    key={todo.id}
    todo={todo}
  />
))}
```

Now:

```text
todo.id
   ↓
stable logical identity
   ↓
correct reconciliation
   ↓
correct state preservation
```

---

## 31. List Rendering Mental Model

```text
Data
 │
 ↓
filter / sort / transform
 │
 ↓
map()
 │
 ↓
React elements
 │
 ├── key=A
 ├── key=B
 └── key=C
 │
 ↓
Reconciliation
 │
 ↓
Correct identities preserved
 │
 ↓
UI
```

---

## 32. Quick Revision

```text
List Rendering
├── use map()
├── extract complex item components
└── provide stable keys
```

```text
Good key:
✓ unique among siblings
✓ stable across renders
✓ based on item identity
✓ usually database/application ID
```

```text
Bad key:
✗ changing random value
✗ unstable value
✗ duplicate value
✗ index in dynamic/reorderable lists
```

Most important mental model:

```text
key ≠ list position
key = logical identity among siblings
```

---

## 33. Key Takeaways

- Use JavaScript `map()` to render collections.
- Complex list items should often become reusable components.
- Every list child needs a suitable **key**.
- A key identifies an item among its siblings.
- Stable IDs are usually the best keys.
- Keys are essential to React's reconciliation and state-preservation behavior.
- Array indexes are risky when lists can change order, insert, delete, or filter items.
- Never generate a fresh random key during every render.
- Generate/store stable IDs when data records are created.
- `key` is not available as a normal component prop.
- Put the key where the list element/component is created.
- Keys need to be unique among siblings, not globally.
- Keys can also intentionally reset component state.
- Treat state/prop arrays immutably when filtering or sorting.
- Remember: **a key represents identity, not position**.

---

## Next Lesson

➡️ [Lesson 08 — Conditional Rendering](./08-conditional-rendering.md)
