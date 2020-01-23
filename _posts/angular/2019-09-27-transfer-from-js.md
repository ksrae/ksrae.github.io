---
title: "js프로젝트에서 Angular 프로젝트의 데이터 접근 (transfer data js to angular)"
date: 2019-09-27 12:01:00 +0900
comments: true
categories: angular
tags: [storage, window]
---


## 분석
angular에서 js 파일에 접근할 때는 d.ts를 활용하거나 import 하여 사용할 수 있습니다.<br>
그러나, 반대의 경우 js에서 angular에 접근하고 싶을 때는 어떻게 할까요?<br><br>

일반적으로는 불가능합니다. 왜냐하면 스코프(Scope)가 다르기 때문입니다.<br><br>

다시 말해서, angular 라는 클래스 안에 선언된 변수는 외부 라이브러리에서 접근할 수 없습니다.<br><br>

예를 들면, 아래와 같습니다.


```js
function sayHello () {
  const hello = 'Hello CSS-Tricks Reader!'
  console.log(hello)
}
sayHello() // 'Hello CSS-Tricks Reader!'
console.log(hello) // Error, hello is not defined
```

외부에서는 노출된 sayHello라는 함수는 접근할 수 있으나 그 내부에 선언된 hello 에는 접근할 수 없으며 이를 스코프라고 합니다.<br>
(반대의 경우, 함수 내부에서 바깥에 선언된 변수 (또는 함수)에 접근하는 것은 클로져라고 합니다.)<br><br>

이와 같은 케이스는 다양하게 있으나 근본적으로 "외부에서 내부(또는 함수 간)에 선언된 변수는 접근할 수 없다."는 것은 같습니다.


## 해결

그러면 어떻게 하면 js에서 angular의 내부 변수에 접근할 수 있을까요? 다시 말해서 어떻게 하면 js에서 angular에 값을 전달할 수 있을까요?

### 1. storage의 활용

local storage나 session storage를 활용할 수 있겠습니다. <br>
즉, js 에서 local storage에 값을 저장하면 이 값을 가져가는 방식입니다. <br><br>

이는 매우 간단히 문제를 해결할 수 있는 방법입니다.<br><br>

다만, storage에 저장된 시점을 알 수 없으므로 변수시로 체크하거나 값이 설정되었을 것이라고 가정하고 접근하여야 하는 또 다른 문제가 발생하므로 좋은 방법이라고 할 수는 없습니다.


### 2. global scope의 활용

angular와 js의 영역 상단의 전역 scope를 활용하는 방법입니다.<br>
window 객체에 변수를 선언하고 이를 통해 값을 전달할 수 있습니다.<br><br>

n초 뒤에 js에서 값이 변경되면 이를 감지하여 angular에 랜더링하는 코드를 작성해보겠습니다<br><br>

component에서 window에 pipe라는 객체를 만듭니다. 이를 통해 js -> angular로 값을 전달할 것입니다.
```ts
  // app.component.ts
    window['pipe'] = (value) => { 
      this.render = value;
    }    
```

이제 js에서 setTimeout을 통해 n초 뒤에 pipe에 값을 전달합니다.
```js
// global.js
var n = 1000;

setTimeout(() => {
	window.pipe('qwerqwf');
}, n);
```

이를 실행하면 잘 동작하는 것을 알 수 있습니다.



## 참고사이트

[자바스크립트 스코프(scope)](https://yuddomack.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%8A%A4%EC%BD%94%ED%94%84scope)

[[번역] 자바스크립트 스코프와 클로저(JavaScript Scope and Closures)](https://medium.com/@khwsc1/%EB%B2%88%EC%97%AD-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%8A%A4%EC%BD%94%ED%94%84%EC%99%80-%ED%81%B4%EB%A1%9C%EC%A0%80-javascript-scope-and-closures-8d402c976d19)