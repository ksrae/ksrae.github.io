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
	{% raw %}
	{{ observable$ | async }}
	{% endraw %}
</div>
```

하지만 Observable 변수를 사용할 때마다 async pipe를 사용해야 하므로 사용이 불편하고 코드가 반복되는 단점이 있습니다.

```html
<div>{% raw %}{{ observable$ | async }}{% endraw %}</div>
<div>{% raw %}{{ observable$ | async }}{% endraw %}</div>
<div>{% raw %}{{ observable$ | async }}{% endraw %}</div>
```

# CommonModule에서 async에 alias 활용하기
반복을 줄이는 가장 좋은 방법은 ngIf에 async pipe를 사용하고 alias를 설정하는 것입니다.


```html
<ng-container *ngIf="observable$ | async as data">
	<div>{% raw %}{{ data }}{% endraw %}</div>
	<div>{% raw %}{{ data }}{% endraw %}</div>
	<div>{% raw %}{{ data }}{% endraw %}</div>
</ng-container>
```

ngFor에서도 async pipe를 사용하고 alias를 설정할 수 있습니다.
```html
<ng-container *ngFor="let item of observable$ | async as data">
	<div>{% raw %}{{ item }}{% endraw %}</div>
</ng-container>
```

이렇게 Observable 변수를 CommonModule을 통해 async pipe를 연결하고 alias를 정의하면, 해당 scope 내에서는 일반 변수처럼 활용할 수 있으며, 반복되는 코드를 줄일 수 있습니다.



# Control flow에서 async에 alias 사용하기
CommonModule 대신 Control flow를 사용할 수 있는데 Control flow에서도 async pipe에 alias를 설정할 수 있습니다.
단, CommonModule과는 다르게 괄호로 감싸지 않고, 세미콜론(;)을 사용하여 구분합니다.

```html
@if (observable$ | async; as data) {
	<div>{% raw %}{{ data }}{% endraw %}</div>
	<div>{% raw %}{{ data }}{% endraw %}</div>
	<div>{% raw %}{{ data }}{% endraw %}</div>
}
```


# Reference
[Control flow for with async pipe](https://stackoverflow.com/questions/78549745/angular-new-control-flow-for-with-async-pipe-and-aliasing-variable-with-as)