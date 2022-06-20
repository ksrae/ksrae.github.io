---
title: "null 값을 처리하는 명령의 비교(How To Use Double Question Marks: ??)"
date: 2022-06-20 11:06:00 +0900
comments: true
categories: angular
tags: [nullish, coalescing, operator]
---

값을 비교할 때, undefined 또는 null을 비교하기 위해 우리는 일반적으로 if 또는 삼항연산자를 활용합니다.<br/>
```tsx
if (value !== undefined || value !== null) {
  return value;
}
return '';

// or
value = value !== undefined || value !== null ? value : '';
```

## Nullish coalescing Operator
Typescript는 매번 변수를 비교할 때 undefined와 null을 체크해야 하는 불편함을 줄이기 위해 단축명령어인 '\|\|' 를 제공하며, 이를 Nullish coalescing Operator라고 합니다.

```tsx
value = value || '';
```

그러나 '\|\|' 는 때때로 개발자의 의도와 다른 결과를 내는 경우가 있어서 문제가 됩니다. 어떠한 문제점이 있는지 살펴보도록 합시다.

## '\|\|' 의 문제점
일반적으로 개발자가 '\|\|'를 사용하는 의도는 변수의 값이 undefined이거나 null이 아닌 경우를 체크하는 것입니다. <br/>
그러나 '\|\|'는 이런 의도를 만족시키지 못합니다. 왜냐하면 '\|\|'는 falsy를 의미하기 때문입니다. <br/>
falsy의 조건은 다음과 같습니다.<br/>
- undefined
- null
- false
- 0
- ''
<br/><br/>
즉, undefined, null뿐 아니라 다른 값도 비교하므로 개발자의 의도와 다른 결과를 내기도 합니다.

## 새로운 명령어 '??'
Typescript는 3.7부터 새로운 명령어 '??'를 제공합니다.<br/>
'??'는 nullish 상태인 undefined와 null을 체크하는 명령어로 개발자의 정확한 의도대로 동작합니다.<br/>

```tsx
value = value ?? '';
```

## '\|\|' 와 '??' 의 비교
코드를 통해 두 명령어의 차이를 비교해봅시다.<br/>
만일 아래와 같은 값이 있다고 할 때, 아래와 같은 두 값은 어떤 결과를 나타내는지 알아봅시다.<br/>

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

결과는 아래와 같습니다.
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

## Template에서의 문제
typescript 3.9까지는 아직 template에서 '??'를 지원하지 않습니다. <br/>
따라서 template에서는 아직 '\|\|'를 사용해야 합니다.<br/>


## 결과
높은 가독성을 위해 가능하면 '\|\|'를 사용하지 않는 것을 권장합니다.<br/>
'\|\|' 는 '??' 보다 포괄적인 범위의 값을 false로 판단하므로 보다 정확한 목적을 가지고 사용해야하며,<br/>
'\|\|'를 사용하는 경우 다른 개발자가 코드를 읽을 때 의도를 알 수 없으므로 nullish는 '??'를 활용하고, falsy 값은 별도로 비교 표기하는 것이 좋습니다.<br/>


## 참고사이트
- [Double Question Marks Or Nullish Coalescing Operator](https://www.angularjswiki.com/angular/double-question-marks-or-nullish-coalescing-operator-in-angular-typescript/)