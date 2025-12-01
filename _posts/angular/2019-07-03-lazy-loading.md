---
title: "Lazy Loading Module and Component"
date: 2019-07-01 15:00:00 +0900
comments: true
categories: angular
tags: [routing]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Module%EC%9D%B4%EB%82%98-Component%EB%A5%BC-Lazy-Loading-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
<br/>

# Angular Lazy Loading: A Comprehensive Guide to Module and Component Implementation

Lazy loading is a crucial performance optimization technique in Angular applications that can significantly improve initial load times and overall application efficiency. In this comprehensive guide, we'll explore different approaches to implementing lazy loading at both the module and component levels, including modern standalone components.

## Understanding Lazy Loading in Angular

In Single Page Applications (SPAs), traditionally all application code would be loaded upfront, resulting in longer initial load times. Lazy loading addresses this by loading features on-demand, splitting your application into smaller chunks that are loaded only when needed.

## Benefits and Trade-offs

### Advantages
- **Optimized Initial Load**: Only essential modules are loaded at startup, significantly reducing the initial bundle size
- **Better Resource Management**: Resources are loaded only when needed, improving memory usage
- **Loose Coupling**: Modules become more independent, making the application more maintainable
- **Enhanced Scalability**: Easier to manage large applications with multiple features
- **Improved Caching**: Smaller bundles can lead to more efficient browser caching

### Potential Challenges
- **Navigation Delay**: First access to lazy-loaded routes may experience a slight delay
- **State Management**: Need careful consideration of state management across lazy-loaded modules
- **Data Access**: Observable data isn't available until the module is loaded

## Implementation Approaches

### 1. Module-based Lazy Loading

The traditional approach involves configuring routes to lazy load entire feature modules. Here's a complete example:

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { AppRoutingModule } from './app.routing';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';
import { MainComponent } from './common/ts/components/main.component';

@NgModule({
  declarations: [
    AppComponent,
    MainComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }

// app.routing.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

const routes: Routes = [
  { 
    path: 'material', 
    loadChildren: () => 
      import('./modules/material/material.module').then(m => m.MaterialModule) 
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### 2. Standalone Component Lazy Loading

Angular's standalone components provide two modern approaches to lazy loading:

#### Using loadComponent

```typescript
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: 'dashboard',
    loadComponent: () => 
      import('./dashboard/dashboard.component')
        .then(m => m.DashboardComponent)
  },
  {
    path: 'profile',
    loadComponent: () => 
      import('./profile/profile.component')
        .then(m => m.ProfileComponent)
  }
];

// dashboard.component.ts
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-dashboard',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="dashboard">
      <h1>Dashboard</h1>
      <!-- Dashboard content -->
    </div>
  `
})
export class DashboardComponent {
  // Component logic
}
```

#### Using Route-based Lazy Loading

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => 
      import('./admin/admin.routes')
        .then(m => m.adminRoutes)
  }
];

// admin/admin.routes.ts
export const adminRoutes: Routes = [
  {
    path: '',
    component: AdminLayoutComponent,
    children: [
      {
        path: 'users',
        loadComponent: () => 
          import('./users/users.component')
            .then(m => m.UsersComponent)
      },
      {
        path: 'settings',
        loadComponent: () => 
          import('./settings/settings.component')
            .then(m => m.SettingsComponent)
      }
    ]
  }
];
```

### 3. Component-Level Lazy Loading with ngIf

While not technically lazy loading in the routing sense, using `ngIf` can achieve similar benefits for component-level code splitting:

```typescript
// lazy.component.ts
@Component({
  selector: 'app-lazy-component',
  standalone: true,
  template: `
    <div *ngIf="showComponent">
      <heavy-component></heavy-component>
    </div>
    <button (click)="toggleComponent()">Toggle Component</button>
  `
})
export class LazyComponent {
  showComponent = false;

  toggleComponent() {
    this.showComponent = !this.showComponent;
  }
}
```

## Best Practices and Considerations

1. **Module Structure**
   - Keep feature modules independent and self-contained
   - Use shared modules wisely to avoid duplicate code loading
   - Consider using micro-frontends for very large applications

2. **Performance Optimization**
   - Implement preloading strategies for frequently accessed modules
   - Use route data preloading for better UX
   - Monitor bundle sizes using tools like webpack-bundle-analyzer

3. **State Management**
   - Plan state management carefully across lazy-loaded modules
   - Consider using NgRx or other state management solutions
   - Implement proper cleanup in lazy-loaded modules

4. **Error Handling**
   - Implement proper error handling for module loading failures
   - Provide fallback UI for loading states
   - Consider retry strategies for failed loads

## Common Pitfalls to Avoid

1. **BrowserModule Usage**
   - Import BrowserModule only in the root module
   - Use CommonModule in feature modules
   - Avoid duplicate provider declarations

2. **Circular Dependencies**
   - Be careful with cross-module dependencies
   - Use proper module structuring
   - Consider using service interfaces

3. **Over-optimization**
   - Don't lazy load very small modules
   - Consider the trade-off between load time and navigation delay
   - Profile your application to make informed decisions

## Conclusion

Lazy loading is an essential technique in modern Angular applications, offering significant performance benefits and improved user experience. While it requires careful planning and implementation, the advantages far outweigh the challenges. The introduction of standalone components has made lazy loading even more flexible and easier to implement.

When implementing lazy loading, consider your application's specific needs, user patterns, and performance requirements. Use the appropriate strategy - whether it's module-based, standalone components, or component-level lazy loading with ngIf - based on your use case.

By following the best practices and considering the trade-offs discussed in this guide, you can successfully implement lazy loading in your Angular applications and achieve optimal performance and maintainability.