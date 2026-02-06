---
title: "Bypass type checking on Template"
date: 2023-01-31 10:00:00 +0900
comments: true
categories: angular
tags: [casting]
---

> TypeScript, unlike JavaScript, rigorously checks types, generates errors for type mismatches, and enforces correct type specifications. In this guide, we'll explore methods for bypassing type checking when the exact type of a variable is uncertain.

Let's consider a sample JSON object:

```tsx
person = {
  name: 'John Doe',
  age: 30
};
```

Accessing a non-existent property on the `person` object, as shown below, will typically result in a TypeScript error:

```tsx
console.log(this.person['firstname'])
```

To circumvent this error and proceed, we can employ type assertion techniques. This involves temporarily casting `this.person` to the `any` type to bypass the type checker. Note that the result will be `undefined` since the property doesn't exist.

## Casting in TypeScript

There are two primary ways to perform type casting in TypeScript: using `<any>` or `as any`. Ensure you enclose the expression in parentheses to avoid syntax errors.

```tsx
console.log(
  (<any>this.person)['firstname']
);
console.log(
  (this.person as any)['firstname']
);
```

## Casting in Templates

Attempting to cast directly in a template (e.g., within an Angular component's template) using the `as` keyword, similar to component code, will result in an error. For instance, the following snippet will fail:

```html
<p>
{{ (person as any).firstname }}
</p>

<!-- not working -->
```

### Casting with `$any`

Within templates, you must use the `$any` helper function for casting:

```html
<p>
{{ $any(person).firstname }}
</p>

<!-- working, but empty value -->
```

In this scenario, although the property doesn't exist and no value is displayed, the absence of a TypeScript error ensures smooth processing. (It's important to note that bypassing type checking is generally discouraged. Defining more accurate types is typically the preferred approach.)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Template%EC%97%90%EC%84%9C-%ED%83%80%EC%9E%85-%EA%B2%80%EC%82%AC-%EC%9A%B0%ED%9A%8C%ED%95%98%EA%B8%B0)
<br/>