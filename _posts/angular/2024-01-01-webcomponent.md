---
title: "Angular Standalone 환경에서 Web Component 만들기 (Creating Web Component in Angular: Standalone Based)"
date: 2024-01-01 14:46:00 +0900
comments: true
categories: angular
tags: [standalone, webcomponent]
---

이 글에서는 Angular에서 Web Component를 만드는 과정을 살펴보겠습니다. <br/>
이 글에서는 Angular Standalone 환경일 때 Web Component를 만드는 방법에 대해 정리하였습니다.<br/>
많은 글들이 Module 방식에 중점을 두고 있으므로 Module 방식은 다른 문서를 확인하시기 바랍니다.


## Workspace 설정
쉬운 관리를 위해 먼저 workspace를 생성합니다.

```ng new workspace -no-create-application```

그런 다음, Web Component 의 소스가 될 Library와 이를 감싸 Web Component로 변환할 Application을 추가합니다.

```ng g library web-component-lib``` <br/>
```ng g application web-component-app```





## library 구현

라이브러리 코드에 원하는 기능을 작성합니다. 필요한 경우 Angular의 기능 (예: @Input 및 @Output)을 포함시켜도 됩니다.<br/>
필요한 경우 Shadow DOM을 설정합니다.

```ts
@Component({
  selector: 'lib-web-component-lib',
  standalone: true,
  imports: [],
  template: `
  {% raw %} {{ setValue }} {% endraw %}
  <button (click)="onClick($event)">Emit</button>
  `,
  styles: ``,
  encapsulation: ViewEncapsulation.ShadowDom // optional
})
export class WebComponentLibComponent {
  @Input() setValue!: string;
  @Output() onResult = new EventEmitter();

  onClick(e: any) {
    this.onResult.emit(e);
  }
}
```



## application 구현

웹 페이지를 호출하는 것이 목적이 아닌 library를 호출하는 것이 목적이므로 몇가지 기존 설정의 변경이 필요합니다.

### app folder 제거
이 application에서는 appComponent가 아닌 library를 직접 호출하여야 하므로 불필요한 app 폴더를 제거합니다.


### main.config.ts 추가
src 폴더에 `main.config.ts` 파일을 추가하여 다음의 코드를 추가합니다. 
- 필요한 경우 provider를 추가 합니다.

```ts
import { ApplicationConfig } from '@angular/core';

export const appConfig: ApplicationConfig = {
    providers: [
      // add code here
    ],
};
```


## @angular/elements 설치
library를 webcomponent로 만들기 위해 @angular/elements의 createCustomElement() 함수가 반드시 필요합니다. <br/>
이를 사용하기 위해 @angular/elements를 install 해야 합니다.

```
npm i @angular/elements
```



### main.ts 수정

기존 `main.ts` 파일의 내용을 모두 제거하고 다음의 코드로 대체합니다.

```ts
import {createApplication} from '@angular/platform-browser';
import {appConfig} from './main.config';
import {createCustomElement} from '@angular/elements';
import { WebComponentLibComponent } from 'web-component-lib';
import { ApplicationRef } from '@angular/core';

(async () => {
  const app: ApplicationRef = await createApplication(appConfig);

  // Define Web Components
  const webComponentLibComponent = createCustomElement(WebComponentLibComponent, {injector: app.injector});
  customElements.define('web-component-lib', webComponentLibComponent);
})();
```
customElements.define의 첫 번째 매개변수는 외부에서 이 library를 호출하는 이름이 됩니다.


### index.html 수정

기존 app-root를 제거하고 호출할 라이브러리의 이름을 추가합니다.<br/>
라이브러리의 이름은 customElements.define의 첫 번째 매개변수와 동일해야 합니다.

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <title>Web Component</title>
        <base href="/" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
    </head>
    <body>
        <web-component-lib></web-component-lib>
        <script src="./main.js" type="module"></script>
        <script src="./polyfills.js" type="module"></script>
    </body>
