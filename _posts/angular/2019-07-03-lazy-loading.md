---
title: "Lazy Loading Module and Component"
date: 2019-07-01 15:00:00 +0900
comments: true
categories: angular
tags: [routing]
---

Module 또는 Component 단위로 Lazy Loading 하는 방법을 정리합니다. <br>



## 1. Lazy Loading

SPA의 특성상 모든 파일이 로드 되어야 서버의 호출 없이 다음 페이지로의 전환이 이루어 지는데<br>이를 Lazy Loading으로 처리하여 초기 로딩을 분산하는 방법입니다.<br><br>
기존 로딩 방식과 다른 점은 모듈 로드 시 해당 모듈의 경로만 지정하는 점이 다르며,<br>부모가 자식 모듈을 호출하지 않는다는 점입니다.<br>
<br>

## 2. 장단점

### 2.1. 장점

- 필요한 부분만 로딩하므로 초기 로딩의 부하를 분산할 수 있습니다.
- 더 이상 부모 모듈이 자식 모듈을 호출하지 않아도 됩니다. 즉, 느슨한 연결로 구성하므로 서로 독립적으로 구성할 수 있으며 쉽게 붙이거나 뗄 수 있도록 구성됩니다.


### 2.2. 단점

- 첫 로딩 시 한꺼번에 저장해두는 방식 보다 늦게 로딩 되므로 즉시 전환보다는 약간의 딜레이가 생깁니다. 
- 사용자는 Loading 되기 전의 Observable 데이터에 접근할 수 없고 Loading이 완료된 다음 데이터부터 받을 수 있습니다.

## 3. Lazy Loading 구현

### 3.1. Module Routing으로 구현

app.routing.ts의 loadChildren과 material.routing.ts의 routing 부분만 참고하시면 됩니다.<br>

```ts
// <app.module.ts>
import { NgModule } from '@angular/core';
import { AppRoutingModule } from './app.routing';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { MainComponent } from './common/ts/components/main.component';

import { MaterialModule } from './common/ts/modules/material/material.module';

@NgModule({
  declarations: [
    AppComponent,
    MainComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    MaterialModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

```ts
// <app.routing.ts>

import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { MainComponent } from './common/ts/components/main.component';

const routes : Routes = [
  { path : '', component : MainComponent  },
  { path: 'material', loadChildren: './modules/material/material.module#MaterialModule' } // 모듈을 lazy Loading 한다.

];

@NgModule({
  imports: [ RouterModule.forRoot(routes)],
  exports: [ RouterModule ]
})

export class AppRoutingModule { }
```
<br>

```ts
// <material.module.ts>

import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

import { MaterialRoutingModule } from './material.routing';
import { MaterialMainComponent } from './components/material/material.main.component';

@NgModule({
  exports: [
    MaterialMainComponent
  ],
  declarations: [    
    MaterialMainComponent
  ],
  imports: [
    CommonModule,
    MaterialRoutingModule
  ],
  providers: [

  ]

})

export class MaterialModule { }
```
<br>

```ts
// <material.routing.ts>

import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { MaterialMainComponent } from '../../components/material/material.main.component';

const materialRoutes: Routes = [
  { path: '', component: MaterialMainComponent }
];

@NgModule({
    imports: [ RouterModule.forChild(materialRoutes)],
    exports: [ RouterModule ]
  })

  export class MaterialRoutingModule { }
```

### 3.2. standalone에서의 lazy loading
standalone의 경우 두 가지 방법을 활용하여 component를 lazy loading할 수 있습니다.
첫번째는 loadComponent 입니다.

#### loadComponent
route에서 component를 직접 lazy load하도록 설정할 수 있습니다.

```ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    children: [
      { path: '', redirectTo: 'login', pathMatch: 'full'},
      { path: 'login', loadComponent: () => import('./login/login.component').then(m => m.LoginComponent) },
    ]
  }
];

