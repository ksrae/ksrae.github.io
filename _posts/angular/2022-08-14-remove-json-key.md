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

가장 흔한 방법으로는 delete를 활용하는 방법입니다.

```tsx
let origin = {
  name: 'kim',
  birth: '990101',
  email: 'kim@test.com'
};

delete origin['birth'];
```

그러나 특정 항목에 직접 접근하여 제거하는 방식은 좋은 방법이 아니며, JSON 변수를 배열처럼 접근하는 방식도 추천할 수 없는 방식입니다.<br/>
특히 delete의 경우 메모리에 할당된 object값을 실제로 제거하는 것이므로 메모리 접근 시간으로 인해 처리 속도가 느립니다. <br/>
<br/>

## const를 활용한 제거

우리는 JSON객체의 각 항목을 const 변수에 포함할 수 있습니다. 이를 활용하여 특정 항목을 분리하여 추출한 후 포함시키지 않는 방법입니다.

```tsx
let origin = {
  name: 'kim',
  birth: '990101',
  email: 'kim@test.com'
};

const {birth, ...obj} = origin;
origin = obj;
```

const 변수에 특정항목 (여기서는 birth)을 별도로 정의한 후 기존 JSON 변수에 해당 항목을 제외한 범위의 값 (여기서는 obj) 만 재주입 하는 방법입니다.<br/>
delete를 사용하지 않으므로 더 빠르고 간결하게 해결할 수 있는 장점이 있습니다.<br/>
<br/>

## const를 활용한 가변적인 값 제거

위에서는 고정된 값을 제거하는 방법에 대해 알아보았으니,
이번에는 const를 활용하여 filter 함수를 제작해보겠습니다.<br/>
<br/>
그 전에 우리는 interface를 정의할 때 키가 가변적인 경우에 대해 알아봅시다.
아래의 interface 정의와 같이 key를 대괄호에 묶어서 정의할 수 있습니다.<br/>

```tsx
export interface IData {
  [key: string]: string
}
```

가변적인 object값의 제거도 위의 interface와 유사한 방법을 활용하면 쉽게 구현할 수 있습니다.<br/>

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

위의 예제에서 filter함수를 보면 key값을 받아 origin 변수로부터 해당 값만을 제거하고 리턴하는 것을 볼 수 있습니다.<br/>
<br/>



