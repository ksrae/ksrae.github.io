---
title: "how to use name attribute of router-outlet"
date: 2022-12-26 13:29:00 +0900
comments: true
categories: angular
tags: [routing, routeroutlet]
---

[한국어(Korean) Page](https://velog.io/@ksrae/router-outlet%EC%97%90%EC%84%9C-name-attr-%EC%82%AC%EC%9A%A9-%EB%B0%A9%EB%B2%95)
<br/>

> In this session, we will explore the usage of the `router-outlet` tag's `name` attribute within the Angular framework.


# Understanding the `name` Attribute

The `name` attribute is utilized within the `router-outlet` tag as follows:

```html
<router-outlet name="route1"></router-outlet>
```

This `name` attribute is intrinsically linked to the `outlet` configuration within your application's `Routes`.

## Deep Dive into Outlets

When configuring routing, we leverage the `Routes` array from `@angular/router`. We can assign an `outlet` to each path, which may not always be utilized in simple routing scenarios, but offers significant flexibility.

```tsx
const routes: Routes = [
  {path: 'path1', component: Route1Component, outlet: 'route1' }
]
```

## Implementing Navigation

Once an `outlet` is defined, we can establish navigation within our templates or components. The structure of the `outlets` object adheres to the following pattern:

`Outlet Defined in Routes : Path Assigned to the Outlet`

### Navigation within Templates

To implement navigation in a template, use the following approach:

```html
<a [routerLink]="[{outlets:{route1:['path1']}}]">Go to Path1</a>
```

### Navigation within Components

Alternatively, navigation can be initiated from within a component:

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
    ]);
  }
}
```

## Examining the Resultant URL

Upon execution and navigation to the target page, you will observe a distinct URL structure in the browser:

```
http://localhost:4200/(route1:path1)
```

This occurs because, unlike standard routes that rely on the `path` defined in `Routes`, the outlet now acts as the key, and the path becomes the outlet's value. This architecture enables the versatile expression of diverse paths, guided by the `<router-outlet>`'s `name` attribute configuration.

## Configuring Child Outlets

Let's investigate the setup of child component structures using outlets. The structure of the `Routes` array within `app-routing.module.ts` might resemble the following:

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

(The examples below demonstrate scenarios where the outlet is the same but the path differs. This approach is taken because when outlets differ, the code will certainly work.)

In `app.component.html`, since `Route1Component` will be invoked, we include the `<router-outlet>`. To streamline navigation, we can also use `<a routerLink>` elements:

```html
  <a [routerLink]="['./path1', {outlets:{route1:['path-first']}}]">First</a>
  <br/>
  <a [routerLink]="['./path1', {outlets:{route1:['path-second']}}]">Second</a>
  <br/>
  <router-outlet></router-outlet>
```

Within `route1.component.ts`, as the child component structure is outlet-based, the `name` attribute must be configured on the `<router-outlet>`:

```html
  <router-outlet name="route1"></router-outlet>
```

Running this setup will yield the following URLs, respectively:

```
http://localhost:4200/path1/(route1:path-first)
http://localhost:4200/path1/(route1:path-second)
```

This demonstrates that diverse outlets can be configured even for child paths that are not at the root level. Furthermore, it confirms the seamless operation of incorporating multiple paths within a single outlet. Leveraging this capability allows for expanding the application of routes beyond simple page navigation, potentially repurposing them for module reusability.

# References

- [Angular Named Router Outlet + Popup Example](https://www.concretepage.com/angular-2/angular-2-4-named-router-outlet-popup-example)
- [The Angular 10/9 Router-Outlets: Named and Multiple Outlets (Auxiliary Routes) Example](https://www.techiediaries.com/angular-router-multiple-outlets/)
- [Multiple named router-outlet angular 2](https://stackoverflow.com/questions/38038001/multiple-named-router-outlet-angular-2)