---
title: "자체 라이브러리 만들기 (Creating Library)"
date: 2019-07-15 18:05:00 +0900
comments: true
categories: angular
tags: [library, package]
---


자체 개발한 라이브러리를 마치 외부 라이브러리를 사용하는 것처럼 만들 수 있습니다.<br><br>

Angular6 부터 ng-packagr가 정식 지원 되어 쉽게 만을 수 있으니 <br>
자주 사용하는 기능은 라이브러리로 만들어두고 package화 해서 편리하게 재활용 하시기 바랍니다.<br>

## 생성

cli를 활용하여 library를 생성합니다.<br>
유의할 점은 angular 프로젝트 내에서 진행해야 한다는 점입니다.<br>
즉, ng new 로 프로젝트를 생성한 뒤 아래의 command를 입력하여 library를 생성해야 함을 유의 하시기 바랍니다.<br>

```bash
ng generate library [library-name]
```

그러면 angular root 폴더에 projects 폴더 안에 library-name으로 폴더가 생성되어 있습니다.


## 분석

각 파일을 열어 생성된 라이브러리가 올바른지 확인합니다.

### angular.json
생성한 프로젝트 명으로 검색해보면 설정이 생성되어 있습니다.<br>
여기에서 projectType이 "library" 인지 확인하여야 합니다.<br>

```json
"my-lib": {
  "projectType": "library",
  "root": "projects/my-lib",
  "sourceRoot": "projects/my-lib/src",
  "prefix": "lib",
  "architect": {
    "build": {
      "builder": "@angular-devkit/build-ng-packagr:build",
      "options": {
        "tsConfig": "projects/my-lib/tsconfig.lib.json",
        "project": "projects/my-lib/ng-package.json"
      }
    },
```

### package.json

package.json에서 ng-packagr이 있는지 확인합니다.


### tsconfig.json

폴더가 어디에 생성되는지 확인합니다.

```json
"paths": {
  "my-lib": [
    "dist/my-lib"
  ],
```

### 각 파일의 목적

 > {% raw %}<package.json> {% endraw %}

   > - npm package처럼 배포할 수 있도록 함.

> {% raw %}<public_api.ts> {% endraw %}

  > - entry file. 여기에 export할 모든 파일을 포함하여야 한다.
  > - module에 export를 선언하였어도 여기에 함께 선언해야 한다.

> {% raw %}<ng-package.json> {% endraw %}

  > - ng-packagr 설정 파일. angular cli가 자동생성한다.


## library 작성

모든 component 등의 파일을 선언하고 export 합니다.

### module
```ts
import { NgModule } from '@angular/core';
import { MyLibComponent } from './my-lib.component';
import { FooComponent } from './foo/foo.component';

@NgModule({
  imports: [
  ],
  declarations: [
    MyLibComponent,
    FooComponent
  ],
  exports: [
    MyLibComponent,
    FooComponent
  ]
})
export class MyLibModule { }
```


### public_api.ts

module과 module에서 선언한 export 파일명을 작성합니다.
```ts
export * from './lib/my-lib.service';
export * from './lib/my-lib.component';
export * from './lib/my-lib.module';
```




## 빌드

빌드는 기존 프로젝트와 마찬가지로 ng build를 사용합니다.

```bash
ng build my-lib
```

실행이 성공하면 dist>my-lib 폴더가 생성됩니다.


## package 생성 

dist에 생성된 폴더로 이동해서 package 명령을 내려야 하므로 폴더를 이동합니다.

```bash
cd dist/my-lib
```

### public package

npm publish 로 public package를 만들 수 있습니다. 

```bash
npm publish
```

이 때, npm에 로그인 되어있지 않으면 401 에러가 발생합니다.<br><br>

npm link 명령을 사용하면 빌드시 매번 라이브러리를 재설치 하는 과정을 피할 수 있습니다.


#### public 에러
```
npm ERR! code E401
npm ERR! 401 Unauthorized - PUT https://registry.npmjs.org/my-lib - You must be logged in to publish packages.

npm ERR! A complete log of this run can be found in:
```


### private package

만일 public package로 만들기 원하지 않으시면 npm pack 명령을 사용하면 됩니다.<br>
실행하면 npm publish와 같이 동작하나 로그인 과정을 거치지 않고 완료하며 tgz 파일을 생성합니다.

```
> npm pack

npm notice
npm notice package: my-lib@0.0.1
npm notice === Tarball Contents ===
npm notice 1.4kB esm2015/lib/my-lib.component.js
-- 생략 --
npm notice 60B   lib/my-lib.service.d.ts
npm notice 118B  public-api.d.ts
npm notice === Tarball Details ===
npm notice name:          my-lib
npm notice version:       0.0.1
npm notice filename:      my-lib-0.0.1.tgz
-- 생략 --
npm notice total files:   26
npm notice
my-lib-0.0.1.tgz
```

### package.json 에 자동화 코드 설정

쉬운 빌드 -> package화를 위해 package.json에서 스크립트로 만들어두면 쉽습니다.

```json
"scripts": {
  "build_lib": "ng build --prod my-lib",
  "npm_pack": "cd dist/my-lib && npm pack",
  "package": "npm run build_lib && npm run npm_pack"
},
```

이제 npm package 명령을 내리면 한번에 빌드부터 package까지 진행되어 tgz 파일을 생성합니다.


## 적용

생성한 라이브러리를 사용하기 위해 적용하고자 하는 프로젝트에서 package를 install 해야 합니다.<br>
public 으로 생성한 경우 기존 npm 라이브러리와 마찬가지로 진행하고, <br>
private 인 경우 먼저 tgz 파일을 해당 프로젝트로 옮겨야 합니다.<br><br>

파일을 이동하였으면 tgz 파일을 public package와 마찬가지로 install 합니다.

```bash
npm install ./my-lib
```

이 때, 해당 모듈이 node_modules 폴더에 설치되었는지 확인합니다.




## 사용

이제 프로젝트에서 라이브러리를 사용할 수 있습니다.

```ts
// app.module.ts
import { NgModule } from '@angular/core';
import { MyLibModule } from 'my-lib';

@NgModule({
  imports: [
  ],
  declarations: [
    MyLibModule
  ],
  exports: [
    
  ]
})
export class AppModule { }
```


## 유의점

기존 앱 프로젝트에 함께 라이브러리 프로젝트를 생성할 수 없습니다. 반드시 별도의 프로젝트로 생성하여야 합니다.<br><br>

angular-cli v6.0.3 기준으로 아래의 문제가 있습니다. 버전에 따라 개선되거나 변경될 수 있으니 참고 하시기 바랍니다.
- library 빌드 시 assets이 포함되지 않는 문제가 있습니다. assset 폴더를 직접 복사하도록 하고 있습니다.
- 해결될 때까지는 각 프로젝트에 library의 assets를 복사하여야 합니다.
- 라이브러리가 스크립트를 사용하는 경우에도 적용되지 않을 수 있습니다. 각 프로젝트 angular.json 파일에 아래와 같이 스크립트 코드를 추가하여 실행하여야 합니다.



참고: 
> [Creating a Library in Angular 6 – Angular In Depth](https://blog.angularindepth.com/creating-a-library-in-angular-6-87799552e7e5)

> [Creating Libraries](https://angular.io/guide/creating-libraries)
