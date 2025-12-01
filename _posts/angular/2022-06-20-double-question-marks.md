---
title: "How To Use Double Question Marks: ??"
date: 2022-06-20 11:06:00 +0900
comments: true
categories: javascript
tags: [nullish, coalescing, operator]
---

When comparing values, developers often use `if` statements or ternary operators to check for `undefined` or `null`.

```tsx
if (value !== undefined || value !== null) {
  return value;
}
return '';

// or
value = value !== undefined || value !== null ? value : '';
```

## Nullish Coalescing Operator

To reduce the verbosity of repeatedly checking for `undefined` and `null`, TypeScript provides the "\|\|" operator, commonly referred to as the Nullish Coalescing Operator.

```tsx
value = value || '';
```

However, using "\|\|" can sometimes lead to unexpected behavior that deviates from the developer's intent. Let's explore the potential issues.

## Problems with "||"

Typically, developers use "\|\|" to check if a variable's value is neither `undefined` nor `null`.<br/>

However, "\|\|" doesn't always fulfill this intention, as it evaluates based on falsy values.<br/>

The conditions for a falsy value are as follows:
- undefined
- null
- false
- 0
- ''

<br/><br/>

In essence, "\|\|" treats `undefined` and `null`, along with other values, as false. This can yield results that differ from what the developer expects. To address this, TypeScript introduced a new operator.

## The New Operator "??"

Starting with version 3.7, TypeScript provides the new "??" operator.

```tsx
value = value ?? '';
```


The "??" operator specifically checks for nullish values, i.e., `undefined` and `null`, ensuring the code behaves as intended by the developer.<br/>

In other words, "??" only considers `undefined` and `null` as false.<br/>

For example, ` '', false, 0` are falsy values but not nullish, so "??" treats them as true.

```tsx
let book = {
	Publisher: null,
	Amount: 400,
	Version: 0,
	Title: 'Angular',
	SubTitle:''
	IsFreeBook: false
};

let falsyBook = {
  type: book.Type || "PDF",
  Publisher: book.Publisher || "Angular publisher",
  Amount: book.Amount || 400,
  Version: book.Version || 1,
  Title: book.Title || "Angular",
  SubTitle: book.SubTitle || "Angular Book",
  IsFreeBook: book.IsFreeBook || true,
}

let nullishBook = {
  type: book.Type ?? "PDF",
  Publisher: book.Publisher ?? "Angular publisher",
  Amount: book.Amount ?? 400,
  Version: book.Version ?? 1,
  Title: book.Title ?? "Angular",
  SubTitle: book.SubTitle ?? "Angular Book",
  IsFreeBook: book.IsFreeBook ?? true,
}
```

The results are as follows:
```tsx
// falsyBook
{
  type: "PDF",
  Publisher: "Angular publisher",
  Amount: 400,
  Version: 1,
  Title: "Angular",
  SubTitle: "Angular Book",
  IsFreeBook: true,
}

// nullishBook
{
  type: "PDF",
  Publisher: "Angular publisher",
  Amount: 400,
  Version: 0,
  Title: "Angular",
  SubTitle: "",
  IsFreeBook: false,
}
```

## Issues in Templates

As of TypeScript version 3.9, the "??" operator isn't yet supported in templates.<br/>

Therefore, you must still use "\|\|" in templates.

## Conclusion

For enhanced code readability, it's generally recommended to avoid using "\|\|" when possible.<br/>

Since "\|\|" considers a broader range of values as false compared to "??" , it should be used with a clear and specific purpose.<br/>

When using "\|\|", other developers might not immediately understand your intent. Therefore, it's best to use "??" for nullish checks and explicitly compare falsy values separately for clarity.

## References

- [Double Question Marks Or Nullish Coalescing Operator](https://www.angularjswiki.com/angular/double-question-marks-or-nullish-coalescing-operator-in-angular-typescript/)