---
title: "Json Deep Copy"
date: 2022-10-18 14:23:00 +0900
comments: true
categories: javascript
tags: [json]
---
## Deep Copy

deep copy에 대한 내용은 검색을 통해 쉽게 찾아볼 수 있습니다만, 그럼에도 여기에 다시 정리하는 이유는 Deep Copy의 목적에 따라 한계가 있음이 명확히 정리되지 않은 글이 많기 때문입니다.

이번 글에서는 Deep Copy란 무엇인지 간단히 알아보고, 여러 방법들을 나열하고, 가장 빠르고 정확한 방법 순서대로 정리해보려고 합니다.

## Json Deep Copy의 목적

일반적으로 Json 값을 다른 변수에 할당할 때 아래와 같이 부등호를 활용합니다.

```
const jsonValue = {
   key: 'test',
   value: 'this is a shallow copy'
};
const copyValue = jsonValue;
```

위의 예시와 같이 Json 값을 가진 jsonValue 변수를 다른 변수인 copyValue에 할당하였을 때,  값이 object인 경우 javascript는 해당 값을 별도의 메모리에 복사하지 않고, 기존 값의 주소를 참조하도록 합니다. 그렇기 때문에 만일 복사된 copyValue 변수의 값을 변경하는 경우 이전 변수인 jsonValue의 값을 변경하게 되어 두 변수의 값이 모두 변경되는 현상이 발생합니다. 이와 같은 방식을 Shallow Copy라고 합니다.

```
copyValue.key = 'is this a shallow copy?'

console.log({jsonValue}, {copyValue});

// Result
jaonValue
{key: "is this a shallow copy?", value: "this is a shallow copy"}

copyValue
{key: "is this a shallow copy?", value: "this is a shallow copy"}

```

위의 Shallow Copy와는 반대로 복사한 object 값을 별도의 메모리에 할당하여 각각 독립적인 값으로 활용하는 방식을 Deep Copy라고 합니다. javascript에서는 이러한 방식을 내장함수로 제공하고 있지 않으므로 여러가지 대안들이 인터넷에 제시되어 있습니다. 대표적으로 아래의 4가지 방법이 제시되어 있는데 각각을 알아보고 추천 여부를 정리해보려고 합니다.

## Json Deep Copy 의 4가지 방법

### Json Stringify and Parsing (추천)

가장 기본적인 방법으로 json을 string으로 변경한 뒤 다시 json으로 parsing 하는 방법 입니다.

가장 확실한 방법이며, [mdn에서는 javascript의 deep copy](https://developer.mozilla.org/en-US/docs/Glossary/Deep_copy)에 대해 이 방법만 소개하고 있습니다.  아래에 소개할 재귀보다는 처리속도가 느리지만 json의 복잡도가 올라갈수록 서드파티보다 좋은 성능을 발휘하므로 처리 속도에 민감하지 않은 작업은 이 방법을 사용하시는 것을 추천합니다.

```
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

### 재귀 함수 (추천)

재귀 함수를 작성하여 모든 키를 옮기는 방법 입니다. 복잡도와 관계 없이 처리속도가 빠르며, 배열과 같은 형태도 문제 없이 복사합니다.

```
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

### ... 활용 (비추천)

...을 통해서도 복사가 가능합니다.

```
const test = {
   a: 1,
   b: 2
}

const testDeepCopy = {...test};
```

...를 활용해도 deep copy가 가능합니다. 처리속도는 차치하고라도 가장 작성이 쉽다는 장점이 있기에 많이 선호하는 방식입니다.

그러나, 이 방식은 치명적인 단점이 있습니다. 바로 ***하위 1단계만 deep copy가 가능하다***는 점입니다.  보다 더 하위 단계를 deep copy하려면 다음과 같이 진행해야 합니다.

```
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

이 방법은 json의 모든 구성을 알아하며, 공통으로 활용할 수 없으므로 오히려 위의 두 가지 방법보다 더 불편합니다.

따라서 이 방법은 추천하지 않습니다. 혹시 타 사이트에서 이런 내용을 검색하셔서 사용하셨다면 유의하여야 합니다.




### lodash의 cloneDeep 함수 활용 (비추천)

```
let deepCopy = _.cloneDeep(test);
```

이 방법은 서드파티를 활용하므로 추천하지 않습니다. 또한, json의 복잡도가 올라가면 parsing 방식보다도 처리속도가 느리므로 추천하지 않습니다.

해당 서드파티를 이미 사용 중이더라도 위의 추천하는 방식을 활용하시는 것을 권장 드립니다.


## 결론

매 프로젝트마다 작성해야 하는데 머리에서 즉시 떠올리며 바로 작성할 수 있는 코드는 재귀 함수 보다는 parsing 방식이 더 쉽기 때문에, 일반적으로 parsing 방식을 추천하며, 속도를 중요시 한다면 재귀함수를 사용하실 것을 추천합니다.
