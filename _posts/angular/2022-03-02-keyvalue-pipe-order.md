---
title: "keyvalue pipe로 data 정렬하기 (sorting of keyvalue pipe data)"
date: 2022-03-02 00:00:00 +0900
comments: true
categories: angular
tags: [keyvalue, pipe]
---

 template에서 object의 key와 value를 쉽게 구분하여 사용할 수 있도록 기본 제공되는 keyvalue pipe를 활용할 때, 정렬 순서를 커스터마이징 해봅니다.<br/>
<br/>


# keyvalue pipe 란?
keyvalue pipe는 json값의 key와 value를 쉽게 구분하여 template에 표시할 수 있도록 돕습니다.<br/>
또한 ngFor와 함께 사용하여 json의 모든 구성을 원본의 수정 없이 key,value로 쉽게 나누어 출력할 수 있는 장점도 있습니다.<br/>


## 예시

```tsx
// component
...
let data = {
  id: 1,
  name: 'test',
  age: 100,
  gender: 'unknown'
  ...
};
...
```

의 데이터가 있을 때 이를 keyvalue를 활용하면 다음과 같이 나타낼 수 있습니다.

```html
{% raw %}
<ng-container *ngFor="let item of data | keyvalue">
  <p>{{item.key}} => {{item.value}}</p>
</ng-container>

<!-- result:
  age => 100,
  gender => 'unknown',
  id => 1,
  name => 'test'
-->
{% endraw %}
```

변수로 선언했던 data의 값이 key와 value로 나뉘어 화면에 표시됨을 볼 수 있습니다.<br/>
<br/>
여기에서 한가지 재미있는 점은 json 데이터가 정렬되어 출력된다는 것 입니다.<br/>
keyvalue pipe는 기본으로 a->z 로 정렬되도록 되어 있기 때문입니다. 이제 이 정렬을 변경하는 방법을 알아 보겠습니다.

# 정렬 (sorting)

keyvalue pipe를 사용해서 데이터를 원하는 방식으로 정렬해 봅시다.<br/>
그전에 sorting에 대해서 먼저 간단히 알아보겠습니다.

## 기본 sorting
javascript에서 기본으로 제공하는 sort 함수는 값 없이 사용할 경우 a->z로 정렬하는데 이 때 정렬을 변경하고 싶다면 다음과 같이 sort 함수를 수정할 수 있습니다.


```tsx
arr.sort((a, b) => a - b);
```

또는 반대로 정렬하려면 아래와 같이 순서를 변경합니다.

```tsx
arr.sort((a, b) => b - a);
```


## keyvalue 에서 sorting

keyvalue는 기본 parameter로 sorting을 추가할 수 있는 기능을 제공합니다.

```
{{ input_expression | keyvalue [ : compareFn ] }}

compareFn:	(a: KeyValue<K, V>, b: KeyValue<K, V>)
```

keyvalue는 parameter가 없는 경우 ascend 정렬을 기본값으로 가지고 있으며, 그래서 keyvalue를 사용하는 경우 자동으로 정렬되게 되는 것입니다.<br/>
<br/>
다시 말해서 우리가 원하는 sorting을 적용하려면, 위의 기본 sorting을 customizing 한 뒤에 keyvalue의 parameter로 추가해야 합니다.<br/>
아래의 예제를 통해 알아보고 마무리 하겠습니다.<br/>

```tsx
// component
...
let data = {
  id: 1,
  name: 'test',
  age: 100,
  gender: 'unknown'
  ...
};

// ascend
ascending = (a: KeyValue<K, V>, b: KeyValue<K, V>) => {
  return a - b;
}

original = (a: KeyValue<K, V>, b: KeyValue<K, V>) => {
  return 0;
}

descending = (a: KeyValue<K, V>, b: KeyValue<K, V>) => {
  return b - a;
}
...
```

```html
{% raw %}
<ng-container *ngFor="let item of data | keyvalue: original"> <!-- ascending, original, descending 중 원하는 방식을 입력합니다. -->
  <p>{{item.key}} => {{item.value}}</p>
</ng-container>

<!-- result:
  id => 1,
  name => 'test',
  age => 100,
  gender => 'unknown'
-->
{% endraw %}
```




# 참고 사이트
- [Angular 공식 사이트 keyvaluepipe](https://angular.io/api/common/KeyValuePipe)
- [angular-keyvalue-pipe-sort-properties-iterate-in-order](https://stackoverflow.com/questions/52793944/angular-keyvalue-pipe-sort-properties-iterate-in-order)

