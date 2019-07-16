---
title: "Componentless Routing"
date: 2019-07-04 11:59:00 +0900
comments: true
categories: angular
tags: [routing]
---

### 분석

- Routing을 설정하면 Component를 일반적으로 아래와 같이 설정합니다.
    ```js
     {path: '', component: AppComponent, children: [
      { path: '', redirectTo: 'main', pathMatch: 'full' },
      { path: 'main', component: MainComponent },       
     ...
     ]}
    ```

이 때 / 나 /main을 입력하여 MainComponent에 접근하려면 반드시 AppComponent를 거친 후 redirectTo에 의해 MainComponent를 호출하게 됩니다. 
만일 Bootstrap으로 구성하는 경우라면 AppComponent가 Bootstrap에 의해 1번, Routing에 의해 2번 호출되어 이중 호출되는 경우가 발생하게 됩니다.

또한, AppComponent가 별다른  기능없이 routing만 관여하는 \<router-outlet>\</router-outlet> 만 있는 parentComponent인 경우라면 Componentless Routing을 활용하여 아예 무의미한 component 생성을 피할 수 있는 장점도 있습니다.


### 해결 방법

해결 방법은 바로 Componentless Routing 을 사용하는 것입니다.
대단한 방법은 아니고 단순히 부모의 component를 제거하면 됩니다.
이러면 해당 페이지의 root 는 없고 children에서 설정한 redirectTo 페이지로 바로 접근할 수 있습니다.
부모가 없으므로 반드시 부모 대신 default로 받아줄 자식을 redirectTo로 지정하여야 합니다.

  ```js  
   {path: '', children: [
    { path: '', redirectTo: 'main', pathMatch: 'full' },
	  { path: 'main', component: MainComponent },
   ...
   ]}
  ```

이 경우 MainComponent에 접근하려는 경우 / 또는 /main을 입력하였을 때 redirectTo에 의해 MainComponent를 호출하는데 부모가 없으므로 호출한 대상의 \<router-outlet>\</router-outlet> 에 포함되게 됩니다.
따라서 조금 더 간단한 routing과 component를 구성할 수 있을 것입니다.

끝.


