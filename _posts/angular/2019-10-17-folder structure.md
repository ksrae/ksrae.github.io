---
title: "folder structure modification"
date: 2019-10-17 12:19:00 +0900
comments: true
categories: angular
tags: []
---


## 기본구조

Angular에서 기본으로 제공하는 구조는 하나의 폴더에 한 페이지에 관련된 파일이 모두 들어가는 구조입니다.

![folder structure modification-1.PNG](/assets/img/angular/folder structure modification-1.PNG)

즉, 폴더 하나에 component, html, css, module 등 페이지를 구성하는 모든 요소를 넣습니다.



## 의문

이대로 구성하여 실제 업무에 적용해보면 불편한 점들이 많습니다.<br>
특히, 세분화된 팀의 경우 더욱 그렇습니다.<br>
<br>
예를 들어, 퍼블리셔/프론트 또는 그 이상으로 세분화된 경우는 더욱 그렇습니다.<br>
하나의 폴더에 여러명이 접근하다보니 불필요한 파일을 봐야 하는 불편함이 있습니다.<br>

다시 말해서 퍼블리셔의 경우 css 파일과 html 파일을 주로 접근하지만 component, module은 접근하지 않습니다.<br>
<br>
반대로, 프론트 개발자의 경우는 그 반대로 component, module, 그리고 종종 html 파일에 접근하지만 css 파일은 접근하지 않습니다.<br>
<br>
service의 경우 여러 component에서 접근할 수 있는데 이 때는 어느 폴더에 넣어야 할지 난감합니다.<br>
<br>
따라서 실제 업무에 유용한 폴더 구조를 구성할 방법을 구상하였고, 그 결과 현재는 다음과 같은 구조를 기본으로 협업에 활용하고 있습니다.<br>



## 변경

폴더 구조를 변경할 때 아래와 같은 사항을 고려하였습니다.<br>

> 필요없는 파일에 접근할 필요 없게 하자<br>
> 가능한 depth를 줄이자<br>
> 각 파일의 특징과 기능별로 묶자<br>

고려한 사항을 바탕으로 구성한 폴더 구조는 아래 그림과 같습니다.<br>

![folder structure modification-2.PNG](/assets/img/angular/folder structure modification-2.PNG)


> component: <i>component 파일만 모읍니다. 확장자를 .component.ts로 구성합니다.</i><br>
> declarations: <i> d.ts 파일 모음입니다.</i><br>
> directives: <i> directive 파일 모음 입니다.</i><br>
> guards: <i>guard 파일 모음입니다.</i><br>
> helpers: <i>service에서 inject하는 service를 helper라고 명명하고 여기에 담습니다.</i><br>
> interfaces: <i>interface 파일 모음 입니다.</i><br>
> modules: <i>module 파일 모음입니다.</i><br>
> pipes: <i>pipe 파일 모음입니다.</i><br>
> services: <i>component에서 inject하는 service 파일 모음입니다.</i><br>
> styles: <i>css 파일 모음입니다.</i><br>
> templates: <i>html 파일 모음입니다.</i><br>


#### 주의할 점

component / module / style / template 은 접근하기 쉬워야 하고 개발자들 간의 쉬운 소통을 위해 같은 폴더 구조를 갖습니다.<br>
<br>
그러기 위해 같은 페이지에 들어갈 파일은 모두 같은 폴더 구조 내에 넣어야 합니다.<br>

![folder structure modification-3.PNG](/assets/img/angular/folder structure modification-3.PNG)




## 결론

위와 같이 구성하는 경우 아래와 같은 장점이 있습니다.<br>

- 참고할 필요 없는 파일은 접근할 필요가 없어서 집중도가 높습니다.<br>
- 기능별로 구분하였기 때문에 유사한 파일의 참고가 쉽습니다.<br>
- 폴더 depth가 얕으므로 파일 접근이 쉽습니다.<br>

<br>
물론 단점도 있습니다.<br>

- 유사한 폴더를 여기저기 만들어야 하므로 전체적인 폴더 숫자가 많습니다.<br>
- component / module / style / template 같은 폴더 구조를 유지하여야 하므로 폴더 관리자가 필요합니다.<br>
<br>

비록 위와 같은 단점이 있지만 협업에 유리하다는 더 큰 장점이 있으므로 위와 같은 구조로 구성하실 것을 강력 추천 드립니다.<br>
<br>
끝.
