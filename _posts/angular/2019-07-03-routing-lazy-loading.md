---
title: "Lazy Loading"
date: 2019-07-01 15:00:00 +0900
comments: true
categories: angular
tags: [routing]
---

## 1. 목적
- Module 또는 Component 단위로 Lazy Loading 하는 방법과 장단점을 정리한다.

## 2. Lazy Loading
- Lazy Loading은 흔히 용량이 파일을 로딩하는 경우 페이지의 다른 부분을 호출하고 나중에 파일 로딩이 끝나면 페이지에 적용하는 방식으로 많이 이용하는데 Angular는 Module 단위로 Lazy Loading할 수 있다.

## 3. 장단점
### 3.1. 장점
- 필요한 부분만 로딩하므로 초기 로딩의 부하를 분산할 수 있다.

### 3.2. 단점
- 페이지보다 늦게 로딩 되므로 로딩이 완료되기 전까지 사용자에게 
- Loading 되기 전의 Observable 데이터는 접근할 수 없고 다음 데이터부터 받을 수 있다.
(이는 Behavior Subject 방식이나 Replay Subject 방식을 활용하면 해결할 수 있다.)

## 4. Lazy Loading 구현
### 4.1. Module Routing으로 구현

app.routing.ts의 loadChildren과 material.routing.ts의 routing 부분만 참고하면 된다.

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
  { path: 'material', loadChildren: './modules/material/material.module#MaterialModule' }

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

### 4.2. component를 ngIf로 호출하여 Lazy Loading 구현
ngIf가 false 인 경우 브라우저가 해당 dom을 로드하지 않으므로 dom을 구성하는데 필요한 관련 데이터도 로드하지 않는다. ngIf가 true가 되었을 때 비로소 해당 dom의 모든 데이터를 로드하므로, 이 또한 Lazy Loading의 또 다른 기법이라 할 수 있다. module의 그것과 동일한 효과를 볼 수 있다.
component와 dom의 하나하나까지 세분화하여 관리할 수 있는 장점이 있으나 dom의 enable을 직접 컨트롤 해야하는 점이 있다.

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

## 5. 주의사항
Lazy Loading을 Routing으로 구현하는 경우 BrowserModule은 최상위 Module만 사용해야 하며 하위 Module에서는 같은 기능을 사용하기 위해 CommonModule을 import 해야 한다.