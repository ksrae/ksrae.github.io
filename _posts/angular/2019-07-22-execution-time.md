---
title: "Excution Time"
date: 2019-07-22 14:42:00 +0900
comments: true
categories: javascript
tags: [console, time]
---


자바스크립트 수행 시간 측정 방법에 대해 알아보겠습니다.

일반적으로는 Date의 getTime()를 사용하여 시간을 계산합니다.
결과값은 ms 입니다.

```js
var beginTime = new Date().getTime();

// 처리할 코드

console.log(new Date().getTime() - startTime);
```


만일 간단히 개발과정에서의 처리 속도 확인 용도라면 console.time / timeEnd 기능을 사용할 수 있습니다.
time은 시작점, timeEnd는 끝점을 의미하며, str은 반드시 같은 값을 가져야 합니다.
마찬가지로 결과값은 ms 입니다.

```js
console.time(str);

// 처리할 코드

console.timeEnd(str);

```


참고:
[자바스크립트 수행시간 측정 - 제타위키](https://zetawiki.com/wiki/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8_%EC%88%98%ED%96%89%EC%8B%9C%EA%B0%84_%EC%B8%A1%EC%A0%95)

[NodeJS 프로그램 수행시간 측정 : 네이버 블로그](http://blog.naver.com/PostView.nhn?blogId=deepplin&logNo=60203067634)
