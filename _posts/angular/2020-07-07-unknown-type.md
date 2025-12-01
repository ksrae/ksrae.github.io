---
title: "Unknown Type"
date: 2020-07-07 18:03:00 +0900
comments: true
categories: javascript
tags: [typescript, types]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Unknown-%ED%83%80%EC%9E%85)
<br/>

The `unknown` type was introduced in TypeScript 3.0.

Similar to `any`, it can be used to declare variables of any type, and it allows you to assign any value to it, just like `any`.

So, what are the characteristics of the `unknown` type?

## Characteristics

1. Variables declared with types other than `any` cannot receive values of type `unknown`.
2. `unknown` variables can receive values of any variable type without any restrictions.
3. If an `unknown` variable is of type Object, Array, or Function, you cannot access its internal values. In other words, all of the following will result in errors:

```tsx
let user: unknown = {
  name: 'user1',
  age: 20
}

console.log(user.name) // X error
console.log(user['name']) // X error

let word: unknown = 'thissentenceislongenough';
console.log(word.length) // X error

// even unknownTypeFunction() occurs error
// also someClass = new unknownTypeClass() occurs error
```

Function calls and class instantiations also raise errors.

## Purpose

While the `any` type allows anything, the `unknown` type allows nothing. Why is such a peculiar `unknown` type needed, and where can it be useful?

TypeScript developers state the following:

> This is useful for APIs that want to signal “this can be any value, so you must perform some type of checking before you use it”. This forces users to safely introspect returned values.
> 

In essence, it serves as a message stating, "This value can be of any type, so you must check its type before using it." This compels users to ensure the returned values are handled safely.

It can also be utilized in another way.

In Generic functions, you might sometimes want to use a commonly used value from an Object passed as a parameter, but you cannot due to rule violations. In this case, explicit type casting is also not allowed. However, you can circumvent this by converting it to the `unknown` type and then performing the type conversion. This isn't necessarily the correct way to use it, but if used carefully, it can significantly reduce unnecessary code.

The following example demonstrates code that attempts to convert the name to lowercase during mapping.

```tsx
mapping<T>(list: T[]) {
	return list.map((item: T) => ({
		...item,
		name: item.name.toLowerCase() // X error
	});
}
```

However, since the type `T` is not defined, this is not allowed. Even if you attempt type casting as shown below, you still encounter an error because access itself is not permitted.

```tsx
mapping<T>(list: T[]) {
	return list.map((item: T) => ({
		...item,
		name: (item.name as string).toLowerCase() // X error
	});
}
```

However, you can pass this by using `unknown` as shown below. While not the ideal approach, it can be useful for bypassing simple exceptions.

```tsx
mapping<T>(list: T[]) {
	return list.map((item: T) => ({
		...item,
		name: (item.name as unknown as string).toLowerCase() // O success
	});
}
```

However, be aware that frequent use can reduce readability and result in a function that doesn't meet its intended purpose.

## Reference Sites

- [Announcing TypeScript 3.0 RC TypeScript](https://devblogs.microsoft.com/typescript/announcing-typescript-3-0-rc-2/#the-unknown-type)
- ['unknown' vs. 'any'](https://stackoverflow.com/questions/51439843/unknown-vs-any)