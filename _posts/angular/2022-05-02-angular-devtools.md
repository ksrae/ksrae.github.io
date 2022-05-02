---
title: "Chrome DevTools에 Angular DevTool 적용하기 (Angular Devtool in Chrome DevTools)"
date: 2022-05-02 14:02:00 +0900
comments: true
categories: angular
tags: [chrome, tool]
---

이번 시간은 chrome의 개발자 도구(DevTools)에 Angular 탭을 추가하는 방법과 사용 방법에 대하여 알아보겠습니다. <br/>


## Chrome DevTools에 Angular 탭 추가하기
아래의 단계를 통해 Chrome DevTools에서 Angular 탭을 추가해 봅시다.<br/>
- [Chrome Web Store](https://chrome.google.com/webstore/category/extensions) 에 접속하여 Angular DevTools를 검색하거나, 또는 [Angular DevTools](https://chrome.google.com/webstore/detail/angular-devtools/ienfalfjdbdpebioblfackkekamfmbnh) 링크를 클릭합니다. <br/>
- 설치하기를 눌러 설치를 마칩니다. 설치가 되면 해당 버튼은 "Chrome에서 삭제" 라는 문구로 변경됩니다. <br/>
- 새 창을 열거나 Chrome을 재시작하면, Chrome DevTools에서 Angular 탭이 추가된 것을 확인할 수 있습니다. <br/>


## Angular DevTool 사용하기
Angular DevTool을 사용하기 위한 제약조건을 알아보고, 사용 방법도 함께 알아보겠습니다.

### 제약조건
Angular DevTool은 Angular로 개발된 사이트를 분석할 수 있도록 도와주는 개발자 도구입니다. <br/>
그러나, 사용하려면 아래와 같은 몇 가지 제약이 있으므로 유의하여야 합니다. <br/>
- Angular로 개발된 사이트만 분석할 수 있습니다. <br/>
- Version 9 이상이어야 하며, 반드시 ivy를 지원해야 합니다. <br/>
- production 환경으로 빌드된 프로젝트는 분석할 수 없습니다. 오직 Develop (또는 dev) 환경에서만 가능합니다. <br/>

### 사용방법
좌측에는 component 단위로 구분된 component 항목이 있으며 해당 항목을 클릭할 경우 해당하는 template이 표시되며, 우측에는 아래와 같은 component의 속성이 표시됩니다. <br/>
- Input과 Output 변수 <br/>
- component의 변수 <br/>
- component에서 사용하는 service의 변수 <br/>

이 Tool의 장점은, 서비스의 경우 서비스에 연결된 프로퍼티도 확인가능하며, 상태관리인 경우도 확인이 가능하여 변수가 현 상황에 어떤 값을 가지고 있는지 확인할 수 있다는 점이며, <br/>
또 하나의 장점이라고 한다면 실시간으로 변경되는 값이 모두 반영된다는 것입니다. 즉, 초당 변경되는 값을 저장하는 변수가 있다고 가정할 때 매 초 해당 변수의 값이 변경되는 것을 확인할 수 있습니다. <br/>

어떠한 컴포넌트에 어떠한 변수가 연결되어 있는지 쉽게 확인할 수 있으므로 콘솔을 통해 디버깅하는 횟수를 줄일 수 있을 것으로 기대합니다. <br/>

자세한 사항은 Angular 공식 사이트를 참조하세요.

## 참고사이트
- [Angular DevTools Overview](https://angular.io/guide/devtools)