---
title: "router-outlet에서 name attr 사용 방법 - how to use name attribute of router-outlet"
date: 2022-12-26 13:29:00 +0900
comments: true
categories: angular
tags: [routing, routeroutlet]
---

> 이번 시간은 router-outlet 태그의 name attribute의 활용에 대해 알아보겠습니다.

<br/>

# name attribute
name attribute는 다음과 같이 사용합니다.

```html
<router-outlet name="route1"></router-outlet>
```

이 name attribute는 Routes의 outlet 설정과 관련이 있습니다.


## outlet

routing을 구성할 때, '@angular/router'의 Routes를 활용하는데 이 때 우리는 (일반적인 routing을 구성할 때는 사용하지 않았지만) 각 path에 outlet을 적용할 수 있습니다.

```ts
const routes: Routes = [
  {path: 'path1', component: Route1Component, outlet: 'route1' }
]
```

## navigation

위와 같이 outlet을 설정하였으면 이제 template이나 또는 component에서 navigation을 설정합니다.<br/>
<br/>
이 때, "outlets" object의 구성은 다음과 같습니다.<br/>
`Routes 에서 설정한 outlet : outlet이 설정된 object의 path 명`


### Template

template에서 navigation 하는 방법은 다음과 같습니다.

```html
<a [routerLink]="[{outlets:{route1:['path1']}}]">go to path1</a>
```

### Component

component에서 navigation 하는 방법은 다음과 같습니다.

```tsx
import { Router } from '@angular/router';
...
export class AppComponent {
  constructor(
    private router: Router
  ) {}
  move() {
    this.router.navigate([
      {outlets: {route1: 'path1'}}
    ])
  }
}
```

## Url

실행하고 해당 페이지로 이동하면, 브라우저에서 기존과 다른 url 구성을 확인할 수 있습니다.

```
http://localhost:4200/(route1:path1)
```

이는 기본 route는 Routes에 설정된 path를 보고 구성하였으나, 이제 outlet이 key가 되고 path가 outlet의 value가 되는 구성으로 이루어지기 때문입니다.<br/>
따라서, 다양한 path를 `<router-outlet>` 의 name attribute 구성에 따라 다양하게 표현할 수 있습니다.


## child outlet
child component 구조를 outlet으로 설정할 경우에 대해 알아봅시다.<br/>
<br/>
app-routing.module.ts의 Routes 의 구조는 다음과 같습니다.

```tsx
const routes: Routes = [
  { path: '', children: [
    {path: 'path1', component: Route1Component, children: [
      {path: 'path-first', component: Path1Component, outlet: 'route1'},
      {path: 'path-second', component: Path2Component, outlet: 'route1'},
    ]},
  ]},
];
```
(outlet이 다르면 확실히 동작하므로 약간 응용하여 outlet이 같고, path가 다른 경우의 예를 알아봅시다.)<br/>
<br/>
app.component.html은 Route1Component를 호출하면 되므로 `<router-outlet>` 을 호출합니다.<br/>
쉽게 이동할 수 있도록 `<a routerLink>` 를 함께 활용해 봅시다.

```html
  <a [routerLink]="['./path1', {outlets:{route1:['path-first']}}]">first</a>
  <br/>
  <a [routerLink]="['./path1', {outlets:{route1:['path-second']}}]">second</a>
  <br/>
  <router-outlet></router-outlet>
 ```

route1.component.ts 는 child component의 구조가 outlet 형태이므로 `<router-outlet>` 에 name attribute를 설정해야 합니다.

```html
  <router-outlet name="route1"></router-outlet>
 ```

실행해보면 각각 다음과 같은 URL을 갖는 것을 확인할 수 있습니다.

```
http://localhost:4200/path1/(route1:path-first)
http://localhost:4200/path1/(route1:path-second)
```

즉, root가 아닌 하위 path 에도 다양한 outlet을 구성할 수 있다는 것을 확인할 수 있습니다.<br/>
또한, 하나의 outlet에 여러 path를 넣어도 잘 동작함을 확인할 수 있습니다.<br/>
이를 활용하여 단순히 page 이동으로서의 route가 아닌 module의 재활용을 위한 용도로 확장하여 사용해도 좋을 것 같습니다.<br/>
<br/>



# Reference

[Angular Named Router Outlet + Popup Example](https://www.concretepage.com/angular-2/angular-2-4-named-router-outlet-popup-example)

[The Angular 10/9 Router-Outlets: Named and Multiple Outlets (Auxiliary Routes) Example](https://www.techiediaries.com/angular-router-multiple-outlets/)

[Multiple named router-outlet angular 2](https://stackoverflow.com/questions/38038001/multiple-named-router-outlet-angular-2)
