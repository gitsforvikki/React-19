# Lesson 15 — Updating Objects and Arrays in State

React state should be treated as **immutable**.

That means:

> Do not modify the existing object or array directly. Create a new one and pass it to the setter.

---

# Objects in State

## 1. Do Not Mutate Object State

Avoid:

```jsx
user.name = "Aman";
setUser(user);
```

This mutates the existing object.

Prefer:

```jsx
setUser((user) => ({
  ...user,
  name: "Aman",
}));
```

This creates a new object.

---

## 2. Update a Nested Object

Suppose:

```jsx
const [user, setUser] = useState({
  name: "Vikash",
  address: {
    city: "Bengaluru",
  },
});
```

Update city like this:

```jsx
setUser((user) => ({
  ...user,
  address: {
    ...user.address,
    city: "Pune",
  },
}));
```

Important rule:

> Create a new object at every changed level.

---

## 3. Spread Is a Shallow Copy

```js
const copy = {
  ...user,
};
```

The root object is new, but nested objects can still be shared.

So this is still wrong:

```js
const nextUser = {
  ...user,
};

nextUser.address.city = "Pune";
```

because `address` is still the same nested object.

---

# Arrays in State

## 4. Do Not Mutate Array State

Avoid:

```js
items.push(newItem);
setItems(items);
```

Prefer:

```jsx
setItems((items) => [
  ...items,
  newItem,
]);
```

---

## 5. Add, Remove, Update

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
  items.filter(
    (item) => item.id !== id
  )
);
```

### Update One Item

```jsx
setItems((items) =>
  items.map((item) =>
    item.id === id
      ? {
          ...item,
          status: "Done",
        }
      : item
  )
);
```

This pattern is worth memorizing:

```text
Add    → spread
Remove → filter
Update → map
```

---

## 6. Copying an Array Is Not Enough

This can still mutate old state:

```js
const nextItems = [
  ...items,
];

nextItems[0].name =
  "Changed";
```

Why?

The array is new, but the objects inside are still shared.

Correct approach:

```jsx
setItems((items) =>
  items.map((item) =>
    item.id === id
      ? {
          ...item,
          name: "Changed",
        }
      : item
  )
);
```

---

## 7. Nested Objects Inside Arrays

Example:

```jsx
setDevelopers((developers) =>
  developers.map((developer) =>
    developer.id === id
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

For changed nested data, create new references along the changed path.

---

## 8. Be Careful with Mutating Array Methods

These mutate the original array:

```text
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```

Use non-mutating patterns instead.

For sorting:

```js
const sorted =
  items.toSorted(compareFn);
```

or:

```js
const sorted = [
  ...items,
].sort(compareFn);
```

---

## 9. Structural Sharing

You do not need to clone every object.

In:

```jsx
setItems((items) =>
  items.map((item) =>
    item.id === id
      ? { ...item, status: "Done" }
      : item
  )
);
```

only the changed item gets a new object.

Unchanged items can safely keep their old references.

This is called **structural sharing**.

---

## 10. Avoid Deep Cloning Everything

Do not use this as a normal React update strategy:

```js
JSON.parse(
  JSON.stringify(state)
);
```

Usually you only need to copy the changed path.

This is clearer and more efficient.

---

## 11. Avoid Redundant State

If this can be calculated:

```js
const itemCount =
  items.length;
```

you usually do not need:

```js
const [
  itemCount,
  setItemCount,
] = useState(0);
```

Do not store derived data unless there is a real need.

---

## Common Mistakes

### Mistake 1 — Mutating object state

```js
user.name = "Aman";
```

### Mistake 2 — Mutating array state

```js
items.push(newItem);
```

### Mistake 3 — Copying only the outer object

Nested references can still be shared.

### Mistake 4 — Copying the array but mutating one of its objects

The item object can still belong to old state.

### Mistake 5 — Deep cloning everything

Usually unnecessary.

---

## Interview Questions

### Why should React state be treated as immutable?

Because creating new references makes updates predictable and works well with React's render and equality model.

### How do you update an object property?

```jsx
setUser((user) => ({
  ...user,
  name: "Aman",
}));
```

### How do you update nested state?

Create new objects at every changed level.

### Is spread syntax a deep copy?

No. Spread creates a shallow copy.

### How do you add an item to array state?

```jsx
setItems((items) => [
  ...items,
  newItem,
]);
```

### How do you remove an item?

```jsx
setItems((items) =>
  items.filter(
    (item) => item.id !== id
  )
);
```

### How do you update one item?

```jsx
setItems((items) =>
  items.map((item) =>
    item.id === id
      ? { ...item, ...changes }
      : item
  )
);
```

---

## Quick Revision

```text
OBJECT
update → spread

NESTED OBJECT
update → spread every changed level

ARRAY
add    → spread
remove → filter
update → map
sort   → toSorted() or copy + sort
```

Remember:

```text
spread = shallow copy
```

and:

```text
new array
≠
all nested objects copied
```

---

## Next Lesson

➡️ [Lesson 16 — Controlled vs Uncontrolled Components](./16-controlled-uncontrolled.md)
