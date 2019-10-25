---
title: "라우팅 옵션과 Guard (Routing Options and Guard)"
date: 2019-07-05 11:28:00 +0900
comments: true
categories: angular
tags: [routing, guard]
---


route에서 canActivate와 canDeactivate, canActivateChild, canLoad 옵션처리 방식을 알아보고, <br> 
함께 Guard 사용 방법을 알아보겠습니다. <br>

- canLoad: 모듈 사용을 방지. LazyLoad 된 모듈에서만 사용한다. canLoad는 모듈이 최초 로드되는 한 번만 실행.
- canActivate: 인증되지 않은 사용자를 방지한다. 모듈이 호출될 때마다 실행.
- resolve: 모듈이 로드 되기 전 함수를 미리 실행하여 결과를 리턴.
- canDeactivate: 모듈이 종료되기 전 함수를 실행.

<br>

-- router에서 로그인 인증하고 로그인 유저가 아니면 페이지 강제 이동하는 방법
[angular - Redirect to a different component inside @CanActivate in Angular2 - Stack Overflow](https://stackoverflow.com/questions/34711889/redirect-to-a-different-component-inside-canactivate-in-angular2)

<br>

canActivate와 canActivateChild
[Routing Angular Applications: CanActivate and CanActivateChild ― Scotch](https://scotch.io/courses/routing-angular-2-applications/canactivate-and-canactivatechild)

<br>
<br>
<br>
여러 기능을 모아서 하나의 guard에 구성해도 관계없으나 필요한 기능마다 별도의 guard를 만들어 두는게 가드의 목적을 명확히 할 수 있습니다.<br><br>
예시에서는 칸을 줄이기 위해 하나의 guard에 모두 구성해 보겠습니다.<br>
<br>

## routing module

routing이 포함된 모듈에 resolve를 추가하되 {리턴값 받을 변수명: 가드명} 을 기입합니다.<br>


```ts
import { MainGuard } from '../../guards/main.guard';

const routes: Routes = [
  { path: '', canActivateChild: [MainGuard], children: [
    { path: '', redirectTo: 'main', pathMatch: 'full' },
    { path: 'main', component: MainComponent, canActivate: [MainGuard], resolve: {itemList: MainGuard}, canDeactivate: [MainGuard] },
	{ path: 'login', loadChildren: '../login/login.module#LoginModule', canLoad: [MainGuard], data: {preload: true}}
  ]}
];

```
<br>

## guard

받아줄 guard를 작성합니다.<br>


```ts
import { Injectable } from '@angular/core';
import { ActivatedRouteSnapshot, RouterStateSnapshot, CanLoad, CanActivate, CanActivateChild, Resolve, CanDeactivate } from '@angular/router';
// service
import { ItemService } from '../services/item.service';
import { UserService } from '../services/user.service';

@Injectable({
  providedIn: 'root'
})
export class MainGuard implements CanLoad, CanActivate, CanActivateChild, Resolve<Item[]>, CanDeactivate<boolean> {

  /**
   * @param {ActivatedRoute} route
   * @param {Router} router
   * @memberof ListGuard
   */
  constructor(
	private itemService: ItemService,
	private userService: UserService
  ) { }

  canLoad(route: Route): Observable<boolean>|Promise<boolean>|boolean {
    // this.logHelper.set();
    this.itemService.onInit();
    return true;
  }

  async canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean|UrlTree>|Promise<boolean|UrlTree>|boolean|UrlTree {
    if (!this.userService.userInfo || this.userService.userInfo.isGuest) {
      this.router.navigate([`/login`], { relativeTo: this.route.root });
        return false;
    }
    return true;
  }

  canActivateChild( route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean|UrlTree>|Promise<boolean|UrlTree>|boolean|UrlTree {
    return true;
  }

  async resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<any>|Promise<any>|any {
		return await this.itemService.get();
  }

  canDeactivate() {
    this.itemService.clear();
    return true;
  }
}

```
<br>

## component 

이제 resolve의 결과를 받을 MainComponent를 작성합니다.<br>

```ts

import { Component, OnInit, ChangeDetectionStrategy, ChangeDetectorRef, NgZone } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';

@Component({
    selector: 'app-main',
    templateUrl: '../../templates/main.html',
    styleUrls: ['../../style/main.scss'],
    changeDetection: ChangeDetectionStrategy.OnPush
})

export class MainComponent implements OnInit {
  itemList: any[];

  constructor(
    private router: Router,
    private route: ActivatedRoute,
    private cd: ChangeDetectorRef,
    private zone: NgZone
  ) {

  }

  ngOnInit() {
    this.itemList = this.route.snapshot.data['itemList'];
  }

```