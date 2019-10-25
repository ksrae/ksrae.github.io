---
title: "All Error Handler Class 만들기"
comments: true
categories: angular
tags: [error, module]
date: 2019-07-01 17:37:00 +0900
---


Angular 에러 발생 이벤트를 모두 캐치하는 클래스를 만들어보겠습니다. <br>


## 1. 사용방법

내장함수인 ErrorHandler를 활용하며, 최상위 모둘에 적용합니다.<br><br>
에러를 처리하는 handleError 함수를 사용하여 에러 처리를 한 곳에서 진행할 수 있습니다.<br>

## 2. 코드

```ts
class MyErrorHandler implements ErrorHandler {
  handleError(error) {
    // do something with the exception
  }
}

@NgModule({
  providers: [{provide: ErrorHandler, useClass: MyErrorHandler}]
})
class AppModule {}
```

<br>
끝.