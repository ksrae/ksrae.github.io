---
title: "ngDoc - Install"
date: 2023-06-13 16:12:00 +0900
comments: true
categories: angular
tags: [document, ngdoc]
---

# ngdoc으로 앵귤러 프로젝트를 간단히 문서화하기

## 소개
angular 프로젝트의 문서화를 [구글의 라이브러리](https://m3.material.io/components) 나 [MS의 Fluent2](https://fluent2.microsoft.design/) 등과 같은 형태의 문서로 작성하고 싶을 때 ngdoc를 활용하는 것을 추천합니다.

이 라이브러리는 직접 테스트해볼 수 있는 샘플과 샘플에 해당하는 코드를 제공할 수 있으며, md 스타일의 문서를 제공하므로 쉽게 문서화 작업을 진행할 수 있습니다.

이 블로그 게시물에서는 ngdoc을 설치하고 Angular 프로젝트 내에서 설정하는 과정을 안내합니다. 
몇 가지 간단한 단계만 거치면 필요한 모든 구성을 자동으로 처리하는 ngdoc을 실행할 수 있습니다.

이번 시간은 간단한 설치 방법과 각각의 기능을 추가하는 방법에 대해 설명하고,
다음에 보다 상세한 내용을 정리하는 시간을 갖도록 하겠습니다.

## ngdoc 설치
시작하려면 Angular 프로젝트에 ngdoc을 설치해야 합니다. 터미널을 열고 다음 명령을 실행합니다:

```
ng add @ng-doc/add
```
이 명령은 자동으로 ngdoc을 설치하고 프로젝트 내에 설정합니다.

### 주의
ng add @ng-doc/add는 반드시 ngdoc를 구성할 위치에서 실행해야 합니다.
예를 들어 문서화 파일을 src/docs에서 작업한다고 하면 해당 폴더까지 이동한 뒤에 위의 설치를 진행해야 합니다.


### 수동 설치(선택 사항)
설치 프로세스를 보다 세밀하게 제어하고 싶거나 자동 설치에 문제가 있는 경우, 공식 ngdoc 문서에서 수동 설치 지침을 참조할 수 있습니다. 
이 문서는 프로세스를 안내하는 자세한 단계를 제공하여 ngdoc을 성공적으로 설치하는 데 도움을 줍니다.

수동 설치 가이드를 찾으려면 NgDoc 웹사이트(https://ng-doc.com/docs/getting-started/installation)를 방문하세요.


## entity 
ngDoc를 구성하는 기본 요소를 의미합니다.
일반적으로 category, page 로 구성되며, 
non-standlone인 경우 module을 필요로 합니다.

module은 angular14 이전 버전의 개발 방식에서의 module의 역할과 유사합니다.
page에 포함될 ngDoc 외부의 compoent는 module의 declaration에 정의해야 합니다.
단, 기존 module과는 달리 page를 declaration 하거나 import할 필요는 없습니다.


### category 
category는 예를 들면 파일 시스템에서 폴더의 역할을 합니다.
유사한 문서를 그룹화 하는 역할을 담당하며 다른 역할은 없습니다.

다음의 명령을 통해 categorty를 생성합니다.
```
ng g @ng-doc/builder:category "[category명]"
```

그러면 다음과 같은 ng-doc.category.ts 파일이 생성됩니다.

```
import {NgDocCategory} from '@ng-doc/core';

export const MyAwesomeCategory: NgDocCategory = {
  title: 'MyAwesomeCategory',
};

export default MyAwesomeCategory;
```

### page 
page는 md 파일을 기본으로 하되 ng-doc.page.ts 파일을 통해 추가 기능을 쉽게 지원합니다.

다음의 명령을 통해 page를 생성합니다.
```
ng g @ng-doc/builder:page "[page명]"
```

index.md와 ng-doc.page.ts 파일이 생성됩니다.
기본 문서는 index.md 파일에 작성하고, component를 추가하는 등의 작업은 page.ts에서 작업합니다.


non-standalone인 경우에는 page 추가 시 파라미터 -m을 붙여야 합니다.
```
ng g @ng-doc/builder:page "[page명]" -m
```

이 때는 위와 같이 2개의 파일 외에도 ng-doc.module.ts 파일이 추가로 생성됩니다.
```
import {NgModule} from '@angular/core';
import {CommonModule} from '@angular/common';

@NgModule({
	imports: [CommonModule],
	// Declare you demo components here
	declarations: [],
})
export class PageModule {}
```

그리고 이 때는 ng-doc.page.ts에 아래와 같이 위의 모듈을 import 해야 합니다.

```
import {NgDocPage} from '@ng-doc/core';
import {PageModule} from './ng-doc.module';

const Page: NgDocPage = {
	title: `demo`,
	mdFile: './index.md',
  imports: [PageModule],
};

export default Page;
```

### api
