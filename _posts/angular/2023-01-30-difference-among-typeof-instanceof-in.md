---
title: "difference among typeof, instanceof, keyof and in"
date: 2023-01-30 11:07:00 +0900
comments: true
categories: javascript
tags: [typescript, instanceof, typeof, keyof, in]
---

[한국어(Korean) Page](https://velog.io/@ksrae/typeof-instanceof-keyof-and-in%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90)
<br/>

> In TypeScript, the `typeof`, `instanceof`, `keyof`, and `in` operators each serve distinct roles in type manipulation and object inspection. This post will delve into each of these operators, providing clear examples and use cases.
 

## `typeof` Operator

The `typeof` operator is used to determine the type of a variable or value at runtime. It returns a string indicating the type.

```jsx
let myNumber = 123;
console.log(typeof myNumber); // Output: number

let myString = "hello";
console.log(typeof myString); // Output: string
```

In the example above, `typeof myNumber` returns `"number"` because `myNumber` is a number. Similarly, `typeof myString` returns `"string"` because `myString` is a string.

### Returnable Types

The following types can be returned by the `typeof` operator:

- `"undefined"`: For `undefined` values.
- `"boolean"`: For boolean values.
- `"number"`: For numeric values (Number).
- `"bigint"`: For numbers larger than the Number type.
- `"string"`: For string values.
- `"symbol"`: For Symbol values (introduced in ECMAScript 2015).
- `"function"`: For functions.
- `"object"`: For all other objects, including arrays and null.

> Note that number is a 64-bit floating point format. See [Double-precision floating-point format](https://en.wikipedia.org/wiki/Double-precision_floating-point_format) for more details.
> 

> `bigint` is for [Arbitrary-precision arithmetic](https://en.wikipedia.org/wiki/Arbitrary-precision_arithmetic).
> 

## `instanceof` Operator

The `instanceof` operator checks if an object is an instance of a particular class or constructor function.

In JavaScript, it verifies the existence of a function's `prototype`. In TypeScript, it confirms whether a variable conforms to the datatype of a class. It returns a boolean value.

```jsx
function Car(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
}

const auto = new Car('Honda', 'Accord', 1998);
console.log(auto instanceof Car); // true
Car.prototype = {};
console.log(auto instanceof Car); // false
```

In the first `console.log`, `auto` is an instance of `Car`, so it returns `true`. However, after changing `Car.prototype`, the second `console.log` returns `false` because `auto` is no longer considered an instance of the modified `Car` constructor.

```tsx
class User {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

const person = new User('username');
console.log(person instanceof User); // true
```

`person` is an instance of the `User` class, so the expression returns `true`.

### Type Guard

When multiple classes implement the same interface, `instanceof` helps to determine the specific class of an object, implementing a type guard.

```tsx
class Foo {}
class Bar extends Foo {}
class Bas {}

var bar = new Bar();
console.log(bar instanceof Bar); // true
console.log(bar instanceof Foo); // true
console.log(bar instanceof Object); // true
console.log(bar instanceof Bas); // false
```

Here, `bar` is declared as type `Bar`, inheriting from `Foo`. The `console.log` statements return `true` when checking against `Foo` or `Bar`. A type guard goes further, ensuring that actions are performed based on the specific type (Foo or Bar) of `bar`. Especially with generic types, this prevents type errors by ensuring clarity before function execution.

## `keyof` Operator

The `keyof` operator is used to create a union of keys from an object type.

### JSON

```jsx
let myObject = { name: "John", age: 30 };
let myObjectKey: keyof typeof myObject; // 'name' | 'age'
myObjectKey = 'name';
```

In this example, `myObjectKey` can be either `"name"` or `"age"` because `keyof typeof myObject` creates a union of the keys of `myObject`. Using `keyof` enhances type safety by restricting the possible values of `myObjectKey` to the existing keys of `myObject`. Note the necessity of combining with `typeof` to achieve the desired result. Type variables offer a simpler approach.

### Type

The `keyof` operator can also be used with Type variables.

```tsx
type User = { name: string, age: number, address: string };
let userKey: keyof User;
// 'name' | 'age' | 'address'
```

In this example, `userKey` can be `"name"`, `"age"`, or `"address"` because `keyof User` creates a union of the keys defined in the `User` type.

## `in` Operator

The `in` operator checks if an object or array has a specific property. Commonly used to verify the existence of keys in JSON objects or values in enums.

```jsx
let myObject = { name: "John", age: 30 };
console.log("name" in myObject); // Output: true
console.log("email" in myObject); // Output: false
```

The `in` operator returns `true` because `myObject` has a `"name"` property and `false` because it does not have an `"email"` property.

### Index in Array

In arrays, `in` checks for index existence, not value existence.

```tsx
let count = ['one', 'two', 'three', 'four'];
0 in count          // true
(1 + 2) in count    // true
6 in count          // false
"one" in count      // false, should be index value, not content.
"length" in count   // true, because length is a property of the array.
```

### Key in JSON

In JSON, `in` verifies key existence.

```tsx
let user = {
  name: 'username',
  age: 10
}
console.log('name' in user) // true
```

### Dynamic Key in Type

You can define restricted, variable keys in JSON, utilizing only the key values from defined JSON or types.

```tsx
let myObject = { name: "John", age: 30 };
type ObjType = {
  [key in keyof typeof myObject]: number // 'name' | 'age'
}
```

Types can simplify this further:

```tsx
type AllowedKeys = 'name' | 'age';

type ObjType1 = {
  [key in AllowedKeys]: number // 'name' | 'age'
}
```

### Not Working in Interface

Using `in` with an interface results in an error because interfaces only define the *shape* of an object, not its actual contents.

```tsx
interface User {
  name: string;
  age: number;
}

let anotherUser: User = {
  name: 'anotheruser',
  age: 20
}

console.log('name' in User) // error
console.log('name' in anotherUser) // true
```

In summary, `typeof`, `instanceof`, `keyof`, and `in` operators serve specific purposes in TypeScript for type and property checking. Proper usage enhances code reliability and enables efficient type manipulation.

## References

- [MDN-instanceof](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/instanceof)
- [typescript MyObject.instanceOf()](https://stackoverflow.com/questions/24705631/typescript-myobject-instanceof)
- [TypeScript Handbook 10 - Advanced Types](https://infoscis.github.io/2017/06/19/TypeScript-handbook-advanced-types/)
