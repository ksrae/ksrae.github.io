---
title: "js프로젝트에서 Angular 프로젝트의 데이터 접근 (transfer data js to angular)"
date: 2019-09-27 12:01:00 +0900
comments: true
categories: angular
tags: [storage, window]
---

## Analysis: Accessing Angular from JavaScript

In Angular, accessing JavaScript files typically involves utilizing declaration files (`.d.ts`) or employing `import` statements. However, the question arises: how do we achieve the reverse – accessing Angular components or variables from within JavaScript?

Generally, direct access is not feasible due to differing scopes. Variables declared within an Angular class are, by default, inaccessible from external JavaScript libraries.

Consider this illustrative JavaScript example:

```jsx
function sayHello () {
  const hello = 'Hello CSS-Tricks Reader!'
  console.log(hello)
}
sayHello() // 'Hello CSS-Tricks Reader!'
console.log(hello) // Error, hello is not defined
```

While the externally exposed `sayHello` function is accessible, the `hello` variable declared within its scope remains private. This concept is known as scope. Conversely, accessing variables or functions declared outside a function from within that function is known as closure.

These scoping rules apply across various scenarios, but the fundamental principle remains consistent: variables declared within a specific scope (e.g., inside a function or class) are not directly accessible from outside that scope.

## Solutions: Bridging the Gap Between JavaScript and Angular

So, how can we enable JavaScript to access Angular's internal variables? In other words, how can we transmit values from JavaScript to Angular?

### 1. Leveraging Storage Mechanisms

One approach involves utilizing `localStorage` or `sessionStorage`. JavaScript can store values in storage, which Angular can then retrieve.

This method offers a straightforward solution to the problem.

However, it introduces complexities. Determining when the value has been written to storage can be challenging, necessitating periodic checks or assumptions about the data's availability. These factors can make this approach less than ideal.

### 2. Utilizing the Global Scope

Another technique involves leveraging the global scope accessible to both Angular and JavaScript. We can declare variables on the `window` object and use them to pass values.

Let's illustrate this with an example where JavaScript modifies a value after a delay, which Angular then detects and renders.

Within the Angular component, create an object named `pipe` on the `window` object. This `pipe` will serve as the conduit for transferring values from JavaScript to Angular.

```tsx
  // app.component.ts
  window['pipe'] = (value) => { 
    this.render = value;
  }    
```

Now, in JavaScript, use `setTimeout` to transmit a value to the `pipe` after a specified delay.

```jsx
// global.js
var n = 1000;

setTimeout(() => {
	window.pipe('qwerqwf');
}, n);
```

Executing this code demonstrates a functional data transfer between JavaScript and Angular.

## References

- [JavaScript Scope (scope)](https://yuddomack.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%8A%A4%EC%BD%94%ED%94%84scope)
- [Translation] JavaScript Scope and Closures](https://medium.com/@khwsc1/%EB%B2%88%EC%97%AD-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%8A%A4%EC%BD%94%ED%94%84%EC%99%80-%ED%81%B4%EB%A1%9C%EC%A0%80-javascript-scope-and-closures-8d402c976d19)