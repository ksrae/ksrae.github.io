---
title: "Typescript 버전 안맞을 때"
date: 2019-07-01 12:22:00 +0900
comments: true
categories: angular
tags: [typescript]
---

### 0. 개요

ng update를 진행 한 뒤 typescript 버전이 맞지 않다는 에러가 발생하는 문제를 해결합니다.


### 1. 원인

업데이트 후 ng serve 명령 시 아래와 유사한 에러 메시지가 발생합니다.

    ERROR in The Angular Compiler requires TypeScript >=3.4.0 and <3.5.1 but 3.5.1 was found instead.
    

### 2. 해결

기존 typescript를 제거하고 재설치 하는 방법입니다.

#### 2-1. TypeScript 삭제

        npm uninstall typescript

#### 2-2. TypeScript 재설치

        npm install typescript@">=3.4.0 <3.5.0" --save-dev --save--exact

재설치 시 범위를 에러 메시지와 동일하게 적용할 시 해당 범위의 가장 높은 버전이 설치 되어 에러가 해결됩니다.

