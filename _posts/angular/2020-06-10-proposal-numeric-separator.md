---
title: "Could Not Find Plugin proposal-numeric-separator"
date: 2020-06-11 14:05:00 +0900
comments: true
categories: angular
tags: [error]
---


## 원인
`@babel/compat-data`에 새로운 플러그인이 추가될 때 기존 `@babel/preset-env`의 버전과 충돌합니다.
<br>
`available-plugins.js` 파일에 정의되어 있지 않은 플러그인인 경우 에러를 던집니다.

## 해결
1) package-lock.json과 node_modules를 제거합니다.

2) package.json에 연관성을 정의합니다.

```
  "resolutions": {
    "@babel/preset-env": "^7.8.0"
  }
```

3) npm 설치와 빌드를 실행합니다.

```
npm inatall
npm run build
```

* 해당코드는 Angular8에서 작성하였습니다.

### 참고 사이트
- [stackoverflow/could-not-find-plugin-proposal-numeric-separator](https://stackoverflow.com/questions/60780664/could-not-find-plugin-proposal-numeric-separator)


