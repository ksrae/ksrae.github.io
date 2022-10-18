---
title: "Json Deep Copy"
date: 2022-10-18 14:23:00 +0900
comments: true
categories: javascript
tags: [json]
---
## Deep Copy

deep copy에 대한 내용은 검색을 통해 쉽게 찾아볼 수 있습니다만, 그럼에도 여기에 다시 정리하는 이유는 Deep Copy의 목적에 따라 한계가 있음이 명확히 정리되지 않은 글이 많기 때문입니다.`<br/>`

이번 글에서는 Deep Copy의 설명은 생략하고, (쉽게 찾아볼 수 있습니다.) 여러 방법들을 나열해보고, 가장 빠르고 정확한 방법 순서대로 정리해보려고 합니다. `<br/>`



## Json Deep Copy의 목적

Json 값을 가진 변수를 다른 변수에 할당하였어도 해당 값이 복사되지 않고, 참조 됩니다. 복사된 다른 변수는 이전 변수의 값을 변경하게 되어 두 변수의 값이 모두 변경되는 현상입니다. `<br/>`

따라서 두 값을 별도로 저장하려면 Deep Copy 방식을 적용해야 하는데, javascript는 해당 기능을 별도의 함수로 제공하지 않으므로 직접 작성하거나 서드파티를 활용해야 합니다.



## Json Deep Copy 의 4가지 방법

### Json to String and Json Parsing (추천)

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

가장 기본적인 방법으로 json을 string으로 변경한 뒤 다시 json으로 parsing 하는 방법 입니다.`<br/>`

가장 확실한 방법이며, [mdn에서는 javascript의 deep copy](https://developer.mozilla.org/en-US/docs/Glossary/Deep_copy)에 대해 이 방법만 소개하고 있습니다.  아래에 소개할 재귀보다는 처리속도가 느리지만 json의 복잡도가 올라갈수록 서드파티보다 좋은 성능을 발휘하므로 처리 속도에 민감하지 않은 작업은 이 방법을 사용하시는 것을 추천합니다. `<br/>`



### 재귀 함수 (추천)

재귀 함수를 작성하여 모든 키를 옮기는 방법 입니다. 복잡도와 관계 없이 처리속도가 빠르며, 배열과 같은 형태도 문제 없이 복사합니다. `<br/>`

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



...를 활용해도 deep copy가 가능합니다. 처리속도는 차치하고라도 가장 작성이 쉽다는 장점이 있기에 많이 선호하는 방식입니다.`<br/>`

그러나, 이 방식은 치명적인 단점이 있습니다. 바로 하위 1단계만 deep copy가 가능하다는 점입니다.  하위 단계를 deep copy하려면 다음과 같이 진행해야 합니다.`<br/>`

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

이 방법은 모든 구성을 알아하며, 공통으로 활용할 수 없으므로 오히려 위의 두 가지 방법보다 더 불편합니다. `<br/>`

따라서 이 방법은 추천하지 않습니다. 혹시 타 사이트에서 이런 내용을 검색하셔서 사용하셨다면 유의하여야 합니다.`<br/>`



### lodash의 cloneDeep 함수 활용 (비추천)

```
let deepCopy = _.cloneDeep(test);
```

이 방법은 서드파티를 활용하므로 추천하지 않습니다. 또한, json의 복잡도가 올라가면 parsing 방식보다도 처리속도가 느리므로 추천하지 않습니다.`<br/>`

해당 서드파티를 이미 사용 중이더라도 위의 추천하는 방식을 활용하시는 것을 권장 드립니다.



## 결론

매 프로젝트마다 작성해야 하는데 머리에서 즉시 떠올리며 바로 작성할 수 있는 코드는 재귀 함수 보다는 parsing 방식이 더 쉽기 때문에, 일반적으로 parsing 방식을 추천하며, 속도를 중요시 한다면 재귀함수를 사용하실 것을 추천합니다.`<br/>`
