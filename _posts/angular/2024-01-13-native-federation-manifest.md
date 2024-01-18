---
title: "Angular 프로젝트에 Native Federation의 manifest를 활용한 Micro Frontend 구축(Micro Frotend with Native Federation by Manifest)"
date: 2024-01-13 14:11:00 +0900
comments: true
categories: angular
tags: [nativefederation, manifest]
---

이 글에서는 Native Federation을 활용하여 Micro Frontend를 구축하는 과정을 예제를 통해 살펴보겠습니다.

## Native Federation 정의
Native Federation은 module federation을 esbuild 등 다양한 번들로 Micro Frontend 환경을 구축할 수 있도록 지원하는 라이브러리입니다.<br/>
<br/>
Angular에서는 Micro Frontend 환경을 구축하기 위해 module federation을 사용합니다. [관련 링크](https://www.npmjs.com/package/@angular-architects/module-federation).<br/>
현재 module federation은 Angular 12에서 16까지 지원되며, 그 이상의 버전에서도 webpack 환경인 경우 사용 가능합니다.<br/>
그런데 만일 Angular 16 이상을 사용하는 프로젝트에서 esbuild 번들러를 사용한다면 module federation의 활용이 어려울 수 있으므로, native Federation를 활용하여 micro frotend 환경을 구축하는 것을 추천합니다.[관련 링크](https://www.npmjs.com/package/@angular-architects/native-federation)<br/>
<br/>
<br/>


## 개요
Micro Frontend는 기본적으로 두 개 이상의 프로젝트를 연결하는 개념입니다. <br/>
따라서 이 글에서는 두 개의 프로젝트를 생성하고, 하나는 호스트로, 다른 하나는 원격 프로젝트로 구성하여 호스트가 원격 컴포넌트를 가져오는 코드를 작성할 것입니다.

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


### manifest 설정
assets에 federation.manifest.json 파일을 생성합니다.<br/>
여기서 key는 원격 프로젝트명이고, value는 원격 프로젝트의 URL을 입력합니다.

```json
{
	"native-federation-remote-manifest": "http://localhost:4201/remoteEntry.json"
}
```

### main.ts 설정
`bootstrap.ts`` 파일을 호출하기 전 `initFederation`을 호출해야 합니다.<br/> 
또한 `initFederation`의 파라미터로 `manifest` 파일을 연결합니다.
```ts
import { initFederation } from '@angular-architects/native-federation';

initFederation('/assets/federation.manifest.json')
  .catch(err => console.error(err))
  .then(_ => import('./bootstrap'))
  .catch(err => console.error(err));
```


### route 에서 remote component 호출
이 글에서는 라우트에서 원격 프로젝트의 컴포넌트를 호출합니다. 이때 반드시 원격의 federation.config.js에 해당 컴포넌트가 노출되었는지 확인해야 합니다.<br/>
<br/>

이 예제에서는 두 개의 원격 컴포넌트를 연결합니다.
```ts
// app.routes.ts
import { loadRemoteModule } from '@angular-architects/native-federation';
import { Routes } from '@angular/router';

export const routes: Routes = [
  {path: 'remote-app', loadComponent: () =>
    loadRemoteModule('native-federation-remote-manifest', './Component').then((m) => m.AppComponent),
  },
  {path: 'remote-sample', loadComponent: () =>
    loadRemoteModule('native-federation-remote-manifest', './SampleComponent').then((m) => m.SampleComponent),
  }
];
```

#### route에서 remote route 호출
remote의 route를 호출할 수도 있습니다. 이 때는 loadComponent가 아닌 loadChildren을 사용해야 합니다.
```ts
{
    path: 'remote-routes',
    // loadChildreas instead of loadComponent !!!
    loadChildren: () =>
      loadRemoteModule('native-federation-remote-manifest', './routes').then((m) => m.APP_ROUTES),
  },
```

### template 작성
이제 a 태그로 해당 route를 호출합니다.
```html
<a [routerLink]="['/remote-app']" routerLinkActive="router-link-active" >REMOTEAPP</a>
<br/>
<br/>
<a [routerLink]="['/remote-sample']" routerLinkActive="router-link-active" >REMOTESAMPLE</a>
<br/>
<br/>

<router-outlet></router-outlet>
```

여기까지 host 설정이 완료되었습니다.



## Remote 프로젝트 설정
이제 remote 프로젝트를 생성하고 host와 연결합니다.

### project 생성
```ng new native-federation-remote-manifest```

### native federation 설치
```npm i @angular-architects/native-federation -D```

### native federation 설정
host와는 달리 `type`을 `remote`로 설정합니다.
```ng g @angular-architects/native-federation:init --type remote```

설정이 완료되면 host와 마찬가지로 `federation.config.js` `angular.json` `package.json` 이 추가 또는 변경됩니다.

### federation.config.js 수정
호스트의 설정과 유사하지만 원격에서는 호스트에 노출할 컴포넌트를 작성해야 합니다.<br/>
여기에서는 호스트의 라우트에서 정의한 두 개의 컴포넌트를 정의합니다. 또한 호스트 manifest와 동일한 name을 작성합니다.


```js
const { withNativeFederation, shareAll } = require('@angular-architects/native-federation/config');

module.exports = withNativeFederation({

  name: 'native-federation-remote-manifest',

  exposes: {
    './Component': './src/app/app.component.ts',
    './SampleComponent': './src/app/sample/sample.component.ts',
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
  ]
  
});
```

### main.ts 작성
remote에는 manifest가 없으므로 파라미터가 없는 initFederation을 정의합니다.
```ts
import { initFederation } from '@angular-architects/native-federation';

initFederation()
  .catch(err => console.error(err))
  .then(_ => import('./bootstrap'))
  .catch(err => console.error(err));
```


### sample.component.ts 작성
노출할 component를 임의로 작성합니다.


### .ts 에러인 경우
만일 appComponent가 아닌 다른 component를 노출하려고 할 때 ts 파일 에러가 발생하는 경우 tsconfig.app.json에 ts 파일을 include 합니다.
```json
{
  ...
  "include": [
    "src/**/*.d.ts",
    "src/**/*.ts",
  ]
}
```

## 실행
host와 remote 두 프로젝트 모두를 실행하고, host에서 링크를 클릭하여 remote component가 제대로 호출되는지 확인 합니다.


## 결론
두 개의 독립된 프로젝트가 마치 하나의 프로젝트인 것처럼 host에서 remote의 component를 사용하는 micro frontend방식에 대해서 native-federation으로 구축해 보았습니다.<br/>
이를 통해 하나의 프로젝트를 다른 여러 프로젝트를 통해 구축할 수 있다는 것을 확인할 수 있었고 또한 native federation을 통해 webpack이 아닌 esbuild 번들러를 사용하는 프로젝트에서도 micro frontend 환경을 구축할 수 있다는 것을 확인할 수 있습니다.<br/>
<br/>
module federation도 일부 설정의 차이만 있을 뿐 방식은 유사하므로 참고하시기 바랍니다.<br/>
아래 참고 사이트를 통해 더 자세하고 정확한 정보를 확인하십시오.


## 참고 사이트
[micro-frontends-with-modern-angular-part-1-standalone-and-esbuild](https://www.angulararchitects.io/blog/micro-frontends-with-modern-angular-part-1-standalone-and-esbuild/)
