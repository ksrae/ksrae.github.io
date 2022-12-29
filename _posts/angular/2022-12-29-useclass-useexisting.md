---
title: "different between useClass and useExisting"
date: 2022-12-29 11:52:00 +0900
comments: true
categories: angular
tags: [provider, useexisting, useclass]
---

> 이번 시간은 module에서 providers를 설정 시 useClass와 useExisting 의 차이를 알아보겠습니다.


# useClass
Injectable class의 이름을 변경하여 사용할 수 있습니다.<br/>
이 때, 이름은 반드시 존재하는 다른 Injectable class의 명이여야 합니다.<br/>
<br/>
사용법은 다음과 같습니다.<br/>
<br/>
```tsx
providers: [
  { provide: ProductService, useClass: fakeProductService }
]
```
<br/>

# useExisting
기존 토큰을 참조하되 별칭을 지정합니다.<br/>
반드시 두 injectable class가 존재해야 합니다.<br/>
<br/>
```tsx
providers: [
   NewProductService,
   { provide: ProductService, useExisting: NewProductService },
];
```
<br/>

# different between useClass and useExisting
얼핏 보면 useClass와 useExisting은 유사해 보이는데 왜 다른 옵션을 제공하고 있을까요?<br/>
이 두 차이를 예제를 통해 자세히 알아봅시다.<br/>
<br/>

## Setting Default Services for Testing
두가지 service가 존재한다고 합니다. 형태는 유사하지만 값이 다른 모양이므로 쉽게 구분할 수 있습니다.<br/>
<br/>

```tsx
@Injectable()
export class TestService {
  index = 0;

  add(): number {
    return ++this.index;
  }
}
```
<br/>

```tsx
@Injectable()
export class NewTestService {
  index = 0;

  add(): number {
    this.index += 10;
    return this.index;
  }
}
```

component에서 두 service를 매 초 호출한다고 가정합시다.<br/>
<br/>

```tsx
export class AppComponent {
   constructor(
      private testService: TestService,
      private newTestService: NewTestService
} {
   setInterval(() => {
      console.log('test', this.testService.add());
      console.log('newTest', this.newTestService.add());
   }, 1000)
}
```

그리고 각각의 모듈의 다른 설정의 예제를 통해 왜 이런 기능을 구분하게 되었는지 알아보겠습니다.<br/>
<br/>

## 기본 provider 설정
일반적인 방법으로 providers에 service를 직접 설정해보고 결과를 확인해봅시다.<br/>
<br/>

```tsx
// app.module
...
providers: [
   TestService,
   NewTestService
]
```

결과는 예상과 같이 두 service가 각각 다르게 동작함을 확인할 수 있습니다.<br/>
```
// result
test: 1
newTest: 10
test: 2
newTest: 20
test: 3
newTest: 30
test: 4
newTest: 40
```
<br/>

## useClass 활용
useClass를 활용하여 TestService를 NewTestClassService로 명명해봅시다.<br/>
<br/>

```tsx
// app.module
providers: [
   TestService,
   {provide: NewTestService, useClass: TestService },
]
```

결과는 NewTestService가 TestService로 전환되었으며, 기존 TestService와는 다른 Service인 것처럼 각각 다르게 동작함을 확인할 수 있습니다.<br/>
```
// result
test: 1
newTest: 1
test: 2
newTest: 2
test: 3
newTest: 3
test: 4
newTest: 4
```
<br/>

## useExisting 활용
위의 useClass와 동일한 조건에서 useClass만 useExisting으로 변경해보고 결과를 확인해봅시다.<br/>
<br/>

```tsx
// module
providers: [
   TestService,
   {provide: NewTestService, useExisting: TestService },
]
```

결과는 NewTestService가 TestService로 전환되었으며, 기존 TestService와 마치 하나의 Service인 것처럼 데이터를 공유하고 있음을 확인할 수 있습니다. <br/>

```
// result
test: 1
newTest: 2
test: 3
newTest: 4
test: 5
newTest: 6
test: 7
newTest: 8
``` 
