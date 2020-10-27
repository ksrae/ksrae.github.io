---
title: "빠른 Lazy Loading 방법  (How To Make Lazy Loading Faster)"
date: 2020-10-27 10:00:00 +0900
comments: true
categories: angular
tags: [lazyloading]
---


## 환경
Angular 10

  
## 분석
Lazy Loading은 초기 로딩 시간을 줄여주는 역할을 하는 장점이 있지만,
해당 페이지가 로딩될 때 SPA의 특징인 즉시 페이지가 변경되는 빠른 로딩이 되지 않는다는 문제가 있습니다.

이 두 가지를 모두 만족하는 방법은 없을까요?
여기 PreloadAllModules와 QuicklinkStrategy 라는 두 가지 방안을 제시합니다.



## PreloadingAllModules
기존의 모든 모듈을 전부 로딩하는 방법과는 다르게 
초기 로딩 시 모든 모듈을 실행 가능한 최소한의 양만 로드하여 대기하는 방법입니다.

이 방법은 매우 유용하며 빠른 속도와 기존보다 매우 작은 양의 네트워크 사용량을 가집니다.
그러나 대규모의 프로젝트에서는 여전히 많은 양의 네트워크 사용량을 필요로 하며, 
라우터는 모든 사전 로드된 모듈에 경로를 등록해야하기 때문에,
UI 스레드에서 집중적인 계산을 유발하고 사용자 경험 저하를 초래할 수 있습니다.

따라서, 라우터가 적은 소규모의 프로젝트에 추천합니다.

### 사용법

최초 RouterModule.forRoot를 선언할 때 옵션을 설정해주면 되며, 
PreloadAllModules는 @angular/roter에 내장되어 있습니다.

```tsx
import { RouterModule, Routes, PreloadAllModules } from '@angular/router';

@NgModule({
  imports: [
    RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules })
  ],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

## QuicklinkStrategy
QuicklinkStrategy는 Angular 기본 라이브러리가 아니므로 npm을 통해 다운 받아야 합니다.

```
npm i ngx-quicklink --save
```

QuicklinkStrategy는 대규모의 프로젝트에서 더 나은 대안일 수 있습니다.

동작원리는 다음과 같습니다.

> 1. IntersectionObserver Api를 통해 로직 화면에 표시되는 routerlink를 찾아냅니다.
> 2. requestIdleCallback을 통해 브라우저가 안정화 될 때까지 대기합니다.
> 3. navigator.connection.effectiveType을 통해 유저가 느린 연결 상태이거나 navigator.connection.saveData를 통해 데이터-절약 가능상태 (data-saver) 인지 파악합니다.
> 4. Angular의 사전 판독 계획 (prefetching strategy)을 통해 lazy load될 모듈을 준비합니다. 



QuicklinkStrategy는 quicklink와 3가지 면에서 다른 점이 있습니다.

> 1. quicklink는 가능한 경우 link[rel="prefetch"] 를 통해 리소스를 미리 가져오고, 그 뒤에 XMLHttpRequest로 대체하지만, ngx-quicklink는 Angular 라우터의 현재 모듈을 preloading 하는 원리(mechanism) 때문에 XMLHttpRequest만 사용합니다. 비록 Guess.js에서 사용하는 것과 같은 link[rel="prefetch"]를 사용하는 것이 더 나은 대안일 수 있으나 대부분의 경우에 유저들은 이 점을 발견하지 못할 것입니다.
> 2. ngx-quicklink는 단지 관련 스크립트 번들만 다운로드하는 것이 아니라 컨텐츠를 파싱하고 평가합니다. 이 방법은 유저가 페이지를 변경할 때 성능의 큰 향상을 가져옵니다.
> 3. ngx-quicklink는 모든 요청된 모듈을 사전 로드하기 위해 모든 부모 모듈을 다운로드 합니다. 

### 사용법

최초 RouterModule.forRoot를 선언할 때 옵션을 설정해주면 되며,
ngx-quicklink에서 import 해야 합니다.

```tsx
import { RouterModule, Routes } from '@angular/router';
import { QuicklinkStrategy } from 'ngx-quicklink';

@NgModule({
  imports: [
    RouterModule.forRoot(routes, { preloadingStrategy: QuicklinkStrategy })
  ],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```


## 참고 사이트
[quicklink-angular-prefetching-preloading-strategy](https://blog.mgechev.com/2018/12/24/quicklink-angular-prefetching-preloading-strategy/)

[route-preloading-in-angular](https://web.dev/route-preloading-in-angular/)