---
title: "Routing Resolve More Detail"
date: 2019-07-08 11:00:00 +0900
comments: true
categories: angular
tags: [routing, resolve]
---



component가 onInit 에서 service에서 가져온 값을 template에 적용할 때 template에서 에러가 발생한다.
따라서 아래와 같이 ? 를 넣어주어 해결하는 방법을 사용한다.

```html
<div> \{{item?.key}} </div>
<div> ]{{item?value}} </div>
```

만일 routing을 통해 해당 페이지에 접근하는 경우 이를 보다 효율적으로 처리할 수 있는데 Resolve를 활용할 수 있다.

Resolve는 Guard와 유사하게 사용하는데 기본적으로는 아래와 같이 사용한다.

```ts
// resolve.service.ts
import { Injectable } from '@angular/core';
import { Resolve, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import {Observable} from 'rxjs/Observable';
import 'rxjs/add/observable/of';

export PageResolve implments Resolve<Observable<any>> {
  resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot) {
    return Observable.of(null);
  }
}
```

위에서 보다시피 resolve는 반드시 Observable<T> 를 리턴해야 한다.

이를 route에 적용한다. 이 때 json 형태로 key를 설정해야 하며, 이 key값을 사용해서 component가 값을 가져가게 된다.

```ts
// resolve.route.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { PageComponent } from './page.component';
import { PageResolve } from './page/PageResolve';

// lazy loading
const routes: Routes = [
  { path: 'page', resolve: {page: PageResolve}, component: PageComponent }
];

@NgModule({
    imports: [ RouterModule.forRoot(routes)],
    exports: [ RouterModule ]
  })

  export class PageRouting { }
```

이제 PageComponent의 OnInit에서 resolve값을 가져온다.

```ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-page',
  template: `<div> \{{ page }} </div>`
})

export class PageComponent implements OnInit {
  page: string;

  constructor(private route: ActivatedRoute) {

  }

  ngOnInit() {
    this.page = this.route.snapshot.data['page'];
  }
}
```

끝.