---
title: "Typescript 버전 안맞을 때"
date: 2019-07-01 12:22:00 +0900
comments: true
categories: angular
---


만일 에러 메시지가 아래와 같다면 다음과 같은 절차에 의해 TypeScript를 재설치 한다.

    ERROR in The Angular Compiler requires TypeScript >=3.4.0 and <3.5.1 but 3.5.1 was found instead.
    


1. TypeScript 삭제

        npm uninstall typescript

2. TypeScript 재설치

        npm install typescript@">=3.4.0 <3.5.0" --save-dev --save--exact

해당 범위의 가장 높은 버전이 설치 된다.

