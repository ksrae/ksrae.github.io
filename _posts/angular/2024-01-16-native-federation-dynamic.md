---
title: "Angular 프로젝트에 Native Federation에서 동적으로 Micro Frontend 구축(Micro Frotend with Native Federation without Manifest)"
date: 2024-01-16 13:06:00 +0900
comments: true
categories: angular
tags: [nativefederation, dynamic]
---

이전 글에서는 Manifest를 사용하여 remote를 정의하는 정적인 방식으로 Micro Frontend를 구축하는 방법을 살펴봤습니다.<br/>
이번 글에서는 Native Federation에서 manifest를 정의하지 않고 동적으로 remote를 정의하여 Micro Frontend를 구축하는 방법을 예제를 통해 살펴보겠습니다.

## 개요
remote의 경우 이전 글에서 작성한 코드와 변화가 없으므로, 여기에서는 호스트 작성 방법에 중점을 둘 것입니다.

## Host(Shell) 프로젝트 설정 
### project 생성
먼저 호스트 프로젝트를 생성합니다. 이 예제에서는 Angular 17로 esbuild를 기반으로 하는 프로젝트를 생성하며, 프로젝트명은 `native-federation-shell` 로 정했습니다.
```ng new native-federation-shell```

### native federation 설치
```npm i @angular-architects/native-federation -D```

### native federation 설정
다양한 방식에 대응할 수 있도록 `type`을 `dynamic-host` 로 설정합니다.
```ng g @angular-architects/native-federation:init --type dynamic-host```

설정이 완료되면 `federation.config.js` 파일이 생성되고, `angular.json` 의 설정이 변경되어 있음을 확인할 수 있습니다.
<br/>
또한 `package.json` 의 `dependencies`에 `es-module-shims`가 추가되어 있는 것을 확인할 수 있습니다. (^1.5.12 or higher)


### main.ts 설정
`bootstrap.ts` 파일을 호출하기 전 `initFederation` 을 호출해야 합니다.<br/> 
`initFederation`를 정의할 때 `manifest`를 정의하지 않으므로 remote와 같이 파라미터 없이 사용합니다.
```ts
import { initFederation } from '@angular-architects/native-federation';

initFederation()
  .catch(err => console.error(err))
  .then(_ => import('./bootstrap'))
  .catch(err => console.error(err));
```


### loadRemoteModule 정의
component에서 동적으로 remote의 component를 호출하는 코드를 작성합니다.<br/> 
router에서 remote의 component를 호출할 때와 동일한 loadRemoteModule 함수를 사용하지만, 파라미터가 다르므로 주의가 필요합니다.

``` ts
function loadRemoteModule<T = any>(options: LoadRemoteModuleOptions): Promise<T>;

export type LoadRemoteModuleOptions = {
    remoteEntry?: string; // url
    remoteName?: string; // 반드시 remote의 federation.config.js에서 정의한 name과 일치해야 함.
    exposedModule: string; // export된 component의  key 값
};
```

### remote에 federation.config.js 정의

위의 타입을 기반으로 remote의 component를 호출하는 코드를 작성합니다. <br/>
Remote의 URL은 http://localhost:4202이며, 아래와 같이 federation.config.js을 설정했습니다.

```js
const { withNativeFederation, shareAll } = require('@angular-architects/native-federation/config');

module.exports = withNativeFederation({
  // host에서 remoteName을 정의할 때 이 값과 일치해야 함.
  name: 'native-federation-remote',

  // host에서 exposedModule을 설정할 때 exposes에 정의된 키 값에 포함된 값이어야 함.
  exposes: {
    './DynamicComponent': './src/app/dynamic/dynamic.component.ts',
  },

  shared: {
    ...shareAll({ singleton: true, strictVersion: true, requiredVersion: 'auto' }),
  },

  skip: [
    'rxjs/ajax',
    'rxjs/fetch',
    'rxjs/testing',
    'rxjs/webSocket',
    // Add further packages you don't need at runtime
  ],


});

```

### router-outlet을 대신할 dom 정의

component에서 route가 아닌 방식으로 remote의 component를 호출할 때, router-outlet을 사용할 수 없다는 점을 고려해야 합니다. 따라서 router-outlet을 대신할 DOM을 정의해야 합니다. <br/>
또한, 정의한 DOM에 remote의 component를 추가하려면 createComponent() 함수가 필요하며, 이 함수는 ViewContainerRef에서 가져올 수 있습니다. <br/>
ViewContainerRef를 가져오려면 DOM을 ViewChild로 가져와 ViewContainerRef로 정의해야 합니다. 

```ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [],
  templateUrl: '<div #remote></div>',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class AppComponent {
  title = 'native-federation-shell';
  cdr = inject(ChangeDetectorRef);

	// Remote 컴포넌트를 가져올 위치이므로 ViewChild로 DOM 정보를 가져옵니다. 반드시 read, static 설정이 필요합니다.
  @ViewChild('remote', {read: ViewContainerRef, static: true}) viewContainer!: ViewContainerRef;

  ...
}
```

### remote component 호출

마지막으로 `loadRemoteModule()` 로 remote 컴포넌트를 가져온 후, `createComponent()` 로 정의한 DOM에 추가합니다. <br/>
만약 컴포넌트의 changeDetection이 OnPush로 설정되어 있다면, 반드시 수동으로 ChangeDetectorRef를 정의해야 합니다



```ts
import { ChangeDetectionStrategy, ChangeDetectorRef, Component, ViewChild, ViewContainerRef, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { loadRemoteModule } from '@angular-architects/native-federation';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule],
  templateUrl: '<div #remote></div>',
  styleUrl: './app.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class AppComponent {
  title = 'native-federation-shell';
  cdr = inject(ChangeDetectorRef);

  @ViewChild('remote', {read: ViewContainerRef, static: true}) viewContainer!: ViewContainerRef;

  async openDynamic(e: any) {
    const remoteComponent = await loadRemoteModule({
			// manifest가 없으면 반드시 필요함. module federation에서는 remoteEntry.js였으나 .json으로 변경됨.
      remoteEntry: 'http://localhost:4202/remoteEntry.json', 
      exposedModule: './DynamicComponent',
      remoteName: 'native-federation-remote',
    }).then(m => m.DynamicComponent); // remote에 정의된 노출할 component

    const ref = this.viewContainer.createComponent(remoteComponent);
    this.cdr.markForCheck(); // changeDetection이 OnPush이면 정의해야 함.
  }
}
```
