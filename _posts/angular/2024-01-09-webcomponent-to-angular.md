---
title: "Angular에 작성한 Web Component 적용하기 ()"
date: 2024-01-09 01:14:00 +0900
comments: true
categories: angular
tags: [webcomponent]
---

이 글에서는 지난 시간에 작성했던 [webcomponent](https://ksrae.github.io/angular/webcomponent/) 를 angular project에 적용하는 방법에 대해서 다루어 보겠습니다.<br/>
특히 이 글에서는 index.html이 아닌 component 내에서 webcomponent를 호출하는 방법을 정리하려고 합니다.<br/>

## webcomponent 준비
[지난 글을 참조](https://ksrae.github.io/angular/webcomponent/)하여 webcomponent를 작성합니다.

## 새 프로젝트 생성
webcomponent를 적용하기 위한 새로운 angular project를 생성합니다.

```
npm new angular-webcomponent
```


## script 파일 복사
이전 글에서 작성했던 webcomponent의 3 파일을 새 프로젝트로 복사합니다.
- bundle.js
- main.js
- polyfills.js

새 프로젝트의 위치는 어느 곳이든 상관 없으나 여기에서는 src/scripts 폴더를 생성한 뒤 파일을 복사하였습니다.

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


### angular.json에 추가 (failed)
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


### index.html에 추가 (succeed)
script 파일은 index.html에 {% raw %}<script>{% endraw %} 태그에 추가할 수 있습니다.<br/>
이 경우 script 경로는 외부에서 인식이 가능한 (일반적으로) assets 폴더 내에 있으면 편리합니다.

```html
...
<script src="./assets/scripts/bundle.js"></script>
```

단, 이 경우 script를 global하게 호출하므로 필요하지 않은 경우에도 로드되어 있어 최적화에 좋지 않으므로 가벼운 프로젝트나 자주 사용하는 경우에만 이 방법을 사용하는 것이 좋습니다.


### component에 import 하기 (succeed, 추천)
component에서 다음과 같이 js를 형태 그대로 import 할 수 있습니다.

```ts
import "./src/scripts/bundle.js";
```

필요한 component에서만 import 하므로 성능 최적화에 좋은 방법이므로 이 방법을 추천합니다.


## CUSTOM_ELEMENTS_SCHEMA 설정

위와 같이 정의하여도 실행되지 않는데 이는 webcomponent는 angular 프로젝트 내에서 작성된 component가 아니기 때문입니다.<br/>
따라서 외부 component를 허용하도록 Schmema를 설정해주어야 합니다.

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