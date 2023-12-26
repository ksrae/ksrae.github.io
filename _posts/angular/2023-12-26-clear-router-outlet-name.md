---
title: "outlet name attribute으로 생성한 route 제거하기 - How to clear route made with outlet name attribute"
date: 2023-12-26 20:03:00 +0900
comments: true
categories: angular
tags: [routing]
---

Angular 애플리케이션에서 Route Outlet Name을 활용하여 동적으로 삽입된 컴포넌트에 대한 Route를 관리하는 것은 효율적인 UI 구성을 위한 강력한 수단입니다. <br/>그러나 동시에 현재 Route를 Outlet에서 제거하는 것은 일반적인 Route 제거 방법과는 다른 접근이 필요합니다. <br/>이 글에서는 Outlet의 Name attribute로 생성된 Route를 제거하는 세 가지 방법에 대해 논의하고자 합니다.



## Outlet의 Name Attribute 값 null로 설정하기
Outlet의 Name attribute 값을 null로 설정하는 방법은 간단하면서도 직관적입니다. <br/>현재 Route를 Outlet에서 제거하기 위해 브라우저의 주소 표시줄에 새로운 Route를 생성하지 않고도 Outlet의 Name attribute 값을 null로 설정함으로써 현재 Route를 간단히 제거할 수 있습니다.

```ts
// app-popup.component.ts

@Component({
  selector: 'app-popup',
  standalone: true,
  imports: [RouterModule],
  template: `
    <dialog open #modal>
      <h2>Popup</h2>
      <p>This is a popup content.</p>
      <a [routerLink]="['/', {outlets: {popupType: null}}]">Close</a>
    </dialog>
  `,
  styleUrls: ['./popup.component.css'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class PopupComponent {}

```


## Route Navigate로 제거하기
Router.navigate를 활용하여 현재 Route를 Outlet에서 제거하는 방법은 다소 복잡할 수 있지만, 더 세밀한 제어를 가능하게 합니다. <br/>이 방법을 통해 현재 Route를 특정 시점에 제거할 수 있습니다.


```ts
// app-popup.component.ts

@Component({
  selector: 'app-popup',
  standalone: true,
  imports: [RouterModule],
  template: `
    <dialog open #modal>
      <h2>Popup</h2>
      <p>This is a popup content.</p>
      <button (click)="close()">Close</button>
    </dialog>
  `,
  styleUrls: ['./popup.component.css'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class PopupComponent {
  router = inject(Router);
  route = inject(ActivatedRoute);

  close() {
    this.router.navigate(['/', { outlets: { popupType: null } }], {
      relativeTo: this.route.parent,
    });
  }
}

```



## Location으로 제거하기
Location.back() 메서드를 활용하여 현재 Route를 제거하는 방법은 브라우저의 뒤로가기 동작과 유사한 방식으로 동작합니다.<br/> 이를 통해 브라우저의 기본 네비게이션 동작과 일관성을 유지할 수 있습니다.

```ts
// app-popup.component.ts

@Component({
  selector: 'app-popup',
  standalone: true,
  imports: [RouterModule],
  template: `
    <dialog open #modal>
      <h2>Popup</h2>
      <p>This is a popup content.</p>
      <button (click)="back()">Back</button>
    </dialog>
  `,
  styleUrls: ['./popup.component.css'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class PopupComponent {
  location = inject(Location); // @angular/common

  back() {
    this.location.back();
  }
}

```

## 결론
Angular 애플리케이션에서 Route Outlet Name을 활용한 동적 컴포넌트 관리는 풍부한 사용성을 제공하지만, 현재 Route를 Outlet에서 효과적으로 제거하는 것은 일반적인 Route 제거와는 다른 고려 사항을 필요로 합니다. <br/>
상황에 적절한 방법을 선택하여 적용함으로써 안정적이고 효율적인 UI 관리를 실현해 봅시다.



## 전체 샘플

먼저 route를 호출하는 component를 작성합니다.

```ts
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterModule],
  templateUrl: `
  <a [routerLink]="['/', {outlets: {popupType: ['popup']}}]" >Open Popup</a>
  <router-outlet name="popupType"></router-outlet>
  `,
  styleUrl: './app.component.scss'
})
export class AppComponent {
  title = 'model-router-name-control';

}
```


이 route를 통해 노출할 popup을 작성합니다.

```ts
@Component({
  selector: 'app-popup',
  standalone: true,
  imports: [
    RouterModule
  ],
  template: `
  <dialog open #modal>
    <h2>Popup</h2>
    <p>This is a popup content.</p>

    <a [routerLink]="'/', {outlets: {popupType: null}}]">Close</a>
    <button (click)="back($event)">Back</button>
    <button (click)="close($event)">Close</button>
  </dialog>
  `,
  styleUrl: './popup.component.css',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class PopupComponent {
  loc = inject(Location); // @angular/common
  route = inject(Router);
  router = inject(ActivatedRoute);

  back(e: any) {
    this.loc.back();
  }
  close(e: any) {    
    this.route.navigate(['/', { outlets: { popupType: null } }], {
      relativeTo: this.router.parent
    });
  }
}
```