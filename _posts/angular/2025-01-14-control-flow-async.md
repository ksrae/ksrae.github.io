---
title: "Control flow - How to use async pipe with alias"
date: 2025-01-14 10:57:00 +0900
comments: true
categories: angular
tags: [controlflow, ngif, ngfor, async]
---

[한국어(Korean) Page](https://velog.io/@ksrae/Control-flow-async-pipe%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95%EA%B3%BC-alias-%EC%84%A4%EC%A0%95%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
<br/>

Let's explore how to leverage the async pipe with alias configurations within Angular's Control Flow for managing asynchronous data streams.

# Utilizing the Async Pipe with Observables in HTML

When working with Observable variables directly within HTML templates, the async pipe provides a convenient mechanism to subscribe to the values emitted by the Observable.

```html
<div>
    {{ observable$ | async }}
</div>
```

However, repeatedly applying the async pipe to the same Observable variable can lead to verbose and repetitive code, reducing maintainability.

```html
<div>{{ observable$ | async }}</div>
<div>{{ observable$ | async }}</div>
<div>{{ observable$ | async }}</div>
```

# Leveraging Async with Aliases in CommonModule

A superior approach to minimize repetition involves employing the `ngIf` directive in conjunction with the async pipe and alias assignment.

```html
<ng-container *ngIf="observable$ | async as data">
    <div>{{ data }}</div>
    <div>{{ data }}</div>
    <div>{{ data }}</div>
</ng-container>
```

Similarly, the `ngFor` directive can benefit from async pipe usage and alias declarations for iterating over asynchronous collections.

```html
<ng-container *ngFor="let item of observable$ | async as data">
    <div>{{ item }}</div>
</ng-container>
```

By connecting Observable variables through the async pipe and defining aliases within the CommonModule's directives, we gain the ability to treat the emitted values as regular variables within the component's scope, thereby reducing redundant code.

# Utilizing Async with Aliases in Control Flow

Angular's Control Flow offers an alternative to CommonModule directives. Control Flow also supports alias declarations for async pipes.  A key distinction is the syntax: Control Flow employs parentheses and semicolon separators.

```html
@if (observable$ | async; as data) {
    <div>{{ data }}</div>
    <div>{{ data }}</div>
    <div>{{ data }}</div>
}
```

# Reference

[Control flow for with async pipe](https://stackoverflow.com/questions/78549745/angular-new-control-flow-for-with-async-pipe-and-aliasing-variable-with-as)