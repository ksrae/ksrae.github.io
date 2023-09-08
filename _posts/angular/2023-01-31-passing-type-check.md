---
title: "Template에서 타입 검사 우회하기(Bypass type checking on Template)"
date: 2023-01-31 10:00:00 +0900
comments: true
categories: angular
tags: [casting]
---

> Typescript는 Javascript와 다르게 Type을 검사하고, 알맞은 Type이 아닌 경우 에러를 발생시키며, 올바른 타입을 지정할 것을 요구합니다. 이번 시간은 때때로 변수의 정확한 타입을 알 수 없을 경우 타입 검사를 무시하는 방법에 대해 알아봅시다.

한 json 값이 있다고 가정합시다.
```tsx
person = {
  name: 'John Doe',
  age: 30
};
```

만일, 아래와 같이 person 객체에 존재하지 않는 값을 호출하면 에러가 발생합니다.

```tsx
console.log(this.person['firstname'])
```

이 에러를 무시하고 진행하려면 타입 검사 우회방법을 활용해야 합니다. 
다시 말해 this.person의 타입을 임시로 any형으로 변경하여 에러를 피하는 방법입니다.
이 때 결과는 존재하지 않는 값이므로 undefined가 출력됩니다.


## Typescript에서 캐스팅하기

두 가지 방법이 있는데 `<any>` 또는 `as any`로 캐스팅할 수 있습니다.
이 때 괄호로 감싸주어야 에러가 발생하지 않습니다.
```tsx
console.log(
  (<any>this.person)['firstname']
);
console.log(
  (this.person as any)['firstname']
);
```

## Template에서 캐스팅하기
만일 template에서 component와 같은 방법으로 캐스팅을 시도하면 에러가 발생합니다.
즉, 다음은 에러가 발생합니다.

```html
<p>
{{ (person as any).firstname }}
</p>

<!-- not working -->
```

### $any로 캐스팅하기
Template에서는 $any로 캐스팅해야 합니다.

```html
<p>
{{ $any(person).firstname }}
</p>

<!-- working, but empty value -->
```

이럴 경우 존재하지 않으므로 값은 출력되지 않지만 에러가 발생하지 않아 정상적으로 처리되는 것을 확인할 수 있습니다.
(대부분의 경우 타입 검사 우회는 추천하지 않습니다. 보다 정확한 타입을 선언하여 사용하는 것을 추천합니다.)