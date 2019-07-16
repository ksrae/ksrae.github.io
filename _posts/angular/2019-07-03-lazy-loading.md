---
title: "Lazy Loading Module and Component"
date: 2019-07-01 15:00:00 +0900
comments: true
categories: angular
tags: [routing]
---

Module 또는 Component 단위로 Lazy Loading 하는 방법을 정리합니다.

## 1. Lazy Loading
SPA의 특성상 모든 파일이 로드 되어야 서버의 호출 없이 다음 페이지로의 전환이 이루어 지는데 이를 Lazy Loading으로 처리하여 초기 로딩을 분산하는 방법입니다.
기존 로딩 방식과 다른 점은 모듈 로드 시 해당 모듈의 경로만 지정하는 점이 다르며, 부모가 자식 모듈을 호출하지 않는다는 점입니다.


## 2. 장단점
### 2.1. 장점
- 필요한 부분만 로딩하므로 초기 로딩의 부하를 분산할 수 있습니다.
- 더 이상 부모 모듈이 자식 모듈을 호출하지 않아도 됩니다. 즉, 느슨한 연결로 구성하므로 서로 독립적으로 구성할 수 있으며 쉽게 붙이거나 뗄 수 있도록 구성됩니다.


### 2.2. 단점
- 첫 로딩 시 한꺼번에 저장해두는 방식 보다 늦게 로딩 되므로 즉시 전환보다는 약간의 딜레이가 생깁니다. 
- 사용자는 Loading 되기 전의 Observable 데이터에 접근할 수 없고 Loading이 완료된 다음 데이터부터 받을 수 있습니다.

## 3. Lazy Loading 구현
### 3.1. Module Routing으로 구현

app.routing.ts의 loadChildren과 material.routing.ts의 routing 부분만 참고하시면 됩니다.

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

### 3.2. component를 ngIf로 호출하여 Lazy Loading 구현

ngIf가 false 인 경우 브라우저가 해당 dom을 로드하지 않으므로 dom을 구성하는데 필요한 관련 데이터도 로드하지 않는 방법을 이용하는 것입니다.
ngIf가 true가 되었을 때 비로소 해당 dom의 모든 데이터를 로드하므로, module의 Lazy Loading 방식과 동일한 효과를 볼 수 있습니다.
component와 dom의 하나하나까지 세분화하여 관리할 수 있는 장점이 있으나 dom의 enable을 직접 컨트롤 해야하는 유의점이 생깁니다.

```html 
<!-- <main.html> -->
<material *ngIf="isEnabled"></material>
```

```ts
// <material.component.ts>

import { Component, OnInit, OnDestroy, ChangeDetectionStrategy, ChangeDetectorRef } from '@angular/core';
import { RouterLink } from '@angular/router';

@Component({
  selector: 'material',
  template: `
  <h1>hello material world</h1>
  `,

  changeDetection: ChangeDetectionStrategy.OnPush
})

export class MaterialComponent implements OnInit {

  constructor(
    private cd: ChangeDetectorRef) { }

  ngOnInit() {}

  ngOnDestroy() {}

}
```

## 4. 주의사항
Lazy Loading을 Routing으로 구현하는 경우 BrowserModule은 최상위 Module만 사용해야 하며 하위 Module에서는 같은 기능을 사용하기 위해 CommonModule을 import 해야 합니다.