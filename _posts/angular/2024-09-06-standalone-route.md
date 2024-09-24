---
title: "Standalone Routing"
date: 2024-09-06 11:10:00 +0900
comments: true
categories: angular
tags: [standalone, routing]
---

Angular Standalone에서는 특히 라우팅(Routing) 부분에서 많은 변화가 있습니다. 독립적인 컴포넌트 관리와 더불어 성능 최적화 및 코드 구조 단순화에 기여합니다. 이 글에서는 Standalone 방식의 라우팅을 살펴보고, 기존의 모듈 기반 라우팅과 어떻게 다른지 설명하겠습니다.

## Angular의 전통적인 모듈 기반 라우팅
전통적인 모듈 방식에서는 다음과 같이 라우팅이 구성됩니다:

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { CategoryComponent } from './category.component';
import { CategoryListComponent } from './category-list.component';

const routes: Routes = [
  { path: '', component: CategoryListComponent },
  { path: ':name', component: CategoryComponent },
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
})
export class CategoryRoutingModule {}
```

위의 예제에서는 라우트 경로가 RouterModule에 의해 모듈 내에서 관리되고 있습니다. 모듈을 통해 라우팅 설정을 분리하고, 필요 시 forChild() 메서드를 이용해 서브 모듈 간의 라우팅을 처리합니다.

## Standalone 컴포넌트 방식의 라우팅
Standalone 컴포넌트 방식에서는 기존의 NgModule 없이 컴포넌트와 라우팅을 독립적으로 관리할 수 있습니다. 이러한 구조는 모듈을 선언하지 않아도 컴포넌트와 서비스를 개별적으로 로드할 수 있게 하며, Angular 애플리케이션의 초기 로딩 속도를 최적화하고 코드 구조를 간소화하는 데 큰 장점이 있습니다.

Standalone 방식에서 라우팅을 설정하는 방법은 다음과 같습니다:

### 1. Default Export 활용
Angular의 라우터는 import()에서 default export가 포함된 경우 이를 자동으로 처리하므로, .then()을 생략할 수 있습니다. 즉, 아래와 같은 코드로 lazy loading을 구성할 수 있습니다:

```typescript
// 메인 애플리케이션의 routes.ts:
export const ROUTES: Route[] = [
  { path: 'admin', loadChildren: () => import('./admin/routes') },
];

// admin/routes.ts:
export default [
  { path: 'home', component: AdminHomeComponent },
  { path: 'users', component: AdminUsersComponent },
] satisfies Route[];
```

### 2. Lazy Loading 방식
Standalone 컴포넌트 방식에서는 lazy loading을 보다 유연하게 사용할 수 있습니다. 기존의 loadChildren 방식이 아닌 **loadComponent**를 사용하여, 모듈이 아닌 컴포넌트를 개별적으로 지연 로딩할 수 있습니다. 이는 성능 최적화와 코드 스플리팅에 있어 매우 효율적입니다.

```typescript
import { Routes } from "@angular/router";

