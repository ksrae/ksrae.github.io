---
title: "Shared Module"
date: 2019-07-04 15:44:00 +0900
comments: true
categories: angular
tags: [module]
---

중복 사용되는 모듈을 Shared Module로 공유하는 방법 입니다.<br>




## 1. component를 여러 module에서 사용할 때

  - 다른 module에서 선언한 component를 또 다른 module에서 중복 선언하면 에러가 발생합니다.
  - 에러를 분석해보면 이러한 경우 상위 또는 다른 module에 선언하고 그 module을 참조하라고 되어 있습니다.
  - 매번 다른 module을 선언하면 복잡해지므로 공통적으로 사용할 shared module을 만들고 여기에 component를 선언한 뒤 중복 component를 사용하는 module에서 shared module을 참조하면 해결됩니다.
  (* 모든 module에서 shared module을 참조하도록 설정해도 됩니다.)


## 2. service가 여러 module에서 선언될 때 문제

  - service는 자주 여러 module에서 사용되는데 여러 module에서 선언되면 문제가 있습니다.
  - module마다 service를 선언하게 되면 이는 service를 각각의 module이 new를 하는 것과 마찬가지입니다.
  - 따라서 service도 component과 마찬가지로 shared module에 올린 뒤 각각의 module이 shared module을 참조하면 됩니다.
  - 그러나 일반적으로 아래와 같이 service를 선언하는 방법인 아래와 같이 사용한다면 기존 각각의 module에서 선언한 것과 동일하게 각각의 module이 new 하는 문제가 발생합니다.
    ```js
    providers: [
      SomeService
    ]
    ```

  - 따라서 이를 해결하기 위해 ModuleWithProviders 를 활용하여야 합니다.
    ``` js
    export class SharedModule {
      static forRoot(): ModuleWithProviders {
        return {
          SomeService
        }
      }
    }
    ```

  - module 중 최상위 module은 반드시 SharedModule.forRoot() 로 참조하고, 하위 module은 SharedModule 만 참조하도록 해야 합니다.
    

### 2-1. Angular 6+ 에서의 service

Angular 6이후로 service가 module에 선언하지 않고 service에 직접 선언하므로 공통 service는 module에서 선언하지 않습니다.<br>
service의 provideIn 설정을 root로 하거나 sharedModule을 설정하면 됩니다.<br>

```ts
@Injectable({
	providedIn: 'root' // 또는 sharedModule
})
```
