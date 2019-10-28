---
title: "@ng-toolkit/universal ERROR: Cannot read property 'unshift' of undefined"
date: 2019-08-19 17:16:00 +0900
comments: true
categories: angular
tags: [universal]
---

@ng-toolkit/universal ERROR: Cannot read property 'unshift' of undefined 에러에 대해 알아보겠습니다.<br><br>

Angular universal에서 window를 사용하기 위해 @ng-toolkit/universal 을 설치해야 합니다.

```
ng add @ng-toolkit/universal
```

그런데 Angular 8 에서는 설치하려고 시도하면 unshift 에러가 발생한다.

```
@ng-toolkit/universal ERROR: Cannot read property 'unshift' of undefined
```

검색해보니 버전의 문제인 것으로 파악되었습니다.<br><br>

ng add로 설치해보면 @ng-toolkit/universal 버전이 1.1.21 이 설치 되는데 <br>이는 너무 낮은 버전으로
최신 버전인 7.1.2를 설치해야 위의 에러가 해결 됩니다.


참고: 
[ERROR: Cannot read property 'unshift' of undefined i'm getting this error when adding SSR to exist angular 6 project · Issue #578 · maciejtreder/ng-toolkit · GitHub](https://github.com/maciejtreder/ng-toolkit/issues/578)