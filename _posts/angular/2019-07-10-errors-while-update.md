---
title: "Errors while Update"
date: 2019-07-10 12:00:00 +0900
comments: true
categories: angular
tags: [update, error]
---


ng update를 진행 한 뒤 발생하는 다양한 에러를 나열하고 해결 방법을 정리합니다.


### 1. Typescript 버전 에러

업데이트 후 실행 시 아래와 유사한 에러가 발생합니다.

    ERROR in The Angular Compiler requires TypeScript >=3.4.0 and <3.5.1 but 3.5.1 was found instead.
    
#### 분석

typescript 버전이 angular/cli와 맞지 않는다는 에러 입니다.


#### 해결

기존 typescript를 제거하고 재설치 하는 방법입니다.

        npm uninstall typescript

        npm install typescript@">=3.4.0 <3.5.0" --save-dev --save--exact

재설치 시 범위를 에러 메시지와 동일하게 적용할 시 해당 범위의 가장 높은 버전이 설치 되어 에러가 해결됩니다.


### 2. @angular-devkit/build-angular 버전 에러

업데이트 후 실행 시 아래의 에러가 발생합니다.

```
Schema validation failed with the following errors:
  Data path ".builders['app-shell']" should have required property 'class'.
Error: Schema validation failed with the following errors:
  Data path ".builders['app-shell']" should have required property 'class'.
    at MergeMapSubscriber._registry.compile.pipe.operators_1.concatMap.validatorResult [as project] (C:\Users\ksrae\AppData\Roaming\npm\node_modules\@angular\cli\node_modules\@angular-devkit\core\src\workspace\workspace.js:215:42)
    at MergeMapSubscriber._tryNext (C:\Users\ksrae\AppData\Roaming\npm\node_modules\@angular\cli\node_modules\rxjs\internal\operators\mergeMap.js:69:27)
    at MergeMapSubscriber._next (C:\Users\ksrae\AppData\Roaming\npm\node_modules\@angular\cli\node_modules\rxjs\internal\operators\mergeMap.js:59:18)
    at MergeMapSubscriber.Subscriber.next (C:\Users\ksrae\AppData\Roaming\npm\node_modules\@angular\cli\node_modules\rxjs\internal\Subscriber.js:67:18)
    at MergeMapSubscriber.notifyNext (C:\Users\ksrae\AppData\Roaming\npm\node_modules\@angular\cli\node_modules\rxjs\internal\operators\mergeMap.js:92:26)
    at InnerSubscriber._next (C:\Users\ksrae\AppData\Roaming\npm\node_modules\@angular\cli\node_modules\rxjs\internal\InnerSubscriber.js:28:21)
    at InnerSubscriber.Subscriber.next (C:\Users\ksrae\AppData\Roaming\npm\node_modules\@angular\cli\node_modules\rxjs\internal\Subscriber.js:67:18)
    at MapSubscriber._next (C:\Users\ksrae\AppData\Roaming\npm\node_modules\@angular\cli\node_modules\rxjs\internal\operators\map.js:55:26)
    at MapSubscriber.Subscriber.next (C:\Users\ksrae\AppData\Roaming\npm\node_modules\@angular\cli\node_modules\rxjs\internal\Subscriber.js:67:18)
    at SwitchMapSubscriber.notifyNext (C:\Users\ksrae\AppData\Roaming\npm\node_modules\@angular\cli\node_modules\rxjs\internal\operators\switchMap.js:86:26)
```

#### 분석

 에러 메시지에서는 파악할 수 없으나 @angular-devkit/build-angular 버전이 angular/cli와 맞지 않아 발생하는 에러 입니다.


#### 해결

기존 @angular-devkit/build-angular를 제거하고 재설치 하는 방법입니다.

        npm uninstall @angular-devkit/build-angular

        npm install @angular-devkit/build-angular@0.13.8


다양한 버전이 있으나 2019년 7월 기준으로 적용 가능한 가장 최신 버전입니다.