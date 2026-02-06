---
title: "Routing Resolve More Detail"
date: 2019-07-08 11:00:00 +0900
comments: true
categories: angular
tags: [routing, resolve, component]
---

When a component attempts to apply a value fetched from a service within its `onInit` lifecycle hook to its template, an error can arise if the template tries to access the variable before the value has been assigned. This is a common issue in asynchronous data binding. A simple workaround involves using the safe navigation operator (`?.`) to allow for nullable values, preventing errors when the template initially renders.

```html
<div> {{item?.key}} </div>
<div> {{item?.value}} </div>
```

However, a more robust solution is to ensure that the component receives the necessary data *before* it is fully initialized and rendered. This can be achieved by leveraging the `resolve` feature within Angular's routing mechanism. Resolvers pre-fetch data before a route is activated, ensuring that the component has the required data available upon initialization.

```tsx
// resolve.service.ts
import { Injectable } from '@angular/core';
import { Resolve, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import {Observable} from 'rxjs/Observable';
import 'rxjs/add/observable/of';

@Injectable()
export class PageResolve implements Resolve<Observable<any>> {
  resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot) {
    return Observable.of(null);
  }
}
```

As demonstrated above, a resolver must return one of three types: `Observable<any>`, `Promise<any>`, or `any`. The returned value is then attached to the route's `data` property as a JSON object. The component can then retrieve this value using a specified key.

To configure the resolver, you need to define it within your route configuration:

```tsx
// resolve.route.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { PageComponent } from './page.component';
import { PageResolve } from './page.resolve';

// lazy loading
const routes: Routes = [
  { path: 'page', resolve: {page: PageResolve}, component: PageComponent }
];

@NgModule({
    imports: [ RouterModule.forRoot(routes)],
    exports: [ RouterModule ]
  })

  export class PageRoutingModule { }
```

With the resolver configured, the `PageComponent` can now retrieve the resolved data within its `OnInit` lifecycle hook:

```tsx
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-page',
  template: `<div>  {{ page }} </div>`
})

export class PageComponent implements OnInit {
  page: string;

  constructor(private route: ActivatedRoute) { }

  ngOnInit() {
    this.page = this.route.snapshot.data['page'];
  }
}
```

In summary, using route resolvers provides a clean and efficient way to ensure that components have the necessary data before they are initialized, preventing potential errors related to asynchronous data loading. This approach enhances the robustness and user experience of your application.

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/%EB%8D%94-%EC%83%81%EC%84%B8%ED%95%98%EA%B2%8C-%EC%A0%95%EB%A6%AC%ED%95%9C-Routing-Resolve)
<br/>