---
title: "Angular에 작성한 Web Component 사용하기 (How to use webcomponent in Angular)"
date: 2024-01-09 01:14:00 +0900
comments: true
categories: angular
tags: [webcomponent]
---

이번 글에서는 webcomponent를 Angular 프로젝트에서 사용하는 방법에 대해 살펴보겠습니다.<br/>
특히, index.html이 아닌 Angular component 에서 WebComponent를 호출하는 방법에 중점을 둘 것입니다.

## webcomponent 준비
[지난 글을 참조](https://ksrae.github.io/angular/webcomponent/)하여 webcomponent를 작성합니다.

## 새 프로젝트 생성
webcomponent를 적용하기 위한 새로운 angular project를 생성합니다.

```
npm new angular-webcomponent
```


## script 파일 복사
이전에 작성한 WebComponent의 3개 파일(bundle.js, main.js, polyfills.js)을 새 프로젝트로 복사합니다. <br/>
폴더의 위치는 중요하지 않습니다. 이 글에서는 src/scripts 폴더를 생성하였습니다.<br/>
<br/>
(만일 webcomponent를 npm으로 설치한 경우는 이 과정이 필요 없으므로 이 영역은 무시해도 됩니다.)


## webcomponent tag 호출
임의의 component에서 webcomponent tag를 작성합니다.

```html
<my-library-component></my-library-component>

```


## script import

angular에서 외부 스크립트를 추가하는 방법은 3가지 입니다.
- angular.json의 "script" 에 추가
- index.html에 {% raw %}<script>{% endraw %} 로 추가
- component의 import 에 추가


### angular.json에 추가 (실패)
script 파일은 다음과 같이 angular.json의 script에 추가할 수 있습니다.

```json
...
"scripts": [
  "./src/scripts/bundle.js"
]
```

이를 실행하면 다음의 경고가 나타나며 적용되지 않습니다.
```
Failed to resolve dependency: ./main.js, present in 'optimizeDeps.include'
Failed to resolve dependency: ./polyfills.js, present in 'optimizeDeps.include'
```

외부에서 접속 가능하도록 assets 폴더에 넣어도 마찬가지 현상이 발생하며, 
만일 main.js와 polyfills.js를 추가하면 중복된 함수 에러가 발생하므로 이 역시 불가능 합니다.


### index.html에 추가 (성공)
index.html에 {% raw %}<script>{% endraw %} 태그를 사용하여 스크립트 파일을 추가할 수 있습니다. <br/> 이 때 스크립트 경로는 일반적으로 assets 폴더 내에 위치하면 편리합니다.

```html
...
<script src="./assets/scripts/bundle.js"></script>
```

단, 이 방법은 script가 모든 페이지에 로드되므로 최적화에 좋지 않습니다. 작은 프로젝트나 자주 사용되는 경우에만 추천됩니다.


### component에 import 하기 (성공, 추천)
component에서 다음과 같이 js를 형태 그대로 import 할 수 있습니다.

```ts
import "./src/scripts/bundle.js";
```

이 방법은 필요한 컴포넌트에서만 스크립트를 로드하는 것이 최적화에 좋습니다. <br/>
특히 큰 프로젝트에서는 여러 컴포넌트에서 사용되는 경우에만 이 방법을 사용하세요.


## CUSTOM_ELEMENTS_SCHEMA 설정

위와 같이 정의한 뒤 프로젝트를 실행해도 webcomponent가 로드되지 않는데 이는 webcomponent는 angular 프로젝트 내에서 작성된 component가 아니기 때문입니다.<br/>
따라서 외부 component를 허용하도록 Schmema를 설정해주어야 합니다.<br/>
사용 가능한 schema는 두 가지가 있는데 사용자 지정 요소를 허용하는 ```CUSTOM_ELEMENMTS_SCHEMA``` 와 알 수 없는 요소나 속성을 사용할 때 관련 에러를 무시하도록 허용하는 ```NO_ERRORS_SCHEMA``` 가 있습니다.<br/>
상황에 따라 선택할 수 있으나 NO_ERROES_SCHMEA의 경우 관련 에러를 모두 무시하므로 필요한 에러 메시지를 받을 수 없는 상황이 있을 수 있어 사용 전에 잘 검토해야 합니다.


```ts
import { CUSTOM_ELEMENTS_SCHEMA, Component } from '@angular/core';

@Component({
  ...
  schema: [CUSTOM_ELEMENTS_SCHEMA]
})
...
```

이제 실행하면 원하는 결과대로 webcomponent를 성공적으로 호출하는 것을 확인할 수 있습니다.


component의 소스 코드는 다음과 같습니다.
```ts
import { CUSTOM_ELEMENTS_SCHEMA, Component } from '@angular/core';
import "../scripts/bundle.js";

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [],
  template: `<my-library-component></my-library-component>`,
  styleUrl: './app.component.scss',
  schemas: [CUSTOM_ELEMENTS_SCHEMA]
})
export class AppComponent {
  title = 'webcomponent-angular';
}

```


## 참고 사이트
[Using Web Components in an Angular application: Joyful & Fun](https://developer.vonage.com/en/blog/using-web-components-in-an-angular-application-joyful-fun)