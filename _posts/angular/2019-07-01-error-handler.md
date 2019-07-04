---
title: "ErrorHandler"
comments: true
categories: Angular
tags: [error]
date: 2019-07-01 17:37:00 +0900
---



Angular 에러 발생 이벤트를 모두 캐치하여 처리할 수 있다.

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