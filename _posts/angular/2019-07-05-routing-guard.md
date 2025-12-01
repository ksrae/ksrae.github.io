---
title: "Understanding Angular Route Guards: canActivate, canDeactivate, canActivateChild, and canLoad"
date: 2019-07-05 11:28:00 +0900
comments: true
categories: angular
tags: [routing, guard]
---

[한국어(Korean) Page](https://velog.io/@ksrae/%EB%9D%BC%EC%9A%B0%ED%8C%85-%EC%98%B5%EC%85%98%EA%B3%BC-Guard)
<br/>

This article explores the use of route guards in Angular, focusing on `canActivate`, `canDeactivate`, `canActivateChild`, and `canLoad`. We'll demonstrate how to implement these guards to control route access and module loading.

- `canLoad`: Prevents the loading of a module, primarily used with lazy-loaded modules. It executes only once when the module is initially loaded.
- `canActivate`: Protects routes by preventing access to unauthorized users. It runs every time a route is activated.
- `resolve`: Executes a function before a route is activated, providing pre-fetched data to the component.
- `canDeactivate`: Executes a function before navigating away from a route, allowing you to prevent the user from leaving the page under certain conditions.

### Reference Links

- **Redirecting with CanActivate:** [angular - Redirect to a different component inside @CanActivate in Angular2 - Stack Overflow](https://stackoverflow.com/questions/34711889/redirect-to-a-different-component-inside-canactivate-in-angular2)
- **CanActivate and CanActivateChild Explained:** [Routing Angular Applications: CanActivate and CanActivateChild ― Scotch](https://scotch.io/courses/routing-angular-2-applications/canactivate-and-canactivatechild)

While it's possible to combine multiple functionalities into a single guard, creating separate guards for each specific purpose promotes clarity and maintainability. For demonstration purposes, the following example combines several guard implementations into one.

## Routing Module Configuration

Within your routing module, configure the `resolve` option to execute a guard and assign its return value to a variable. The syntax is `{variableName: GuardName}`.

```tsx
import { MainGuard } from '../../guards/main.guard';
import { Routes } from '@angular/router';
import { MainComponent } from './main.component'; // Assuming MainComponent is in the same directory, adjust as necessary

const routes: Routes = [
  {
    path: '',
    canActivateChild: [MainGuard],
    children: [
      { path: '', redirectTo: 'main', pathMatch: 'full' },
      {
        path: 'main',
        component: MainComponent,
        canActivate: [MainGuard],
        resolve: { itemList: MainGuard },
        canDeactivate: [MainGuard],
      },
      {
        path: 'login',
        loadChildren: () => import('../login/login.module').then(m => m.LoginModule),
        canLoad: [MainGuard],
        data: { preload: true },
      },
    ],
  },
];
```

## Implementing the Guard

Create the guard class implementing the `CanLoad`, `CanActivate`, `CanActivateChild`, `Resolve`, and `CanDeactivate` interfaces.

```tsx
import { Injectable } from '@angular/core';
import {
  ActivatedRouteSnapshot,
  RouterStateSnapshot,
  CanLoad,
  CanActivate,
  CanActivateChild,
  Resolve,
  CanDeactivate,
  Route,
  UrlTree,
  Router,
} from '@angular/router';
import { Observable } from 'rxjs';
// service
import { ItemService } from '../services/item.service';
import { UserService } from '../services/user.service';
import { Item } from '../models/item.model'; // Adjust path as necessary

@Injectable({
  providedIn: 'root',
})
export class MainGuard
  implements CanLoad, CanActivate, CanActivateChild, Resolve<Item[]>, CanDeactivate<boolean>
{
  constructor(
    private itemService: ItemService,
    private userService: UserService,
    private router: Router,
  ) {}

  canLoad(route: Route): Observable<boolean> | Promise<boolean> | boolean {
    // Additional logic before loading the module.
    this.itemService.onInit();
    return true;
  }

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ):
    | Observable<boolean | UrlTree>
    | Promise<boolean | UrlTree>
    | boolean
    | UrlTree {
    if (!this.userService.userInfo || this.userService.userInfo.isGuest) {
      this.router.navigate(['/login']);
      return false;
    }
    return true;
  }

  canActivateChild(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ):
    | Observable<boolean | UrlTree>
    | Promise<boolean | UrlTree>
    | boolean
    | UrlTree {
    return true;
  }

  resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<Item[]> | Promise<Item[]> | Item[] {
    return this.itemService.get();
  }

  canDeactivate(): boolean {
    this.itemService.clear();
    return true;
  }
}
```

## Using Resolved Data in the Component

Access the data resolved by the guard in your component using the `ActivatedRoute`.

```tsx
import { Component, OnInit, ChangeDetectionStrategy, ChangeDetectorRef, NgZone } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';

@Component({
    selector: 'app-main',
    templateUrl: './main.component.html',  // Adjust path
    styleUrls: ['./main.component.scss'],  // Adjust path
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class MainComponent implements OnInit {
  itemList: any[];

  constructor(
    private router: Router,
    private route: ActivatedRoute,
    private cd: ChangeDetectorRef,
    private zone: NgZone
  ) {}

  ngOnInit() {
    this.itemList = this.route.snapshot.data['itemList'];
  }
}
```