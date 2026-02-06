---
title: "Json Deep Copy"
date: 2022-10-18 14:23:00 +0900
comments: true
categories: javascript
tags: [json]
---

## Deep Copy Explained for Developers

While numerous resources detail deep copying, many lack a clear articulation of its limitations relative to its intended purpose. This post aims to clarify deep copying, outline various methods, and present them in order of speed and accuracy.

## The Purpose of JSON Deep Copy

When assigning a JSON value to another variable, the assignment operator (=) is often used:

```ts
const jsonValue = {
   key: 'test',
   value: 'this is a shallow copy'
};
const copyValue = jsonValue;
```

In the above example, when assigning `jsonValue` to `copyValue`, JavaScript, for object values, creates a reference to the original memory location rather than copying the value to a new memory location. Consequently, modifying `copyValue` also modifies `jsonValue`, resulting in both variables reflecting the change. This behavior is known as a *Shallow Copy*.

```ts
copyValue.key = 'is this a shallow copy?'

console.log({jsonValue}, {copyValue});

```

```bash
// Result
jaonValue
{key: "is this a shallow copy?", value: "this is a shallow copy"}

copyValue
{key: "is this a shallow copy?", value: "this is a shallow copy"}
```

Deep Copy, conversely, allocates a new memory space for the copied object, ensuring that modifications to the copy do not affect the original. JavaScript doesn't provide a built-in deep copy function, leading to several alternative approaches. We'll examine four common methods and assess their suitability.

## Four Methods for JSON Deep Copy

### JSON Stringify and Parsing (Recommended)

This fundamental technique involves converting the JSON to a string and then parsing it back into JSON.<br/>

It's a reliable method, and [MDN recommends it for JavaScript deep copying](https://developer.mozilla.org/en-US/docs/Glossary/Deep_copy). While it might be slower than recursion for simple objects, it scales well with JSON complexity, often outperforming third-party libraries. We recommend this approach for operations that are not extremely sensitive to speed.

```ts
const jsonValue = {
   id: 123,
   name: 'test',
   min: 0,
   max: 100,
   isTest: true,
   more: {
      {
         initial: {
            date: '00-00-00',
            value: 10,
         },
         complete: {
            isComplete: false,
            date: null,
            value: null,
         }
      }
   }
}
const deepCopy = JSON.parse(JSON.stringify(jsonValue));
```

### Recursive Function (Recommended)

This method involves creating a recursive function that iterates through all keys and copies them. It’s generally fast, regardless of complexity, and handles arrays effectively.

```ts
function cloneObject(obj) {
  var clone = {};
  for (var key in obj) {
    if (typeof obj[key] == "object" && obj[key] != null) {
      clone[key] = cloneObject(obj[key]);
    } else {
      clone[key] = obj[key];
    }
  }

  return clone;
};
```

### Spread Operator (...) (Not Recommended)

The spread operator can also be used for copying.

```ts
const test = {
   a: 1,
   b: 2
}

const testDeepCopy = {...test};
```

While the spread operator is easy to use, offering a concise syntax, it has a critical limitation: ***it only performs a deep copy for the first level of the object***. Deep copying deeper levels requires manual spreading at each level:

```ts
const test = {
   a: 1,
   b: {
      c: 2,
      d: {
         e: 3
      }
   }
};

const testDeepCopy = {
   ...test,
   b: {
      ...test.b,
      d: {
         ...test.b.d,
      }
   }
};
```

This approach necessitates knowledge of the entire JSON structure and is not reusable, making it less convenient than the previous two methods.<br/>

Therefore, we do not recommend using the spread operator for deep copying. Exercise caution if you've encountered this suggestion on other sites.

### Using lodash's cloneDeep Function (Not Recommended)

```ts
let deepCopy = _.cloneDeep(test);
```

We advise against using third-party libraries for this purpose. Furthermore, `cloneDeep`'s performance degrades with increasing JSON complexity, often being slower than the `JSON.parse(JSON.stringify())` method.<br/>

Even if you're already using lodash, consider using the recommended methods above.

## Conclusion

For most projects, the parsing method is generally recommended because it is easier to recall and implement than the recursive approach. However, if speed is a primary concern, consider using the recursive function.


## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Json-Deep-Copy)
<br/>