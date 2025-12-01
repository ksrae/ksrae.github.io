---
title: "Manage Modal with Angular Route"
date: 2023-12-22 14:36:00 +0900
comments: true
categories: angular
tags: [routing]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Angular-Route%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Modal-%EA%B4%80%EB%A6%AC)
<br/>

Opening and closing modals in Angular can be achieved through various methods. This article explores controlling modals using routes, a straightforward and effective approach where a modal opens upon navigating to a specific address and closes when navigating away.

## 1. Project Setup

First, configure your Angular project to use routes. If you're working with modules, import `RouterModule`; for standalone components, define `provideRouter()`. This example uses the standalone approach.

```tsx
import { Routes } from '@angular/router';

export const appRoutes: Routes = [
  { path: '', children: [
    {path: 'modal', loadChildren: () => import('./modal/modal.routes').then(m => m.modalRoutes)},
  ]}
];
```

## 2. Template Route Configuration

Add the code to open and close the modal to your HTML template.

Since we'll be calling the modal through a route, set `routerLink` on an `<a>` tag.  Define a `<router-outlet>` where the modal will be displayed.

```html
<!-- app.component.html -->
<button>
  <a [routerLink]="['/modal/hellomodal']"> Open Hello Modal </a>
</button>
<br/>
<button>
  <a [routerLink]="['/modal/samplemodal']"> Open Sample Modal </a>
</button>
<router-outlet></router-outlet>
```

## 3. Define Modal Routes

Define routes for each modal if you're using multiple modals. Create a `modal.routes.ts` file and write the following.

Here, we've added `helloModal` and `sampleModal`.

```tsx
import { Routes } from '@angular/router';
import { HelloModalComponent } from './hellomodal/hellomodal.component';
import { SamplemodalComponent } from './samplemodal/samplemodal.component';

export const modalRoutes: Routes = [
  {path: 'hellomodal', component: HelloModalComponent},
  {path: 'samplemodal', component: SamplemodalComponent}
];
```

## 4. Write Modal Components

Write the individual components that make up the modal. As a simple example, let's write `HelloModalComponent`.

```html
<!-- hellomodal.component.html -->
<dialog open>
  <h2>Hello World Modal</h2>
  <p>This is a simple modal content.</p>

  <button>
    <a [routerLink]="['/']" routerLinkActive="active"> OK </a>
  </button>
</dialog>
```

## 5. Standalone Component Imports Issue

When using standalone components, you must import `RouterModule` into the component to use `routerLink`.

Otherwise, an error will occur on `routerLink` in the template, so be careful.

```tsx
import { RouterModule } from '@angular/router';
@Component({
  ...
  standalone: true,
  imports: [RouterModule],
  ...
})
```

## 6. Miscellaneous

If you're already using Router in your project and want to manage modals with Router, you can use the `name` attribute of `router-outlet`.

For details, refer to [How to Use the Name Attribute of router-outlet](https://ksrae.github.io//angular/router-outlet-name/).

## Conclusion

Managing modals through Angular routes allows you to write modular and structurally clean code.

This concludes the article on how to control modals using Angular routes. I hope this example helps you solve any difficulties in managing modals in future projects.