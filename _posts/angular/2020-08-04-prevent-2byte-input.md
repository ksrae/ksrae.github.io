---
title: "Input에서 2바이트 글자 막기 (Prevent 2byte-word Input)"
date: 2020-08-04 17:48:00 +0900
comments: true
categories: angular
tags: [input, form]
---


숫자만 입력 받는 Directive를 만들려고 하다가 마침 아래의 사이트가 있기에 바로 적용하였습니다.

[Digit Only Directive in Angular](https://codeburst.io/digit-only-directive-in-angular-3db8a94d80c3)

이 사이트의 코드를 적용해보니 영어는 입력이 잘 막혀서 문제가 없었는데 2byte 글자인 한글은 막아지지 않는다는 문제를 발견하였습니다.<br>

결국 직접 수정하기로 하고, 강제로 입력을 막기 위해 onPress, onKeyDown 등의 키보드 이벤트를 수정하였습니다.<br><br>

마침내 입력을 막을 수는 있었으나 여전히 입력 중인 글자는 막을 수 없었고, 결국 blur 때에 한번 더 체크해서 이를 제거해야 했습니다.<br><br>

그래서 또 다른 방법을 찾던 중 아래의 사이트를 발견하게 되었습니다.  <br>

기존에 한 글자 정도 보이는 것을 이 코드를 적용하면 아예 2-바이트도 보이지 않는 상태를 만들어 주며, 아주 심플하면서도 유용하여 적극 추천합니다.<br><br>

(코드가 간단하게 구성되었고 원문을 꼭 방문하시라고 별도의 설명을 생략하였습니다.)

[angular-numbers-only-directive - StackBlitz](https://stackblitz.com/edit/angular-numbers-only-directive?file=app%2Fnumbers-only.directive.ts)