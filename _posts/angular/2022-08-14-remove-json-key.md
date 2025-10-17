---
title: "Remove a Specific Key Value in JSON Dynamically"
date: 2022-08-14 18:14:00 +0900
comments: true
categories: javascript
tags: [json]
---

Removing specific entries from an array during development is a common task, and the `filter()` method provides a straightforward solution. However, when dealing with JSON objects, the absence of a built-in `filter()` equivalent often presents a challenge. A common, but suboptimal, approach involves creating a new JSON variable and manually reconstructing the object, excluding the desired entries.

```tsx
let origin = {
  name: 'kim',
  birth: '990101',
  email: 'kim@test.com'
};

// remove birth
let filtered = {
  birth: origin.birth,
  email: origin.email
};

...
```

This method introduces unnecessary variable creation and becomes unwieldy when dealing with JSON objects containing a large number of properties, requiring manual entry for each property.<br/>

Therefore, this article explores alternative techniques for removing specific items from JSON objects, mirroring the functionality of the `filter` method in arrays. We will also delve into methods for dynamically removing desired items.

## Delete Removal

A frequently encountered approach involves utilizing the `delete` operator.

```tsx
let origin = {
  name: 'kim',
  birth: '990101',
  email: 'kim@test.com'
};

delete origin['birth'];
```

However, directly accessing and removing specific properties using this method is generally discouraged. Furthermore, treating JSON variables as arrays is not a recommended practice. The `delete` operator actually removes the object value allocated in memory, leading to slower processing times due to memory access overhead.

## Removal Using `const`

A more efficient approach leverages the ability to assign individual properties of a JSON object to `const` variables. By extracting the properties to be removed into separate `const` variables, we can reconstruct the JSON object without including those properties.

```tsx
let origin = {
  name: 'kim',
  birth: '990101',
  email: 'kim@test.com'
};

const {birth, ...obj} = origin;
origin = obj;
```

This method defines specific properties (in this case, `birth`) as separate `const` variables. It then reassigns the original JSON variable with only the remaining properties (represented by `obj`). This approach avoids the use of `delete`, resulting in faster and more concise code.

## Dynamically Removing Values Using `const`

Having explored methods for removing fixed values, let's examine how to create a filter function using `const` to remove values dynamically.<br/>

Before proceeding, it's essential to understand how to define an interface with dynamic keys. As demonstrated in the interface definition below, keys can be defined within brackets.

```tsx
export interface IData {
  [key: string]: string
}
```

Removing dynamic object values can be easily implemented using a similar approach to the interface described above.

```tsx
let origin = {
  name: 'kim',
  birth: '990101',
  email: 'kim@test.com'
};

origin = filter('birth');

function filter(key: string): unknown {
  const { [key]: keyval, ...obj} = test;
  return obj;
}

```

In the `filter` function above, the `key` value is received, and the corresponding value is removed from the `origin` variable before returning the modified object.



