---
title: "Prevent 2byte-word Input"
date: 2020-08-04 17:48:00 +0900
comments: true
categories: angular
tags: [input, form]
---

# Implementing a Digit-Only Directive in Angular: Addressing Korean Input Issues

I initially sought to implement a directive in Angular that restricts input to numerical characters only. I came across the following resource and attempted to apply its solution:

[Digit Only Directive in Angular](https://codeburst.io/digit-only-directive-in-angular-3db8a94d80c3)

While the provided code effectively blocked English character input, I discovered an issue: it failed to prevent the entry of Korean characters, which are represented as 2-byte characters.

<br>

Consequently, I opted to directly modify the code. To enforce input restriction, I adjusted keyboard events such as `onPress` and `onKeyDown`.

<br><br>

Although I was able to successfully block most input, characters being entered during composition were still not effectively blocked. As a result, I had to implement an additional check during the `blur` event to remove these characters.

<br><br>

In search of a more refined solution, I discovered the following resource:

<br>

Applying this code prevents even the brief display of 2-byte characters during input. Its simplicity and effectiveness make it highly recommended.

<br><br>

(Due to the straightforward nature of the code, and to encourage visiting the original source, detailed explanations are omitted here.)

<br/><br/>

## Reference
[angular-numbers-only-directive - StackBlitz](https://stackblitz.com/edit/angular-numbers-only-directive?file=app%2Fnumbers-only.directive.ts)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/Input%EC%97%90%EC%84%9C-2%EB%B0%94%EC%9D%B4%ED%8A%B8-%EA%B8%80%EC%9E%90-%EB%A7%89%EA%B8%B0)
<br/>