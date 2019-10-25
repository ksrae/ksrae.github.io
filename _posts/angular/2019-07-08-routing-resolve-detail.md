---
title: "Routing Resolve More Detail"
date: 2019-07-08 11:00:00 +0900
comments: true
categories: angular
tags: [routing, resolve, component]
---



component가 onInit 에서 service에서 가져온 값을 template에 적용할 때 template에서 에러가 발생합니다. <br>
이는 변수에 값이 할당되기 전에 template에서 변수를 읽으려고 시도하기 때문인데 <br>이를 해결하려면 nullable을 허용하는 ? 를 넣어 해결하는 방법을 사용할 수 있습니다.<br>

```html
{% raw %}
<div> {{item?.key}} </div>
<div> {{item?value}} </div>
{% endraw %}
```

보다 더 확실하게 처리하려면 해당 component가 로딩되기 전에 값을 할당하면 되는데 이는 routing의 resolve를 활용하여 처리할 수 있습니다.


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

위에서 보다시피 resolve는 Observable<any>|Promise<any>|any 세 가지 중 하나를 리턴해야 합니다.<br>
이 리턴 값을 route에 json 형태로 설정하며, 설정된 key값을 사용해서 component가 값을 가져가게 됩니다.<br>


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
  {% raw %}
  template: `<div>  {{ page }} </div>`
  {% endraw %}
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