---
title: "Excution Time"
date: 2019-07-22 14:42:00 +0900
comments: true
categories: javascript
tags: [console, time]
---

Let's explore methods for measuring JavaScript execution time.

The conventional approach involves using the `getTime()` method of the `Date` object to calculate time differences. The resulting value represents milliseconds.

```jsx
const beginTime = new Date().getTime();

// Code to be timed

console.log(new Date().getTime() - beginTime);
```

Alternatively, for quick verification of processing speed during development, the `console.time` and `console.timeEnd` functions can be employed.

`console.time` marks the starting point, and `console.timeEnd` marks the ending point. The `str` argument passed to both functions must have the same value. The output represents the elapsed time in milliseconds.

```jsx
const timerName = "myTimer"; // Use a descriptive timer name
console.time(timerName);

// Code to be timed

console.timeEnd(timerName);
```

## References

- [JavaScript Execution Time Measurement - Zeta Wiki](https://zetawiki.com/wiki/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8_%EC%88%98%ED%96%89%EC%8B%9C%EA%B0%84_%EC%B8%A1%EC%A0%95)
- [NodeJS Program Execution Time Measurement : Naver Blog](http://blog.naver.com/PostView.nhn?blogId=deepplin&logNo=60203067634)