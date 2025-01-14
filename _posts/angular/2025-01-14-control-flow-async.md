---
title: "Control flow - How to use async pipe with alias"
date: 2025-01-14 10:57:00 +0900
comments: true
categories: angular
tags: [controlflow, ngif, ngfor, async]
---

이번 시간에는 Control folw에서 async pipe를 사용하는 방법과 alias 설정하는 방법을 알아보겠습니다.

# html에서 Observable 변수에 async pipe 사용하기
html에서 Observable 변수를 사용할 때 async pipe를 사용하면 Observable이 발행하는 값을 구독할 수 있습니다.
```html
<div>
	{{ observable$ | async }}
</div>

하지만 Observable 변수를 사용할 때마다 async pipe를 사용해야 하므로 사용이 불편하고 코드가 반복되는 단점이 있습니다.

```html
<div>{{ observable$ | async }}</div>
<div>{{ observable$ | async }}</div>
<div>{{ observable$ | async }}</div>
```

# CommonModule에서 async에 alias 활용하기
반복을 줄이는 가장 좋은 방법은 ngIf에 async pipe를 사용하고 alias를 설정하는 것입니다.


```html
<ng-container *ngIf="observable$ | async as data">
	<div>{{ data }}</div>
	<div>{{ data }}</div>
	<div>{{ data }}</div>
</ng-container>
```

ngFor에서도 async pipe를 사용하고 alias를 설정할 수 있습니다.
```html
<ng-container *ngFor="let item of observable$ | async as data">
	<div>{{ item }}</div>
</ng-container>
```

이렇게 Observable 변수를 CommonModule을 통해 async pipe를 연결하고 alias를 정의하면, 해당 scope 내에서는 일반 변수처럼 활용할 수 있으며, 반복되는 코드를 줄일 수 있습니다.



# Control flow에서 async에 alias 사용하기
CommonModule 대신 Control flow를 사용할 수 있는데 Control flow에서도 async pipe에 alias를 설정할 수 있습니다.
단, CommonModule과는 다르게 괄호로 감싸지 않고, 세미콜론(;)을 사용하여 구분합니다.

```html
@if (observable$ | async; as data) {
	<div>{{ data }}</div>
	<div>{{ data }}</div>
	<div>{{ data }}</div>
}
```


# control flow에서 for문을 사용할 때 track은 필수 입니다.
Control flow에서 for를 사용할 때 에러가 발생한다면 track을 사용했는지 체크합니다. track은 async pipe의 trackBy와 동일한 역할을 하며, Control flow에서는 필수로 사용해야 합니다.

```html
@for(let item of data; track item.id) {
	<div>{{ item.id }}</div>
}
```



# Reference
[Control flow for with async pipe](https://stackoverflow.com/questions/78549745/angular-new-control-flow-for-with-async-pipe-and-aliasing-variable-with-as)