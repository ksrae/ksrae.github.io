---
title: "Unknown Type"
date: 2020-07-07 18:03:00 +0900
comments: true
categories: angular
tags: [types]
---


Unknown 타입은 Typescript 3.0 부터 적용 되었습니다.

any형과 비슷하게 모든 형태의 값에 선언할 수 있으며, any처럼 어떠한 값을 넣어도 허용합니다.

그러면 unknown타입의 특징은 무엇일까요?

## 특징

1. any외의 형 선언된 변수는 unknown 타입의 값을 가져올 수 없습니다.
2. unknown 변수는 어떠한 변수 타입의 값이든 상관 없이 가져올 수 있습니다.
3. unknown 변수가 Object / Array / Function 등의 타입이라면 내부의 값에 접근할 수 없습니다. 즉, 아래는 모두 에러가 됩니다.

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

함수 호출, 클래스 호출도 에러를 일으킵니다.

## 목적

any형은 무엇이든 허용하는 반면에 unknown 형은 아무것도 허용하지 않습니다. 도대체 이런 희안한 unknown형은 왜 어디에 필요한 것일까요?<br><br>

Typescript 개발자는 아래와 같이 말합니다.

> *This is useful for APIs that want to signal “this can be any value, so you must perform some type of checking before you use it”. This forces users to safely introspect returned values.*

*("이 값은 any 형의 값일 수 있으니 사용하기 전에 반드시 타입 선언여부를 체크하라"는 식의 메시지로서 사용할 때 유용하다. 이는 사용자에게 리턴값을 의심의 여지없이 안전하게 하도록 강요한다.)*

또 다른 유형으로도 활용할 수 있습니다.<br>

Generic 함수에서 때때로 파라미터로 넘어온 Object에서 공통으로 사용중인 값을  활용하고 싶을 때가 있지만, 규칙 위반으로 사용할 수 없습니다. 이 때는 강제 형 변환도 허용되지 않습니다. <br>이 때, unknown형으로 변환시킨 뒤에 형 변환을 하는 방식으로 이를 회피할 수 있습니다. 올바른 사용 방식은 아니나 잘 활용한다면 불필요한 코드를 많이 줄일 수 있을 것입니다.<br><br>

아래의 예제는 mapping 시 name을 lowerCase로 변환하고자 하는 코드 입니다. 

```tsx
mapping<T>(list: T[]) {
	return list.map((item: T) => ({
		...item,
		name: item.name.toLowerCase() // X error
	});
}
```

하지만, T형은 정의되지 않았으므로, 이를 허용하지 않습니다. 아래와 같이 형 변환을 시도해도 접근 자체가 허용되지 않으므로 역시 에러를 일으킵니다.

```tsx
mapping<T>(list: T[]) {
	return list.map((item: T) => ({
		...item,
		name: (item.name as string).toLowerCase() // X error
	});
}
```

그러나, 아래와 같이 unknown을 활용하면 통과할 수 있습니다. 올바른 방법이라 할 수 없으나 간단한 예외를 통과 시키고자 할 때는 유용합니다. <br>단, 자주 활용하게 되면 가독성이 떨어지고 목적에 맞지 않는 함수가 될 수 있으므로 유의하여야 합니다.

```tsx
mapping<T>(list: T[]) {
	return list.map((item: T) => ({
		...item,
		name: (item.name as unknown as string).toLowerCase() // O success
	});
}
```

## 참고 사이트
- [Announcing TypeScript 3.0 RC TypeScript](https://devblogs.microsoft.com/typescript/announcing-typescript-3-0-rc-2/#the-unknown-type)
- ['unknown' vs. 'any'](https://stackoverflow.com/questions/51439843/unknown-vs-any)