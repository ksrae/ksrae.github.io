---
title: "Angular 8 New Lazy Loading Routing"
date: 2019-07-11 12:55:00 +0900
comments: true
categories: angular
tags: [routing, lazyloading]
---



Angular의 Routing Lazy Loading 방법이 변경되었습니다.<br><br>

이는 Ivy에 동적 import 방식을 적용하기 위함이며, 기존 방식도 여전히 유효하나 앞으로 새로운 방식을 유지할 것으로 보입니다.<br>


## 변경 사항

기존 lazy loading 방식은 텍스트로 해당 모듈까지의 경로 및 모듈명까지 입력하는 방식이므로 사실 보기 좋은 방식은 아니었습니다. <br><br>
새로 바뀐 방식에서는 모듈의 링크를 import에 기입하고 promise로 해당 모듈을 호출하는 방식입니다.<br>


```ts
// 기존
{ path: 'about', loadChildren: '../about/about.module#AboutModule', data: {preload: true} },

// 변경
{ path: 'about', loadChildren: () => import('../about/about.module').then(m => m.AboutModule), data: {preload: true} },
```

Lazy Loading이 아닌 기존 Routing 은 그대로 유지 되며, Ivy를 적용하지 않는 경우 어느 방식을 사용하여도 관계 없습니다.<br><br>


참고로 8.1 버전에서는 변경된 방식을 사용하여도 빌드시 Ivy에서 에러가 발생합니다.

      ERROR in Unable to write a reference to AboutComponent in c:/about.component.ts from c:/about.module.ts


Ivy를 적용하기에는 아쉽게도 이런 저런 에러가 아직 남아 있어 production 에서 사용하기에는 적합하지 않은 것 같습니다.


## 업데이트
위의 Ivy 에러는 이후 버전에서 해결되었으므로 Ivy에서도 사용할 수 있습니다.

## 참고 사이트
- [Lazy Loaded Routes Angular V8 with Ivy](https://fireship.io/snippets/lazy-loaded-routes-angular-v8-ivy/)