export const routes: Routes = [
  {
    path: "",
    pathMatch: "full",
    component: CategoryListComponent,
  },
  {
    path: ":name",
    loadComponent: () => import("./category.component").then(m => m.CategoryComponent),
  },
];
```

## 라우트 수준의 Providers 설정
Standalone 방식의 또 다른 특징은 라우트 수준에서 providers를 직접 정의할 수 있다는 점입니다. 전통적인 모듈 방식에서는 providers는 모듈 단위로 선언되었지만, Standalone 방식에서는 각 라우트별로도 providers를 정의할 수 있습니다. 이를 통해 특정 라우트에서만 필요한 서비스나 상태 관리 객체를 그 라우트에 한정하여 제공할 수 있습니다.

```typescript
export const routes: Routes = [
  {
    path: "",
    component: CategoryListComponent,
    providers: [CategoryHttpClient, CategoryState],  // 라우트별로 서비스 제공
  },
  {
    path: ":name",
    component: CategoryComponent,
  },
];
```

## 라우트에서 Provider Scoping
Standalone Route에서는 Provider Scoping을 명시적으로 설정할 수 있어, Lazy Loading을 사용하지 않고도 특정 라우트에만 서비스를 제공할 수 있습니다. 이 방식은 불필요한 NgModule 사용을 줄여주며, 원하는 라우트에만 서비스를 쉽게 제공할 수 있습니다.

### 1. Provider Scoping 예시
예를 들어 /admin 경로에만 특정 서비스를 제공하고 싶다면, 아래와 같이 providers 속성을 사용하여 해당 경로의 서비스 범위를 설정할 수 있습니다.

```typescript
export const ROUTES: Route[] = [
  {
    path: 'admin',
    providers: [
      AdminService,  // Admin 전용 서비스
      { provide: ADMIN_API_KEY, useValue: '12345' },  // Admin API 키 제공
    ],
    children: [
      { path: 'users', component: AdminUsersComponent },
      { path: 'teams', component: AdminTeamsComponent },
    ],
  },
  // 다른 경로들은 AdminService 또는 ADMIN_API_KEY에 접근 불가
];
```

위 예시에서는 /admin 경로 하위의 모든 자식 라우트(users, teams)에 대해 AdminService와 ADMIN_API_KEY를 사용할 수 있습니다. 하지만 /admin 외부의 다른 라우트는 해당 서비스나 값을 사용할 수 없습니다.

### loadChildren 없이 Provider Scoping 활용하기
Lazy Loading이 필요하지 않은 경우에도 providers 속성을 사용하여 라우트 범위 내에서만 특정 서비스를 제공할 수 있게 되었습니다. 이에 대한 장점은 다음과 같습니다.

- 서비스 범위 명시적 관리: 특정 경로에만 서비스를 제공하여 메모리 효율성을 높일 수 있습니다.
- Lazy Loading이 불필요한 경우: 굳이 Lazy Loading을 사용하지 않아도, 라우트 단위로 서비스 제공 범위를 관리할 수 있습니다.

## Lazy Loading과 Provider Scoping의 결합
Lazy Loading과 Provider Scoping은 서로 결합하여 사용할 수도 있습니다. 예를 들어, /admin 경로를 lazy loading으로 로드하면서 해당 경로 내의 서비스 범위를 정의하는 방식입니다.

```typescript
export const ROUTES: Route[] = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes'),
    providers: [
      AdminService,
      { provide: ADMIN_API_KEY, useValue: 'admin-key' },
    ],
  },
];
```

이 방식에서는 admin 경로를 lazy loading으로 동적 로드하고, 해당 경로 내에서만 AdminService와 ADMIN_API_KEY를 제공할 수 있습니다.

## Guard와 함께 사용하는 Provider Scoping
Provider Scoping을 활용한 라우팅에서 Guard를 추가로 사용하여 경로에 대한 접근을 제한할 수도 있습니다. 예를 들어, /admin 경로에 진입하기 전에 특정 조건을 만족해야 한다면, canActivate를 사용하여 접근 권한을 제어할 수 있습니다.
더 이상 guard는 class의 구조를 가지지 않고, Route[] 타입의 변수형을 갖습니다.

```typescript
import { AuthGuard } from './auth.guard';

export const ROUTES: Route[] = [
  {
    path: 'admin',
    providers: [
      AdminService,
      { provide: ADMIN_API_KEY, useValue: 'admin-key' },
    ],
    canActivate: [AuthGuard],  // AuthGuard 적용
    loadChildren: () => import('./admin/admin.routes'),
  },
];
```
위 코드에서는 AuthGuard를 통해 /admin 경로에 대한 접근을 제한하고 있습니다. Guard는 경로에 접근하기 전에 조건을 확인하여, 해당 조건을 만족하지 않으면 접근을 차단할 수 있습니다.
