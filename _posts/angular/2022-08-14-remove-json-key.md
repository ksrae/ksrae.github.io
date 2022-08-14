---
title: "JSON에서 가변적인 특정 항목 제거하기 (Remove a Specific Key Value in JSON Dynamically)"
date: 2022-08-14 18:14:00 +0900
comments: true
categories: javascript
tags: [json]
---

우리는 개발하면서 Array의 특정 항목을 제거할 때가 있습니다.<br/>
그럴 때 filter()를 사용하면 간단하게 원하는 항목을 제거할 수 있습니다.<br/>
<br/>
그런데 JSON에서는 filter()를 제공하지 않으므로 골치 아픈 경우가 많습니다.<br/>
때로는 이를 위해 아래와 같이 새로운 json 변수를 생성하여 새로 작성하기도 합니다.<br/>

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

위의 방법은 불 필요한 변수를 생성하고 있으며, 매우 많은 값을 가진 Json의 경우에는 일일이 작성하여야 하므로 대응하기가 어렵습니다.<br/>

그래서, 이번 시간에는 Array의 filter와 같이 특정 항목을 제거할 수 있는 방법을 알아보겠습니다.<br/>
또한 가변적으로 원하는 항목을 제거할 수 있는 방법도 함께 알아보겠습니다.<br/>

## delete 제거

가장 흔항 방법으로는 delete를 활용하는 방법입니다.

```tsx
const test = {
  test1: 'test1',
  test2: 'test2',
};

delete test[test1];
```

그러나 특정 항목에 직접 접근하여 제거하는 방식은 좋은 방법이 아니며, JSON 변수를 배열처럼 접근하는 방식도 추천할 수 없는 방식입니다.<br/>
특히 eslint의 같은 코드 규칙을 정할 때 이를 예외적으로 허용해야하는 점도 고려해야합니다.<br/>
<br/>

## const를 활용한 제거

우리는 JSON객체의 각 항목을 const 변수에 포함할 수 있습니다. 이를 활용하여 특정 항목을 분리하여 추출한 후 포함시키지 않는 방법입니다.

```tsx
const test = {
  test1: 'test1',
  test2: 'test2',
};

const {test1, ...obj} = test;
test = obj;
```

const 변수에 특정항목 (여기서는 test1)을 별도로 정의한 후 기존 JSON 변수에 해당 항목을 제외한 범위의 값 (여기서는 obj) 만 재주입 하는 방법입니다.<br/>
delete를 사용하지 않아 eslint 규칙에 어긋나지 않으면서도 간결하게 해결할 수 있는 장점이 있습니다.<br/>
<br/>

## const를 활용한 가변적인 값 제거

이번에는 const를 활용하여 filter 함수를 제작해보겠습니다.<br/>
<br/>
그 전에 우리는 interface를 정의할 때 키가 가변적인 경우 다음과 같이 정의합니다.<br/>

```tsx
export interface ITest {
  [key: string]: string
}
```

이와 유사한 방법을 활용하면 쉽게 filter함수를 만들 수 있습니다.<br/>

```tsx
let test = {
  test1: 'test1',
  test2: 'test2',
};

test = filter('test1');

function filter(key: string): unknown {
  const { [key]: keyval, ...obj} = test;
  return obj;
}

```

위의 예제에서 filter함수를 보면 key값을 받아 test 변수로부터 이 값을 빼내는 것을 볼 수 있습니다.<br/>
<br/>



