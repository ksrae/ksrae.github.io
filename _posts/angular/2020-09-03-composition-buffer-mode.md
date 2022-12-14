---
title: "2바이트 문자 입력 시 마지막 글자 반영되지 않는 문제 (missing the last word while input 2-byte letters)"
date: 2020-09-03 17:31:00 +0900
comments: true
categories: angular
tags: [input, form]
---



## 원인
여러 앱에서도 종종 발생하는 문제로 input에서 한글 또는 2-byte 글자를 입력하다보면 마지막 글자가 빠지는 현상이 나타납니다.<br/>
이는 글자와 관계없는 키(예를들어, 화살표나 doc키 등)을 한 번 더 입력하면 해결되는데 매우 불편한 것이 사실입니다.<br/><br>

Angular에서도 이런 문제가 있는데, 이를 해결하기 위해 `@angular/forms`에서 `COMPOSITION_BUFFER_MODE`를 제공합니다.


## 해결

최초 실행 모듈 (일반적으로 app.module) 에 `COMPOSITION_BUFFER_MODE`를 설정합니다.

```tsx
import { COMPOSITION_BUFFER_MODE } from '@angular/forms';
...

@NgModule({
    providers: [
        { provide: COMPOSITION_BUFFER_MODE, useValue: false }
    ]
})

...
```

이는 Angular 공식 사이트에서 간략하게 정리하여 소개하고 있는데 아래와 같습니다.<br/>

```
Provide this token to control if form directives buffer IME input until the "compositionend" event occurs.

"compositionend" 이벤트가 발생할 때까지 form directives가 IME 입력을 버퍼링하도록 제어하기 위해 이 토큰을 제공합니다.
```


## 참고 사이트
- [input 한글 입력시 짤림 현상](https://gogomalibu.tistory.com/151?category=851911)
- [COMPOSITION_BUFFER_MODE](https://angular.io/api/forms/COMPOSITION_BUFFER_MODE)