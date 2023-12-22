---
title: "Angular Route를 활용한 Modal 관리(Manage Modal with Angular Route)"
date: 2023-12-22 14:36:00 +0900
comments: true
categories: angular
tags: [routing]
---

Angular에서 Modal을 열고 닫는 방법은 다양하게 존재합니다. 그 중에서도 Route를 활용하여 Modal을 제어하는 방법을 살펴보겠습니다. 이 방식은 특정 주소로 이동할 때 Modal이 열리고, 해당 주소에서 벗어나면 Modal이 닫히는 간편하면서도 효과적인 방법입니다.

## 1. 프로젝트 설정
먼저, Angular 프로젝트에서 Route를 사용할 수 있도록 설정합니다. 만약 모듈 방식으로 작업 중이라면 RouterModule을 import하고, standalone인 경우 provideRouter()를 정의합니다. 이 예제에서는 standalone을 기준으로 작성하였습니다.

```ts
import { Routes } from '@angular/router';

export const appRoutes: Routes = [
  { path: '', children: [
    {path: 'modal', loadChildren: () => import('./modal/modal.routes').then(m => m.modalRoutes)},
  ]}
];

```


## 2. Template에 Route 설정
html에 모달을 열고 닫을 수 있는 코드를 추가합니다.<br/>
Route를 통해 모달을 호출할 예정이므로 a tag에 routetLink를 설정합니다.<br/>
모달이 노출될 위치에 {% raw %}<router-outlet>{% endraw %} 을 정의 합니다.

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


## 3. Modal Routes 정의
Modal을 여러 개 사용할 수 있도록 각각의 Modal에 대한 Routes를 정의합니다. modal.routes.ts 파일을 생성하고 다음과 같이 작성합니다.<br/>
여기에서는 helloModal과 sampleModal 을 추가해보았습니다.

```ts
import { Routes } from '@angular/router';
import { HelloModalComponent } from './hellomodal/hellomodal.component';
import { SamplemodalComponent } from './samplemodal/samplemodal.component';

export const modalRoutes: Routes = [
  {path: 'hellomodal', component: HelloModalComponent},
  {path: 'samplemodal', component: SamplemodalComponent}
];

```


## 4. Modal Components 작성
Modal을 구성하는 각각의 Component를 작성합니다. 간단한 예시로 HelloModalComponent를 작성해보겠습니다.

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



## 5. standalone component imports issue
standalone인 경우 routerLink를 사용하려면 반드시 component에 RouterModule을 import 해주어야 합니다.
그렇지 않으면 template에서 routerLink에 에러가 발생하므로 주의해야 합니다.
```ts
import { RouterModule } from '@angular/router';
@Component({
  ...
  standalone: true,
  imports: [RouterModule],
  ...
})
```


## 6. 기타
프로젝트에서 Router를 사용 중인데 모달을 Router로 관리하고 싶다면, router-outlet의 name을 활용할 수 있습니다. 
자세한 내용은 [router-outlet에서 name attr 사용 방법 - how to use name attribute of router-outlet](https://ksrae.github.io//angular/router-outlet-name/) 을 참고하세요.


## 결론
Angular Route를 통해 Modal을 관리함으로써 모듈화되고 구조적으로 깔끔한 코드를 작성할 수 있습니다.<br/>
<br/>
이상으로 Angular Route를 활용하여 Modal을 제어하는 방법에 대한 글을 마칩니다. 향후 프로젝트에서 Modal 관리에 어려움을 겪을 때 이 예시를 참고하여 해결할 수 있기를 바랍니다.