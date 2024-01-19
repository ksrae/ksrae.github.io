---
title: "private 라이브러리 만들기 (Creating Private Library) - revised"
date: 2024-01-18 19:50:00 +0900
comments: true
categories: angular
tags: [library, package]
---

2019년 7월에 작성한 [Creating Library 문서](https://ksrae.github.io//angular/creating-library/)가 미흡하여 보다 이해하기 쉽게 재작성하였습니다.<br/>
<br/>
이 글에서는 자체 개발한 라이브러리를 마치 외부 라이브러리처럼 사용하는 방법에 대해 다루어 보겠습니다.<br><br>

Angular6 부터 ng-packagr가 정식 지원 되어 쉽게 만을 수 있으니 자주 사용하는 기능은 라이브러리로 만들고 package화 해서 편리하게 재활용 하시기 바랍니다.<br/>


## 1. 프로젝트 생성

일반 프로젝트를 생성합니다.

```
ng new app
```

## 2. library 를 프로젝트에 추가
cli를 활용하여 library를 생성합니다.<br>

```bash
ng generate library [library-name]
```

그러면 angular root 폴더에 projects 폴더 안에 library-name으로 폴더가 생성되어 있습니다.<br/>
참고로 이 글에서는 library명을 mylib로 하였습니다.

## 3. library 작성
원하시는 라이브러리 코드를 작성합니다.
 
## 4. public_api.ts 작성
작성한 모든 component, module(프로젝트가 module based 인 경우), service 등을 아래와 같이 import 합니다.<br/>


module과 module에서 선언한 export 파일명을 작성합니다.
```ts
export * from './lib/my-lib.service';
export * from './lib/my-lib.component';
export * from './lib/my-lib.module';
```


## 4-1. module based 프로젝트의 경우 exports에 추가
모든 component 등의 파일을 declalations에 선언하고 export에 추가합니다.<br/>
exports에는 service는 포함하지 않습니다.<br/>
이 과정은 standalone은 관계가 없으므로 스킵합니다.

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

## 5. package.json에 script 추가

쉬운 package 화를 위해 script 명령어를 package.json에 추가합니다.

```
"scripts": {
  "packagr": "ng-packagr -p ng-package.json"
}
```

## 6. 빌드(script 실행)
command 창에서 npm 명령을 실행합니다.
```
npm run packagr
```

## 7. 폴더 확인
dist/[library-name] 폴더가 생성되어 있습니다. <br/>
이 글에서는 dist/mylib가 생성되었습니다.

## 8. package화 
dist/[library-name] 폴더로 이동하여 npm 명령을 수행합니다.<br/>
이 과정은 library를 package화 하는 과정입니다.

```
npm pack
```

## 9. 확인
[library-name]-[version].tgz 파일이 생성된 것을 확인합니다. 여기서는 mylib-0.0.1.tgz 파일이 생성되었습니다.<br/>
<br/>
이제 library 생성은 완료하였고, 어플리케이션에서 이 library를 호출해봅시다.

## 10. 새 프로젝트 생성
만들어진 library를 사용할 프로젝트를 생성합니다.
```
ng new newApp
```

## 11. tgz 파일 복사
library 에서 생성한 tgz 파일을 새 프로젝트의 폴더에 복사합니다. 위치는 상관 없습니다.<br/>
이 글에서는 local-package 폴더를 생성하고 그 안에 mylib-0.0.1.tgz 파일을 복사하였습니다.

## 12. tgz install
생성한 폴더로 이동합니다. root이면 이동하지 않아도 됩니다.<br/>
그리고 tgz 파일을 install 합니다. 과정은 npm package 들과 동일하지만 여기서는 파일명을 입력하는 것만 주의하면 됩니다.
```
npm i [library-name]-[version].tgz
```
여기서는 `npm i mylib-0.0.1.tgz` 를 실행하였습니다.

## 13. package.json 확인
package.json 파일을 열어 dependencies에 올바르게 설치되었는지 확인합니다.

```
"dependencies": {
  [library-name]: "file:[folder]/[library-name]-[version].tgz
}
```
여기서는 `mylib: file:local-package/mylib-0.0.1.tgz` 가 추가되었습니다. <br/>
만일 없거나 임의로 파일의 위치를 변경한 경우 수동으로 입력해줍니다. 

## 14. 사용
component에 적용합니다. library의 selector는 library를 생성했을 때 component의 selector를 사용합니다.<br/>
module based 인 경우 module에 library module을 추가해주어야 하며, standalone component는 호출할 수 없습니다.<br/>
standalone의 경우 library의 module 또는 component(standalone인 경우) 를 imports 해야 합니다.

```
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [CommonModule, RouterOutlet, MylibModule],
  template: '<lib-mylib></lib-mylib>',
})
export class AppComponent {
  title = 'app-call-local-library';
}
```

## 결론
손쉽게 library를 작성하고 이를 private package로 만들어 배포하는 방법에 대해 알아보았습니다.<br/>
library 내에서 다른 외부 라이브러리를 사용해도 활용할 때 같이 적용되므로 개발하고 적용하는데에 큰 어려움이 없을 것으로 생각합니다.<br/>


## 느낀점
이전 글에서는 public package 까지 모두 포함한 글을 간단하게 정리하려다보니 두서없는 글이 되어버려 혼선이 많았습니다.<br/>
글을 간단하게 작성하면서도 정확히 전달하는 것이 매우 어려움을 다시금 깨닳게 되었고 더 직관적이고 쉬운 글을 작성하도록 노력하겠습니다.<br/>