```

#### loadChildren에서 route 호출
두번째 방법은 loadChildren에서 module이 아닌 route를 직접 호출하는 방식입니다.
이렇게 route를 직접 호출하면 하위 component들이 모두 lazy loading됩니다.
이 기능은 standalone에서만 사용할 수 있으므로 유의해야 합니다.

```ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    children: [
      { path: '', redirectTo: 'login', pathMatch: 'full'},
      { path: 'login', loadChildren: () => import('./login/login.routes').then(m => m.loginRoutes) },
    ]
  }
];

// login.routes
...
export const loginRoutes: Routes = [
  { path: '', component: LoginComponent }
];

```


### 3.3. component를 ngIf로 호출하여 Lazy Loading 구현

ngIf가 false 인 경우 브라우저가 해당 dom을 로드하지 않으므로 <br>dom을 구성하는데 필요한 관련 데이터도 로드하지 않는 방법을 이용하는 것입니다.<br><br>
ngIf가 true가 되었을 때 비로소 해당 dom의 모든 데이터를 로드하므로, <br>module의 Lazy Loading 방식과 동일한 효과를 볼 수 있습니다.<br><br>
component와 dom의 하나하나까지 세분화하여 관리할 수 있는 장점이 있으나 <br>dom의 enable을 직접 컨트롤 해야하는 유의점이 생깁니다.<br><br>

```html 
<!-- <main.html> -->
<material *ngIf="isEnabled"></material>
```
<br>

```ts
// <material.component.ts>

import { Component, OnInit, OnDestroy, ChangeDetectionStrategy, ChangeDetectorRef } from '@angular/core';
import { RouterLink } from '@angular/router';

@Component({
  standalone: true // module 코드라면 이 부분만 제거하면 됩니다.
  selector: 'material',
  template: `
  <h1>hello material world</h1>
  `,

  changeDetection: ChangeDetectionStrategy.OnPush
})

export class MaterialComponent implements OnInit {

  constructor() { }

  ngOnInit() {}

  ngOnDestroy() {}

}
```

## 4. 주의사항

standalone이 아닌 Module로 구성된 경우에는 Lazy Loading을 Routing으로 구현하는 경우에는 유의할 점이 있습니다.<br><br>

반드시 BrowserModule은 최상위 Module만 사용해야 합니다.<br>
또한 하위 Module에서는 같은 기능을 사용하기 위해 CommonModule을 import 해야 합니다.

## 5. 결론
Lazy Loading은 초기 로딩 속도를 크게 개선하며, 사용자 경험을 향상시키는 데 중요한 역할을 합니다. 특히 대규모 SPA(Single Page Application)에서 초기 로드 속도가 사용자 만족도에 직접적으로 영향을 미치는 점을 고려할 때, Lazy Loading은 반드시 도입해야 할 필수적인 설계 방식입니다.<br><br>

장점으로는 필요한 모듈과 컴포넌트만 로드하여 불필요한 리소스 낭비를 막을 수 있고, 부모와 자식 모듈 간의 의존성을 낮춤으로써 애플리케이션 구조를 보다 유연하고 독립적으로 관리할 수 있다는 점이 있습니다. 또한, standalone 방식에서도 간단한 구현으로 컴포넌트를 효율적으로 로드할 수 있어 코드의 가독성과 유지보수성도 크게 개선됩니다.<br><br>

다만, Lazy Loading을 도입할 때 약간의 로드 딜레이와 데이터 접근 제한 같은 단점이 있을 수 있으나, 적절한 설계와 로딩 경험 개선(예: 로딩 스피너)으로 충분히 극복 가능합니다.<br><br>

이러한 이점을 고려할 때, Lazy Loading은 대규모 애플리케이션 개발뿐만 아니라, 확장성과 성능이 중요한 모든 프로젝트에서 반드시 사용해야 하는 설계 방식입니다. 초기에 조금의 구현 노력이 필요할 수 있지만, 장기적으로 유지보수성과 사용자 경험에서 확실한 이점을 제공하는 Lazy Loading을 꼭 도입하시길 추천드립니다.