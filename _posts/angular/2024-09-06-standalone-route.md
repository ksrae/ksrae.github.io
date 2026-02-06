---
title: "Standalone Routing"
date: 2024-09-06 11:10:00 +0900
comments: true
categories: angular
tags: [standalone, routing]
---

Angular Standalone components introduce significant changes, particularly in routing. They contribute to independent component management, performance optimization, and code structure simplification. This post explores Standalone routing and contrasts it with traditional module-based routing.

## Traditional Module-Based Routing in Angular

In the traditional module approach, routing is structured as follows:

```tsx
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

In this example, route paths are managed within a module by the RouterModule. Routing configurations are separated via modules, and routing between submodules is handled using the `forChild()` method when necessary.

## Standalone Component-Based Routing

The Standalone component approach enables independent management of components and routing without the traditional NgModule. This structure allows components and services to be loaded individually without declaring modules, significantly optimizing the initial loading speed of Angular applications and simplifying the code structure.

Here's how to configure routing in Standalone mode:

### 1. Leveraging Default Exports

The Angular router automatically handles default exports in `import()`, allowing you to omit `.then()`.  You can configure lazy loading with the following code:

```tsx
// Main application's routes.ts:
export const ROUTES: Route[] = [
  { path: 'admin', loadChildren: () => import('./admin/routes').then(m => m.adminRoutes) },
];

// admin/routes.ts:
export default [
  { path: 'home', component: AdminHomeComponent },
  { path: 'users', component: AdminUsersComponent },
] satisfies Route[];
```

### 2. Lazy Loading Approach

Standalone components enable more flexible lazy loading. Instead of the traditional `loadChildren` approach, you can use `loadComponent` to individually lazy load components rather than modules. This is highly efficient for performance optimization and code splitting.

```tsx
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

## Route-Level Providers Configuration

Another feature of Standalone mode is the ability to directly define providers at the route level.  In the traditional module approach, providers were declared at the module level. In Standalone mode, providers can be defined for each route, allowing you to provide services or state management objects needed only for that specific route.

```tsx
export const routes: Routes = [
  {
    path: "",
    component: CategoryListComponent,
    providers: [CategoryHttpClient, CategoryState],  // Providing services per route
  },
  {
    path: ":name",
    component: CategoryComponent,
  },
];
```

## Provider Scoping in Routes

Standalone Routes allow explicit configuration of Provider Scoping, enabling service provision to specific routes without using Lazy Loading. This reduces the need for unnecessary NgModules and simplifies service provision to desired routes.

### 1. Provider Scoping Example

To provide a specific service only to the `/admin` route, use the `providers` property to set the service scope for that route:

```tsx
export const ROUTES: Route[] = [
  {
    path: 'admin',
    providers: [
      AdminService,  // Admin-specific service
      { provide: ADMIN_API_KEY, useValue: '12345' },  // Providing Admin API key
    ],
    children: [
      { path: 'users', component: AdminUsersComponent },
      { path: 'teams', component: AdminTeamsComponent },
    ],
  },
  // Other routes cannot access AdminService or ADMIN_API_KEY
];
```

In this example, `AdminService` and `ADMIN_API_KEY` are available to all child routes (users, teams) under the `/admin` route. However, other routes outside of `/admin` cannot use these services or values.

### Leveraging Provider Scoping Without `loadChildren`

Even when Lazy Loading is not required, the `providers` property can be used to provide specific services only within the route's scope. The benefits include:

- **Explicit Service Scope Management:** Services are provided only to specific paths, increasing memory efficiency.
- **When Lazy Loading is Unnecessary:**  Service provision scope can be managed per route without using Lazy Loading.

## Combining Lazy Loading and Provider Scoping

Lazy Loading and Provider Scoping can be combined.  For example, you can lazy load the `/admin` route while defining the service scope within that route.

```tsx
export const ROUTES: Route[] = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes').then(m => m.adminRoutes),
    providers: [
      AdminService,
      { provide: ADMIN_API_KEY, useValue: 'admin-key' },
    ],
  },
];
```

In this approach, the `admin` route is dynamically loaded using lazy loading, and `AdminService` and `ADMIN_API_KEY` are provided only within that route.

## Using Provider Scoping with Guards

In routing with Provider Scoping, Guards can be added to restrict access to routes. For example, if certain conditions must be met before entering the `/admin` route, `canActivate` can be used to control access permissions.
The guard no longer has a class structure, but has a Route[] type variable.

```tsx
import { AuthGuard } from './auth.guard';

export const ROUTES: Route[] = [
  {
    path: 'admin',
    providers: [
      AdminService,
      { provide: ADMIN_API_KEY, useValue: 'admin-key' },
    ],
    canActivate: [AuthGuard],  // Applying AuthGuard
    loadChildren: () => import('./admin/admin.routes').then(m => m.adminRoutes),
  },
];
```

In this code, access to the `/admin` route is restricted using `AuthGuard`. The Guard checks conditions before accessing the route and blocks access if the conditions are not met.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Standalone-Routing)
<br/>