</html>
```

#### input, output 작성
만일 library에 input과 output이 있을 경우 webcomponent에서도 사용할 수 있습니다. <br/>
input의 경우 attribute와 유사한 방식으로 사용할 수 있으며, (단, 대문자가 들어간 경우 ```하이픈(-)소문자``` 로 표시해야 합니다. ) <br/>
output의 경우 addEventListener로 이벤트를 가져와야 합니다. <br/><br/>

index.html을 다음과 같이 고쳐봅시다.

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <title>Web Component</title>
        <base href="/" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
    </head>
    <body>
        <web-component-lib set-value="'sample'"></web-component-lib>
        <script src="./main.js" type="module"></script>
        <script src="./polyfills.js" type="module"></script>
        <script>
          const el = document.querySelector('web-component-lib'); // or set id to the dom and find the id.
          el.addEventListener('onResult', (event) => {
            console.log({event});
          });
        </script>
    </body>

</html>
```


### angular.json 수정

"outputHashing": "all" 을 제거하거나 "none"으로 수정합니다. 만일 "outputHashing" 값이 "all"로 설정되어 있으면 빌드 할 때마다 main-[hash].js 와 같이 파일명에 hash가 붙어 빌드할 때마다 파일명이 변경되기 때문에 이를 방지하기 위함 입니다.


## build

library와 application을 각각 build 합니다.
package.json에 등록해두면 편리합니다.

```json
    "build:lib": "ng build web-component-lib --configuration production",
    "build:app": "ng build web-component-app --configuration production",
```


## 실행
빌드된 app이 정상적으로 작동하는지 확인하기 위해 dist 폴더에서 app을 찾아 http-server에 올려 보세요. (단순히 파일을 실행해서는 동작하지 않습니다.) <br/><br/>

http-server가 없다면 크롬의 확장 프로그램 (예: [Simple WebServer](https://simplewebserver.org/download.html) )을 활용할 수도 있습니다.
 

## 개선 
dist 폴더에서 app 폴더의 파일 중 index.html 파일을 보면 main.js와 polyfills.js를 script로 호출하고 있는 것을 볼 수 있습니다.<br/>
여러 library가 있을 경우 파일명이 모두 동일하므로 사용이 불편할 수 있으므로 이를 개선해봅시다.

#### script 파일 추가
scripts 폴더를 생성하고 postbuild-bundler.js 을 생성합니다.

```js
async function loadModules() {
    await import('./polyfills.js');
    await import('./main.js');
  }

  loadModules();
```

#### angular.json 수정
angular.json 파일을 수정합니다.

```json
// angular.json
"scripts": [
    {
        "input": "./scripts/postbuild-bundler.js",
        "bundleName": "bundle",
        "inject": false
    }
]
```

#### package.json 수정

빌드할 때 script 파일도 동시에 실행되도록 package.json에 script를 하나 더 추가합니다.

```json
"build:all": "npm run build:lib && npm run build:app && node ./postbuild-bundler.js",
```

### 빌드

아래의 명령을 실행하여 빌드와 동시에 script 파일도 생성되도록 합니다.
<br/>
```npm run build:all```


#### index.html 수정

이제 dist 파일의 app 폴더의 index.html 에서 두개의 스크립트를 호출하는 코드를 제거하고 bundle.js 만 호출하도록 수정합니다.


```html
...
<script src="bundle.js" type="module"></script>
<!-- remove those codes
  <script src="./main.js" type="module"></script>
  <script src="./polyfills.js" type="module"></script> 
-->
...

```


## package
library를 별도로 package화 할 수 있으므로 환경에 따라 다양하게 구성할 수 있습니다.


## import 방법

webcomponent를 사용하는 방법에 대해서는 [Web Components In Angular](https://coryrylan.com/blog/using-web-components-in-angular) 문서를 참고하고, 필요한 경우 관련 문서를 별도로 작성할 예정입니다.


## 고려사항

빌드된 웹 컴포넌트를 사용하려면 main.js, polyfills.js, bundle.js 파일을 꼭 포함해야 합니다. <br/>
이를 하나의 파일로 결합하면 더 편리할 것 같지만 마땅한 해결책을 찾을 수 없습니다.<br/><br/>

fs-extra와 concat을 install 하여 main.js와 polyfills.js를 하나의 파일로 합치는 방법을 제시하는 문서를 참고하였으나 실행하였을 경우 중복된 변수에 대한 에러가 발생하여 사용할 수 없습니다.


